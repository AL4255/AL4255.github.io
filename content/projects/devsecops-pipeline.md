---
title: "DevSecOps Pipeline"
date: 2025-07-12
status: "in progress"
stack: [Jenkins, OWASP ZAP, Docker, Python, Postgres]
tags: [cloud-security, docker, programming]
---

A CI/CD pipeline that builds a deliberately vulnerable Python web app and runs **OWASP ZAP** against it automatically, so security testing is a pipeline stage instead of an afterthought.

## How it works

- **Jenkins** runs the pipeline.
- **The vulnerable app** is a small Python service built from its own Dockerfile.
- **ZAP** runs as a daemon and is driven through its API to scan the running app.
- Everything shares one Docker network.

## Security decisions

- **Jenkins gets the Docker socket** so it can build images. That's effectively root on the host, which is acceptable for a local lab and a big red flag anywhere else.

## What I'd do next

Fail the build on high-severity ZAP findings, and move credentials out of the compose file into secrets.
