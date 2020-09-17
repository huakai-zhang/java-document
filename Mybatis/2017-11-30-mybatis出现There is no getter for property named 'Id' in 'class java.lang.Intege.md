---
layout:  post
title:   mybatis出现There is no getter for property named 'Id' in 'class java.lang.Intege
date:   2017-11-30 17:39:08
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# Mybatis

---

当传入数据只有一个时mybatis中&lt;if&gt;判断会出现There is no getter for property named 'Id' in 'class java.lang.Intege

用"_parameter"代替当前参数





```java
<select id="selectOrderById"  parameterType="java.lang.Integer" resultType="java.util.Map">
        select order_id,order_num
        from order
        where deleted = 0 
        <if test="_parameter != null">
            and order_id = #{_parameter,jdbcType=INTEGER}
        </if>
</select>
```


