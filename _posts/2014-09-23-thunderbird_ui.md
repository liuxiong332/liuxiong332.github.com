---
layout: post
title:  "GetSubFolders"
date:   2014-09-11 10:25:49
categories: thunderbird
---
 
##账号设置

账号设置用来增加、删除、管理用户账号。
它的chrome地址为`chrome://messenger/content/AccountManager.xul`。

左下角有个账号操作按钮，id为`accountActionsButton`，点击
之后会弹出几个菜单项。 

###增加账号
增加邮件菜单将会调用AccountManager.js
中的AddMailAccount函数。它的调用流程如下.
	AddMailAccount -> NewMailAccount -> msgNewMailAccount，
msgNewMailAccount会打开emailWizard.xul对话框来设置新账号。

##账号设置向导对话框

账号设置对话框的chrome地址为
`chrome://messenger/content/accountcreation/emailWizard.xul`。

当用户填写账号信息并且验证可用后，用户可以点击完成来进行真正的账号
增加操作。完成按钮的id为create_button，它会调用
`gEmailConfigWizard.onCreate()`来创建账号。最终系统会调用
createInBackend.js中的createAccountInBackend来创建真正的账号。