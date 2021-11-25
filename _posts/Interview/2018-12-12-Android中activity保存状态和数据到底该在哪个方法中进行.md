---
layout:     post
title:      Android中保存状态和数据应该在哪个方法中进行
subtitle:   
date:       2018-12-12
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
    - Interview
---
今天接到一个电面，途中面试官问到一个问题，如果一个activity在后台的时候，因为内存不足可能被杀死，在这之前如果想保存其中的状态数据，比如说客户填的一些信息之类的，该在哪个方法中进行。
 我听到的第一反应就是说：在onPause方法中进行保存状态的操作。但是面试官说：onPause()的持续时间很短，假如要进行一些长时间的操作呢？

然后我就纠结了，因为我知道，如果是因为内存不足而被清理，onDestroy()方法一般是不会被执行的。所以只好实话实说，只知道onDestroy在这种情况下不一定会执行，所以不能在其中做操作。
事后又去看了一下官方文档，发现：对于activity的销毁，有下面这么一个表：

<img src="/img/article/avtivity_killable.jpg"/>


"Killable"表示当前activity是否可以被杀死，意思是说当上面标记为Killable的方法返回之后，activity就可能随时被杀死。从表中不难看出在onPause方法调用完之前，activity都是不能够被杀死的，而onStop()和onDestroy()都是可以被杀死的。但是图中又标出了一个黄色的标记：HONEYCOMB。
官方文章原文是这样说的：Starting with Honeycomb, an application is not in the killable state until itsonStop() has returned. 
从Honeycomb(Android 3.0)开始，应用只有等到onStop()方法返回之后，才可以被杀死，也就是说在执行完onStop()方法之前，应用都不可能被杀死。

you should use the onPause() method to write any persistent data (such as user edits) to storage.
你应该在onPause()方法中去存储那些持久性的数据，比如用户的输入等。

the method onSaveInstanceState(Bundle) is called before placing the activity in such a background state, allowing you to save away any dynamic instance state in your activity into the given Bundle, to be later received in onCreate(Bundle) if the activity needs to be re-created.
onSaveInstanceState(Bundle)将在activity转入“background state后台状态”之前被调用，能让我们存储一些activity的动态的状态值到Bundle对象中，以便在之后调用onCreate(Bundle)方法时用到。

这里提到一个“background state后台状态”，我们来看看activity的几种状态：

前台状态：
通俗的说就是可以看到，且可以操作(有焦点)的状态

可视状态：
即可以看得见(没有被完全遮挡)，但是没有焦点，不可以触摸操作；比如躲在对话框后面的activity

后台状态：
已经看不到了，系统可以将这个进程杀死来回收内存。如果在这种状态下activity被系统杀死了，那么在用户重新打开这个activity的时候，它的onCreate方法会使用之前onSaveInstanceState(Bundle)保存的状态数据，来让自己恢复到之前的状态

空进程状态：
一个没有持有任何activity和任何应用组件的进程，比如Services或者广播接受者，当内存不足的时候，它们将会被先杀死并回收。

也就是说onSaveInstanceState(Bundle)会在activity转入后台状态之前被调用，也就是onStop()方法之前，onPause方法之后被调用；我们都知道在默认情况下，在旋屏之后，activity会重新经历一次生命周期，下面的log就是在点击旋屏之后的执行顺序：

<img src="/img/article/callback_order.jpg"/>

这样看起来，那种要保存的数据，应该在onPause下完成，而activity的一些状态值，比如组件宽高之类的，应该在onSaveInstanceState中保存在Bundle中去。

但是要注意的是，官方文档最后又给出了一个警告：
Note that it is important to save persistent data in onPause() instead of onSaveInstanceState(Bundle) because the latter is not part of the lifecycle callbacks, so will not be called in every situation as described in its documentation.

由于onSaveInstanceState(Bundle)方法不是activity生命周期中的回调方法之一，所以在activity被杀死的时候，它是不能保证百分百的被执行的。。。。

看样子onSaveInstanceState()也靠不住啊，还是在onPause中做吧，面试官也不一定靠谱。

生命周期图：

<img src="/img/article/activity_lifecycle.jpg"/>


转自：http://blog.csdn.net/cyp331203/article/details/44985087
