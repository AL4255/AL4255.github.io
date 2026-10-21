---
title: "Azure SSH Honeypot"
date: 2026-10-16
repo: "https://github.com/AL4255/azure-honeypot"
---

A small Azure VM that sits on the open internet and records who attacks it: brute-force logins, the commands attackers type, the malware they try to download, and probes against every other TCP port.

The goal is to deliberately invite attacks without becoming a risk to myself or anyone else. Most of the work went into the second half of that sentence.

## The setup

```
                    ┌──────────────── Azure NSG ────────────────┐
 Internet ─────────►│ 22222 from my IP ......... allow          │
                    │ 22222 from anyone else ... deny           │
                    │ any other TCP port ....... allow (bait)   │
                    └─────────────────────┬─────────────────────┘
                                          ▼
                    vm-honeypot  (Ubuntu 24.04, Standard_B1s)
                      :22    ──Docker──► Cowrie :2222   fake SSH
                      :23    ──Docker──► Cowrie :2223   fake Telnet
                      :22222 ──────────► sshd           real SSH (me)
                      other  ──────────► nothing → logged as SCAN
```

It's built in three layers, each with one job:

1. **Terraform** creates the Azure resources: network, firewall, VM and a budget alert.
2. **cloud-init** configures the VM on first boot: it moves the real SSH server, turns on scan logging and installs Docker.
3. **Docker** runs [Cowrie](https://github.com/cowrie/cowrie), the fake SSH/Telnet server attackers actually talk to.

## Layer 1: Terraform

Everything lives in one resource group, `rg-honeypot`, so `terraform destroy` removes all of it with nothing left behind costing money.

| Resource | Purpose |
|---|---|
| VNet `10.50.0.0/16` + one subnet | Its own network, not peered with anything else |
| Network Security Group | The firewall (below) |
| Standard public IP | Standard SKU blocks all inbound traffic unless an NSG allows it |
| Linux VM, `Standard_B1s` | 1 vCPU / 1 GB is plenty for Cowrie. SSH keys only, no password login |
| Budget, $15/month | Emails at 80% and 100% of spend |

Secrets never touch the code. The subscription ID, my IP and alert email are passed in as `TF_VAR_*` environment variables at deploy time.

## Layer 2: The firewall, where rule order is everything

Azure NSG rules are checked from the **lowest priority number up**, and the first match wins.

```
Inbound
 100  Allow  TCP 22222  from <my IP>
 110  Deny   *   22222  from anyone
 200  Allow  TCP *      from Internet

Outbound
 100  Allow  DNS (53)
 110  Allow  TCP 80, 443
4000  Deny   everything else to Internet
```

Rule 200 opens *every* TCP port, because that's the bait. Without rule 110 sitting in front of it, that would include my real admin port. The explicit deny has to come **before** the broad allow.

Outbound is the more important direction. A honeypot that gets turned into an attack bot is a liability, so the VM can only make DNS lookups and web requests. It can't send SSH, SMTP, SMB or anything else bots use to spread.

## Layer 3: Inside the VM

### Moving the real SSH out of the way

Two programs can't listen on the same port, and the trap needs port 22. Before Cowrie starts, cloud-init writes:

```
# /etc/ssh/sshd_config.d/10-admin.conf
Port 22222
PasswordAuthentication no
```

and restarts SSH. Ubuntu 24.04 starts SSH via `ssh.socket`, so that's the unit that gets restarted.

### Cowrie in Docker

```bash
docker run -d --name cowrie --restart unless-stopped \
  -p 22:2222 -p 23:2223 \
  -e COWRIE_TELNET_ENABLED=yes \
  -v cowrie-var:/cowrie/cowrie-git/var \
  cowrie/cowrie:latest
```

Cowrie runs as a normal user inside the container, and on Linux only root can bind ports below 1024. So Cowrie listens on 2222/2223, and Docker's `-p 22:2222` forwards the host's port 22 to it. Logs are kept in a named volume, so they survive container restarts.

When an attacker "logs in", they get an **emulated** shell. Commands like `uname -a` or `cat /etc/passwd` return convincing fake output, and nothing actually runs on the VM. Every username, password, command and download URL is written to `cowrie.json`.

### Logging port scans

Anything that isn't 22, 23 or 22222 hits a closed port. One iptables rule logs each new connection attempt to the kernel log:

```
iptables -I INPUT 1 -p tcp --syn ! --dport 22222 \
  -m limit --limit 20/second -j LOG --log-prefix "SCAN: "
```

The rule also matches 22 and 23, but that traffic never reaches it: Docker rewrites it to the container *before* the `INPUT` chain, where this rule sits. So the scan log only contains ports with nothing listening, which is exactly what I wanted.

## Containing the risk

There are several layers between an attacker and anything that matters:

1. **An emulated shell.** Nothing they type executes.
2. **A container.** Cowrie runs as a non-root user inside Docker.
3. **A VM with nothing on it.** It has no managed identity and no credentials, so there's nothing to steal and no Azure access to pivot with.
4. **An isolated network.** It has its own VNet with no peering.
5. **Outbound filtering.** Only DNS and web traffic can leave.

Ports 80 and 443 have to stay open for a reason: when an attacker runs `wget http://…/malware`, Cowrie really fetches the file so the sample is captured. Downloads from URLs on any other port fail at the firewall.

## Running it

```bash
export TF_VAR_subscription_id=$(az account show --query id -o tsv)
export TF_VAR_admin_ip=$(curl -s ifconfig.me)
export TF_VAR_alert_email="you@example.com"

terraform init
terraform plan -out tf.plan
terraform apply tf.plan
```

After a few minutes for cloud-init, I connect with:

```bash
ssh -i ~/.ssh/honeypot_ed25519 -p 22222 azureuser@<public-ip>
```

Leave off `-p 22222` and you log into your own honeypot. Two helper scripts pull data over that admin connection: `scripts/attacks.sh` for totals (top IPs, passwords, commands, downloads, scanned ports) and `scripts/live.sh` for a live feed.

## Caveats

- **Only TCP is logged.** The NSG bait rule is TCP-only, so UDP probes are dropped by Azure before they reach the VM.
- **Scan counts are a lower bound.** Logging is capped at 20 per second, so large bursts are undercounted.
- **The budget is only an alert.** Azure budgets send emails but don't stop resources, so `terraform destroy` is the real off switch.
- **The Cowrie image isn't pinned.** `:latest` means each deploy may pull a different version.
