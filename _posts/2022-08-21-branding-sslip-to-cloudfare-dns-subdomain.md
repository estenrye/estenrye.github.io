---
layout: post
title: Branding sslip.io to a Cloudflare hosted DNS Subdomain
date: 2022-07-21 19:35:00 -0500
categories:
  - livestream
  - cloudflare
  - dns
  - dynamic-dns
tags:
  - dns
  - sslip.io
---

# What is sslip.io?

[sslip.io](https://sslip.io) is a free DNS service that, when queried with a hostname
with an embedded IP address returns that IP address.  It was inspired by xip.io which
was created by [Sam Stephenson](https://github.com/sstephenson) when he worked at
[Basecamp](https://basecamp.com/).

# Why use sslip.io?

As I write this, I am on a Delta flight to San Jose, CA.  I have 3 hours until I land,
and I want to deploy some of my common Kubernetes platform tooling to my 
[Rancher Desktop](https://rancherdesktop.io/) local Kubernetes instance.  Part of my 
tooling uses [cert-manager](https://cert-manager.io) to issue my services free, trusted
[LetsEncrypt](https://letsencrypt.io) certificates.  This works great as long as one of two things
are true:

1. You have a publicly queriable DNS name that returns a valid public IP that resolves to
   your machine for an HTTP-01 challenge.
2. You have a public DNS that cert-manager can write to for a DNS-01 challenge.

Another reason I want to use this tool is that I heavily use Ingress in my Kubernetes
development.  Kubernetes Ingress uses the DNS name and http path to route traffic to the
appropriate service.  I also don't want to have to edit my Cloudflare DNS every time I
hop onto a new wifi network.

Lastly, I am curious if I can actually satisfy an HTTP-01 challenge while in the air.

# First things first, branding sslip.io

This step is completely optional.  I just wanted to see if I could do it.  If you don't
have a Cloudflare hosted DNS with appropriate permissions to modify it, feel free to
skip this section.  The rest of this tutorial will function just fine without it.

This first bit of code will create the NS records for sslip.io in my `a.rye.ninja`
subdomain.  Note that you will need to substitute your Cloudflare API token and
Zone Id for my 1Password cli commands.

```bash
OP_ACCOUNT='ryefamily.1password.com'
OP_VAULT='Home_Lab'
OP_ITEM='cloudflare_api_token_a'

CLOUDFLARE_API_TOKEN=`op item get ${OP_ITEM} --account ${OP_ACCOUNT} --vault ${OP_VAULT} --fields password`
CLOUDFLARE_ZONE_ID=`op item get ${OP_ITEM} --account ${OP_ACCOUNT} --vault ${OP_VAULT} --fields zone_id`

curl -X POST "https://api.cloudflare.com/client/v4/zones/${CLOUDFLARE_ZONE_ID}/dns_records/" \
    -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
    -H "Content-Type: application/json" \
    --data '{"type":"NS","name":"a","content":"ns-aws.sslip.io.","proxied":false,"ttl":"120"}' | jq

curl -X POST "https://api.cloudflare.com/client/v4/zones/${CLOUDFLARE_ZONE_ID}/dns_records/" \
    -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
    -H "Content-Type: application/json" \
    --data '{"type":"NS","name":"a","content":"ns-gce.sslip.io.","proxied":false,"ttl":"120"}' | jq

curl -X POST "https://api.cloudflare.com/client/v4/zones/${CLOUDFLARE_ZONE_ID}/dns_records/" \
    -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
    -H "Content-Type: application/json" \
    --data '{"type":"NS","name":"a","content":"ns-azure.sslip.io.","proxied":false,"ttl":"120"}' | jq
```

After executing the above commands, you should be able to query the new DNS service.

```bash
dig 127-0-0-1.a.rye.ninja +short
```

You should get the following response.

```bash
127.0.0.1
```

# Tools you will need for the tutorial

You'll need Rancher Desktop installed

```bash
brew install rancher-desktop
```

If you haven't already cloned my [cd-homelab](https://github.com/estenrye/cd-homelab)
repository, you should probably go do that now.

It has a Brewfile that will help you install all the tools necessary to work through
the remainder of this tutorial.

```bash
git clone https://github.com/estenrye/cd-homelab.git
cd cd-homelab
brew bundle
```

# Disabling the default Traefik Ingress Controller

Rancher Desktop ships with Traefik as its default Ingress Controller.  This isn't
what I want in my environment so I am going to disable it to free up ports 80 and
443 for the ingress-nginx Ingress Controller

TODO: Write Instructions for this.

# Deploying Ingress-Nginx to my Local Kubernetes Cluster

In this section we'll deploy an ingress controller to our local Kubernetes cluster.
This is necessary as it will both allow traffic into the cluster and serve our
HTTP-01 challenges for LetsEncrypt.

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

Once everything finishes deploying, you should be able to curl localhost on port 80.

```bash
curl http://localhost
```

You should get the following output.

```html
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

# Deploying cert-manager to my Local Kubernetes Cluster

Next step is to install cert-manager and its respective custom resource
definitions.  To do that, run the following helm command.

```bash
helm repo add cert-maanger https://charts.jetstack.io

helm upgrade --install \
  cert-manager cert-manager/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.9.1 \
  --set installCRDs=true
```

Now that we have cert-manager and its CRDs installed, now we need to configure a
`ClusterIssuer` object.  The following yaml will create the required object.

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: example-issuer
  namespace: cert-manager
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: example-issuer-account-key
    solvers:
    - http01:
        ingress:
          class: nginx
```

