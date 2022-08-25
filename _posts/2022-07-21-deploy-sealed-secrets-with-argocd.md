---
layout: post
title: Deploying Sealed Secrets With ArgoCD
date: 2022-07-21 19:35:00 -0500
categories:
  - notes
  - argocd
  - secrets
  - kubernetes
tags:
  - continuous delivery
  - argocd
  - secrets
  - kubernetes
---

# References

- [Github | Bitnami/Sealed-Secrets Helm Install Docs](https://github.com/bitnami-labs/sealed-secrets#helm-chart)
- [Github | ExternalDNS](https://github.com/kubernetes-sigs/external-dns/tree/master/charts/external-dns)
  - [Github | ExternalDNS | Setting up ExternalDNS for Services on Cloudflare](https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/cloudflare.md)
  - [Github | ExternalDNS | Setting up ExternalDNS on GKE with nginx-ingress-controller](https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/nginx-ingress.md)
- [Github | ingress-nginx | ExternalDNS Service Configuration](https://github.com/kubernetes/ingress-nginx/blob/2843bb264f5d31b2ed2514c300c65c33aca2557a/charts/ingress-nginx/README.md#externaldns-service-configuration)

- [Sealed Secrets: Protecting your passwords before they reach Kubernetes](https://docs.bitnami.com/tutorials/sealed-secrets)
- [ExternalDNS for NGINX Ingress Controller Using F5 CIS with BIG-IP DNS](https://community.f5.com/t5/technical-articles/externaldns-for-nginx-ingress-controller-using-f5-cis-with-big/ta-p/291871)
- [YouTube | The missing piece - Kubernetes ExternalDNS ](https://www.youtube.com/watch?v=9HQ2XgL9YVI)

# Objectives

- Install ingress-nginx in a Kubernetes Cluster using ArgoCD
- Install Sealed Secrets in a Kubernetes Cluster using ArgoCD
  - Deploy Sealed Secrets using imperative command
  - Deploy Sealed Secrets using imperative command
- Using Sealed Secrets in a GitOps workflow.
  - Setting up the kubeseal CLI
- Use Sealed Secret to hold bootstrap secrets for 1PasswordConnect and Cert-Manager

# Install Sealed Secrets in a Kubernetes Cluster using ArgoCD

You can use either the imperative command or the declarative yaml to deploy.
If you don't control the namespace where argocd is deplyed, you will likely need
to use the imperative command, otherwise my preference is the declarative manifest.
The reason I prefer the declarative approach is that ArgoCD can monitor a git repo
with Application Manifests and automatically apply any changes.

## Deploy Sealed Secrets using imperative command

```bash
argocd app create sealed-secrets \
  --repo https://bitnami-labs.github.io/sealed-secrets \
  --helm-chart sealed-secrets \
  --revision v2.4.0 \
  --dest-namespace kubeseal \
  --dest-server 'https://kubernetes.default.svc' \
  --helm-set fullnameOverride=sealed-secrets-controller \
  --sync-option CreateNamespace=true
```

## Deploy Sealed Secrets using declarative yaml

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sealed-secrets
  namespace: argocd
spec:
  project: default
  source:
    chart: sealed-secrets
    repoURL: 'https://bitnami-labs.github.io/sealed-secrets'
    targetRevision: v2.4.0
    helm:
      parameters:
        - name: fullnameOverride
          value: sealed-secrets-controller
  destination:
    server: https://kubernetes.default.svc
    namespace: kubeseal
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
```

# Using Sealed Secrets in a GitOps workflow.

The main reason I am installing sealed secrets in my cluster is to enable
me to put cluster bootstrapping secrets in a public repository.  Sealed
Secrets use public key cryptography to provide this capability.  The first
thing we will need to do is get the public key from the cluster we are
targeting for deployment.  This plus the namespace and the name of the
secret are used to encrypt and salt the plaintext secret.  This ensures
that only the namespace in the cluster we intend deploy the secret can
decrypt the secret.

## Setting up the `kubeseal` CLI

These days I am using homebrew to install most of my CLI tools.  To install
kubeseal, run the following command.

```bash
brew install kubeseal
```

## Enabling Offline Encryption

By default, kubeseal will use your current kubernetes context to retrieve the
current sealing certificate from the target cluster.  This may not be the best
solution if you want to enable developers who are not granted access to the
target cluster to be able to access the encryption certificate.  One way to do
this is to use kubeseal to fetch an offline copy and publish it to a known URL.
This allows the ops team to abstract the fact that the key is rotated every 30
days from the developers.  But because I am an actively lazy devops engineer
we are going to automate this with Kubernetes.

### Building a multi-arch Docker Image with Kubeseal and Kubectl


```yaml
FROM golang:latest as build
LABEL maintainer="Esten Rye <esten.rye+docker@ryezone.com>"
WORKDIR /app
ARG VERSION="main"
RUN GO111MODULE=on CGO_ENABLED=0 go install -a github.com/bitnami-labs/sealed-secrets/cmd/kubeseal@${VERSION}
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/$(uname -m)/kubectl"
RUN chmod 555 /app/kubectl /go/bin/kubeseal

FROM scratch
WORKDIR /
COPY --from=build /go/bin/kubeseal kubeseal
COPY --from=build /app/kubectl kubectl
CMD [ "/kubeseal", "--version" ]
```

### Building a 