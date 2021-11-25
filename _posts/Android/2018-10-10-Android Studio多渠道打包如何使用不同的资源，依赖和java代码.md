---
layout:     post
title:      Android Studio多渠道打包如何使用不同的资源，依赖和java代码
subtitle:   
date:       2018-10-10
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

在日常开发中，我们或多或少都会碰到多渠道打包的一些问题，有些是同一个版本要上传到不同的平台，有些是要提供给不同的代理商，
中间可能需要改动里面的图片或其他的一些资源文件，如果每个版本都维护一套代码替换会令人吐血~ 好在可以使用Gradle的productFlavors实现多渠道。
废话不多说，上代码.........

### 资源文件的适配
假如有这么一个需求：现在需要打包两个版本，但要求两个版本的某些图片不一样，并且app的名称也不同，简单的做下适配 
###### 1、这里我新建立了一个res资源文件，res_cq，把这个版本的图片放到这资源文件下，这样项目在打包时就会直接覆盖res文件下对应的资源             
<img src="/img/article/20180613173408.png"/>            
###### 2、在gradle里配置不同平台采用不同的资源文件
```java
//以下代码放在android{}内
//配置资源文件路径，可动态指定不同版本资源文件
    sourceSets {
        main {
            manifest.srcFile 'src/main/AndroidManifest.xml'
            java.srcDirs = ['src/main/java']
            resources.srcDirs = ['src/main/resources']
            aidl.srcDirs = ['src/main/aidl']
            renderscript.srcDirs = ['src/maom']
            res.srcDirs = ['src/main/res']
            assets.srcDirs = ['src/main/assets']
            jniLibs.srcDir 'src/main/jniLibs'
        }

        //订制版APP对应的资源文件路径,“chunqin”在productFlavors里定义
        chunqin.res.srcDirs = ['src/main/res_cq']
        }
    } 
    
    productFlavors{
        ewd{
            buildConfigField "int","ENV_TYPE","0"
            // 对resValue在java代码中的使用：String app_id = getResources().getString(R.string.app_id);
            resValue("string", "app_id", "50074")
            resValue("string", "app_start", "1")
            // 对manifestPlaceholders中资源的使用：在AndroidManifest.xml文件中的application节点下
            // andorid:icon="${icon}"
            // android:app_name="${app_name}"
            manifestPlaceholders=[app_name:"@string/app_name",icon:"@mipmap/ic_launcher"]
        }
        chunqin{
            //java代码中可以使用if(BuildConfig.EVN_TYPE==1)来判断是哪个渠道
            buildConfigField "int","ENV_TYPE","1"
            //包名
            applicationId "com.ewd.cq"
            manifestPlaceholders=[app_name:"@string/app_name_cq",icon:"@mipmap/ic_launcher"]
        }
    }

```

### Java代码适配
多渠道打包时如果遇到java代码也有不一致的地方，解决方式如下：                
有一种解决办法就是：在代码里进行判断，根据渠道的不一样，加载不同的图片和布局，这是一种解决办法。但是当渠道有很多时，代码就会变得很难维护，而且指定渠道用到的资源文件都会被打入所有 Apk 中。所以这个方法并不值得推荐。那么，有什么好的解决办法呢？

办法 Google 早就给我们想好了，而且相当简单，那就是：在 main 的同级目录下创建以渠道名命名的文件夹，在渠道文件夹下有两种类型的文件会被用到，一种是资源文件，一种的java代码文件，资源文件跟在main中的资源文件使用的方式方法是一样的，有不同的资源文件时，只要命名跟main文件中的资源命名是样的，就会自动替换掉main中的资源文件，不过java文件夹下面的java文件不太一样，不会自动替换掉main中的java文件，所以使用的时候，如果是渠道独有的java文件的话，在main中就不要存在该java文件就行，否则会报文件重复的错误。


更多可参考
https://www.jianshu.com/p/872dc6f89cb4                         
https://blog.csdn.net/CHX_W/article/details/78660462
