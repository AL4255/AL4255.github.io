---
title: "Azure SSH Honeypot"
date: 2026-10-16
status: "done"
stack: [Azure, Terraform, Cowrie, Docker, cloud-init, iptables]
tags: [cloud-security, azure, terraform, detection]
---

A small, locked-down Azure VM that sits on the open internet and records who attacks it: brute-force logins, the commands attackers run, the malware they try to download, and port scans on every other port.

## How it works

```
 Internet ──► Azure NSG ──► vm-honeypot (Ubuntu 24.04, B1s)
               │               ├─ port 22 / 23  → Cowrie (fake SSH/Telnet in Docker)
               │               ├─ other ports   → logged as scans (iptables LOG)
               │               └─ port 22222    → real SSH, my IP only
               └─ outbound: DNS + web only
```

**Cowrie** pretends to be a Linux server. Attackers "log in" to an emulated shell, so nothing they type runs on the real system, and every username, password and command gets recorded.

## Security decisions

- **Real SSH moved to 22222 and allow-listed to one IP.** Port 22 belongs to the trap. The real admin port is invisible to everyone else.
- **Outbound restricted to DNS and web.** If the box were ever genuinely compromised, it can't be used to attack other systems. This is the control that matters most: a honeypot that turns into a bot is a liability.
- **Its own resource group and VNet.** There's no peering and nothing to pivot into.
- **Budget alerts at 80% and 100%.** A cost guardrail is a security control too.

## What I'd do next

Ship the Cowrie logs into a SIEM and build dashboards: top source countries, credential pairs, and download URLs.
