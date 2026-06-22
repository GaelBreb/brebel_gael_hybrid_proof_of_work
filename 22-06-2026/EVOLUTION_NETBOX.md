# Évolution – Piloter l'infrastructure par NetBox

Ce document recense les changements nécessaires pour que l'ensemble des playbooks, rôles et fichiers d'inventaire soient pilotés par NetBox plutôt que par des valeurs hardcodées. L'objectif est qu'ajouter un site revienne uniquement à décrire ce site dans NetBox — sans toucher au code des rôles.

---

## Philosophie générale

Le principe directeur est le suivant : **aucune valeur spécifique à un site ne doit apparaître dans le code des rôles ou des playbooks**. Toute la connaissance d'un site (ses réseaux, ses IPs, ses services déployés, son rôle VPN) vit dans NetBox sous forme de config context. Les rôles lisent ces variables et s'adaptent.

Les variables de feature flags à centraliser dans les config contexts NetBox :

| Variable | Valeurs | Signification |
|----------|---------|---------------|
| `pfsense_role` | `onprem` / `remote` | Serveur VPN ou client |
| `site_has_dmz` | bool | Le site a une DMZ |
| `site_has_bastion` | bool | Le site a un bastion |
| `site_deploy_elk` | bool | ELK déployé sur ce site |
| `site_deploy_netbox` | bool | NetBox déployé sur ce site |
| `site_deploy_siteweb` | bool | Skillmatrix déployé sur ce site |
| `fw_admin_network` | CIDR | Réseau admin du site |
| `fw_users_network` | CIDR | Réseau users du site |
| `fw_dmz_network` | CIDR | Réseau DMZ du site |
| `fw_services_network` | CIDR | Réseau services du site |
| `fw_bastion_ip` | IP | IP du bastion |
| `fw_siteweb_ip` | IP | IP du siteweb |
| `fw_elk_ip` | IP | IP d'Elasticsearch |
| `fw_netbox_ip` | IP | IP de NetBox |

Ces variables sont déjà partiellement écrites dans NetBox par `netbox_populate.yml` — il s'agit principalement de s'assurer que le code les consomme plutôt que de les redéfinir en dur.

---

## 1. Playbooks site-spécifiques → playbooks génériques

### `pfsense_vlans.yml` + `pfsense_vlans_s2.yml`

**Problème** : deux playbooks séparés, l'un ciblant `pfsense_s1` avec des IPs S1 hardcodées, l'autre ciblant `pfsense_s2` avec des IPs S2 hardcodées.

**Cible** : un seul playbook `pfsense_vlans.yml` ciblant `hosts: pfsense` qui lit les IPs de chaque interface et les IDs VLAN depuis les variables du host (issues du config context NetBox). La définition des VLANs et des adresses IP pfSense doit être dans le config context de chaque site, puis consommée par le rôle.

**Bénéfice** : un nouveau site S3 déclenche automatiquement la création des bons VLANs sans créer de `pfsense_vlans_s3.yml`.

---

### `dns.yml`

**Problème** : `hosts: pfsense_s1` — DNS uniquement configuré sur S1.

**Cible** : `hosts: pfsense` avec `when: pfsense_role == 'onprem'` ou un feature flag `site_has_dns_server`. Si un futur site on-prem a son propre DNS, il sera couvert automatiquement.

---

## 2. Rôle `openvpn` — références hardcodées à pfsense_s2

### `group_vars/openvpn/vars.yml`

**Problème** : les variables `rw_dmz_network`, `rw_website_network`, `rw_s2_admin_network` référencent directement `hostvars['pfsense_s2']`. Si un site S3 s'ajoute comme client VPN, ces réseaux ne sont pas inclus dans les routes poussées par le serveur road warrior.

```yaml
# Actuel — hardcodé à S2
rw_dmz_network: "{{ hostvars['pfsense_s2']['vpn_client_dmz_network'] }}"
```

**Cible** : construire dynamiquement la liste `local_network` dans `openvpn_roadwarrior.yml` en bouclant sur `groups['vpn_clients']` et en agrégeant les réseaux de chaque client.

---

### `host_vars/pfsense_s2/vars.yml`

**Problème** : toutes les variables `vpn_client_*` (`vpn_client_local_network`, `vpn_client_iroute`, `vpn_client_users_network`, etc.) sont hardcodées dans ce fichier. Ces variables sont dupliquées par rapport à ce que NetBox sait déjà sur les réseaux du site.

**Cible** : ces variables doivent venir du config context NetBox du site S2. Le fichier `host_vars/pfsense_s2/vars.yml` ne devrait plus contenir que `ansible_password` (ref vault) et `pfsense_role`.

La migration nécessite de s'assurer que `netbox_populate.yml` écrit bien tous ces champs dans le config context, puis de supprimer les variables du fichier host_vars.

---

### `playbooks/register_rw_client.yml`

**Problème** : accède directement à `hostvars['pfsense_s1']` pour récupérer les variables VPN (WAN IP, ports, CA, clé TLS).

**Cible** : référencer le serveur VPN via le groupe `vpn_servers` (`groups['vpn_servers'][0]`) plutôt que par son nom de host. Fonctionnellement identique aujourd'hui, mais résistant à un renommage ou à une évolution de l'architecture.

---

## 3. Rôle `pfsense_firewall` — killswitches hardcodés

