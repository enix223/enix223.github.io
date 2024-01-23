---
title: 'Notes of "Kubernetes in Actions" - Service'
date: 2023-11-01T13:09:46+08:00
draft: false
toc: true
tags: [k8s, kubernetes, kubernetes-in-action]
---

container如果需要外界访问或被其他container访问，则需要借助Kubernetes Service来完成转发。

## 创建Service

`pod-kubia.yaml`

```yaml
apiVerison: v1
kind: Pod
spec:
  containers:
  - name: kubia
    ports:
    - name: http
      containerPort: 8080
    - name: https
      containerPort: 8443
```

`service-kubia.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  ports:
  # service listen on port 80, and forward request to container's port 8080
  - name: http
    port: 80
    targetPort: http
  # service listen on port 8000, and forward request to container's port 8080
  - port: 8000
    targetPort: 8080
  # service listen on port 443, and forward request to container's port 8083
  - port: 443
    targetPort: 8443
  selector:
    app: kubia
```

```shell
kubectl apply -f pod-kubia.yaml
kubectl apply -f service-kubia.yaml
```

## 查看Service

```shell
kubectl get svc
```

## 设置sessionAffinity亲和属性

如果需要让来自同一个客户IP的请求，访问同一个container，可以设置亲和属性为`ClientIP`。

```yaml
apiVersion: v1
kind: Service
spec:
  sessionAffinity: ClientIP
```

## 发现服务

### 通过环境变量发现

在关联了服务的Pod中，运行`env`，可以看到关联的服务IP和端口

```shell
kubectl exec kubia-3inly env
```

关联的服务信息是以`_SERVICE_HOST`和`_SERVICE_PORT`为结尾的环境变量。本例子是

* KUBIA_SERVICE_HOST
* KUBIA_SERVICE_PORT

### 通过DNS发现服务

container绑定了service后，k8s会把service的FQDN添加到container的`/etc/resolv.conf`文件中。

服务的FQDN格式:

```
<service-name>.default.svc.cluster.local
```

Pod可以通过以下三种方式找到service:

* `<service-name>.default.svc.cluster.local`
* `<service-name>.default`
* `<service-name>`

例如本例service name = kubia：

* `kubia.default.svc.cluster.local`
* `kubia.default`
* `kubia`

## 代理外部服务

### 服务endpoint

获取服务的endpoint

```shell
kubectl get endpoints kubia
```

### 手动指定service的endpoint

1) 创建服务时，不指定selector

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  ports:
  - port: 80
```

2) 创建endpoint对象

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  # endpoint对象的名称必须和步骤1创建的服务名称一致!!
  name: external-service
subsets:
  # service forward的IP地址
  - addresses:
    - ip: 11.11.11.11
    - ip: 22.22.22.22
    ports:
    - port: 80
```

### 创建外部服务的别名

创建`api.example.com`的别名服务

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  type: ExternalName
  # 外部服务的FQDN
  externalName: api.example.com
  ports:
  - port: 80
```

### Service对集群外提供服务

#### 通过`NodePort`对外提供服务

在每个noe节点中监听NodePort端口对外提供服务，NodePort的端口范围必须为30000-32767，可以通过kube-apiserver时通过--service-node-port-range参数来设定。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  # 使用NodePort作为对外提供服务的方式
  type: NodePort
  ports:
  # 80是集群内部提供服务的端口
  - port: 80
    targetPort: http
    # 每个node节点都会预留30123端口作为kubia对外提供服务的端口
    nodePort: 30123
  selector:
    app: kubia
```

#### 通过`LoadBalancer`对外提供服务

如果k8s集群运行于支持load balancer的云服务商，则可以使用服务商提供的LB。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  # 使用LoadBalancer作为对外提供服务的方式
  type: LoadBalancer
  ports:
  # 80是集群内部提供服务的端口
  - port: 80
    targetPort: http
  selector:
    app: kubia
```

#### 通过`Ingress`对外提供服务

使用LB的缺点是，每个服务都需要一个LB，且每个LB都需要一个公共IP，而Ingress只需要一个，通过不同的路由规则路由请求到不同的Service。

Ingress对象需要配合Ingress controller一起使用。

##### 创建Ingress对象

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubia
spec:
  rules:
  - host: kubia.example.com
    http:
      paths:
      - path: /kubia
        backend:
          serviceName: kubia
          servicePort: 80
      - path: /foo
        backend:
          serviceName: foobar
          servicePort: 80
```

##### 获取Ingress对象

```shell
kubectl get ingresses
```

### 感知Pod的就绪状态

Pod启动或初始化可能需要时间，而Service只会把请求路由到可以接受服务的pod，以免出现pod无法处理请求的情况。

#### readiness probes

readiness probe会定时执行，检查pod是否可以接受客户端的请求。

readiness probe的类型:

* *Exec*, 执行脚本，通过exit status code来判断是否healthy
* *HTTP GET*, 发起http get请求，通过status code来判断是否healthy
* *TCP Socket*, 发起tcp连接，如果连接成功，则认为healthy

跟liveness probe不一样，readiness probe如果失败了，不会杀掉对应的pod，只会断开pod与service的关联。

#### 在Pod template中添加readiness probe

```yaml
spec:
  template:
    spec:
      containers:
      - name: kubia
        image: luksa/kubia
        readinessProbe:
          exec:
            command:
            - ls
            - /var/ready
```

### Headless Service

Headless service不会分配ClusterIP，访问service的时候，会直接连接对应的Pod。

#### 创建headless service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia-headless
spec:
  # headless
  clusterIP: None
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: kubia
```

无需yaml启动一个pod

```shell
kubectl run dnsutils --image=tutum/dnsutils --command -- sleep infinity
```

获取headless service的DNS A record

```shell
kubectl exec dnsutils nslookup kubia-headless
```
