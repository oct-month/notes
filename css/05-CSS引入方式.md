# CSS 引入方式

按照 CSS 样式书写的位置（或者引入的方式），CSS 样式表可以分为三大类：

1. 行内样式表（行内式）
2. 内部样式表（嵌入式）
3. 外部样式表（链接式）

## 1、内部样式表

内部样式表是写到 html 页面的内部，是将所有的 CSS 代码抽取出来，单独放到一个 \<style\> 标签中。

```html
<style>
    div {
        color: red;
    }
</style>
```

- \<style\> 标签理论上可以放在 HTML 文档的任何地方，但一般会放在文档的 \<head\> 标签中
- 通过此种方式，可以方便控制当前整个页面中的元素样式设置
- 代码结构清晰，但是并没有实现结构与样式完全分离

## 2、行内样式表

行内样式表是在元素标签内部的 style 属性中设定 CSS 样式。适合于修改简单样式。

```html
<div style="color: red; font-size: 12px;"></div>
```

- style 其实就是标签的属性
- 样式写在双引号中间，写法要符合 CSS 规范
- 可以控制当前的标签设置样式

## 3、外部样式表

实际开发都是外部样式表，适合于样式比较多的情况。

样式单独写到 CSS 文件中，之后把 CSS 文件引入到 HTML 页面中使用。

**使用步骤：**

1. 新建一个 .css 的文件，把 CSS 代码放在里面
2. 在 HTML 文件中，用 \<link\> 标签引入这个文件

```css
<link rel="stylesheet" href="CSS文件路径"/>
```

| 属性 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| rel  | 定义当前文档与被链接之间的关系，在这里需要指定为"stylesheet"，表示被链接的文档是一个样式表文件 |
| href | 定义所链接外部样式表文件的URL地址，可以是相对路径、绝对路径或网络地址。 |

## 4、总结

| 样式表     | 优点                     | 缺点         | 使用情况 | 控制范围 |
| ---------- | ------------------------ | ------------ | -------- | -------- |
| 行内样式表 | 书写方便，权重高         | 结构样式混写 | 较少     | 一个标签 |
| 内部样式表 | 部分结构和样式分离       | 没有彻底分离 | 较多     | 一个页面 |
| 外部样式表 | 完全实现结构和样式相分离 | 需要引入     | 最多     | 多个页面 |

