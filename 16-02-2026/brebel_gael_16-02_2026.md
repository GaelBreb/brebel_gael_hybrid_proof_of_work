Did some tests to match netbox elements to infra schema (cf: [netbox-tests/](./netbox-tests/))

Still some doubts about some concepts as: 
- how to define correctly ip addresses for adequate entities (global, sites, vpn, vm)
- how to configure VPN to connect to sites instead of VM ? If it's even possible/authorized/good practices.
- How to specify what roles which ip range/vm will play.
- how to link all this to ansible/terraform to automate the deployment

Tested notebook LLM to do some researches on proxmox and notebox documentation

Casually discussed T-ESP tasks, attribute task to a member (documenting hardware requirements), gave a feedback to an other on WBS and a test on new OBS format.