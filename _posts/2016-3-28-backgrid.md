---
layout: post
title:  "Package Structure"
date:   2016-3-28 18:59:49
categories: css
---

## 简介

鉴于对已有的New Grid的蹩脚设计，我花了两天时间整理了一份可用的grid - `Backgrid`.

[`Backgrid`](http://backgridjs.com/)是一个用于可重用的和可定制化的grid组件。它具有如下核心概念:

* Column: 从来配置每一列的元数据
* Cell: Grid中的最小单元，用于定义每一个显示单元的UI组件
* Header: grid的header组件
* Row: 将Cell组装成一行的UI组件
* Body: Grid的框架UI
* Grid: 最外层UI View.

在原有的`Backgrid`上，我加上支持弹出Edit功能，参见https://github.com/liuxiong332/backgrid。

## Examples

#### 简单实例

通过`Backbone`来创建`Collection`

```js
var Territory = Backbone.Model.extend({});

var Territories = Backbone.Collection.extend({
  model: Territory,
  url: "http://backgridjs.com/examples/territories.json"
});
```

定义`Columns`模型，用于将`Model`的数据映射到对应`Column`的UI上

```js
var columns = [{
  name: "id", // The key of the model attribute
  label: "ID", // The name to display in the header
  editable: false, // By default every cell in a column is editable, but *ID* shouldn't be
  // Defines a cell type, and ID is displayed as an integer without the ',' separating 1000s.
  cell: Backgrid.IntegerCell.extend({
    orderSeparator: ''
  })
}, {
  name: "name",
  label: "Name",
  // The cell type can be a reference of a Backgrid.Cell subclass, any Backgrid.Cell subclass instances like *id* above, or a string

  // This is converted to "StringCell" and a corresponding class in the Backgrid package namespace is looked up
  cell: Backgrid.StringCell.extend({
    editor: Backgrid.PopCellEditor // Use the popup editor otherwise the in place editor.
  })
}, {
  name: "pop",
  label: "Population",
  cell: "integer" // An integer cell is a number cell that displays humanized integers
}, {
  name: "percentage",
  label: "% of World Population",
  cell: "number" // A cell type for floating point value, defaults to have a precision 2 decimal numbers
}, {
  name: "date",
  label: "Date",
  cell: "date"
}, {
  name: "url",
  label: "URL",
  cell: "uri" // Renders the value in an HTML anchor element
}];
```

定义`Grid`实例，然后插入已有UI元素即可。

```js
// Initialize a new Grid instance
var grid = new Backgrid.Grid({
  columns: columns,
  collection: territories
});

// Render the grid and attach the root to your HTML document
$("#example-1-result").append(grid.render().el);
```

[查看实例效果](http://liuxiong332.github.io/backgrid/assets/grid.html).

#### 定制带Aggregation rows的grid

因为Bing Ads的grid的头部和尾部都有aggregation行，下面我就利用`Backgrid`的高度组件化和可定制性，来实现这个新feature。

扩展已有的`Body`，加上头部和尾部aggregation rows支持。

```js
// This grid will add some aggregation information rows at the top and bottom of the grid.
 // These aggregation rows will show some aggregation info such as sum or average value.
 var AggregateBody = Backgrid.Body.extend({
   initialize: function(options) {
     Backgrid.Body.prototype.initialize.apply(this, arguments);
     this.createAggregateColumns();
     var topRows = options.topRows.map(this.transformAggregateRow, this);
     var bottomRows = options.bottomRows.map(this.transformAggregateRow, this);
   },

   createAggregateColumns: function() {
     var newColumns = this.columns.map(function(column) {
       return _.extend({}, column.attributes, {editable: false, cell: "string"});
     });
     this.aggregateColumns = new Backgrid.Columns(newColumns);
   },

   transformAggregateRow: function(row) {
     var model = row;
     return new this.row({columns: this.aggregateColumns, model: model});
   },

   render: function() {
     var fragment = document.createDocumentFragment();
     this.topRows.forEach(function(row) {
       fragment.appendChild(row.render().el);
     }, this);

     this.bottomRows.forEach(function(row) {
       fragment.appendChild(row.render().el);
     }, this);

     this.el.appendChild(fragment);

     this.delegateEvents();
     return this;
   }
 });
 ```

 现在我们可以通过如下方法来为grid增加Aggregation rows
 ```js
 // Initialize a new Grid instance
  var grid = new Backgrid.Grid({
  columns: columns,
  collection: territories,
	body: AggregateBody, // The new AggregateBody can contains additional rows.
	topRows: [new Backbone.Model({'name': "Hello World"})],
  bottomRows: [new Backbone.Model({'pop': "133300 population"})]
});
```

[查看实例效果](http://liuxiong332.github.io/backgrid/assets/aggregate-row-grid.html)

#### 包含pageable的grid

首先扩展`Backbone.PageableCollection`(增加了分页功能的collection).

```js
var PageableTerritories = Backbone.PageableCollection.extend({
  model: Territory,
  url: "http://backgridjs.com/examples/territories.json",
  state: { pageSize: 15 },
  mode: "client" // page entirely on the client side
});

var pageableTerritories = new PageableTerritories();
```

最后实例化`Paginator`UI组件，并插入到已有元素中。
```js
// Initialize the paginator
var paginator = new Backgrid.Extension.Paginator({
  collection: pageableTerritories
});
$("#example-1-result").append(pageableGrid.render().el);
```

[查看实例效果](http://liuxiong332.github.io/backgrid/assets/pageable-grid.html)

**所有实例均可从https://github.com/liuxiong332/backgrid/tree/master/examples看到.**
