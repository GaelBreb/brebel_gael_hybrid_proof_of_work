Connexion à environnemnt proxmox
Installation en local de netbox
Lecture de documentation, compréhension des notions de regions, site, cluster, vm, leurs options et dépendances en partie.
Création de premiers sites, cluster groups, cluster, vm, interfaces pour se familiariser avec l'outil et ses fonctionnalités
Réflection sur l'automatisation de la mise en place de ces éléments.
Distingo entre Ansible/Terraform 
-> Les deux permettent l'orchestration/création d'infrastructure, Terraform étant plus pensé pour des environnement Cloud, Ansible plus polyvalent. 
    Pour la partie configuration management : 
    Terraform = programmation déclarative (on déclare l'état désiré)
    Ansible = programmation impérative (on indique comment atteindre l'état désiré (les processus et actions))
-> Pendant réunion il a été dit qu'à priori nous utiliserions Ansible  pour la création de l'infra (et mise à jour de NetBox en conséquence) qui lui même fait appel à Terraform pour la configuration des VM ensuite.

cf : [Mise_a_jour_doc_netbox.md](./Mise_a_jour_doc_netbox.md) et [CR réunion du 26-01-2026](./T-NSA-810_CR_Reunion_26-01-2026.pdf)