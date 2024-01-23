---
title: 'K8s Networking Troubleshooting'
date: 2023-11-04T12:20:37+08:00
draft: true
---

UFW 

If using flannel as the network plugin, please be sure add a rule to allow traffic to `8285/udp`, and `8472/udp`.

* [UFW Essentials: Common Firewall Rules and Commands](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands#allow-incoming-rsync-from-specific-ip-address-or-subnet)
* [Ports and Protocols](https://kubernetes.io/docs/reference/networking/ports-and-protocols/)
* [Debugging DNS Resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)
* [How to delete a UFW firewall rule on Ubuntu / Debian Linux](https://www.cyberciti.biz/faq/how-to-delete-a-ufw-firewall-rule-on-ubuntu-debian-linux/)
* [Debuging service](https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/)
* [Flannel troubleshooting](https://github.com/flannel-io/flannel/blob/master/Documentation/troubleshooting.md)