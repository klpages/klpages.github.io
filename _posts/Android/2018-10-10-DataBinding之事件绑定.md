---
layout:     post
title:      DataBinding之事件绑定
subtitle:   
date:       2018-10-10
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

DataBinding的事件绑定有2种方式：  
	- Method References(方法引用)  
	- Listener Bindings(事件监听)

1、Method References
	
- **创建响应方法**  
可以直接引用和listener方法签名(方法名称和一个参数列表（方法的参数的顺序和类型）组成)一致的方法，也可以自定义方法，但是我们自定义的方法参数和返回值必须与监听器中方法一样。比如对于view的onClick事件，它对应的listener的处理方法是：  
```java
        public interface OnClickListener {
    		void onClick(View v);
    	}
```

注意我们创建的方法参数和返回值必须与监听器中方法一样，所以我们创建如下响应方法：  
```java
    public class MyHandler {
	    public void myOnClick(View view){
	    	Toast.makeText(view.getContext(),"hello",Toast.LENGTH_SHORT).show();
	    }
    }
```

- **在布局文件中设置**  
```java
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
    	<data>
    		<variable name="handlers" type="com.databingexample.MyHandler"/>
    	</data>
    <LinearLayout  android:layout_width="match_parent" android:layout_height="match_parent" android:orientation="vertical">
    	<TextView  
			android:layout_width="wrap_content" android:layout_height="wrap_content" 
			android:text="测试事件绑定" 
			android:onClick="@{handlers::myOnClick}""/>
    </LinearLayout>
    </layout>
```


2、 Listener Bindings
	
- **创建响应方法**  
这种方法只要求返回值一样即可，这样我们可以自己定制参数，用刚刚那个例子来重写一下  
```java
        public class MyHandler {
	    public void myOnClick(View view,String name){
	    	Toast.makeText(view.getContext(),"hello"+name,Toast.LENGTH_SHORT).show();
	    }
    }
```

- **在布局文件中设置**
```java
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
    	<data>
    		<variable name="handlers" type="com.databingexample.MyHandler"/>
    	</data>
    <LinearLayout  android:layout_width="match_parent" android:layout_height="match_parent" android:orientation="vertical">
    	<TextView  
			android:layout_width="wrap_content" android:layout_height="wrap_content" 
			android:text="测试事件绑定" 
			android:onClick='@{(view)->handlers.myOnClick(view,"测试")}'/>
    </LinearLayout>
    </layout>
```
注意，整个响应表达式是一个lambda表达式，在编译的时候数据绑定会创建一个默认响应方法，当事件触发时，默认响应方法执行这个lambda表达式。lambda表达式可以根据->分成两部分： 
->左边的部分是(view) ，这里是原来listener中void onClick(View v)中的参数，这个参数必须全部写或者全部不写。