### `killswitch_elk.yml`, `killswitch_netbox.yml`

**Problème** : les règles référencent `hostvars['pfsense_s2']` pour récupérer les réseaux DMZ, users et admin. Elles utilisent `when: inventory_hostname == 'pfsense_s1'` pour conditionner les règles côté serveur.

**Cible** :
- Remplacer `hostvars['pfsense_s2']` par une boucle sur `groups['vpn_clients']`
- Remplacer `when: inventory_hostname == 'pfsense_s1'` par `when: pfsense_role == 'onprem'`

---

### `killswitch_siteweb.yml`, `killswitch_vpn.yml`

**Problème** : conditions `when: inventory_hostname == 'pfsense_s2'`.

**Cible** : `when: pfsense_role == 'remote'` ou `when: site_deploy_siteweb` selon le contexte de la règle.

---

### Ciblage d'un site spécifique à l'invocation

Supprimer les noms de hosts du code des rôles ne signifie pas perdre la capacité de cibler un site précis. Les deux niveaux sont indépendants :

- **Dans le code du rôle** : les conditions utilisent `pfsense_role`, `site_deploy_*` et les variables `fw_*` — aucun nom de host hardcodé.
- **À l'invocation** : l'opérateur cible le site voulu via `--limit` ou une variable.

```bash
# Killswitch ELK sur tous les sites
ansible-playbook playbooks/killswitch.yml --tags elk

# Killswitch ELK uniquement sur S2
ansible-playbook playbooks/killswitch.yml --tags elk --limit pfsense_s2

# Killswitch bastion sur S3 seulement
ansible-playbook playbooks/killswitch.yml --tags bastion --limit pfsense_s3
```

Cette séparation est plus propre que le hardcode actuel : le rôle décrit *comment* couper un service, l'opérateur décide *où* le couper.

---

## 4. Rôle `pfsense_dns` — IPs et réseaux hardcodés

**Problème** :
- `source: "10.1.3.0/24"` et `source: "10.1.1.0/24"` hardcodés — devraient utiliser `fw_services_network` et `fw_admin_network`
- `source: "{{ hostvars['pfsense_s2'].vpn_client_local_network }}"` — référence directe à S2 au lieu de boucler sur `groups['vpn_clients']`

**Cible** : le rôle DNS doit être refactorisé sur le même modèle que le rôle `pfsense_firewall` — lecture des réseaux depuis les variables `fw_*` et itération sur les groupes VPN.

---

### `group_vars/pfsense/vars.yml` — `dns_hosts`

**Problème** : l'entrée skillmatrix a `ip: "10.2.2.1"` hardcodée.

**Cible** : récupérer l'IP depuis le config context du site ou depuis les hostvars du groupe `siteweb` (`hostvars[groups['siteweb'][0]].ansible_host`).

---

## 5. Inventaire statique — `hosts.yml`

**Problème** : les hosts `elasticsearch`, `netbox`, `bastion_s2`, `siteweb` ont leurs IPs hardcodées dans `hosts.yml`. Pour un nouveau site, il faudrait ajouter des entrées manuellement.

**Cible** : ces hosts devraient être découverts via l'inventaire dynamique NetBox. Seul `pfsense_s1` reste en inventaire statique (hub VPN, genuinement unique). Les VMs Linux de tous les sites distants viennent de NetBox.

---

## 6. Divers

### `group_vars/filebeat/vars.yml`

**Problème** : `filebeat_elasticsearch_host: "http://10.1.3.1:9200"` duplique la valeur déjà dans `roles/filebeat/defaults/main.yml`.

**Cible** : supprimer la définition de `group_vars/filebeat/vars.yml` — le default du rôle suffit. Si l'IP doit devenir dynamique, la lire depuis `hostvars` du groupe `elk`.

---

### `group_vars/bastion/vars.yml` et `group_vars/siteweb/vars.yml`

**Problème** : `bastion_osh_cmd` contient l'IP WAN de S2 (`5.135.60.147`) en dur. `siteweb_public_ip` également.

**Cible** : lire l'IP WAN depuis les host_vars du pfSense du site correspondant (`s2_wan_ip`) ou depuis un champ NetBox.

---

## Ordre d'implémentation suggéré

| Priorité | Composant | Effort | Bénéfice |
|----------|-----------|--------|----------|
| 1 | `host_vars/pfsense_s2/vars.yml` → config context NetBox | Moyen | Débloque tout le reste |
| 2 | `group_vars/openvpn/vars.yml` → boucle vpn_clients | Faible | Routes RW correctes pour N sites |
| 3 | `pfsense_vlans.yml` + `pfsense_vlans_s2.yml` → fusion | Moyen | Un seul playbook VLANs |
| 4 | Rôle `pfsense_dns` | Moyen | DNS site-agnostique |
| 5 | Killswitches → `pfsense_role` + boucle | Faible | Killswitches fonctionnels sur S3+ |
| 6 | `hosts.yml` → inventaire dynamique NetBox | Élevé | Zéro déclaration manuelle de VM |
| 7 | Divers (filebeat, bastion, siteweb) | Faible | Nettoyage |

La priorité 1 est le prérequis de tout le reste : tant que les variables réseau de S2 sont dans `host_vars` et pas dans NetBox, les rôles ne peuvent pas boucler dessus proprement.
