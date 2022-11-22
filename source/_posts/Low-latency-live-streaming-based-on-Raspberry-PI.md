---
title: 基于树莓派的低延时直播
date: 2022-11-22 10:22:28
tags: Raspberry
categories: 实战
thumbnail: https://img0.baidu.com/it/u=3350889853,2756525951&fm=253&fmt=auto&app=138&f=JPEG?w=900&h=450
---
# 基于树莓派的低延时
## 方案一
![方案图](https://tvax2.sinaimg.cn/large/9f8a45fbgy1h8dp9zn33oj20ms0d7gm5.jpg)
### 具体步骤
1. 通过ffmpeg采集摄像头数据，设定分辨率以及帧率
1. ffmpeg将采集到的数据通过管道传输到命令行中
1. golang程序读使用webrtc与浏览器进行p2p打洞
1. 打洞成功后，将读取到的视频帧进行组装发送
1. js程序接收到数据帧后进行解析，并渲染到canvas上进行展示
### 遇到的问题
1. 时延还是在三秒，没有达到webrtc号称的500ms
1. 稳定性不如推流拉流
### 下一步的思路
1. 尝试使用c++程序替换golang程序，看是否是语言性能上的瓶颈