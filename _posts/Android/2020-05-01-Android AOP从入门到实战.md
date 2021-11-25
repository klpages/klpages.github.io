---
layout:     post
title:      Android AOP从入门到实战
subtitle:   
date:       2020-05-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
--- 		

### 前言

在Android中AOP也越来越多的被提到，那么AOP到底是什么呢？它和OOP是什么关系？本文笔者将带领大家初探Android中AOP的使用。



### 一、什么是AOP?

AOP：面向切面编程(Aspect Oriented Programming)，通过预编译方式以及运行期间动态代理的方式实现程序功能的统一维护的一种技术。简单来说就是对业务逻辑中的相同功能或步骤进行提取，做一个统一的管理，已达到代码复用与解耦的效果。

如果你有AOP还是不太理解，那么不着急，请继续往下看，后续我们会通过AOP与OOP的对比以及AOP的一些实际使用场景进行分析，帮您认清AOP。

### 二、AOP与OOP的区别

OOP：面向对象编程(Object Oriented Programming)，针对业务处理过程的实体及其属性和行为进行抽象封装，以获得更加清晰高效的逻辑单元划分，然后通过的Java的继承、多态以及7大设计原则进行编程，以提高软件的重用性、灵活性和扩展性。

如果您对OOP还不是太理解，建议阅读[Java学习了那么多年还不理解面向对象吗]([http://rjgc.cn/2018/12/12/Java%E5%AD%A6%E4%B9%A0%E4%BA%86%E9%82%A3%E4%B9%88%E5%A4%9A%E5%B9%B4%E8%BF%98%E4%B8%8D%E7%90%86%E8%A7%A3%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E5%90%97/](http://rjgc.cn/2018/12/12/Java学习了那么多年还不理解面向对象吗/))

**区别：**

1. 思想角度不同：AOP是以横向角度进行编程的思想，而OOP是一种纵向的编程思想
2. 侧重点不同：AOP注重相同功能的统一维护，而OOP更加注重业务逻辑单元的划分

### 三、AOP的实现

AOP思想已经提出了很多年，市面也有许多框架可以帮助我们简单实现AOP，目前比较好比较火的框架有**APT、AspectJ、ASM**,它们被称为AOP三剑客。

#### 1、APT

目前Android注解解析的方式有2种：一种是运行期通过反射去解析，另一种就是在编译期使用APT去解析注解，并生成`.java`文件。

APT(Annotation Processing Tool)即注解处理器，作用在编译期，用来扫描和处理代码中的注解，生成` .java` 文件。**注意：**APT并不能对源文件进行修改操作，只能生成新的文件，例如往原来的类中添加方法或者生成一个新的`.java`文件。

**APT使用流程：**

1. 自定义注解处理器，需要继承AbstractProcess
2. 可以借助JavaPoet来自定义需要生成的代码
3. 注册注解处理器@AutoService(Processor.class)
4. 使用@SupportedAnnotationTypes()设置自定义注解处理器支持的注解类型
5. 在build.gradle中使用annotationProcessor引入自定义的注解处理器



APT的具体使用及实战此处不再过多介绍，后续会单独介绍。

#### 2、AspectJ

AspectJ是一个基于Java语言的AOP框架，它有两种实现方式：

一种是直接使用AspectJ来编写，然后使用AspectJ的编译器(ajc编译器)用来编译`.aj`文件，生成Java标准的class文件；

另一种是使用Java语言和注解，通过AspectJ提供的织入器 (aspectjweaver)，织入代码到目标class文件。

但是Android Studio没有`ajc`编译器，无法直接编译AspectJ文件，因此，如果想在android中使用AspectJ就需要在gradle中做一些额外的配置，比较麻烦。但是好在有很多库可以帮助我们集成AspectJ，此处推荐沪江的开源项目**[gradle_plugin_android_aspectjx](https://github.com/HujiangTechnology/gradle_plugin_android_aspectjx)**，这个库的接入方法也非常简单，具体可以参考：**[看AspectJ在Android中的强势插入](https://www.jianshu.com/p/5c9f1e8894ec)**
这篇博客详细介绍了AspectJ的各种注解的用法，使用起来并不会太复杂，非常适合在项目中使用。

因为Android中的AspectJ是阉割版的，因此在**Android中AspectJ仅支持编译期(`.class`转`.dex`期间)代码注入**，它的工作原理是：**通过Gradle Transform，在class文件生成后至dex文件生成前，遍历并匹配所有符合AspectJ文件中声明的切点的class文件，然后向目标`.class`文件织入Aspect代码，用来建立与Aspect程序的连接(获得执行的对象、方法、参数等)**。

#### 3、ASM

ASM是一个字节码操作框架，它可以直接生产`.class`字节码文件，也可以在类被加载进JVM之前动态修改类行为，比如在方法执行前后增加一些代码

使用ASM一定要对字节码有一定的了解，那什么是字节码呢？

字节码和ASM使用及实战推荐查看  [ASM官网](https://asm.ow2.io/index.html) 

[字节码增强技术探索](https://tech.meituan.com/2019/09/05/java-bytecode-enhancement.html)

#### 4、AOP三剑客的区别与联系

- APT

  **作用时机：**编译期(生成.java文件前)

  **操作对象：**java源码

  **优点：**

  1. 可以减少样板代码
  2. 生成代码位置可控

  **缺点：**

  1. 生成的代码需要在运行时主动调用
  2. 无法修改源文件，只能新增

- AspectJ

  **作用时机：**编译期(class文件生成后，dex文件生成前)

  **操作对象：**java源码，class文件及jar包

  **优点：**

  1. 功能强大，易使用

  **缺点：**

  1. 出了问题不好排查
  2. 编译时间变长
  3. 无法对Activity生命周期织入埋点统计，因为Activity.class不参与打包（android.jar位于android设备内），只有参与打包的那些支持库比如v4、v7、v13、androidx才能织入

- ASM

  **作用时机：**编译期(生成class文件后)或运行期(加载进JVM前)

  **操作对象：**.class文件

  **优点：**

  1. ASM可以直接操作字节码，那么理论上对字节码的任意修改，都可以实现
  2. ASM可以直接操作字节码，因此相比其他框架效率更高

  **缺点：**

  1. 学习成本高，上手难，需要对java字节码有一定的理解

### 四、Android中AOP的应用场景及实战

1. 系统权限申请

2. 业务权限拦截(登录拦截等)

3. 日志埋点

4. 性能监控

5. 防抖动

   .........

应用场景非常广，笔者整理了相关代码已上传至github，代码中注释比较详细这里就不展开说了，欢迎star

[Aop集成框架Demo](https://github.com/Don-Lee/AopDemo)







