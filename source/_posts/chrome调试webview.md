---
title: chrome调试Android webview
date: 2018-03-03 09:29:35
tags:
  - debug
---

# 前提条件
手机正常连接电脑，并且可以在开发工具内部debug

# chrome设置
打开`chrome://inspect`，勾选`Discover USB devices`

# 在代码中添加运行调试的设置

```java
if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT){
    webView.setWebContentsDebuggingEnabled(true);
}
```

# 运行程序
经过以上的设置，运行程序，打开webview的页面，加载网页，这样在chrome的`chrome://inspect`页面就可以看到当前打开的页面了

![](./chrome_webview_debug_01.jpg)

点击链接下面的`inspect`，即可以查看当前webview中的内容，并且可以在`console`中执行js以及执行通过webview的`addJavascriptInterface`注入的native接口

![](./chrome_webview_debug_02.jpg)

并且可以通过`sources`标签调试js代码就像在浏览器内部调试网页中的js一样。LOL