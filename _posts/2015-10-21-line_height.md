---
layout: post
title:  "line height"
date:   2015-10-21 20:25:49
categories: css
---

`line-height` 属性设置行间的距离（行高）。
可以有的值为"normal", <number>, <length>, <percentage>

例如：
```css
/* Keyword values */
line-height: normal;

/* Unitless: use this number multiplied by
the element's font size */
line-height: 3.5;

/* <length> values */
line-height: 3em;

/* <percentage> values */
line-height: 34%;

/* Global values */
line-height: inherit;
line-height: initial;
line-height: unset;
```

请注意，一般使用<number>来设置line-height值，使用em或者%会有一些微妙的细节需要注意，大多数不是你想要的行为，
如下示例代码.

```html
<style media="screen">
.green {
  line-height: 1.1;
  border: solid limegreen;
}
.red {
  line-height: 1.1em;
  border: solid red;
}
h1 {
  font-size: 30px;
}
.box {
  width: 18em;
  display: inline-block;
  vertical-align: top;
  font-size: 15px;
}
</style>
<div class="box green">
 <h1>Avoid unexpected results by using unitless line-height</h1>
  length and percentage line-heights have poor inheritance behavior ...
</div>

<div class="box red">
 <h1>Avoid unexpected results by using unitless line-height</h1>
  length and percentage line-heights have poor inheritance behavior ...
</div>

<!-- The first <h1> line-height is calculated from its own font-size   (30px × 1.1) = 33px  -->
<!-- The second <h1> line-height results from the red div's font-size  (15px × 1.1) = 16.5px,  probably not what you want -->
```
