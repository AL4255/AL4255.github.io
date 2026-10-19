---
title: "Least-Privilege Azure Role"
date: 2026-09-27
status: "in progress"
stack: [Azure RBAC, Terraform]
tags: [cloud-security, azure, terraform]
---

A custom Azure RBAC role, defined in Terraform, for a "VM operator" who can **read, start and restart** VMs, and nothing else. They can't create, delete or resize anything.

## Why

Built-in roles like *Virtual Machine Contributor* are much broader than most people need. Least privilege means writing down exactly the actions a job requires:

```hcl
actions = [
  "Microsoft.Compute/virtualMachines/read",
  "Microsoft.Compute/virtualMachines/start/action",
  "Microsoft.Compute/virtualMachines/restart/action",
  "Microsoft.Compute/virtualMachines/instanceView/read",
]
```

The role is scoped to a single resource group, so even those actions only apply there.

## What I'd do next

Assign it to a test identity and prove the negative: confirm that `delete` and `resize` are actually denied.
