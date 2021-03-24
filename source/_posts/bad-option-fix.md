---
title: 修复nfs报错bad option
date: 2021-03-24 10:43:36
tags: nfs
categories: k8s
thumbnail: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Faliyunzixunbucket.oss-cn-beijing.aliyuncs.com%2Fjpg%2F11112aa13b7e61391d3a38c6c1702643.jpg%3Fx-oss-process%3Dimage%2Fresize%2Cp_100%2Fauto-orient%2C1%2Fquality%2Cq_90%2Fformat%2Cjpg%2Fwatermark%2Cimage_eXVuY2VzaGk%3D%2Ct_100&refer=http%3A%2F%2Faliyunzixunbucket.oss-cn-beijing.aliyuncs.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1619145901&t=77be97660dfb565e12c8d6eea4b13cc7
---

# 起因
k8s安装nfs之后，安装应用后部分机器上的pod卡在了镜像创建上，报错如下
```
 Warning  FailedMount  8m25s (x55 over 10h)   kubelet  Unable to attach or mount volumes: unmounted volumes=[redis-data], unattached volumes=[start-scripts health redis-data config redis-tmp-conf default-token-dlhmb]: timed out waiting for the condition
  Warning  FailedMount  2m11s (x333 over 11h)  kubelet  MountVolume.SetUp failed for volume "pvc-0fc9cd8d-793e-413b-b9d2-51a7da98eecf" : mount failed: exit status 32
Mounting command: mount
Mounting arguments: -t nfs 172.18.116.129:/data/k8s/default-redis-data-my-redis-slave-0-pvc-0fc9cd8d-793e-413b-b9d2-51a7da98eecf /var/lib/kubelet/pods/ecec2b7a-affc-4c36-85fa-3877d93ab8a9/volumes/kubernetes.io~nfs/pvc-0fc9cd8d-793e-413b-b9d2-51a7da98eecf
Output: mount: /var/lib/kubelet/pods/ecec2b7a-affc-4c36-85fa-3877d93ab8a9/volumes/kubernetes.io~nfs/pvc-0fc9cd8d-793e-413b-b9d2-51a7da98eecf: bad option; for several filesystems (e.g. nfs, cifs) you might need a /sbin/mount.<type> helper program.
```

# 解决方案
在出错的机器上安装nfs-common即可解决
```
sudo apt install nfs-common
```