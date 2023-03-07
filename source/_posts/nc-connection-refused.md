---
title: nc测试本地端口一直拒绝
date: 2023-03-06 11:30:45
tags: 实战
categories: linux
thumbnail: https://img-blog.csdnimg.cn/img_convert/85087db091a35e54313ea9a41a321bd8.png
---

# nc测试本地端口一直拒绝
## 原因
混淆了nc/netcat/ncat之间的关系，且系统将nc/netcat默认指向了ncat，导致使用命令nc，其实使用的是ncat
## 解决方法
1. yum没有netcat，使用nc安装的是ncat，不要被外面的文章骗了，建议使用源码安装
2. 下载netcat `wget https://gigenet.dl.sourceforge.net/project/netcat/netcat/0.7.1/netcat-0.7.1.tar.gz`
3. 解压 `tar -zxvf netcat-0.7.1.tar.gz -C /usr/local/`
4. 改名
    ```
    cd /usr/local
    mv netcat-0.7.1 netcat
    ```
5. 安装
    切换目录：`cd /usr/local/netcat`
    配置，把文件存放在/opt/netcat下，删除时，卸载软件时，只要删除这个文件就行了：
    `./configure --prefix=/opt/netcat`
    编译：`make`
    安装：`make install`
    > 由于我的系统比较特殊，此处有一个报错`configure: error: cannot guess build type; you must specify one`
    解决办法
    1.在系统/usr路径下搜索 config.guess 和 config.sub 这两个文件。
    2.在当前编译工具目录下同样搜索 config.guess 和 config.sub 这两个文件。
    3.用系统的 config.guess 和 config.sub 文件替换当前编译工具目录下的这两个文件。
    4.重新执行configure。

6. 配置
    `vim /etc/profile`
    添加以下内容：
    ```
    # set  netcat path

    export NETCAT_HOME=/opt/netcat

    export PATH=$PATH:$NETCAT_HOME/bin
    ```

    保存，退出，并使配置生效：
    `source /etc/profile`

7. 删除原先的链接，重建
    ```
    rm /usr/bin/nc

    ln -s /opt/netcat/bin/nc /usr/bin/nc
    ```

8. 验证
    `nc 127.0.0.1 6443`
