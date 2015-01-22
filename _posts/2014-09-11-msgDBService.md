---
layout: post
title:  "GetSubFolders"
date:   2014-09-11 10:25:49
categories: thunderbird
---
 
thunderbird中有两个与数据库相关的重要的类，分别是nsMsgDBService和
nsMsgDatabase。

###nsMsgDataBase
顾名思义，这个就是数据库对象了。它包含了一些数据库有关的操作，如Open
操作就是打开相应的数据库文件。

###nsMsgDBService 
从名字上来看，nsMsgDBService是专门用来对数据库进行动作的Service。它包含了
很多通用操作，可以被其他需要建立和打开数据库的组件使用，相当于
封装了对应的database。

####OpenFolderDB

根据nsIMsgFolder对应的文件路径，从缓存中获取相应的database，若此前已经打开，
则直接返回。否则，则通过对应的contractID获取相应的database，然后调用Open
来打开database。
其中contractID为 `@mozilla.org/nsMsgDatabase/msgDB-<localStoreType>`

####CreateNewDB
创建nsIMsgFolder指定的database，和上面一样，首先根据contractID获取对应的
database，然后调用Open打开对应的数据库即可。 

####OpenMailDBFromFile
从指定的文件路径中打开database，这个和OpenFolderDB相似。