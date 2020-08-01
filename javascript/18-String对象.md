# String对象

## 基本包装类型

> 把简单数据类型包装成为了复杂数据类型
>
> JS 有 3 个特殊的引用类型：String、Number、Boolean

```js
var str = 'andy'
console.log(str.length)		// 4
```

```js
// （1）把简单数据类型包装为复杂数据类型
var temp = new String('andy')
// （2）把临时变量的值给str
var str = temp
// （3）销毁这个临时变量
delete temp
```

## 字符串不可变

指的是里面的值不可变，虽然看上去可以改变内容，但其实是地址变了，内存中开辟了一个内存空间。

字符串所有的方法，都不会修改字符串本身，操作完成会返回一个新的字符串。

## 使用

1、查找字符位置

```js
var str = '改革春风吹满地，春天里那个百花开'
// 从字符串的开始往后查找
console.log(str.indexOf('春'))		// 2
// 从索引号为 3 的位置开始往后查找
console.log(str.indexOf('春', 3))	// 8
// 从字符串的后面开始往前查找
console.log(str.lastIndexOf('春'))	// 8
// 从索引号为 3 的位置开始往前查找
console.log(str.lastIndexOf('春', 3))// 2
```

2、通过索引访问字符

```js
var str = '改革春风吹满地，春天里那个百花开a'
// 返回相应索引号的字符
console.log(str.charAt(3))			// '风'
console.log(typeof str.charAt(3))	// string
// 返回相应索引号的字符的ASCII码
console.log(str.charCodeAt(16))		// 97
// 直接通过下标访问（H5 新增）
console.log(str[3])					// '风'
```

3、字符串的操作

```js
var str = 'andyandy'
// 1、拼接
console.log(str.concat('red'))		// andyandyred
console.log(str)					// andyandy
str += 'red'
console.log(str)					// andyandyred
// 2、截取
console.log(str.substr(2, 3))		// dya
console.log(str)					// andyandyred
console.log(str.slice(2, 5))		// dya
console.log(str.substring(2, 5))	// dya
// 3、替换
console.log(str.replace('a', 'b'))	// bndyandyred
console.log(str)					// andyandyred
```

4、字符串转换为数组

```js
var str = ',andy,jquery,mono,'
console.log(str.split(','))			// ['', 'andy', 'jquery', 'mono', '']
```

5、转换大小写

```js
var str = 'andyandyD'
console.log(str.toUpperCase())		// ANDYANDYD
var str2 = 'ANFEjefFe'
console.log(str.toLowerCase())		// andyandyd
```

