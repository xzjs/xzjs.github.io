---
title: 记录安装vsftpd踩坑的全过程
date: 2021-04-21 17:02:25
tags: ftp
categories: ftp
thumbnail: https://img2.baidu.com/it/u=106334478,1245385895&fm=26&fmt=auto&gp=0.jpg
---

# 安装
使用ubuntu安装的，`sudo apt-get install vsftpd`,有些文章坑人名字都写错了

# 坑一 匿名用户默认路径在哪儿
配置文件是`/etc/vsftpd.conf`
但是你是看不到默认路径的，如果这个时候往ftp上传东西会报553错误，没有读写权限，踩了一下午坑，发现默认的路径在`/srv/ftp`
但直接往这个文件夹里传东西是写不进去的，也不能将当前目录设置为大家都可访问，这个时候需要在目录里再创建一个文件夹`mkdir public`
将当前文件夹设置为ftp所有`sudo chown ftp:ftp public`
再将文件夹设置为777`sudo chmod -R 777 public`
重启ftp`sudo service vsftped restart`
这个时候匿名用户已经可以上传了，但是会发现下载不了，接着踩第二个坑

# 坑二 匿名用户无法下载
这个依然要改配置文件，但配置文件里没有，需要自己添加，在文件的末尾添加如下配置
```
anon_umask=022
anon_other_write_enable=YES
```
加了这两句之后就可以下载和删除了，但之前测试时候上传的文件因为权限不对，还是没有办法下载的，需要把之前的测试文件删掉，重新上传，至此，一个简单的匿名FTP就搭建好了，方便开发人员共享一些东西
