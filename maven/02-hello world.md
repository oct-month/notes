# 用maven新建一个工程

## 一、手动创建

1、编写pom.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 常量，用maven必配 -->
    <modelVersion>4.0.0</modelVersion>

    <!-- gav坐标，maven认为世界上的所有项目都可以通过gav唯一的确定 -->
    <groupId>cn.sun</groupId>       <!-- 表示你是哪个组织 -->
    <artifactId>demo</artifactId>   <!-- 你的项目是什么 -->
    <packaging>jar</packaging>      <!-- 生成jar，还可以选择war、ear -->
    <version>1.0.0-SNAPSHOT</version>        <!-- 版本号 -->
    <!-- X.X.X-里程碑 -->
    <!--第一个X 大版本 有重大变革
 		第二个X 小版本 修复Bug，增加功能
		第三个X 各种更新
		里程碑：
            SNAPSHOT 快照，开发版
            alpha 内部测试
            beta 公开测试
            Release | RC 发布版
            GA 正式版本 -->

    <!-- 依赖 -->
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.9</version>
            <!-- scope为test，表示这个依赖只在测试时使用，打包后不需要，所以发布后就没有了 -->
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```

2、建立maven项目的目录结构

| 目录                          | 目的                                  |
| ----------------------------- | ------------------------------------- |
| ${basedir}                    | 项目根目录，存放pom.xml和所有的子目录 |
| ${basedir}/src/main/java      | 项目的java源码                        |
| ${basedir}/src/main/resources | 项目的资源文件，比如properties文件    |
| ${basedir}/src/test/java      | 项目的测试类，比如junit代码           |
| ${basedir}/src/test/resources | 测试使用的资源文件                    |

3、编写Hello World程序类

src/main/java/cn/sun/demo/HelloMaven.java

```java
package cn.sun.demo;

public class HelloMaven
{
    public void hello()
    {
        System.out.println("Hello Maven");
    }
}
```

4、在项目根目录下运行 mvn compile

maven会自动下载依赖，然后编译。

编译成功后，项目根目录下会出现一个target目录

target/classes目录下就是编译好的类

mvn clean 命令可以清除编译产生的文件

5、编写测试类

src/test//java/cn/sun/demo/HelloTest.java

```java
package cn.sun.demo;

import org.junit.Test;

public class HelloTest
{
    private HelloMaven hm = new HelloMaven();
    @Test
    public void test()
    {
        hm.hello();
    }
}
```

6、运行测试 mvn test

target/surefire-reports目录下会生成测试报告



## 二、命令创建

1、在cmd中运行：

```sh
mvn -B archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes  -DgroupId=cn.sun -DartifactId=demo
```

-DgroupId后指定的是groupId的值

-DartifactId指定的是artifactId的值

自动产生的xml长这样：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>cn.sun</groupId>
  <artifactId>demo</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>demo</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

2、也可以直接运行：

```sh
mvn archetype:generate
```

然后回提示选项，自行选择即可。



## 三、mvn的常用命令

```sh
mvn compile			# 编译源代码
mvn test-compile	# 编译测试代码
mvn test			# 先编译后运行测试
mvn site			# 产生site
mvn package			# 打包
mvn install			# 把项目安装到本地的respository，之后就可以在其它项目中的依赖里引用了
mvn clean			# 清除产生的项目
mvn deploy			# 发布到remote
mvn eclipse:eclipse	# 生成eclipse项目
mvn idea:idea		# 生成idea项目
mvn eclipse:clean	# 清除eclipse的一些系统设置
```

