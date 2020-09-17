---
layout:  post
title:   mybatis 动态sql之foreach
date:   2017-09-06 16:39:57
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# Mybatis

---
















```java
<select id="selectPostIn" resultType="domain.blog.Post"> 
	SELECT * FROM POST P WHERE ID in 
	<foreach item="item" index="index" collection="list" open="(" separator="," close=")"> 
		#{item} 
	</foreach> 
</select>
```

