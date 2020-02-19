---
layout: single
author_profile: true
title: "mysql Lock wait timeout exceeded"
date: 2020-02-04 10:30:53
toc: true
tags:
  - mysql
categories:
  - mysql
---
请求接口响应时间超长，耗时几十秒才返回错误提示，后台日志中出现Lock wait timeout exceeded; try restarting transaction的错误信息。

场景：

1. 在同一事务内先后对同一条数据进行插入和更新操作；
2. 多台服务器操作同一数据库；
3. 瞬时出现高并发现象；

原因：
1. 在高并发的情况下，Spring事物造成数据库死锁，后续操作超时抛出异常。
2. Mysql数据库采用InnoDB模式，默认参数:innodb_lock_wait_timeout设置锁等待的时间是50s，一旦数据库锁超过这个时间就会报错

解决方法：

1.查看数据库当前的进程，看一下有无正在执行的慢SQL记录线程。

```
mysql> show  processlist;
```
2.查看当前的事务-当前运行的所有事务

```
mysql> SELECT * FROM information_schema.INNODB_TRX;
```
3.当前出现的锁
```
mysql> SELECT * FROM information_schema.INNODB_LOCKs;
```
4.锁等待的对应关系
```
mysql> SELECT * FROM information_schema.INNODB_LOCK_waits;
```
解释：看事务表INNODB_TRX，里面是否有正在锁定的事务线程，看看ID是否在show processlist里面的sleep线程中，如果是，就证明这个sleep的线程事务一直没有commit或者rollback而是卡住了，我们需要手动kill掉。

搜索的结果是在事务表发现了很多任务，这时候最好都kill掉。

5.批量删除事务表中的事务

我这里用的方法是：kill 线程ID

kill掉以后再执行SELECT * FROM information_schema.INNODB_TRX; 就是空了。

接口就能正常的访问。

 

暴力方法是把<code>show processlist</code>列出来都所有进程都kill掉
在mysql终端内执行kill {id}

后面会不定时为大家更新文章，敬请期待。

喜欢的朋友可以关注下。