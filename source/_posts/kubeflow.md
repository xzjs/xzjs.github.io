---
title: kubeflow
date: 2023-02-28 11:40:37
tags: k8s ML 机器学习
categories: 实战
thumbnail:
---
# kubeflow初体验
## 安装
### 配置StorageClass
sc.yaml
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```
执行`kubectl -f sc.yaml`
执行`kubectl patch storageclass standard -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'`将刚创建的sc设置为默认sc
### 下载