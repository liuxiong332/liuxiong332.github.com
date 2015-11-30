---
layout: post
title:  "React"
date:   2015-10-21 20:25:49
categories: css
---

## Web package

为了适应大规模的web app开发，web component应运而生。web component是将每个独立的UI部分作为
一个整体进行封装，从而达到封装和复用。

在大型web应用中，通常需要使用web component达到下列两个目的：

- 复用已有的web component。一个应用只需要实现一个Button，既能节省开发时间，又能实现统一的界面风格。

- 逻辑代码分离。复杂的应用可能由几十个组件构建而成，将不同功能的组件充分解耦是应对复杂性的关键因素。

那么如何规定web package的结构才能达到上述目的呢？

## Npm package

在`Node`的世界里，`npm`是`Node`的模块依赖管理工具。我们每天从`npm`上获取成千上万的开源包来构建我们
的应用。为了适用社区的需要，我们很自然的使用`npm`作为我们的包构建工具。

通过`npm`来配置我们的web package，我们可以使用`package.json`来规定我们的包依赖和其他的信息。我们
可以将每个web component做成一个包，类似于下面这样

```json
{
  "name": "double-list",
  "version": "0.0.1",
  "dependencies": {
    "react": "^0.14.0"
  }
}
```

这样我们就规定了`double-list`这个web组件依赖`react 0.14`。当我们开发完成之后，我们可以发布出去，
这样其他的开发者就可以复用了。

某一天，我们开发一个其他的组件`account-selection`，我们可以通过如下的`package.json`来声明一个
新的web component。

```json
{
  "name": "account-selection",
  "version": "0.0.1",
  "dependencies": {
    "double-list": "^0.0.1"
  }
}
```

就这样很轻松的复用了`double-list 0.0.1`这个组件，并且，即使`double-list`升级到了`1.0`，也不会破坏
我们的`account-selection`，因为我们维护了依赖的版本号。这对于大型应用至关重要。谁都不希望由于一个
组件的升级而导致其他依赖它的组件无法使用。

说了这么多`npm package`的好处，但是`web package`毕竟不像一般的package，主要是`web package`还需要
运行在浏览器上，所以我们还需要将目光关注在`web package`的结构上。

关于`web package`如何组织，下面提供了两种组织方式。第一种方案(stylesheets alone)属于比较传统的，但是局限性较大的，而相对的，
第二种(css module)则比较激进，但是更加优雅。

但是这两个方案都有如下相似点：

- 使用nodejs的模块包含机制`CMD`(比如，`var lodash = require('lodash')`)，这主要是和`node package`兼容。

- 使用`webpack`来打包，从而将`node package`打包成浏览器可使用的code。

## stylesheets alone

这种方案使用传统的`js`和`style`，它们将会分开提供。目录结构类似如下：

```
- dist
- src
- styles
- examples
- test
```

`src`和`test`分别用来放置`js`代码和测试代码，而`styles`则放置`css`文件，`examples`则包含着组件的使用用例和注意事项。

使用此组件的开发者除了需要声明依赖之外，还需要将`css`文件单独包含进web页面中。

