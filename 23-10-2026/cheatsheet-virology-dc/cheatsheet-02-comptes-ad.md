# Cheatsheet 02 — Comptes, groupes & SPN Active Directory

Gestion des comptes de domaine depuis un DC (commandes `net`, `dsadd`, `setspn`).
À exécuter en **administrateur du domaine** (`LAB\Administrator`).

---

## Utilisateurs

| Action | Commande |
|---|---|
| Créer un utilisateur | `net user alice <PASSWORD> /add /domain` |
| Supprimer | `net user alice /del /domain` |
| Infos / appartenances | `net user alice /domain` |
| Réinitialiser le mot de passe | `net user alice NouveauMdp! /domain` |
| Mot de passe non expirant | via `dsmod user` (voir ci-dessous) |
| Lister tous les utilisateurs | `net user /domain` |

Via `dsadd` (plus complet) :
```cmd
dsadd user "CN=Bob,CN=Users,DC=lab,DC=local" -samid bob -pwd <PASSWORD> -disabled no
dsmod user "CN=Bob,CN=Users,DC=lab,DC=local" -pwdneverexpires yes
```

---

## Groupes

| Action | Commande |
|---|---|
| Ajouter à un groupe | `net group "Domain Admins" alice /add /domain` |
| Retirer d'un groupe | `net group "Domain Admins" alice /del /domain` |
| Lister les membres | `net group "Domain Admins" /domain` |
| Créer un groupe | `net group MonGroupe /add /domain` |

---

## Compte de service kerberoastable (cible d'attaque)

Un compte avec un **SPN** (Service Principal Name) devient attaquable par Kerberoasting.

```cmd
net user svc_sql Service2024! /add /domain
setspn -A MSSQLSvc/metasploitable3.lab.local:1433 svc_sql
```

| Action | Commande |
|---|---|
| Attacher un SPN | `setspn -A MSSQLSvc/host.lab.local:1433 svc_sql` |
| Lister les SPN d'un compte | `setspn -L svc_sql` |
| Rechercher un SPN en double | `setspn -Q MSSQLSvc/...` |
| Supprimer un SPN | `setspn -D MSSQLSvc/host.lab.local:1433 svc_sql` |

> Pour un mot de passe faible exprès (plus facile à casser en TP) : choisir un mot de passe
> court basé sur un mot de dictionnaire — à n'utiliser qu'en lab.

---

## Comptes vulnérables supplémentaires (optionnel, pour enrichir le TP)

**AS-REP roastable** (pré-authentification Kerberos désactivée) — via PowerShell AD :
```powershell
Set-ADAccountControl -Identity alice -DoesNotRequirePreAuth $true
```

**Mot de passe en clair dans la description** (mauvaise pratique classique) :
```cmd
dsmod user "CN=Bob,CN=Users,DC=lab,DC=local" -desc "mdp: Ete2024!"
```

---

## Unités d'organisation (OU)

```cmd
dsadd ou "OU=Workstations,DC=lab,DC=local"
dsadd ou "OU=Servers,DC=lab,DC=local"
```

---

## Politique de mot de passe (pour un lab plus permissif)

Par défaut : complexité exigée. Assouplir via `Default Domain Policy` (gpmc.msc →
Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies),
ou désactiver la complexité :
```cmd
REM via secpol / GPO ; à ne faire qu'en lab
```

---

## Outils graphiques

| Outil | Commande |
|---|---|
| Utilisateurs et ordinateurs AD | `dsa.msc` |
| Gestion des stratégies de groupe | `gpmc.msc` |
| Gestion DNS | `dnsmgmt.msc` |
