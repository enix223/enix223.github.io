---
title: 'Redmi 9a Install Lineage 20'
date: 2023-11-23T11:31:56+08:00
draft: true
---

## Unlock Bootloader

## Install TWRP img

1. Download [TWRP img](https://mifirm.net/model/dandelion.ttt#twrp)

2. Connect phone with usb, and enable developer mode, and check with adb:

```shell
adb devices
```

3. Reboot to bootloader

```shell
adb reboot bootloader
```

4. Flash fastboot

```shell
fastboot flash recovery recovery-TWRP-3.4.2B-0216-REDMI9A-CN-wzsx150.img
```

For A/B type partitions device

```shell
fastboot boot recovery-TWRP-3.4.2B-0216-REDMI9A-CN-wzsx150.img
```

1. Enter recovery

```shell
fastboot reboot recovery
```

## Download Lineage 20 OS
