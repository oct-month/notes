# 生成 HTML

## 1、生成标签

直接输入标签名，然后按 tab 。

比如 div 然后 tab 键，生成 \<div\>\</div\> ：

```html
<div></div>
```

## 2、生成多个相同标签

标签名加上 * 就可以了。

比如 div*3 就可以快速生成 3 个 div ：

```html
<div></div>
<div></div>
<div></div>
```

## 3、生成父子标签

如果有父子关系的标签，可以用 > 。

比如 ul>li ，可以生成：

```html
<ul>
    <li></li>
</ul>
```

## 4、生成兄弟标签

如果是有兄弟关系的标签，用 + 就可以了。

比如 div+p ：

```html
<div></div>
<p></p>
```

## 5、生成带有类名的标签

使用 标签.类名。

比如 p.nav：

```html
<p class="nav"></p>
```

## 6、生成带有 ID 的标签

使用 标签#ID。

比如 p#banner：

```html
<p id="banner"></p>
```

## 7、生成有顺序的类名

使用 $ 号。

比如 div.demo$*5 ：

```html
<div class="demo1"></div>
<div class="demo2"></div>
<div class="demo3"></div>
<div class="demo4"></div>
<div class="demo5"></div>
```

## 8、生成有内容的标签

使用 {} 。

比如 div{ghs} ：

```html
<div>ghs</div>
```

div{$}*5

```html
<div>1</div>
<div>2</div>
<div>3</div>
<div>4</div>
<div>5</div>
```

## 9、混合使用

ul.gray>li{ghs}#banner$*5

```html
<ul class="gray">
    <li id="banner1">ghs</li>
    <li id="banner2">ghs</li>
    <li id="banner3">ghs</li>
    <li id="banner4">ghs</li>
    <li id="banner5">ghs</li>
</ul>
```

