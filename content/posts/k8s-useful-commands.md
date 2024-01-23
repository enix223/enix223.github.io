---
title: 'K8s Useful Commands'
date: 2023-11-18T16:44:30+08:00
draft: true
---



kubectl drain cs-yellow --ignore-daemonsets --delete-emptydir-data
ansible cs-yellow -u root -m shell -a "apt-mark unhold kubelet kubectl && apt-get update && apt-get install -y kubelet='1.27.8-1.1' kubectl='1.27.8-1.1' && apt-mark hold kubelet kubectl"
ansible cs-yellow -u root -m shell -a "systemctl daemon-reload && systemctl restart kubelet"
kubectl uncordon cs-yellow

ansible dvms -u root -m shell -a "curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.27/deb/Release.key | gpg --dearmor --yes -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg"
ansible dvms -u root -m shell -a "echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.27/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list"


kubectl exec -it dnsutils -- bash -c 'ping 10.244.8.5'
kubectl exec -it dnsutils -- bash -c 'ping 10.244.4.5'

kubectl exec -it dnsutils -- bash -c 'nc -z 10.244.4.4 53 && echo ok'
kubectl exec -it dnsutils -- bash -c 'nslookup harbor.harbor && echo ok'

kubectl describe po  -n kube-system -l k8s-app=kube-dns

kubectl get po  -n kube-system -l k8s-app=kube-dns

kubectl logs -f -n kube-system -l k8s-app=kube-dns

kubectl delete po  -n kube-system -l k8s-app=kube-dns


## Print podCIDR for each nodes

```shell
echo "cs-blue podCIDR = $(kubectl get nodes cs-blue -o jsonpath='{.spec.podCIDR}')"
echo "cs-green podCIDR = $(kubectl get nodes cs-green -o jsonpath='{.spec.podCIDR}')"
echo "cs-neptune podCIDR = $(kubectl get nodes cs-neptune -o jsonpath='{.spec.podCIDR}')"
echo "cs-orange podCIDR = $(kubectl get nodes cs-orange -o jsonpath='{.spec.podCIDR}')"
echo "cs-pluto podCIDR = $(kubectl get nodes cs-pluto -o jsonpath='{.spec.podCIDR}')"
echo "cs-purple podCIDR = $(kubectl get nodes cs-purple -o jsonpath='{.spec.podCIDR}')"
echo "cs-red podCIDR = $(kubectl get nodes cs-red -o jsonpath='{.spec.podCIDR}')"
echo "cs-yellow podCIDR = $(kubectl get nodes cs-yellow -o jsonpath='{.spec.podCIDR}')"
```

```
cs-blue podCIDR = 10.244.3.0/24
cs-green podCIDR = 10.244.8.0/24
cs-neptune podCIDR = 10.244.6.0/24
cs-orange podCIDR = 10.244.5.0/24
cs-pluto podCIDR = 10.244.4.0/24
cs-purple podCIDR = 10.244.9.0/24
cs-red podCIDR = 10.244.0.0/24
cs-yellow podCIDR = 10.244.7.0/24
```