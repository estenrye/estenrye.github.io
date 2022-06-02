---
layout: post
title: Platform9 Live - Stream Notes - VS Code Server Docker-in-Docker
date: 2022-06-01 21:45:00 -0500
categories:
  - Platform9 Live
tags:
  - VS Code Server
  - Kubernetes
---

# Extensions Installed.

- GitHub.vscode-pull-request-github
- ms-kubernetes-tools.vscode-kubernetes-tools
  - Dependencies:
    - kubectl
    - helm
    - minikube

# Helm Chart Todos

- Provide secret injection for ssh key mapped to `/config/.ssh/id_rsa`
- Add service account for managing Kubernetes cluster
  - Allow deploy stuff to our local cluster.
  - Allow us to use kubectl

# Image Customization Todos

- add ZSH
- add ZSH customizations
- install Python3
- docker cli

# Things to Research

- How to run SSH Agent in the background.
  - enables password protected ssh keys with vscode git ui.
  - enables password-less ssh
- How to build containers using docker

# Adding docker capabilities to My VS Code Server instance on Kubernetes

## References

- [Docker in Docker?](https://itnext.io/docker-in-docker-521958d34efd)
- [Github: jpetazzo/dind](https://github.com/jpetazzo/dind)
- [Docker Hub: docker](https://hub.docker.com/_/docker/)
- [HowToGeek: How to Secure Docker’s TCP Socket With TLS](https://www.howtogeek.com/devops/how-to-secure-dockers-tcp-socket-with-tls/)
- [Docker: Use the Docker command line](https://docs.docker.com/engine/reference/commandline/cli/)

## Summary

I want to build Docker images in my Kubernetes hosted VS Code Server instance.
To do this, I am going to add a sidecar container which is running the Docker
daemon using Docker-in-Docker.  Then I will add the docker-cli to my VS Code
Server image and connect it to the Docker socket in the side car.  This should
allow me to build a docker image in a browser.

## Adding the Docker-in-Docker sidecar daemon container to the pod

Steps:

1. Select Docker-in-Docker image.
2. Create a new deployment in helm chart to deploy docker-in-docker.
3. Add a service for the docker socket.
4. Add a network policy to restrict access to the docker socket to containers within the same namespace.
  - allow only vscode server pod to access dind pod.

### Step 1: Select Docker-in-Docker image.

Browse the tags at [Docker Hub: library/docker | Tags](https://hub.docker.com/_/docker/?tab=tags).

I am selecting the `dind-rootless` tag, which will provide me a rootless installation of the Docker-in-Docker daemon
using the latest version of docker.  The helm chart will provide a tag override in the `values.yaml` that will allow
me to specify a more specific tag if I desire a more consistent deployment.

### Step 2: Create a new deployment in the VS Code Server helm chart.

- Got Docker-in-Docker deployed alongside vscode-server.
  - not working yet.
  - TLS Certificates are not being generated because Kubernetes Secrets mounted to paths are read-only.