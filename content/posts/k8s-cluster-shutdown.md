---
title: 'K8s Cluster Shutdown'
date: 2024-01-17T09:00:39+08:00
draft: true
---

## Shutdown worker nodes

1. stop pod scheduling and drain existing pods on node

```shell
kubectl cordon <worker node name>
kubectl drain <worker node name> --ignore-daemonsets --delete-emptydir-data
```

2. stop kubelet

```shell
systemctl stop kubelet
```

3. stop docker

```shell
systemctl stop docker
```

4. shutdown

```shell
shutdown now
```