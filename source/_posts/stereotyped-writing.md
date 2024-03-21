---
title: 后端面试八股文
date: 2024-03-21 12:12:46
tags: golang mysql redis linux tcp
categories: 面试
thumbnail: https://img2.baidu.com/it/u=3355645902,3598570405&fm=253&fmt=auto&app=120&f=JPEG?w=627&h=417
---
# golang
# mysql
## 事务的四个特性
原子性、一致性、隔离性、持久性
## left join和right join的区别
left会保存全部左边的，right会保存全部右边的
# redis
## redis的五个数据类型
string，list，hash，set，sort set
# linux
## linux 三剑客
grep，sed，awk
# 网络
## tcp是如何保持可靠性的
1. 校验和
2. 序列号和确认应答
3. 超时重传机制
4. 连接管理
5. 流量控制
6. 拥塞控制

## http协议常用状态码
301 永久重定向
302 临时重定向
400 客户端请求中的语法错误
401 未登录
403 拒绝访问
404 未找到
500 服务端执行请求时发生错误
502 服务器网关错误
503 服务器处于负载或停机维护