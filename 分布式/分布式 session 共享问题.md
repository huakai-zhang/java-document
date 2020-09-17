## 关于cookie和session的联系
cookie 中会包含：名字、值、过期时间、路径、域等信息。git.spring.com/blog/
cookie 会带到 http 请求头中发送给服务器端。如果 cookie 没有设置过期时间的话，那么 cookie 的默认生命周期是浏览器的会话。
### session机制
1. session 是容器对象，客户端在请求服务端的时候，服务端会根据客户端的请求判断是否包含了 sessionid 的标识
2. 如果已经包含了。说明该客户端之前已经建立了会话。sessionid 是一个唯一的值
3. 如果 sessionid 不存在，那么服务端会为这个客户端生成一个 sessionid。JSESSIONID

# 基于 session 复制
# 基于 session 统一存储
# 基于 cookie 机制
## 基于 JTW 的解决方案
Json Web Token，客户端和服务端信息安全传递以及身份认证的一种解决方案。

### JWT 的组成
jwt 由 3个部分组成：
header
{
type:"jwt",
alg:'HS256'
}
playload
jwt本身规范提供的格式 claims： iss iat exp sub
自定义一些 claims
signature
Base64 去编码
