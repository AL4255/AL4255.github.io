---
title: "AutoCloud"
date: 2026-09-25
status: "in progress"
stack: [Python, Claude API, Terraform, Azure CLI]
tags: [programming, azure, terraform, python]
---

An AI-powered CLI that turns a plain-English request ("a Linux VM with a locked-down NSG in UK South") into Terraform, validates it, estimates the cost, and deploys it to Azure.

## How it works

1. **Generate:** the request goes to Claude, which writes Terraform for the Azure provider.
2. **Validate and self-heal:** `terraform validate` runs. If it fails, the error goes back to the model to fix, up to 3 attempts.
3. **Estimate:** you see an estimated monthly cost *before* anything is created.
4. **Deploy and manage:** list, inspect and destroy the projects it created.

## Security decisions

- **A human approves before `apply`.** The deploy prompt defaults to *No*. Generated infrastructure is untrusted code until someone has read the plan.
- **Secrets stay out of the project.** The API key and subscription ID come from environment variables or a config file in the home directory, never from generated files or the repo.

## What I'd do next

Run a policy scanner (Checkov / tfsec) on the generated Terraform as part of validation, so "valid" also means "not obviously insecure".
