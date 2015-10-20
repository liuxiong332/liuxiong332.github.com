---
layout: post
title:  "list style"
date:   2015-10-20 20:25:49
categories: css
---

text-align: describe how **inline content**(like text) is aligned in parent block element.

## list-style

对于`li`来说，我们可以设置`list-style`来改变list的风格。

* list-style-type

设置list item元素的外观，它可以应用到`<li>`或者`display: list-item`的元素上。可以设置的值有circle，square, none

* list-style-image

设置`list-item`前面的marker的image

* list-style-position

设置list item前面的marker相对于`list item`box的位置，`inside`或者`outside`.

参考[list-item-position example](https://developer.mozilla.org/samples/cssref/list-style/)

* list-style

上面三个的简写

## list item

对于非`ul`或者`ol`，怎么样才能做出list这种效果呢，答案就是`display: list-item`.

`ul`和`ol`的display属性为`display: block`，从而得知支撑list外观的就是`<li>`元素，而`<li>`默认具有`display: list-item`，
（list item元素是块级元素）,如果要将某块元素实现为`list`样式，可以设置其`display: list-item`，然后设置对应的`list-style`就可以改变风格了。

下面是一个例子：

```html
<style media="screen">
  div {
    margin: 0 0 0 20px;
  }
  p {
    display: list-item;
    list-style: circle;
  }
</style>

<div id="id">
  <p>Hello Nimei</p>
  <p>Hello World</p>
</div>
```

`div`元素将会具有`ul`一样的显示效果。
