---
layout: post
title:  "thread tree view"
date:   2014-09-25 10:25:49
categories: thunderbird
---
 
##选择Folder

当用户点击FolderTreeView中的一项时，将会刷新对应的threadTree。

	<tree id="folderTree"
		ondblclick="gFolderTreeView.onDoubleClick(event);"
        onselect="FolderPaneSelectionChange();">
		
当用户单击folder时，将会调用commandglue.js中的
FolderPaneSelectionChange函数。

	function FolderPaneSelectionChange() {
		gFolderDisplay.show(folders.length ? folders[0] : null);
	}

也就是说，系统会通过gFolderDisplay调用show方法，来显示指定
的folder。

## tabmail

在messenger.xul中有个tabmail节点，它是一个自定义节点，通过XBL
绑定生成的。它的定义在tabmail.xml中。

在tabmail中定义了很多方法，其中registerTabType表示注册对应的
TabType，用于打开相应的tab。而OpenFirstTab则打开默认的一个tab。
下面是tabmail的OpenFirstTab的代码片段.
	
	let firstTab = {mode: this.defaultTabMode, busy: false,
                    canClose: false, thinking: false, _ext: {}};
    firstTab.mode.tabs.push(firstTab);
	this.tabInfo[0] = this.currentTabInfo = firstTab;
	let tabOpenFirstFunc = firstTab.mode.openFirstTab ||
                           firstTab.mode.tabType.openFirstTab;
    tabOpenFirstFunc.call(firstTab.mode.tabType, firstTab);
    this.setTabTitle(null);
	
它将会用默认的TabType调用OpenFirstTab来打开第一个tab。

对于thunderbird来说，在OnLoadMessenger中有如下代码段。

	tabmail.registerTabType(mailTabType);
    // glodaFacetTab* in glodaFacetTab.js
    tabmail.registerTabType(glodaFacetTabType);
    tabmail.openFirstTab();
	
它将会注册mailTabType，然后调用openFirstTab来打开tab。其中
mailTabType包含了与mail相关的tabtype，位于mailtabs.js文件中。

	let mailTabType = {
		name: 'mail',
		panelId: 'mailContent',
		modes: { 
			folder { ... },
			message: { ... },
			glodaList: { ... }
		}
	};
	
mailTabType包含了很多的tabType，其中有folder，message，glodaList
等mode，它们都作为modes属性的子元素。folder是默认的mode，因此
首先会调用folder mode的OpenFirstTab。

	openFirstTab: function(aTab) {
        this.openTab(aTab, true, new MessagePaneDisplayWidget(), true);
        aTab.firstTab = true;
        aTab.folderDisplay.makeActive();
    },

folder模式的openFirstTab将会调用mailTabType的openTab方法。

	openTab: function(aTab, aIsFirstTab, aMessageDisplay, aFolderPaneVisible) {
		aTab.messageDisplay = aMessageDisplay;
		aTab.folderDisplay = new FolderDisplayWidget(aTab, aTab.messageDisplay);
		aTab.folderDisplay.msgWindow = msgWindow;
		aTab.folderDisplay.tree = document.getElementById("threadTree");
		aTab.folderDisplay.treeBox = aTab.folderDisplay.tree.boxObject.QueryInterface(
									   Components.interfaces.nsITreeBoxObject);
		aTab.folderDisplay.folderPaneVisible = aFolderPaneVisible;
		if (aIsFirstTab) {
		  aTab.folderDisplay.messenger = messenger;
		}
	}

它将会设置folderDisplay和messageDisplay属性。初始化folderDisplay时，会
实例化FolderDisplayWidget，FolderDisplayWidget是主要的显示控制类。

当调用openTab初始化属性后，openFirstTab将会调用folderDisplay的makeActive
来激活。默认的folderDisplay是FolderDisplayWidget实例，因此系统会调用
FolderDisplayWidget.prototype.makeActive来激活文件视图。

## FolderDisplayWidget

