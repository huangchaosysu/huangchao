---
layout: single
author_profile: true
title: "python爬虫入门-03-分布式爬虫"
date: 2018-10-24 10:30:53
# toc: true
tags:
  - python
  - 爬虫
categories:
  - python
  - python爬虫
---


上一篇介绍的通用爬虫架构不同，下面是一个聚焦爬虫的架构图，与前者相比，它不仅要保存网页，还要提取出网页中的指定内容。

![](/assets/images/spider/spider05.jpg)

Crawler_core  从任务队列获取爬虫任务，请求网页并将其存储到Mongodb，同时解析出网页中的URLs并缓存到Redis。最后通知Common-clean-platform抽取网页的指定字段。

Common-clean-platform  收到Crawler_core的通知后，从Mongodb中取出网页，根据配置进行数据抽取，形成结构化的数据，保存到Mongodb。

Scheduler_manager  负责任务调度（如启停），状态控制（如爬取数量），redis资源清理等。

Resource_manager  封装Mysql、Mongodb、Redis接口。Mysql存储任务基本信息、配置文件、任务实时状态等。Mongodb存储网页、结构化数据。Redis缓存队列、集合等数据结构。

Proxy  代理服务器。建立网络爬虫的第一原则是：所有信息都可以伪造。你可以用非本人的邮箱发送邮件，或者通过命令行自动化鼠标的行为。但是有一件事情是不能作假的，那就是你的IP地址。如果你在爬取的过程中不想被人发现，或者不想IP被封杀，那么就需要使用代理。


### 流程控制 – 任务

Scheduler_manager定时读取存储在某个地方的任务信息，根据任务的周期等配置进行调度，下面是一个最基本的任务启停流程。

![](/assets/images/spider/spider06.jpg)

* 当一个任务可以开始时，Scheduler_manager会将其基本信息(包括task_id，种子url，启动时间等)插入Reids的任务队列中。如果任务结束，就从队列中删除。
* 每个Crawler_core实例定时读取Redis任务队列中的任务，插入到本地的内存任务队列中。
* 相同的任务可以存在于不同的Crawler_core实例中，一个Crawler_core实例中也可以有相同的任务。
* Crawler_core的抓取线程从内存队列中获得可执行的任务，依次抓取和解析。


### 流程控制 - 数据

现在每个Crawler_core实例都有了待处理的任务，接下来就要对每个任务的url进行处理了。继续使用Redis作为公共缓存来存放URLs，多个Crawler_core实例并行存取todo集合。

![](/assets/images/spider/spider07.jpg)

* Todo集合  Crawler_core从集合中取出url进行处理，并将解析得到的url添加到todo集合中。
* Doing集合  Crawler_core从todo中取出url，都同时保存到doing中，url处理完成时被从doing中删除。主要用于恢复数据。
* Parser todo队列  Crawler_core将已经保存到mongodb的页面的url发送到parser todo队列，Parser读取数据后进行解析。
* Filter todo队列  Parser将已经保存到mongodb的解析结果的url发送到filter todo队列，Filter读取数据后进行清洗。

### 流程控制 – failover

如果一个Crawler_core的机器挂掉了，就会开始数据恢复程序，把这台机器所有未完成的任务恢复到公共缓存中。

![](/assets/images/spider/spider08.jpg)

1. 监控到192.168.0.1心跳停止。
2. Master遍历正在运行的任务: task_jdjr:1489050240345等。
3. 得到doing和todo集合：
    1. task_jdjr:1489050240345:crawler:doing: 192.168.0.1
    2. task_jdjr:1489050240345:crawler:todo
4. 将doing中的数据全部移动到todo中。