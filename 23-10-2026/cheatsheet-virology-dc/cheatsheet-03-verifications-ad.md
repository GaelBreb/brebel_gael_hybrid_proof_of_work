# Cheatsheet 03 — Tests & vérifications de santé AD

Commandes pour vérifier qu'un contrôleur de domaine et le domaine fonctionnent correctement.

---

## Le DC est-il fonctionnel ?

| Vérification | Commande | Attendu |
|---|---|---|
| Domaine de la machine | `echo %USERDNSDOMAIN%` | `LAB.LOCAL` |
| Partages critiques | `net share` | **SYSVOL** et **NETLOGON** présents |
| Diagnostic complet | `dcdiag` | majorité de tests `passed` |
| Diagnostic (erreurs seules) | `dcdiag /q` | peu/pas de sortie |
| Localiser le DC | `nltest /dsgetdc:lab.local` | renvoie le DC |
| Rôles FSMO | `netdom query fsmo` | le DC détient les 5 rôles |

> Sur une machine Metasploitable3 promue en DC, `dcdiag` remonte du bruit (services SNMP,
> GlassFish, drivers échouant à charger) : ce sont les services vulnérables résiduels, **pas**
> des défauts du DC. Les tests qui comptent : SYSVOL partagé + réplication (ci-dessous).

---

## DNS

| Vérification | Commande | Attendu |
|---|---|---|
| Résolution du domaine | `nslookup lab.local` | répond via le DC |
| Enregistrements SRV du DC | `nslookup -type=SRV _ldap._tcp.lab.local` | pointe vers le DC |
| Résolution Internet (forwarder) | `nslookup google.com` | résout |
| Redirecteurs configurés | `dnscmd /info /forwarders` | liste `8.8.8.8` etc. |

---

## Réplication / SYSVOL (DC unique)

| Vérification | Commande |
|---|---|
| État réplication | `repadmin /replsummary` |
| SYSVOL partagé | `net share` (présence de SYSVOL) |

Sur un DC **unique**, pas de partenaire de réplication : les avertissements DFSR/SYSVOL
juste après promotion se stabilisent seuls.

---

## Objets du domaine

| Vérification | Commande |
|---|---|
| Lister les ordinateurs du domaine | `dsquery computer` |
| Lister les utilisateurs | `dsquery user` |
| Vérifier un compte précis | `net user alice /domain` |
| Vérifier un SPN | `setspn -L svc_sql` |

---

## Jonction d'un poste — vérifications (côté client)

| Vérification | Commande | Attendu |
|---|---|---|
| Domaine du poste | `systeminfo \| findstr /i Domain` | `lab.local` |
| Résolution du domaine | `nslookup lab.local` | via le DC (`172.28.128.4`) |
| Canal sécurisé avec le DC | `nltest /sc_query:lab.local` | `Success` |
| Connexion avec compte domaine | ouverture de session `LAB\alice` | réussie |

---

## Synchronisation d'heure (critique pour Kerberos)

Kerberos exige un écart **< 5 minutes** entre le DC et les membres.

| Action | Commande (sur le client) |
|---|---|
| Voir la source de temps | `w32tm /query /status` |
| Resynchroniser sur le DC | `w32tm /resync` |
| Comparer avec le DC | comparer `time /t` sur les deux machines |

> Un décalage d'horloge trop grand → échec de la jonction au domaine ou de l'authentification,
> même quand le DNS est correct. Second suspect après le DNS en cas d'échec de jonction.
