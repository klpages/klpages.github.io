---
layout:     post
title:      AppCompatSpinner双向绑定注意事项
subtitle:   
date:       2018-10-10
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---


最近用MVVM开发程序时，遇到了一个小问题->  
某个页面初始化时Spinner控件有时无法选中设定的值,布局文件如下：
```java
<android.support.v7.widget.AppCompatSpinner
                    android:layout_width="0dp"
                    android:layout_height="@dimen/height_48_default"
                    android:paddingEnd="@dimen/spacing_16"
                    android:adapter="@{adapter}"
                    android:selectedItemPosition="@={water.peopleRange}"
                    android:background="@android:color/white"
                    android:onItemSelected="@{(parent,view,position,id)->viewModel.spinnerItemChanged(parent,view,position,id)}"
                    app:permission="@{permission}"
                    app:layout_constraintStart_toEndOf="@id/tv_people_number"
                    app:layout_constraintTop_toTopOf="@id/tv_people_number"
                    app:layout_constraintEnd_toEndOf="parent"/>
```
解决方案如下：   
需要先对选中项(water.peopleRange)赋值,然后在初始化Spinner的adapter即可
