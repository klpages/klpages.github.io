---
layout:     post
title:      ScrollView嵌套带有EditText的RecyclerView
subtitle:   
date:       2019-03-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

众所周知，ScrollView嵌套RecyclerView时会有许多问题，比如RecyclerView显示不全等，所以推荐使用NestedScrollView替代ScrollView,
但是使用NestedScrollView嵌套带有EditText的RecyclerView时，当用户在EditText输入时Recyclerview会上下跳动，这就比较尴尬了，
如下所示：  

<img src="/img/article/scrollview_recyclerview.gif" height="530" width="300" >

因此只能继续使用ScrollView,那么解决RecyclerView显示不全的方法也很简单,如下：
* 用RelativeLayout包裹RecyclerView即可,网上有说用LinearLayout包裹也可以,但是我这里不行
* PS：RelativeLayout里不可以设置多余属性,否者会出现上图中上下滚动的现象  
下面贴出xml源码,具体设置可查看如下源码：  

```java
 <android.support.constraint.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:fitsSystemWindows="true"
        tools:context=".view.scheme.WaterTreatmetSchemeActivity"
        android:focusableInTouchMode="true"
        android:focusable="true">

        <include
            android:id="@+id/toolbar"
            layout="@layout/toolbar"/>

        <ScrollView
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:fillViewport="true"
            android:scrollbars="none"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/toolbar">
            <android.support.constraint.ConstraintLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content">
                <TextView
                    android:id="@+id/tv_tips"
                    android:layout_width="wrap_content"
                    android:layout_height="@dimen/height_48_default"
                    android:text="@string/water_treatment_scheme"
                    android:gravity="center_vertical"
                    android:layout_marginStart="@dimen/spacing_16"
                    app:layout_constraintStart_toStartOf="parent"
                    app:layout_constraintTop_toTopOf="parent"/>
                <RelativeLayout
                    android:id="@+id/rl_water"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    app:layout_constraintStart_toStartOf="parent"
                    app:layout_constraintEnd_toEndOf="parent"
                    app:layout_constraintTop_toBottomOf="@id/tv_tips">
                    <android.support.v7.widget.RecyclerView
                        android:id="@+id/rv_water"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginStart="@dimen/spacing_16"
                        android:layout_marginEnd="@dimen/spacing_16"
                        android:adapter="@{adapter}"
                        app:layoutManager="@{layoutManager}"
                        android:nestedScrollingEnabled="false"/>
                </RelativeLayout>

                <View
                    android:id="@+id/view_cost"
                    android:layout_width="0dp"
                    android:layout_height="@dimen/height_48_default"
                    app:bgColor="@{@color/water_background}"
                    app:cornerRadius="@{8}"
                    android:layout_marginTop="@dimen/spacing_16"
                    android:layout_marginStart="@dimen/spacing_16"
                    android:layout_marginEnd="@dimen/spacing_16"
                    app:layout_constraintStart_toStartOf="@id/rl_water"
                    app:layout_constraintEnd_toEndOf="@id/rl_water"
                    app:layout_constraintTop_toBottomOf="@id/rl_water"/>
                <TextView
                    android:id="@+id/tv_construction_cost"
                    android:layout_width="wrap_content"
                    android:layout_height="@dimen/height_48_default"
                    android:text="@string/construction_cost"
                    android:gravity="center_vertical"
                    android:textColor="@android:color/white"
                    android:layout_marginStart="@dimen/spacing_16"
                    app:layout_constraintStart_toStartOf="@id/view_cost"
                    app:layout_constraintTop_toTopOf="@id/view_cost"/>
                <EditText
                    android:id="@+id/et_construction_cost"
                    android:layout_width="wrap_content"
                    android:layout_height="@dimen/height_48_default"
                    android:text="@={DataBindingUtils.convertIntToString(water.constrPrice)}"
                    android:enabled="@{MyConfig.INFOPURVIEW_PERMS==permission?false:true}"
                    android:textColor="@android:color/white"
                    android:background="@color/water_background"
                    android:paddingStart="@dimen/spacing_8_default"
                    android:paddingEnd="@dimen/spacing_8_default"
                    app:layout_constraintStart_toEndOf="@id/tv_construction_cost"
                    app:layout_constraintTop_toTopOf="@id/tv_construction_cost"/>
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="@dimen/height_48_default"
                    android:text="@string/price_unit"
                    android:textColor="@android:color/white"
                    android:gravity="center_vertical"
                    app:layout_constraintStart_toEndOf="@id/et_construction_cost"
                    app:layout_constraintTop_toTopOf="@id/et_construction_cost"/>

                <Button
                    android:id="@+id/btn_add_device"
                    android:layout_width="@dimen/width_120"
                    android:layout_height="@dimen/height_48_default"
                    android:text="@string/add_device"
                    android:gravity="center"
                    android:textColor="@android:color/white"
                    android:onClick="@{()->viewModel.chooseDevices()}"
                    android:enabled="@{permission==MyConfig.INFOPURVIEW_PERMS?false:true}"
                    app:layout_constraintStart_toStartOf="@id/view_cost"
                    app:layout_constraintEnd_toEndOf="@id/view_cost"
                    app:layout_constraintTop_toBottomOf="@id/view_cost"/>

                <Button
                    android:id="@+id/btn_back"
                    android:layout_width="@dimen/width_120"
                    android:layout_height="@dimen/height_48_default"
                    android:text="@string/back_doc"
                    android:textColor="@android:color/white"
                    android:onClick="@{()->viewModel.pageUp()}"
                    app:layout_constraintStart_toStartOf="parent"
                    app:layout_constraintTop_toBottomOf="@id/btn_add_device"
                    app:layout_constraintEnd_toStartOf="@+id/btn_next"/>

                <Button
                    android:id="@+id/btn_next"
                    android:layout_width="@dimen/width_120"
                    android:layout_height="@dimen/height_48_default"
                    android:text="@string/next_doc"
                    android:onClick="@{()->viewModel.pageDown()}"
                    android:textColor="@android:color/white"
                    app:layout_constraintStart_toEndOf="@id/btn_back"
                    app:layout_constraintEnd_toEndOf="parent"
                    app:layout_constraintBottom_toBottomOf="@id/btn_back"/>
            </android.support.constraint.ConstraintLayout>
        </ScrollView>


    </android.support.constraint.ConstraintLayout>
```

ok
