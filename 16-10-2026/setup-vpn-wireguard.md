# Setup VPN WireGuard — accès distant à la Kali partagée

**Objectif :** l'équipe (2-6 personnes) se connecte en VPN et atterrit en **SSH sur la VM Kali**,
qui sert de poste d'attaque partagé vers le lab. Les cibles ne sont **jamais** exposées à Internet.

**Statut :** configuration validée et fonctionnelle (testée depuis l'extérieur en 4G).

**Architecture :**
```
Co-équipier (client WireGuard)
      │  Endpoint : kali-viro-project.freeddns.org:42420  (UDP)
      ▼
Freebox  ──redirection 42420/UDP──►  PC hôte Windows (192.168.1.111)
                                          │  redirection VirtualBox NAT 51820/UDP
                                          ▼
                                     VM Kali  ──► réseau lab 172.28.128.0/24
                                  (WireGuard sur 51820 + SSH)
```

**Règle des ports :** `42420` uniquement côté Internet (Freebox + endpoint client).
`51820` partout à l'intérieur (VirtualBox, Kali).

---

## Choix retenus

- WireGuard tourne **sur Kali**.
- Kali en **NAT** (isolée du LAN domestique) → double redirection (Freebox + VirtualBox).
- Accès équipe en **SSH + tmux partagé**.
- Authentification VPN **par clé publique** : seuls les détenteurs d'une clé valide entrent ;
  le port ne répond pas aux scanners.
- DNS dynamique **Dynu** : `kali-viro-project.freeddns.org`.

---

## Plan d'adressage VPN

Sous-réseau dédié au tunnel, distinct du lab :

| Rôle | IP VPN |
|---|---|
| Kali (serveur WG) | 10.10.10.1 |
| Coéquipier 1 | 10.10.10.2 |
| Coéquipier 2 | 10.10.10.3 |
| … | 10.10.10.x |

Port WireGuard interne : **UDP 51820**. Port exposé sur Internet : **UDP 42420**.

---

## Vue d'ensemble des étapes

1. Installer WireGuard + SSH sur Kali
2. Générer les clés (serveur + 1 par coéquipier)
3. Config serveur WireGuard sur Kali
4. Config client (une par personne)
5. Redirection VirtualBox (hôte → Kali)
6. Redirection Freebox (Internet → hôte)
7. DNS dynamique (IP maison qui change)
8. SSH + tmux partagé
9. Tests & vérifications
10. Durcissement / sécurité

---

## Étape 1 — Installer WireGuard + SSH sur Kali

```bash
sudo apt update
sudo apt install -y wireguard openssh-server tmux
sudo systemctl enable --now ssh
```

Activer le routage IP (utile seulement si des clients doivent router *à travers* Kali vers le lab) :
```bash
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-wg.conf
sudo sysctl --system
sudo sysctl net.ipv4.ip_forward     # doit afficher = 1
```

> Attention à l'orthographe exacte de la clé : `net.ipv4.ip_forward` (ipv**4**, des points,
> underscore uniquement dans `ip_forward`).

---

## Étape 2 — Générer les clés

Sur Kali :
```bash
mkdir -p ~/wg-keys && cd ~/wg-keys
umask 077

# Clés du serveur (Kali)
wg genkey | tee server_private.key | wg pubkey > server_public.key
```

Pour un client, deux cas :
- **Le coéquipier génère sa paire chez lui** (recommandé) et t'envoie seulement sa clé **publique**.
- Ou tu génères la paire pour lui :
  ```bash
  wg genkey | tee client1_private.key | wg pubkey > client1_public.key
  ```

Afficher une clé :
```bash
cat ~/wg-keys/server_public.key
```

Si une clé privée existe déjà et qu'il manque la publique, on la redérive (sans régénérer) :
```bash
wg pubkey < client1_private.key > client1_public.key
```

> `wg pubkey` lit toujours la clé sur l'entrée standard (`|` ou `<`), jamais en argument.
> **Règle d'or :** une clé privée ne quitte jamais la machine de son propriétaire ; on n'échange
> que des clés **publiques**.

---

## Étape 3 — Config serveur WireGuard (sur Kali)

