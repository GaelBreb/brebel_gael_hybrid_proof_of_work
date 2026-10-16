# Vagrant Lab — Command Cheatsheet

Run all commands from the folder containing your `Vagrantfile`
(e.g. `D:\metasploitable3-workspace`).

VM names in this lab: `ub1404`, `win2k8`, `win2k8b` (and later `dc`, `member` for the AD lab).

---

## Daily driving

| Goal | Command |
|---|---|
| See what's running / stopped | `vagrant status` |
| Boot a specific VM | `vagrant up win2k8` |
| Boot everything in the Vagrantfile | `vagrant up` |
| Graceful shutdown (keeps the VM + data) | `vagrant halt win2k8` |
| Shut down everything | `vagrant halt` |
| Freeze to disk (fast resume, uses disk space) | `vagrant suspend win2k8` |
| Wake a suspended VM | `vagrant resume win2k8` |
| Reboot a VM | `vagrant reload win2k8` |
| Reboot **and** re-run provisioning | `vagrant reload win2k8 --provision` |

> **Free up RAM** → use `halt`. **Quickest pause/resume** → use `suspend`.

---

## Connecting

| Target | Command / method |
|---|---|
| Linux VM shell | `vagrant ssh ub1404` |
| Linux VM via SSH client | `ssh vagrant@172.28.128.3` (pw: `vagrant`) |
| Windows VM — desktop | VirtualBox Manager → select VM → **Show** |
| Windows VM — RDP | `vagrant rdp win2k8` |
| Windows VM — PowerShell (WinRM) | `vagrant powershell win2k8` |
| Windows VM — run one command | `vagrant winrm win2k8 -c "ipconfig"` |
| Windows VM — SSH | `ssh vagrant@192.168.56.3` |

**Default credentials (Metasploitable3):** `vagrant` / `vagrant`

**Find a Windows VM's IP:** `vagrant winrm win2k8 -c "ipconfig"`
→ use the **192.168.56.x** or **172.28.128.x** address (host-only network),
   NOT the 10.0.2.15 NAT address (that one is only for the VM's internet access).

---

## Provisioning

| Goal | Command |
|---|---|
| Re-run provisioning scripts on a built VM | `vagrant provision win2k8` |
| Boot a new VM and provision it | `vagrant up win2k8b` |

---

## Destroying / rebuilding

| Goal | Command |
|---|---|
| Delete a VM entirely (must rebuild after) | `vagrant destroy win2k8` |
| Delete without the y/n prompt | `vagrant destroy win2k8 -f` |
| Delete everything | `vagrant destroy -f` |

> `destroy` is irreversible — the VM and its data are gone. Use `halt` for everyday "turn it off."

---

## Boxes (downloaded base images)

| Goal | Command |
|---|---|
| List downloaded boxes | `vagrant box list` |
| Remove a box image | `vagrant box remove rapid7/metasploitable3-win2k8` |
| Update a box | `vagrant box update` |

---

## Troubleshooting

- **"non-zero exit status" on the cleanup step** → usually the final `rm C:\startup\*` line; the VM is still fine. Check with `vagrant status`.
- **VM won't boot / stuck** → `vagrant halt <name> -f` then `vagrant up <name>`.
- **Lost track of IPs** → `vagrant winrm <name> -c "ipconfig"` (Windows) or `vagrant ssh <name> -c "ip a"` (Linux).
- **Check VirtualBox directly** → open VirtualBox Manager to see real VM state if Vagrant and reality disagree.

---

## RAM-saving habits (16 GB host)

- Only power on the VMs you're actively using: `vagrant halt` the rest.
- Close heavy browsers while labbing (Firefox/Chrome can eat 2–3 GB = a whole VM).
- Don't bump the Metasploitable Windows boxes above 2 GB — they don't need it.
- Check live usage: **Task Manager → Performance → Memory**.
