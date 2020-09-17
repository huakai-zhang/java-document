---
layout:  post
title:   Ajax动态提交参数名和参数值(SpingMVC为例)
date:   2017-11-29 10:33:12
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-日常积累

---

以修改商品是否特价，是否新品功能为例：


```java
<html>  
<head>  
    <script type="text/javascript" src="jquery.min.js"></script>  
</head>  
<body>  
<button οnclick="changeStatus('tejia','1',true)">点击后将原特价商品设置为不是特价商品</button>
<button οnclick="changeStatus('xinpin','1',true)">点击后将原新品设置为不是新品</button> 
<script type="text/javascript">
    function changeStatus(statusName,productID,status) {
        var dataParams = {};
        status = !status;
        dataParams[statusName] = status;
        dataParams["productID"] = productID;
        $.ajax({
            url : "change_product_status",
            type : "post",
            data : dataParams,
            async : false,
            dataType : "json",
            success : function(data){

            }
        });
    }
</script>  
</body>  
</html>
```
后台Controller直接使用Product实体类接收：


```java
@RequestMapping(value = "change_product_status",method = RequestMethod.POST,produces = ResponseContentType.APPLICATION_JSON)
    @ResponseBody
    public String changeProductStatus(Product product){
        return null;
    }
```
product实体类：


```java
public class product{
    private Integer productID;
    private Boolean tejia;
    private Boolean xinpin;

    public Integer getProductID() {
        return productID;
    }

    public void setProductID(Integer productID) {
        this.productID = productID;
    }

    public Boolean getTejia() {
        return tejia;
    }

    public void setTejia(Boolean tejia) {
        this.tejia = tejia;
    }

    public Boolean getXinpin() {
        return xinpin;
    }

    public void setXinpin(Boolean xinpin) {
        this.xinpin = xinpin;
    }
}
```


此时，Product接收到的参数点击第一个按钮时：

productID = 1,tejia = false

点击第二个按钮时：

productID = 1,xinpin = false

