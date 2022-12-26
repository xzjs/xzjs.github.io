---
title: Time.time踩坑
date: 2022-12-26 16:03:24
tags: go
categories: 开发
thumbnail: https://pic1.zhimg.com/v2-a4bd0e19fc987be19e1133fa61016226_1440w.jpg?source=172ae18b
---

# 问题
后端数据格式定义
```go
type Plan struct {
	gorm.Model
	UserID    uint      `json:"-"`
	CardID    uint      `json:"card_id"`
	Sum       int       `json:"sum"` //总金额
	CycleID   uint      `json:"cycle_id"`
	Total     int       `json:"total"` //总次数
	Frequency int       `json:"frequency"`
	Floor     int       `json:"floor"` //下限
	Ways      []Way     `json:"way_id" gorm:"many2many:plan_ways;"`
	Start     time.Time `json:"start"`
	End       time.Time `json:"end"`
}
```
前端传入数据
```json
{
    "card_id": 1,
    "sum": 0,
    "cycle_id": 2,
    "total": 0,
    "frequency": 3,
    "floor": 99,
    "ways": [
        {"ID":1},
        {"ID":2},
        {"ID":3}
    ],
    "start": "2022-01-01 00:00:00",
    "end": "2023-01-01 00:00:00"
}
```
报错
```json
{
    "Layout": "\"2006-01-02T15:04:05Z07:00\"",
    "Value": "\"2022-01-01 00:00:00\"",
    "LayoutElem": "T",
    "ValueElem": " 00:00:00\"",
    "Message": ""
}
```
# 原因
go 中使用 json.Unmarshal 转换结构体时，若结构体中有时间类型作为解析字段时，使用的是国际标准 RFC3339 （2006-01-02T15:04:05Z07:00） 格式来作为默认格式进行解析的。
算是一个常识性问题，将传入的时间字符串改为RFC3339格式即可
```json
{
    "card_id": 1,
    "sum": 0,
    "cycle_id": 2,
    "total": 0,
    "frequency": 3,
    "floor": 99,
    "ways": [
        {"ID":1},
        {"ID":2},
        {"ID":3}
    ],
    "start": "2022-01-01T00:00:00Z",
    "end": "2023-01-01T00:00:00Z"
}
```