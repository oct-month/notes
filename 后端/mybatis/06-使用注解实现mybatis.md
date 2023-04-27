# 使用注解实现mybatis

1、面向接口编程

​	a) 扩展性好

​	b) 分层开发中，上层不用管具体的实现

​	c) 大家都遵循共同的标准，使得开发变得容易

​	d) 规范性更好。

2、注解的实现

​	a) 编写Dao接口

```java
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;

import java.util.List;

public interface InterDao
{
    @Select("select * from student")
    public List<Student> getList();

    @Insert("insert into student(ID, name, age) values(#{ID}, #{name}, #{age})")
    public int insert(Student student);
}
```

​	b) 在核心配置文件中导入

```xml
<mappers>
	<mapper class="InterDao"/>
</mappers>
```

3、使用

```java
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class ZTest
{
    @Test
    public void test_1() throws Exception
    {
        SqlSession session = MyBatisUtil.getSession();
        InterDao dao = session.getMapper(InterDao.class);
        List<Student> list = dao.getList();
        for (Student student : list) {
            System.out.println(student);
        }
        session.close();
    }
    @Test
    public void test_2() throws Exception
    {
        SqlSession session = MyBatisUtil.getSession();
        InterDao dao = session.getMapper(InterDao.class);
        int result = dao.insert(new Student(4, "老孙tou", 18));
        System.out.println(result);
        session.commit();
        session.close();
    }
}
```

