---
layout:     post
title:      调试原生 Android 应用中的WebView
subtitle:   
date:       2018-10-10
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

使用 Chrome 开发者工具在您的原生 Android 应用中调试 WebView。

在 Android 4.4 (KitKat) 或更高版本中，使用 DevTools 可以在原生 Android 应用中调试 WebView 内容。

 - 开启WebView的调试功能 
 ```java
 if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    WebView.setWebContentsDebuggingEnabled(true);
}
```
此设置适用于应用的所有 WebView。

**提示：** WebView 调试不会受上面代码中 debuggable 标志的状态的影响。如果您希望仅在 debuggable 为 true 时启用 WebView 调试，请添加如下代码：
```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    if (0 != (getApplicationInfo().flags & ApplicationInfo.FLAG_DEBUGGABLE))
    { WebView.setWebContentsDebuggingEnabled(true); }
}
```
- 在 DevTools 中打开 WebView
在chrome地址栏中输入chrome://inspect 并回车，然后点击您想要调试的 WebView 下方的 inspect  
<img src="/img/article/webview_debug.png"/>   

- 故障排除
在 chrome://inspect page 上无法看到您的 WebView？

1. 验证已为您的应用启用 WebView 调试。
2. 在设备上，打开应用以及您想要调试的 WebView。然后，刷新 chrome://inspect 页面。
3. 需要翻墙
