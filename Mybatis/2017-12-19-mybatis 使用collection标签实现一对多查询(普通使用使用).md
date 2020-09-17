---
layout:  post
title:   mybatis 使用collection标签实现一对多查询(普通使用使用)
date:   2017-12-19 17:17:16
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# Mybatis

---

Mapper：


```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="OrderHeaderMapper">
    <resultMap id="OrderView_M" type="OrderView_M">
        <id column="order_id" property="orderID" jdbcType="INTEGER"/>
        <result column="order_number" property="orderNumber" jdbcType="VARCHAR"/>
        <collection property="orderDetail" ofType="OrderDetailView_M">
            <id column="order_detail_id" property="orderDetailID" jdbcType="INTEGER"/>
            <result column="product_name" property="productName" jdbcType="VARCHAR"/>
        </collection>
    </resultMap>
    <select id="getOrderListPage" resultMap="OrderView_M">
        SELECT
            oh.order_id,
            oh.order_number,
            od.order_detail_id,
            od.product_name
        FROM
            order_header oh
        LEFT JOIN order_detail od ON od.order_id = oh.order_id
        ORDER BY
            oh.order_date DESC
    </select>
</mapper>
```


```java

```

```java
public class OrderView_M {  
    private Integer orderID;  
    private String orderNumber;   
    private List<OrderDetailView_M> orderDetail;  
  
    public Integer getOrderID() {  
        return orderID;  
    }  
  
    public void setOrderID(Integer orderID) {  
        this.orderID = orderID;  
    }  
  
    public String getOrderNumber() {  
        return orderNumber;  
    }  
  
    public void setOrderNumber(String orderNumber) {  
        this.orderNumber = orderNumber;  
    }  
  
    public List<OrderDetailView_M> getOrderDetail() {  
        return orderDetail;  
    }  
  
    public void setOrderDetail(List<OrderDetailView_M> orderDetail) {  
        this.orderDetail = orderDetail;  
    }  
}  

public class OrderDetailView_M {  
    private Integer orderDetailID;  
    private String productName;  
  
    public String getProductName() {  
        return productName;  
    }  
  
    public void setProductName(String productName) {  
        this.productName = productName;  
    }  

    public Integer getOrderDetailID() {  
        return orderDetailID;  
    }  
  
    public void setOrderDetailID(Integer orderDetailID) {  
        this.orderDetailID = orderDetailID;  
    }  
}
```


