# Cheatsheet 01 — Promotion DC & DNS (Windows Server 2008 R2)

Référence condensée pour promouvoir un Windows Server 2008 R2 en contrôleur de domaine et
configurer son DNS. Domaine de référence : `lab.local` (NetBIOS `LAB`).

---

## Prérequis

- **Instantané VirtualBox avant promotion** (opération difficilement réversible).
- IP **statique** sur l'interface du lab (ex. `172.28.128.4`), passerelle vide.
- DNS préféré de la machine pointé **sur elle-même** (`127.0.0.1`).
- Accès **console VirtualBox** : `vagrant winrm` cesse de fonctionner après promotion
  (les comptes locaux migrent vers le domaine).

---

## Promotion

| Étape | Commande / action |
|---|---|
| Lancer l'assistant | `dcpromo` |
| Type | Créer un domaine dans une **nouvelle forêt** |
| FQDN | `lab.local` |
| NetBIOS | `LAB` |
| Niveau fonctionnel forêt | Windows Server 2008 R2 (le domaine suit automatiquement) |
| Options | cocher **Serveur DNS** |
| Délégation DNS | ignorer l'avertissement (normal en forêt isolée) → Oui |
| Chemins NTDS/SYSVOL | par défaut |
| Mot de passe DSRM | fort, **à noter** (mode restauration) |
| Fin | redémarrage automatique |

Version non interactive :
```cmd
dcpromo /unattend:unattend.txt
```
```ini
[DCInstall]
InstallDNS=Yes
NewDomain=Forest
NewDomainDNSName=lab.local
DomainNetbiosName=LAB
ReplicaOrNewDomain=Domain
ForestLevel=4
DomainLevel=4
DatabasePath="C:\Windows\NTDS"
LogPath="C:\Windows\NTDS"
SYSVOLPath="C:\Windows\SYSVOL"
SafeModeAdminPassword=MotDePasseDSRM!
RebootOnCompletion=Yes
```
(`ForestLevel=4` / `DomainLevel=4` = niveau 2008 R2.)

---

## Après redémarrage — connexion

- Les comptes locaux sont devenus comptes de domaine : `LAB\Administrator`, `LAB\Vagrant`.
- `LAB\Administrator` = admin du domaine (Domain/Enterprise/Schema Admins).

---

## Redirecteurs DNS (forwarders)

Permettent aux membres de résoudre les noms Internet via le DC.

Interface graphique : `dnsmgmt.msc` → clic droit serveur → **Propriétés** → onglet
**Redirecteurs** → **Modifier** → ajouter `8.8.8.8`, `1.1.1.1`.

Ligne de commande :
```cmd
dnscmd /resetforwarders 8.8.8.8 1.1.1.1
dnscmd /info /forwarders          REM vérifier
```
Test : `nslookup google.com` depuis le DC doit résoudre.

---

## Pièges fréquents

| Symptôme | Cause / correction |
|---|---|
| Avertissement « IP dynamique » | la carte NAT est en DHCP (normal) ; vérifier que la carte du lab est statique |
| Avertissement « délégation DNS impossible » | normal en forêt isolée → continuer |
| `vagrant winrm` ne répond plus | attendu après promotion → passer par la console / RDP |
| Mot de passe DSRM refusé | complexité requise (Maj+min+chiffre+symbole, 8+) |
