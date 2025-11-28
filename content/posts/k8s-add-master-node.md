---
title: 'K8s Add Master Node'
date: 2024-06-22T11:33:42+08:00
draft: true
---

# 扩展k8s集群的control plane节点

1. 安装docker

    ```shell
    ansible-playbook install-docker.yml
    ```

2. 安装k8s

    ```shell
    ansible-playbook install-k8s.yml
    ```

3. 安装nerdctl

    ```shell
    ansible-playbook install-nerdctl.yml
    ```

4. 安装nfs common

    ```shell
    ansible-playbook install-nfs-common.yml
    ```

5. 安装ossfs

    ```shell
    ansible-playbook install-ossfs.yml
    ```

6. 更新hosts

    ```shell
    ansible-playbook install-setup-internal-dns.yml
    ```

7. 增加ishare用户

    ```shell
    ansible-playbook add-user-ishare.yml
    ```

8. 重新上传certificate (master主机)

    ```shell
    kubeadm init phase upload-certs --upload-certs
    ```

9.  获取join command (master主机)

    ```shell
    kubeadm token create --print-join-command
    ```

10. 加入集群 (新主机)

    ```shell
    $JOIN_COMMAND_FROM_STEP2 --control-plane --certificate-key $KEY_FROM_STEP1
    ```