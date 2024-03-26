---
title: 大模型演示平台踩坑记
date: 2024-03-21 15:12:07
tags: baichuan ffmpeg 
categories: 踩坑
thumbnail: https://aeiljuispo.cloudimg.io/v7/https://cdn-uploads.huggingface.co/production/uploads/640dd3e0c364a086c6322ad2/acwcllU0PQz4Bg3gchhYo.png?w=200&h=200&f=face
---
# 大模型
## 百川
### 坑
#### 问题 
`AttributeError: 'list' object has no attribute 'as_dict'`
#### 解决方法
`pip install bitsandbytes==0.41.0 -i https://pypi.tuna.tsinghua.edu.cn/simple/`
指定低版本bitsandbytes