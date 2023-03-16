---
layout:  post
title:   Redis 缓存的 Java 实现
date:   2018-08-20 17:08:23
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# 分布式

---



## 使用Redis缓存

1.在启动类上加@EnableCaching注解 2.Controller加缓存

```java
@GetMapping("list")
@Cacheable(cacheNames = "product", key = "123")
public ResultVO list() {
        //1. 查询所有商家商品
        List<ProductInfo> productInfoList = productService.findUpAll();
        //2. 查询类目（一次性查询）
        List<Integer> categoryTypeList = productInfoList.stream()
                .map(e -> e.getCategoryType())
                .collect(Collectors.toList());

        List<ProductCategory> productCategoryList = categoryService.findByCategoryTypeIn(categoryTypeList);
        //3. 数据拼装
        List<ProductVO> productVOList = new ArrayList<>();
        for (ProductCategory productCategory : productCategoryList) {
            ProductVO productVO = new ProductVO();
            productVO.setCategoryType(productCategory.getCategoryType());
            productVO.setCategoryName(productCategory.getCategoryName());

            List<ProductInofVO> productInofVOList = new ArrayList<>();
            for (ProductInfo productInfo : productInfoList) {
                if (productInfo.getCategoryType().equals(productCategory.getCategoryType())) {
                    ProductInofVO productInofVO = new ProductInofVO();
                    BeanUtils.copyProperties(productInfo, productInofVO);
                    productInofVOList.add(productInofVO);
                }
            }
            productVO.setProductInofVOList(productInofVOList);
            productVOList.add(productVO);
        }

        return ResultVOUtil.success(productVOList);
    }
```

3.controller返回的类需要序列化

## IDEA序列化序列码生成插件：GenerateSeriaVersionUID

## 使用 RedisDesktopManager 链接服务器Redis

使用RedisDesktopManager时，会出现Can’t connect to redis-server的错误，要想使用RedisDesktopManager成功链接服务器Redis，需要进行下面几步配置：

1. 服务器开启6379端口 
2. 修改redis.conf

```java
# 表明所有域名均可访问
bind 0.0.0.0
# 表示在登录时不需要密码
protected-mode yes no
```

1. 使用非守护方式启动redis

```java
cd /usr/local/redis-4.0.9/src
./redis-server /usr/local/redis-4.0.9/redis.conf
```

1.  测试链接  
2.  访问controller中的list路径展示结果 

第二次访问不会在执行list方法中的代码，直接从redis中获取数据，但是修改了商品之后redis不会重新获取，仍然显示旧数据

## 使用CacheEvict

在修改商品的controller上添加@CacheEvict(cacheNames = "product", key = "123") 表示修改的时候清除服务器redis中的该缓存。

## 使用CachePut

表示在修改操作时更新该redis缓存，但是使用该注解时，设置redis的方法必须和修改的方法返回相同的类。

```java
@Override
    @Cacheable(cacheNames = "product", key = "123")
    public ProductInfo findOne(String productId) {
        return productInfoRepository.findOne(productId);
    }

    @Override
    @CachePut(cacheNames = "product", key = "123")
    public ProductInfo save(ProductInfo productInfo) {
        return productInfoRepository.save(productInfo);
    }
```

## Redis的key

key的默认值为空，如果不写key或者key为空，redis会默认选择方法的参数做为key值。

```java
@Override
    @Cacheable(cacheNames = "product")
    public ProductInfo findOne(String productId) {
        return productInfoRepository.findOne(productId);
    }

    @Override
    @CachePut(cacheNames = "product")
    public ProductInfo save(ProductInfo productInfo) {
        return productInfoRepository.save(productInfo);
    }
```

上诉写法，修改完成之后，获取的依然是之前的数据，因为redis将productId和productInfo做为了key。

## CacheConfig的用法

```java
@Service
@CacheConfig(cacheNames = "product")
public class ProductServiceImpl implements ProductService {

    @Autowired
    private ProductInfoRepository productInfoRepository;

    @Override
    @Cacheable(key = "123")
    public ProductInfo findOne(String productId) {
        return productInfoRepository.findOne(productId);
    }

    @Override
    @CachePut(key = "123")
    public ProductInfo save(ProductInfo productInfo) {
        return productInfoRepository.save(productInfo);
    }
}
```

## 使用SpEL表达式

不同的商家不同的商品列表，动态的加载：

```java
@GetMapping("list")
@Cacheable(cacheNames = "product", key = "#storeId")
public ResultVO list(String storeId) {...}
```

## 使用condition

condition中的条件成立才进行缓存：

```java
@GetMapping("list")
@Cacheable(cacheNames = "product", key = "#storeId", condition = "#storeId.length() > 3")
public ResultVO list(String storeId) {...}
```

## 使用unless

@Cacheable(cacheNames = “product”, key = “#storeId”, condition = “#storeId.length() &gt; 3”, unless = “#result.getCode() != 0”)

unless表示如果不的意思，也就是返回结果的code，如果不不等于0，也就是等于0时才会进行缓存。