FolderDisplayWidget位于文件folderDisplay.js中. 当openFirstTab时，将会
调用makeActive来激活folderDisplay。makeActive将会设置gFolderDisplay用来
管理右方的视图。

从上面知道，当用户单击folder时，将会调用`gFolderDisplay.show`, 这样我们
可以先关注`FolderDisplayWidget.prototype.show`方法。

	show: function(aFolder) {
		if(!aFolder.isServer) {
			this._nonViewFolder = null;
			this.view.open(aFolder);
		}
		this.makeActive();
	}
	
上述代码中的view是DBViewWrapper实例。也就是说，show方法会调用
DBViewWrapper.prototype.open方法打开对应的folder。

	open: function(aFolder) {
		...  
		msgDatabase = this.displayedFolder.msgDatabase;
		...
		this._enterFolder();
	}
	
	_enterFolder: function() {
		this._applyViewChanges();
	}
	
	_applyViewChanges: function() {
		this.dbView = this._createView();
	}
	
	_createView: function() {
		let dbviewContractId = "@mozilla.org/messenger/msgdbview;1?type=";
		dbviewContractId += "threaded";
		dbView = Cc[dbviewContractId]
                   .createInstance(Ci.nsIMsgDBView);
		dbView.init(this.listener.messenger, this.listener.msgWindow,
                this.listener.threadPaneCommandUpdater);
		dbView.open(this._underlyingFolders[0], sortType, sortOrder, viewFlags,
                  outCount);
	}
	
从上面可知，open将会设置dbView。show方法在调用了DBViewWrapper的open
方法之后，会调用makeActive来激活。

	makeActive: function() {
		if (this.view.dbView) {
			this._showThreadPane();
			if (wasInactive) {
				if (this.treeBox) {
					this.treeBox.view = this.view.dbView;
				}
			}
		}
	}

当dbView不为null时，将会设置treeBox.tree属性，这样threadTree就可以显示了。
而dbView在open时已经被设置了。因此这样就可以显示对应folder。

顺便说一句，view.dbView对应着nsMsgThreadedDBView.cpp中的
nsMsgThreadedDBView实例。它实现了nsIMsgDBView方法和nsITreeView，
前者封装了DB的显示控制，后者为threadTree提供数据源。

##nsMsgThreadedDBView 

这个是threadTree的view实现部分。它实现了nsITreeView接口。它的基类
是nsMsgDBView，这个类继承了nsIMsgDBView和nsITreeView，前者用于dbView的
操作，后者用于实现threadTree的view行为。

也就是说从msgThreadedDBView可以获取所有要显示的msgHdr，然后根据这个
msgHdr从database就可以获取到message了。

那么这个dbview是如何被更新的呢？

仔细观察nsMsgDBView，可以发现它还实现了nsIDBChangeListener，这个是
数据改变监听接口，这样我们就知道了dbview是通过这个接口来接收通知，
从而改变UI。

那么dbChangeListener是如何被注册的呢？

	NS_IMETHODIMP nsMsgDBView::Open(nsIMsgFolder *folder, ...) 
	{
		msgDBService->RegisterPendingListener(folder, this);
	}
	
在open方法中，dbview通过nsMsgDBService的RegisterPendingListener将
监听接口注册进去。打开nsMsgDBService的实现，发现当调用它的注册函数时
会调用nsMsgDatabase的注册函数。也就是说，我们就监听了nsMsgDatabase的
数据变化。

	NS_IMETHODIMP nsMsgDatabase::AddNewHdrToDB(nsIMsgDBHdr *newHdr, bool notify)
	{
		if (notify)
		{
		  nsMsgKey threadParent;

		  newHdr->GetThreadParent(&threadParent);
		  NotifyHdrAddedAll(newHdr, threadParent, flags, NULL);
		}
	}

当调用AddNewHdrToDB将新的nsMsgHdr写入数据库时，可以发出HdrAdd通知，这样dbView注册
的通知接口就会收到通知，于是threadTree就知道新的msg可用了！