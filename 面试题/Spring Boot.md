#### 出Springboot项目启动的几种方式吗？

1. main

2. java -jar jar包路径 --参数(--server.port=8081)

3. spring-boot-maven-plugin

   mvn spring-boot:run 

   mvn spring-boot:help -Ddetail 

   mvn spring-boot:run -Drun.arguments="--server.port=8888" 

