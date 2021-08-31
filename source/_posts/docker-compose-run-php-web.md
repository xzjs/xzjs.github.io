---
title: 基于docker-compose跑起一个php网站
date: 2021-08-31 11:03:06
tags: docker php
categories: docker
thumbnail: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.xsumb.com%2Fuploads%2Fallimg%2F200329%2F124051EV-1.jpg&refer=http%3A%2F%2Fwww.xsumb.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1632971063&t=261b2ed66cbce30b68412b834f542c98
---
昨天拿到一个用thinkphp开发的网站，需要跑起来看看效果，但苦于自己好久不开发PHP了，手里已经没有集成环境，便想着用docker搭一套环境，本以为可以信手拈来，没想到还踩了不少坑，也对之前一知半解的东西加深了一下理解，故在这里记录一下
## 准备镜像
这个没有多大问题，去[hub.docker.com](hub.docker.com)找些官方的镜像就可以了，我这儿用到了nginx，php-fpm,mysql,获取命令放在这儿，方便大家直接使用

`docker pull nginx`

`docker pull bitnami/php-fpm`

`docker pull mysql`

## 编写docker-compose.yml
```yml
version: "3"
services:
  nginx:
    image: nginx
    ports:
      - 80:80
    volumes:
      - ./default.conf:/etc/nginx/conf.d/default.conf ##配置文件位置映射
      - .:/usr/share/nginx/html     ##网页文件位置映射
    depends_on:
      - php
  php:
    image: bitnami/php-fpm
    volumes:
      - .:/app ##网页文件位置映射
    depends_on:
      - mysql
    links: 
      - mysql
  mysql:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_PASSWORD: EisBzdye6adiJHHj ##数据库用户密码
      MYSQL_DATABASE: www_ammopic_com
      MYSQL_USER: www_ammopic_com
    ports: 
      - 3306:3306
    volumes:
      - ./www_ammopic_com.sql:/docker-entrypoint-initdb.d/db.sql
```

### 问题1：依赖启动
nginx依赖于php-fpm,php-fpm又依赖于mysql，depends_on很好的解决了这个问题
### 问题2：mysql自动填充创建数据库以及填充数据
虽然可以把mysql先运行起来，然后把sql文件拷贝进去，然后进入容器执行导入脚本，但总觉得这样操作很low，都想好了要做一条命令启动一个环境了，如果还有这么多操作就违背初衷了。
查了一下，mysql有个机制，可以从`/docker-entrypoint-initdb.d/`这个路径下检查sql文件，sh脚本等，从而实现自动导入数据的功能，非常好用，完美的解决了这个问题
### 问题3：各容器之间如何通信，统一端口映射到主机非常不优雅
之前操作的时候因为经常开发，所以像nginx，mysql这些都是把端口直接映射到主机的，这样方便了开发，但又违背了初衷。所幸links完美的解决了这个问题，这样在代码里连接mysql，在nginx里连接php-fpm都很方便了，而且不会对外暴露，非常好用

## 编写nginx配置文件
```nginx
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;
    root   /usr/share/nginx/html/public;

    location / {
        if (!-e $request_filename){
            rewrite  ^(.*)$  /index.php?s=$1  last;   break;
        }
        index  index.html index.php index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    location ~ \.php$ {
        root           /app/public;
        fastcgi_pass   php:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```
这种写了不知道多少遍的配置文件，居然也出现了翻船的时候，正应了那句话，拳不离手曲不离口。
### 问题1：root目录
thinkphp和laravel一样，是需要将根目录映射到public目录的，这该死的记性
### 问题2：连接到php-fpm应该如何写
以前写配置的时候nginx和php-fpm都是在同一台机器上，所以代码位置也放在了一起，这一次代码分别在nginx容器和php-fpm容器里，一时还写不对了。最后测试发现，在连接php-fpm的配置里，需要写的文件位置是php-fpm容器里的代码路径，如上述文件的`/app/public`

## 最后执行
`docker compose up`,镜像没有问题，但网站依旧访问不了，究其原因，是因为奇葩的开发者写的奇葩的逻辑，与本文想讲的技术无关了，但依然忍不住想在这里吐槽，这种不靠谱的程序员什么时候能少一点……