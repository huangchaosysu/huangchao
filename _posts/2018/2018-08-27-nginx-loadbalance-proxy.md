---
layout: single
author_profile: true
title: "nginx负载均衡与反向代理配置"
date: 2018-08-27 14:24:53
# toc: true
tags:
  - linux
  - nginx
categories:
  - linux
---

### 负载均衡

nginx 的 upstream目前支持 4 种方式的分配 
1. 轮询（默认）   
每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。 
2. weight  
指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。 
3. ip_hash  
每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。  
4. fair（第三方）  
按后端服务器的响应时间来分配请求，响应时间短的优先分配。  
5. url_hash（第三方）

配置示例：

```
http {
  xxxxx
  upstream tomcat_pool 
  {
    server 192.168.0.223:8080 weight=4 max_fails=2 fail_timeout=30s;
    server 192.168.0.224:8080 weight=4 max_fails=2 fail_timeout=30s;
    # iphash;  # 如果采用iphash方法，那么取消上面的weight参数， 增加这一行
  }

  server 
  {
      listen       80;        #监听端口    
      server_name  localhost;
  
  #默认请求设置
  location / {
      proxy_pass http://tomcat_pool;    #转向tomcat处理
  }
  xxxxx
}
```

max_fails这个参数表示如果fail_timeout这么长的时间内请求失败的数码超过这个值,那么nginx就把对应的服务标记为不可用,不可用的时间为fail_timeout定义的时常


### 反向代理

配置示例: 

```
http {
  xxxxx
  location /lile {
    配置一： proxy_pass  http://192.168.0.37/;
    配置二： proxy_pass  http://192.168.0.37;
  } 
  xxxxx
}
```

注意有没有最后的/是有区别的

当proxy_pass为：http://192.168.0.37  的时候，返回的数据如下：  
1. 浏览器请求访问http://192.168.0.224/lile/  
2. 到达192.168.0.224后，location /lile 匹配到之后，转发的地址为：http://192.168.0.37/lile/  
3. 然后到达192.168.0.37，匹配到了location /lile，所以就去/data目录下取数据  

当proxy_pass为： http://192.168.0.37/  的时候，返回的数据如下：    
1. 浏览器请求访问http://192.168.0.224/lile/   
2. 达192.168.0.224后，location /lile 匹配到之后，转发的地址为：http://192.168.0.37/，这里在proxy_pass的 http://192.168.0.37/ 的“/”会把/lile给替换掉  
3. 然后到达192.168.0.37，直接匹配到的是root /web1，所以就去/web1目录下取数据 