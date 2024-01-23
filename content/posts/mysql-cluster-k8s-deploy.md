---
title: 'Mysql Cluster K8s Deploy'
date: 2023-11-11T18:06:09+08:00
draft: true
---

Execute the following to get the administrator credentials:

  echo Username: root
  MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace ishare secret-mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d)

To connect to your database:

  1. Run a pod that you can use as a client:

    ```shell
    kubectl run mysql-client --tty -i --image docker.io/bitnami/mysql:8.0.35-debian-11-r0 --namespace ishare --env MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD --command -- bash
    ```

  2. To connect to primary service (read/write):

    ```shell
    mysql -h mysql-primary.ishare.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"
    ```

  3. To connect to secondary service (read-only):

    ```shell
    mysql -h mysql-secondary.ishare.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"
    ```
