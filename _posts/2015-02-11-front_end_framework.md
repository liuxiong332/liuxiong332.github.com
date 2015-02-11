---
layout: post
title:  "Front-end Framework"
date:   2015-02-11 10:25:49
categories: front-end framework
---


# 前端框架概述

根据框架提供的功能可以分为：

1. **MVC轻量级框架**，代表框架Backbone, Spine
2. **MVC全栈框架**, 代表框架Angular，Ember
3. **UI库**, 严格来说并不是框架，只是提供了一些UI类。代表库有React, Mithril, KnockoutJS, Vue.js

# 框架介绍

### Angular

AngularJS是建立在这样的信念上的：即**声明式编程**应该用于构建用户界面以及编写软件构建，而**指令式编程**非常适合来表示业务逻辑。框架采用并扩展了传统HTML，通过双向的数据绑定来适应动态内容，双向的数据绑定允许模型和视图之间的自动同步。因此，AngularJS使得对DOM的操作不再重要并提升了可测试性。

Angular包含如下特性：

* **数据绑定**

    通过数据绑定来自动同步ViewModel和UI。

* **控制器**

    AngularJS让你将诸如注册回调函数，更新ViewModel等操作放在控制器中，从而使代码更加可读。

* **Directives**

    Directives可以创建可重用的UI控件，它通过声明新的HTML语法来增加新的组件,从而隐藏了复杂的DOM结构和行为。Directives增加了UI的模块化和可维护性。

* **依赖注入**

    依赖注入可以增加组件的可测试性和可替代性。

* **可测试性**

    AngularJS的可测试性让人印象深刻。


### React

  React通过jsx来生成可重用的组件。
