# 基本的crud操作

1、搭建mybatis框架

​		a) 导入相关jar包

​		b) 编写核心配置文件（配置数据库连接的相关信息和mapper映射文件）

​		c) 编写mapper映射文件

​		d) 编写实体类

​		e) 编写dao操作

2、crud实现内容

数据库建库sql

```plsql
create database test;		-- 创建test数据库
\c test;					-- 连接数据库
create table student(		-- 创建student表
	ID int primary key not null,
    name text not null,
    age int default 0
);
```

mybatis.xml

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

mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="studentMapper">
    <!-- 查询单个学生 -->
    <select id="selectStudent" resultType="Student" parameterType="int">
        select * from student where ID = #{ID}
    </select>
    <!-- 查询所有学生 -->
    <select id="selectAll" resultType="Student">
        select * from student
    </select>
    <!-- 添加学生信息 -->
    <insert id="addStudent" parameterType="Student" useGeneratedKeys="true">
        insert into student(ID, name, age) values(#{ID}, #{name}, #{age})
    </insert>
    <!-- 修改学生信息 -->
    <update id="updateStudent" parameterType="Student">
        update student set name=#{name}, age=#{age} where ID=#{ID}
    </update>
    <!-- 删除学生信息 -->
    <delete id="deleteStudent" parameterType="int">
        delete from student where ID=#{ID}
    </delete>
</mapper>
```

Student.java

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

    @Override
    public String toString()
    {
        return "Student{" +
                "ID=" + ID +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

studentDao.java

```java
import org.apache.ibatis.session.SqlSession;

import java.io.IOException;
import java.util.List;

public class StudentDao
{
    public Student getById(int ID) throws IOException
    {
        SqlSession session = MyBatisUtil.getSession();
        Student student = session.selectOne("studentMapper.selectStudent", ID);
        session.close();
        return student;
    }
    public int add(Student student) throws IOException
    {
        SqlSession session = MyBatisUtil.getSession();
        int result = session.insert("studentMapper.addStudent", student);
        session.commit();   // 提交事务
        session.close();
        return result;
    }
    public int update(Student student) throws IOException
    {
        SqlSession session = MyBatisUtil.getSession();
        int result = session.update("studentMapper.updateStudent", student);
        session.commit();
        session.close();
        return result;
    }
    public int delete(int ID) throws IOException
    {
        SqlSession session = MyBatisUtil.getSession();
        // Student student = session.selectOne("studentMapper.selectStudent", ID);
        int result = session.delete("studentMapper.deleteStudent", ID);
        session.commit();
        session.close();
        return result;
    }
    public List<Student> getAll() throws IOException
    {
        SqlSession session = MyBatisUtil.getSession();
        List<Student> result = session.selectList("studentMapper.selectAll");
        session.close();
        return result;
    }
}
```

NTest.java  测试文件

```java
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class NTest
{
    @Test
    public void test_1() throws Exception
    {
        SqlSession session = MyBatisUtil.getSession();
        Student student = session.selectOne("studentMapper.selectStudent", 0);
        System.out.println(student);
        session.close();
    }
    @Test
    public void test_2() throws Exception
    {
        StudentDao dao = new StudentDao();
        System.out.println(dao.getById(0));
    }
    @Test
    public void test_3() throws Exception
    {
        StudentDao dao = new StudentDao();
        Student student = new Student();
        student.setID(8);
        student.setName("哈哈哈哈");
        student.setAge(90);
        int a = dao.add(student);
        System.out.println(a);
    }
    @Test
    public void test_4() throws Exception
    {
        StudentDao dao = new StudentDao();
        Student student = dao.getById(0);
        student.setName("被透了");
        student.setAge(13);
        System.out.println(dao.update(student));
    }
    @Test
    public void test_5() throws Exception
    {
        StudentDao dao = new StudentDao();
        System.out.println(dao.delete(9));
    }
    @Test
    public void test_6() throws Exception
    {
        StudentDao dao = new StudentDao();
        List<Student> list = dao.getAll();
        for (Student student : list) {
            System.out.println(student);
        }
    }
}
```

