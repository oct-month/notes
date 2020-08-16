# CSS 简介

CSS 的主要使用场景就是美化网页，布局页面的。

## 1、HTML 的局限性

HTML 非常单纯，它只关注内容的语义。

很早的时候，世界上的网站虽然很多，但是它们都很丑。

虽然 HTML 可以通过标签属性来设置简单的样式，但是这样做会导致代码的复杂和维护的困难。



## 2、CSS - 网页美容师

CSS 是`层叠样式表`（Cascading Style Sheets）的简称。也称为` CSS 样式表`或`级联样式表`。

CSS 是一种标记语言。

CSS 主要用于设置 HTML 页面中的文本内容（字体、大小、对齐方式等）、图片的外形（宽高、边框、边距等）以及版面的布局和外观显示样式。

CSS 可以美化 HTML，让 HTML 更漂亮，让页面布局更简单。

**总结：**

- HTML 主要做结构，显示元素内容。
- CSS 美化 HTML，布局网页。
- CSS 最大价值：HTML 专注于结构呈现，CSS 专注于样式设计。即结构与样式相分离。



## 3、CSS 语法规范

使用 CSS 时，需要遵从一定的规范。

CSS 由两个主要的部分构成：选择器以及一条或多条声明。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
        p {
            color: red;
            font-size: 12px;
        }
    </style>
</head>
<body>
    <p>有点意思</p>
</body>
</html>
```

- 选择器是用于指定 CSS 样式的 HTML 标签，花括号内是对该对象设置的具体样式。
- 属性和属性值以 “键值对” 的形式出现。
- 属性是对指定的对象设置的样式属性，例如字体大小、文本颜色等
- 属性和属性值之间用` : `分开。
- 多个键值对之间用` ; `分开。



## 4、CSS 代码风格

### 1）样式格式书写

**紧凑格式**

```css
h3 {color: deeppink; font-size: 20px;}
```

**展开格式**

```css
h3 {
    color: deeppink;
    font-size: 20px;
}
```

### 2）样式大小写

```css
h3 {
    color: deeppink;
}
```

```css
H3 {
    COLOR: DEEPPINK;
}
```

### 3）空格规范

```css
h3 {
    color: deeppink;
}
```

1. 属性值前面，冒号后面，保留一个空格
2. 选择器（标签）和大括号中间保留空格

