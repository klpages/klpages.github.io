---
layout:     post
title:      AndroidStudio报variantOutput.getProcessResources() is obsolete警告
subtitle:   
date:       2019-03-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
    - Android Studio
---
如果你android studio gradle的版本≥3.3.0那么及会遇到如下警告：  
```java
WARNING: API 'variantOutput.getProcessResources()' is obsolete and has been replaced with 'variantOutput.getProcessResourcesProvider()'.
It will be removed at the end of 2019.
For more information, see https://d.android.com/r/tools/task-configuration-avoidance.
To determine what is calling variantOutput.getProcessResources(), use -Pandroid.debug.obsoleteApi=true on the command line to display more information.
Affected Modules: app
```
简单翻译一下：API 'variantOutput.getProcessResources()' 已经过时了，将在2019年末被移除，请用variantOutput.getProcessResourcesProvider()替换。  
如果你代码中有该API，替换掉即可；  
如果源码中没有，那么有可能因为我们依赖的第三方插件使用了过时的API导致的。我们暂时可以无需理会，反正不影响运行，等待依赖库更新即可。
