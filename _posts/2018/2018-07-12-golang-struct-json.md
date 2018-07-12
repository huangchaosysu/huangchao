---
layout: single
author_profile: true
title: "golang-Struct和Json互转"
date: 2018-07-12 10:24:53
# toc: true
tags:
  - golang
  - json
  - go-struct
categories:
  - golang
---

使用golang做web后端的时候经常遇到需要解析或发送json数据的时候，这篇帖子教你如何操作。

看下面的代码  
```
import "encoding/json"

type Product struct {
    Name      string
    ProductID int64
    Number    int
    Price     float64
    IsOnSale  bool
}

type JiraHttpReqFieldProject struct {
	Key string `json:"key"`
}

type JiraHttpReqField struct {
	Project     JiraHttpReqFieldProject   `json:"project,omitempty,inline"`
	Summary     string                    `json:"summary"`
	Description string                    `json:"description"`
}
```

正常的Json序列化的结果三这样的  
```
func main() {
    p := &Product{}
    p.Name = "Xiao mi 6"
    p.IsOnSale = true
    p.Number = 10000
    p.Price = 2499.00
    p.ProductID = 1
    data, _ := json.Marshal(p)
    fmt.Println(string(data))
}

//屏幕输出
{"Name":"Xiao mi 6","ProductID":1,"Number":10000,"Price":2499,"IsOnSale":true}
```

可以看到所有的字段名字跟我们定义的一模一样, 实际应用中我们大部分情况下都是使用小写形式来传递参数的的。一种解决办法是把struct的字段定义成小写的形式，但是，鉴于golang的这个语言对变量的访问控制，不推荐是。那么如何解决呢？使用给结构体的字段打Tag

就像JiraHttpReqField这样，tag的语法如下  
```
`json:"属性名,其他特性" bson:"属性名，其他特性"`
```

如果定义tag的属性也是struct类型的，则需要添加inline, tag里面加上omitempy，可以在序列化的时候忽略0值或者空值