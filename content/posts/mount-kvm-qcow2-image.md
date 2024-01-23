---
title: 'Mount Kvm Qcow2 Image'
date: 2024-01-16T23:22:47+08:00
draft: true
---

1. load the NBD kernel module

```shell
sudo modprobe nbd max_part=8
```

2. connect qcow2 image to nbd device

```shell
sudo qemu-nbd --connect=/dev/nbd0 ~/example.qcow2
```

## Qcow2 with lvm

3. if your vm is installed with lvm, then use lvdisplay to show the volume

```shell
lvdisplay
```

4. Mount lv

```shell
sudo mkdir /mnt/qcow2
sudo mount /dev/ubuntu-vg/ubuntu-lv /mnt/qcow2
```

## Qcow2 with raw volume

```shell
fdisk -l /dev/nbd0
```

```shell
sudo mkdir /mnt/qcow2
sudo mount /dev/nbd0p1 /mnt/qcow2
```