---
layout:  post
title:   JS(history,location的使用)返回上一页以及返回上一页并刷新
date:   2017-10-11 16:00:26
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-日常积累

---

```java
window.history.go(-1);//返回上一页不刷新
window.history.back();  //返回上一页  
window.location.href = document.referrer;//返回上一页并刷新，真正实现页面后退并刷新页面
history.go(1);//前进一页
history.forward(); //前进一页
history.length;//用length属性查看历史中的页面数
```



因为windows对象引用不是必须的。所以windows.history.go() == history.go()的。


```java
<button οnclick="history.go(-1);">返回上一页</button>
<a href="javascript:history.back();">返回上一页</a>
```
**history.go(-1)和history.back()的区别** 1.history.go(-1)表示后退与刷新。如数据有改变也随之改变 2.history.back()只是单纯的返回到上一页。**Javascript刷新页面的几种方法：**


```java
history.go(0);
location.reload();
location=location;
location.assign(location);
document.execCommand('Refresh');
window.navigate(location);
location.replace(location);
document.URL=location.href ;
```

