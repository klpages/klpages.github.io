---
layout:     post
title:      Android Databinding 报错Cannot resolve symbol BR.xxx.    
subtitle:   
date:       2019-12-21
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

在使用DataBinding开发的过中，经常会出现无法引用到BR类的字段的情况，如下图所示：  
<img src="/img/article/BR.png"/>   

一般情况下会有以下几种解决办法：   

- 1、Clean Project   
- 2、Rebuild Project  
- 3、Invalidate caches/restart

但是很不幸，有时候以上方法仍然无法解决，此处记录以下我的解决方法，以备他日不时之需：  

**重新导入BR类的包或者在BR类前面写上完整的包名**   

<img src="/img/article/BR1.png"/>