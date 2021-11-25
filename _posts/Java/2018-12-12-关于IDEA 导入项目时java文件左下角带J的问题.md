---
layout:     post
title:      关于IDEA 导入项目时java文件左下角带J的问题
subtitle:   
date:       2018-12-12
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Java
---

- IDEA导入项目后发现所有的java文件左下角均带字母J,如下所示：  
<img src="/img/article/J1.png"/>  

- 这种情况说明项目的资源路径配置是有问题的,解决方案如下：  
File -> project structure, 打开Project Structure面板,删除Content Root并重新添加即可  
<img src="/img/article/J2.png"/> 
