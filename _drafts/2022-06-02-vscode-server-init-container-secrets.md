---
layout: post
title: VS Code Server Kuberenetes Init Container
date: 2022-06-02 09:01:00 -0500
categories:
  - Platform9 Live
tags:
  - VS Code Server
  - Kubernetes
---

# Problem

At the end of my last live stream, I was struggling to get Docker-in-Docker into a running state in my
VS Code Server pod.  Core to my struggles was that my Kubernetes Secrets were not writable from inside
the pod.  This breaks the automatic generation of Docker TLS Certificates used to secure Docker-in-Docker's
docker socket.  In this post, I am exploring initializing a Kubernetes Secret using an init container.

# What makes an init container special?

Init containers are just like regular containers except that they start up before the regular containers do, 
do not support `lifecycle`,`livenessProbe`, `readinessProbe`, or `startupProbe`, and must sequentially run 
to completion before the Pod can be ready.  For the problem above, they are ideal for the use case because
they can be used to initialize Pod state that I would rather not specify defaults for in the `values.yaml`
of my helm chart.

# Goals

- Add Smallstep CA as a deployment, using PV file backend as high availability is not required.
- Add a cert-manager Issuer for the namespace.
- 
- Create a docker image that will generate the CA, Server and Client certificates for Docker-in-Docker
  and publishes those certificates to Kubernetes Secrets.
- Add an init container to the VS Code Server deployment that calls the newly created image.
- Automate Renewal

# Adding Smallstep CA as a Helm Chart Dependency

## Add the dependency to Chart.yaml

### Chart.yaml
```yaml
dependencies:
  - name: step-certificates
    repository: https://smallstep.github.io/helm-charts/
    version: 1.18.2+20220324
```

### values.yaml
```yaml
step-certificates: {}
```

## Generate config for Values.yaml

1. Generate step-certificates values.yaml from step cli.

```bash
step ca init --helm --deployment-type=standalone --name=vscode-server-ca --dns=step-ca.svc.cluster.local --address=:443 --provisioner=step-ca-provisioner > values.yml
```

1. Add Password to values.yml.

```bash
PROVISIONER_PASSWORD='your-password-here'
B64_PASSWORD=`echo -n ${PROVISIONER_PASSWORD} | base64`
sed -i -e "s|provisioner_password\:.*\$|provisioner_password\: ${B64_PASSWORD}|" values.yml
sed -i -e "s|ca_password\:.*\$|ca_password\: ${B64_PASSWORD}|" values.yml
```

# References

## Kubernetes Documentation

- [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

## Helm Documentation

- [Helm Dependency](https://v3.helm.sh/docs/helm/helm_dependency/)
- [Overriding Subchart Values from a Parent Chart](https://helm.sh/docs/chart_template_guide/subcharts_and_globals/#overriding-values-from-a-parent-chart)


## Blog Posts

- [Exploring Kubernetes Init Container, ConfigMap & Secrets](https://www.monkeylittle.com/blog/2017/02/22/exploring-kubernetes-init-container-configmap-secret.html) by John Turner, posted February 22, 2017
- [Automating TLS certificate management in Docker](https://smallstep.com/blog/automate-docker-ssl-tls-certificates/) by Carl Tashian, Smallstep, posted October 25, 2021
- [Chart Dependencies](https://werf.io/documentation/v1.2/advanced/helm/configuration/chart_dependencies.html), werf.io