---
title: 'Create K8s high availability cluster With HAProxy'
date: 2024-01-19T22:58:26+08:00
draft: false
toc: true
tags: [k8s, kubernetes]
---

## Preparation

To setup a k8s cluster, I will prepare 4 machines with the following network setting:

| hostname | ip | role |
|-|-|-|
| gm-mini | 192.168.31.199 | HAProxy |
| gm-red | 192.168.31.200 | k8s master |
| gm-green | 192.168.31.201 | k8s master |
| gm-blue | 192.168.31.202 | k8s worker |
| gm-orange | 192.168.31.203 | k8s worker |

## 1. create master on gm-red

```shell
sudo kubeadm init \
    --apiserver-advertise-address=192.168.31.200 \
    --image-repository=registry.aliyuncs.com/google_containers \
    --kubernetes-version=v1.29.0 \
    --service-cidr=10.96.0.0/12 \
    --pod-network-cidr=10.244.0.0/16 \
    --cri-socket=unix:///run/containerd/containerd.sock \
    --control-plane-endpoint=192.168.31.199:6443 \
    --upload-certs
```

## 2. Apply CNI network plugin

```shell
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

## 3. Join as control-panel nodes (gm-green)

```shell
sudo kubeadm join 192.168.31.199:6443 \
    --token 7yszg3.su99ir6t8m9o8ttr \
	--discovery-token-ca-cert-hash sha256:xxxx \
	--control-plane \
    --certificate-key yyyyy
```

## 4. Join as worker node (gm-blue, gm-orange)

```shell
sudo kubeadm join 192.168.31.199:6443 \
    --token aaa.bbb \
	--discovery-token-ca-cert-hash sha256:xxxx
```
