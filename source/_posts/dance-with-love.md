---
title: 与爱为舞面试
date: 2024-03-14 21:02:28
tags:
categories: 面试
thumbnail:
---
1. 布隆过滤器
1. 浏览器从输入到返回的全过程
1. http协议
1. https加密过程
1. golang的底层
1. 链表入环节点

## 百度面试
1. TCP拥塞控制
2. SQL语句非常薄弱
3. 事务的特性
4. join
5. 桶排序
6. linux操作，将a.txt 和 b.txt 的第一列交集写入c
在Linux中，可以使用命令行工具和脚本语言来取两个文件的交集。

方法一：使用sort和comm命令
```
sort file1.txt > file1_sorted.txt
sort file2.txt > file2_sorted.txt
comm -12 file1_sorted.txt file2_sorted.txt
```

这种方法先使用sort命令对两个文件进行排序，然后使用comm命令检测两个排序后的文件的交集。选项"-12"表示只输出交集部分。

方法二：使用grep命令

`grep -Fxf file1.txt file2.txt`

这种方法使用grep命令的选项来进行模式匹配。选项"-F"表示按照固定字符串而不是正则表达式进行匹配，"-x"表示完全匹配每一行，"-f"表示匹配来自文件的内容。

方法三：使用awk命令

`awk 'NR==FNR {a[$0]; next} $0 in a' file1.txt file2.txt`

这种方法使用awk命令来遍历两个文件。先将第一个文件的行存储在数组a中，然后逐行读取第二个文件并检查该行是否在数组a中，如果在则打印输出。

以上是几种取交集的实现方法，它们都能在Linux系统中有效地找到两个文件的交集。