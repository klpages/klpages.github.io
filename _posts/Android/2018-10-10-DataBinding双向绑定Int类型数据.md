---
layout:     post
title:      DataBinding双向绑定Int、Double等类型数据
subtitle:   
date:       2018-10-10
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

DataBinding双向绑定String类型数据时没有任何问题，但是双向绑定int、double等类型数据时会有些问题，特此记录解决方案：              

创建inverseMethod方法                     
```java
public class DataBindingUtils {

   @InverseMethod("convertIntToString")
    public static int convertStringToInt(String value) {
        try {
            return Integer.parseInt(value);
        } catch (NumberFormatException e) {
            return 0;
        }
    }

    public static String convertIntToString(int value) {
        return String.valueOf(value);
    }


}

```

xml文件调用
```java
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">
    <data>
        <import type="cn.util.DataBindingUtils"/>
        <variable
            name="deviceConfig"
            type="cn.data.model.DeviceConfig"/>
    </data>
    <android.support.constraint.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="@dimen/height_48_default"
        android:layout_marginStart="@dimen/spacing_16"
        android:layout_marginEnd="@dimen/spacing_16"
        android:layout_marginTop="@dimen/spacing_8_default"
        android:background="@android:color/white">

        <EditText
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@={DataBindingUtils.convertIntToString(deviceConfig.deviceNum)}"
            android:inputType="number"
            app:layout_constraintTop_toBottomOf="@id/tv_dci_device_name"
            app:layout_constraintStart_toStartOf="parent"/>


    </android.support.constraint.ConstraintLayout>

</layout>
```
