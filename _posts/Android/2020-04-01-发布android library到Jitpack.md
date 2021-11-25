---
layout:     post
title:      发布Android library到JitPack
subtitle:   
date:       2020-04-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---
作为一名合格的开发者，除了会使用轮子外还要会造轮子，那么造了轮子如何发布呢？这里介绍一种目前最简单的发布方式[JitPack](https://jitpack.io/)

#### 1.添加android-maven 插件

1. 在项目根目录下的build.gradle中添加如下代码

   ```groovy
   buildscript { 
     dependencies {
       classpath 'com.github.dcendents:android-maven-gradle-plugin:2.1' // Add this line
     }
   }
   ```

2. 在library的build.gradle的**最外层**添加如下代码

   ```groovy
   apply plugin: 'com.github.dcendents.android-maven'  
   
   group='com.github.YourUsername'
   ```

#### 2.发布项目到GitHub

android studio中可通过如下步骤发布到GitHub

![android_library1.png](/img/article/android_library1.png)

#### 3.发布版本

1. 打开自己的Github主页，找到这个工程，点releases

   ![android library2 .png](/img/article/android_library2.png)

2. 随便输入一个版本号，点击publish release按钮

   ![android_library3.png](/img/article/android_library3.png)



#### 4.发布项目

打开[jitpack](https://jitpack.io/)，输入github上项目的地址，点击Look up按钮，如下图

![android_library4.png](/img/article/android_library4.png)



至此，项目发布成功，你可以使用上图中的依赖来引用你发布的library了。