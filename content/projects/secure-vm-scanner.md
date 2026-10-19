---
title: "Secure VM + Compliance Scanner"
date: 2026-04-03
status: "done"
stack: [Azure, Terraform, Python, Azure SDK]
tags: [cloud-security, azure, terraform, python]
---

Terraform deploys a hardened Linux VM. A Python scanner then checks the *live* VM against a security baseline and scores it, all from one `deploy-and-scan.sh`.

## How it works

- **Terraform** builds the resource group, VNet, subnet, NSG and an Ubuntu VM with **password authentication disabled** (SSH keys only).
- **`scanner.py`** uses `DefaultAzureCredential` and the Azure Compute SDK to pull the VM's real configuration and check:
  - password authentication is off
  - SSH key authentication is configured
  - disk encryption
  - network security settings
- It prints a score and compliance percentage.

## Why scan what you just deployed?

Terraform tells you what you *asked for*. A scanner tells you what *exists*. Drift, manual portal changes and provider defaults all live in that gap.

## What I'd do next

Run the scanner on a schedule and fail a CI pipeline when the score drops.
