# Présentation — VPN & NetBox

---

## VPN Site-to-Site

### Ce qu'on a mis en place

Deux sites reliés par un tunnel chiffré. S1 est le serveur, S2 est le client. Tout le trafic inter-sites passe par ce tunnel — aucune VM n'est exposée directement sur Internet.

- **`openvpn_common`** — importe la CA sur S1  
- **`openvpn_server`** — génère le certificat serveur, crée les règles de routage par client, démarre OpenVPN  
- **`openvpn_client`** — importe la CA, génère son certificat, se connecte

Ordre d'exécution : CA → serveur → client

### Difficultés rencontrées

- Définir les bonnes règles de pare-feu

### Évolutions envisagées

- Ajouter un site S3 \= juste un nouveau client dans l'inventory, zéro modification du serveur  
- Rotation automatique des certificats à expiration

---

## VPN Road Warrior (Client-to-Site)

### Ce qu'on a mis en place

Admins et users se connectent depuis n'importe où. Deux profils aux accès distincts, sur le même serveur S1, ports différents.

- 3 instances OpenVPN : UDP/1195 (users), TCP/443 (fallback si UDP bloqué), UDP/1196 (admins)  
- 1 certificat par utilisateur — révocable individuellement

### Qui accède à quoi

| Profil | Instance | Accès |
| :---- | :---- | :---- |
| User | UDP/1195 ou TCP/443 | Site web \+ DMZ uniquement |
| Admin | UDP/1196 | Site web \+ DMZ \+ réseau admin S1 \+ réseau admin S2 \+ services |

La séparation est structurelle : deux instances avec des routes différentes. Un user ne peut pas atteindre les réseaux admin — ils ne lui sont simplement pas routés.

- Ajout d'un utilisateur \= 1 playbook, 1 certificat généré et fichier `.ovpn` prêt à l'emploi  
- MFA : le mode utilisé supporte nativement un backend LDAP/Radius

### Difficultés rencontrées

- Génération du fichier `.ovpn` client : embarquer CA \+ certificat \+ clé TLS dans un fichier unique propre → résolu via template Jinja2

### Évolutions envisagées

- Ajouter un site S3 \= juste un nouveau client dans l'inventory, zéro modification du serveur  
- Rotation automatique des certificats à expiration

---

## Refacto NetBox

### Ce qu'on a mis en place

NetBox est notre source de vérité pour l'infrastructure. On y déclare les VMs, les IPs, les rôles. Ansible s'en sert comme inventory dynamique : plus besoin de maintenir des listes de serveurs à la main.

- **`netbox.yml`** — déploie NetBox sur sa VM  
- **`netbox_populate.yml`** — remplit NetBox avec les données de chaque site  
- **`inventory/netbox.yml`** — expose tout ça comme hosts Ansible avec groupes automatiques (`vpn_clients`, `deploy_elk_True`...)

