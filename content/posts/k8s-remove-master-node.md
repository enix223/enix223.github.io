---
title: 'K8s Remove Master Node'
date: 2024-06-22T11:33:47+08:00
draft: true
---

1. Drain the nodes

    ```shell
    kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
    ```

2. Get members in etcd cluster

    ```shell
    ETCDCTL_API=3 etcdctl member list --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key
    ```

3. Remove member from etcd cluster

    ```shell
    ETCDCTL_API=3 etcdctl member remove <member-ID> --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key
    ```

4. Remove the master node

    ```shell
    kubectl delete node <node-name>
    ```
