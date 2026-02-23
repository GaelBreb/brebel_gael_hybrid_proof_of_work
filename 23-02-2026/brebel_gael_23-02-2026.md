Watched a webinar from NetboxLabs x RedHat: "Exploring the Red Hat Ansible Certified Collection for NetBox"
presenting multiple points :

● NetBox Labs
● NetBox as a Network Source of Truth (NSoT)
● Network Automation Reference Architecture
● Ansible Automation Platform (AAP)
● Ansible Certified Collection for NetBox
● Use Cases + Demos
● Q & A

the different use cases showed example on how to setup these tools for :

Use Case 1 - NetBox as a Dynamic Inventory Source for Ansible Automation Platform

Use Case 2 - Define Intended Network State in NetBox

Use Case 3 - Query and Return Elements from NetBox

It appears that a lot of plugin and modules already exists (at the date of the webinar 80+) to automate a lot of things.

What I get from this webinar is that we can use Ansible as a preliminary definition of our aimed environment/network/devices, use it to automatically setup our netbox data (and maybe even our netbox instance prior to that), and then, make ansible "listen" to netbox and trigger specific event depending on what changes occurs on netbox.

All this could take the shape of yaml files as intended in the subject (IaC).

We could also have our observability services (grafana/prometheus) trigger certain events as well.

Netbox -> we want this
Ansible -> I do this
Grafana -> we have this

if (netbox !== grafana) {
    Ansible
}

[webinar record](https://www.youtube.com/watch?v=9oG9WVpvRSQ)
[webinar slides](https://netboxlabs.com/wp-content/uploads/2024/08/Webinar-Exploring-the-Red-Hat-Ansible-Certified-Collection-for-NetBox.pdf?utm_campaign=Q2_Educational%20Webinars&utm_medium=email&_hsmi=320722548&utm_source=hs_email)

Browsing this forum :
[network automation architecture](https://netboxlabs.com/blog/network-automation-architecture/)

Watched playlist from netbox labs to learn netbox :
[NetBox Zero To Hero](https://www.youtube.com/playlist?list=PL7sEPiUbBLo_iTds-NV-9Tu05Gg2Aj8N7) 

This video on IPAM is particularily interesting for us :
[NetBox Zero To Hero - Video 4 - IP Addressing and VLANs ](https://www.youtube.com/watch?v=XCwO6RGpBpI&list=PL7sEPiUbBLo_iTds-NV-9Tu05Gg2Aj8N7&index=5)

as well as 
[NetBox Zero To Hero - Video 8 - What About Virtualization?](https://www.youtube.com/watch?v=D5iDdjZMUeo&list=PL7sEPiUbBLo_iTds-NV-9Tu05Gg2Aj8N7&index=9)


Next time, go to follows those modules :
[Network Automation - Zero to Hero](https://www.youtube.com/playlist?list=PL7sEPiUbBLo_xC3uUZsDKas_IlHJ9E2cp)