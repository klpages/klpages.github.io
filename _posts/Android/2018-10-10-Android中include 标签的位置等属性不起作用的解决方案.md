---
layout:     post
title:      Android中include 标签的位置等属性不起作用的解决方案
subtitle:   
date:       2018-10-10
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

Android中经常会使用include标签复用布局，但是有时发现给include设置位置时没有效果，研究后发现需要重载include的layout_width和layout_height属性后其他的属性才会生效。
```java
<include
                android:id="@+id/device_selection"
                layout="@layout/layout_device_selection"
		//设置宽高属性后，其他属性才会生效
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                app:layout_constraintTop_toBottomOf="@id/tv_building_type" />
```