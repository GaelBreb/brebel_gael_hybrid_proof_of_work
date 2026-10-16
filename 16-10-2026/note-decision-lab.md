# Lab Pentest + Détection — Note de décision

**Objectif :** document partagé pour que l'équipe décide ensemble de l'architecture du lab,
destiné au **mouvement latéral (pentest offensif)** et à la **détection (blue team)**.

**Contrainte matérielle actuelle :** 1 PC, **16 Go de RAM**, hôte Windows, VirtualBox + Vagrant.

---

## 1. Les deux approches possibles

Nous avons deux façons de construire ce lab. Elles ne couvrent pas les mêmes attaques.

### Option A — Lab Workgroup (sans domaine)
2–3 machines Windows **autonomes** (Windows 10/11 client suffit), dans un même workgroup,
partageant des **comptes locaux** avec des identifiants réutilisés. Le mouvement latéral se fait
de machine en machine via les identifiants/hash d'admin local. **Pas de serveur Windows,
pas de contrôleur de domaine.**

### Option B — Lab Active Directory (avec domaine)
Un **contrôleur de domaine** (Windows Server, obligatoire) + 1–2 machines membres jointes au
domaine. Les vulnérabilités viennent des **mauvaises configurations du domaine** (Kerberos, ACL,
délégations, comptes de service…). C'est ce que visent réellement les pentests internes en entreprise.

> **Point clé :** il n'y a pas d'Active Directory sans contrôleur de domaine, et un contrôleur de
> domaine ne peut tourner que sur **Windows Server** (un Windows client 8/10/11 ne peut pas être
> promu DC — limite Microsoft). En revanche le DC peut tourner en **Server Core** (sans interface
> graphique) sur ~2 Go, au lieu de ~4 Go en version complète.

---

## 2. Ce que chaque lab permet de pratiquer

| Technique | Workgroup | AD (domaine) |
|---|:---:|:---:|
| Pass-the-Hash (rejeu de hash admin local) | ✅ | ✅ |
| Réutilisation d'identifiants / password spraying | ✅ | ✅ |
| Relais NTLM (SMB relay) | ✅ | ✅ |
| Exécution à distance (PsExec, WMI, WinRM) | ✅ | ✅ |
| Élévation de privilèges locale | ✅ | ✅ |
| Vol d'identifiants (Mimikatz, LSASS, SAM) | ✅ | ✅ |
| Kerberoasting / AS-REP roasting | ❌ | ✅ |
| DCSync / DCShadow | ❌ | ✅ |
| Golden / Silver tickets (Kerberos) | ❌ | ✅ |
| Abus de délégation (unconstrained / RBCD) | ❌ | ✅ |
| Abus d'ACL de domaine (GenericAll, WriteDACL) | ❌ | ✅ |
| Cartographie BloodHound, GPP, LAPS, ADCS | ❌ | ✅ |

**À retenir :** le workgroup couvre toutes les attaques basées sur les **comptes locaux**
(fondamentaux et très utilisés dans la vraie vie). L'AD ajoute **tout le volet Kerberos / domaine**,
qui représente la **majeure partie** de ce qu'exploitent réellement les pentests internes modernes.

---

## 3. Logique pédagogique attaque / défense

**Côté attaque (red team) :** le workgroup enseigne la brique de base — réutilisation
d'identifiants, Pass-the-Hash, relais, exécution à distance. L'AD enseigne la couche supérieure —
compromettre un domaine entier via ses mauvaises configurations, jusqu'à l'admin de domaine.

**Côté défense (blue team) :** dans les deux cas on installe **Sysmon** sur les machines Windows
et un **SIEM** pour collecter et alerter. La boucle **attaque → log → alerte** est :
- plus **simple et lisible** en workgroup (peu de bruit, idéal pour débuter la détection),
- plus **riche et réaliste** en AD (télémétrie Kerberos, événements de domaine).

**SIEM recommandé (adapté à nos 16 Go) :** **Wazuh all-in-one** (~4 Go, une seule VM, collecte +
alerting intégrés). Elastic (6–8 Go) est plus lourd ; Security Onion (12–16 Go à lui seul) est
hors budget sur cet hôte.

---

## 4. Création des vulnérabilités (uniquement pour le lab AD)

En workgroup, « rendre vulnérable » = créer des comptes locaux à mot de passe réutilisé et
désactiver la signature SMB. Simple et manuel.

En AD, quatre façons d'injecter les mauvaises configurations :

