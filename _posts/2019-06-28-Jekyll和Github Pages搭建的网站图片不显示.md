---
layout:     post
title:      Jekyll和Github Pages搭建的网站图片不显示
subtitle:   
date:       2019-06-28
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Blog
---

之前在github上使用github的图片源时直接使用那一长串的https链接就可以了，但是在 github page搭建的博客上不可以这么用，遮掩的后果是无法显示图片。
- 第一种解决方法(推荐)  
要想在自己博客上显示某张图片，直接将这张图片一起上传到网站目录下就可以了，然后 使用图片的相对路径。如下是一个例子： 
```
<img src="/img/article/img_demo.png"/>
```
<img src="/img/article/img_demo.png"/>

- 第二种解决方法  
将图片上传到图床，然后图床会提供一个url，然后使用这个url就可以显示图片了。  
推荐的图床：[SM.MS](https://sm.ms/)
