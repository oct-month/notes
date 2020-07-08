# profile

## 1、多profile文件

在编写主配置文件时，文件名可以是 application-{profile}.properties/yml

默认使用application.properties/yml



## 2、yml多文档块

```yaml
spring:
  profiles:
    active: pro		# 激活配置块pro
server:
  port: 8081
---
spring:
  profiles:
    - dev			# 指定属于哪个环境
server:
  port: 8082
---
spring:
  profiles:
    - pro
server:
  port: 8084
```



## 3、激活指定profile

1、在properties文件中指定

```properties
# 激活 application-dev.properties/yml
spring.profiles.active=dev
```

2、命令行激活

```sh
--spring.profiles.active=dev
```

3、JVM参数

```sh
-Dspring.profiles.active=dev
```

