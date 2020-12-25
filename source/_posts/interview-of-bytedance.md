---
title: 字节跳动面试
date: 2020-12-24 18:24:22
tags: golang 字节跳动
categories: 面试
thumbnail: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.21ic.com%2F21ic_pic%2FFANKEJI%2F5d42913caf335.jpg&refer=http%3A%2F%2Fimg.21ic.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1611471486&t=44988bc8a6c57a5ebc5aab97ebe96d36
---
应朋友之邀，今天下午去字节送了颗人头，最后不负众望，被面试官撵出来了……
# 一面
## 谈一下之前重构百度账号中心的方案
吹了一波之前在百度改造restful接口的方案，但面试官并不感冒，提了一个显示文章的列表的场景，但感觉没有理解面试官的意思，没有提出面试官满意的restful解决方案，刚开始就得了个负分，这块得抽空找大佬再探讨探讨，等后面有什么心得再补充吧
## mysql索引快的原理
回答这个问题需要先看一下数据库的存储结构
![页结构](https://tva2.sinaimg.cn/large/9f8a45fbly1gm03j3bfj1j20cv0amq5a.jpg)
### 页和页之间的关系
![页和页之间的关系](https://tva3.sinaimg.cn/large/9f8a45fbly1gm03jnw7cij20op0ann1t.jpg)

> 有个知识，之前不知道的
聚集索引：以主键创建的索引，叶子节点存储的是表中的数据
非聚集索引：非主键创建的索引，叶子节点中存储的是主键和索引列，使用非聚集索引查询数据，会查询到叶子上的主键，再根据主键查到数据（这个过程叫做回表）

没有用索引的时候，需要遍历双向链表来定位对应的页，有了索引，可以用**二分查找**，这么弱智的答案，我当时居然没想到，这也是后面面试官问为什么主键建议用自增字段的答案

## 页码跳页性能（即sql offset会不会影响性能）
mysql查询时，offset过大影响性能的原因是多次通过主键索引访问数据块的I/O操作
InnoDB会，MyISAM不会，如图所示
![InnoDB和MyISAM对比图](https://tva2.sinaimg.cn/large/9f8a45fbly1gm03k5e9idj20q60md761.jpg)
InnoDB的二级索引对应的是主键，mysql查询的时候会根据主键将数据块查出来，然后执行offset丢弃，如果只查主键就不会有性能问题。MyISAM的主键索引和二级索引都指向数据块，因此没有这方面的问题
### 优化措施
先查询偏移后的主键，再查询数据块
```sql
select a.* from member as a inner join (select id from member where gender=1 limit 300000,1) as b on a.id=b.id
```
## golang中new和make的区别
1. make 仅用来分配及初始化类型为 slice、map、chan 的数据。new 可分配任意类型的数据.
2. new 分配返回的是指针，即类型 *Type。make 返回引用，即 Type.
3. new 分配的空间被清零, make 分配空间后，会进行初始化.

## 有一个字母翻译对照表，1代表A，2代表B，以此类推至26代表Z，现给一个整形数组，例如[1,2,3,4,5,6,7,8,9],求共有多少种翻译方式

```go
package main

import (
	"fmt"
)

func main() {
	s := []int{1, 2, 3, 4, 5, 6, 7, 8, 9}
	dict := make(map[int]string)
	for i := 1; i < 27; i++ { //初始化字典
		dict[i] = string('A' + (i - 1))
	}
	count := 0
	for i := 0; i < len(s); i++ {
		if i == 0 {
			count++
		} else if s[i-1]*10+s[i] < 27 && s[i-1]*10+s[i] > 0 {
			count++
		}
	}
	fmt.Println(count)
}
```

## redis zset的数据结构
![zset结构](https://upload-images.jianshu.io/upload_images/6217974-0fcba56ef3b682ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如图所示，L0层存储所有的数据，L1层随机抽取几个组成一个系数索引，L2层进一步抽取L1层，从而组成一个多层稀疏索引，这样就可以用二分法快速的找出所需要的数据了
## 利用redis做一个延时事件执行系统(设计)
![延时执行设计](https://upload-images.jianshu.io/upload_images/6217974-e1cdb7bcbf1a5f08.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
当时想的设计跟盒子科技的这张PPT高度类似，以时间为score，以事件数组为value，存储成一个zset，然后定期去取一定数量的事件进行执行。如果延时执行事件比较稀疏，就设定一个值，比如每次取的事件必须是一分钟内的，一分钟内没有时间则等待下次再取。
# 二面
## PHP代码sleep的时候，进程状态是怎样的，进程都有哪几种状态
1. 创建状态
2. 就绪状态
3. 运行状态
4. 阻塞状态
5. 终止状态

当代码执行sleep的时候进程处于阻塞状态
## linux系统中如何查看进程状态
使用命令 `ps -aux`，STAT列即为进程状态
![进程示例](https://tvax3.sinaimg.cn/large/9f8a45fbly1glzwl979zwj20gi08bdia.jpg)

### linux上进程有五种状态
1. R——Runnable（运行）：正在运行或在运行队列中等待
2. S——sleeping（中断）：休眠中，受阻，在等待某个条件的形成或接收到信号
3. D——uninterruptible sleep(不可中断)：收到信号不唤醒和不可运行，进程必须等待直到有中断发生
4. Z——zombie（僵死）：进程已终止，但进程描述还在，直到父进程调用wait4()系统调用后释放
5. T——traced or stoppd(停止)：进程收到SiGSTOP,SIGSTP,SIGTOU信号后停止运行
### 状态后缀表示：
`<`：优先级高的进程
`N`：优先级低的进程
`L`：有些页被锁进内存
`s`：进程的领导者（在它之下有子进程）
`l`：ismulti-threaded (using CLONE_THREAD, like NPTL pthreads do)
`+`：位于后台的进程组

## 数据库事务的实现原理
数据库不同的存储引擎可能会有一些区别。这里拿常用的InnerDB存储引擎举例：
### 原子性实现原理：
通过数据库Undo Log实现的。事务中在操作任何数据之前，首先将原数据备份到Undo Log然后进行数据的修改。如果事务中有任意操作发生异常或用户执行了 Rollback 语句，那么数据库就会使用Undo Log中的备份将数据恢复到事务开始之前的状态。 
### 一致性实现原理：
与原子性实现原理一样也是利用Undo Log
### 持久性实现原理：
通过数据库Redo Log实现的，Redo Log与Undo Log 相反，Redo Log 记录的是新数据的备份，事务提交之前，会把数据备份到Redo Log中并持久化。当系统崩溃时，虽然数据没有持久化到数据库中，但是 Redo Log 已经持久化。系统可以根据 Redo Log 的内容，将所有数据恢复到最新的
### 隔离性实现原理：
隔离性的实现原理比较特殊，是通过数据库锁的机制实现的。
隔离性分四个级别：读未提交（Read uncommitted）、读已提交（Read committed）、可重复读（Repeatable reads）、可序列化(Serializable)
MySQL的默认隔离级别就是Repeatable,Oracle默认Read committed
#### 读未提交：一个事务可以读到另外一个事务未提交的数据。脏读
实现：事务在读数据的时候并未对数据进行加锁。
事务在发生更新数据的瞬间，必须先对其加 行级共享锁，直到事务结束才释放。
举例：事务A读取某行记录时(没有加锁)，事务2也能对这行记录进行读取、更新。当事务B对该记录进行更新时，事务A读取该记录，能读到事务B对该记录的修改版本，即使该修改尚未被提交。
 事务A更新某行记录时，事务B不能对这行记录做更新，直到事务A结束。
#### 读已提交：一个事务可以读到另外一个事务提交的数据。不可重复读
实现：事务对当前被读取的数据加 行级共享锁（当读到时才加锁），一旦读完该行，立即释放该行级共享锁；
 事务在更新某数据的瞬间（就是发生更新的瞬间），必须先对其加 行级排他锁，直到事务结束才释放。
原理：事务A读取某行记录时，事务B也能对这行记录进行读取、更新；当事务B对该记录进行更新时，事务A再次读取该记录，读到的只能是事务B对其更新前的版本，或者事务B提交后的版本。
事务A更新某行记录时，事务B不能对这行记录做更新，直到事务1结束。
流程描述：事务A读操作会加上共享锁，事务B写操作时会加上排他锁，当事务B正在写操作时，事务A要读操作，发现有排他锁，事务A就会阻塞，等待排他锁释放(事务B写操作提交才会释放)，才能进行读操作。
#### 可重复读
实现：事务在读取某数据的瞬间（就是开始读取的瞬间），必须先对其加 行级共享锁，直到事务结束才释放；
事务在更新某数据的瞬间（就是发生更新的瞬间），必须先对其加 行级排他锁，直到事务结束才释放。
举例：事务A读取某行记录时，事务B也能对这行记录进行读取、更新；当事务B对该记录进行更新时，事务A再次读取该记录，读到的仍然是第一次读取的那个版本。
事务A更新某行记录时，事务B不能对这行记录做更新，直到事务1结束。
#### 可序列化(Serializable) 写操作串联执行
实现：事务在读取数据时，必须先对其加 表级共享锁 ，直到事务结束才释放；
事务在更新数据时，必须先对其加 表级排他锁 ，直到事务结束才释放。
举例：事务A正在读取A表中的记录时，则事务B也能读取A表，但不能对A表做更新、新增、删除，直到事务A结束。
事务A正在更新A表中的记录时，则事务B不能读取A表的任意记录，更不可能对A表做更新、新增、删除，直到事务A结束。
原理：在读操作时，加表级共享锁，事务结束时释放；写操作时候，加表级独占锁，事务结束时释放。
## 聚簇索引
跟聚集索引是一个东西，参见上面的聚集索引
## 反转一个链表，如1->2->3->4->5->6，转为1->5->4->3->2->6

```go
package main

import "fmt"

// ListNode 链表节点
type ListNode struct {
	Val  int
	Next *ListNode
}

//反转链表的实现
func reversrList(head *ListNode) *ListNode {
	cur := head
	var pre *ListNode = nil
	for cur != nil {
		pre, cur, cur.Next = cur, cur.Next, pre //这句话最重要
	}
	return pre
}

// Print 打印链表
func (l *ListNode) Print() {
	for l != nil {
		if l.Val > 0 {
			fmt.Println(l.Val)
		}
		l = l.Next
	}
}

func main() {
	m := 1
	n := 4
	root := new(ListNode)//头结点
	p := root
	for i := 1; i < 7; i++ {//初始化链表
		node := new(ListNode)
		node.Val = i
		p.Next = node
		p = p.Next
	}
	p.Next = new(ListNode)//尾节点
	l1 := root
	var l2, l3 *ListNode
	p = root
	index := 0
	for l3 == nil {//截成三段
		if index == m {
			l2 = p.Next
			p.Next = nil
			p = l2
		}
		if index == n {
			l3 = p.Next
			p.Next = nil
		}
		p = p.Next
		index++
	}
	l2 = reversrList(l2)// 反转l2
	list := []*ListNode{l2, l3}
	p = l1
	index = 0
	for index < len(list) {//拼接起来
		for p.Next != nil {
			p = p.Next
		}
		p.Next = list[index]
		index++
	}
	l1.Print()
}
```

