# 整合mybatis

1、步骤：

​	a) 导入相关jar包

```sh
mybatis.jar及其依赖
mybatis-spring.jar
spring-的一系列jar包
```

​	b) 编写配置文件

​	spring的配置   cn/sun/spring/beans.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 配置数据源 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="org.postgresql.Driver"/>
        <property name="url" value="jdbc:postgresql://127.0.0.1:5432/test"/>
        <property name="username" value="postgres"/>
        <property name="password" value="postgres"/>
    </bean>
    <!-- 配置SqlSessionFactory -->
    <bean id="SqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:cn/sun/spring/mybatis.xml"/>
    </bean>
    <!-- 配置SqlSession -->
    <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="SqlSessionFactory"/>
    </bean>
    <bean id="studentDao" class="cn.sun.spring.StudentDaoImp">
        <property name="sqlSession" ref="sqlSessionTemplate"/>
    </bean>

</beans>
```

​	mybatis的配置	cn/sun/spring/mybatis.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <package name="cn.sun.spring"/>
    </typeAliases>
    <mappers>
        <mapper resource="cn/sun/spring/mapper.xml"/>
    </mappers>
</configuration>
```

​	mapper的配置	cn/sun/spring/mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.sun.spring.studentMapper">
    <select id="selectStudent" resultType="Student">
        select * from student
    </select>
</mapper>
```

​	c) 实现	

​	Dao接口

```java
package cn.sun.spring;

import java.util.List;

public interface StudentDao
{
    public List<Student> selectStudent();
}
```

​	Dao的实现

```java
package cn.sun.spring;

import org.mybatis.spring.SqlSessionTemplate;

import java.util.List;

public class StudentDaoImp implements StudentDao
{
    private SqlSessionTemplate sqlSession;
    @Override
    public List<Student> selectStudent()
    {
        return sqlSession.selectList("cn.sun.spring.studentMapper.selectStudent");
    }

    public void setSqlSession(SqlSessionTemplate sqlSession)
    {
        this.sqlSession = sqlSession;
    }
}
```

​	d) 测试

```java
package cn.sun.spring;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.util.List;

public class NTest
{
    @Test
    public void test() throws Exception
    {
        ApplicationContext context = new ClassPathXmlApplicationContext("cn/sun/spring/beans.xml");
        StudentDao dao = context.getBean("studentDao", StudentDao.class);
        List<Student> students = dao.selectStudent();
        for (Student student : students) {
            System.out.println(student);
        }
    }
}
```



2、声明式事务管理

​	/cn/sun/spring/beans.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 配置数据源 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="org.postgresql.Driver"/>
        <property name="url" value="jdbc:postgresql://127.0.0.1:5432/test"/>
        <property name="username" value="postgres"/>
        <property name="password" value="postgres"/>
    </bean>
    <!-- 声明式事务配置 开始 -->
    <!-- 配置事务管理器 -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!-- 配置事务通知 -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <tx:attributes>
            <!-- 配置哪些方法使用什么样的事务，配置事务的传播特性 -->
            <tx:method name="add" propagation="REQUIRED"/>	<!-- 如果存在一个事务，则支持当前事务。不存在则开启 -->
            <tx:method name="remove" propagation="REQUIRED"/>
            <tx:method name="selectStudent" read-only="true"/>	<!-- 只读 -->
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
    <aop:config>
        <aop:pointcut id="pointcut" expression="execution(* cn.sun.spring.StudentDao*.*(..)"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut"/>
    </aop:config>
    <!-- 声明式事务配置 结束 -->
    <!-- 配置SqlSessionFactory -->
    <bean id="SqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:cn/sun/spring/mybatis.xml"/>
    </bean>
    <!-- 配置SqlSession -->
    <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="SqlSessionFactory"/>
    </bean>
    <bean id="studentDao" class="cn.sun.spring.StudentDaoImp">
        <property name="sqlSession" ref="sqlSessionTemplate"/>
    </bean>

</beans>
```







