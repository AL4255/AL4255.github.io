---
title: "SIEM Attack Lab"
date: 2025-07-02
status: "in progress"
stack: [Docker Compose, Elasticsearch, Kibana, Filebeat, DVWA]
tags: [cloud-security, docker, detection]
---

A local lab for practising detection: attack a deliberately vulnerable web app, then find the evidence in a SIEM.

## How it works

- **DVWA** (Damn Vulnerable Web App) is the target: SQL injection, XSS, command injection.
- **Filebeat** ships logs into **Elasticsearch**.
- **Kibana** is where I hunt for the attacks and build dashboards.

All of it runs from one `docker-compose.yaml`.

## Security decisions

- **Lab only.** Elasticsearch security is turned off to keep setup simple, so this must never be exposed beyond localhost. That's a deliberate trade-off, and it's written down so it stays deliberate.

## What I'd do next

Write detection rules for each DVWA attack type, and feed in the honeypot's Cowrie logs for real-world traffic.
