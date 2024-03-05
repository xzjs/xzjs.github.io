---
title: 记一次漏洞修复经历
date: 2024-02-22 21:39:03
tags: nginx mysql php bug
categories: 运维
thumbnail: https://i0.sinaimg.cn/IT/s/s/2008-03-13/bb6f42d59e16147a8c6958b732e0ef6f.bmp
---
最近整了个团队的网站，要上线的时候所里扫描了一下，结果发现一堆高危漏洞，真是不看不知道，一看吓一跳。
## 1. nginx 缓冲区错误漏洞(CVE-2022-41741)
nginx 1.19.1版本之前，存在缓冲区溢出漏洞。远程攻击者可通过向nginx发送特定HTTP请求，导致内存损坏。
修复方式：升级到最新版本
`vim /etc/yum.repos.d/nginx.repo`
```bash
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```
`yum install nginx`
然后启动nginx服务即可

## 2. Nginx 信任管理问题漏洞(CVE-2021-3618)
ALACA是一种应用层协议内容混淆攻击，利用实现不同协议但使用兼容证书（如多域或通配符证书）的TLS服务器。在TCP/IP层访问受害者流量的MiTM攻击者可以将流量从一个子域重定向到另一个子域，从而产生有效的TLS会话。这破坏了TLS的身份验证，并且可能会发生跨协议攻击，其中一个协议服务的行为可能会在应用层危害另一个协议。
修复方式：升级到最新版本
上一个bug修复的同时会修复

## 3. nginx 越界写入漏洞（CVE-2022-41742）
NGINX开源1.23.2和1.22.1之前的版本、NGINX开放源代码订阅R2 P1和R1 P1之前的版本以及NGINX Plus R27 P1和R26 P1以前的版本在模块ngx_http_mp4_module中存在漏洞，该漏洞可能允许本地攻击者使用特制的音频或视频文件导致工作进程崩溃，或导致工作进程内存泄漏。当在配置文件中使用mp4指令时，此问题仅影响使用模块ngx_http_mp4_module构建的NGINX产品。此外，只有当攻击者能够使用模块ngx_http_mp4_module触发对特制音频或视频文件的处理时，才有可能进行攻击。
修复方式：升级到最新版本

## 4.PHP 安全漏洞(CVE-2022-31629)
PHP 7.4.31之前版本、8.0.24之前版本和8.1.11之前版本存在安全漏洞，攻击者利用该漏洞可以能够在受害者的浏览器中设置一个标准的不安全 cookie。
修复方式：升级到最新版本

## 5.OpenSSH CBC模式信息泄露漏洞(CVE-2008-5161)
在(1)SSH Tectia客户端和服务器以及Connector 4.0 through 4.4.11, 5.0 through 5.2.4, and 5.3 through 5.3.8;Client and Server and ConnectSecure 6.0 through 6.0.4; IBM System z 6.0.4上的Linux服务器;适用于IBM z / OS 5.5.1及更早版本，6.0.0和6.0.1的服务器;和Client 4.0-J through 4.3.3-J and 4.0-K through 4.3.10-K;

以及(2)OpenSSH 4.7p1以及可能的其他版本中的SSH协议中的错误处理，当在Cipher Block Chaining(CBC)模式下使用block cipher algorithm时，远程攻击者可以更容易地从SSH会话中的任意密文块中恢复某些明文数据向量。

修复方式：升级到最新版本
修复步骤：
###  为了防止安装失败，先安装telnet以防无法登录
1. 检查是否安装了telnet
```
rpm -q telnet-server 
rpm -q telnet
```
2. 安装telnet服务
`yum install telnet* -y`
3. 启动telnet服务
```
systemctl enable telnet.socket
systemctl start telnet.socket
```
### 安装新的openssh
1. 安装依赖包
`yum -y install zlib* pam-* gcc openssl-devel`
2. 备份原有ssh服务版本
```
mv /etc/ssh /etc/ssh.bak
mv /usr/bin/ssh /usr/bin/ssh.bak
mv /usr/sbin/sshd /usr/sbin/sshd.bak
```
3. 下载安装新的openssh服务
去 http://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/ 下载最新版软件
4. 删除现有的安装SSH的相关软件包
```rpm -e `rpm -qa | grep openssh` --nodeps```
我这儿提示啥都没删除，什么鬼
5. 安装新的openssl
因为新的openssh依赖于openssl，所以需要安装新的openssl
去 https://www.openssl.org/source/ 下载最新版软件
解压 `tar -zxvf openssl-3.2.1.tar.gz`
`./config --prefix=/usr/local/openssl `,报错Can't locate IPC/Cmd.pm in @INC
`yum install perl-IPC-Cmd`安装依赖进行修复
`make && make install`开始安装

