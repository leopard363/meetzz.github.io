---
layout:     post
title:      网页meta标签配置no-cache
subtitle:   no-cache
date:       2019-04-12
author:     LSC
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - 缓存
    - meta
---

>网页meta标签配置no-cache

# 网页meta标签配置no-cache



#### html

```html
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<meta http-equiv="X-UA-Compatible" content="IE=8">
<meta http-equiv="Expires" content="0">
<meta http-equiv="Pragma" content="no-cache">
<meta http-equiv="Cache-control" content="no-cache">
<meta http-equiv="Cache" content="no-cache">
```



#### Java

```java
response.setHeader("Cache-Control","no-cache"); 
response.setHeader("Pragma","no-cache"); 
response.setDateHeader("Expires",0); 
```

