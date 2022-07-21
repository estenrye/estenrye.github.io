---
layout: post
title: Deploying 1Password Connect With ArgoCD
date: 2022-07-20 19:35:00 -0500
categories:
  - notes
  - argocd
  - secrets
tags:
  - continuous delivery
  - argocd
  - secrets
  - 1password
---

# References

- [Integrating Kubernetes with 1Password for infrastructure secrets](https://blog.bennycornelissen.nl/post/onepassword-on-kubernetes/)

# Source Code

- [CD-HomeLab](https://github.com/estenrye/cd-homelab)
- [1Password Connect Helm Chart](https://github.com/1Password/connect-helm-charts)
- [1Password Operator](https://github.com/1Password/onepassword-operator)

# Deploy and Configure CertManager

- Build a Helm Chart to Wrap CertManager Helm Chart
  - Include CertManager as a Dependency
  - Add default values.
  - Create a LetsEncrypt DNS-01 ClusterIssuer
  - Create a Cloudflare API Token Secret
  - Publish to a helm repository or git repository
  
- Deploy Helm Chart Imperatively using the ArgoCD CLI
  
## Build a Helm Chart to Wrap CertManager Helm Chart

Create a new chart directory.   This will hold our chart files.

```bash
helm create cert-manager-install
```

Clean up the templates directory, delete everything except the
`_helpers.tpl` file

```bash
rm -rf \
  ./cert-manager-install/templates/tests \
  ./cert-manager-install/templates/*.txt \
  ./cert-manager-install/templates/*.yaml
```

### Include CertManager as a Dependency

The first file we need to modify is the `Chart.yaml` file.
This file will hold our chart metadata and most importantly,
our chart dependencies.  This is how we will instll cert-manager
without needing to maintain our own chart to install it.

```yaml
apiVersion: v2
name: cert-manager-install
description: |
  A Helm chart for installing and configuring cert-manager
  in my home lab environment using Cloudflare, Lets Encrypt
  and the ACME DNS01 challenge Cluster Issuer.
type: application
version: 0.1.0
appVersion: "v1.8.2"
dependencies:
  - name: cert-manager
    repository: https://charts.jetstack.io
    version: v1.8.2
    condition: cert-manager.enabled
```

Our cert-manager chart dependency has four key components.

- `name` specifies the name of the chart.
- `repository` specifies the url of the chart repository where the chart is hosted.
- `version` specifies which release of the chart to download
- `condition` specifies a chart value helm will interogate to determine if the
  dependency should be installed.

At this point you should be able to build the chart dependencies using the following command:

```bash
helm dependency build ./cert-manager-install
```

This will download a tgz file that contains the cert-manager chart dependency.  Once this is
downloaded, you can test the templating using the following command:

```bash
helm template my-cm ./cert-manager-install --set cert-manager.enabled=false
# output should be empty

helm template my-cm ./cert-manager-install --set cert-manager.enabled=true
# output should generate all objects deployed by the cert-manager chart dependency.
```

### Add default values

The next file we need to modify is the values.yaml file.  This file holds our default
configuration for the helmchart.  Delete everything in thid file and replace it with the
following:

```yaml
acme_email: ''
acme_server: https://acme-v02.api.letsencrypt.org/directory
cloudflare_api_token: ''

cert-manager:
  enabled: false
  global:
    logLevel: 2
  installCRDs: true
  podDnsPolicy: "None"
  podDnsConfig:
    nameservers:
      - "1.1.1.1"
      - "1.0.0.1"
```

The first three keys `acme_email`, `acme_server` and `cloudflare_api_token` are
all values used by the templates in the chart we are creating.  Everything under
the `cert-manager` key are values fed into the `cert-manager` chart dependency.

### Create a LetsEncrypt DNS-01 ClusterIssuer

Next, we are going to create a new template file called
`./cert-manager-install/templates/ClusterIssuer.yaml`.  This file will create our
ClusterIssuer object that will configure cert-manager to use Cloudflare for DNS-01
ACME Challenges.

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: {{ include "cert-manager-install.fullname" . }}
  labels:
    {{- include "cert-manager-install.labels" . | nindent 4 }}
spec:
  acme:
    email: {{ .Values.acme_email }}
    server: {{ .Values.acme_server }}
    privateKeySecretRef:
      name: cloudflare-cluster-issuer-private-key-secret
    solvers:
      - dns01:
          cloudflare:
            apiTokenSecretRef:
              name: {{ include "cert-manager-install.fullname" . }}-cloudflare
              key: api-token
```

### Create a Cloudflare API Token Secret

Lastly, we are going to create a new template file called
`.cert-manager-install/templates/secret.yaml` that will create a secret object that
will hold our Cloudflare API token.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "cert-manager-install.fullname" . }}-cloudflare
  labels:
    {{- include "cert-manager-install.labels" . | nindent 4 }}
type: Opaque
data:
  api-token: {{ .Values.cloudflare_api_token | b64enc }}
```

### Publish to a helm repository or git repository

The last step is to publish your helm chart to either a git or helm repository.

## Deploy Helm Chart Imperatively using the ArgoCD CLI

Use the following command to deploy the chart using ArgoCD

```bash
argocd app create cert-manager-config \
  --repo https://estenrye.github.io/helm-charts \
  --helm-chart cert-manager-install \
  --revision 0.1.0 \
  --dest-namespace default \
  --dest-server 'https://kubernetes.default.svc' \
  --helm-set acme_email=your-email@example.com \
  --helm-set cloudflare_api_token=your-cloudflare-api-token \
  --helm-set cert-manager.enabled=true
```