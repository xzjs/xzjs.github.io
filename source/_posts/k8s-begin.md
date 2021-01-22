---
title: 手摸手从0开始k8s
date: 2021-01-22 18:13:40
tags: k8s
---
# 安装kubeadm
[参考文档](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
基本没有遇到什么问题
# 创建集群
使用了阿里云的镜像
```
sudo kubeadm init --pod-network-cidr 172.16.0.0/16 \
    --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers
```
上面的命令执行成功后，会输出一条和kubeadm join相关的命令，后面加入worker node的时候要使用。另外，给自己的非sudo的常规身份拷贝一个token，这样就可以执行kubectl命令了
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
# 安装calico插件
[yaml文件](https://docs.projectcalico.org/v3.11/manifests/calico.yaml)
修改里面的CALICO_IPV4POOL_CIDR的值来避免和宿主机所在的局域网段冲突（把原始的192.168.0.0/16 修改成了172.16.0.0/16)
`kubectl apply -f calico.yaml`
## 坑
这儿踩了个大坑，一直有一个node在running，但是没有ready，导致我后面pod之间互相无法访问，查了一下日志发现有一个fatal
```
int_dataplane.go 1035: Kernel's RPF check is set to 'loose'.  \
This would allow endpoints to spoof their IP address.  \
Calico requires net.ipv4.conf.all.rp_filter to be set to 0 or 1. \
If you require loose RPF and you are not concerned about spoofing, \
this check can be disabled by setting the IgnoreLooseRPF configuration parameter to 'true'.
```
查了资料发现calico需要内核参数是0或者1，但是Ubuntu20.04上默认是2，这样就会导致calico插件报这个错误，使用下面的命令修改
```
#修改/etc/sysctl.d/10-network-security.conf
$ sudo vi /etc/sysctl.d/10-network-security.conf

#将下面两个参数的值从2修改为1
#net.ipv4.conf.default.rp_filter=1
#net.ipv4.conf.all.rp_filter=1

#然后使之生效
$ sudo sysctl --system
```
# 安装ingress
k8s中的服务如果想被外网访问，就需要用ingress做一个负载均衡
[yaml文件](https://github.com/kubernetes/ingress-nginx/blob/controller-v0.41.2/deploy/static/provider/baremetal/deploy.yaml)
## 坑
安装的时候由于只有一个node，ingress默认不往master上装，需要让master节点参与工作负载
`kubectl taint nodes --all node-role.kubernetes.io/master-`
此处使用NodePort + External IP的方式，需要修改该yaml文件中name为ingress-nginx-controller的service，添加externalIP
```
externalIPs:
 - 10.0.7.144(node的ip)
```
# 部署mysql
## 创建pv
pv.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/k8s/mysql
```
创建pv
`kubectl create -f pv.yaml`
## 创建pvc
mysql-pvc.yaml
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql-pvc
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```
创建pvc
`kubectl create -f mysql-pvc.yaml`
## 使用helm安装mysql
`helm install mysql bitnami/mysql --set auth.rootPassword=root --set primary.persistence.existingClaim=mysql-pvc --set auth.database=forum`
### 坑
1. 使用文件修改设置无效，完全不知道为啥，只能在命令行里打这么一串了
2. mysql 启动会检测，这个时候会报`error: 'Can't connect to local MySQL server through socket '/opt/bitnami/mysql/tmp/mysql.sock' (2)'`,得进pod将这玩意的权限改成777，mysql才能启动成功
# 安装redis
## 创建pv
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-pv
spec:
  storageClassName: manual
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/k8s/redis
```
`kubectl create -f redis-pv.yaml`
`helm install redis --set usePassword=false --set cluster.enabled=false bitnami/redis`
> 不想要主从，所以cluster.enabled=false
# 部署应用
编写部署文件forum.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: forum
spec:
  selector:
    matchLabels:
      app: forum
  template:
    metadata:
      labels:
        app: forum
    spec:
      containers:
      - name: forum
        image: registry.cn-hangzhou.aliyuncs.com/xzjs/forum
        env:
        - name: DBUSER
          value: root
        - name: DBPWD
          value: root
        - name: DBHOST
          value: mysql
        - name: DBPORT
          value: '3306'
        - name: DBNAME
          value: forum
        - name: USER_IP
          value: 10.0.7.144
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8888
---
apiVersion: v1
kind: Service
metadata:
  name: forum
spec:
  selector:
    app: forum
  ports:
  - port: 8888
```
`kubectl apply -f forum.yaml`
# 配置ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: forum
spec:
  rules:
    - host: www.rsaicp.com
      http:
        paths:
          - path: /api/v1/forum/
            pathType: Prefix
            backend:
              service:
                name: forum
                port:
                  number: 8888
```
`kubectl apply -f forum-ing.yaml`
# 挂载host
![ihost](https://upload-images.jianshu.io/upload_images/6217974-08d12f9abbbf5c31.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 访问应用测试
![apifox](https://upload-images.jianshu.io/upload_images/6217974-d7492bc1c2bd6e3d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 安装elasticsearch+flutend+kibana
## 安装eck
有了这货装es全家桶会轻松一点，直接用helm装要了老命了
`kubectl apply -f https://download.elastic.co/downloads/eck/1.3.1/all-in-one.yaml`
![WX20210122-105738](https://upload-images.jianshu.io/upload_images/6217974-b3048dfb07b592da.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 创建PV
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: es-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/k8s/es
```
没有加StorageClass,这样就会使用默认的sc，es可以自动识别，分配pvc
```
kubectl create -f es-pv.yaml
```
## 安装es
```
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: es
spec:
  version: 7.10.2
  nodeSets:
  - name: default
    count: 1
    config:
      node.store.allow_mmap: false
```
`kubectl apply -f es.yaml`
然后检测服务是否正常`kubectl get elasticsearch`
![Screen Shot 2021-01-22 at 4.29.22 PM](https://upload-images.jianshu.io/upload_images/6217974-831d99b5345c53ef.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 安装fluentd
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  labels:
    k8s-app: fluentd-logging
    version: v1
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-logging
      version: v1
  template:
    metadata:
      labels:
        k8s-app: fluentd-logging
        version: v1
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
        env:
          - name:  FLUENT_ELASTICSEARCH_HOST
            # value: "quickstart-es-http"
            value: "10.99.15.145"
          - name:  FLUENT_ELASTICSEARCH_PORT
            value: "9200"
          - name: FLUENT_ELASTICSEARCH_SCHEME
            value: "https"
          # Option to configure elasticsearch plugin with self signed certs
          # ================================================================
          - name: FLUENT_ELASTICSEARCH_SSL_VERIFY
            value: "false"
          # Option to configure elasticsearch plugin with tls
          # ================================================================
          - name: FLUENT_ELASTICSEARCH_SSL_VERSION
            value: "TLSv1_2"
          # X-Pack Authentication
          # =====================
          - name: FLUENT_ELASTICSEARCH_USER
            value: "elastic"
          - name: FLUENT_ELASTICSEARCH_PASSWORD
            value: "ms7xW92w791drxr30yPg92EG"
          # Logz.io Authentication
          # ======================
          - name: LOGZIO_TOKEN
            value: "ThisIsASuperLongToken"
          - name: LOGZIO_LOGTYPE
            value: "kubernetes"
          - name: FLUENTD_SYSTEMD_CONF
            value: disable
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      serviceAccount: fluent-account
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```
还需要创建一个授权角色，否则会报错
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluent-account
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: fluent-account
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: ServiceAccount
    name: fluent-account
    namespace: default
```
```
kubectl apply -f role.yaml
kubectl apply -f fluentd.yaml
```
## 安装kibana
```yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: kibana
spec:
  version: 7.10.2
  count: 1
  elasticsearchRef:
    name: es
  http:
    tls:
       selfSignedCertificate:
        disabled: true
```
`kubectl apply -f kibana.yaml`
检测服务是否可用`kubectl get kibana`
![Screen Shot 2021-01-22 at 4.49.59 PM](https://upload-images.jianshu.io/upload_images/6217974-1e1daa2f6e344d4a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
使用nodeport对外网暴露
`nohup kubectl port-forward service/kibana-kb-http --address 0.0.0.0 5601 &`
访问服务器的5601端口，出现登录界面
登录帐号为elastic，登录密码使用`PASSWORD=$(kubectl get secret quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')`获取
# 打完收工
至此，一套k8s就部署好了，以后有什么问题我会补充在这里的