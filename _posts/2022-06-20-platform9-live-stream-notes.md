---
layout: post
title: Platform9 Live - Stream Notes - VS Code Server - Kubeconfig
date: 2022-06-01 21:45:00 -0500
categories:
  - Platform9 Live
tags:
  - VS Code Server
  - Kubernetes
---

# VS Code Server Kube Config

## Goals for the Stream

- Modify my VS Code helm chart to setup a kubernetes role
  - Full privileges to the vscode-server namespace.
  - Link this role with a role binding to the service account generated by the helm chart.
  - configure my kubectl and vs code kubernetes plugins to use the service account token.

## Creating a role for vscode-server

### Resources Referenced

- [StackOverflow: Creating admin role for the namespace](https://stackoverflow.com/questions/68592209/creating-admin-role-for-the-namespace)
- [Kubernetes Documentation: Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

### Cluster Role Binding Definition

For this role, I want full administrative permissions for the namespace.

To generate the ClusterRoleBinding:

```bash
k create clusterrolebinding \
  --serviceaccount=vscode-server:code-server-vscode-server \
  --clusterrole=admin admin-cluster-role-binding \
  --dry-run=client \
  -o yaml
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: null
  name: admin-cluster-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- kind: ServiceAccount
  name: code-server-vscode-server
  namespace: vscode-server
```

Turning it into a template that can be executed by the Helm Chart

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: {{ include "vscode-server.fullname" . }}-binding
  labels:
    {{- include "vscode-server.labels" . | nindent 4 }}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- kind: ServiceAccount
  name: {{ include "vscode-server.serviceAccountName" . }}
  namespace: {{ .Release.Namespace }}
```

## Building a Custom VS Code Server Image

In order to use brew to install pre-commit and other tools, I need build-essentials.
To start with, we are going to add build-essentials to the image and create a build
process.

### References

- [How to use apt install correctly in your Dockerfile](https://techoverflow.net/2021/01/13/how-to-use-apt-install-correctly-in-your-dockerfile/)

### Customizing the image

```Dockerfile
FROM linuxserver/code-server:4.4.0
ENV DEBIAN_FRONTEND=noninteractive
RUN apt update \
  && apt install -y \
         build-essential \
  && apt clean \
  && rm -rf /var/lib/apt/lists/*
```

### Adding a reusable Github workflow

```yaml
name: Build and Publish Docker Image

on:
  workflow_call:
    inputs:
      DOCKER_IMAGENAME:
        required: true
        type: string
      DOCKER_USERNAME:
        required: true
        type: string
      PUSH_IMAGE:
        required: true
        type: boolean
    secrets:
      docker_password:
        required: true

jobs:
  # define job to build and publish docker image
  build-and-push-docker-image:
    name: Build Docker image and push to repositories
    environment: Docker

    # run only when code is compiling and tests are passing
    runs-on: ubuntu-latest

    # steps to perform in job
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      # setup Docker buld action
      - name: Set up Docker Buildx
        id: buildx
        uses: docker/setup-buildx-action@v1

      - name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ inputs.DOCKER_USERNAME }}
          password: ${{ secrets.docker_password }}

      - name: Build image and push to Docker Hub and GitHub Container Registry
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: docker/${{ inputs.DOCKER_USERNAME }}/${{ inputs.DOCKER_IMAGENAME }}
          # Note: tags has to be all lower-case
          tags: |
            ${{ inputs.DOCKER_USERNAME }}/${{ inputs.DOCKER_IMAGENAME }}:latest
          # build on feature branches, push only on main branch
          push: ${{ inputs.PUSH_IMAGE }}
```

### Using reusable Github Workflow

```yaml
# This is a basic workflow to help you get started with Actions

name: CI | Docker | estenrye/vscode-server


# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the master branch
  push:
    paths:
      - docker/estenrye/vscode-server/**
      - .github/workflows/CI-estenrye-vscode-server.yml
      - .github/workflows/CI-Docker-Image-Build-Template.yml

  pull_request:
    branches: [ master ]
    paths:
      - docker/estenrye/vscode-server/**
      - .github/workflows/CI-estenrye-vscode-server.yml
      - .github/workflows/CI-Docker-Image-Build-Template.yml

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # define job to build and publish docker image
  build-and-push-docker-image:
    name: Build Docker image and push to repositories
    uses: ./.github/workflows/CI-Docker-Image-Build-Template.yml
    with:
      DOCKER_IMAGENAME: vscode-server
      DOCKER_USERNAME: estenrye
      PUSH_IMAGE: ${{ github.ref == 'refs/heads/master' }}
    secrets:
      docker_password: ${{ secrets.DOCKER_PASSWORD }}
```