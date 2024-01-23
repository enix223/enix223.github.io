---
title: 'K8s Create Pvc Pv Manually'
date: 2023-11-18T15:16:18+08:00
draft: true
---

## PV

```yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: PersistentVolume
  metadata:
    annotations:
      pv.kubernetes.io/provisioned-by: k8s-sigs.io/nfs-subdir-external-provisioner
    creationTimestamp: "2023-11-18T07:14:37Z"
    finalizers:
    - kubernetes.io/pv-protection
    name: pvc-30175895-d642-439f-b9c4-5c797718dabf
    resourceVersion: "3284"
    uid: 71a30160-25b5-4dc7-a54b-73d3d654e3e7
  spec:
    accessModes:
    - ReadWriteOnce
    capacity:
      storage: 1Mi
    claimRef:
      apiVersion: v1
      kind: PersistentVolumeClaim
      name: pvc-nginx
      namespace: default
      resourceVersion: "3278"
      uid: 30175895-d642-439f-b9c4-5c797718dabf
    nfs:
      path: /var/lib/nfs/default-pvc-nginx-pvc-30175895-d642-439f-b9c4-5c797718dabf
      server: 192.168.1.180
    persistentVolumeReclaimPolicy: Delete
    storageClassName: nfs-client
    volumeMode: Filesystem
  status:
    phase: Bound
kind: List
metadata:
  resourceVersion: ""
```

## PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"pvc-nginx","namespace":"default"},"spec":{"accessModes":["ReadWriteOnce"],"resources":{"requests":{"storage":"1Mi"}},"storageClassName":"nfs-client"}}
    pv.kubernetes.io/bind-completed: "yes"
    pv.kubernetes.io/bound-by-controller: "yes"
    volume.beta.kubernetes.io/storage-provisioner: k8s-sigs.io/nfs-subdir-external-provisioner
    volume.kubernetes.io/storage-provisioner: k8s-sigs.io/nfs-subdir-external-provisioner
  creationTimestamp: "2023-11-18T07:14:37Z"
  finalizers:
  - kubernetes.io/pvc-protection
  name: pvc-nginx
  namespace: default
  resourceVersion: "3286"
  uid: 30175895-d642-439f-b9c4-5c797718dabf
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Mi
  storageClassName: nfs-client
  volumeMode: Filesystem
  volumeName: pvc-30175895-d642-439f-b9c4-5c797718dabf
status:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Mi
  phase: Bound
```
