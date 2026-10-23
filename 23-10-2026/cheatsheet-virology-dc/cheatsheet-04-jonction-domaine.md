# Cheatsheet 04 — Jonction d'un poste Windows au domaine

Procédure pour joindre une workstation (Windows 10) à un domaine AD, avec les pièges réseau.
Poste de référence : Windows 10, host-only `172.28.128.20`, domaine `lab.local`, DC `172.28.128.4`.

---

## Prérequis réseau (dans l'ordre)

1. **IP statique** sur la carte host-only : `172.28.128.20` / `255.255.255.0`, passerelle vide.
2. **DNS préféré = le DC** : `172.28.128.4`. ← l'erreur n°1 des jonctions ratées.
3. **Heure** à moins de 5 min de celle du DC (Kerberos).

---

## Le piège « deux cartes réseau »

Un poste avec une carte **NAT** (passerelle) + une carte **host-only** (sans passerelle)
privilégie le DNS de la carte NAT → il n'interroge pas le DC et ne trouve pas `lab.local`.

**Vérifier quel DNS est réellement utilisé :**
```cmd
nslookup lab.local
```
Si la réponse vient d'un autre serveur que `172.28.128.4` (ex. `10.0.2.3`), le DNS de la NAT
prime.

**Solutions :**
```cmd
REM désactiver temporairement la carte NAT (le temps de la jonction)
netsh interface set interface "Ethernet" admin=disable

REM ... joindre le domaine ...

REM réactiver après (si Internet nécessaire sur le poste)
netsh interface set interface "Ethernet" admin=enable
```
Solution propre et permanente : configurer des **redirecteurs DNS sur le DC** (voir cheatsheet 01)
et n'utiliser que le DC comme DNS sur le poste → la carte NAT devient inutile pour le lab.

---

## Joindre le domaine

Interface graphique :
Ce PC → Propriétés → **Modifier les paramètres** → onglet Nom de l'ordinateur → **Modifier** →
cocher **Domaine** → `lab.local` → identifiants `LAB\Administrator` → redémarrer.

Ligne de commande (PowerShell admin) :
```powershell
Add-Computer -DomainName lab.local -Credential LAB\Administrator -Restart
```

---

## Vérifier la jonction

| Vérification | Commande | Attendu |
|---|---|---|
| Domaine du poste | `systeminfo \| findstr /i Domain` | `lab.local` |
| Canal sécurisé | `nltest /sc_query:lab.local` | `Success` |
| Connexion compte domaine | session `LAB\alice` | réussie |
| Côté DC : objet ordinateur | `dsquery computer` | le poste apparaît |

---

## Isolement vs accès Internet du poste

| Config | Résout lab.local | Résout Internet | Atteint Internet |
|---|---|---|---|
| host-only seul, DNS = DC | ✅ | ✅ (via forwarder DC) | ❌ (pas de passerelle) |
| host-only + NAT | ✅ | ✅ | ✅ |

Pour un lab, **host-only seul + DNS = DC** est recommandé : poste isolé d'Internet (plus
réaliste et plus sûr), résolution assurée par le DC. Réactiver la NAT ponctuellement si un
téléchargement est nécessaire.

---

## Dépannage jonction

| Symptôme | Cause probable |
|---|---|
| « Domaine introuvable » | DNS du poste ne pointe pas sur le DC (90 % des cas) |
| « Échec de l'authentification » | écart d'heure > 5 min, ou mauvais identifiants |
| Résout via le mauvais serveur | conflit deux cartes → désactiver la NAT |
| Caractères du mot de passe faux | disposition clavier (AZERTY/QWERTY) en saisie masquée |
