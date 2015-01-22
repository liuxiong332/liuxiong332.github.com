---
layout: post
title:  "GetSubFolders"
date:   2014-09-11 10:25:49
categories: thunderbird
---
 
当用户增加账号时，就会调用SetIncomingServer来设置server，设置server时，
他会调用如下代码：

	rv = aIncomingServer->GetRootFolder(getter_AddRefs(rootFolder));
    rv = rootFolder->GetSubFolders(getter_AddRefs(enumerator));
	mailSession->OnItemAdded(rootFolder, msgFolder);
	notifier->NotifyFolderAdded(msgFolder);
	
上述代码首先获得root folder，然后通过root folder来获取子folder，并通过
mailSession(nsMsgMailSession)和nofitier(nsFolderNotificationServer)来通知
其他观察者文件夹的变化。当UI收到通知就会刷新界面，这样我们就可以在folderPane
看到界面变化了。

pop3是如何实现GetSubFolders呢？
pop3的Folder是nsLocalMailFolder,他的实现大致如下。
	若没有初始化 -> msgPluggableStore.discoverSubFolders(从已有的目录中
	建立子folder) -> 如果是server文件夹 -> 创建默认的Mail Folder -> 
	设置默认的文件Flag.