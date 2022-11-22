---
title: mindspore踩坑记
date: 2022-10-24 15:24:45
tags: mindspore
categories: AI
thumbnail: https://www.mindspore.cn/static/img/logo_black.74c3909e.png
---
老美亡我之心不死，卡完这个卡那个，现在又开始卡GPU了，AI国产化迫在眉睫，勃，三尺微命，一介书生，奉命来蹚出一条国产化道路。
## 安装MindSpore
华为给了一份安装文档[pip方式安装MindSpore Ascend 910版本](https://gitee.com/mindspore/docs/blob/r1.9/install/mindspore_ascend_install_pip.md),基本是照着操作的，但错误百出，国内公司在文档这块儿还是不行，即使是华为也不例外，哎，一路磕磕绊绊往下走吧。
### 安装开发套件包
找了半天的文档[安装开发套件包](https://www.hiascend.com/document/detail/zh/canncommercial/51RC2/envdeployment/instg/instg_000064.html)
### 报错 Device 0 call rtSetDevice failed, ret[507000]
检查了一下npu，发现npu也报错了，等华为工程师看看吧
![WechatIMG47](https://tvax3.sinaimg.cn/large/9f8a45fbgy1h7gh0m5dzsj20ji02y3yz.jpg)