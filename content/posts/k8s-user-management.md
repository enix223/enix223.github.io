---
title: 'K8s User Management'
date: 2023-11-21T11:44:55+08:00
draft: true
---

## Create user CSR

```shell
openssl genrsa -out ishare.key 2048
openssl req -new -key ishare.key -out ishare.csr
```

## Approve CSR

```shell
openssl x509 -req -in ishare.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out ishare.crt -days 500
```

## Create role

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: ishare
  name: ishare-admin
rules:
  - apiGroups: ["", "extensions", "apps"]
    resources:
      - "deployments"
      - "pods"
      - "services"
      - "statefulsets"
      - "secret"
      - "configmap"
      - "persistentvolumes"
      - "persistentvolumeclaims"
    verbs:
      - "get"
      - "list"
      - "watch"
      - "create"
      - "update"
      - "patch"
      - "delete"
  - apiGroups: ["storage.k8s.io"]
    resources:
      - "storageclasses"
    verbs:
      - "get"
      - "list"
      - "watch"
```

## Create role binding

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: ishare-rolebinding
  namespace: ishare
subjects:
  - kind: User
    name: ishare
    apiGroup: ""
roleRef:
  kind: Role
  name: ishare-admin
  apiGroup: ""
```

## Create .kube/config

Login to the user to authorized

```shell
kubectl config set-cluster kubernetes --server=https://192.168.1.185:6443 --embed-certs --certificate-authority=/etc/kubernetes/pki/ca.crt
kubectl config set-credentials ishare --client-certificate=~/ishare.crt --client-key=~/ishare.key
kubectl config set-context ishare-context --cluster=kubernetes --namespace=ishare --user=ishare
```

References:

* [How to create user in Kubernetes cluster and give it access?](https://discuss.kubernetes.io/t/how-to-create-user-in-kubernetes-cluster-and-give-it-access/9101/3)