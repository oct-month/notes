# YAML语法

## 1、基本语法

k: v	表示一对键值对，冒号后面有一个空格。

以空格的缩进来控制层级关系。左对齐的一列数据看做一个层级。

```yaml
server:
	port: 8080
	path: /path
```

属性和值是大小写敏感的。



## 2、值的写法

字面量：普通值

直接写，字符串不用加上引号。

加双引号：不会转义特殊字符，\n表示换行

加单引号：会转义特殊字符，\n表示 \n

```yaml
name: "zhang\nsan"
namee: '\z\f\d\e'
```

对象、Map

```yaml
# friends对象
friends:
	lastName: 张三
	age: 20
```

```yaml
friends: {lastName: 张三, age: 18}
```

数组，List、Set

```yaml
# pets有3个
pets:
 - cat
 - dog
 - pig
```

```yaml
pets: [cat, dog, pig]
```


