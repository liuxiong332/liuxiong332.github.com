---
layout: post
title:  "Javascript测试框架Jasmine"
date:   2014-05-02 10:25:49
categories: Javascript
---

Jasmine是一个常用的JS测试框架，可用来进行BDD(行为驱动测试)单元测试，它的常见使用
方式如下：

```js
describe("A suite", function() {
  it("contains spec with an expectation", function() {
    expect(true).toBe(true);
  });
});
```

###测试结构

describe表示一个测试Suite，他是用来组织各种测试用例，将测试用例进行分组。
it则表示一个测试实例，不同的测试实例是不相关的。他们都有两个参数，第一个参数表示
测试标识字符串，用来生成测试报告，第二个参数是测试函数。

###断言(Expect)

Jasmine是用Expect来判断测试的结果是否与预期相符。类似于下面的代码片段：

```js
it("and has a positive case", function() {
  expect(true).toBe(true);
});
```

Expect表达式后面需要链接Matcher函数，用来进行匹配动作。toBe表示精确匹配，相当于
使用`===`来进行比较。Jasmine提供了很多Matcher，例如下面列出来的常用的Matcher。

* toEqual相当于使用`==`来进行比较；

* toMatch是使用正则表达式匹配规则来比较；

还有很多Matcher，具体可以参考Jasmine文档。

## Setup和Teardown

有很多测试用例需要使用一些公用代码，例如在测试之前需要设置测试环境，测试之后销毁
测试环境，Jasmine为这些初始化和销毁提供了beforeEach和afterEach函数。beforeEach是
在每个Spec之前运行，afterEach实在每个Spec之后运行。例如下面的实例。

```js
describe("A spec (with setup and tear-down)", function() {
  var foo;

  beforeEach(function() {
    foo = 0;
    foo += 1;
  });

  afterEach(function() {
    foo = 0;
  });

  it("is just a function, so it can contain any code", function() {
    expect(foo).toEqual(1);
  });

  it("can have more than one expectation", function() {
    expect(foo).toEqual(1);
    expect(true).toEqual(true);
  });
});
```

在每个Spec运行之前都会运行beforeEach设置foo变量，在每个Spec运行完之后都会运行
afterEach来重新设置foo变量。

##Spy

在我们的测试一个功能时，为了防止其他模块干扰，常常需要将其他模块进行Mock，以关注
单一的测试点。Jasmine将Mock称作Spy。例如有A,B两个子模块，A依赖于B，现在要测试A模
快，为了防止B模块干扰A模块，我们可以将B进行Spy，从而专注于A模块的测试。