关于这个方案的文件组织结构，可以查看[这个Repository](https://github.com/liuxiong332/web-package-start-kit/tree/master)，
一个比较简单但是非常有代表性的实例可以参见[todomvc示例](https://github.com/liuxiong332/web-package-start-kit/tree/todomvc)。

这个实例使用了`backbone`和`jQuery`来构建这个组件，符合现在很多前端项目的实现栈。我想很多现有的代码都可以改写成这种`web package`。

但是，这个方案由如下局限性：

1. CSS文件需要另外考虑。当组件的嵌套层次多的时候，我想大多数人对于将`css`文件放在哪儿可能需要费一番脑筋。也许需要一个`css manager`来管理
这些css的加载问题。

2. CSS的变量都是全局的，这对于`web package`的复用来说是致命的。谁也无法担保有多个`web component`使用同一个`class`。

3. 封装性比较差。你无法阻止用户使用DOM直接操纵你的组件内部，并且`jQuery`鼓励你这么做。

## react css module

好吧，又是`react`。你也许认为我是`react`偏执的脑残粉，但是这里我想说的重点是`css module`。其实将`react`换成`angular 2`也是可以得，这两
者拥有相似的`web component`血统。

### Why css module

关于CSS的缺点，我在上一篇文章中已经提到过，CSS最大的问题就是全局命名空间，这个是阻止web组件化的最大因素。

关于这个问题的解决方案，目前我知道的比较优雅的有两种，分别是`css in js`和`css module`。

`css in js`的简介在[这儿](https://speakerdeck.com/vjeux/react-css-in-js)，vjeux向我们详细解释了为什么需要`css in js`。
它是将样式直接嵌入到js代码中，从而全部放弃`css`，显而易见，这样就简单的避免了CSS的一系列问题。

但是将style全部放入js之后，会导致dom的膨胀，我们可以看看`css in js`之后DOM页面的样子.

![material ui dom](/images/material-ui-dom.png)

这是[material ui](http://www.material-ui.com/#/)组件库的一个页面展开后的DOM截图，从中看出DOM的膨胀非常大，这样会
导致页面的加载速度显著下降。

除了使DOM膨胀外，`inline style`也不支持`css`中的伪类(`:active`)和`media-query`。

诚然这种方案非常理想，但是这些局限性还是深深地困扰着社区的开发者，未来可能会有一个完美的解决方案。但是现在貌似不是非常好的使用时机。

另外一个解决方案就是`css module`。这里有关于`css module`的[简单介绍](http://glenmaddern.com/articles/css-modules)。

下面是一个简单的使用用例：

```css
/* components/submit-button.css */
.common { /* font-sizes, padding, border-radius */ }
.normal { composes: common; /* blue color, light blue background */ }
.error { composes: common; /* red color, light red background */ }
```

```js
/* components/submit-button.js */
import styles from './submit-button.css';

buttonElem.outerHTML = `<button class=${styles.normal}>Submit</button>`
```

通过import css文件来使用css文件中的样式，并且css文件中的变量会进行重新命名来保证全局唯一。这样就保证了css的变量不会
和其他css冲突了。

例如上述css文件可能会翻译为：

```css
.components_submit_button__common__abc5436 { /* font-sizes, padding, border-radius */ }
.components_submit_button__normal__def6547 { /* blue color, light blue background */ }
.components_submit_button__error__1638bcd { /* red color, light red background */ }
```

从而保证了名字的全局唯一性。当js使用import导入才能获得相应的名字，也就是说只有显示指定才能使用对应的样式，杜绝一切不明确的css规则。

它的核心卖点是：

1. 默认的scoped命名空间。再也不用担心命名冲突了，有木有！

2. 单职责原则。每个`css`文件只负责某个模块的样式。以后我们不需要使用全局查找对应css名字来找到影响组件的样式了。

3. CSS依赖性。你需要显示import才能使用css模块，每个组件依赖的css文件清晰可追寻，终于不用再茫茫css文件中找到你需要的样式了，也不用担心在地球的某个css样式
破坏了组件的现有样式。

这里有个`web component`示例[double-list-ui](https://github.com/liuxiong332/double-list-ui)，使用这种方式感觉非常清晰，最重要的是放心，舒心啊。

### css module structure

使用`css module`的文件结构非常简单，无需单独的`styles`文件夹了。因为`css`文件将会被`webpack`或者`browserify`解析。

它的好处就显而易见了：

1. 局部命名空间，避免全局污染（虽然是使用hash重命名的方式实现的）

2. 组件使用者不需要关心`css`。只需要在app build的时候使用`webpack`进行打包即可

3. 可追溯的依赖。可以明确的看出组件依赖哪些`css`样式。

当然它也有如下的缺点：

1. 自定义组件的样式比较麻烦。之前的class是全局的，我们可以通过重写部分样式来自定义UI，但是在`css module`中做不到。目前有两种方法来解决这个问题：

  1. 组件内设置变量参数，允许用户传入`styles`来重新定义组件样式。这种方式对于大型组件来说代价很大，适合样式不太多的component；

  2. 使用`inline css`。没错，它会导致DOM膨胀，但是因为你只会允许用户自定义某些样式，因此在一定范围内是可以接受的。

1. 设置`class`的时候需要多一层间接层，对于`jade`之类的模板来说，就比较繁琐了。

虽然有这么多缺点，但是无疑`css module`是时下一个比较好的解决方案。

更多细节，查看[double-list-ui](https://github.com/liuxiong332/double-list-ui)示例，我相信各位使用了会跟我一样有一种如释重负的感觉！
