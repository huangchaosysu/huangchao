---
layout: single
author_profile: true
title: "python实用技法-02"
date: 2018-10-16 10:30:53
# toc: true
tags:
  - python
categories:
  - python
  - python实用技法
---



* 我们想在字典上对数据执行各式各样的计算，例如：最大值、最小值、排序等

### 解决方案

zip()函数用于将可迭代的对象作为参数，将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的列表。

假设有一个字典，在股票名称和对应价格之间做了映射：
```
prices={
'ACME':45.23,
'AAPL':612.78,
'IBM':205.55,
'HPQ':37.20,
'FB':10.75
}
```
为了能对字典内容做些有用的计算，通常会利用zip()函数将字典的键和值反转过来。
```
prices={
'ACME':45.23,
'AAPL':612.78,
'IBM':205.55,
'HPQ':37.20,
'FB':10.75
}
```

```
#找出价格最低放入股票
min_price=min(zip(prices.values(),prices.keys()))
print(min_price)


#找出价格最高放入股票
max_price=max(zip(prices.values(),prices.keys()))
print(max_price)

#同样，要对数据排序只要使用zip()再配合sorted()
prices_sorted=sorted(zip(prices.values(),prices.keys()))
print(prices_sorted)
运行结果：

(10.75, 'FB')
(612.78, 'AAPL')
[(10.75, 'FB'), (37.2, 'HPQ'), (45.23, 'ACME'), (205.55, 'IBM'), (612.78, 'AAPL')]
```
注意，zip()创建的迭代器只能被消费一次，例如下面

```
zip_price=zip(prices.values(),prices.keys())
min_price=min(zip_price) #ok
min_price=min(zip_price) #报错
```