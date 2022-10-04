---
layout: post
title: Podcast Listening Notes
date: 2022-09-10T12:14:00+00:00
categories:
  - notes
tags:
  - random-thoughts-and-ideas
---

# Podcasts

I have decided to listen through the backlog of the 
[Kubernetes Podcast by Google](https://kubernetespodcast.com/)
on my drive to my future brother-in-law's bachelor party.

Listened to the following podcasts
- https://kubernetespodcast.com/episode/003-gvisor/
  - Questions for later:
    - Should I implement gVisor in an air-gapped environment?
    - What type of workloads require this level of isolation?
    - Kubernetes provides a means to select the container runtime environment,
      add gVisor as a capability to play with it eventually?
- https://kubernetespodcast.com/episode/005-documentation/
  - Currently my notes site is Jekyll based.
  - Should I consider migrating to Hugo at some point?
  - Primary reason for Kubernetes docs to migrate from Jekyll to Hugo
    was for performance as the site grew in scale.
- https://kubernetespodcast.com/episode/006-skaffold/
  - Should I include Skaffold in my development toolset?
  - Skaffold is a command line tool that facilitates continuous development for 
    Kubernetes applications. You can iterate on your application source code locally 
    then deploy to local or remote Kubernetes clusters. Skaffold handles the workflow 
    for building, pushing and deploying your application. It also provides building 
    blocks and describe customizations for a CI/CD pipeline.
  - How does this compare to a tool like ArgoCD?
- https://kubernetespodcast.com/episode/007-kustomize-with-a-k/
- https://kubernetespodcast.com/episode/008-security/
- https://kubernetespodcast.com/episode/009-sre/