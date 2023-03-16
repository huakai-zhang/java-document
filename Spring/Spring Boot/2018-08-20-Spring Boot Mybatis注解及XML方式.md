---
layout:  post
title:   Spring Boot Mybatis注解及XML方式
date:   2018-08-20 09:45:58
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# Spring Boot

---



## 接口

```java
public interface ProductCategoryMapper {

    @Insert("insert into product_category(category_name, category_type) values(#{category_name, jdbcType=VARCHAR}, #{category_type, jdbcType=INTEGER})")
    int insertByMap(Map<String,Object> map);

    @Insert("insert into product_category(category_name, category_type) values(#{categoryName, jdbcType=VARCHAR}, #{categoryType, jdbcType=INTEGER})")
    int insertByObject(ProductCategory productCategory);

    @Select("select * from product_category where category_type = #{categoryType}")
    @Results({
            @Result(column = "category_type", property = "categoryType"),
            @Result(column = "category_id", property = "categoryId"),
            @Result(column = "category_name", property = "categoryName")
    })
    ProductCategory findByCategoryType(Integer categoryType);

    @Update("update product_category set category_name = #{categoryName} where category_type = #{categoryType}")
    int updateByCategoryType(@Param("categoryName") String categoryName, @Param("categoryType")Integer categoryType);

    @Update("update product_category set category_name = #{categoryName} where category_type = #{categoryType}")
    int updateByCategoryObject(ProductCategory productCategory);

    @Delete("delete from product_category where category_type = #{categoryType}")
    int deleteByCategoryType(Integer categoryType);

    ProductCategory selectByCategoryType(Integer categoryType);
}
```

## XML

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.imooc.dataobject.mapper.ProductCategoryMapper">

    <resultMap id="BaseResultMap" type="com.imooc.dataobject.ProductCategory">
        <id column="category_id" property="categoryId" jdbcType="INTEGER"/>
        <result column="category_name" property="categoryName" jdbcType="VARCHAR" />
        <result column="category_type" property="categoryType" jdbcType="INTEGER" />
    </resultMap>

    <select id="selectByCategoryType" resultMap="BaseResultMap" parameterType="java.lang.Integer">
        select category_id, category_name, category_type
        from product_category
        where category_type = #{categoryType, jdbcType = INTEGER}
    </select>
</mapper>
```

## 测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Slf4j
public class ProductCategoryMapperTest {

    @Autowired
    private ProductCategoryMapper mapper;

    @Test
    public void insertByMap() {
        Map<String, Object> map = new HashMap<>();
        map.put("category_name", "不爱");
        map.put("category_type", 101);
        int result = mapper.insertByMap(map);
        Assert.assertEquals(1, result);
    }

    @Test
    public void insertByObject() {
        ProductCategory productCategory = new ProductCategory();
        productCategory.setCategoryName("不爱");
        productCategory.setCategoryType(102);
        int result = mapper.insertByObject(productCategory);
        Assert.assertEquals(1, result);
    }

    @Test
    public void findByCategoryType() {
        ProductCategory result = mapper.findByCategoryType(101);
        Assert.assertNotNull(result);
    }

    @Test
    public void updateByCategoryType() {
        mapper.updateByCategoryType("不带分类", 102);
    }

    @Test
    public void updateByObject() {
        ProductCategory productCategory = new ProductCategory();
        productCategory.setCategoryName("不爱");
        productCategory.setCategoryType(102);
        mapper.updateByCategoryObject(productCategory);
    }

    @Test
    public void deleteByCategoryType() {
        mapper.deleteByCategoryType(102);
    }

    @Test
    public void selectByCategoryType() {
        ProductCategory productCategory = mapper.selectByCategoryType(101);
        Assert.assertNotNull(productCategory);
    }
}
```

