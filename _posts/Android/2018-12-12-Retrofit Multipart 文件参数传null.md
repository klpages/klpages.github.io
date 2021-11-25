---
layout:     post
title:      Retrofit Multipart 文件参数传null
subtitle:   
date:       2018-12-12
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

##### 项目背景：
最近项目开发过程中使用了rxjava2 依次多个请求，每次请求都是拿着上一个请求的结果去执行请求，因此遇到了上传文件接口传null的情况。
而@Multipart是必须上传一个文件,若不传,则报错java.lang.IllegalStateException: Multipart body must have at least one part.

##### 解决方案：
MultipartBody.Part part = MultipartBody.Part.createFormData("",""); 

直接传入两个空字符串的part就可以了,不可以传null;
