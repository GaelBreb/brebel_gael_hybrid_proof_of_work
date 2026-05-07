Morning :
Investigating origin of attack discovered yesterday
Established that multiples ips from Vietnam accessed the compromized VMs from 19-01 to 08-02.
 Wrote a report to professor :
 Suite à la découverte (tardive) d'hier, nous avons enquêté un peu plus et avons trouvé des accès au GUI pfSense du site remote depuis plusieurs ips localisées principalement au Vietnam, qui coïncident avec la date présumée de dépôt du zip contenant les éléments exposés hier.
 
Dans les logs de connexion de cette VM qui eux remontent jusqu'à la création de la VM, nous avons relevés plusieurs de ces connexions :
 
Jan 19 20:23:05 pfSense php-fpm[433]: /index.php: Successful login for user 'admin' from: 113.22.137.193 (Local Database)
Jan 20 09:10:08 pfSense php-fpm[433]: /index.php: Successful login for user 'admin' from: 1.52.65.242 (Local Database)
Jan 20 13:41:11 pfSense php-fpm[432]: /index.php: Successful login for user 'admin' from: 1.52.65.242 (Local Database)
Jan 21 11:24:52 pfSense php-fpm[89494]: /index.php: webConfigurator authentication error for user 'admin' from: 1.52.65.242
Jan 21 11:24:52 pfSense sshguard[4898]: Attack from "1.52.65.242" on service unknown service with danger 10.
Jan 21 18:45:46 pfSense php-fpm[89494]: /index.php: Successful login for user 'admin' from: 1.52.65.242 (Local Database)
Jan 21 18:46:05 pfSense php-fpm[432]: /index.php: Successful login for user 'admin' from: 1.52.65.242 (Local Database)
Jan 22 23:13:02 pfSense php-fpm[432]: /index.php: Successful login for user 'admin' from: 1.52.65.242 (Local Database)
Jan 23 02:26:23 pfSense php-fpm[432]: /index.php: webConfigurator authentication error for user 'admin' from: 1.52.65.242
Jan 23 02:26:23 pfSense sshguard[4898]: Attack from "1.52.65.242" on service unknown service with danger 10.
Jan 24 19:45:27 pfSense php-fpm[89494]: /index.php: webConfigurator authentication error for user 'admin' from: 1.52.65.73
Jan 24 19:45:27 pfSense sshguard[4898]: Attack from "1.52.65.73" on service unknown service with danger 10.
Jan 26 12:00:20 pfSense php-fpm[92709]: /index.php: webConfigurator authentication error for user 'admin' from: 113.23.23.20
Jan 26 12:00:20 pfSense sshguard[4898]: Attack from "113.23.23.20" on service unknown service with danger 10.
Jan 27 03:09:14 pfSense php-fpm[92709]: /index.php: Successful login for user 'admin' from: 210.245.78.5 (Local Database)
Jan 28 04:29:33 pfSense php-fpm[92709]: /index.php: webConfigurator authentication error for user 'admin' from: 210.245.78.5
Jan 28 04:29:33 pfSense sshguard[4898]: Attack from "210.245.78.5" on service unknown service with danger 10.
Jan 29 21:38:33 pfSense php-fpm[89662]: /index.php: webConfigurator authentication error for user 'admin' from: 210.245.78.5
Jan 29 21:38:33 pfSense sshguard[4898]: Attack from "210.245.78.5" on service unknown service with danger 10.
Jan 31 13:51:01 pfSense php-fpm[41848]: /index.php: webConfigurator authentication error for user 'admin' from: 42.118.2.248
Jan 31 13:51:01 pfSense sshguard[4898]: Attack from "42.118.2.248" on service unknown service with danger 10.
Jan 31 17:31:08 pfSense php-fpm[89662]: /index.php: Successful login for user 'admin' from: 42.118.2.248 (Local Database)
Jan 31 17:52:59 pfSense sshguard[16019]: Now monitoring attacks.
Jan 31 17:53:12 pfSense sshguard[16019]: Exiting on signal.
Jan 31 17:53:13 pfSense sshguard[16053]: Now monitoring attacks.
Jan 31 17:53:14 pfSense login[6143]: login on ttyv0 as root
Feb  5 07:25:56 pfSense php-fpm[75579]: /index.php: Successful login for user 'admin' from: 23.94.216.234 (Local Database)
Feb  6 10:15:51 pfSense php-fpm[75579]: /index.php: Successful login for user 'admin' from: 113.23.19.87 (Local Database)
Feb  6 13:44:50 pfSense php-fpm[75579]: /index.php: Successful login for user 'admin' from: 113.23.19.87 (Local Database)
Feb  7 14:24:42 pfSense php-fpm[69567]: /index.php: Successful login for user 'admin' from: 223.181.104.135 (Local Database)
 
Les accès originaux ont été fait depuis 113.22.137.193 le 19 Janvier puis depuis 1.52.65.242 le 20 Janvier.  
Ensuite, de multiples connexions depuis des ips 210.245.x, 42.118.x, 1.52.x entre le 20 et le 31 Janvier.
 
Enfin, d'autres connexion entre le 5 et le 7 Février localisées ailleurs, une Inde 223.181.104.135 et une en Hollande 23.94.216.234 et une toujours au Vietnam 113.23.19.87
 
Après cette date il ne semble pas y avoir d'autres accès à la machine depuis des adresse ips qui nous sont inconnues, nonobstant les scans systémiques de la part de bots à la recherche de ports ouverts ou de fichier .env.
 
Le 10 Mars avait été mis en place des règles strictes sur l'accés via internet, bloquant tout accès venant d'adresses inconnues ainsi que le trafic sortant depuis pfSense du site on prem.
 
Conclusion, l'accès a été fait via le GUI de pfSense en utilisant les identifiants par défauts que nous n'avions pas encore changés à ce moment là.




then monitored more things, checked some rules, desactivate some, harden others

Afternoon :
Continued to share authorized ssh key to necessary VMs, identified one that have no communication with the others (our vm website), investigated, tried some things without success. Gave up.

Then worked on ESP, added tickets for web interface for users in product backlog and in specifications document.

