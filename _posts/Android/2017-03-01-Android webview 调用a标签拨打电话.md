---
layout:     post
title:      Android webview 调用a标签拨打电话
subtitle:   
date:       2017-03-01
author:     Don
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

在html中拨打电话需要使用a标签，如下所示  
```html
<a href="tel:12345678900">打电话</a>
```

而在webview里面如果不做处理直接加载html页面的话， android认为是页面跳转，直接提示找不到页面。  
好了，话不多说，直接上解决方案：  
```java
mWebView.setWebViewClient(new WebViewClient() {

            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                if (url.contains("tel:")) {
                    Intent intent = new Intent(Intent.ACTION_VIEW,
                            Uri.parse(url));
                    startActivity(intent);
                    return true;
                }
                view.loadUrl(url);
                return true;
            }
        });
```

另外请不要忘记添加拨打电话的权限  
```java
<uses-permission android:name="android.permission.CALL_PHONE"/>
``` 


