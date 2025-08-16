

# IPv6 Home Lab

## References

- [YouTube: UK IPv6 Council: Kubernetes and IPv6](https://www.youtube.com/watch?v=JeYp_VJcwe4)
- [Support Tools: Running K3s in an IPV6-Only Environment with Cillium](https://support.tools/k3s-ipv6-cilium/)
- [Medium: David Elizondo: Planning a k3s Multi-Cluster Network with L2 and BGP on Unifi UDM](https://medium.com/@david-elizondo/planning-a-k3s-multi-cluster-network-with-l2-and-bgp-on-unifi-udm-ae4480a7b4f7)
- [map59.com: Ubiquiti UDM Pro running FRR BGP](https://medium.com/@david-elizondo/planning-a-k3s-multi-cluster-network-with-l2-and-bgp-on-unifi-udm-ae4480a7b4f7)
- [Medium: David Elizondo: Multi-Cluster Kubernetes Lab Setup with Cillium Cluster Mesh](https://medium.com/@david-elizondo/multi-cluster-kubernetes-lab-setup-with-cilium-cluster-mesh-e609b61f0fc7)
- [Medium: Muhammad Iqbal: K3s - IPv6 Only Configuration](https://qeba.medium.com/k3s-ipv6-only-configuration-8c159385a0a)
- [Jana: How to configure K3s for IPv6-only operation using Cillium](https://jsteuernagel.de/posts/k3s-cilium-ipv6/)
- [Shane Renshaw: The IPv6 only Kubernetes Home Lab](https://renshaw.au/posts/the-cluster/)
- [r/Ubiquiti: [Guide/Example] BGP Peering For IPv4 and IPv6](https://www.reddit.com/r/Ubiquiti/comments/8l4ylj/guideexample_bgp_peering_for_ipv4_and_ipv6/)
- [Medium: Rob De Graaf: Having fun with Cilium (BGP) — Talos and UniFi Cloud Gateway Ultra](https://medium.com/@rob.de.graaf88/having-fun-with-cilium-bgp-talos-and-unifi-cloud-gateway-ultra-111ffb39757e)
- [NRSC: Network Startup Resource Center: BGP for All](https://learn.nsrc.org/bgp)
- [FRR: BGP](https://docs.frrouting.org/en/latest/bgp.html)
- [Network World: Jeff Doyle: The Logic of Bad IPv6 Address Management](https://www.networkworld.com/article/742850/the-logic-of-bad-ipv6-address-management.html)
- [Vegards Blog: My Unifi Gateway just learned to do BGP!](https://vegard.blog.engen.priv.no/?p=437)
- [kube-vip.io](https://kube-vip.io/)
- [Medium: Çağatay Gürtürk: Kubernetes on Raspberry Pi and BGP Load Balancing with UniFi Dream Machine Pro](https://itnext.io/kubernetes-on-raspberry-pi-and-bgp-load-balancing-with-unifi-dream-machine-pro-d5b94b6cfe99)
- [chrisdooks.com: Configure BGP on a UniFi Dream Machine (UDM) v3.1.x or later](https://chrisdooks.com/2023/06/26/configure-bgp-on-a-unifi-dream-machine-udm-v3-1-x-or-later/)


## Configuring BGP on my UniFi UDM SE

```
log syslog informational

! -*- bgp -*-
!
hostname $UDMSE_HOSTNAME
password $PASSWORD
frr defaults traditional
log file stdout
!
! 2-byte ASNs (64512 to 65534) for private use.
! 4-byte ASNs (4200000000 to 4294967294) for private use.
!
router bgp 4200000000
  bgp ebgp-requires-policy

```

## Registering an ARIN Account

References
- https://www.arin.net/resources/guide/
- [BGP multihoming - Part 1](https://www.youtube.com/watch?v=YEjuuAV7fH0)
- [BGP multihoming - Part 2](https://www.youtube.com/watch?v=Tig0X5_qJHU)

## Images to airgap
- docker.io/calico/apiserver:v3.30.2
- docker.io/calico/kube-controllers:v3.30.2
- docker.io/calico/typha:v3.30.2
- quay.io/tigera/operator:v1.38.3