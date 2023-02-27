---
title: 手摸手pyside6+pySerial实战
date: 2023-02-17 11:58:30
tags: python qt 串口
categories: 实战
thumbnail: https://tvax2.sinaimg.cn/large/9f8a45fbgy1hbhu7ael26j203k03kjrf.jpg
---
# 手摸手pyside6+pySerial实战
最近接了个活，开发一个软件，要实现以下几个功能
1. 读取串口数据并保存起来
2. 播放选定的视频
3. 读取摄像头并保存成视频
4. 串口数据与视频同步保存

## 读取串口
[pyserial](https://pyserial.readthedocs.io/en/latest/pyserial.html)
### linux的串口port查询
`ls /dev/*`
找到带usb的就是
### windows串口查询
打开设备管理器，找串口，类似于COM6的就是

## 可视化界面
pythonQT已经做得很成熟了，使用起来也比较容易，感觉可以抛弃Electron，市面上的文章还停留在qt5，我早早地用上了pyside6
文档：[pyside6](https://doc.qt.io/qtforpython-6/#)
文档很不全，又很多细节没有，可以看qt的c++文档，然后自己翻译成python
### 在windows下出现文件不可读或者地址不可写
指定路径的时候直接使用的str，这样在linux和mac下是可以用的，但在windows下是会报错的，所以写路径的时候一律使用Qurl