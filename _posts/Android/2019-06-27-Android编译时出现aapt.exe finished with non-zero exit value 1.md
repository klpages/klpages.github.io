---
layout:     post
title:      Android编译报错''finished with non-zero exit value 1
subtitle:   
date:       2019-06-27
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---
Android Studio编译项目时经常会遇到如下错误：
```java
Process 'command 'C:\Users\Administrator\AppData\Local\Android\Sdk\build-tools\27.0.0\aapt.exe'' finished with non-zero exit value 1
```
解决方案可以归纳为如下几种：

- Android Studio 问题，重启后打开就好
- Clean、Rebuild 缓存问题
- SDK 版本问题
- 项目中各种包之间版本互斥问题
- 资源文件问题  用的图片应该是png而不是jpg
- Build.Gradle 文件里加几句代码...

如果上述方法不好定位错误的话，可以在android studio 的终端(底部的Terminal)中输入如果代码(PS：gradlew命令在项目根目录下才有效，如果进入到module中是无效的)：    
```java
gradlew processDebugResources --debug
```
然后将终端输出的信息复制到文本框中搜索“aapt”,因为报错是在aapt里，所以我们重点关注aapt的信息,然后你会看到错误文件的路径及位置。


更多可参考:[完美解决Android编译时出现aapt.exe finished with non-zero exit value 1 吐血整理](http://evassmat.com/8GOa)
