# mybatis开端

1、什么是mybatis

​		是一个基于java的持久层框架。

2、持久化：数据从瞬时状态转变为持久状态

3、持久层：完成持久化工作的代码块。

4、mybatis就是帮助程序员将数据存入数据库中，和从数据库中取数据。

5、传统的jdbc操作：有很多重复代码块。通过框架可以减少重复代码，提高开发效率。

6、mybatis是一个半自动化的ORM框架。

7、mybatis的功能：

​		mybatis支持普通的SQL查询，存储过程和高级映射。消除了几乎所有的jdbc代码和参数的手工设置以及结果集的检索。

8、如何使用mybatis？

​		a) 导入mybatis相关jar包

```sh
asm-7.1.jar
cglib-3.3.0.jar
commons-logging-1.2.jar
javassist-3.27.0-GA.jar
log4j-1.2.17.jar
log4j-api-2.13.3.jar
log4j-core-2.13.3.jar
mybatis-3.5.5.jar			# mybatis核心包，其它的都是依赖包
ognl-3.2.14.jar
postgresql-42.2.14.jar		# 数据库驱动jar包
slf4j-api-1.7.30.jar
slf4j-log4j12-1.7.30.jar
```

​		b) 编写mybatis核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="org.postgresql.Driver"/>
                <property name="url" value="jdbc:postgresql://127.0.0.1:5432/test"/>
                <property name="username" value="postgres"/>
                <property name="password" value="postgres"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="mapper.xml"/>
    </mappers>
</configuration>
```

​		c) 创建SqlSessionFactory，以及获得SqlSession。

```java
import java.io.IOException;
import java.io.InputStream;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

public class MyBatisUtil
{
    public static SqlSessionFactory getsqlSessionFactory() throws IOException
    {
        String resource = "mybatis.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        return sqlSessionFactory;
    }
    public static SqlSession getSession() throws IOException
    {
        SqlSessionFactory sqlSessionFactory = getsqlSessionFactory();
        return sqlSessionFactory.openSession();
    }
}
```

​		d) 创建实体类

```java
public class Student
{
    private int ID;
    private String name;
    private int age;

    public int getID()
    {
        return ID;
    }

    public void setID(int ID)
    {
        this.ID = ID;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    public int getAge()
    {
        return age;
    }

    public void setAge(int age)
    {
        this.age = age;
    }
}

```

​		e) 编写sql语句的映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="studentMapper">
    <select id="selectStudent" resultType="Student">
        select * from student where id = #{id}
    </select>
</mapper>
```

​		f) 测试

```java
import org.apache.ibatis.session.SqlSession;

public class NTest
{
    public static void main(String[] args) throws Exception
    {
        SqlSession session = MyBatisUtil.getSession();
        Student student = session.selectOne("studentMapper.selectStudent", 0);
        System.out.println(student);
        session.close();
    }
}
```



