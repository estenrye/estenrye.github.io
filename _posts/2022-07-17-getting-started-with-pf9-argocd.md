---
layout: post
title: Getting Started with Platform9 ArgoCD
date: 2022-07-17 16:02:00 -0500
categories:
  - notes
  - argocd
tags:
  - continuous-delivery
  - argocd
---

# Getting Started with Platform9's Hosted ArgoCD

## Prerequisites

- [Platform9: ArgoCD Quickstart](https://platform9.com/docs/argo/argocd-quickstart)
- [Platform9: Login to ArgoCD](https://platform9.com/docs/argo/login-to-argocd)

## References

### Documentation

- [ArgoCD Docs: Operator Manual | Declarative Setup](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup)

### Videos

- [TechWorld with Nana: ArgoCD Tutorial for Beginners | GitOps CD for Kubernetes](https://youtu.be/MeU5_k9ssrs)
## Install the argocd CLI and connect it to Platform9 ArgoCD

```bash
brew install argocd

PF9_INSTANCE=my-instance.platform9.play
argocd login ${PF9_INSTANCE} --sso --grpc-web-root-path argocd
```

## Adding Repository Credentials

1. Create a dedicated SSH Key for ArgoCD

```bash
ssh-keygen -t ecdsa-sha2-nistp256 -f ~/.ssh/argocd_id_ecdsa
```

2. Add Key to GitHub as an authorized SSH Key.

3. Add Private SSH Repository to ArgoCD:

```bash
argocd repo add git@github.com:estenrye/cd-homelab.git --ssh-private-key-path ~/.ssh/argocd_id_ecdsa
```

## Adding a new project for managing ArgoCD configuration

ArgoCD can be managed declaritively, but we first need to tell it where to find 
its configuration.  To do that, we first create a project that allows us to
deploy to the `argocd` namespace in the `in-cluster`.  The main reason we are
separating this into its own project is so we can control who has permissions
to deploy apps to the cluster.

```bash
# Create the Project
argocd proj create argocd --description 'Project for Declaratively Managing ArgoCD'

# Add Source and Destinations
argocd proj add-source argocd 'git@github.com:estenrye/cd-homelab.git'
argocd proj add-destination argocd in-cluster argocd
```

