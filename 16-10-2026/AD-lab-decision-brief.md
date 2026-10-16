# Active Directory Pentest + Detection Lab — Decision Brief

**Purpose:** a shared reference so the team can make a collegial decision on how to build
our Active Directory lab for lateral-movement pentesting **and** blue-team detection.

**Host constraints (current):** 1 PC, **16 GB RAM**, Windows host, VirtualBox + Vagrant.

---

## 1. What we're actually building (and why it differs from Metasploitable)

Our current targets (Metasploitable3) are **service-vulnerable**: deliberately broken
software (old ProFTPD, vulnerable IIS, etc.). You exploit a *service* and you're in.

An **Active Directory lab is different**. The machines are normal, reasonably-patched
Windows boxes. The vulnerabilities live in the **domain configuration**:

- Kerberoastable service accounts (weak passwords on SPN accounts)
- AS-REP roasting (accounts with pre-auth disabled)
- Dangerous ACLs / delegations (GenericAll, WriteDACL, unconstrained delegation)
- Credential reuse + pass-the-hash / pass-the-ticket
- DCSync rights, GPP passwords, over-privileged users

**Why this is the right target:** this *is* what real-world internal pentests look like.
Almost every corporate breach escalates through AD misconfigurations, not through a single
exploitable service. Practicing here maps directly to real engagements — offense and defense.

**Minimum realistic topology:**
- **1 Domain Controller** — Windows Server (runs AD DS + DNS). *Must* be Windows Server;
  a client Windows (8/10/11) cannot be a domain controller.
- **1–2 domain-joined members** — where we land and pivot from (client Windows or Server).

---

## 2. How to create the vulnerabilities — four options

| Option | What it is | Setup effort | RAM | Learning value | Fits 16 GB? |
|---|---|---|---|---|---|
| **BadBlood / vulnerable-AD** | Build one clean DC, run a script that injects hundreds of random misconfigs | Low–medium | Lightest | High (you build the DC) | **Yes** ✅ |
| **GOAD-Light** | Pre-built 2-VM vulnerable domain (Vagrant + Ansible) | Low | ~8–10 GB | High (curated attack paths) | **Tight but yes** ⚠️ |
| **Full GOAD** | Full multi-DC vulnerable forest (5 VMs) | Low (but long) | 24 GB+ | Very high | **No** ❌ |
| **Build by hand** | Manually create each misconfig ourselves | High | Lightest | Highest (you set every vuln) | **Yes** ✅ |

**Shortlist recommendation:** **BadBlood on our own DC** or **GOAD-Light**.
- *BadBlood* = most control, lightest, we learn the DC setup, randomised vulns each run.
- *GOAD-Light* = less manual work, curated/documented attack paths, slightly heavier.
- *Full GOAD* is the dream setup but needs a 32 GB (ideally more) host — park it until/unless we upgrade hardware.

---

## 3. The detection side (blue team)

Goal: run an attack, watch it generate telemetry, and have an alert fire. The classic
**attack → log → detect** loop.

**Components:**
- **Sysmon** on each Windows box → rich process/network/registry logging
- A **SIEM** to collect and alert on those logs

**SIEM options vs. our RAM:**

| SIEM | RAM | Notes |
|---|---|---|
| **Wazuh (all-in-one)** | ~4 GB | Single VM, endpoint collection + alerting built in. **Best fit for us.** |
| Elastic Stack | 6–8 GB | More capable, heavier — we'd feel it on 16 GB |
| Splunk (free) | 2–4 GB | Workable, free tier has data-volume limits |
| Security Onion | 12–16 GB alone | **Too big for this host** |

**Recommendation:** **Wazuh all-in-one** — standard home-detection-lab choice, fits the budget.

---

## 4. The hard constraint: RAM

A full attack-and-detect AD lab wants more than 16 GB if everything runs at once:

| Machine | RAM |
|---|---|
| Domain Controller (Server) | 4 GB |
| 1 domain-joined member | 2–3 GB |
| Kali (attacker) | 3 GB |
| Wazuh (detection) | 4 GB |
| Host (Windows + overhead) | 5 GB |
| **Total if all on** | **~18–19 GB** ❌ over budget |

**This is manageable by not running everything at once:**
- **Attack run:** DC + 1 member + Kali ≈ 14 GB ✅
- **Detection study:** DC + 1 member + Wazuh (Kali off) ≈ 15 GB ✅
- Browser closed during labbing either way.

**Hardware note for the team:** 16 GB works if we're disciplined about what's powered on.
**32 GB** is the upgrade that lets the whole lab (incl. full GOAD later) run comfortably at
once. It's a cheap upgrade relative to the quality-of-life gain — worth discussing if this
becomes an ongoing team project.

---

## 5. Recommended starting point (for discussion)

> **1 Windows Server DC + BadBlood + 1 domain-joined member + Kali**,
> then add **Wazuh** once the attack paths work.

Rationale: lightest realistic AD attack-and-detect lab, hits every classic lateral technique
(kerberoasting, AS-REP roasting, pass-the-hash, DCSync), and the detection pipeline is simple
to stand up. We prove the loop end-to-end on one member, then scale to a second member for
richer lateral movement once we've seen it light up in the SIEM.

**Phased plan:**
1. Stand up the DC, promote to domain, join one member.
2. Run BadBlood to inject misconfigs.
3. Attack from Kali — confirm the classic paths work.
4. Add Sysmon + Wazuh — confirm the attacks generate alerts.
5. Add a second member for lateral movement; tune detections.
6. (Later / if we upgrade RAM) migrate to GOAD for a richer forest.

---

## 6. Open questions for the team

- **Member OS:** Windows 10/11 client (realistic workstation) vs. another Windows Server?
- **Vuln source:** BadBlood (control + light) vs. GOAD-Light (curated + documented)?
- **RAM:** stay on 16 GB with disciplined power-management, or budget a 32 GB upgrade?
- **Exposure (separate decision):** if we host this to be reachable remotely, we use a
  **VPN front door (WireGuard)** — never direct port-forwarding of vulnerable boxes.
  (Details in a separate note; flagged here so it's on the radar.)
- **Scope / safety:** keep the lab on an isolated host-only network, no routing to the home
  LAN, ideally a dedicated machine if it runs long-term.

---

*Prepared as a shared decision aid. Nothing here is committed — the point is to pick the
option the team is happy to own and operate.*
