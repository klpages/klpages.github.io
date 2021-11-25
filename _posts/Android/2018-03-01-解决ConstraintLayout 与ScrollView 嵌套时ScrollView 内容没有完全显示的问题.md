---
layout:     post
title:      解决ConstraintLayout 与ScrollView 嵌套时ScrollView 内容没有完全显示的问题
subtitle:   
date:       2018-03-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

ConstraintLayout 布局中有ScrollView 时，ScrollView 的宽高都要设置为0dp 才可以正确的约束布局，如下所示：
```java
  <ScrollView
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:scrollbars="none"
            android:fillViewport="true"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/toolbar">
  </ScrollView>
```
