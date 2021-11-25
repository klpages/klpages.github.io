---
layout:     post
title:      Android Killer反编译apk过程中遇到的一些问题
subtitle:   
date:       2018-12-12
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

#### 1.正在反编译APK源码，请稍等......
<img src="/img/article/androidkillerproblem1.jpg"/>
解决方法：  

关闭Androidkiller 再重新打开 选择对应的最近的项目中双击打开即可 就可以成功了。   
或者可以按此方法解决：https://www.gudanba.com/639.html

#### 2.APK 编译失败,无法继续下一步签名 
```
>        ... 11 more
APK 编译失败，无法继续下一步签名!
```
解决方法：   
Apktool Framework下的1.apk版本太老旧，删除即可。  
这个1.apk正常在路径为:C:\Users\Administrator\apktool\framework  ；如果不对可进行全盘收索  

#### 3.APK 反编译失败，无法继续下一步源码反编译!
```
当前 Apktool 使用版本：Android Killer Default APKTOOL
正在反编译 APK，请稍等...
>I: 使用 ShakaApktool 2.0.0-20150914
>I: 正在加载资源列表...
>I: 反编译 AndroidManifest.xml 与资源...
>I: 正在从框架文件加载资源列表: C:\Users\apktool\framework\1.apk
>I: 常规资源列表...
>I: 反编译资源文件...
>I: 反编译 values */* XMLs...
>Exception in thread "main" b.a.a.e: resource spec: 0x01010543
>   at b.a.d.a.p.a(Unknown Source)
>   at b.a.d.a.q.a(Unknown Source)
>   at org.c.b.b.c.a(Unknown Source)
>   at com.rover12421.shaka.a.b.p.a(Unknown Source)
>   at b.a.d.a.p.b(Unknown Source)
>   at b.a.d.a.w.a(Unknown Source)
>   at b.a.d.a.w.a(Unknown Source)
>   at b.a.d.a.a.t.d(Unknown Source)
>   at b.a.d.a.a.t.a(Unknown Source)
>   at b.a.d.a.a.u.h(Unknown Source)
>   at b.a.d.a.a.w.a(Unknown Source)
>   at b.a.d.a.a.w.a(Unknown Source)
>   at b.a.d.a.a(Unknown Source)
>   at b.a.d.a.c(Unknown Source)
>   at b.a.a.b(Unknown Source)
>   at b.a.E.a(Unknown Source)
>   at b.b.a.a(Unknown Source)
>   at b.b.a.a(Unknown Source)
>   at com.rover12421.shaka.cli.Main.main(Unknown Source)
APK 反编译失败，无法继续下一步源码反编译!
```
解决方法：  
由于本地ShakaApkTool版本太低，需要apktool。 更新方式如下：
- Step1  
下载最新版apktool，放到androidkiller安装目录下bin/apktool文件夹下

- Step2  
修改apktool文件夹下apktool.bat文件中apktool的名称为你下载的名字  
<img src="/img/article/androidkillerproblem3.png"/>  

- Step3  
切换到Android 面板，点击apktool管理器，添加你下载的最新版apktool  
<img src="/img/article/androidkillerproblem2.png"/>   
