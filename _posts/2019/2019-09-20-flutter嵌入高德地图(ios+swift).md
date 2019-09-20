---
layout: single
author_profile: true
title: "flutter编写高德地图插件(ios)"
date: 2019-09-20 10:30:53
toc: true
tags:
  - flutter
  - ios
categories:
  - flutter
---

### 概述
这边文章主要讲解如何在flutter中引用高德地图（iOS）， 用到了flutter的插件机制。

### 准备工作
1. 安装flutter
2. 安装xcode， cocoapod

以上两步参考[官方教程](https://flutterchina.club/setup-macos/)

### 创建插件项目

    flutter create --template=plugin -i swift -a kotlin amap_swfit

1. <code>--template</code>表明这是一个flutter插件项目
2. -i swift指定这个插件项目ios端是使用swift语言
3. -a kotlin指定这个插件项目的android端使用kotlin语言

执行完以后的项目目录大概是这样的
```
amap_swift
  --android  //Flutter插件的Android端代码在这里
  --example  //插件的示例代码
    --android
    --build
    --ios
    --lib
    --test
    ...
  --ios  //Flutter插件的ios相关代码在这里
    --Assets
    --Classes
  --lib
    --amap_swift.dart  //插件的flutter端实现在这里
  --test
```

### 插件实现-swift
1. 修改example/ios/Runner/Info.plist, 添加一下配置项, 以启用flutter的UiKitView
    ```
    <key>io.flutter.embedded_views_preview</key>
    <true/>
    ```
2. 修改ios/amap_swift.podspec, 添加一下参数
    ```
    s.dependency 'AMap3DMap'    //引入高德地图
    s.static_framework = true   //编译需要
    ```
3. 使用xcode打开example/ios/Runner.xcworkspace

file -> Workspace Settings -> Build System 修改为Legacy Build System

4. 编写View

编辑ios/Classes/AmapSwiftPlugin.h, 添加以下内容, 作用是引入高德地图的组件
```
#import <MAMapKit/MAMapKit.h>
#import <AMapFoundationKit/AMapFoundationKit.h>
```

ios/Classes目录下新建SwiftAmapView.swift
```
import Flutter
import Foundation

class SwiftAMapView : NSObject,FlutterPlatformView{
    //Todo： Flutter插件里面的View必须满足 FlutterPlatformView 协议的要求
    let frame: CGRect;
    let viewId: Int64;
    let amapView: MAMapView;
    var messenger: FlutterBinaryMessenger!
    
    init(_ frame: CGRect,viewID: Int64,args :Any?, binaryMessenger: FlutterBinaryMessenger) {
        self.frame = frame;
        self.viewId = viewID;
        self.messenger=binaryMessenger;
        self.amapView = MAMapView(frame: self.frame);  //初始化地图
    }
    
    
    func initMethodChannel(){
        let amapChannel = FlutterMethodChannel.init(name: "amap_swift", binaryMessenger: messenger);  //Flutter与原生代码通信用的消息通道
        amapChannel.setMethodCallHandler({
            (call: FlutterMethodCall, result: FlutterResult) -> Void in
            if(call.method == "startLoaction"){
                //do something
            }else if(call.method == "stopLoaction"){
                
            }
        });
    }
    
    //FlutterPlatformView这个协议的view方法的重写， 系统调用这个方法来生产ui组件
    func view() -> UIView {
        initMethodChannel()
        return self.amapView
    }
}
```

5. 编写ViewFactory

新建SwiftAMapViewFactory.swift, 这是个View的工厂， 负责View的实例化

```
class SwiftAMapViewFactory : NSObject,FlutterPlatformViewFactory{
    var messenger: FlutterBinaryMessenger!
    
    func create(withFrame frame: CGRect, viewIdentifier viewId: Int64, arguments args: Any?) -> FlutterPlatformView {
        return SwiftAMapView(frame,viewID : viewId , args : args,binaryMessenger:messenger);
    }
    
    @objc public init(messenger: (NSObject & FlutterBinaryMessenger)?) {
        super.init()
        self.messenger = messenger
    }
}
```

6. 编辑SwiftAmapSwiftPlugin.swift

这个文件是开头创建项目的flutter命令自动生成的

```
import Flutter
import UIKit

public class SwiftAmapSwiftPlugin: NSObject, FlutterPlugin {
  public static func register(with registrar: FlutterPluginRegistrar) {
    let channel = FlutterMethodChannel(name: "amap_swift", binaryMessenger: registrar.messenger())
    let instance = SwiftAmapSwiftPlugin()
    registrar.addMethodCallDelegate(instance, channel: channel)

    // regist view factory into flutter， 添加下面2行代码
    let messenger = registrar.messenger() as! (NSObject & FlutterBinaryMessenger)
    registrar.register(SwiftAMapViewFactory(messenger: messenger), withId: "AmapView")
    // 注意上面的withId参数， 这个参数就是UIKitView里面的view_type参数
  }

  public func handle(_ call: FlutterMethodCall, result: @escaping FlutterResult) {
    result("iOS " + UIDevice.current.systemVersion)
  }
}
```


### 插件实现-flutter（dart）
编辑lib/amap_swift.dart

```
import 'dart:async';

import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

// 自动生成的代码， 注释或删除
<!-- class AmapSwift{
  static const MethodChannel _channel = const MethodChannel('amap_swift');

  static Future<String> get platformVersion async {
    final String version = await _channel.invokeMethod('getPlatformVersion');
    return version;
  }
} -->

//Flutter组件
class SwiftAmap extends StatelessWidget {
    @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Scaffold(
      body: UiKitView(
        viewType: "AmapView",
        creationParamsCodec: const StandardMessageCodec(),
      ),
    );
  }
}
```


### 运行
1. 运行一遍flutter run命令
2. xcode下编译运行

### 编译错误处理
1. 编译过程中如果遇到提示info.plist缺少某个参数的错误， 修改example/ios/Runner/Info.plist对应的参数值， 原因大概为缺少环境变量， 可以修改为固定值