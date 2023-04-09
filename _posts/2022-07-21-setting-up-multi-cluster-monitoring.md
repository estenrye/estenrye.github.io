---
layout: post
title: Setting up Multicluster Monitoring
date: 2022-07-21 19:35:00 -0500
categories:
  - monitoring
tags:
  - cortex
---

# References

- [Cortex 101: Horizontally Scalable Long Term Storage for Prometheus - Chris Marchbanks, Splunk](https://www.youtube.com/watch?v=f8GmbH0U_kI)
- [PromCon EU 2019: Two Households, Both Alike in Dignity: Cortex and Thanos](https://www.youtube.com/watch?v=KmJnmd3K3Ws)
- [Intro: Cortex - Tom Wilkie, Grafana Labs & Bryan Boreham, Weaveworks](https://www.youtube.com/watch?v=_7Wnta-3-W0)
- [Deep Dive: Cortex - Tom Wilkie, Grafana Labs & Bryan Boreham, Weaveworks](https://www.youtube.com/watch?v=mYyFT4ChHio)
- [Monitoring Kubernetes with Prometheus – Tom Wilkie](https://www.youtube.com/watch?v=kG9p417sC3I)
- [Intro to Scaling Prometheus with Cortex - Tom Wilkie, Grafana Labs & Ken Haines, Microsoft](https://www.youtube.com/watch?v=oHGoZd-9jWA)
- [Cortex: Intro and Production Tips - Bryan Boreham, Grafana Labs & Alvin Lin, Amazon Web Services](https://www.youtube.com/watch?v=zNE_kGcUGuI)
- [Cortex: Multi-tenant Scalable Prometheus - Bryan Boreham, Weaveworks & Jacob Tlisi, Grafana Labs](https://www.youtube.com/watch?v=VPBKNfRRytg)
- [Platform9: Kubernetes Monitoring at Scale with Prometheus and Cortex](https://platform9.com/blog/kubernetes-monitoring-at-scale-with-prometheus-and-cortex/)
- [Simplifying Kubernetes Monitoring with Prometheus](https://www.youtube.com/watch?v=2IZ5O_Gml6o)

# Documentation

- [Cortex Docs: cortex-helm-chart](https://cortexproject.github.io/cortex-helm-chart/)
- [Cortex Docs: Architecture](https://cortexmetrics.io/docs/architecture/)

# Components

- S3 for storage



# Management Cluster

- ingress-nginx
- cert-manaager
- nfs-csi
- sealed-secrets
- external-dns
- argocd
- etcd
- minio-operator
- prometheus
- cortex