---
title: '阿里云csi云盘Kuberntes PVC扩容'
date: 2024-03-23T14:42:20+08:00
draft: true
---

```sh
kubectl edit pvc taosdata-tdengine-0
```

```yaml
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      # increase the size here   
      storage: 25Gi
  storageClassName: alicloud-disk-essd-sz-zone-d
  volumeMode: Filesystem
```

等待自动扩容