Créer `/etc/wireguard/wg0.conf` :
```bash
sudo nano /etc/wireguard/wg0.conf
```
```ini
[Interface]
Address = 10.10.10.1/24
ListenPort = 51820
PrivateKey = <contenu de server_private.key>

# Coéquipier 1
[Peer]
PublicKey = <clé publique du client 1>
AllowedIPs = 10.10.10.2/32

# Coéquipier 2
[Peer]
PublicKey = <clé publique du client 2>
AllowedIPs = 10.10.10.3/32
```

Démarrer et activer au boot :
```bash
sudo systemctl enable --now wg-quick@wg0
sudo wg show          # doit lister l'interface et les peers
```

Ajouter un coéquipier plus tard : ajouter un bloc `[Peer]`, puis
`sudo systemctl restart wg-quick@wg0`.

---

## Étape 4 — Config client (une par personne)

Emplacement du fichier côté client :
- **wg-quick** : `/etc/wireguard/wg0.conf` (le nom sans extension = nom de l'interface),
  `sudo chmod 600` dessus (il contient la clé privée).
- **NetworkManager** (pratique en desktop) : `nmcli connection import type wireguard file wg0.conf`,
  puis activation via l'applet réseau.

Contenu :
```ini
[Interface]
Address = 10.10.10.2/24
PrivateKey = <clé privée DE CE client>

[Peer]
PublicKey = <clé publique du serveur Kali>
Endpoint = kali-viro-project.freeddns.org:42420
AllowedIPs = 10.10.10.0/24
PersistentKeepalive = 25
```

- `AllowedIPs = 10.10.10.0/24` → le client atteint les machines du VPN (donc Kali en `10.10.10.1`
  pour le SSH). C'est le modèle « Kali partagée » : on travaille *depuis* Kali.
- Ajouter `172.28.128.0/24` seulement pour router le lab directement via le tunnel (nécessite
  `ip_forward=1` côté Kali).

Monter / couper le tunnel :
```bash
sudo wg-quick up wg0
sudo wg-quick down wg0
```

---

## Étape 5 — Redirection VirtualBox (hôte → Kali)

Kali étant en NAT, rediriger UDP 51820 de l'hôte vers la VM.

**Interface :** VirtualBox → Kali → Configuration → Réseau → **Carte 1 (NAT)** → Avancé →
Redirection de ports → ajouter :
- Nom : `WireGuard` · Protocole : **UDP** · Port hôte : `51820` · Port invité : `51820`

**Ou en ligne de commande** (VM éteinte ; VBoxManage via son chemin complet si pas dans le PATH) :
```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" modifyvm "<nom-vm-kali>" --natpf1 "WireGuard,udp,,51820,,51820"
```

Vérifier que l'hôte relaie bien le port (Kali démarrée) :
```powershell
netstat -an -p UDP | findstr 51820      # doit montrer 0.0.0.0:51820
```

> La redirection NAT n'est active que si la VM Kali **tourne**.

---

## Étape 6 — Redirection Freebox (Internet → hôte)

Freebox OS → Paramètres → Mode avancé → **Gestion des ports / Redirections** → ajouter :
- **IP Destination** : `192.168.1.111` (IP LAN du PC hôte)
- **Redirection active** : cochée
- **IP source** : Toutes
- **Protocole** : **UDP**
- **Port de début / fin** : `42420`
- **Port de destination** : `51820`

> Le port exposé est `42420` (la Freebox refuse les ports ≥ 49152, donc pas 51820 côté externe).
> **Fige l'IP du PC hôte** en bail DHCP statique (Freebox OS → DHCP → baux statiques) pour que la
> redirection ne casse pas au reboot. **N'ouvre que ce port.**

**Coupe-circuit :** décocher « Redirection active » ferme l'accès VPN en un clic quand le lab ne
sert pas, sans supprimer la règle.

---

## Étape 7 — DNS dynamique (Dynu)

Nom d'hôte : `kali-viro-project.freeddns.org`.

La Freebox ne sachant pas piloter Dynu, la mise à jour de l'IP se fait par le **client Dynu IP
Update** installé sur le **PC hôte Windows** (allumé en permanence pour le lab). Il pousse l'IP
publique vers Dynu à chaque changement.

Vérifier la résolution :
```bash
nslookup kali-viro-project.freeddns.org      # doit renvoyer l'IPv4 publique de la Freebox
```

> **Test anti-CGNAT :** compare l'IPv4 WAN affichée dans Freebox OS avec l'IP publique réelle.
> Si elles diffèrent (WAN en `100.64.x.x`), la redirection ne marche pas → demander une IP
> full-stack à Free. (Free est en IP publique native par défaut.)

---

## Étape 8 — SSH + tmux partagé

Les coéquipiers se connectent à Kali par son IP VPN :
```bash
ssh kali@10.10.10.1
```

Session tmux partagée (tout le monde voit le même écran) :
```bash
tmux new -s lab       # premier connecté
tmux attach -t lab    # les suivants
```

> Pour du multi-utilisateur propre, créer un compte par personne (`sudo adduser alice`) plutôt que
> tout le monde sous `kali`.

---

## Étape 9 — Tests & vérifications

**Serveur (Kali) :**
```bash
sudo wg show                       # peers + dernier handshake
sudo ss -ulnp | grep 51820         # WireGuard écoute en UDP
```

**Client (après `wg-quick up`) :**
```bash
sudo wg show                       # chercher "latest handshake" + trafic reçu
ping 10.10.10.1
ssh kali@10.10.10.1
```

**Test de bout en bout :** depuis l'extérieur (collègue en 4G, ou téléphone en partage de
connexion **données mobiles**, Wi-Fi coupé), monter le tunnel et faire `ssh kali@10.10.10.1`.

