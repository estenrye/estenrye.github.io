---
layout: post
title: Configuring Highly Available LDAP in an Airgapped Environment
date: 2022-09-10T12:14:00+00:00
categories:
  - homelab
  - ldap
  - airgap
tags:
  - cd-homelab
  - automate-someday
  - work-in-progress
---

# Open Questions

- Should I host my LDAP services on the Bare Metal Kubernetes controlplane nodes?
- Should I host LDAP services in my Kubernetes Cluster, rather than as a systemd service?
- If I host as a systemd service, which virtual ip routing solution should I use?
   - [keepalived](https://keepalived.readthedocs.io/en/latest/)?  Tool I am most familiar with.
   - [pacemaker](https://wiki.clusterlabs.org/wiki/Pacemaker)?  Based on Heartbeat.

# Reference Material

## Journals

- [Highly Available LDAP](https://www.linuxjournal.com/article/5505) by Jay D. Allen on November 30, 2002,
  [Linux Journal](https://www.linuxjournal.com)