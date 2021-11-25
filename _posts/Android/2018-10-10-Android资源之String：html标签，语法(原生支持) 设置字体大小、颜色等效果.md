---
layout:     post
title:      Android资源之String：html标签，语法(原生支持) 设置字体大小、颜色等效果
subtitle:   
date:       2018-10-10
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

在开发过程中有时候难免会在string.xml文件中设置一些效果，例如设置字体颜色、大小，如下所示：    
```java
<string name="report_runtime"><Data><![CDATA[<font color="#000FFF">运行时间</font><font color="#FFF000"><big><big>%1$1d</big></big>小时</font> \t \t <font color="#000FFF">空气净化总量</font>&#160;<font color="#FFF000"><big><big>%2$d</big></big>立方米</font>]]></Data></string>
```                     
&#160; 表示空格   
\t  表示tab     
\n  表示换行
%1$1d、%2$d等的用法：     
%n$md：代表输出的是整数，n代表是第几个参数，设置m的值可以在输出之前放置空格，也可以设为0m,在输出之前放置m个0            
设置字体颜色时推荐使用CDATA标签，android中只支持<font>标签的color和face标签，不支持size标签，所以文字的大小只能通过标签<big>或者<small>来相对调节，
经过笔者测试，<big>标签可以嵌套使用，效果也是嵌套增长，例如“<big><big>我是示例文字</big></big>实现后的效果，比<big>我是示例文字</big>的表现效果要
big了一点，同理，我认为<small>标签也是可以嵌套使用的。 

以上设置完成后需要在java代码中如下设置：
```java
String s = getString(R.string.report_runtime, 420,171600);
textview.setText(Html.fromHtml(s));
```
运行效果为：
<img src="/img/article/report_1.png"/>   

以下标签应该也是可以支持的，未测试：          
"b");     ==>StyleSpan(Typeface.BOLD), <br/>                                                                             									
"i");      ==>StyleSpan(Typeface.ITALIC),		                          
"u");     ==>UnderlineSpan		                    
"tt");     ==>TypefaceSpan("monospace"),		                        
"big");   ==>RelativeSizeSpan(1.25f),		                          
"small"); ==>RelativeSizeSpan(0.8f),		                              
"sup");   ==>SubscriptSpan(),//上下标		                          
"sub");   ==>SuperscriptSpan(),		                        
"strike"); ==>StrikethroughSpan(),//删除线		                        
"li");        ==>new BulletSpan(10),//用在首位，多个列表的圆点符号		<br/>                      	
"marquee"); TextUtils.TruncateAt.MARQUEE                      

由其applyStyles 方法可知还支持			                  
"font;":	   <br/>                                                                   		
    ";height="    ==>Height(size),		                        
    ";size="        ==>AbsoluteSizeSpan(size, true),		                        
    ";fgcolor="    ==>ForegroundColorSpan(c);		                      
    ";color="       ==>ForegroundColorSpan(c);		                        
    ";bgcolor="   ==>BackgroundColorSpan(c);		                        
    ";face="        ==>TypefaceSpan(sub),	                        
    
“a;”:	                                
    ”;href=“  ==>URLSpan(sub),		                    
"annotation;" ==>Annotation(key, value),	                      

另附：  [HTML特殊字符编码对照表](http://evassmat.com/8GTP)
        
