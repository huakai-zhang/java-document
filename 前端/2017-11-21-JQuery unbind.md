---
layout:  post
title:   JQuery unbind
date:   2017-11-21 10:27:48
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-日常积累

---

```java
$("button").click(function(){
  $("p").unbind();
});
```



**定义和用法** unbind() 方法移除被选元素的事件处理程序。 该方法能够移除所有的或被选的事件处理程序，或者当事件发生时终止指定函数的运行。 ubind() 适用于任何通过 jQuery 附加的事件处理程序。

**取消绑定元素的事件处理程序和函数** 规定从指定元素上删除的一个或多个事件处理程序。 如果没有规定参数，unbind() 方法会删除指定元素的所有事件处理程序。

**语法** $(selector).unbind(event,function) event可选。规定删除元素的一个或多个事件由空格分隔多个事件值。如果只规定了该参数，则会删除绑定到指定事件的所有函数。 function可选。规定从元素的指定事件取消绑定的函数名。

**使用 Event 对象来取消绑定事件处理程序** 规定要删除的事件对象。用于对自身内部的事件取消绑定（比如当事件已被触发一定次数之后，删除事件处理程序）。 如果未规定参数，则 unbind() 方法会删除指定元素的所有事件处理程序。



**示例：**



```java
<html>
<head>
    <script type="text/javascript" src="jquery.min.js"></script>
</head>
<body>
<input type="button" id="btn" value="点击我"/>
<input type="button" id="delAll" value="删除全部绑定函数"/>
<input type="button" id="delFun2" value="删除第二个绑定函数"/>
<br/>
<div class="info"></div>
<script type="text/javascript">
    $(document).ready(function () {
        // 为id为btn的按钮添加绑定事件
        $("#btn").bind('click', fun1 = function () {
            $(".info").append('<p>绑定函数1</p>');
        }).bind('click', fun2 = function () {
            $(".info").append('<p>绑定函数2</p>');
        }).bind('click', fun3 = function () {
            $(".info").append('<p>绑定函数3</p>');
        })
        $("#delAll").bind('click', function () {
            $("#btn").unbind(); //删除全部绑定事件
        })
        $("#delFun2").bind('click', function () {
            $("#btn").unbind('click', fun2);  //删除第二个绑定函数
        })
    })
</script>
</body>
</html>
```


**语法** $(selector).unbind(eventObj) eventObj可选。规定要使用的事件对象。这个 eventObj 参数来自事件绑定函数。



示例：



```java
<html>
<head>
    <script type="text/javascript" src="jquery.min.js"></script>
    <script type="text/javascript">
        $(document).ready(function(){
            var x=0;
            $("p").click(function(e){
                $("p").animate({fontSize:"+=5px"});
                x++;
                if (x>=2)
                {
                    $(this).unbind(e);
                }
            });
        });
    </script>
</head>
<body>
<p style="font-size:20px;">点击这个段落可以增加其大小。只能增加两次。</p>
</body>
</html>
```





