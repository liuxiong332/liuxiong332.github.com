---
layout: post
title:  "Pluggable Store"
date:   2014-09-06 10:25:49
categories: thunderbird
---
 
pluggableStore默认实现有两个，他们的contractID分别是`@mozilla.org/msgstore/berkeleystore;1`,
  `@mozilla.org/msgstore/maildirstore;1`。
通过源码追踪，可以发现Berkeley store对应的类是`nsMsgBrkMBoxStore`.

`DiscoverSubFolders(nsIMsgFolder* aParentFolder, bool aDeep)`查找
aParentFolder对应目录下的文件和目录，如果发现相应文件则实例化nsIMsgFolder，
并加入到aParentFolder中。
aParentFolder是需要发现文件的目录文件夹，aDeep表示是否迭代分析子文件夹。

`AddSubFolders(nsIMsgFolder* parent, nsCOMPtr<nsIFile>& path)`用来分析parent
对应路径下的文件，并将实例化城nsIMsgFolder，并将插入到parent的子文件夹中。

这个方法将枚举parent目录下的所有文件，如果发现合适的文件名，调用parent的方法
nsIMsgFolder::AddSubFolder将
对应的文件nsIFile实例化为nsIMsgFolder，并插入到parent中。

`CreateFolder(nsIMsgFolder* aParent, nsAString& aFolderName, nsIMsgFolder**)`
在parent目录下创建相应的文件，并调用parent.AddSubFolder创建nsIMsgFolder实例，然后
初始化为msf文件。

`GetSummaryFile(nsIMsgFolder* aFolder, nsIFile** summaryFile)`获取aFolder对应的
msf(mail summary file)文件，nsMsgBrkMBoxStore的实现是在aFolder的文件路径后加上.msf
后缀名。