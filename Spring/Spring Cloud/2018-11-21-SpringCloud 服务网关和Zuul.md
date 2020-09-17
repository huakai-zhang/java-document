---
layout:  post
title:   SpringCloud 服务网关和Zuul
date:   2018-11-21 16:02:42
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-# 分布式

---



## 服务网关的要素

稳定性，高可用 性能，并发性 安全性 扩展性

## 常用的网关方案

Nginx + Lua Kong Tyk Spring Cloud Zuul

## Zuul的特点

路由 + 过滤器 = Zuul 核心是一系列的过滤器

## Zuul路由


1.创建项目 ![img](https://img-blog.csdnimg.cn/20181119101516838.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

2.修改配置

```java
spring:
  application:
    name: api-geteway
  cloud:
    config:
      discovery:
        enabled: true
        service-id: CONFIG
      profile: dev
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/

zuul:
  routes:
    myProduct:
      path: /myProduct/**
      serviceId: product
    #简写方法
    #product: /myProduct/**
  #排除某些路由
  ignored-patterns:
    - /**/product/listForOrder
```

3.启动类添加注解@EnableZuulProxy

#### Cookie传递

```java
zuul:
  routes:
    myProduct:
      path: /myProduct/**
      serviceId: product
      #敏感头默认过滤（Cookie,Set-Cookie,Authorization）,设置为空表示均不过滤
      sensitiveHeaders:
```

zuul.sensitive-headers: 设置全局的敏感头

## Pre和Post过滤器

#### Pre

```java
// token校验
@Component
public class TokenFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return PRE_DECORATION_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletRequest request = requestContext.getRequest();

        // 从参数里获取,也可以从cookie，header里获取
        String token = request.getParameter("token");
        if (StringUtils.isEmpty(token)) {
            requestContext.setSendZuulResponse(false);
            requestContext.setResponseStatusCode(HttpStatus.UNAUTHORIZED.value());
        }
        return null;
    }
}
```


![img](https://img-blog.csdnimg.cn/20181119110318715.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

#### Post

```java
// 返回值中添加Header
@Component
public class AddResponseHeaderFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return POST_TYPE;
    }

    @Override
    public int filterOrder() {
        return SEND_RESPONSE_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletResponse response = requestContext.getResponse();
        response.setHeader("X-Foo", UUID.randomUUID().toString());
        return null;
    }
}
```


![img](https://img-blog.csdnimg.cn/2018111911021874.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

## Zuul限流


时机：请求被转发之前调用 使用令牌桶：（谷歌插件RateLimiterFilter） ![img](https://img-blog.csdnimg.cn/20181119132604419.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzemN5MTk5NTAz,size_16,color_FFFFFF,t_70)

```java
@Component
public class RateLimiterFilter extends ZuulFilter {
	//每秒钟提供100个令牌
    private static  final RateLimiter RATE_LIMITER = RateLimiter.create(100);

    @Override
    public String filterType() {
        return PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return SERVLET_DETECTION_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        if (!RATE_LIMITER.tryAcquire()) {
            throw new RateLimitException();
        }
        return null;
    }
}
```

## Zuul跨域

1.通常Spring的处理方式：在被调用的类或方法上增加@CrossOrigin注解 2.在Zuul里增加CorsFilter过滤器

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

import java.util.Arrays;

/**
 * C - Cross O - Origin R - Resource S - Sharing
 */
@Configuration
public class CorsConfig {

    @Bean
    public CorsFilter corsFilter() {
        final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        final CorsConfiguration config = new CorsConfiguration();

        config.setAllowCredentials(true);
        config.setAllowedOrigins(Arrays.asList("*"));
        config.setAllowedHeaders(Arrays.asList("*"));
        config.setAllowedMethods(Arrays.asList("*"));
        config.setMaxAge(300L);

        source.registerCorsConfiguration("/**", config);

        return new CorsFilter(source);
    }

}
```

