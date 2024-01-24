---
title: 'Kubernetes NFS Volume'
date: 2023-11-04T12:20:22+08:00
draft: false
---

## 1. 环境准备

| Hostname | OS | IP | Note |
|-|-|-|-|
| gm-mini | debian 12 | 192.168.31.199 | KVM host, nfs export path: `/var/lib/nfs` |
| gm-red | Ubuntu 22.04 | 192.168.31.200 |VM machine, k8s control-plane |
| gm-green | Ubuntu 22.04 | 192.168.31.201 |VM machine, k8s control-plane |
| gm-blue | Ubuntu 22.04 | 192.168.31.202 |VM machine, k8s worker |
| gm-orange | Ubuntu 22.04 | 192.168.31.203 |VM machine, k8s worker |

## 2. 安装nfs server

**2.1. 检查兼容性**

安装nfs前，需要检查OS是否支持NFS (是否包含module knfsd.ko)

```shell
grep NFSD /boot/config-`uname -r`
```

输出:

```
CONFIG_NFSD=m
CONFIG_NFSD_V2_ACL=y
CONFIG_NFSD_V3_ACL=y
CONFIG_NFSD_V4=y
CONFIG_NFSD_PNFS=y
CONFIG_NFSD_BLOCKLAYOUT=y
# CONFIG_NFSD_SCSILAYOUT is not set
# CONFIG_NFSD_FLEXFILELAYOUT is not set
# CONFIG_NFSD_V4_2_INTER_SSC is not set
CONFIG_NFSD_V4_SECURITY_LABEL=y
```

从输出可以知道，os支持nfs v4.

**2.2. 安装nfs server**

```shell
sudo apt install nfs-kernel-server
```

## 3. 配置NFS

nfs文件夹地址为`/var/lib/nfs`, 网络地址为`192.168.31.0/24`

```shell
sudo mkdir -p /var/lib/nfs
sudo tee -a /etc/exports > /dev/null << EOF
/var/lib/nfs 192.168.31.0/255.255.255.0(rw,no_root_squash,subtree_check)
EOF
sudo exportfs -a
sudo /etc/init.d/nfs-kernel-server reload
```

配置完成后，还要配置防火墙规则，打开`2049`端口，因为主机使用的是`firewalld`，此处以`firewalld`为例配置防火墙:

```shell
sudo firewall-cmd --permanent \
    --zone=public \
    --add-rich-rule='rule family="ipv4" source address="192.168.31.0/24" port port="2049" protocol="tcp" accept'
sudo firewall-cmd --reload
```

## 4. 安装k8s nfs volume provisioner

登录k8s control plane `gm-red`

**4.1. 安装nfs client**

为了简化重复工作，本文使用`ansible`统一安装`nfs-common`包

gm-install-nfs.yaml

```yaml
---
- name: Install nfs-common
  hosts: gvms
  become: true

  tasks:
    - name: apt update
      apt:
        upgrade: yes
        update_cache: yes

    - name: apt install nfs-common 
      apt:
        pkg:
          - nfs-common
        state: latest
        update_cache: true
```

使用ansible-playboook批量执行安装任务:

```shell
ansible-playbook gm-install-nfs.yaml --ask-become-pass
```

**4.2. Install helm**

本文通过helm chart来安装nfs provisioner，所以安装chart前，需要确保k8s master已安装了helm，若未安装，则通过如下指令安装helm.

```shell
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

**4.3. 安装nfs external subdir provisioner chart**

执行nfs subdir provisioner的helm chart安装

```shell
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm upgrade nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --install \
    --set replicaCount=2 \
    --set storageClass.name=nfs-client \
    --set storageClass.defaultClass=true \
    --set nfs.server=192.168.31.199 \
    --set nfs.path=/var/lib/nfs
```

大功告成

## 5. References

* [NFS Server Setup](https://wiki.debian.org/NFSServerSetup)
* [Getting: bad option; for several filesystems (e.g. nfs, cifs) when trying to mount azure file share in K8 container](https://stackoverflow.com/questions/66145158/getting-bad-option-for-several-filesystems-e-g-nfs-cifs-when-trying-to-mou)
* [How to Set Up an NFS Mount on Debian 11](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-debian-11)
* [Open a Port or Service](https://firewalld.org/documentation/howto/open-a-port-or-service.html)