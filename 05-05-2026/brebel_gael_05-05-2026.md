Morning :
Updated product backlog with gamification tickets for T-ESP

Reviewed technical and skills matrices

Looked after disk writing errors on pfsense S2 VM for T-NSA
And then investigate suspect scan and request on public ip of the VM -> classic bot scanning of public ip, no leak found

Afternoon :
Investigated why a colleague constantly get banned by pfsense whil running it's anisble playboooks -> it seems that it's playbook try ssh keys before vaulted password, resulting in increasing it's danger score on pfsense until it gets to more than 30 and get banned. -> temporarily put his ip address in sshguard whitelist to see if it resolve the problem. Need to change it's ansible configuration to start by using vaulted passwords

Got documented on client to site configuration in pfsense with openvpn. First in GUI interface to get acquainted with differents option and parameter, as well as steps expected (https://www.youtube.com/watch?v=QDaRpaIH4GE). Then with ansible modules.
Followed by this attached article : https://www.it-connect.fr/pfsense-configurer-un-vpn-ssl-client-to-site-avec-openvpn/

Reviewed POC for search engine documentation

Reviewed a PR for CI/CD pipeline on T-ESP project