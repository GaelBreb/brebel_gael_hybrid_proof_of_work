# SSH par clé + accès coéquipiers — Kali partagée

**Contexte :** la Kali partagée est joignable via le VPN WireGuard (`ssh user@10.10.10.1`).
On passe le SSH en **authentification par clé** et on crée **un compte par personne**.

**Règle d'or anti-verrouillage :** on installe et on **teste** l'accès par clé AVANT de désactiver
le mot de passe. Garder une session SSH ouverte pendant toute la manip, en filet de sécurité.

---

## Partie A — Ce que fait chaque coéquipier (sur SA machine)

Chacun génère sa **paire de clés SSH** (différente de la clé WireGuard) et n'envoie que la clé
**publique**.

```bash
ssh-keygen -t ed25519 -C "prenom-poste"
# Entrée pour l'emplacement par défaut (~/.ssh/id_ed25519)
# choisir une passphrase (recommandé)
```

Cela crée :
- `~/.ssh/id_ed25519`      → **clé privée, reste chez la personne, ne circule jamais**
- `~/.ssh/id_ed25519.pub`  → clé **publique**, c'est ça qu'elle t'envoie

Afficher la clé publique à transmettre :
```bash
cat ~/.ssh/id_ed25519.pub
```
(une ligne commençant par `ssh-ed25519 AAAA... prenom-poste`)

> La clé publique n'est pas un secret, elle peut passer par n'importe quel canal.
> La clé privée, elle, ne quitte jamais la machine de son propriétaire.

---

## Partie B — Ce que tu fais sur Kali (admin)

### 1. Créer un compte par personne

```bash
sudo adduser alice          # répéter pour chaque coéquipier
# (mot de passe demandé : sert de secours, on s'appuiera sur la clé)
```

Si la personne doit pouvoir utiliser `sudo` :
```bash
sudo usermod -aG sudo alice
```

### 2. Installer sa clé publique

Pour chaque personne, déposer sa clé publique dans son `~/.ssh/authorized_keys` :
```bash
sudo mkdir -p /home/alice/.ssh
echo 'ssh-ed25519 AAAA...la_cle_publique_d_alice... alice-poste' | sudo tee -a /home/alice/.ssh/authorized_keys
sudo chown -R alice:alice /home/alice/.ssh
sudo chmod 700 /home/alice/.ssh
sudo chmod 600 /home/alice/.ssh/authorized_keys
```

> Les permissions sont strictes et **obligatoires** : SSH refuse la clé si `.ssh` n'est pas en 700
> et `authorized_keys` en 600, ou si le propriétaire est mauvais.

### 3. TESTER l'accès par clé (avant de désactiver le mot de passe)

Depuis la machine du coéquipier, VPN monté :
```bash
ssh alice@10.10.10.1
```
La connexion doit s'ouvrir **sans demander de mot de passe de compte** (juste la passphrase de la
clé, côté client). Tant que ce test n'est pas concluant pour au moins un compte, **ne pas passer à
l'étape 4**.

### 4. Désactiver l'authentification par mot de passe (durcissement)

Une fois l'accès par clé confirmé, éditer la config SSH :
```bash
sudo nano /etc/ssh/sshd_config
```
Régler (décommenter / modifier) :
```
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```
Appliquer :
```bash
sudo systemctl restart ssh
```

> Garder une session déjà ouverte pendant ce redémarrage. Si quelque chose casse, on peut
> réactiver `PasswordAuthentication yes` depuis cette session.

### 5. Vérifier

```bash
# le service tourne
sudo systemctl status ssh
# le mot de passe est bien refusé (doit échouer / ne pas proposer de mot de passe)
ssh -o PubkeyAuthentication=no alice@10.10.10.1
```

---

## Partie C — Révoquer un accès

Retirer proprement une personne (SSH **et** VPN) :

**SSH :**
```bash
sudo rm /home/alice/.ssh/authorized_keys    # retire sa clé
# ou, pour fermer tout le compte :
sudo usermod -L alice                       # verrouille le compte
# ou suppression complète :
sudo deluser --remove-home alice
```

**VPN (WireGuard) :** retirer son bloc `[Peer]` de `/etc/wireguard/wg0.conf`, puis
```bash
sudo systemctl restart wg-quick@wg0
```

---

## Partie D — Fiche à transmettre à chaque coéquipier

> **Accès au lab — [prénom]**
>
> 1. Génère ta clé WireGuard et ta clé SSH (voir ci-dessous), envoie-moi les deux **clés publiques**.
> 2. Je te renvoie : ta config VPN + ton nom d'utilisateur SSH.
>
> **Clé WireGuard :**
> ```
> wg genkey | tee wg_private.key | wg pubkey > wg_public.key
> cat wg_public.key      # → me l'envoyer
> ```
>
> **Clé SSH :**
> ```
> ssh-keygen -t ed25519 -C "ton-prenom"
> cat ~/.ssh/id_ed25519.pub   # → me l'envoyer
> ```
>
> **Ta config VPN (fichier wg0.conf) :**
> ```ini
> [Interface]
> Address = 10.10.10.X/24          # X = numéro que je te donne (3, 4, ...)
> PrivateKey = <TA clé privée WireGuard>
>
> [Peer]
> PublicKey = <clé publique du serveur Kali>   # je te la donne
> Endpoint = kali-viro-project.freeddns.org:42420
> AllowedIPs = 10.10.10.0/24
> PersistentKeepalive = 25
> ```
>
> **Se connecter :**
> ```
> sudo wg-quick up wg0            # monter le VPN
> ssh ton-user@10.10.10.1         # se connecter à la Kali partagée
> tmux attach -t lab              # rejoindre la session partagée (ou: tmux new -s lab)
> ```

---

## Récap de l'ordre d'exécution

1. Chaque personne génère ses clés (WG + SSH), envoie les deux publiques.
2. Toi : `adduser`, installer la clé SSH, ajouter le `[Peer]` WireGuard, attribuer l'IP VPN.
3. **Tester** une connexion par clé réussie.
4. Désactiver `PasswordAuthentication` dans sshd, redémarrer ssh.
5. Vérifier, et garder une session ouverte en filet pendant l'étape 4.
