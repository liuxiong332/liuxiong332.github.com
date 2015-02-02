---
layout: post
title:  "message display"
date:   2014-09-28 10:25:49
categories: thunderbird
---


##threadTree select

当threadtree被点击时，即表示选中一条消息，会调用threadPane.js
中的ThreadPaneSelectionChanged函数.

	function ThreadPaneSelectionChanged()
	{
		GetThreadTree().view.selectionChanged();
	}

在这个函数中会调用threadTree.view的selectionChanged方法。从前面我们
知道view对应的是nsMsgDBView类对象，因此它会调用
nsMsgDBView::selectionChanged方法。selectionChanged有如下实现。

	nsresult rv = mTreeSelection->GetRangeAt(0, &startRange, &endRange);

    if (startRange >= 0 && startRange == endRange &&
        uint32_t(startRange) < GetSize())
    {
      if (!mRemovingRow)
      {
        if (!mSuppressMsgDisplay)
          LoadMessageByViewIndex(startRange);
        else
          UpdateDisplayMessage(startRange);
      }
    }

它会调用LoadMessageByViewIndex来加载和显示指定索引message。

	NS_IMETHODIMP nsMsgDBView::LoadMessageByViewIndex(nsMsgViewIndex aViewIndex)
	{
	  nsCString uri;
	  nsresult rv = GetURIForViewIndex(aViewIndex, uri);
	  if (!mSuppressMsgDisplay && !m_currentlyDisplayedMsgUri.Equals(uri))
	  {
		nsCOMPtr<nsIMessenger> messenger (do_QueryReferent(mMessengerWeak));
		messenger->OpenURL(uri);

		UpdateDisplayMessage(m_currentlyDisplayedViewIndex);
	  }
	}

它首先会调用GetURIForViewIndex来获取到消息的URI，它实际上调用的是对应msgFolder的
GenerateMessageURI来形成message URI。

	folder->GenerateMessageURI(aMsgKey, aURI);

然后通过messenger调用openURL，messenger是全局的nsIMessenger对象，它在
mailWindow.js中的CreateMailWindowGlobals中初始化。

现在我们就可以将注意力集中在nsMessenger::OpenURL函数中了。

##nsMessenger::OpenURL

下面是OpenURL的代码片段。

	nsMessenger::OpenURL(const nsACString& aURL)
	{
	  nsCOMPtr <nsIMsgMessageService> messageService;
	  nsresult rv = GetMessageServiceFromURI(aURL, getter_AddRefs(messageService));

	  if (NS_SUCCEEDED(rv) && messageService)
	  {
		messageService->DisplayMessage(PromiseFlatCString(aURL).get(), mDocShell, mMsgWindow, nullptr, nullptr, nullptr);
		AddMsgUrlToNavigateHistory(aURL);
		mLastDisplayURI = aURL; // remember the last uri we displayed....
		return NS_OK;
	  }
	}

首先根据URI获取对应的message service，然后调用message service的DisplayMessage来显示指定的message。

那么根据URI获取message service的规则是怎样的呢？我们可以查看nsMsgUtils.cpp中的GetMessageServiceFromURI。

	nsresult GetMessageServiceFromURI(const nsACString& uri, nsIMsgMessageService **aMessageService)
	{
		nsAutoCString contractID;
		rv = GetMessageServiceContractIDForURI(PromiseFlatCString(uri).get(), contractID);
		nsCOMPtr <nsIMsgMessageService> msgService = do_GetService(contractID.get(), &rv);
	}

	nsresult GetMessageServiceContractIDForURI(const char *uri, nsCString &contractID)
	{
		contractID = "@mozilla.org/messenger/messageservice;1?type=";
		contractID += protocol.get();
	}

也就是说它会加载指定协议的message service。在这个message service上进行相关的动作。

##pop3 OpenURL

很多协议的message service会调用FetchMessage来更新指定的Message。

pop3对应的Message service是nsMailboxService，它在FetchMessage会运行如下代码。

	rv = docShell->LoadURI(url, loadInfo, nsIWebNavigation::LOAD_FLAGS_NONE, false);

LoadURI会通过nsDocShell进行一系列很复杂的调用。

	nsDocShell::DoURILoad(nsIURI * aURI, ...)
	{
		NS_NewChannel(getter_AddRefs(channel),...);
		rv = DoChannelLoad(channel, uriLoader, aBypassClassifier);
	}

NS_NewChannel位于nsNetUtil.h中，也即是说系统通过NS_NewChannel来创建
新的nsIChannel对象.

