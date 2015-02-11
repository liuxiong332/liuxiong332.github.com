---
layout: post
title:  "WPS投票协议"
date:   2015-01-12 10:25:49
categories: 协议文档
---


# 概述

后台服务器和客户端邮箱都通过类似于下面的html邮件来进行通信。

```html
<body>
<action type="vote" stage="start" id="1234">
  <p class="action-subject">去哪儿吃饭</p>
  <p class="action-content">吃饭好啊</p>
  <ul>
    <li class="action-option">地点一 </li>
    <li class="action-option">地点二</li>
    <li class="action-option">地点三</li>
  </ul>

  <p class="action-promotor">liuxiong@kingsoft.com</p>
  <ul >
    <li class="action-receiver">liuxiong@kingsoft.com</li>
    <li class="action-receiver">gonghao@kingsoft.com</li>
  </ul>
</action>
</body>
```

其中，自定义参数信息放入**body>action** 自定义标签中。**action**作为body的一部分能方便客户端进行分析，而且服务器也不会将其过滤，因为action中的子标签是可以显示的。

action的属性分别代表如下内容：

* **type**：表示要使用的服务，例如vote表示发起投票服务。
* **stage**：表示服务的不同动作，例如start表示发起一个投票邀请。
* **id**：动作发起方生成的id，用于区别不同的投票请求。

每个投票交互动作都有相应的stage属性，通过stage属性可以区分投票的不同动作，具体stage如下图所示：
![WPS VOTE PROTOCOL](/assets/images/wps_vote_protocol.png)


图中的数字表示一次投票需要经历的序列过程。即上述的stage属性。

* **start**：发起方发起投票请求
* **end**: 发起方要求停止投票(图中未标识)
* **promotor_start**: 服务器响应发起方，发起投票成功
* **receiver_vote**: 服务器要求客户端进行投票
* **receiver_retry**: 客户端投票无效，服务器要求重新投票
* **receiver_complete**: 服务器通知客户端投票成功
* **vote/cancel**: 投票方可以选择投票或者取消，对于非bolt通过发送邮件来模拟
* **promotor_fresh**: 当客户端投票时，服务器通知发起方刷新
* **promotor_end**: 当投票时间已到，或者发起人要求停止投票，或者所有人都已投票完成之后，服务器结束投票
* **publish**：发起方要求发布投票详情
* **receiver_publish**：服务器将投票详情发送给客户端。

# 请求参数
请求参数位于`action`的子节点中，每个参数都可以通过参数对应的`class`属性来获取。例如，当发起`start`请求时，我们需要投票的主题、内容、选项、接收人，那么我们就可以将主题对应的节点用`class="action-subject"`来修饰，客户端可以通过`css`来获取对应的`subject`。

例如，对于上述的start请求html，"去哪儿吃饭"被修饰为`class=action-subject`，而`"action-subject"`表示主题，因此"去哪儿吃饭"就是投票的主题。
同理，"吃饭好啊"就是投票的内容，"地点一，地点二，地点三"则是投票的选项，`liuxiong@kingsoft.com`和`gonghao@kingsoft.com`是投票的接收方。

# API参考
html格式为

```html
<body>
   <action type="vote" stage="<stage_action>" id="<id>">
  </action>
</body>
```

action的子节点参数定义如下：

* **action-subject**: 投票主题
* **action-content**: 投票内容
* **action-promotor**: 投票发起方
* **action-receiver**: 投票接收方
* **action-option**：投票选项
* **vote-option**: 投票人的投票id

对于每个stage可以使用的参数如下（每个描述的第一行为stage，第二行开始为对应stage可以使用的参数）

1.  **start/receiver_vote**

    *action-subject, action-content, action-option, action-promotor, action-receiver*

2.  **vote**

    *vote-option*

3.  **Promotor_fresh/promotor_end/receiver_publish**

    *action-option, action-receiver*

    其中对于每个action-option的投票人都需要使用action-receiver来作为子节点，形如

    ```html
    <div class="action-option" data-value="地点一">
      <div class="action-receiver">liuxiong@kingsoft.com</div>
      <div class="action-receiver">gonghao@kingsoft.com</div>
    </div>
    ```

    `data-value`是选项的内容，它的形式跟`<div class="action-option">地点一</div>`效果一样。