| Méthode | Principe | Effort | RAM | Tient en 16 Go ? |
|---|---|---|---|:---:|
| **BadBlood / vulnerable-AD** | On monte un DC propre, un script injecte des centaines de misconfigs | Faible–moyen | Le plus léger | ✅ |
| **GOAD-Light** | Domaine vulnérable pré-construit, 2 VM (Vagrant + Ansible) | Faible | ~8–10 Go | ⚠️ juste |
| **GOAD complet** | Forêt multi-DC vulnérable (5 VM) | Faible (mais long) | 24 Go+ | ❌ |
| **À la main** | On crée chaque misconfig soi-même | Élevé | Le plus léger | ✅ |

**Présélection :** **BadBlood** (contrôle + léger, on apprend à monter le DC) ou **GOAD-Light**
(moins de travail manuel, chemins d'attaque documentés). GOAD complet = à réserver à un hôte 32 Go+.

---

## 5. Budget RAM — le facteur décisif

| Configuration | RAM estimée | Tient en 16 Go ? |
|---|---|:---:|
| **Workgroup :** 2 cibles + Kali + hôte | ~13 Go | ✅ confortable |
| **Workgroup + Wazuh** | ~17 Go | ⚠️ juste (passer à 1 cible ou jongler) |
| **AD (DC Server Core) :** DC + 1 membre + Kali + Wazuh + hôte | ~16 Go | ⚠️ ça passe |
| **AD (DC version complète GUI) :** idem | ~18–19 Go | ❌ dépasse |

**Conséquences :**
- Le **workgroup** respire plus largement sur 16 Go (pas de serveur, Windows client plus léger).
- L'**AD** tient en 16 Go **à condition** d'utiliser un **DC en Server Core (~2 Go)** et de ne pas
  tout allumer en même temps (ex. attaque : DC + membre + Kali ; détection : DC + membre + Wazuh).
- Une mise à niveau vers **32 Go** rend tout confortable (et ouvre la porte à GOAD complet).
  Upgrade peu coûteux au regard du confort — à débattre si le projet s'installe dans la durée.

---

## 6. Comparatif global

| Critère | **Workgroup** | **AD (DC Server Core)** |
|---|---|---|
| Serveur Windows nécessaire | Non | Oui (DC, ~2 Go en Core) |
| Effort de mise en place | Faible (~1 jour) | Moyen (promotion, DNS, vulns) |
| Confort RAM sur 16 Go | Large (~13 Go) | Juste mais OK (~16 Go) |
| Mouvement latéral (PtH, relais, PsExec) | ✅ | ✅ |
| Attaques Kerberos / domaine | ❌ | ✅ |
| DCSync, délégation, ACL, BloodHound | ❌ | ✅ |
| Couverture « monde réel » | Partielle (comptes locaux) | Complète (cible réelle des pentests) |
| Lab de détection | ✅ propre et simple | ✅ plus riche et réaliste |

---

## 7. Recommandation (à débattre)

Notre objectif initial était explicitement **« Active Directory et vulnérabilités de domaine »** —
donc l'AD est la **destination**. Mais le workgroup est le meilleur **premier lab** : rapide, léger,
il enseigne les fondamentaux (réutilisation d'identifiants, PtH, relais) sur lesquels tout le reste repose.

> **Chemin proposé — les deux, en deux temps :**
> 1. **Commencer en Workgroup** : prouver la boucle attaque → détection, maîtriser PtH / relais / PsExec
>    à moindre coût RAM et effort.
> 2. **Passer ensuite à l'AD (DC Server Core + BadBlood + 1 membre + Kali, puis Wazuh)** une fois
>    l'équipe à l'aise et la question RAM tranchée. Les machines du workgroup pourront être
>    rejointes au domaine ensuite.

Cela couvre à terme **toutes** les techniques de mouvement latéral (kerberoasting, AS-REP, PtH,
DCSync) avec une détection qui se déclenche dans le SIEM.

---

## 8. Questions ouvertes pour l'équipe

- **Point de départ :** Workgroup d'abord, ou directement l'AD ?
- **OS des machines membres (si AD) :** Windows 10/11 client (poste de travail réaliste) ou un
  second Windows Server ?
- **Source des vulns (si AD) :** BadBlood (contrôle + léger) ou GOAD-Light (documenté) ?
- **RAM :** rester sur 16 Go avec gestion disciplinée de l'allumage, ou budgéter 32 Go ?
- **Exposition Internet (décision séparée) :** si on héberge le lab pour un accès distant, on passe
  par un **tunnel VPN (WireGuard)** — jamais de redirection de port directe vers des machines
  volontairement vulnérables. (Note détaillée à part ; mentionné ici pour mémoire.)
- **Cloisonnement / sécurité :** lab sur réseau **host-only** isolé, aucun routage vers le LAN
  domestique, idéalement une machine dédiée si le lab tourne sur la durée.

---

*Document d'aide à la décision. Rien n'est figé — l'objectif est de choisir l'option que l'équipe
est prête à assumer et à exploiter.*