```cpp
nsDocShell::DoChannelLoad(nsIChannel * aChannel, ...)
{
	rv = aURILoader->OpenURI(aChannel, openFlags, this);
}

nsURILoader::OpenURI(nsIChannel *channel, ...)
{
	nsresult rv = OpenChannel(channel);
	channel->AsyncOpen(loader, null);
}

nsURILoader::OpenChannel(nsIChannel* channel, ...)
{
	nsRefPtr<nsDocumentOpenInfo> loader =
		new nsDocumentOpenInfo(aWindowContext, aFlags, this);
	loader->Prepare();
}
```

`nsDocumentOpenInfo`是Channel Content的观察者，当调用`channel->AsyncOpen`时，`nsDocumentOpenInfo::OnDataAvailable`等方法将会被调用。

```cpp
void nsDocumentOpenInfo::OnStartRequest(nsIRequest *request, nsISupports * aCtxt)
{
	rv = DispatchContent(request, aCtxt);
}

void nsDocumentOpenInfo::DispatchContent(nsIRequest *request, nsISupports * aCtxt)
{
	int32_t count = mURILoader->m_listeners.Count();
  nsCOMPtr<nsIURIContentListener> listener;
  for (int32_t i = 0; i < count; i++) {
    listener = do_QueryReferent(mURILoader->m_listeners[i]);
    if (listener) {
      if (TryContentListener(listener, aChannel)) {}
    }
  }
}

bool nsDocumentOpenInfo::TryContentListener(nsIURIContentListener* aListener,  nsIChannel* aChannel)
{
	aListener->CanHandleContent();
  nsresult rv = aListener->DoContent(mContentType.get());
}
```

由上可知，当从channel获取内容时，documentInfo将会从对每个listener调用`TryContentListener`来判断是否能处理此content，而`TryContentListener`则调用`DoContent`来真正处理Content。

对于邮件来说，处理对应Content的是nsMsgWindow类，位于`nsMsgWindow.cpp`中。

```cpp
void nsMsgWindow::DoContent(const char *aContentType, bool aIsContentPreferred)
{
  nsCOMPtr<nsIDocShell> messageWindowDocShell;
  GetMessageWindowDocShell(getter_AddRefs(messageWindowDocShell));
  return ctnListener->DoContent(aContentType, aIsContentPreferred, request);
}
```

loader是nsIURIContentListener类型对象，它在nsDocShell被初始化为
nsDSURIContentListener对象。

	nsresult nsDocumentOpenInfo::Prepare()
	{
		m_contentListener = do_GetInterface(m_originalContext, &rv);
	}

m_contentListener就是nsDSURIContentListener对象。channel调用AsyncOpen
异步打开文件流。其中异步回调对象是nsDSURIContentListener。

当asyncOpen开始时，会调用回调对象的OnStartRequest方法。OnStartRequest
会调用如下代码。

	aListener->DoContent(mContentType.get(), ...,
	m_targetStreamListener );

aListener就是上面说的nsDSURIContentListener对象。

	nsDSURIContentListener::DoContent() {
		rv = mDocShell->CreateContentViewer(aContentType,
			request, aContentHandler);
	}

aContentHandler是nsIStreamListener接口对象，将此对象设置为上述文件流
的真正回调接口，就可以输出数据到rfc822引擎中进行分析，并显示在
相应界面中。

##NS_NewChannel

现在我们来关注创建新的nsIChannel是怎么实现的。

	nsIOService::NewChannelFromURI(nsIURI *aURI, nsIChannel **result)
	{
		return NewChannelFromURIWithProxyFlags(aURI, nullptr, 0, result);
	}

	nsIOService::NewChannelFromURIWithProxyFlags(nsIURI *aURI, ...)
	{
		nsCOMPtr<nsIProtocolHandler> handler;
		rv = GetProtocolHandler(scheme.get(), getter_AddRefs(handler));

		rv = handler->NewChannel(aURI, result);
	}

	nsIOService::GetProtocolHandler(const char* scheme, nsIProtocolHandler** result)
	{
		nsAutoCString contractID(NS_NETWORK_PROTOCOL_CONTRACTID_PREFIX);
        contractID += scheme;
        rv = CallGetService(contractID.get(), result);
	}

其中，NS_NETWORK_PROTOCOL_CONTRACTID_PREFIX为@mozilla.org/network/protocol;1?name=。
也就是说我们需要实现对应的nsProtocolHandler service。

通过这个service我们可以调用NewChannel来新建nsIChannel。

##Mailbox protocol handler

对于pop3这种邮件协议使用的Protocol handler是nsMailBoxService。它使用nsMailboxProtocol
来获取nsIChannel。

	nsMailboxProtocol * protocol = new nsMailboxProtocol(aURI);
	protocol->Initialize();

Initialize中包含了很多其他uri和folder操作，可以查看源码。
