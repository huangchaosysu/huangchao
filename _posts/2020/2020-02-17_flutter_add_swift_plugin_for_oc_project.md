---
layout: single
author_profile: true
title: "add swift plugin for objectivd-c project"
date: 2020-02-17 10:30:53
toc: true
tags:
  - flutter
categories:
  - flutter
---
### Flutter编译错误， 输出如下
```
Xcode's output:
↳
    === BUILD TARGET Runner OF PROJECT Runner WITH CONFIGURATION Debug ===
    ld: warning: Auto-Linking library not found for -lswiftSwiftOnoneSupport
    ld: warning: Auto-Linking library not found for -lswiftCoreAudio
    ld: warning: Auto-Linking library not found for -lswiftCore
    ld: warning: Auto-Linking library not found for -lswiftQuartzCore
    ld: warning: Auto-Linking library not found for -lswiftCoreImage
    ld: warning: Auto-Linking library not found for -lswiftCoreGraphics
    ld: warning: Auto-Linking library not found for -lswiftAVFoundation
    ld: warning: Auto-Linking library not found for -lswiftCoreMedia
    ld: warning: Auto-Linking library not found for -lswiftDispatch
    ld: warning: Auto-Linking library not found for -lswiftObjectiveC
    ld: warning: Auto-Linking library not found for -lswiftCoreFoundation
    ld: warning: Auto-Linking library not found for -lswiftUIKit
    ld: warning: Auto-Linking library not found for -lswiftDarwin
    ld: warning: Auto-Linking library not found for -lswiftMetal
    ld: warning: Auto-Linking library not found for -lswiftsimd
    ld: warning: Auto-Linking library not found for -lswiftFoundation
    Undefined symbols for architecture x86_64:
```

### 解决办法
xcode Runner项目跟目录

1) File -> New -> File
![我的头像](/assets/images/posts/f1.jpg)

2) 选择swift file
![我的头像](/assets/images/posts/f2.png)

3) Confirm Create Bridging Header
![我的头像](/assets/images/posts/f3.png)

4) Go to Build Settings and set Always Embed Swift Standard Libraries to YES
![我的头像](/assets/images/posts/f4.png)

5) 重新编译


其他参考：

```
Plugin side
[!] Unable to determine Swift version for the following pods
TL;DR: Add s.swift_version = '5.0' to ios/your_project.podspec.
New Swift plugin podspec are generated with s.swift_version = '5.0' as of #44324. Existing Swift plugin authors should add this line to ios/your_project.podspec.
Example freshly generated ios/test_swift_plugin.podspec:

#
# To learn more about a Podspec see http://guides.cocoapods.org/syntax/podspec.html.
# Run `pod lib lint test_swift_plugin.podspec' to validate before publishing.
#
Pod::Spec.new do |s|
  s.name             = 'test_swift_plugin'
  s.version          = '0.0.1'
  s.summary          = 'A new flutter plugin project.'
  s.description      = <<-DESC
A new flutter plugin project.
                       DESC
  s.homepage         = 'http://example.com'
  s.license          = { :file => '../LICENSE' }
  s.author           = { 'Your Company' => 'email@example.com' }
  s.source           = { :path => '.' }
  s.source_files = 'Classes/**/*'
  s.dependency 'Flutter'
  s.platform = :ios, '8.0'

  # Flutter.framework does not contain a i386 slice. Only x86_64 simulators are supported.
  s.pod_target_xcconfig = { 'DEFINES_MODULE' => 'YES', 'VALID_ARCHS[sdk=iphonesimulator*]' => 'x86_64' }
  s.swift_version = '5.0'
end
If you have tested on versions of Swift other than 5.0, feel free to change this.
s.swift_versions = [ '4.0', '4.2', '5.0', '5.1' ]

fatal error: 'test_swift_plugin/test_swift_plugin-Swift.h' file not found
TL;DR: Update ios/Classes/YourPlugin.m to the following import snippet of Objective-C code.

New Swift plugins Objective-C implementation file have a new import pattern as of #44324 to support being used as libraries instead of frameworks so your users won't need use_frameworks!. Existing Swift plugin authors should update this import.
Example freshly generated ios/Classes/TestSwiftPlugin.m

#import "TestSwiftPlugin.h"
#if __has_include(<test_swift_plugin/test_swift_plugin-Swift.h>)
#import <test_swift_plugin/test_swift_plugin-Swift.h>
#else
// Support project import fallback if the generated compatibility header
// is not copied when this plugin is created as a library.
// https://forums.swift.org/t/swift-static-libraries-dont-copy-generated-objective-c-header/19816
#import "test_swift_plugin-Swift.h"
#endif

@implementation TestSwiftPlugin
...
Why is there a thin Objective-C wrapper class by default in Swift plugins?
❓❓❓
TL;DR: Don't worry about it
Apps had generated code to import plugin Objective-C headers instead of importing the clang module. See #41007 and https://github.com/flutter/flutter/pull/42204/files#r333285661
Once you feel confident all your app customers are on 1.10.15 or later and you have the Swift-foo, you can try to remove these wrapper Objective-C files. Please test your plugin on a fresh Objective-C flutter app before you publish to see how it's being used!

Objective-C app-side
Undefined symbol: _swift_ ...
TL;DR: Add use_framework! to your app's ios/Podfile
This is the result of Objective-C apps not being hooked up to look for Swift symbols at link time. You can fix this on the app side by using use_framework!, by adding a single dummy Swift file to tell Xcode to hook all that up, or (if you have the Xcode know-how) by adding $(TOOLCHAIN_DIR)/usr/lib/swift-5.0/$(PLATFORM_NAME) to LIBRARY_SEARCH_PATHS.

Undefined symbol: __swift_FORCE_LOAD_$_swiftCompatibilityDynamicReplacements
TL;DR: Add use_framework! to your app's ios/Podfile
Pretty much the same as the above, but on Swift 5. You can fix this on the app side by using use_framework!, by adding a single dummy Swift file to tell Xcode to hook all that up, or (if you have the Xcode know-how) by adding $(TOOLCHAIN_DIR)/usr/lib/swift/$(PLATFORM_NAME) to LIBRARY_SEARCH_PATHS.

This copy of libswiftCore.dylib requires an OS version prior to 12.2.0.
TL;DR: Add use_framework! to your app's ios/Podfile
On iOS >12.2 the Swift runtime is not embedded in the app, but is baked into the OS. You can fix this on the app side by using use_framework!, by adding a single dummy Swift file to tell Xcode to hook all that up, or (if you have the Xcode know-how) by adding /usr/lib/swift before $(inherited) in LD_RUNPATH_SEARCH_PATHS .

Why is there an Objective-C GeneratedPluginRegistrant wrapper class by default in Swift apps?
❓❓❓
TL;DR: Don't worry about it
Objective-C plugins were not defining clang modules, so could not be imported from Swift files. This was fixed for new Objective-C modules in #41828.

Summary
Swift apps should now be able to use (new or migrated) Swift and Objective-C plugins (on master channel at the moment)
Objective-C apps should be able to use (new or migrated) Swift plugins as frameworks (Podfile contains use_frameworks!)
Objective-C apps still cannot use Swift plugins as libraries (Podfile does not contain use_frameworks!), which is the default for a newly created Objective-C Flutter project. I'm going to use this issue to track these outstanding issues.
```