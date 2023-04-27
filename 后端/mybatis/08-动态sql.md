# 动态sql

1、根据不同的查询条件，生成不同的sql语句。

2、map文件

​	mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.sun.test.studentMapper">
    <select id="getStudentByCondition" parameterType="java.util.Map" resultType="cn.sun.test.Student">
        select * from student
        <where>
            <if test="name!=null">
                name like CONCAT('%', #{name}, '%')
            </if>
            <if test="ID!=null">
                and ID=#{ID}
            </if>
            <if test="age!=null">
                and age=#{age}
            </if>
        </where>
    </select>
</mapper>
```

