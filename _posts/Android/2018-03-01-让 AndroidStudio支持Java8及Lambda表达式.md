---
layout:     post
title:      让 AndroidStudio支持Java8及Lambda表达式
subtitle:   
date:       2018-03-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---


AndroidStudio从2.1开始官方通过Jack支持Java8，从而使用Lambda等特性。

配置非常简单，只需2步：<br/>
### 1、将如下代码添加到你module的 build.gradle    <br/>

```
android {                
          //让android studio支持java8                  
          compileOptions {                      
              sourceCompatibility JavaVersion.VERSION_1_8                                       
          }             
        }      
```

### 2、将如下代码添加到你project的 build.gradle    

```java
//支持lamdba         
buildscript {               
     repositories {             
        mavenCentral()              
     }              

     dependencies {                 
        classpath 'me.tatarka:gradle-retrolambda:3.7.0'
     }              
}                 

// Required because retrolambda is on maven central                       
repositories {                    
 mavenCentral()                     
}                 

apply plugin: 'me.tatarka.retrolambda'            
```
  
