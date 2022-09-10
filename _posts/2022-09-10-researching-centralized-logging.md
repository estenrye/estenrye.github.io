---
layout: post
title: Configuring Centralized Logging in an Airgapped Environment
date: 2022-09-10T12:14:00+00:00
categories:
  - homelab
  - logging
  - airgap
tags:
  - cd-homelab
  - automate-someday
  - work-in-progress
---

# Open Questions

- What open-source log aggregation software should I use?
  - Elastic-Fluent-Kibana (EFK)?
  - Grafana Loki?

- What system logs are important to collect? Which ones are noise?

- Which log collection daemon do I want to use?
  - Fluentd?
  - Fluentbit?
  - Elastic Filebeat?
  - syslog?
  - journald?