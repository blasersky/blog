---
layout: post
title:  "how to set up lxc in proxmox on raspi with static ip"
tags: proxmox raspi
---
performing the usual steps to create an lxc container in proxmox with static ip fails on raspberry pi with the following error message
```
unable to open file '/etc/network/interfaces.tmp.3239' - No such file or directory
```
there is a workaround, however. first, make sure that `ifupdown2` is installed. when creating the lxc, perform all the steps normally up until the network setup. in the network tab, choose *static* but leave ip and gateway fields blank. on the proxmox host, edit the `/etc/network/interfaces` file. it should look like this:
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet manual

auto vmbr0
iface vmbr0 inet manual
        address 192.168.0.137
        gateway 192.168.0.1
        dns-nameservers 1.1.1.1 1.0.0.1
        netmask 255.255.255.0
        bridge-ports eth0
        bridge-stp off
        bridge-fd 0
```
in this setup, `192.168.0.137` is the *eye pee a dress* of the proxmox host, and `192.168.0.1` is the router. start the container. in the container shell, execute the following commands:
```
ifconfig eth0 192.168.0.138 netmask 255.255.255.0 up
ip route add default via 192.168.0.1 dev eth0
```
we assume that we want to have `192.168.0.138` as the container *eye pee a dress*. verify settings by issuing `ip a` and `ip r`. ping the router, the proxmox host and an external ip. if everything works fine, update the system and install `ifupdown2`. now edit the `/etc/network/interfaces` file in the lxc, it should look like this:
```
auto lo
iface lo inet loopback
iface lo inet6 loopback

auto eth0
iface eth0 inet static
    address 192.168.0.138/24
    gateway 192.168.0.1
```
finally, in the proxmox gui assign the `192.168.0.138` as the ip of the container and `192.168.0.1` as its gateway. make sure to reserve the `192.168.0.138` in router's dhcp.

further reading: [https://github.com/pimox/pimox7/issues/160#issuecomment-2966730039](https://github.com/pimox/pimox7/issues/160#issuecomment-2966730039)