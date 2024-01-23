---
title: 'Notes of "Kubernetes in Actions" - Statefulset'
date: 2023-10-16T14:09:31+08:00
draft: false
toc: true
tags: [k8s, kubernetes, kubernetes-in-action]
---

## StatefulSet介绍

StatefulSet特点:

* 每个Pod拥有一个唯一确定的身份标识
* StatefulSet确保不会有两个同样标识的pod存在(at-most-one)
* StatefulSet需要每个Pod都创建一个headless Service，用于给pod提供DNS解析，hostname格式为: `<pod-name>.<service-name>.default.svc.cluster.local`

### Scaling StatefulSet

#### Scaling down

每次减少StatefulSet的replica数量时，都可以预知哪个pod被减少，例如SS有3个replica，分别为：`pod-0`, `pod-1`, `pod-2`，如果replica减少为2，则首先会删除`pod-2`；如果replica减少为1，则继续会删除`pod-1`。

#### Scaling up

扩容依次增加的pod的名称的尾号数字会一次递增。

### 关联特定的Storage

每个Pod关联一个PVC，PVC的名称也是可预知的，例如Pod名称为`pod-0`，则PVC的名称为`pvc-0`，pvc的suffix跟pod的suffix一致。

当用户scale down时，StatefulSet只会删除对应的pod，而不会删除pvc，删除动作需要由用户自行执行。

当用户再次执行scale up时，新的Pod又会关联之前的PVC，实现重用PVC和PV。

## 使用StatefulSet

### 创建service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  clusterIP: None
  selector:
    app: kubia
  ports:
  - name: http
    port: 80
```

### 创建StatefulSet

```yaml
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: kubia
spec:
  # use kubia service for the pod
  serviceName: kubia
  template:
    # pod created by this statefulset will have label app=kubia
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia-pet
        ports:
        - name: http
          containerPort: 8080
        volumeMounts:
        - name: data
          mountPath: /var/data
  # PVC create with this template
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      resources:
        requests:
          storage: 1Mi
      accessModes:
      - ReadWriteOnce
```

### 访问StatefulSet的pod

k8s允许我们通过API server访问pod，但是每个请求都必须附带一个`authorization token`。

```
<apiServerHost>:<port>/api/v1/namespaces/<namespace>/pods/<pod-name>/proxy/<path>
```

为了方便开发，k8s提供一个proxy，方便我们直接访问pod

启动proxy （默认端口为8001）

```shell
kubectl proxy
```

现在我们可以直接通过proxy访问pod

```shell
curl http://localhost:8001/api/v1/namespaces/default/pods/kubia-0/proxy/
```

### 通过NonHeadless Service暴露Stateful pods

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia-public
spec:
  selector:
    app: kubia
  ports:
  - port: 80
    targetPort: 8080
```

通过api server访问service

```
<apiServerHost>:<port>/api/v1/namespaces/<namespace>/services/<service-name>/proxy/<path>
```

## StatefulSet中互相发现节点

在StatefulSet中，节点如何其他节点，k8s提供传统的DNS解析方式，也提供一种名为SRV记录的发现方式。

### SRV记录

SRV记录是用于查找服务service的hostname和port的记录。

查询srv记录

```shell
kubectl run -it srvlookup --image=tutum/dnsutils --rm --restart=Never -- dig SRV kubia.default.svc.cluster.local
```

在pod的程序中，我们可以通过查询service的srv记录，发现其他peer节点。service的FQDN格式为:

```
<service>.default.svc.cluster.local
```

### 更新statefulset

修改statefulset的manifest yaml，并重新应用该yaml （例如修改image为新的版本）

删除旧的版本

```shell
kubectl delete po kubia-0 kubia-1
```

k8s 1.7版本及以后，StatefulSet支持Deployment的roll out策略 `spec.updateStrategy`

### StatefulSet处理node节点异常

当k8s node节点不可访问后，k8s本身无法得知节点的状态，但是StatefulSet本身需要确保集群中不能重现两个重名的pod，所以需要由管理员告诉k8s，该如何处理。

#### 手工删除unknown状态的pod

当node断开与master的链接后，pod的状态变为`unknown`，我们需要手工删除该pod

```shell
kubectl delete kubia-0
```

但执行命令后，该pod并没有被立刻删除。因为API server在等待node节点通知，该pod确实已被删除。但是node已经失联，所以该pod一直在等待删除状态。

#### 强制删除pod

```shell
kubectl delete po kubia-0 --force --grace-period 0
```

当执行完强制删除后，新的pod就会被创建出来。
