---
layout:     post
title:      AppCompatSpinner的使用及样式设置
subtitle:   
date:       2018-03-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---
本文讲解内容包含以下几个内容：  
	1. AppCompatSpinner字体大小及颜色设置    
	2. 分割线添加    
	3. 选中item时颜色变化动画      
效果图如下：    
<img src="/img/article/spinner.gif" widht="300px" height="550px"/>

实现以上效果，分3步：  
#### 1.下拉列表框效果实现  
将以下代码复制到styles.xml中即可
```java
<style name="spinner" parent="Widget.AppCompat.DropDownItem.Spinner">
	<!--设置分割线-->
        <item name="android:dropDownListViewStyle">@style/spinnerListStyle</item>
	<!--设置字体颜色及大小-->
        <item name="android:textColor">@color/pm25_good</item>
        <item name="android:textSize">16sp</item>
    </style>

    <style name="spinnerListStyle" parent="@android:style/Widget.ListView.DropDown">
        <item name="android:divider">@color/default_color</item>
        <item name="android:dividerHeight">1dp</item>

        <!-- item按下效果 -->
        <item name="android:listSelector">@android:color/holo_red_light</item>
    </style>
```

#### 2.AppCompatSpinner显示内容的TextView样式设置

我的android架构为MVVM，所以下面的代码看起来有点怪，如果您的代码非MVVM架构，直接使用方法内的代码即可    
DataBindingUtils.java类
```java
    @BindingAdapter("setTextColor")
    public static void setTextColor(AppCompatSpinner spinner, @ColorInt int color) {
        spinner.getViewTreeObserver().addOnGlobalLayoutListener(() -> ((TextView) spinner.getSelectedView()).setTextColor(color));
    }
```


#### 3.xml代码如下：
```java
<android.support.v7.widget.AppCompatSpinner
                    android:id="@+id/spinner_decorate_status"
                    android:layout_width="0dp"
                    android:layout_height="@dimen/height_48_default"
                    android:paddingEnd="@dimen/spacing_16"
                    app:permission="@{permission}"
                    android:adapter="@{adapter}"
		    android:entries="@array/buildingTypeArray"  /*adapter和entries二选一设置即可*/
                    android:selectedItemPosition="@={model.renovationStatusIndex}"
                    android:background="@android:color/white"
                    setTextColor="@{@color/default_color}" /*此句代码设置颜色，调用DataBindingUtils.java类中方法*/
                    android:theme="@style/spinner"	/*此句代码调用步骤1中的样式设置下拉框样式*/
                    android:overlapAnchor="false"  /*此句代码设置下拉框显示在控件下方*/
                    app:layout_constraintStart_toEndOf="@id/tv_decorate_status"
                    app:layout_constraintTop_toTopOf="@id/tv_decorate_status"
                    app:layout_constraintBottom_toBottomOf="@id/tv_decorate_status"
                    app:layout_constraintEnd_toEndOf="parent"/>
```

设置Spinner数据源有2种方式：   
方法一: 在java代码中给Spinner设置adapter，下面贴出activity中spinner相关代码：  

```java
private void initSpinnerData() {
        List<String> buildingType = new ArrayList<>();
        buildingType.add("老式装修(2000年前竣工)");
        buildingType.add("节能建筑(2000年后竣工)");

        ArrayAdapter<String> adapter = new ArrayAdapter<>(getActivity(),
                android.R.layout.simple_spinner_dropdown_item, buildingType);

        getBinding().setAdapter(adapter);

    }
```

方法二:在xml文件中设置android:entries属性     

1. 在res/values 下面创建array.xml文件，代码如下：    

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string-array name="buildingTypeArray">
        <item>老式装修(2000年前竣工)</item>
        <item>节能建筑(2000年后竣工)</item>
    </string-array>
</resources>
```

2. 设置spinner的entries属性即可，spinner的xml源码上文有贴出，此处就比贴了
