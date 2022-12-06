---
layout: post
title: Building tools.rye.ninja - My Network Tools Server
date: 2022-09-23 00:00:00 -0500
categories:
  - home-lab
  - tools
tags:
  - tools
---

# Next Steps
[x] Install apt mirror
[x] Mirror Ubuntu Releases
[x] Install pihole
[x] Install Netbox
[x] Install Nitrokey HSM tooling
[ ] Create a custom step apt repository for step deb packages
[ ] Install step cli
[ ] Install [step-kms-plugin](https://github.com/smallstep/step-kms-plugin)
[ ] Install Step Certificates
[ ] Configure Shoelaces
[ ] Configure pihole DHCP settings
[ ] Install Cert-bot
[ ] Install Unifi Controller
[ ] Install HAProxy
[x] Automate Pi-Hole install

# Hardware

Here's a list of the hardware I used.

- Supermicro 1U Server
- 4 GB RAM
- Nitrokey HSM2

# Network Configuration

# Install

First step was to install and configure keepalived

https://keepalived.readthedocs.io/en/latest/installing_keepalived.html

https://www.redhat.com/sysadmin/keepalived-basics

## Keepalived configuration

```conf
vrrp_instance VI_1 {
        state MASTER
        interface enp2s0.61
        virtual_router_id 141
        priority 255
        advert_int 1
        authentication {
              auth_type PASS
              auth_pass 346728
        }
        virtual_ipaddress {
              10.5.7.2/16
        }
}
```
https://linuxhint.com/pxe_boot_ubuntu_server/
https://linuxhint.com/install_tftp_server_ubuntu/

# Tools.Rye.Ninja

- [PiHole Admin](http://pihole.tools.rye.ninja/admin)
- [Apt Mirror](http://apt.tools.rye.ninja:30081)
- [Netbox](http://netbox.tools.rye.ninja:30082)

# Tools

- [IP Subnet Calculator](https://www.calculator.net/ip-subnet-calculator.html)

# References

## dnsmasq
- [archlinux - dnsmasq - Documentation](https://wiki.archlinux.org/title/Dnsmasq#PXE_setup)


## Pi-Hole

- [Pi-Hole Discourse - Pihole not directing clients to PXE Server](https://discourse.pi-hole.net/t/pihole-not-directing-clients-to-pxe-server/12918)
- [Reddit - How to set up a PXE Boot Option with Pihole?](https://www.reddit.com/r/pihole/comments/kwj2bz/how_to_set_up_a_pxe_boot_option_with_pihole/)
- [Reddit - How to use the Pi-Hole DHCP server for pxe boot?](https://www.reddit.com/r/pihole/comments/5vshcz/how_to_use_the_pihole_dhcp_server_for_pxe_boot/)
- [A Cloud Guru - Implement and Configure a PXE Boot Server](https://learn.acloud.guru/course/91f1ddd0-51e0-4007-b992-21d60983aeaf/learn/ddf43678-74e1-4a79-8e34-51bfb162252e/03def8d6-be7b-4728-a228-39f776bad205/watch)
- [Pi-Hole Documentation - Basic Install](https://docs.pi-hole.net/main/basic-install/)
- [Pi-Hole Discourse - What is setupVars.conf and how do I use it?](https://discourse.pi-hole.net/t/what-is-setupvars-conf-and-how-do-i-use-it/3533)
- [Pi-Hole Discourse - PiHole as DHCP and DNS for multiple VLANs & subnets](https://discourse.pi-hole.net/t/pihole-as-dhcp-and-dns-for-multiple-vlans-subnets/45545/5)
- [Pi-Hole Discourse - Pi-hole as part of a post-installation script](https://discourse.pi-hole.net/t/pi-hole-as-part-of-a-post-installation-script/3523/5)
- [StackExchange - Unix & Linux - Install Pi-Hole without user interaction?](https://unix.stackexchange.com/questions/423715/install-pi-hole-without-user-interaction)

## PXE Boot
- [Danny Payne - Building a PXE Boot Server](https://dannypayne.me/?p=20)
- [LinuxHint - Configuring PXE Network Boot Server on Ubuntu 22.04 LTS](https://linuxhint.com/pxe_boot_ubuntu_server/)
- [ipxe.org - chainloading](https://ipxe.org/howto/chainloading)
- [ipxe.org - download](https://ipxe.org/download)
- [LinuxHint - Ubuntu - Installing and Configuring TFTP Server on Ubuntu](https://linuxhint.com/install_tftp_server_ubuntu/)

## Shoelaces
- [ThousandEyes Engineering - Open-Sourcing Shoelaces: A Tool for Unmanned Server Bootstrapping](https://www.thousandeyes.com/blog/open-sourcing-shoelaces-tool-for-unmanned-server-bootstrapping)
- [Github - thousandeyes - shoelaces](https://github.com/thousandeyes/shoelaces)
- [Github - estenrye - shoelaces](https://github.com/estenrye/shoelaces)

## Netbox
- [Netbox - Installation](https://docs.netbox.dev/en/stable/installation/)
- [YouTube - System MTU One - Netbox Guided Install - v2.9.9 on Ubuntu 20.04.1 LTS](https://www.youtube.com/watch?v=Z5zhIiUKrBI)

## Smallstep
- [step-kms-plugin](https://github.com/smallstep/step-kms-plugin)
- [Step-CA - Documentation - Configuring Postgresql](https://smallstep.com/docs/step-ca/configuration/#postgresql)

## Autheilia
- [Autheilia - LDAP - Introduction](https://www.authelia.com/integration/ldap/introduction/)

## Unifi Controller
- [Github - davecoutts - unify_ubuntu_2004.sh](https://gist.github.com/davecoutts/5ccb403c3d90fcf9c8c4b1ea7616948d)
- [PiMyLifeUp - Installing the UniFi Controller on Ubuntu](https://pimylifeup.com/ubuntu-unifi-controller/)

## HAProxy
- [LinuxHint - Ubuntu - How to install and configure HAproxy on Ubuntu 20.04 LTS](https://linuxhint.com/how-to-install-and-configure-haproxy-load-balancer-in-linux/)

## Apt Repository
- [Medium - Sqooba - Create your own custom and authenticated APT repository](https://medium.com/sqooba/create-your-own-custom-and-authenticated-apt-repository-1e4a4cf0b864)
- [Debian Bug report logs - #322483 - apt-utils: please include an example for apt-ftparchive generate](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=322483)

## VLAN
- [Lifewire - What is a Virtual LAN (VLAN)](https://www.lifewire.com/virtual-local-area-network-817357)

## Keepalived
- [Keepalived - Documentation - Installing keepalived](https://keepalived.readthedocs.io/en/latest/installing_keepalived.html)
- [RedHat - keepalived basics](https://www.redhat.com/sysadmin/keepalived-basics)

## Nitrokey HSM2
- [Nitrokey - HSM2 - Docs](https://docs.nitrokey.com/hsm/linux/)
- [Getting Started with the Nitrokey HSM 2](https://privatekey.dev/getting-started-with-the-nitrokey-hsm-2/)
- [Using the Nitrokey HSM with GPG in macOS](https://blogs.gnome.org/danni/2017/07/07/using-the-nitrokey-hsm-with-gpg-in-macos/)
- [M of N Setup with NitroKey HSM](https://www.ssltrust.com/help/setup-guides/mofn-setup-nitrokey-hsm)
- [Nitrokey Start: Getting started guide (gnuk openpgp token)](https://raymii.org/s/articles/Nitrokey_Start_Getting_started_guide.html)
- [Using OpenPGP subkeys in Debian development](https://wiki.debian.org/Subkeys)

# Ansible Documentation

## Collections

- [ansible.builtin](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html#plugins-in-ansible-builtin)
- [ansible.posix](https://docs.ansible.com/ansible/latest/collections/ansible/posix/index.html#plugins-in-ansible-posix)
- [community.crypto](https://docs.ansible.com/ansible/latest/collections/community/crypto/index.html#plugins-in-community-crypto)
- [community.general](https://docs.ansible.com/ansible/latest/collections/community/general/index.html#plugins-in-community-general)
- [community.postgresql](https://docs.ansible.com/ansible/latest/collections/community/postgresql/index.html)
- [kubernetes.core](https://docs.ansible.com/ansible/latest/collections/kubernetes/core/index.html#plugins-in-kubernetes-core)
- [netbox.netbox](https://docs.ansible.com/ansible/latest/collections/netbox/netbox/index.html#plugins-in-netbox-netbox)

