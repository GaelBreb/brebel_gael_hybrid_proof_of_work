# Résumé de session

## Variable incorrecte dans les règles OpenVPN S2
Les règles SSH et HTTPS sur S2 référençaient `vpn_server_local_network`, une variable définie uniquement sur S1. Remplacé par `vpn_client_remote_network` qui pointe vers le même réseau (`10.1.1.0/24`) mais est disponible dans le contexte de S2.


---

## Mise à jour des IPs admin

Les IPs autorisées à accéder en SSH au pfSense ont été mise à jours et partagées à tout les membres de l'équipe pour s'assurer de ne bloquer personne en effectuant ses propres playboooks


Après toute modification, redéployer les règles :

```bash
ansible-playbook playbooks/firewall.yml
```

Si une IP n'est pas encore déployée et qu'on est bloqué, l'ajouter temporairement depuis la console pfSense (via Proxmox) :

```bash
pfctl -t ADMIN_IPS -T add <MON_IP>
```

---

## Debug problème collègue (Elasticsearch)

**Problème** : impossible de joindre la VM Elasticsearch via Ansible malgré une connexion SSH directe fonctionnelle.

**Cause** : l'inventaire pointait vers l'IP WAN du pfSense (`5.196.45.12`) au lieu de l'IP LAN de la VM (`10.1.3.1`). La VM est derrière le pare-feu pfSense et n'est pas accessible directement depuis l'extérieur.

**Solution** : utiliser pfSense S1 comme jump host dans `hosts.yml` :

```yaml
elasticsearch:
  ansible_host: 10.1.3.1
  ansible_ssh_common_args: "-o StrictHostKeyChecking=no -o ProxyCommand='sshpass -p {{ vault_pfsense_password }} ssh -W %h:%p -o StrictHostKeyChecking=no admin@5.196.45.12'"
```

`vault_pfsense_password` doit être défini dans le vault du projet :

 il l'est dans le groupe pfSense mais ne semble pas accessible actuellement.

```
[ERROR]: Task failed: 'vault_pfsense_password' is undefined
 
Task failed.
 
<<< caused by >>>
 
'vault_pfsense_password' is undefined
Origin: /Users/mathieuex/Documents/epitech/T-NSA-810-TLS_1/ansible/inventory/hosts.yml:17:36
 
15           ansible_become: true
16           ansible_python_interpreter: /usr/bin/python3
17           ansible_ssh_common_args: "-o StrictHostKeyChecking=no -o ProxyCommand='sshpass -p {{ vault_pfsense_passw...
                                      ^ column 36
 
fatal: [elasticsearch]: FAILED! => {"changed": false, "msg": "Task failed: 'vault_pfsense_password' is undefined"}
 
PLAY RECAP *********************************************************************
elasticsearch              : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
 
```

Donc création d'un group_vars/all :

```
mkdir -p inventory/group_vars/all
ansible-vault create inventory/group_vars/all/vault.yml

```

En y ajoutant le mdp de notre pfsense dedans puis en supprimant l'ancien dans group_vars/pfsense étant donné qu'il devait être accessible à tout le monde ?

la co au travers de pf sense fonctionne mais est refusée sur la VM cible


# Compte rendu - session après-midi

## Routage internet via tunnel VPN (S1 → S2)

**Contexte** : les VMs du site S1 ne doivent pas accéder directement à internet. Tout le trafic internet doit passer par le tunnel VPN vers S2, qui lui a accès à internet.

### Modifications Ansible effectuées

- Ajout des variables `vpn_s1_tunnel_ip` et `vpn_s2_tunnel_ip` (`172.16.53.1` / `172.16.53.2`) dans `group_vars/openvpn/vars.yml`
- Création de `roles/pfsense_firewall/tasks/lan_rules_s1.yml` : assignation de l'interface `ovpns1` dans pfSense, création d'une gateway `VPN_TUNNEL_GW` sur `opt2`, règles de routage HTTP/HTTPS/DNS depuis `10.1.3.0/24` via cette gateway
- Mise à jour de `roles/pfsense_firewall/tasks/openvpn_rules_s2.yml` : ajout de règles autorisant HTTP/HTTPS/DNS depuis `10.1.3.0/24` vers internet
- Création de `roles/pfsense_firewall/tasks/nat_s2.yml` : NAT outbound masquerading le trafic de `10.1.3.0/24` en sortie WAN de S2
- Mise à jour de `roles/pfsense_firewall/tasks/main.yml` pour inclure les nouveaux fichiers

### Problèmes rencontrés

- `ovpns1` non accepté comme interface de gateway par pfsensible → assignation préalable via `pfsense_interface` nécessaire
- pfSense auto-crée une gateway `VPN_S2_VPNV4` (dynamic, read-only) sur l'interface assignée → création d'une gateway statique séparée `VPN_TUNNEL_GW` sur `opt2`
- NAT S2 ne couvrait que `10.1.1.0/24` (`vpn_client_remote_network`) → corrigé en `10.1.3.0/24`

### État actuel

La gateway `VPN_TUNNEL_GW` apparaît dans pfSense. Les règles firewall et NAT sont appliquées. Cependant le trafic ne passe toujours pas — la VM ne peut pas pinguer sa propre gateway `10.1.3.254`.

Diagnostic : `ip neigh show` retourne `10.1.3.254 INCOMPLETE` — pfSense ne répond pas aux requêtes ARP sur l'interface `VLAN_SERVICES` (`vtnet2`).

**Cause probable** : problème de configuration réseau dans Proxmox — la VM Elasticsearch et l'interface `vtnet2` de pfSense S1 ne sont pas sur le même bridge ou le même VLAN tag.

**Prochaine étape** : vérifier la configuration des bridges et VLAN tags dans Proxmox pour les deux équipements.

---

## Fix inventaire Elasticsearch (collègue)

**Problème** : playbook ELK impossible à lancer, la VM `10.1.3.1` n'était pas joignable via Ansible.

**Cause** : inventaire pointait vers l'IP WAN du pfSense (`5.196.45.12`) au lieu de l'IP LAN de la VM. La VM est derrière le pare-feu et inaccessible directement depuis l'extérieur.

**Solution** : utiliser pfSense S1 comme jump host via ProxyCommand avec sshpass :

```yaml
ansible_ssh_common_args: "-o StrictHostKeyChecking=no -o ProxyCommand='sshpass -p {{ vault_pfsense_password }} ssh -o StrictHostKeyChecking=no admin@5.196.45.12 nc 10.1.3.1 22'"
ansible_password: "{{ timoty_ssh_pass }}"
```

`vault_pfsense_password` doit être dans `group_vars/all/vault.yml` (pas uniquement dans `group_vars/pfsense/`) pour être accessible au groupe `elk`.

**Résultat** : connexion établie, playbook ELK démarre. Installation d'Elasticsearch bloquée car la VM n'a pas accès à internet (lié au problème ci-dessus).