```bash
mv /usr/bin/openssl /usr/bin/openssl.bak
ln -sf /usr/local/openssl/bin/openssl /usr/bin/openssl
echo "/usr/local/openssl/lib64" >> /etc/ld.so.conf
ldconfig -v
```
检查版本`openssl version`

6. 安装新的openssh服务
```bash
tar -zxvf openssh-9.6p1.tar.gz
cd openssh-9.6p1
./configure --prefix=/usr/local/openssh --with-zlib=/usr/local/zlib --sysconfdir=/etc/ssh  --with-openssl-includes=/usr/local/openssl/include --with-ssl-dir=/usr/local/openssl --with-md5-passwords   --with-pam
make && make install
```

7. 复制源码解压路径的开机启动脚本
```bash
cp openssh-9.6p1/contrib/redhat/sshd.init /etc/init.d/sshd
chmod u+x /etc/init.d/sshd
```

8. 修改开机启动文件
```bash
sed -i '25cSSHD=/usr/local/sbin/sshd' /etc/init.d/sshd
sed -i '41c/usr/local/bin/ssh-keygen -A' /etc/init.d/sshd
```

9. 修改配置文件，允许root用户通过ssh远程登录
`sed -i "/#PermitRootLogin prohibit-password/c\PermitRootLogin yes" /etc/ssh/sshd_config`

10. 启动ssh
```bash
chkconfig --add sshd
cp /usr/local/sbin/sshd /usr/sbin/sshd
systemctl start sshd
```

### 卸载telnet
为了安全，还得把telnet卸载掉
`yum remove telnet*`

## 6. ICMP timestamp请求响应漏洞
### 详细描述
远程主机会回复ICMP_TIMESTAMP查询并返回它们系统的当前时间，这可能允许攻击者攻击一些基于时间认证的协议
### 解决办法
在防火墙上过滤外来的ICMP timestamp（类型 13）报文以及外出的ICMP timestamp回复报文
### 修复步骤
```bash
iptables -A INPUT -p ICMP --icmp-type timestamp-request -j DROP
iptables -A INPUT -p ICMP --icmp-type timestamp-reply -j DROP
```

## 7. 允许Traceroute探测
### 详细描述
本插件使用Traceroute探测来获取扫描器与远程主机之间的路由信息。攻击者也可以利用这些信息来了解目标网络的网络拓扑。

### 解决办法
在防火墙出入站规则中禁用echo-reply（type 0）、time-exceeded（type 11）、destination-unreachable（type 3）类型的ICMP包。

### 修复步骤
```bash
iptables -A INPUT -p ICMP --icmp-type echo-reply -j DROP
iptables -A OUTPUT -p ICMP --icmp-type echo-reply -j DROP
iptables -A INPUT -p ICMP --icmp-type time-exceeded -j DROP
iptables -A OUTPUT -p ICMP --icmp-type time-exceeded -j DROP
iptables -A INPUT -p ICMP --icmp-type destination-unreachable -j DROP
iptables -A OUTPUT -p ICMP --icmp-type destination-unreachable -j DROP
```


## 8. 可通过HTTP获取远端WWW服务信息
### 详细描述
![nginx信息](https://p.ipic.vip/hhii9z.png)
### 解决办法
1. 去 https://github.com/openresty/headers-more-nginx-module/tags 下载合适的组件,解压
2. 执行 nginx -V 查看安装参数，拷贝 configure arguments 后的安装参数,增加--add-module=headers-more-nginx-module,执行./configure
3. 执行 make && make install 安装
4. 修改nginx.conf文件,增加more_clear_headers 'Server';
5. 重启nginx服务

## 9. 可以获取到MySQL/MariaDB/Percona/TiDB Server版本信息
### 详细描述
telnet 192.168.64.147 3306
![](https://p.ipic.vip/eeogiv.png)
### 解决办法
找了一圈，不会隐藏，真操蛋

## 10. SSH版本信息可被获取
### 详细描述
通过telnet连接ssh端口时会显示ssh的版本信息
![](https://p.ipic.vip/knaivu.png)
### 解决办法
`sed -i 's/OpenSSH_9.6/hello world/g' /usr/sbin/sshd`
将版本号隐藏起来

## 11. 探测到SSH服务器支持的算法
描述：本插件用来获取SSH服务器支持的算法列表
处理：无法处理。ssh协议协商过程就是服务端要返回其支持的算法列表。搞个锤子啊






