---
layout:     post
title:      Android 四大组件之BroadcastReceiver
subtitle:   
date:       2018-12-12
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
    - Interview
---
BroadcastReceiver本身就是一种全局监听器，用于监听系统全局的广播消息，因此它可以很方便地实现系统中不同组件之间的通信。例如：
我们希望客户端程序与startService()方法启动的Service之间通信，就可以借助BroadcastReceiver来实现。

#### 1、广播分类                        
    Broadcast分为2种：                            
      a、Normal Broadcast(普通广播)：                   
      Normal Broadcast是完全异步的，可以在同一时刻（逻辑上）被所有接收者接收到，消息的传递效率比较高。
      但缺点是接收者不能将处理的结果传递给下一个接收者，并且无法终止Broadcast Intent的传播。
      b、Ordered Broadcast（有序广播）：                        
      Ordered Broadcast的接收者将按照预先声明的优先级依次接收Broadcast。例：
      A的级别高于B,B高于C，那么Broadcast先传给A,再传给B，最后传给C。优先级别声明在<intent-filter.../>元素的
      android：priority属性中，数越大，优先级别越高，取值范围为-1000 ~ 1000，优先级别也可以调用IntentFilter
      对象的setPriority()进行设置。Ordered Broadcast接收者可以终止Broadcast Intent的传播，一旦终止，
      后面的接收者就无法接收到Broadcast。另外，Ordered Broadcast的接收者可以将数据传递给下一个接收者，
      如：A得到Broadcast后，可以往它的结果对象中存入数据，当Broadcast传给B时，B可以从A的结果对象中得到A存入的数据。
      
      Context提供如下2个方法发送广播：
      sendBroadcast():发送Normal Broadcast
      sendOrderedBroadcast():发送OrderedBroadcast
      
      
#### 2、广播使用的场景： 
    a、同一app内部的同一组件内的消息通信（单个或多个线程之间）； 
    b、同一app内部的不同组件之间的消息通信; 
    c、不同app之间的组件之间消息通信； 
    d、Android系统在特定情况下与App之间的消息通信(例：气质仪中usbhost)。
    
#### 3、广播的使用方式：                           
    a、静态注册(常驻型广播)：
    在AndroidManifest.xml注册，android不能自动销毁广播接收器，也就是说当应用程序关闭后，还是会接收广播。 
    b、动态注册(非常驻型广播)：
    在代码中通过registerReceiver()手工注册.当程序关闭时,该接收器也会随之销毁。当然，也可手工调用unregisterReceiver()进行销毁。
