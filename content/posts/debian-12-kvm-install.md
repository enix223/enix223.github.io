---
title: 'Install KVM on Debian 12'
date: 2023-11-02T11:59:04+08:00
draft: false
toc: true
tags: [debian, kvm]
---

This article is going to elaborate the whole process to install KVM on Debian 12, with Bridge network. 

With the power of bridge network, each VM will have its own LAN IP, and we can connect to the VM directly from other hosts in the same LAN.

## Install dependency packages

### Install QEMU deps

* Install qemu with GUI

    ```shell
    sudo apt install qemu-system libvirt-daemon-system virt-manager aqemu
    ```

* Install qemu without GUI

    ```shell
    sudo apt install --no-install-recommends qemu-system libvirt-clients libvirt-daemon-system virtinst
    ```

### Install bridge network deps

```shell
sudo apt install dnsmasq-base bridge-utils firewalld
```

## Create bridge network

KVM use NAT as the default network type, we need to create the bridge network first, and create the VM with the bridge network.

### Install deps

```shell
sudo apt-get install bridge-utils
```

### Create br0 interface

The bridge network `br0` will assign a static IP, and bridged to the ethernet port `enp1s0f0`. Here are the config files:

* `/etc/network/interfaces.d/br0`

    ```
    auto br0
    iface br0 inet static
        address 192.168.1.180
        broadcast 192.168.1.255
        netmask 255.255.254.0
        gateway 192.168.0.1
        dns-nameservers 192.168.0.1 192.168.1.180
        bridge_ports enp1s0f0
        bridge_stp off
        bridge_waitport 0
        bridge_fd 0
        bridge_maxwait 0
    ```

* `/etc/network/interfaces.d/enp1s0f0`

    ```
    iface enp1s0f0 inet manual
    ```

after creating the above files, we need to restart the networking service

```shell
systemctl restart networking
```

### Disable net filter

create file `/etc/sysctl.d/99-netfilter-bridge.conf`

```
net.bridge.bridge-nf-call-ip6tables = 0
net.bridge.bridge-nf-call-iptables = 0
net.bridge.bridge-nf-call-arptables = 0
```

load `br_netfilter` module

```shell
sudo modprobe br_netfilter
```

load br netfilter at boot with `/etc/modules-load.d/br_netfilter.conf`

```
br_netfilter
```

load netfilter setting:

```shell
sysctl -p /etc/sysctl.d/99-netfilter-bridge.conf
```

### Create virtual network

create file in any place: `bridged-network.xml`

```xml
<network>
    <name>bridged-network</name>
    <forward mode="bridge" />
    <bridge name="br0" />
</network>
```

create virtual network with above xml

```shell
sudo virsh net-define bridged-network.xml
sudo virsh net-start bridged-network
sudo virsh net-autostart bridged-network
```

check network created or not

```shell
sudo virsh net-list
```

### Create VM with bridge network

```shell
sudo virt-install \
  --vcpus=1 \
  --memory=1024 \
  --cdrom=debian-12-amd64-DVD-1.iso \
  --disk size=7 \
  --os-variant=debian12 \
  --network network=bridged-network
```

## References

* [KVM - debian](https://wiki.debian.org/KVM)
* [How to use bridged networking with libvirt and KVM](https://linuxconfig.org/how-to-use-bridged-networking-with-libvirt-and-kvm)
* [Install and Configure Firewalld on Debian 10/11](https://computingforgeeks.com/how-to-install-and-configure-firewalld-on-debian/)
