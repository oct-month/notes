# CSS 字体属性

CSS 字体属性用于定义`字体系列`、`大小`、`粗细`和`文字样式`。

## 1、字体系列

CSS 使用 ==font-family== 属性定义文本的字体系列。

```css
p {
    font-family: "微软雅黑";
}
div {
    font-family: Arial, "Microsoft Yahei", "微软雅黑";
}
```

- 各种字体之间必须用逗号隔开，优先用第一个
- 如果是多个单词组成的字体，加引号
- 尽量使用系统默认自带字体

## 2、字体大小

CSS 使用 ==font-size== 定义字体大小。

```css
p {
    font-size: 20px;
}
```

- px（像素）大小是网页的常用单位
- chrome 默认的字体大小为 16px
- 尽量给一个明确大小
- 可以给 body 指定整个页面的字体大小
- h（标题）标签默认字体会偏大，需要单独指定

## 3、字体粗细

CSS 使用 ==font-weight== 属性设置文本字体的粗细。

```css
p {
    font-weight: bold;
}
```

```css
p {
    font-weight: 700;
}
```

| 属性值  | 描述                             |
| ------- | -------------------------------- |
| normal  | 不加粗                           |
| bold    | 加粗                             |
| 100~900 | 400等同于 normal，700等同于 bold |

## 4、文字样式

CSS 使用 ==font-style== 属性设置文本的风格。

```css
p {
    font-style: normal;
}
```

| 属性值 | 作用                   |
| ------ | ---------------------- |
| normal | 默认值，标准的字体样式 |
| italic | 斜体                   |

## 5、复合属性

复合属性 ==font== 可以把以上文字样式合起来写。

```css
div {
    /* font: font-style font-weight font-size/line-height font-family */
    font: italic 700 16px/20px 'Microsoft Yahei';
}
```

- 使用 font 属性时，必须按上面的顺序写，不能交换顺序，并且各个属性间以空格隔开
- 不需要设置的属性可以省略，但是必须保留 font-size 和 font-family 属性，否则 font 属性不起作用

