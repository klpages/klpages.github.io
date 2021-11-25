---
layout:     post
title:      Android 四大组件之ContentProvider
subtitle:   
date:       2018-12-12
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
    - Interview
---
ContentProvider是不同应用程序之间进行数据交换的标准API,ContentProvider以某种Uri的形式对外提供数据，允许其他应用访问或修改数据；
其他应用程序使用ContentResolver根据Uri去访问操作指定数据。                
ContentProvider为Android四大组件之一，与Activity、Service、BroadcastReceiver相似，都需要在AndroidManifest.xml文件中进行配置。

#### 1、ContentProvider的使用方法
    和开发Activity、Service类似，需要2个步骤：
    a、开发contentProvider子类
    定义自己的contentprovider类，该类需要继承ContentProvider
    b、注册contentProvider
    在AndroidManifest.xml文件中注册该contentProvider，就像注册Activity一样。
    注册contentprovider时需要为它绑定一个Uri(andorid:authorities)。
    
#### 2、ContentProvider、ContentResolver、ContentObserver 之间的关系
      contentProvider：
      内容提供者，主要用于对外提供数据
      contentResolver：
      内容解析者，用于获取内容提供者提供的数据              
      ContentResolver.notifyChange（uri）发出消息
      contentObserver：
      内容监听器，可以监听contentProvider数据的改变
      ContentResolver.registerContentObserver()监听消息
