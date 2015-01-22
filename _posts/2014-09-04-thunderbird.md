---
layout: post
title:  "nsIMsgFolder"
date:   2014-09-04 10:25:49
categories: thunderbird
---

###nsMsgIncomingServer

nsMsgIncomingServer是大多数协议对nsIMsgIncomingServer接口的实现基类，重要方法有：

**CreateRootFolder** 

用来创建Root Folder。即使用serverURI在RDFService中获取相应的RDFResource，这个Resource就是
nsIMsgFolder对象（典型实现是nsMsgDBFolder它同时继承了nsRDFResource和
nsIMsgFolder)，大致实现如下：

	nsIRDFService rdf;
	nsIRDFResource res = rdf.GetResource(serverUri);
	return res.QueryInterface(nsIMsgFolder);
	

**GetServerURI**

 获取ServerURI，返回形如`<localStoreType>://<username>@host`的URI，可以通过这个URI
	获取RootFolder。其中localStoreType是通过GetLocalStoreType获取到的。
	
**GetMsgStore**

 获取nsIMsgPluggableStore接口指针，它是管理Folder的接口指针。
	key为storeContractID的配置包含着XPCOM的contractID，通过获取contractID，
	调用createInstance，即完成创建nsIMsgPluggableStore的任务。默认的store是
	nsMsgBrkMBoxStore，如果在配置中无法找到storeContractID，将会使用这个实现。
	
**CreateLocalFolder**

 创建子Folder，它会调用rootFolder的GetChildNamed来获取local folder，如果获取失败时，
	就会使用GetMsgStore返回的nsIMsgPluggableStore来创建子Folder。

**GetLocalPath**

 获取Server的Root文件路径
	默认实现是使用ProtocolInfo的path，然后创建path类似文件夹`<path>/<hostname>/`
	
###nsIMsgFolder 
	
nsIMsgFolder是文件夹的抽象接口

**AddSubFolder**

 插入指定name的Folder，并返回nsIMsgFolder指针。
	
**GetChildNamed**

根据名字获取子Folder。默认实现是从childFolders容器中返回相应的子Folder。
	
###nsMsgBrkMBoxStore

 默认的nsIMsgPluggableStore的实现，是为了供nsIMsgFolder来管理
子Folder的操作行为，比如创建，删除，复制等操作。

###nsMsgMailSession
 包含了一些类似AddFolderListener的方法，可以为需要监听文件夹变化
 的组件提供观察者方法。
 folderPane中的ui变化就是通过监听nsMsgMailSession的文件夹变化
 来更新ui的。
 
 nsMsgAccountManager中也有AddRootFolderLisnter可以用来监听root folder
 变化的组件来注册监听接口。
 
 nsMsgMailSession提供了注册监听者的接口，那么通知又是怎么发布的呢？
 打开nsMsgDBFolder，我们发现里面有类似如下代码段：
 
	nsCOMPtr<nsIFolderListener> folderListenerManager =
           do_GetService(NS_MSGMAILSESSION_CONTRACTID, &rv);
	return folderListenerManager->OnItemAdded(this, aItem);
	
这就是要求nsMsgMailSession通知其他的监听者Item Add。
也就是说在nsIMsgFolder中包含了发布文件变化的动作。

当删除账号时，将会调用nsAccountManager::RemoteIncomingServer，在这个
函数中包含着通知RootFolder和下级目录的删除通知发布动作。

	for (uint32_t i = 0; i < cnt; i++)
	{
	nsCOMPtr<nsIMsgFolder> folder = do_QueryElementAt(allDescendants, i);
	if (folder)
	{
	  folder->ForceDBClosed();
	  if (notifier)
		notifier->NotifyFolderDeleted(folder);
	  if (mailSession)
	  {
		nsCOMPtr<nsIMsgFolder> parentFolder;
		folder->GetParent(getter_AddRefs(parentFolder));
		mailSession->OnItemRemoved(parentFolder, folder);
	  }
	}
  }
  if (notifier)
	notifier->NotifyFolderDeleted(rootFolder);
  if (mailSession)
	mailSession->OnItemRemoved(nullptr, rootFolder);

当增加账号时，系统会调用nsAccountManager::CreateAccount和CreateIncomingServer来
创建新的账号和server，但是没有发出文件变化的通知啊？原来秘密在
nsMsgAccount::SetIncomingServer中(一般通过脚本account.incomingServer来设置，incomingServer
是属性，在cpp中为SetIncomingServer方法)，下面是核心代码。

	rv = aIncomingServer->GetRootFolder(getter_AddRefs(rootFolder));
    nsCOMPtr<nsIFolderListener> mailSession =
             do_GetService(NS_MSGMAILSESSION_CONTRACTID, &rv);
    mailSession->OnItemAdded(nullptr, rootFolder);
    nsCOMPtr<nsIMsgFolderNotificationService> notifier(do_GetService(NS_MSGNOTIFICATIONSERVICE_CONTRACTID, &rv));
    notifier->NotifyFolderAdded(rootFolder);
	
当设置incomingServer时，mailSession就会发布Item Add通知，folderNotificationService就会发布
Folder Add通知。当ui监听到这些变化时，就会相应更新UI元素。

###nsIMsgFolderNotificationService
nsIMsgFolderNotificationService也包含了folder通知接口，对于需要监听
文件变化的观察者可以注册观察者接口。

###nsObserverService
nsObserverService提供了观察者服务，他可以发布任意的通知，需要接收通知的
组件可以注册到observerService中。
