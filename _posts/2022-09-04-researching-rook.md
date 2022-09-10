---
layout: post
title: Rook/Ceph Background Research
date: 2022-09-04T09:15:00+00:00
categories: [homelab, rook, storage, baremetal]
tags: [kuberenetes]
---

# Rook Implementation Research Notes

## Videos

- [YouTube - Getting Started with Rook](https://www.youtube.com/watch?v=wIRMxl_oEMM)
    - The Linux Foundation
    - January 07, 2020
    - Covers:
        - Installing rook
        - Setting up device usage
        - Setting up storage class
        - Setting up PVC

- [YouTube - Tuesday Tech Tip - Intro to Ceph Clustering Part 1 - When to Consider It](https://www.youtube.com/watch?v=yeAlzSp6yaE)
    - 45 Drives
    - May 26, 2020

- [YouTube - Tuesday Tech Tip - Intro to Ceph Clustering Part 2 - How Ceph Works](https://www.youtube.com/watch?v=HJivYTJ9Y54)
    - 45 Drives
    - June 02, 2020

- [YouTube - Tuesday Tech Tip - Intro to Ceph Clustering Part 3 - Data Security](https://www.youtube.com/watch?v=L5ug8UIs4tI)
    - 45 Drives
    - June 09, 2020

- [YouTube - Tuesday Tech Tip - Intro to Ceph Clustering Part 4 - Self Balancing and Self Healing](https://www.youtube.com/watch?v=jBWVcJYNjeA)
    - 45 Drives
    - June 16, 2020

- [YouTube - Intro to Rook: Storage for Kubernetes](https://www.youtube.com/watch?v=dA29dIK6g5o)
    - Jared Watts, Upbound & Alexander Trost, Cloudical
    - September 04, 2020

- [YouTube - Performance Optimization – Rook on Kubernetes](https://www.youtube.com/watch?v=v1u6bPM4SQU)
    - Mark Darnell & Ryan Tidwell, SUSE
    - September 04, 2020

- [YouTube - Getting started with ceph storage cluster setup](https://www.youtube.com/watch?v=Uvbp3mtOltw)
    - Daniel Persson
    - October 25, 2020

- [YouTube - Tuesday Tech Tip - Ceph Deployment Tools](https://www.youtube.com/watch?v=SLld70mnpbo&list=PL0q6mglL88AN9LGNfmEhu2fxZ_DNm5nZA)
    - 45 Drives
    - November 17, 2020

- [YouTube - Rook: Intro and Ceph Deep Dive](https://www.youtube.com/watch?v=j86OXjC1Jr8)
    - Blaine Gardner, Red Hat / Satoru Takeuchi, Cybozu, Inc.
    - May 14, 2021

- [YouTube - Tuesday Tech Tip - The Simplest Way to Build a Ceph Cluster](https://www.youtube.com/watch?v=ZsQp1vmn22M)
    - 45 Drives
    - August 17, 2021

- [YouTube - Kubernetes Fortnight: Solving storage in Kubernetes with the Rook operator](https://www.youtube.com/watch?v=0SOCWLmSrQs)
    - Platform9 Systems, Inc.
    - Chris Jones / Anup Barve
    - August 19, 2021
    - Covers:
        - High Level Overview and Key topics around Rook and Ceph

- [YouTube - ROOK – Ceph Backed Object Storage for Kubernetes Install and Configure](https://www.youtube.com/watch?v=cR-s26Zzx4Y)
  - Anthony Spiteri
  - September 03, 2021

- [YouTube - Tuesday Tech Tip - Tuning and Benchmarking for your Workload with Ceph](https://www.youtube.com/watch?v=7C9HI_f0bBo)
    - Mitch / 45Drives
    - October 26, 2021
    - Covers:
      - Performance Tuning
        - IOPS and Throughput Benchmarks on Underlying Disks
        - Network Performance Testing with iPerf
        - Latency tests between the nodes.
        - tuned for getting servers performing with lowest possible latency

- [YouTube - Rook: Intro and Ceph Deep Dive](https://www.youtube.com/watch?v=VBVVqiKvlCI)
    - Travis Nielsen, Sebastien Han, Blaine Gardner & Satoru Takeuchi
    - October 29, 2021

- [YouTube - How to set up a cluster with CephAdm](https://www.youtube.com/watch?v=ENsfa8WB6EI)
    - Daniel Persson
    - January 17, 2022

- [YouTube - Introduction to Rook](https://www.youtube.com/watch?v=VSjcm82l5UA)
    - Red Hat Developer
    - Travis Nielson / Rook Maintainer / Red Hat
    - May 10, 2022

- [YouTube - Rook in the cloud or on-prem?](https://www.youtube.com/watch?v=-4usMK00qW0)
    - Red Hat Developer
    - Travis Nielson / Rook Maintainer / Red Hat
    - May 10, 2022

- [YouTube - Getting started with Rook](https://www.youtube.com/watch?v=1SF8sTsMGYY)
    - Red Hat Developer
    - Travis Nielson / Rook Maintainer / Red Hat
    - May 10, 2022

- [YouTube - Installing Rook in an AWS cluster](https://www.youtube.com/watch?v=DYofW39Q5Z8)
    - Red Hat Developer
    - Travis Nielson / Rook Maintainer / Red Hat
    - May 10, 2022


## Tutorials

- [How to Deploy Rook with Ceph as a Storage Backend for your Kubernetes Cluster using CSI](https://platform9.com/learn/tutorials/rook-using-ceph-csi)

## Blog Posts

- [Production-grade Deployment of PVC-based Rook/Ceph Cluster](https://blog.kintone.io/entry/2020/09/18/175030)
- [RedHat article on low-latency performance tuning](https://access.redhat.com/sites/default/files/attachments/201501-perf-brief-low-latency-tuning-rhel7-v2.1.pdf)

## Questions

- How can I set up encryption at rest in Ceph?
- How do I craft a backup strategy for ceph?
- Can backup be managed at the Ceph level?

## Fun tools I found along the way

- [asciinema](https://asciinema.org/) (Terminal Recording Tool)
    - [asciinema-server](https://github.com/asciinema/asciinema-server) for self-hosted publishing
    - [asciinema-player](https://github.com/asciinema/asciinema-player#self-hosting-quick-start) for embedding in a website

# Benchmarking

## Benchmark disks that will eventually make up the OSDs

### IOPS Benchmark

First step is to get a baseline understanding of the disk's IOPS.

```bash
BLOCK_DEVICE=/dev/sdl

# Get a baseline on the disk's IOPS
fio --filename=${BLOCK_DEVICE} \
    --direct=1 \
    --fsync=1 \
    --rw=randwrite \
    --bs=4k \
    --numjobs=1 \
    --iodepth=1 \
    --runtime=60 \
    --time_based \
    --group_reporting \
    --name=4k-sync-write-test
```

Take Note of the IOPS returned.  Track repeated benchmarks with the same
model of drive over and over to get a good understanding of how the drive is
going to perform, noting any variance between the tests.  It's important to
ensure that the variance of the IOPS of the disks in the system is not wildly
different between disks.

### Throughput Benchmark

Next step is to run a throughput benchmark on each of the drives.  The key
metric in this test is the Bandwidth (BW) metric.  Repeat this test for each
drive.

```bash
BLOCK_DEVICE=/dev/sdl

fio --filename=${BLOCK_DEVICE} \
    --direct=1 \
    --fsync=1 \
    --rw=write \
    --bs=1M \
    --numjobs=1 \
    --iodepth=1 \
    --runtime=60 \
    --time_based \
    --group_reporting \
    --name=1M--sync-throughput-test
```

## Benchmark the network that our storage cluster will communicate over.

Make sure the network bandwidth is where it should be.  Ensure retransmission
rate is not high and congestion window is not too small.

Select a node to be the server in this test.

### Install iperf3 on client and server

```bash
sudo apt install -y iperf3
```

### On the Server

First we are going to configure iperf3 on the server to listen on port 5201.

```bash
SERVER_IP_ADDRESS=192.168.200.2

iperf3 -B ${SERVER_IP_ADDRESS} -s
```

### On the Client

Next we need to configure iperf3 on the client to connet to the server.
Note that we are explictly telling iperf3 which interface we are using to
perform the test.  This is important because the ceph cluster may have multiple
interfaces which may be configured for frontend/backend traffic.

```bash
SERVER_IP_ADDRESS=192.168.200.2
CLIENT_IP_ADDRESS=192.168.200.3

# Test from client ==> server
iperf3 -B ${CLIENT_IP_ADDRESS} -c ${SERVER_IP_ADDRESS}

# Test from server ==> client
iperf3 -B ${CLIENT_IP_ADDRESS} -c ${SERVER_IP_ADDRESS} -R
```

There are two metrics we are interested in looking at.

- Retransmission: the number of times the sender needed to resend the packet
  before it was actually accepted.  Smaller numbers are better.
- Congestion Window: the amount of data that can be sent without having to get
  an ACk sent back.  Larger numbers are better, ideally around 1 MB.

Repeat this test on all nodes and interfaces that will be part of the Ceph cluster.

## Get an idea of any packet errors or drops.

```bash
netstat -i | column -t
```

This will give us an overview of the following:

- RX-OK: Received packets that were OK.
- RX-ERR: Any packets received that errored.
- RX-DRP: Any packets that were dropped.
- TX-OK: Sent packets that were OK.
- TX-ERR: Any packets sent that errored.
- TX-DRP: Any packets sent that dropped.

It's good to monitor these statistics over time to see if the network is seeing
any kind of drops or issues.

## Perform a ping test to get a feel for latency between the nodes.

```bash
SERVER_IP_ADDRESS=192.168.200.2

ping ${SERVER_IP_ADDRESS}
```

Ceph performance hinges on network latency being as low as possible.

# Tuning the system for low latency

In this section we want to disable some of the deeper c-states (idle states)
and tune the CPU's p-states for peak performance and getting peak boost clocks.

Using the tuned tool we an set in place multiple, meaningful tunings by enabling
specific tuning profiles.  For Ceph, we want to use the network-latency profile.

## Make sure BIOS isn't going to clobber your settings

## Using tuned to tune the system

```bash
# List tuning profiles
tuned-adm list

# take a look at latency-performance configurations
egrep -v '^$|^\#|\[' /usr/lib/tuned/latency-performance/tuned.conf

# take a look at the network-latency configurations
egrep -v '^$|^\#|\[' /usr/lib/tuned/network-latency/tuned.conf

# set network-latency as the active profile.
tuned-adm profile network-latency

# confirm network-latency profile is the active profile
cat /etc/tuned/active_profile
```

## Verify the tuning profile has been applied

Make sure after applying the tuned profile and restarting the machine that all
of the CPU cores are 100% in the C1 state and not in any of the deeper c-states
which cause additional latency, reduce the impact of power management, reduce
task migrations and reduce the amount of outstanding dirty pages kept in memory.

These changes will result in a higher power draw!

```bash
turbostat sleep 5
```
