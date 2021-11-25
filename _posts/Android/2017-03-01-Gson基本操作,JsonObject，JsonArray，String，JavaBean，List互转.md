---
layout:     post
title:      Gson基本操作,JsonObject，JsonArray，String，JavaBean，List互转
subtitle:   
date:       2017-03-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---
>话不多说，上代码
```java
String、JsonObject、JavaBean 互相转换
    User user = new Gson().fromJson(jsonObject, User.class);
    User user = new Gson().fromJson(string, User.class);
    String string = new Gson().toJson(user);
    JsonObject jsonObject = new Gson().toJsonTree(user).getAsJsonObject(); 
    JsonObject jsonObject = new JsonParser().parse(string).getAsJsonObject();
String、JsonArray、List互相转换
    List<User> userList = gson.fromJson(string, new TypeToken<List<User>>() {}.getType()); 
    List<User> userList = gson.fromJson(jsonArray, new TypeToken<List<User>>() {}.getType()); 
    String string = new Gson().toJson(userList); 
    JsonArray jsonArray = new Gson().toJsonTree(userList, new TypeToken<List<User>>() {}.getType()).getAsJsonArray();
    JsonArray jsonArray = new JsonParser().parse(string).getAsJsonArray();
```
