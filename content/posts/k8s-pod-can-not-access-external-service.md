---
title: '解决Kubernetes Pod无法访问外部网络'
date: 2024-01-23T21:56:28+08:00
draft: false
tags: [k8s, kubernetes, troubleshooting, coredns]
---

## 问题与排查

目前业务使用K8S中部署java的微服务，但在使用过程中，发现某个kubernetes的pod无法访问`api.weixin.qq.com`，尝试通过`nslookup`, `nc`来测试pod是否可以连通服务

```shell
kubectl exec -it dnsutils -- bash
```

在pod中运行:

```shell
nc -z api.weixin.qq.com 443 && echo ok
```

返回结果

```
nc: bad address 'api.weixin.qq.com'
```

但是尝试其他域名，发现可以解析

```shell
nc -z www.baidu.com 443 && echo ok
```

结果却是`ok`。

## 开启CoreDNS的日志记录功能

dns选择性的出现解析异常，怀疑是k8s的CoreDNS出现问题，导致无法解析域名，然后尝试打开CoreDNS的log日志功能

```shell
kubectl -n kube-system edit configmap coredns
```

增加`log`

```yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        log
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  creationTimestamp: "2023-12-30T06:52:54Z"
  name: coredns
  namespace: kube-system
  resourceVersion: "5340888"
  uid: xxxxx
```

保存退出，等待1~2分钟后，等待CoreDNS apply新的配置，查看CoreDNS的日志

```shell
kubectl logs --tail 10 -f -l k8s-app=kube-dns -n kube-system
```

在pod中执行

```shell
nc -z api.weixin.qq.com 443 && echo ok
```

日志如下：
```
[INFO] 10.244.6.133:33772 - 41051 "A IN api.weixin.qq.com.ishare.svc.cluster.local. udp 60 false 512" NXDOMAIN qr,aa,rd 153 0.00048101s
[INFO] 10.244.6.133:33772 - 42091 "AAAA IN api.weixin.qq.com.ishare.svc.cluster.local. udp 60 false 512" NXDOMAIN qr,aa,rd 153 0.000642553s
[INFO] 10.244.6.133:60047 - 41449 "A IN api.weixin.qq.com.svc.cluster.local. udp 53 false 512" NXDOMAIN qr,aa,rd 146 0.00027336s
[INFO] 10.244.6.133:60047 - 42206 "AAAA IN api.weixin.qq.com.svc.cluster.local. udp 53 false 512" NXDOMAIN qr,aa,rd 146 0.000377254s
[INFO] 10.244.6.133:36078 - 29981 "AAAA IN api.weixin.qq.com.cluster.local. udp 49 false 512" NXDOMAIN qr,aa,rd 142 0.000526157s
[INFO] 10.244.6.133:36078 - 29064 "A IN api.weixin.qq.com.cluster.local. udp 49 false 512" NXDOMAIN qr,aa,rd 142 0.000724731s
[INFO] 10.244.6.133:35114 - 13003 "A IN api.weixin.qq.com. udp 35 false 512" NOERROR qr,rd 167 0.002365088s
[INFO] 10.244.6.133:35114 - 14063 "AAAA IN api.weixin.qq.com. udp 35 false 512" SERVFAIL qr,rd 35 2.013423971s
[INFO] 10.244.6.133:35114 - 14063 "AAAA IN api.weixin.qq.com. udp 35 false 512" SERVFAIL qr,aa,rd 35 0.000137506s
[INFO] 10.244.6.133:35114 - 14063 "AAAA IN api.weixin.qq.com. udp 35 false 512" SERVFAIL qr,aa,rd 35 0.000169814s
[INFO] 10.244.6.133:35114 - 14063 "AAAA IN api.weixin.qq.com. udp 35 false 512" SERVFAIL qr,aa,rd 35 0.000196277s
[INFO] 10.244.6.133:35114 - 14063 "AAAA IN api.weixin.qq.com. udp 35 false 512" SERVFAIL qr,aa,rd 35 0.000164776s
[INFO] 10.244.6.133:35114 - 14063 "AAAA IN api.weixin.qq.com. udp 35 false 512" SERVFAIL qr,aa,rd 35 0.000214178s
[INFO] 10.244.6.133:35114 - 14063 "AAAA IN api.weixin.qq.com. udp 35 false 512" SERVFAIL qr,aa,rd 35 0.000153283s
[INFO] 10.244.6.133:35114 - 14063 "AAAA IN api.weixin.qq.com. udp 35 false 512" SERVFAIL qr,aa,rd 35 0.000212498s
[INFO] 10.244.6.133:35114 - 14063 "AAAA IN api.weixin.qq.com. udp 35 false 512" SERVFAIL qr,aa,rd 35 0.000164354s
[INFO] 10.244.6.133:35114 - 14063 "AAAA IN api.weixin.qq.com. udp 35 false 512" SERVFAIL qr,aa,rd 35 0.000188767s
```

发现`api.weixin.qq.com`的解析结果都是`SERVFAIL`，进一步怀疑是否是worker node的host无法访问`api.weixin.qq.com`呢。

远程进入pod所在的host，执行

```shell
nc -z api.weixin.qq.com 443 && echo ok
```

结果是`ok`，使用`nslookup`查询，却发现了`SERVFAIL`

```shell
root@cs-red:~# nslookup api.weixin.qq.com
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	api.weixin.qq.com
Address: 121.14.23.85
Name:	api.weixin.qq.com
Address: 183.2.143.222
Name:	api.weixin.qq.com
Address: 119.147.6.203
Name:	api.weixin.qq.com
Address: 119.147.6.237
** server can't find api.weixin.qq.com: SERVFAIL
```

## 使用google dns

综合以上指令的运行结果，怀疑试试dns解析的问题呢，使用`8.8.8.8`再尝试解析`api.weixin.qq.com`

```shell
root@cs-red:~# nslookup api.weixin.qq.com 8.8.8.8
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
Name:	api.weixin.qq.com
Address: 119.147.6.203
Name:	api.weixin.qq.com
Address: 121.14.23.85
Name:	api.weixin.qq.com
Address: 119.147.6.237
Name:	api.weixin.qq.com
Address: 183.2.143.222
```

结果没有出现`SERVFAIL`。

## CoreDNS配置forward指令

问题定位在dns的解析上，尝试修改CoreDNS的解析功能，增加额外的dns服务器`8.8.8.8`

```yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        log
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf 8.8.8.8 {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  creationTimestamp: "2023-12-30T06:52:54Z"
  name: coredns
  namespace: kube-system
  resourceVersion: "5340888"
  uid: xxxxx
```

保存退出，并等待1~2分钟后，重新进入pod，执行

```
ishare$ nc -z api.weixin.qq.com 443 && echo ok
ok
```

问题终于得到解决。

## 总结

K8S中CoreDNS负责Pod中service的域名解析服务，如果发现pod解析域名出错，不管是外部域名还是k8s内部服务的域名，都需要通过CoreDNS来排查问题，首先最直接的就是打开CoreDNS的日志功能`log`，在排查过程中，仔细查看日志内容，并发现到底是哪个环节出现异常，进而想办法解决问题。

## References

* [Resolving external domains from within pods does not work](https://stackoverflow.com/questions/62664701/resolving-external-domains-from-within-pods-does-not-work)
* [Can't resolve dns from inside k8s pod](https://stackoverflow.com/questions/63111346/cant-resolve-dns-from-inside-k8s-pod)
* [CoreDNS forward](https://coredns.io/plugins/forward/)
* [Debugging DNS resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)