---
layout: post
title:  "thunderbird XUL"
date:   2015-01-25 10:25:49
categories: thunderbird xul
---


# Context Menu

在需要右键菜单的节点上加上`context`属性，然后加上`menupopup`节点。

```html
<menupopup id="ScrollbarContextMenu" type="contextFrame">
  <menuitem align="center" id="Context-newFolder" label="" command=""/>
  <menuitem align="center" id="Context-renameFolder"/>
  <menuseparator />
  <menuitem align="center" id="Context-setting"/>
</menupopup>

<toolbarbutton context="ScrollbarContextMenu"/>
```
