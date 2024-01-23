---
title: 'Helm Cheatsheet'
date: 2023-11-03T11:06:01+08:00
draft: false
toc: true
tags: [helm, k8s, cheatsheet]
---

## Concepts

### Chart

A *Chart* is a helm package. It contains all resource definitions, tools, service.

### Repository

A *Repository* is the place where charts collocated and shared.

### Release

A *Release* is an instance of a chart running in K8S.

## Cheatsheet

### Add a chart repo

```shell
helm repo add <repo-name> <repo-url>
```

example:

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
```

### List all chart repo

```shell
helm repo list
```

### Update chart repo

```shell
helm repo update
```

### Install chart

```shell
helm install [<release-name>] <chart-name>
```

example

```shell
helm install bitnami/mysql --generate-name
```

### List releases

```shell
helm list
```

### Uninstall release

```shell
helm uninstall <release-name>
```

example:

```shell
helm uninstall mysql-1612624192
```

### Get release info

```shell
helm status mysql-1612624192
```
