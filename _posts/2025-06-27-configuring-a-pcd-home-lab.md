---
layout: post
title: Refactoring my Home Lab with Platform9 Private Cloud Director
date: 2023-06-27 16:50:00 -0600
categories:
  - home-lab
  - notes
  - pcd
tags:
  - openstack
  - kubernetes
  - platform9
  - pcd
---

## Netdata

- Provisioned NetData App on my TrueNAS Scale file server.
- Bought HomeLab License for $90/year.
- Attempted to configure OIDC authentication with JumpCloud, but ran into [this bug](https://github.com/netdata/netdata-cloud/issues/1098) that prevents me from using JumpCloud's OIDC Provider.
- Started looking into using [Authentik](https://goauthentik.io) or [Authelia](https://www.authelia.com/) as my centralized authentication dashboard.  May evaluate deploying on Rackspace SPOT.
- Need to try deploying NetData on Rackspace Spot.

## Crossplane 2

To get Crossplane 2 deployed, I had to create a GitHub Personal Authentication Token to allow access to Kubernetes to pull the images from GHCR.

To Create this token:

1. Navigate to [https://github.com/settings/tokens](https://github.com/settings/tokens).
2. Click `Generate new token`
3. Click `Generate new token (classic)`
4. Name the token with the note field.
5. Set the expiration date to never.
6. Check the box next to `read:packages`.
7. Click `Generate Token`.
8. Save the token, github username and the hostname ghcr.io to your secret provider of choice.  Mine is 1Password.

To Deploy Crossplane:

```bash
# Create an Image Pull Secret for ghcr.io
kubectl create secret docker-registry -n crossplane-system ghcr-reg-secret --docker-username=`op read --account ryefamily.1password.com op://Home_Lab/ghcr-pat/username` --docker-password=`op read --account ryefamily.1password.com op://Home_Lab/ghcr-pat/credential` --docker-server=`op read --account ryefamily.1password.com op://Home_Lab/ghcr-pat/hostname`

# Install crossplane
helm repo add crossplane-preview https://charts.crossplane.io/preview
helm repo update
helm install crossplane \
--namespace crossplane-system \
--create-namespace crossplane-preview/crossplane \
--version v2.0.0-preview.1 \
--set imagePullSecrets\[0\]="ghcr-reg-secret"

# Install Netdata Provider
git clone git@github.com:estenrye/provider-netdata.git
cd provider-netdata
kubectl apply -k package
```


## Using Crossplane to provision Netdata Rooms using Kubernetes Resources

The first step I took was to use the [upjet-provider-template](https://github.com/upbound/upjet-provider-template) to create a new provider repository that could be used to convert the NetData terraform provider to a Crossplane provider.  Then I continued to follow the instructions [documented here](https://github.com/crossplane/upjet/blob/main/docs/generating-a-provider.md).



## Tools
- [KeyCloak](https://www.keycloak.org/)
- [Netdata](https://netdata.cello.so/L5BAUWWe2Ti)
- [OpenBenchMarking.org](https://openbenchmarking.org/)
- [Phoronix Test Suite](https://www.phoronix-test-suite.com/)

## References

### OpenStack Cinder
- [YouTube: OpenInfra Foundation: Understanding Cinder Performance in an OpenStack Environment.](https://www.youtube.com/watch?v=UvF1q1Qb0sI)

### Netdata
- [YouTube: Lawrence Systems: Netdata: The Easy to Deploy, Easy to Use, Linux Infrastructure Performance Monitoring Dashboard](https://www.youtube.com/watch?v=Hsq6ebnzPtI)
- [YouTube: Raid Owl: Let's try some Home Lab Monitoring - ft. Netdata](https://youtu.be/F77y2wU30R0?si=7jFebwH3u2izqKqp)


## ToDo
- Open an issue to update Crossplane 2 documentation with steps to create a ghcr registry secret.
- build a kustomize configuration to install Netdata provider.


## Ubuntu Install on Bare Metal Controlplane Hypervisor

### Network Configuration

- bond0
  - Interfaces
    - enp5s0f0
    - enp5s0f1
  