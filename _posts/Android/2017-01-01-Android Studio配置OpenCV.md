---
layout:     post
title:      android studio 配置openCV
subtitle:   
date:       2017-01-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

## android studio 配置openCV的方法：   

### 1、在http://opencv.org/  官网上下载OpenCV-android-sdk     

### 2、解压下载的文件：              
sdk目录即是我们开发opencv所需要的类库；                 
samples目录中存放着若干opencv应用示例；                      
apk目录则存放着对应于各内核版本的OpenCV_x_Manager_x应用安装包。此应用用来管理手机设备中的opencv类库，在运行opencv应用之前，
必须确保手机中已经安装了OpenCV_3.2.0_Manager_3.2_*.apk，否则opencv应用将会因为无法加载opencv类库而无法运行；如果实在不想安装
管理opencv类库的安装包，则需要在app启动时添加如下静态代码块就可：
```java
static{
        System.loadLibrary("opencv_java3");
    }
```

### 3、将OpenCV加入到AndroidStudio中：                                                 
在Android Studio中选择File->New->Import Module，找到OpenCV解压的路径，选择sdk/java文件夹。     

### 4、添加Module Dependency：              
  使用快捷键Shift+Ctrl+Alt+s  或者直接右击app->Open Module Settings 打开下面对话框选择在app module                
的Dependencies一栏将openCVLibrary添加进去         

### 5、在app-->src-->main目录下，新建文件夹jniLibs  并将OpenCV-android-sdk 解压包下面的sdk->native->libs文件夹下面的文件拷贝到jniLibs文件下。
（我只拷贝了armeabi-v7a文件夹）
