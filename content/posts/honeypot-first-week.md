---
title: "I put an SSH honeypot on Azure. Here's who showed up"
date: 2026-10-19
draft: true
tags: [cloud-security, azure, detection]
---

<!--
  Fill the TODOs with real numbers from scripts/attacks.sh.
  Before publishing: redact your VM's public IP, your admin IP, and anything
  identifying in captured payloads. Attacker IPs are fine to show in aggregate.
-->

**The setup:** a tiny Azure VM running [Cowrie](https://github.com/cowrie/cowrie), a fake SSH server, on port 22. The real SSH port is hidden and allow-listed to my IP. ([How it's built →](/projects/honeypot/))

## How fast did they find it?

TODO: time from `terraform apply` to first login attempt.

## By the numbers

TODO: total login attempts, unique IPs, top countries.

## The passwords they tried

TODO: top 10 username/password pairs. What does this say about default credentials?

## What they did once "inside"

TODO: most common commands. Typical patterns:
- recon (`uname -a`, `cat /proc/cpuinfo`): are you worth mining on?
- persistence (adding SSH keys to `authorized_keys`)
- downloading a payload with `wget`/`curl`

## What I learned

TODO: e.g. why password SSH on port 22 is found within minutes, and why outbound filtering matters.
