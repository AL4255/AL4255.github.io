---
title: "Least privilege in Azure: writing a custom RBAC role with Terraform"
date: 2026-10-19
draft: true
tags: [cloud-security, azure, terraform]
---

**The problem:** someone needs to restart VMs when they hang. The obvious fix is to give them *Virtual Machine Contributor*. That role can also create, delete and resize VMs, attach disks, and more. If their account is phished, the attacker gets all of that too.

Least privilege means granting **only the actions the job needs**, on **only the scope it needs**.

## Azure RBAC in one minute

A role assignment has three parts:

| Part | Question it answers | Example |
|---|---|---|
| **Security principal** | *Who?* | a user, group, or managed identity |
| **Role definition** | *What can they do?* | a list of allowed actions |
| **Scope** | *Where?* | subscription → resource group → resource |

Built-in roles are pre-made role definitions. When none fits, you write your own.

## The role

```hcl
resource "azurerm_role_definition" "vm_operator_restricted" {
  name        = "vm-operator-restricted"
  scope       = azurerm_resource_group.demo.id
  description = "Read and start/restart VMs only. No create, delete, or resize."

  permissions {
    actions = [
      "Microsoft.Compute/virtualMachines/read",
      "Microsoft.Compute/virtualMachines/start/action",
      "Microsoft.Compute/virtualMachines/restart/action",
      "Microsoft.Compute/virtualMachines/instanceView/read",
    ]
    not_actions = []
  }

  assignable_scopes = [azurerm_resource_group.demo.id]
}
```

Why each piece is there:

- **`instanceView/read`** lets the operator see whether a VM is running. Without it they're restarting blind.
- **`assignable_scopes`** limits where this role can ever be assigned, not just where it is right now.
- **No wildcards.** `Microsoft.Compute/*` would be easier to write and would quietly grant `delete`.

## Assign it

```hcl
resource "azurerm_role_assignment" "operator" {
  scope              = azurerm_resource_group.demo.id
  role_definition_id = azurerm_role_definition.vm_operator_restricted.role_definition_resource_id
  principal_id       = var.operator_object_id
}
```

## Prove the negative

A permissions test isn't finished until you've seen it **deny** something. Sign in as the operator and try:

```bash
az vm restart -g <rg> -n <vm>   # should work
az vm delete  -g <rg> -n <vm>   # should fail with AuthorizationFailed
```

<!-- TODO: paste the real AuthorizationFailed output here -->

## Takeaways

- Start from the job, not from a built-in role.
- Scope as tightly as possible, both where the role is assigned and where it *can* be assigned.
- Always test what should be denied, not just what should be allowed.
