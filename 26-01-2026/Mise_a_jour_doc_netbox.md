NetBox
Fusionne les fonctionnalités de gestion des adresses IP (IPAM) et de suivi des équipements physiques (DCIM)

Sert de “Source of truth”, de référentiel de l'état désiré de l’infrastructure pour des outils d’automatisation de création/configuration comme Ansible ou Terraform.
Permet d’observer l’état réel de l’infra et de la comparer à l’état souhaité, déclencher des call API en conséquence.
Permet de créer des “events rules” et automatiser les actions à mener selon le changement observé (https://netboxlabs.com/docs/netbox/features/event-rules/ )
Permet de gérer l’adressage dynamique des IP de tous les sites (https://netboxlabs.com/docs/netbox/features/ipam/)
Définir la hiérarchie des adresses (préfix, range, etc.. selon site par exemple)

Ça semble être l’outil qui va servir à “mapper” l’infra comme on souhaite qu’elle soit.
Quelles machines et où ?
Connectées à quoi ? Via quoi ? Interface reseau
Avec quelle ip ?

Et surveiller l’état de celle-ci en temps réel
On observe quoi ? Les différences entre état réel et souhaité ?
Qu’est-ce qu’on déclenche en conséquence de quel événement ?

Automatiser des actions vers d’autres outils (comme Ansible ou Terraform par exemple)

Doc API swagger dispo une fois l’instance netbox setup
Sinon, API graphQL, API REST, webhooks, Prometheus : https://netboxlabs.com/docs/netbox/integrations/rest-api/

Repo d’apprentissage netbox avec moults exemples, notamment pour NetBox Ansible Collection :
https://github.com/netboxlabs/netbox-learning

On pourrait avoir un workflow comme suit : 
Ecriture/modification d’un fichier .yaml référençant l’infra telle qu’on la souhaite (à la manière d’un docker compose)
Provisioning : On le donne à Terraform ou Ansible qui fait des calls API à NetBox pour mettre à jour les instances dans NetBox conformément au fichier .yaml quand il y a une différence et créer les VM, réseau, etc.. dans proxmox.
Configuration management : Ce qui déclenche ensuite des webhooks de netbox, faisant appel à Terraform (declarative) ou Ansible (imperative) pour configurer “l'intérieur” : installer les logiciels, gérer les utilisateurs et appliquer les politiques de sécurité, etc...
