# JsonParseException: Illegal character ((CTRL-CHAR, code 0))
```java
// 错误代码
redisTemplate.opsForValue().set("user:1", "老张", TimeUnit.SECONDS.toSeconds(2));
log.info(redisTemplate.opsForValue().get("user:1").toString());

// 正确写法
redisTemplate.opsForValue().set("user:1", "老张", 2, TimeUnit.SECONDS);
log.info(redisTemplate.opsForValue().get("user:1").toString());
```
