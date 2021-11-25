---
layout:     post
title:      Android 四大组件之Service
subtitle:   
date:       2018-12-12
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
    - Interview
---
Service是Android四大组件中与Activity最相似的组件，它们都代表可执行的程序，Service与Activity的区别在于：Service一直在后台运行，
它没有用户界面，所以绝对不会到前台来。一旦Serivce被启动，它就与Activity一样。它完全具有自己的生命周期。

关于程序中Activity和Service的选择标准是：如果某个程序组件需要在运行时向用户呈现某种界面或者需要与用户交互，就需要使用Activity，
否则就应该考虑使用Service。

#### 1、Service启动方法与区别
  Service有2种启动方式：                
    a、通过Context的startService()方法：通过该方法启动的Service，访问者与Service之间没有关联，因此Service和访问者之间无法
    进行通讯和数据交换，而且即使访问者退出了，Service仍然运行。               
    b、通过Context的bindService()方法：使用该方法启动Service，访问者与Service绑定在了一起，访问者一旦退出，Service也就终止了，
    如果Service和访问者之间需要进行方法调用或数据交换，应选用此方式启动Service。
    
#### 2、Service的生命周期               
    a：startService 
    Service的生命周期：onCreate() --> onStartCommand() -> onDestroy()
    停止服务：stopService()        
    start多次，onCreate只会被调用一次，onStart会调用多次
    
    b：bindService
    Service的生命周期 onCreate() --> onBind()  --> onUnBind() --> onDestroy()
    停止服务：UnbindService()  
    不管调用bindService几次，onCreate只会调用一次，onStart不会被调用，建立连接后，service会一直运行，直到调用unBindService或是之前调用的bindService的Context不存在了，系统会自动停止Service,对应的onDestory会被调用
    
#### 3、Service和Activity的通讯方式
    a、通过Intent
    startService（intent）来启动Service，在intent中放入数据，在Service的onStartCommant()中接收通过intent传过来的值。（性能差）
    b、binder+回调
    在Activity中实现ServiceConnection，在onServiceConnected()中获取Service的实例，通过这个实例就能调用Service的方法和变量了。
    通过回调可以将Service主动将变化通知Activity。
    c、Broadcase方式
    在Service中需要通知更新UI的地方，发送广播，在Activity中注册广播，在BroadcaseRecever中接受广播，更新UI。
    d、EventBus或RxBus
