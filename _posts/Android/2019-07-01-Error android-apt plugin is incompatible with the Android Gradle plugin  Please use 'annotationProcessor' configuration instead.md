---
layout:     post
title:      Error:android-apt plugin is incompatible with the Android Gradle plugin. Please use 'annotationProcessor' configuration instead.
subtitle:   
date:       2019-07-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

有一天，android studio推送了更新，更新后发现项目报错如下：
```java 
Error:android-apt plugin is incompatible with the Android Gradle plugin. Please use 'annotationProcessor' configuration instead.
```
原因是最新版Android Studio所搭配的com.android.tools.build:gradle:3.0.0及更高版本不支持目前1.8版本的apt了，所以，解决方案如下：   
##### 1.把module/build.gradle下的apt插件代码删除
<img src="/img/article/apt1.png"/>

##### 2.把dependencies下的apt全部改为annotationProcessor
<img src="/img/article/apt2.png"/>

##### 3.把project/build.gradle中的apt插件声明删除
<img src="/img/article/apt3.png"/>
