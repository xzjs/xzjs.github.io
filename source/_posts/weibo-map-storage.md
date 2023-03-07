---
title: 微博图床也炸了
date: 2023-03-07 11:15:34
tags: 微博 图床 反盗链
categories: 实战
thumbnail: https://p7.itc.cn/images01/20211206/3a91e138b9b6431797f5cd157eac952e.jpeg
---
# 微博图床也炸了
一波未平，一波又起，正在写怎么使用github action的文章，打算贴个图，结果发现我的微博图床403了，整个23年简直是IT界的世界末日啊
## 修复方案
具体这里：

https://mp.weixin.qq.com/s/LFX37OvOEUcGSdWYD0XWxQ

太长不看：

也就是说，已经阵亡的微博图床，在原先图片链接的前面加上
https://image.baidu.com/search/down?url=
即可恢复访问。

备用服务：

我找到了 4 个图片缓存服务网站，可以让微博图片重新恢复访问。

WordPress：
https://i0.wp.com/图片地址 （图片地址要掉 https://）

Weserv.nl：
https://images.weserv.nl/?url=图片地址

百度 1：
https://image.baidu.com/search/down?url=图片地址

百度 2：
https://gimg2.baidu.com/image_search/&app=2020&src=图片地址 （图片地址要去掉 https://）

真的是太难了，一天到晚在戴着镣铐跳舞，再这样下去还能坚持多久
