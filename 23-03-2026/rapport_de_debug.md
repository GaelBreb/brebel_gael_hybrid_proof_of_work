Rapport de debug - Impossible de se connecter en SSH à pfsense_s1

Problème

`ssh: connect to host 5.196.45.12 port 22: Connection timed out`
Les collègues peuvent se connecter sans problème depuis leur ip.

Investigations

1. Vérification des règles Ansible — Mon IP publique est bien dans l'alias ADMIN_IPS dans `inventory/group_vars/pfsense/vars.yml`. Pas de règle bloquante. Écarté.

2. Blocage port 22 par le FAI (Free) — `nc -zv github.com 22` et `nc -zv github.com 443` : les deux réussissent. Le FAI ne bloque pas le port 22. Écarté.

3. Problème WSL2 — `ssh -p 22 admin@5.196.45.12 -v` depuis PowerShell Windows : même timeout. Écarté.

4. Traceroute — `traceroute 5.196.45.12` : les paquets atteignent le réseau OVH (91.121.x.x, 37.59.x.x) puis se perdent. Les paquets sont droppés au niveau du pare-feu pfSense.

5. Vérification de l'alias sur pfSense — Accès via console Proxmox, puis `pfctl -t ADMIN_IPS -T show` : mon IP n'est pas dans l'alias chargé.

Cause racine

L'IP était dans les fichiers Ansible mais le playbook firewall n'avait pas été  bien appliqué, ou n'avions pas vu une erreur, ou bien encore la configuration avait été écrasée entre temps. L'alias sur pfSense ne contenait pas mon IP.

Résolution

- Ajout temporaire : `pfctl -t ADMIN_IPS -T add <MON_IP_PUBLIQUE>`
- Déploiement permanent : `ansible-playbook playbooks/firewall.yml`
