---
title: 'K8s change control plane endpoint address'
date: 2024-03-22T15:02:04+08:00
draft: true
---

1. Bakcup

```sh
systemctl stop kubelet docker containerd
```

```sh
mv /etc/kubernetes /root/kuberntes-backup
mv /var/lib/kubelet /root/kubelet-backup
mv /var/lib/etcd /root/etcd-backup
```

2. start kubelet

```sh
systemctl start kubelet docker containerd
```

3. Reset kubernetes

```sh
kubeadm reset
```

4. Restore pki and etcd

```sh
mkdir -p /etc/kubernetes
cp -r /root/kuberntes-backup/pki /etc/kubernetes
cp -r /root/etcd-backup /var/lib/etcd
```

5. Init with new control plane endpoint

```sh
kubeadm init \
    --control-plane-endpoint "k8s.hause.com.cn:6443" \
    --apiserver-advertise-address=172.22.133.246 \
    --image-repository registry.aliyuncs.com/google_containers \
    --kubernetes-version v1.29.2 \
    --service-cidr=10.96.0.0/12 \
    --pod-network-cidr=10.244.0.0/16 \
    --cri-socket=unix:///run/containerd/containerd.sock \
    --upload-certs \
    --ignore-preflight-errors=DirAvailable--var-lib-etcd
```

## Reference:

https://ystatit.medium.com/how-to-change-kubernetes-kube-apiserver-ip-address-402d6ddb8aa2