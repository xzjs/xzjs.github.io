---
title: 鉴于开车老被加塞，我做了一个加塞复盘系统（1）
date: 2022-01-19 16:47:53
tags: python yolo deepsort
categories: AI
thumbnail: https://tva3.sinaimg.cn/large/9f8a45fbly1gyj2x7y3t1j21do0rk7r1.jpg
---
众所周知，在北京开车是一件非常痛苦的事，一方面是因为堵，一方面是因为司机不讲公德。奉公守法的我每天不是在被加塞，就是在被加塞的路上，苦啊！鉴于此，我决定写个小项目，记录下那些加塞我的豪横司机，记在小本本上，然后画圈圈诅咒他们。
## 说干就干，先说思路
我手里有很多行车记录仪拍摄的视频，里面一桩桩一件件记录着被各种豪车加塞的场面，把视频做一个检测与追踪，再判断它们的轨迹，当车辆的位置超过我的车的中轴线，我就认为它加塞我加塞成功了，我已经被气的骂娘了，如下图所示
![加塞图](https://tva2.sinaimg.cn/large/9f8a45fbly1gyj4b2hwh1j20ui083aan.jpg)
## 图像检测，还得YOLO
车辆的检测，我决定选用yolo来做（因为我只知道这一种），官方网站[https://ultralytics.com/](https://ultralytics.com/),现在已经到v5了，这玩意当真是非常好用，能识别很多种物体，检测个小汽车自然不在话下
## 图像追踪，deepsort
至于检测后的图像追踪，就没有相关的经验了，请教了相关的专家，给我推荐了deepsort，然而这个算法就没有yolo那么友好，没有官网，没有文档，怎么用也不知道，全看悟性，我在github上找了一个有人集成了yolo和deepsort的项目[https://github.com/Sharpiless/Yolov5-Deepsort](https://github.com/Sharpiless/Yolov5-Deepsort)，作者小哥使用说明写的很简陋，代码也有一些bug，但修了一下勉强能用
## 修改bug
运行demo的时候弹出了报错`TypeError: load() missing 1 required positional argument: 'Loader'`,只需要将引用load处修改为`self.update(float(fo.read(),Loader=yaml.SafeLoader))`，添加缺少的参数即可
由于我是在mac环境下，pytorch不支持GPU，只支持CPU，故无法使用half，需要将代码中的所有half改为float，才可使用
## 运行demo
经过了漫长的pip install和改bug之后，终于将demo跑起来了，展示一下跑出来的效果
![demo效果图](https://tva3.sinaimg.cn/large/9f8a45fbly1gyj2x7y3t1j21do0rk7r1.jpg)
## 下一步
接下来就要开始自己读代码搓代码了，等着看我的加塞检测算法吧，哈哈哈