---
layout: post
title:  "RDF GetResource"
date:   2014-09-06 10:25:49
categories: thunderbird
---
 

在thunderbird中创建一个账号时，系统会调用`nsAccountManager->createIncomingServer`来创建一个
nsIMsgIncomingServer对象。IMAP协议在创建incomingServer时会调用类似如下代码：

	nsCOMPtr<nsIMsgIncomingServer> server =
           do_CreateInstance(serverContractID.get(), &rv);
    rv = server->GetRootFolder(getter_AddRefs(rootFolder));
 
 也就是说，accountManager会通过contractID来实例化相应的server，然后调用GetRootFolder来初始化rootFolder，
 GetRootFolder的实现片段如下：
	
	nsCOMPtr<nsIRDFService> rdf = do_GetService("@mozilla.org/rdf/rdf-service;1", &rv);
	nsCOMPtr<nsIRDFResource> serverResource;
	rv = rdf->GetResource(serverUri, getter_AddRefs(serverResource));
	m_rootFolder = do_QueryInterface(serverResource, &rv);
	
通过`RDF->GetResource(serverURI)`来获取RootFolder，其中serverURI是形如`<scheme>://<username>@domain.com`
的URI。那么RDF是如何通过URI来获取rootFolder Resource的呢？

答案就在nsRDFService.cpp中的RDFServiceImpl::GetResource里面。
RDF通过获取对应的工厂来创建相应的实例。

	contractID = NS_LITERAL_CSTRING("@mozilla.org/rdf/resource-factory;1?name=") + <scheme>
	factory = do_GetClassObject(contractID.get());
	
因此，如果你想要系统通过RDF来获取到跟你的<scheme>匹配的Resource，你需要实现对应的工厂。工厂类中
要实现相应的创建Resource的实例。而工厂类是XPCOM组件。

##XPCOM怎么注册的

对于插件的XPCOM组件，我们知道是通过manifest进行注册XPCOM组件的，下面是一个外部注册XPCOM组件的manifest文件。

	interfaces mivExchangeMsgIncomingServer.xpt
	component {79d87edc-020e-48d4-8c04-b894edab4bd2} mivExchangeMsgIncomingServer.js
	contract @mozilla.org/messenger/server;1?type=exchange {79d87edc-020e-48d4-8c04-b894edab4bd2}
	
系统会在加载时候加载相应的js文件来初始化XPCOM组件，那么系统XPCOM组件(如上面的IMAP factory组件)是如何被注册的呢？

答案就是系统会把contractID和相对应的XPCOM组件类写在一张表中，然后由系统统一进行初始化，而不用manifest，毕竟
使用manifest的效率太低。

例如mailnews的对应表就在`mailnews/build/nsMailModule.cpp`中。下面的代码段是表中的两条记录。

	{ &kNS_IMAPRESOURCE_CID, false, nullptr, nsImapMailFolderConstructor },
	{ NS_RDF_RESOURCE_FACTORY_CONTRACTID_PREFIX "imap", &kNS_IMAPRESOURCE_CID },
	
这两条记录就是imap Folder factory的contractID和相对应的factory类nsImapMailFolderConstructor的映射关系记录。

##Preference

thunderbird为了方便用户配置程序的某种行为，他们通过一个叫做Preference的方法来进行配置。
要在程序中指定可配置选项，需要使用nsIPreferenceBranch，具体可以通过如下XPCOM组件来引入。

	var prefService = Cc["@mozilla.org/preferences-service;1"].getService(Ci.nsIPrefService);
	var prefBranch = prefService.getBranch(branchName);
	
之后可以通过getIntPref，getCharPref来获取配置选项，通过setIntPref写入，通过prefHasUserValue来
查看选项是否存在。
当通过`set*Pref(prefName)`写入选项时，系统就会在配置数据库中增加一条branchName+prefName的配置选项，
例如`branchName="mail.server."，prefName="user"`，那么对应的配置选项名字就是`mail.server.user`。

用户可以通过菜单工具-选项-配置编辑器来打开配置编辑器，然后输入相应的选项名字，就可以编辑对应的选项值了。
