---
layout:  post
title:   mybatis 使用collection标签实现一对多查询(多分页使用)
date:   2017-09-06 16:22:31
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# Mybatis

---

在使用**ListPage分页工具类进行分页操作时，如果使用一对多查询，会造成查询出来每一页数据数量不等于实际规定的每一个数据数量。原因在于，一对多查询的结果是包含了与子表链接的数据，例如在查询10个订单(order_header)数据时，假如有两个订单均包含2条订单条目(order_detail)数据，那么最终查询的10条数据只有8条order_header表的数据。





通常，我们为了解决这种问题，首先会先获取到10条order_header数据，然后在遍历订单数据分别获取每一条订单数据的订单条目数据，这样会浪费数据库的执行时间，下面使用collection进行查询的方法，可以避开普通collection查询造成的问题，也能达到其效果。



**Mapper：**



```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="OrderHeaderMapper">
    <resultMap id="OrderView_M" type="OrderView_M">
        <id column="order_id" property="orderID" jdbcType="INTEGER"/>
        <result column="order_number" property="orderNumber" jdbcType="VARCHAR"/>
        <result column="total_amount" property="totalAmount" jdbcType="DECIMAL"/>
	<result column="pay_status_id" property="payStatusID"/>
        <result column="order_status_id" property="orderStatusID"/>
        <collection property="orderDetail" column="order_id" ofType="OrderDetailView_M" javaType="ArrayList" select="getOrderDetailByOrderID">

        </collection>
    </resultMap>

	<resultMap id="OrderDetailView_M" type="OrderDetailView_M">
		<id column="order_detail_id" property="orderDetailID" jdbcType="INTEGER"/>
		<result column="product_name" property="productName" jdbcType="VARCHAR"/>
		<result column="photo_path" property="photoPath" jdbcType="VARCHAR"/>
	</resultMap>
	<!-- page为一个自定义分页类，其中包括一个pd的map -->
	<select id="getOrderByCustomerListPage" parameterType="page" resultMap="OrderView_M">
        	SELECT
			oh.order_id,
			oh.order_number,
			oh.total_amount,
			oh.pay_status_id,
            		oh.order_status_id
		FROM
			order_header oh
		WHERE oh.pay_status_id = #{pd.orderStatusID}
		ORDER BY oh.order_date DESC
    	</select>

	<select id="getOrderDetailByOrderID" parameterType="java.lang.Integer" resultMap="OrderDetailView_M">
		SELECT
		od.order_detail_id,
		od.product_name,
		od.photo_path,
		od.order_id
		FROM
		order_detail od
		LEFT JOIN product p ON p.product_id = od.product_id
		WHERE od.order_id = #{order_id}
		ORDER BY p.sku
	</select>
</mapper>
```






```java
public class OrderView_M {
    private Integer orderID;
    private String orderNumber;
    private BigDecimal totalAmount;
    private Integer payStatusID;
    private Integer orderStatusID;
    private List<OrderDetailView_M> orderDetail;

    public Integer getPayStatusID() {
        return payStatusID;
    }

    public Integer getOrderStatusID() {
        return orderStatusID;
    }

    public void setOrderStatusID(Integer orderStatusID) {
        this.orderStatusID = orderStatusID;
    }

    public void setPayStatusID(Integer payStatusID) {
        this.payStatusID = payStatusID;
    }

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

    public BigDecimal getTotalAmount() {
        return totalAmount;
    }

    public void setTotalAmount(BigDecimal totalAmount) {
        this.totalAmount = totalAmount;
    }

    public List<OrderDetailView_M> getOrderDetail() {
        return orderDetail;
    }

    public void setOrderDetail(List<OrderDetailView_M> orderDetail) {
        this.orderDetail = orderDetail;
    }
}
```






```java
public class OrderDetailView_M {
    private Integer orderDetailID;
    private String productName;
    private String photoPath;

    public String getProductName() {
        return productName;
    }

    public void setProductName(String productName) {
        this.productName = productName;
    }

    public String getPhotoPath() {
        return photoPath;
    }

    public void setPhotoPath(String photoPath) {
        this.photoPath = photoPath;
    }

    public Integer getOrderDetailID() {
        return orderDetailID;
    }

    public void setOrderDetailID(Integer orderDetailID) {
        this.orderDetailID = orderDetailID;
    }
}
```


