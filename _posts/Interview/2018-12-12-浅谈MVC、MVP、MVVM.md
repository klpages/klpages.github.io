---
layout:     post
title:      浅谈MVC、MVP、MVVM
subtitle:   
date:       2018-12-12
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
    - Interview
---
## **MVC** ##
mvc即模型（model）——— 视图（view）——— 控制器（controller），将应用程序在宏观上分为3种职责。

#### **Model** ####
model模型，负责数据逻辑的处理，比如数据库存取操作，网络操作都在model层处理

#### **View** ####
View视图对应为xml文件，是Model的展现，负责呈现UI并在用户与应用程序交互时与Controller通信。
#### **Controller** ####
controller对应activity和fragment，是连接view和model的桥梁，是程序的主控制器。View告诉controller用户的行为，controller决定如何与model交互；跟据model中的数据更改，controller更新view的状态。

**总结：**     
优点:     
1. 耦合性低。mvc在分离model和view方面做的很好，但是不能完全杜绝model和view的交互
2. 可扩展性好。由于耦合性低，添加需求，扩展代码就可以减少修改之前的代码，降低bug的出现率。
3. 模块职责划分明确。主要划分层M,V,C三个模块，利于代码的维护。

缺点：      
1. mvc一个最大的弊端就是xml作为View层视图能力实在太弱，所以一般情况下我们都是通过Controller层来辅助处理一些视图的。这样的结果就导致Controller既作为控制层的同时又承担了View层的大部分功能，增加了controller和view的耦合。

## **MVP** ##
mvp即模型（model）——— 视图（view）——— 逻辑控制层（presenter），为解决上文中提到的mvc的缺点而生。

#### **Model** ####
与mvc中的model相同，无变化。
#### **View** ####
View视图对应为xml文件以及activity和fragment，负责呈现UI。View和Presenter的交互是双向的，即v和p互相持有对方实例，可以互调对方的方法     
#### **Presenter** ####
Presenter负责完成View与Model间的交互和业务逻辑。Presenter作为Model和View的桥梁，负责从Model拿到数据进行处理并返回给View。但Presenter和其他两层的沟通是通过接口协议进行的，所以每个Presenter中通常会包涵一个或多个接口协议。

**总结：**    
MVP核心思想是：   
设计一个抽象的V层接口，并由具体的View实现该接口，P层内部维护一个该接口的实例引用，一般在构造函数中传递进来赋值(即View层初始化P层实例时)，彼时P层即可通过调用该接口来完成对View层的操作,V层也因持有P层实例，可以进行业务逻辑处理委派。      
因此在MVP模式里通常包含4个要素：         
1.View :负责绘制UI元素、与用户进行交互(在Android中体现为Activity);         
2.View interface :需要View实现的接口，View通过View interface与Presenter进行交互，降低耦合，方便进行单元测试;  View层接口类只应该只有set/get方法，及一些界面显示内容和用户输入，除此之外不应该有多余的内容           
3.Model :负责存储、检索、操纵数据(有时也实现一个Model interface用来降低耦合);          
4.Presenter :作为View与Model交互的中间纽带，处理与用户交互的负责逻辑。          

优点:
1. mvc有的优点mvp都具备
2. 解耦view和presenter
3. view和model完全分离
4. presenter方便单元测试
5. 代码复用率高，比如可以将一个Presenter用于多个视图，而不需要改变Presenter的逻辑

缺点：
1. Presenter层与View层是通过接口进行交互的，接口粒度不好控制。粒度太小，就会存在大量接口的情况，使代码太过碎版化；粒度太大，解耦效果不好。同时对于UI的输入和数据的变化，需要手动调用V层或者P层相关的接口，相对来说缺乏自动性、监听性
2. Presenter除了业务逻辑以外，还有大量的model->view,view->model的数据同步逻辑，造成Presenter比较笨重，不好维护     
3. View层与Presenter层还是有一定的耦合度。一旦V层某个UI元素更改，那么对应的接口就必须得改，数据如何映射到UI上、事件监听接口这些都需要转变，牵一发而动全身。


## **MVVM** ##
mvvm即Model——— View ———ViewModel.

#### **Model** ####
model由于引进了Repository，所以这里的Model的定义相对简单，就是JavaBean

#### **View** ####
View层就是展示数据的，以及接收到用户的操作传递给viewModel层，通过dataBinding实现数据与view的单向绑定或双向绑定 

#### **ViewModel** ####
ViewModel逻辑控制层，负责处理数据和处理View层中的业务逻辑。它还为View提供了将事件传递给Model的通道

MVVM的目标和思想与MVP类似，它利用数据绑定(Data Binding)、命令(Command)、Repository以及jetpack组件打造的数据驱动的架构。    
ps:Command（命令绑定）简言之就是对事件的处理（下拉刷新、加载更多、点击、滑动等事件处理）


**总结：** 

优点：
1. 低耦合。解耦了view和viewmodel
2. 更新UI。我们在工作线程直接修改（在数据是线程安全的情况下）ViewModel的数据即可，不用再考虑要切到主线程更新UI了
3. 可复用性。一个ViewModel可以复用到多个View中。同样的一份数据，可以提供给不同的UI去做展示。对于版本迭代中频繁的UI改动，更新或新增一套View即可

缺点：
1. 数据绑定使得 Bug 很难被调试。你看到界面异常了，有可能是你 View 的代码有 Bug，也可能是 Model 的代码有问题。数据绑定使得一个位置的 Bug 被快速传递到别的位置，要定位原始出问题的地方就变得不那么容易了。
2. 数据双向绑定不利于代码重用。客户端开发最常用的重用是View，但是数据双向绑定技术，让你在一个View都绑定了一个model，不同模块的model都不同。那就不能简单重用View了。


更详细的mvvm可参考[https://tech.meituan.com/android_mvvm.html](https://tech.meituan.com/android_mvvm.html)
