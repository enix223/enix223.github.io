---
title: 'Notes of "Kubernetes in Actions" - Volumes'
date: 2023-10-15T13:09:46+08:00
draft: false
toc: true
tags: [k8s, kubernetes, kubernetes-in-action]
---

Volume 是依赖于Pod而存在的对象，不能单独被创建，pod的volume对pod的所有容器都可见。

## Volume简介

### Volume类型

* **emptyDir** 用于存储透明的数据（随着pod被移除而移除）
* **hostPath** 使用宿主机的文件系统作为volume
* **gitRepo** 使用git仓库作为volume
* **nfs** 使用NFS文件系统
* **gcePersistentDisk/awsElasticBlockStore/azureDisk** 使用GCE/AWS/Azure的磁盘
* **cinder/cephfs/iscsi** 使用网络存储系统
* **configMap/secret/downwardAPI** 使用k8s的特殊对象作为volume
* **persistentVolumeClaim** 动态创建文件系统

## 使用Volumes在容器间共享数据

### `emptyDir` volume

### `gitRepo` volume

### `hostPath` volume

### pod与底层存储的解耦

#### `PersistentVolumes`和`PersistentVolumeClaims`

#### 创建PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity:
    storage: 1Gi
  accessMode:
  # 一个客户端读写
  - ReadWriteOnce
  # 多个客户端只读
  - ReadOnlyMany
  # PVC被删除后，使用Retain保留PV的策略
  persistentVolumeReclaimPolicy: Retain
  # 底层使用GCE Disk
  gcePersistentDisk:
    pdName: mongodb
    fsType: ext4
```

#### 通过PVC动态声明创建PV

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  resources:
    requests:
      storage: 1Gi
    accessModes:
    # 只允许一个客户端读写
    - ReadWriteOnce
    storageClassName: ""
```

#### 在Pod中使用PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
  volumes:
  - name: mongodb-data
    persistentVolumeClaim:
      className: mongodb-pvc
```

#### PV回收

PV的状态分为:

* `Released`
* `Available`

PV的状态为`Release`时，只能手动回收PV。

PV回收策略:

* `Retain` PVC回收后，保留PV
* `Delete` PVC回收后，删除PV

### 动态创建PV

k8s通过`StorageClass`对象来让用户决定怎么创建PV。`StorageClass`不是namespace隔离的，而是k8s全局存在的对象。

#### 定义StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  zone: europe-west1-b
```

#### 在PVC中引用SC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  resources:
    requests:
      storage: 1Gi
    accessModes:
    - ReadWriteOnce
    storageClassName: "fast"
```

#### 无需指定SC，动态创建PV

查看storage class

```shell
kubectl get sc
```

标记为(default)的SC，当我们定义PVC时，不指定storageClassName时，就会使用默认的SC。

#### 强制PVC绑定之前创建的PV

```yaml
kind: PersistentVolumeClaim
spec:
  # 空的SC时，告诉k8s需要绑定之前创建的PV
  storageClassName: ""
```
