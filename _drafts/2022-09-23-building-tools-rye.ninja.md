
# Hardware

Here's a list of the hardware I used.

- Supermicro 1U Server
- 4 GB RAM

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


# Next Steps
- Install apt mirror
- Mirror Ubuntu Releases
- Install pihole
- Configure Shoelaces
- Configure pihole DHCP settings
- Install Step Certificates
- Install Netbox
- Install HAProxy?
- Install Cert-manager
- Install Unifi Controller?


https://www.youtube.com/watch?v=Z5zhIiUKrBI
https://docs.nitrokey.com/hsm/linux/
