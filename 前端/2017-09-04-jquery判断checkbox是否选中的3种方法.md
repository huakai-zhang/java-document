---
layout:  post
title:   jquery判断checkbox是否选中的3种方法
date:   2017-09-04 13:32:38
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-日常积累

---


```java
if ($("#checkbox-id")get(0).checked) {
    // do something
}
```

```java
if($('#checkbox-id').is(':checked')) {
    // do something
}
```

```java
if ($('#checkbox-id').attr('checked')) {
    // do something
}
```
实例：

默认地址的变化：



```java
if($("#defaultAddress").attr("checked")){
            $("#defaultAddress").attr("checked",false);
        }else{
            $("#defaultAddress").attr("checked", true);
        }
```


