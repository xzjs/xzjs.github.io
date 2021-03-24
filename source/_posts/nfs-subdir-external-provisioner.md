---
title: 全网唯一可用的nfs-provisioner
date: 2021-03-24 10:58:33
tags: nfs-provisioner
categories: k8s
thumbnail: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Faliyunzixunbucket.oss-cn-beijing.aliyuncs.com%2Fjpg%2F11112aa13b7e61391d3a38c6c1702643.jpg%3Fx-oss-process%3Dimage%2Fresize%2Cp_100%2Fauto-orient%2C1%2Fquality%2Cq_90%2Fformat%2Cjpg%2Fwatermark%2Cimage_eXVuY2VzaGk%3D%2Ct_100&refer=http%3A%2F%2Faliyunzixunbucket.oss-cn-beijing.aliyuncs.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1619145901&t=77be97660dfb565e12c8d6eea4b13cc7
---

# 起因
安装了nfs服务之后，需要安装一个k8s的nfs驱动，从而能自动创建pv，网上找了好多都无法自动创建pv，包括目前helm里的几个

# 解决方案
找到了helm上一个最近更新的[https://artifacthub.io/packages/helm/nfs-subdir-external-provisioner/nfs-subdir-external-provisioner](https://artifacthub.io/packages/helm/nfs-subdir-external-provisioner/nfs-subdir-external-provisioner)

## 坑
这个helm里的镜像是国外的，国内翻墙也下载不到，贼坑

## 填坑
使用阿里云的容器服务，使用海外机器总算把镜像搞下来了，具体方法见[如何在国内顺畅下载被墙的 Docker 镜像？](https://mp.weixin.qq.com/s/kf0SrktAze3bT7LcIveDYw),最后放出自己下载后构建的镜像`registry.cn-hangzhou.aliyuncs.com/xzjs/nfs-subdir-external-provisioner:v4.0.0`