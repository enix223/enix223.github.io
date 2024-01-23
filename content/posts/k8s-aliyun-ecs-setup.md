---
title: 'Setup K8s with Aliyun ECS'
date: 2023-10-16T13:47:52+08:00
draft: false
toc: true
tags: [kubernetes, k8s, aliyun, ecs]
---

This article will try to explain how to setup a k8s cluster with three nodes with aliyun ecs.

# 1. Environment

1. Aliyun ECS with Ubuntu 22.04
2. 2 cores + 4G ECS
3. Docker 24.0.6

Server has been install docker + containerd

Because containerd installed with docker, so we can use conatainerd as the runtime of k8s, but we need to enable CRI interface in containerd.

# 2. Init MASTER NODE

1.  add apt source

    ```shell
    apt-get update && apt-get install -y apt-transport-https
    ```

2.  get k8s gpg key

    ```shell
    curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes.gpg
    chmod a+r /etc/apt/keyrings/kubernetes.gpg
    ```

3.  add apt source

    ```shell
    cat > /etc/apt/sources.list.d/kubernetes.list << EOF
    deb [arch="amd64" signed-by=/etc/apt/keyrings/kubernetes.gpg] https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
    EOF
    ```

4.  install kubeadm kubectl

    ```shell
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
    ```

5.  enable cri for containerd

    ```shell
    # must create config.toml with default config
    containerd config default > /etc/containerd/config.toml
    ```

    make sure the following values are set

    ```yaml
    # remove cri from disabled plugins
    disabled_plugins = []

    [plugins]

      [plugins."io.containerd.grpc.v1.cri"]
        sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
    
        [plugins."io.containerd.grpc.v1.cri".containerd]
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
            [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
              [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
                SystemdCgroup = true
    ```

6.  create k8s cluster

    ```shell
    kubeadm init \
        --control-plane-endpoint=sun \
        --apiserver-advertise-address=172.18.165.84 \
        --image-repository registry.aliyuncs.com/google_containers \
        --kubernetes-version v1.28.4 \
        --service-cidr=10.96.0.0/12 \
        --pod-network-cidr=10.244.0.0/16 \
        --cri-socket=unix:///run/containerd/containerd.sock
    ```

    Expected output:

    ```
    To start using your cluster, you need to run the following as a regular user:

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

    Alternatively, if you are the root user, you can run:

    export KUBECONFIG=/etc/kubernetes/admin.conf

    You should now deploy a pod network to the cluster.
    Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
    https://kubernetes.io/docs/concepts/cluster-administration/addons/

    You can now join any number of control-plane nodes by copying certificate authorities
    and service account keys on each node and then running the following as root:

    kubeadm join sun:6443 --token zzzz \
        --discovery-token-ca-cert-hash xxxxx \
        --control-plane 

    Then you can join any number of worker nodes by running the following on each as root:

    kubeadm join sun:6443 --token yyyy \
        --discovery-token-ca-cert-hash xxxxxx
    ```

7. install flannel

    ```shell
    kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
    ```

8. Add firewall rules as below

* [ALL] Flannel traffic: allow `8285/udp`, `8472/udp`
* [ALL] Flannel traffic: allow `8285/udp`, `8472/udp`
* [Control-plane] Kubernetes API server: allow `6443/tcp`
* [Control-plane] etcd server client API: allow `2379-2380/tcp`
* [ALL] Kubelet API: allow `10250/tcp`
* [Control-plane] kube-scheduler: allow `10259/tcp`
* [Control-plane] kube-controller-manager: allow `10257/tcp`
* [ALL] NodePort Services: allow `30000-32767/tcp`

# 3. Init WORKER NODE

REPEAT step 1 ~ 6 in MASTER NODE section

create join token

```shell
kubeadm token create --print-join-command
```

# 4. Troubleshooting

**1. node "\"sun\"" not found when running kubeadm init**

We must change the host name to sun in aliyun console, and restart the instance to take effect.

**2. create container failed validation: container.Runtime.Name must be set: invalid argument"**

Because older docker is installed, and then upgrade to new version, the config.toml is the old version, so we must create the config.toml from `containerd config default`

# 5. References:

* [使用kubeadm部署](https://developer.aliyun.com/article/907981)
* [containerd 1.4.9 Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService](https://serverfault.com/questions/1074008/containerd-1-4-9-unimplemented-desc-unknown-service-runtime-v1alpha2-runtimese)
* [Container Runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
* [[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc] default configuration isn't working](https://github.com/containerd/containerd/issues/6964)