**Débogage « rien ne passe » — où se perd le paquet :**
```bash
# sur Kali, pendant un test client :
sudo tcpdump -ni any udp port 51820
```
- Paquets visibles → chaîne réseau OK, regarder la config WireGuard (clés, AllowedIPs).
- Rien → le paquet se perd avant Kali : vérifier dans l'ordre le **protocole et l'IP de la
  redirection Freebox**, le **pare-feu de l'hôte Windows** (autoriser UDP 51820 entrant), la
  **redirection VirtualBox**, et que le test n'est pas un hairpin (sortir puis revenir par la
  box échoue — tester depuis l'extérieur).

Règle pare-feu hôte (PowerShell **administrateur**) :
```powershell
New-NetFirewallRule -DisplayName "WireGuard VPN" -Direction Inbound -Protocol UDP -LocalPort 51820 -Action Allow
```

---

## Étape 10 — Durcissement / sécurité

- **Changer le mot de passe Kali par défaut** (`kali`/`kali`) : `passwd` (ou `sudo passwd kali`).
- SSH : désactiver le login root, passer à l'authentification **par clé** une fois les clés en
  place (`/etc/ssh/sshd_config` → `PasswordAuthentication no`).
- **N'exposer que le port WireGuard** sur la Freebox. Jamais SSH, RDP ni les services du lab
  directement sur Internet.
- Garder le lab (cibles) sur son réseau **host-only**, sans routage vers le LAN domestique.
- Révoquer un accès = retirer le bloc `[Peer]` + `sudo systemctl restart wg-quick@wg0`.
- Couper le VPN hors sessions : décocher la redirection Freebox, ou
  `sudo systemctl stop wg-quick@wg0`.

---

## Récap des ports

| Où | Port | Protocole | Rôle |
|---|---|---|---|
| Freebox (externe) | 42420 | UDP | Seule ouverture Internet |
| VirtualBox NAT (hôte) | 51820 | UDP | Hôte → Kali |
| WireGuard (Kali) | 51820 | UDP | Serveur VPN |
| SSH (Kali, via VPN) | 22 | TCP | Accès équipe, **VPN uniquement** |

---

## Fiche à transmettre à chaque coéquipier

Ce dont la personne a besoin de ta part :
- La **clé publique du serveur** Kali.
- L'**endpoint** : `kali-viro-project.freeddns.org:42420`.
- L'**IP VPN** qui lui est attribuée (`10.10.10.3`, `.4`, …).

Ce qu'elle fait de son côté : générer sa paire, t'envoyer sa clé **publique**, remplir sa config
client (étape 4) avec sa clé privée (qui ne bouge pas de chez elle).
