---
layout: post
title:  "get and set css property"
date:   2015-03-7 20:25:49
categories: css jQuery
---

下面介绍两种获取和设置css属性的方法。

# JavaScript

### 改变局部样式

  * 通过改变class属性，可以改变样式，但是事先要把相应类的样式写入外部样式表中，这种方法适用于整体改变样式风格的元素。

    ```javascript
    document.getElementById('content').className = "newClass"
    ```

  * 通过改变`style`属性，来动态改变某些样式属性，比如，要动态改变元素的宽度，则只能修改`css`的`width`样式，通过设置`className`是行不通的。

    ```javascript  
    document.getElementById('content').style.width = "23"
    ```

### 全局改变样式

  可以改变外部链接样式表的href来动态切换样式。

  例如下列的样式表:

  ```html
  <link rel="stylesheet" href="firefox.css" id="css">
  ```

  可以通过相应的js来切换整体的样式表。

  ```javascript
  document.getElementById('css').href = "fs.css"
  ```

### 获取样式

  JavaScript可以调用style来获取内联在元素style属性中的样式，但是不能获取计算后的样式。计算后的样式需要用`document.getComputedStyle`来获取。

  ```javascript
  var element = document.querySelector('.class');
  var width = window.getComputedStyle(element).width;
  ```

# jQuery

jQuery通过css方法来设置和获取样式信息。

* `css(propertyName)`

    获取计算后的样式，和调用`window.getComputedStyle()`调用类似。*记住*: 返回的样式值是字符串形式，需要自己进行转化。

    propertyName支持CSS和DOM格式的样式属性，例如`css('background-color')`和`css('backgroundColor')`都是可用的。

* `css(propertyName, value)` or `css(propertyValueObject)`

    设置css样式，通过元素的style属性来设置局部属性。

    *特别注意*: value应该是字符串，如果是整数，那么jQuery会默认为像素值，将会转化为类似`<n>px`的字符串。

    如果要取消设置的style样式，可以通过`css(propertyName, '')`，也就是设置空字符串来删除style中的样式。
