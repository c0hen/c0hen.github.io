---
layout: default
title: Infrastructure
description: Reminders for designing infrastructure
tags: devops coding infrastructure design architecture system toolchain
---

* Table of contents
{:toc}

## Infrastructure as Code

### Principles

1. Single source of truth.
1. Separate configuration from code.
1. Test backup restoration.
1. Users need familiarity with the tools used.
1. Clarity can override other concerns, increases security (KISS).
1. Complexity vs benefits received.
1. Less is more, especially regarding code maintenance cost.
1. Risk = Impact * probability.

## Considerations designing a devops toolchain

Example:
- Ansible
- Debian
- Kubernetes
- KVM

### Human

1. User interface clear, simple (including role creation, access).
1. Humans make mistakes, as many automatic checks as practical, no error / message spam!
1. While designing: generalize, modularize, template, override defaults on deeper levels.
1. Small, clear changes.
1. Responsibility, acknowledgement for human checks in processes.

### Technical

1. Degree of flexibility required. 1 or more different virtual machine (VM) technologies to create templates and base clones for further modification by Ansible etc. If more than 1 tool for VM templates is needed, how to sync changes?
1. Network and access (SSH for Ansible), flexibility (hosting providers, future migration).
1. Infrastructure code hosting.
1. Backup, retention, testing, (automatic) restoration.
1. Logging, retention, log alerts (repeated messages, high severity etc).
1. Monitoring (templated), alerts, autorecovery.
1. Does a tool make a service level (SLA) impossible?
