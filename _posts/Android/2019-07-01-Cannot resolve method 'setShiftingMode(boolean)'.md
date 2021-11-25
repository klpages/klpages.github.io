---
layout:     post
title:      Cannot resolve method 'setShiftingMode(boolean)'
subtitle:   
date:       2019-07-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---
在解决BottomNavigationView底部导航栏不受数量限制时遇到了如下错误：
```java
错误: 找不到符号
符号:   方法 setShiftingMode(boolean)
位置: 类型为BottomNavigationItemView的变量 itemView
```
解决方案如下：
直接将itemView.setShiftingMode(false);修改为itemView.setLabelVisibilityMode(LabelVisibilityMode.LABEL_VISIBILITY_LABELED);即可
