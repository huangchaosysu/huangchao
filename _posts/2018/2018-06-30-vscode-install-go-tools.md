---
layout: single
author_profile: true
title: "vscode配置golang环境"
date: 2018-06-30 13:24:53
toc: true
tags:
  - vscode
  - golang
  - linux
categories:
  - linux
---

### 安装golang
请参考官方的[安装教程](https://golang.org/doc/install)

### 安装vscode
直接到vscode的官网[下载](https://code.visualstudio.com/)

### 安装在vscode的扩展那里安装golang插件

    @sort:installs go

第一个就是我们要的插件了。点击安装

### 安装工具
执行这一步前先安装Git，并设置好环境变量GOPATH，GOPATH是你的工作目录，你想在哪个目录写代码就配置成哪个目录。
后面的工具也会安装在这个目录下
配置好之后，在cmd环境下执行下面几个命令（执行下面命令的前提是安装Git和配置好GOPATH，否则会报错）

下面的是微软官方文档的脚本

```
go get -u -v github.com/nsf/gocode
go get -u -v github.com/rogpeppe/godef
go get -u -v github.com/golang/lint/golint
go get -u -v github.com/lukehoban/go-outline
go get -u -v sourcegraph.com/sqs/goreturns
go get -u -v golang.org/x/tools/cmd/gorename
go get -u -v github.com/tpng/gopkgs
go get -u -v github.com/newhook/go-symbols
go get -u -v golang.org/x/tools/cmd/guru
```

实际上为了使用F5调试，我们还需要dlv工具
go get -v -u github.com/peterh/liner github.com/derekparker/delve/cmd/dlv

详细的安装信息和扩展能干什么，可以参看微软的官方文档
https://marketplace.visualstudio.com/items?itemName=lukehoban.Go

调试运行时mode参数可以被设置为：

在国内安装go的官方扩展的时候，经常会出现golang.org访问不了的情况。这个帖子就是叫你如何解决这些问题。

### 方法一, VPN翻墙
链接VPN的方式能够实现全局翻墙，就不存在golang.org访问不了的情况

### 方法二，手动安装
在GOPATH这个变量的目录下，建立如下目录

    $GOPATH/src/golang.org/x
进到该目录，git clone https://github.com/golang/tools.git tools  
把其他你要安装的官方扩展也顺道一起clone下来  
把上面的go get xxx的命令改为go install xxx就可以安装了
