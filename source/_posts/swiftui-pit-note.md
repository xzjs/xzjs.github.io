---
title: swiftui-pit-note
date: 2022-06-23 11:42:18
tags: swiftui
categories: iOS
thumbnail:https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fpic1.zhimg.com%2Fv2-03be98cebbc736d6e6f466f70bae9654_1440w.jpg%3Fsource%3D172ae18b&refer=http%3A%2F%2Fpic1.zhimg.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1658547811&t=9d566698c4d44047cda99dc9b875ef25
---
## picker不显示可选项
刚开始照着[官方的文档](https://developer.apple.com/documentation/swiftui/picker)写选择器，但写出来在模拟器里始终点不出来，后来多方寻找之后才发现需要再外面包一层
```
NavigationView{
    ...
}
```
90后站稳历史舞台后，连苹果的官方文档都不靠谱了