# Array对象

JS 的 `Array` 对象是用于构造数组的全局对象，数组是类似于列表的高阶对象。

1、创建数组

```js
var arr = [1, 2, 3]			// 利用字面量创建数组
var arr1 = new Array()		// 利用Array对象创建一个空数组
var arr2 = new Array(3)		// 创建一个长度为3的空数组
var arr3 = new Array(2, 3)	// 等价于 [2, 3]
```

2、通过索引访问数组元素

```js
var fruits = ['aa', 'bb', 'cc']
var first = fruits[0]
var last = fruits[fruits.length - 1]
```

3、找到某个元素在数组中的位置

```js
var arr = ['red', 'pull', 1, 2, 3, 1, 'pull', 11]
console.log(arr.indexOf('pull'))	// 1	（从开头找，找到了，返回索引）
console.log(arr.indexOf(100))		// -1	（没找到，返回 -1）
console.log(arr.lastIndexOf('pull'))// 6	（从末尾找，找到了，返回索引）
console.log(arr.lastIndexOf(100))	// -1	（没找到，返回 -1）
```

4、判断是否为数组

```js
var arr = []
var obj = {}
// 1、instanceof运算符
console.log(arr instanceof Array)	// true
console.log(obj instanceof Array)	// false
// 2、Array.isArray函数（H5新增）
console.log(Array.isArray(arr))
console.log(Array.isArray(obj))
```

5、遍历数组

```js
var fruits = ['aa', 'bb', 'cc']
fruits.forEach(function (item, index, array) {	// 回调函数
    console.log(item, index);
})
```

6、添加数组元素

```js
var arr = [1, 2, 3]
// 1、在末尾添加
arr.push(4)
console.log(arr.push(5, 11))	// 6	（push的返回值是新数组的长度）
console.log(arr)				// [1, 2, 3, 4, 5, 11]
// 2、在前面添加
console.log(arr.unshift('red', 'pull'))	// 8	（unshift的返回值是新数组的长度）
console.log(arr)				//  ['red', 'pull', 1, 2, 3, 4, 5, 11]
```

6、删除数组元素

```js
var arr = ['red', 'pull', 1, 2, 3, 4, 5, 11]
// 1、删除数组的最后一个元素
console.log(arr.pop())	// 11	（返回值是删除的元素）
console.log(arr)		// ['red', 'pull', 1, 2, 3, 4, 5]
// 2、删除数组开头的一个元素
console.log(arr.shift())	// 'red'	（返回值是删除的元素）
console.log(arr)			// ['pull', 1, 2, 3, 4, 5]
// 3、删除数组指定索引的元素
console.log(arr.splice(1, 1))	// [1]	（删除索引1开始的1个元素，返回删除的元素组成的数组）
console.log(arr);			// ['pull', 2, 3, 4, 5]
console.log(arr.splice(1, 2))	// [2, 3]	（删除索引1开始的2个元素，返回删除的元素组成的数组）
console.log(arr)			// ['pull', 4, 5]
console.log(arr.splice(1))	// [4, 5]	（删除索引1开始的所有元素，返回删除的元素组成的数组）
console.log(arr)			// ['pull']
```

7、复制数组

```js
var arr = ['red', 'pull', 1, 2, 3, 4, 5, 11]
var arr2 = arr.slice()
console.log(arr2)		// ['red', 'pull', 1, 2, 3, 4, 5, 11]
```

8、翻转数组

```js
var arr = ['red', 'pull', 1, 2, 3, 4, 5, 11]
console.log(arr.reverse())	// [11, 5, 4, 3, 2, 1, 'pull', 'red']
console.log(arr)			// [11, 5, 4, 3, 2, 1, 'pull', 'red']
```

9、数组排序

```js
var arr = ['red', 'pull', 1, 2, 3, 4, 5, 11]
console.log(arr.sort())		// [1, 11, 2, 3, 4, 5, 'pull', 'red']
console.log(arr)			// [1, 11, 2, 3, 4, 5, 'pull', 'red']
// 1、升序
arr.sort(function(a, b) {
    return a - b
})
console.log(arr)			// [1, 2, 3, 4, 5, 11, 'pull', 'red']
// 2、降序
arr.sort(function(a, b) {
    return b - a
})
console.log(arr)			// [11, 5, 4, 3, 2, 1, 'pull', 'red']
```

10、数组转换为字符串

```js
var arr = ['red', 'pull', 1, 2, 3, 1, 'pull', 11]
// 1、toSring方法
console.log(arr.toString())		// red,pull,1,2,3,1,pull,11
// 2、join方法
console.log(arr.join(''))		// redpull1231pull11
console.log(arr.join('-'))		// red-pull-1-2-3-1-pull-11
console.log(arr.join('&'))		// red&pull&1&2&3&1&pull&11
```

11、数组截取

```js
var arr = ['red', 'pull', 1, 2, 3, 1, 'pull', 11]
// 截取arr数组的 [4, 7) 部分的元素并返回，原数组不发生变化
console.log(arr.slice(4, 7))	// [3, 1, 'pull']
console.log(arr)				// ['red', 'pull', 1, 2, 3, 1, 'pull', 11]
```

12、合并数组

```js
var array1 = ['a', 'b', 'c']
var array2 = ['d', 'e', 'f']
var array3 = array1.concat(array2)
// concat方法连接数组，返回一个新数组，原数组不发生变化
console.log(array1)     // ['a', 'b', 'c']
console.log(array2)     // ['d', 'e', 'f']
console.log(array3)     // ['a', 'b', 'c', 'd', 'e', 'f']
```

