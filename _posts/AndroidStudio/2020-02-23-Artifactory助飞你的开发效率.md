---
layout:     post
title:      Artifactory助飞你的开发效率
subtitle:   
date:       2020-02-26
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android Studio
    - Android
---

JFrog Artifactory有的人陌生但有的人不陌生。它主要是一款二进制存储管理工具，可以搭建私服，帮助我们管理我们的构件，以提升我们的开发效率。   
笔者将从搭建开始并示范如何提升你的开发效率。   


搭建之前请先android jdk1.8，否则搭建不成功

#### 下载    
下载地址(Windows & Mac)：
链接: https://pan.baidu.com/s/1phFGTa7-UinsfxUpfm--GA 提取码: nv4v 

下载完成后解压相应版本到安装目录。
<div style='display: none'>
#### 破解与安装   
1.打开命令行工具，跳转到你的jar包所在目录，执行 java -jar artifactory-injector-1.1.jar,然后会跳出两个选项，分别是：1-生成密钥字符串；2-破解

<img src="/img/article/artifactory1.webp"/>

2.输入数字 2

<img src="/img/article/artifactory2.webp"/>

3.输入软件包的路径  
<img src="/img/article/artifactory3.webp"/>

4.输入“yes”，之后会出现一大串内容，然后跳出两个选项：1-生成密钥字符串；2-破解;  最后请输入数字1  
<img src="/img/article/artifactory4.webp"/>

5.将密钥字符串拷贝下来(启动后会使用)，然后输入“exit”退出；

6.进入bin目录点击artifactory.bat 启动Artifactory,actifactory运行期间不要关闭artifactory.bat打开的命令行窗口

7.打开浏览器，输入localhost:8081,接下来就会出现一个需要密钥的窗口，直接把上面得到的密钥字符串贴上去即可
</div>

#### 使用教程(以Android Studio为例，示范代理android项目所依赖的库文件，其他IDE同理)    
Artifactory有三种仓库：    
·local(本地私有仓库)：用于内部使用,可上传自己的组件，上传的组件不会向外部进行同步。

·Remote(远程仓库)：用于代理及缓存公共仓库，不能向此类型的仓库上传私有组件。

·Virtual(虚拟仓库)：不是真实在存储上的仓库，它用于组织本地仓库和远程仓库。

众所周知，android项目默认会依赖jcenter和google两个库，而我们访问这2个库的速度是非常缓慢的，那我们可以使用artifactory来解决这个问题。有的朋友
说可以直接使用阿里云的代理库就可以了，是的，的确可以，但是artifactory搭建私服访问速度更快，而且还可以使用本地库来管理我们自己的构件。

1.在浏览器中打开artifactory并登录，默认账号密码是admin/password    

2.点击admin,选中Remote

<img src="/img/article/artifactory1.png"/>

3.点击右上角的new按钮，选择Maven类型，然后输入key(随便写)，url使用的是阿里云代理的库地址,最后点击save即可。    
此处附上阿里云代理仓库列表： https://maven.aliyun.com/mvn/view

<img src="/img/article/artifactory2.png"/>

4.重复步骤3，把jcenter、google、public库都添加进去     

<img src="/img/article/artifactory3.png"/>  

5.点击admin,选中Virtual,然后点击右上角new按钮，选择Maven类型

<img src="/img/article/artifactory4.png"/>  

6.按照上图步骤设置1、2、3后点击保存即可    

7.将android studio中jcenter和google库替换为我们私服中的url即可   

<img src="/img/article/artifactory5.png"/> 


8.local仓库你可以上传自己的Library包等组件，然后用gradle的implementation进行引用即可，此处就不详解了   


完成以上步骤后你可以打开你的ide试一下编译速度，第一次加载的时候会比较慢，因为要把库加载到本地，第二次开始你就会发现你的ide已经飞起来了













