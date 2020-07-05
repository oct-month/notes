# 一、Spring Boot 入门

## 1、Spring Boot 简介

> 简化Spring应用开发的一个框架
>
> 整个Spring技术栈的一个大整合
>
> J2EE开发的一站式解决方案



## 2、微服务

2014，martin fowler

微服务：架构风格

一个应用应该是一组小型服务，可以通过HTTP的方式进行互通

每一个功能元素最终都是一个可独立替换和独立升级的软件单元



## 3、Hello World

浏览器发送GET请求，服务器接受请求并处理，响应Hello World字符串。

a) 创建一个maven工程

b) 导入相关依赖

​	pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>org.example</groupId>
    <artifactId>boot-01</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>boot-01</name>
    <description>Spring boot的Hello World程序</description>

    <properties>
        <java.version>14</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <!-- 这个插件，可以将应用打包成一个可执行jar包 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

c) 编写一个主程序

```java
package com.sun;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @SpringBootApplication 用来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class AppMain
{
    public static void main(String[] args)
    {
        // 启动 Spring 应用
        SpringApplication.run(AppMain.class, args);
    }
}
```

d) 编写相关的Controller、Service

```java
package com.sun.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController
{
    @ResponseBody
    @RequestMapping("/hello")
    public String hello()
    {
        return "Hello World";
    }
}
```

e) 启动AppMain

f) 简化部署

```xml
<!-- 这个插件，可以将应用打包成一个可执行jar包 -->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

将这个project打成jar包，直接使用 java -jar 的命令进行执行。



## 4、主程序类，主入口类

```java
/**
 * 标注一个主程序类
 */
@SpringBootApplication
public class HelloApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(HelloApplication.class, args);
    }
}
```



## 5、自动配置

@**SpringBootApplication**: Spring Boot应用。标注在某个类上说明这个类是Spring Boot的主配置类，Spring Boot就应该运行这个类的main方法来启动Spring Boot应用。

```java
// SpringBootApplication上面的注解
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
```

@**SpringBootConfiguration**: Spring Boot的配置类。标注在某个类上表示这是一个Spring Boot的配置类。在SpringBootConfiguration上又有@Configuration注解，代替原来的xml配置文件。

@**EnableAutoConfiguration**：开启自动配置功能。以前我们需要自己配置的东西，Spring Boot帮我们自动配置。

```java
// EnableAutoConfiguration上面的注解
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
```

@**AutoConfigurationPackage**：自动配置包。<u>*将主配置类@SpringBootApplication标注的类所在包及下面所有子包里面的所有组件扫描到Spring容器*</u>；

```java
// AUtoConfigurationPackage上面的注解
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})
```

@**Import**：Spring的底层注解@Import，给容器中导入一个组件；导入的组件由Registrar这个类来指定该导入哪些组件。

@**Import({AutoConfigurationImportSelector.class})**，导入哪些组件的选择器。

将所有需要导入的组件以全类名的方式放回，这些组件就会被添加到容器中。会给容器中导入非常多的自动配置类（**AutoConfiguration），就是给容器导入这个场景需要的所有组件，并配置好这些组件。

有了自动配置类，免去了我们手动编写配置注入功能组件等的工作。

SpringBoot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作。

J2EE的整体解决方案和自动配置都在spring-boot-auto-configuration.jar中。



