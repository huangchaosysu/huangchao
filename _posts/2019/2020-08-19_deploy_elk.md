---
layout: single
author_profile: true
title: "virtualbox部署elk cluster"
date: 2020-08-19 10:30:53
toc: true
tags:
  - flutter
categories:
  - flutter
---

### Step1 

在virtualbox上安装3个虚拟机， 我这里用的ubuntu

### Step2

参考文档， 在每个虚拟机上安装elasticsearch https://www.elastic.co/guide/en/elasticsearch/reference/7.8/deb.html

### Step3

#### 配置 master 节点 /etc/elasticsearch/elasticsearch.yml
```
cluster.name: my-cluster
node.name: master-node
node.data: false
node.master: true
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 0.0.0.0
http.port: 9200
discovery.zen.minimum_master_nodes: 1
discovery.seed_hosts: ["192.168.0.106", "192.168.0.110", "192.168.0.111"]
cluster.initial_master_nodes: ["192.168.0.106"]  # master ip

xpack.monitoring.enabled: true
xpack.monitoring.collection.enabled: true
xpack.monitoring.collection.interval: 10m
xpack.monitoring.elasticsearch.collection.enabled: true

xpack.security.enabled: true  # 开启认证
```

```
sudo service elastic start
```


#### 配置认证

参考 https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html

### 安装kibana

参考 https://www.elastic.co/guide/en/kibana/current/deb.html#deb-running-systemd


### 配置 data 节点 /etc/elasticsearch/elasticsearch.yml

```
cluster.name: my-cluster
node.name: data-node-1
node.master: false
node.data: true
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 0.0.0.0
http.port: 9200
discovery.zen.ping.unicast.hosts: [ "192.168.0.106" ]

xpack.monitoring.enabled: true
xpack.monitoring.collection.enabled: true
xpack.monitoring.collection.interval: 10m
xpack.monitoring.elasticsearch.collection.enabled: true
```

### 检查集群状态

```
curl 192.168.0.106:9200/_cluster/health?pretty
```

示例结果
```
{
  "cluster_name" : "my-cluster",
  "status" : "green",  
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 0,  
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```