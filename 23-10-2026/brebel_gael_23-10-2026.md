## Matin

- Diagnostic et remise en état du réseau du lab. 
- Vérification de l’adressage des VM : constat d'un delta entre la documentation existante et ce que j'avais setup : (192.168.56.x) alors que le réseau réel est 172.28.128.0/24. - Détection d’un défaut sur la VM Kali, dont l’interface host-only avait perdu son adresse IPv4 — le trafic vers la cible transitait à tort par le NAT, masquant l’adresse de l’attaquant. 
- Correction par attribution d’une IP statique (172.28.128.10) rendue permanente via NetworkManager, puis vérification de la connectivité directe attaquant → cible. 
- Début de la mise en place d’une seconde VM Windows (Windows 10) destinée à simuler un poste de travail pour le mouvement latéral, et installation depuis une ISO. 
- Décision d’architecture : promouvoir la cible Metasploitable3 (Windows Server 2008 R2) en contrôleur de domaine Active Directory, avec le poste Windows 10 comme machine membre faisant évoluer le lab d’un modèle workgroup vers un véritable environnement Active Directory.
- Relecture de PR pour l'ESP

## Après midi

- Déploiement de l’infrastructure Active Directory.
- Après sauvegarde par instantané, promotion de la VM Metasploitable3 en contrôleur de domaine (nouvelle forêt lab.local, niveau fonctionnel 2008 R2, serveur DNS intégré) via dcpromo.
- Vérification de la santé du DC : partages SYSVOL et NETLOGON actifs, domaine opérationnel, accès administrateur confirmé (Domain/Enterprise/Schema Admins) ; interprétation des avertissements dcdiag (bruit des services vulnérables résiduels de Metasploitable).
- Création de comptes de domaine : un utilisateur standard et un compte de service kerberoastable (SPN attaché).
- Finalisation du poste Windows 10 : adressage statique host-only (172.28.128.20), DNS pointé sur le DC, résolution d’un conflit DNS entre les deux cartes réseau, puis jonction réussie au domaine lab.local.
- Configuration de redirecteurs DNS sur le DC pour un environnement propre et isolé.
- Relecture de PR pour Kubequest
cf : [cheatsheets pour la mise en place du DC](./cheatsheet-virology-dc/)
- Ajout d'un utilisateur au vpn pour VM Kali