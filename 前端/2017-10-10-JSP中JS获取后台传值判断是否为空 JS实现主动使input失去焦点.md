---
layout:  post
title:   JSP中JS获取后台传值判断是否为空 JS实现主动使input失去焦点
date:   2017-10-10 13:50:47
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-日常积累

---

```java
$(function () {
        if(${num == null}){
            alert("num为空！");
            return;
        }
        alert(${num});
    });
```





主要解决在手机端，input输入完成，键盘还是弹出状态。
```java
var input = document.getElementById("input-id");
input.blur();
```

