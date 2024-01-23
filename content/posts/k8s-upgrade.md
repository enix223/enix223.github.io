---
title: 'Kubernetes cluster upgrade'
date: 2023-11-18T09:12:10+08:00
draft: true
---

## Upgrade workflow

## Upgrade kubeadm

### Update GPG key in every nodes

```sh
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.27/deb/Release.key | gpg --dearmor --yes -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### Update k8s source list

```shell
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.27/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### Check kubeadm version

```shell
apt update
apt-cache madison kubeadm
```

### Upgrade kubeadm in control plane

```shell
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm='1.27.8-1.1' && \
apt-mark hold kubeadm
```

### Check upgrade plan

```shell
kubeadm upgrade plan
```

### Apply upgrade plan

```shell
kubeadm upgrade apply v1.27.8-1.1
```

### Drain the node

```shell
kubectl drain <node-to-drain> --ignore-daemonsets
```

### Upgrade kubelet & kubectl in node

```shell
apt-mark unhold kubelet kubectl && apt-get update && \
apt-get install -y kubelet='1.27.8-1.1' kubectl='1.27.8-1.1' && \
apt-mark hold kubelet kubectl
```

### Restart daemon

```shell
systemctl daemon-reload
systemctl restart kubelet
```

### Uncordon the node

```shell
kubectl uncordon <node-to-uncordon>
```

## Reference

* [Upgrading kubeadm clusters](https://v1-27.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)