---
layout:  post
title:   SpringBoot开发实战
date:   2018-07-03 17:09:34
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# Spring Boot

---



## @DynamicUpdate注解

```java
@Entity
public class ProductCategory {

    /** 类目id. */
    @Id
    @GeneratedValue
    private Integer categoryId;

    /** 类目名字. */
    private String categoryName;

    /** 类目编号. */
    private Integer categoryType;
}

@Test
    public void saveTest() {
        ProductCategory productCategory = new ProductCategory();
        productCategory.setCategoryId(2);
        productCategory.setCategoryName("男生最爱");
        productCategory.setCategoryType(3);
        repository.save(productCategory);
    }
```

update_time会进行更新。

```java
@Entity
public class ProductCategory {

    /** 类目id. */
    @Id
    @GeneratedValue
    private Integer categoryId;

    /** 类目名字. */
    private String categoryName;

    /** 类目编号. */
    private Integer categoryType;

    private Date createTime;

    private Date updateTime;
}
@Test
    public void saveTest() {
        ProductCategory productCategory = repository.findOne(1);
        productCategory.setCategoryType(1);
        repository.save(productCategory);
    }
```

update_time不会更新。

在ProductCategory添加@DynamicUpdate，再次进行修改，update_time就会进行修改（需要修改为其他categoryType的值）。  @DynamicUpdate表示update对象的时候,生成动态的update语句,如果这个字段的值是null就不会被加入到update语句中。

## 快捷getter and setter

添加依赖

```java
<dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
```

以及插件lombok plugins

在ProductCategory添加@Date注解，就不许在生成getter和setter方法，toString()等方法。  如果只需要getter方法，只需添加@Getter，setter同理。

## javax.transaction.Transactional

测试成功后，测试数据不影响数据库数据。与service中的事务不同，这里的Transactional在测试完成后，无论是否发生异常，都会将所有数据库操作回滚。

```java
@Test
    @Transactional
    public void saveTest() {
        ProductCategory productCategory = new ProductCategory("男生最爱", 4);
        ProductCategory result = repository.save(productCategory);
        // 断言方法
        Assert.assertNotNull(result);
        //Assert.assertNotEquals(null, result);
    }
```

## @JsonProperty

```java
@JsonProperty("id")
private String productId;
```

productId为了让我们看清楚字段的含义，id为借口要求的返回值。

## BeanUtils

Spring提供的一个工具类：  BeanUtils.copyProperties(productInfo, productInofVO);  把productInfo对象的属性值copy到productInofVO中去。

## 分页

org.springframework.data.domain.Page

```java
public interface OrderMasterRepository extends JpaRepository<OrderMaster, String> {

    Page<OrderMaster> findByBuyerOpenid(String buyerOpenid, Pageable pageable);

}
```

测试：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class OrderMasterRepositoryTest {

    @Autowired
    private OrderMasterRepository repository;

    private final String OPENID = "110110";

    @Test
    public void findByBuyerOpenid() {
        // 第一个参数表示第几页，第二个表示一页几个
        PageRequest request = new PageRequest(0,1);

        Page<OrderMaster> result = repository.findByBuyerOpenid(OPENID, request);
        Assert.assertNotEquals(0, result.getTotalElements());
    }
}
```

## @Transient

javax.persistence.Transient  在与数据库对照的实体类中，某个字段添加这个注解，会忽略掉这个字段。

## 日期转换之JsonSerialize

```java
public class Date2LongSerializer extends JsonSerializer<Date> {

    @Override
    public void serialize(Date date, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException, JsonProcessingException {
        jsonGenerator.writeNumber(date.getTime() / 1000);
    }
}

@Data
public class OrderDTO {

    /** ... */

    @JsonSerialize(using = Date2LongSerializer.class)
    private Date createTime;

    @JsonSerialize(using = Date2LongSerializer.class)
    private Date updateTime;
}
```

在返回结果OrderDTO时，会处理器时间。

## 实体类的参数查询到的为null的不显示

单个实体类配置，类头添加注解@JsonInclude(JsonInclude.Include.NON_NULL)

全局配置：

```java
spring:
  jackson:
    default-property-inclusion: non_null
```

#### 忽略方法

@JsonIgnore

## RestTemplate

## freemarker

依赖：

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

```java
@Controller
@RequestMapping("/seller/order")
public class SellerOrderController {


    @GetMapping("/list")
    public ModelAndView list() {
        return new ModelAndView("order/list");
    }

}
```

建立文件夹以及文件：resources/templates/order/list.ftl

## http://www.ibootstrap.cn/

## 异常捕获

```java
@ControllerAdvice
public class SellexceptionHandler {

    @ExceptionHandler(value = SellException.class)
    @ResponseBody
    public ResultVO handlerSellerException(SellException e) {
        return ResultVOUtil.error(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(value = SellerAuthorizeException.class)
    @ResponseStatus(HttpStatus.FORBIDDEN)
    public void handlerSellerAuthorizeException(SellerAuthorizeException e) {

    }
}
```

