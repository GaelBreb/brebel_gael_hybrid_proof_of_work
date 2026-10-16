# WireGuard — Cheatsheet & tests

Référence des commandes WireGuard et procédures de vérification.
Interface : `wg0`. Serveur (Kali) : `10.10.10.1`. Port interne : UDP `51820`. Port exposé : UDP `42420`.

---

## 1. Gestion du tunnel

| Action | Commande |
|---|---|
| Monter le tunnel | `sudo wg-quick up wg0` |
| Couper le tunnel | `sudo wg-quick down wg0` |
| Redémarrer (après modif de conf) | `sudo systemctl restart wg-quick@wg0` |
| Activer au démarrage | `sudo systemctl enable wg-quick@wg0` |
| Désactiver au démarrage | `sudo systemctl disable wg-quick@wg0` |
| État du service | `sudo systemctl status wg-quick@wg0` |

NetworkManager (alternative client) :
```bash
nmcli connection up wg0
nmcli connection down wg0
```

---

## 2. Inspection de l'état

| Objectif | Commande |
|---|---|
| État des peers + handshakes | `sudo wg show` |
| État d'une interface précise | `sudo wg show wg0` |
| Dernier handshake uniquement | `sudo wg show wg0 latest-handshakes` |
| Octets transférés | `sudo wg show wg0 transfer` |
| Config active (mémoire) | `sudo wg showconf wg0` |
| Interface réseau + IP | `ip a show wg0` |
| Port d'écoute confirmé | `sudo ss -ulnp \| grep 51820` |

Lecture de `sudo wg show` :
- `latest handshake: … ago` → tunnel établi.
- `transfer: X received, Y sent` → trafic dans les deux sens = fonctionnel.
- `transfer: 0 B received` (seulement sent) → paquets non reçus en retour → problème réseau/clé.

---

## 3. Clés

| Action | Commande |
|---|---|
| Générer clé privée + publique | `wg genkey \| tee priv.key \| wg pubkey > pub.key` |
| Dériver publique depuis privée | `wg pubkey < priv.key > pub.key` |
| Afficher une clé | `cat pub.key` |
| Clé pré-partagée (optionnel) | `wg genpsk > psk.key` |

Note : `wg pubkey` lit sur l'entrée standard (`|` ou `<`), jamais en argument.

---

## 4. Gestion des peers (serveur)

Fichier serveur : `/etc/wireguard/wg0.conf`.

Ajout d'un peer (à chaud, non persistant) :
```bash
sudo wg set wg0 peer <CLÉ_PUBLIQUE_CLIENT> allowed-ips 10.10.10.X/32
```

Ajout persistant : ajouter un bloc dans `wg0.conf` puis `sudo systemctl restart wg-quick@wg0` :
```ini
[Peer]
PublicKey = <clé publique client>
AllowedIPs = 10.10.10.X/32
```

Suppression d'un peer :
```bash
sudo wg set wg0 peer <CLÉ_PUBLIQUE_CLIENT> remove     # à chaud
# ou retirer le bloc [Peer] du fichier puis restart
```

---

## 5. Structure des fichiers de conf

Serveur (`/etc/wireguard/wg0.conf`) :
```ini
[Interface]
Address = 10.10.10.1/24
ListenPort = 51820
PrivateKey = <clé privée serveur>

[Peer]
PublicKey = <clé publique client>
AllowedIPs = 10.10.10.X/32
```

Client (`/etc/wireguard/wg0.conf`) :
```ini
[Interface]
Address = 10.10.10.X/24
PrivateKey = <clé privée client>

[Peer]
PublicKey = <clé publique serveur>
Endpoint = kali-viro-project.freeddns.org:42420
AllowedIPs = 10.10.10.0/24
PersistentKeepalive = 25
```

Permissions obligatoires : `sudo chmod 600 /etc/wireguard/wg0.conf`.

---

## 6. Tests de bon fonctionnement

### Test 1 — Serveur en écoute (sur Kali)
```bash
sudo wg show                       # interface + peers listés
sudo ss -ulnp | grep 51820         # attendu : UNCONN 0.0.0.0:51820
```
Attendu : le peer client figure dans la liste, le port 51820 écoute en UDP.

### Test 2 — Handshake établi (sur le client, après `up`)
```bash
sudo wg show
```
Attendu : ligne `latest handshake: … seconds ago` + `transfer` avec octets **reçus** non nuls.

### Test 3 — Connectivité VPN
```bash
ping -c 4 10.10.10.1               # depuis client : atteindre le serveur
```
Attendu : réponses ICMP. (Vérifier que le pare-feu du serveur autorise l'ICMP si échec.)

### Test 4 — Accès applicatif (SSH)
```bash
ssh <utilisateur>@10.10.10.1
```
Attendu : session ouverte.

### Test 5 — Où se perdent les paquets (diagnostic)
Sur le serveur, pendant une tentative de connexion client :
```bash
sudo tcpdump -ni any udp port 51820
```
- Paquets visibles → chaîne réseau OK ; investiguer clés / `AllowedIPs`.
- Aucun paquet → perte avant le serveur ; vérifier redirections et pare-feu (section 7).

### Test 6 — Exposition externe (depuis un réseau tiers, ex. 4G)
```bash
curl -4 ifconfig.me                # confirmer une IP publique ≠ IP du site hébergeur
nmap -Pn -F <IP_PUBLIQUE>          # attendu : aucun port TCP ouvert
```
Attendu : WireGuard (UDP) invisible aux scans TCP ; aucun autre service exposé.

---

## 7. Diagnostic des pannes

| Symptôme | Cause probable | Vérification / correction |
|---|---|---|
| `0 B received`, pas de handshake | Paquet n'atteint pas le serveur | `tcpdump` (Test 5) pour localiser |
| Endpoint affiché entre `[ ]` | Résolution en IPv6 | Forcer IPv4 (IP en dur ou enreg. A seul) |
| Rien n'arrive au serveur | Redirection de port protocole/IP | Vérifier UDP + IP destination exacte |
| Rien n'arrive au serveur | Pare-feu hôte bloque le port | Autoriser UDP entrant sur le port |
| Rien n'arrive (virtualisation) | Redirection NAT manquante | Vérifier le port-forwarding de la VM |
| Handshake OK, pas de trafic | `AllowedIPs` mal défini | Vérifier correspondance IP client/serveur |
| Test local échoue par IP publique | Hairpin NAT non supporté | Tester depuis un réseau externe |

Remontée de chaîne type (client → serveur) :
`Client → routeur (redir. UDP) → hôte (pare-feu + redir. NAT) → VM (pare-feu + wg)`.
Tester chaque maillon de l'extérieur vers l'intérieur avec `tcpdump`.

---

## 8. Révocation / coupure

| Action | Commande |
|---|---|
| Révoquer un peer (à chaud) | `sudo wg set wg0 peer <CLÉ_PUB> remove` |
| Révoquer un peer (persistant) | retirer le bloc `[Peer]` + `restart wg-quick@wg0` |
| Couper le VPN (serveur) | `sudo systemctl stop wg-quick@wg0` |
| Couper l'accès Internet | désactiver la redirection de port côté routeur |
| Lister les peers autorisés | `sudo wg show wg0 peers` |
