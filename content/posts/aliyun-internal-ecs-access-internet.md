---
title: '阿里云内网ECS访问外网的方法'
date: 2024-03-23T14:29:39+08:00
draft: true
---

## 配置内网转发

1 .开启内网转发

target: root@hs-red

```sh
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```

2. 配置IPTABLES NAT

放行`172.16.0.0/12`的主机，转发至`172.22.133.246`

```sh
iptables -t filter -A FORWARD -j ACCEPT
iptables -t nat -A POSTROUTING -s 172.16.0.0/12 -j MASQUERADE
```

1. 查看规则

```sh
iptables -t nat -L
```

4. 打开阿里云后台

打开专有网络---路由表---实例名称---添加路由条目