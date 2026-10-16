# Procédure de connexion — VPN WireGuard + accès Kali

Accès au lab via tunnel WireGuard aboutissant sur une machine Kali partagée (SSH).

Prérequis : `wireguard`, `openssh-client` installés sur le poste client.

---

## 1. Génération des clés (sur le poste client)

Deux paires distinctes sont requises : une clé WireGuard (tunnel) et une clé SSH (session).

WireGuard :
```bash
mkdir -p ~/wg-keys && cd ~/wg-keys
umask 077
wg genkey | tee wg_private.key | wg pubkey > wg_public.key
cat wg_public.key
```

SSH :
```bash
ssh-keygen -t ed25519 -C "identifiant"
cat ~/.ssh/id_ed25519.pub
```

Transmettre à l'administrateur les deux clés **publiques** (`wg_public.key`, `id_ed25519.pub`).
Les clés privées ne sont pas transmises.

---

## 2. Éléments fournis par l'administrateur

- IP VPN attribuée (format `10.10.10.X`)
- Nom d'utilisateur SSH

---

## 3. Configuration du client WireGuard

Créer `/etc/wireguard/wg0.conf` :

```ini
[Interface]
PrivateKey = <clé privée WireGuard du poste>
Address = 10.10.10.X/24

[Peer]
PublicKey = B0/79qESFz8ZsQxU194ER7hT4aMET9lgkwp0oSDVGAw=
Endpoint = kali-viro-project.freeddns.org:42420
AllowedIPs = 10.10.10.0/24
PersistentKeepalive = 25
```

Restreindre les permissions :
```bash
sudo chmod 600 /etc/wireguard/wg0.conf
```

Alternative NetworkManager :
```bash
nmcli connection import type wireguard file wg0.conf
```

---

## 4. Connexion

```bash
sudo wg-quick up wg0          # monter le tunnel
sudo wg show                  # vérifier : présence d'un "latest handshake"
ssh <utilisateur>@10.10.10.1  # session sur Kali
tmux attach -t lab            # session partagée (créer : tmux new -s lab)
```

Fermeture du tunnel :
```bash
sudo wg-quick down wg0
```

---

## 5. Diagnostic

| Symptôme | Vérification |
|---|---|
| Pas de `latest handshake` | `Endpoint`, clé privée WireGuard, IP VPN |
| Tunnel actif, SSH refusé | nom d'utilisateur, clé publique SSH transmise |
| Aucune connectivité | sortie de `sudo wg show` |

---

## 6. Contraintes

- Clés privées (WireGuard, SSH) non transmissibles.
- Lab constitué de machines volontairement vulnérables : accès exclusivement via ce VPN,
  aucun routage tiers.
