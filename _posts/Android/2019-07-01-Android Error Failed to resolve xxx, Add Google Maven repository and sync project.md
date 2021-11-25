---
layout:     post
title:      Android Error:Failed to resolve:xxx, Add Google Maven repository and sync project
subtitle:   
date:       2019-07-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

如果在某一天你打开之前的项目后发现报错如下：
```java
Error:Failed to resolve:xxx, Add Google Maven repository and sync project
```

那么，恭喜你，解决方案如下：
在Project的build.gradle中添加如下代码:
<img src="/img/article/google.png"/>
