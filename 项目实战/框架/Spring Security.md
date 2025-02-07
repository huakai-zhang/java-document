# 1 Spring Security iframe 的安全配置

在 Spring 中的 frameOptions 配置为 iframe 的安全配置。
X-Frame-Options 头主要是为了防止站点被别人劫持，所以 iframe 将会在 Spring Security 中默认是拒绝设置的。以防止点击劫持攻击。

要修改这个配置，你可以在 Spring 安全配置中进行下面的配置：

```java
// 完全允许 iframe
httpSecurity.headers().frameOptions().disable();
```

或者也可以配置下面：

```java
// X-Frame-Options 有三个值：
// 1. DENY 表示该页面不允许在 frame 中展示，即便是在相同域名的页面中嵌套也不允许
// 2. SAMEORIGIN 表示该页面可以在相同域名页面的 frame 中展示
// 3. ALLOW-FROM uri 表示该页面可以在指定来源的 frame 中展示
httpSecurity.headers().frameOptions().sameOrigin();
```

# 2 Spring Security 中的 CSRF

CSRF（Cross-site request forgery）跨站请求伪造，也被称为“OneClick Attack” 或者 Session Riding。通过伪造用户请求访问受信任站点的非法请求访问。
从Spring Security4开始CSRF防护默认开启，默认会拦截请求，进行CSRF处理。CSRF为了保证不是其他第三方网站访问，要求访问时携带参数名为\_csrf值为token(token在服务端产生)的内容，如果token和服务端的token匹配成功，则正常访问。

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .headers().frameOptions().disable();
    }
}

```

