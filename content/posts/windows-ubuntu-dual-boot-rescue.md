---
title: 'Windows Ubuntu Dual Boot Rescue'
date: 2024-05-19T00:05:52+08:00
draft: true
---

Boot to windows when in grub minimal shell

```
insmod part_gpt
insmod chain
set root=(hd0,gpt1)
chainloader /EFI/Microsoft/Boot/bootmgfw.efi
boot
```

Install windows bootloader

```
bcdboot C:\Windows /s c: /f ALL 
```