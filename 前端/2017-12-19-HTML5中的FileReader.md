---
layout:  post
title:   HTML5中的FileReader
date:   2017-12-19 17:57:11
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-日常积累
-html5
-FileReader
-图片上传预览

---



## 调用FileReader对象的方法

FileReader无论读取成功或失败，方法并不会返回读取结果，这一结果存储在 result属性中。


**readAsText：**该方法有两个参数，其中第二个参数是文本的编码方式，默认值为 UTF-8。这个方法非常容易理解，将文件以文本方式读取，读取的结果即是这个文本文件中的内容。  **readAsBinaryString：**该方法将文件读取为二进制字符串，通常我们将它传送到后端，后端可以通过这段字符串存储文件。  **readAsDataURL：**这是例子程序中用到的方法，该方法将文件读取为一段以 data: 开头的字符串，这段字符串的实质就是 Data URL，Data URL是一种将小文件直接嵌入文档的方案。这里的小文件通常是指图像与 html 等格式的文件。

## 处理事件

FileReader 包含了一套完整的事件模型，用于捕获读取文件时的状态，下面这个表格归纳了这些事件。


文件一旦开始读取，无论成功或失败，实例的 result 属性都会被填充。如果读取失败，则 result 的值为 null ，否则即是读取的结果，绝大多数的程序都会在成功读取文件的时候，抓取这个值。

```java
fr.onload = function() {  
    this.result;  
};
```

一个简单的图片上传预览功能：

```java
<html>
<head>
    <meta charset="utf-8">
    <title>FileReader</title>
    <script src="jquery.min.js"></script>
</head>
<body>
<section>
    <div>
        <input type="file" accept="image/*" onchange="previewImg(this);"/>
        <div class="current"></div>
    </div>
</section>
<script type="text/javascript">
    function previewImg(fileDom) {
        console.log(fileDom);
        if (fileDom.files && fileDom.files[0]) {
            var reader = new FileReader();
            reader.onload = function (e) {
                var $img = $("<img src='" + e.target.result + "'>");
                $('.current').text('').append($img);
            }
            reader.readAsDataURL(fileDom.files[0]);
        }
    }
</script>
</body>
</html>
```


效果：  ![img](https://img-blog.csdn.net/20171219175612204?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3N6Y3kxOTk1MDM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

