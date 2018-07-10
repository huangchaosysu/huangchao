---
layout: single
author_profile: true
title: "Mysql删除数据后释放磁盘空间"
date: 2018-07-10 19:24:53
# toc: true
tags:
  - mysql
  - linux
  - ubuntu
categories:
  - linux
---

Mysql在设计的时候有个这样子的逻辑，对于表里面的数据，使用delete命令删除了以后，这部分数据占据的空间并没有真正释放，而是只是标记删除而已。就是说你用select查询不到了，但是这部分数据还在的。等有新的数据插入表的时候，mysql会用新的数据覆盖掉原来的数据占据的存储空间。因此，你会发现mysql的数据库文件只增不减。

我们只需要运行OPTIMIZE TABLE 命令就可以释放所有已经删除的磁盘空间。这个命令整理mysql产生的数据碎片，达到回收空间的效果

#### 命令介绍：

语法: OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...  

注意: 这个命令并不是所有的情况下都有效， 另外这条命令会锁表    
示例: 
* 查看表大小
```
SELECT TABLE_NAME, (DATA_LENGTH+INDEX_LENGTH)/1048576, TABLE_ROWS FROM information_schema.tables WHERE TABLE_SCHEMA='dbname' AND TABLE_NAME='tablename'; 
```
* 优化<code>optimize table xxxxxxxx;</code>

根据表的大小不同，上面的命令需要的时间也会不同。请耐心等候。


#### 常见错误

上述命令常见的一个错误: <code>Table does not support optimize, doing recreate + analyze instead</code>

对于这个错误，解决办法如下:  

```
alter table test engine='innodb';  # 注意替换engine类型
analyze table xxxx
```

刚刚这组命令其实是重建了一个新表，然后把旧表的数据插入里面, optimize的原理也一样的。
