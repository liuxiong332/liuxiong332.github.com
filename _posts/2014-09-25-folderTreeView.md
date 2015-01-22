---
layout: post
title:  "Folder Tree View"
date:   2014-09-25 10:25:49
categories: thunderbird
---
 
##FolderTreeView初始化

thunderbird左边栏中的文件夹列表对应messenger.xul中id为folderTree
的tree节点。folderTree是个自定义Tree，需要指定tree属性，tree属性是个
nsITreeView类型对象。

在thunderbird中，folderTree的tree属性石gFolderTree，它定义在
folderPane.js中。它的初始化流程如下。

首先当DOM加载完毕时会调用OnLoadMessenger。OnLoadMessenger函数位于
msgMail3PaneWindow.js文件中。

	<window onload="OnLoadMessenger()">
	
在OnLoadMessenger中会调用InitPanes，InitPanes则会初始化folderTree。

	function InitPanes()
	{
		gFolderTreeView.load(document.getElementById("folderTree"),
                       "folderTree.json");
	}
	
如上，调用gFolderTreeView的load方法来初始化。而在load方法中有这样的
一个赋值。
	
	load: function ftv_load(aTree, aJSONFile) {
		...  aTree.view = this;
	}
	
通过将gFolderView赋值给folderTree的view属性，就完成了folderTree的
初始化过程。

##FolderTree响应

当增加账号或者增加文件夹时，将需要folderTree进行相应的改变。FolderTree
是通过注册监听接口来实现的。

	load: function() {
	...
	// Add this listener so that we can update the tree when things change
    MailServices.mailSession.AddFolderListener(this, Ci.nsIFolderListener.all);
	}
	
在gFolderTreeView的load中，它通过nsMsgMailSession的注册接口来注册folderListener，
从而当文件夹发生变化时，folderTree能相应的改变。

##新增folder

当新增nsMsgFolder时，将会通过mailSession发布OnItemAdded通知。

	OnItemAdded: function(aParentItem, aItem) {
		// if no parent, this is an account, so let's rebuild.
		if (!aParentItem) {
		  if (!aItem.server.hidden) // ignore hidden server items
			this._rebuild();
		  return;
		}
		this._modes[this._mode].onFolderAdded(
		aParentItem.QueryInterface(Components.interfaces.nsIMsgFolder), aItem);
	},
  
对于folderTree来说，如果aParentItem为null表示folder为root folder，则调用rebuild重建
folder tree。

_rebuild会调用对应mode的generateMap，下面是mode为all的generateMap实现（默认mode就是all）。

	generateMap: function ftv_all_generateMap(ftv) {
        let accounts = gFolderTreeView._sortedAccounts();
        MailUtils.discoverFolders();

        return [new ftvItem(acct.incomingServer.rootFolder)
                for each (acct in accounts)];
    }

首先获得所有nsMsgAccount，然后调用discoverFolders来触发所有account加载自己的folder。
最主要的行为是后面返回的ftvItem数组。folder tree要显示的就是ftvItem数组。