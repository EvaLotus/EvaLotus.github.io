---
layout: post
title: 「 CSS 」display属性居然有这么多
date: 2016-11-28 17:11:22
tags: CSS
---
思考一个简单的问题，如何用原生js实现元素的显示隐藏，也就是jQuery中show和hide方法。

你肯定觉得特别简单，隐藏的话，直接`el.style.display='none';`

那显示呢？`el.style.display='block'`？
<!-- more -->

display属性是css中最基本的属性。除了常用的`inline`，`inline-block`，`block`，`none`，其实display还有很多属性值。

这个属性决定了元素的显示类型，也指定了元素怎么生成盒模型。

分为外部显示类型和内部显示类型

```css
/* <display-outside> values */
/* 在流式布局中的角色:行内元素还是块级元素 */
display: block;
display: inline;
display: run-in; /* 大多数浏览器都不支持 */


/* <display-inside> values */
/* 元素内部内容的格式化上下文的类型 */
display: flow; // ??
display: ruby;// ??

display: flow-root;
display: table;/* 内部是table布局 */
display: flex;/* 内部是flex布局 */
display: grid;/* 内部是grid布局 */


/* <display-outside> plus <display-inside> values */
display: block flow; // 可以写两个吗？
display: inline table;
display: flex run-in;

/* <display-listitem> values */
/* 将这个元素的外部显示类型变为 block，并将内部显示类型变为多个 list-item inline 盒。*/
display: list-item;
display: list-item block;
display: list-item inline;
display: list-item flow;
display: list-item flow-root;
display: list-item block flow;
display: list-item block flow-root;
display: flow list-item block;

/* <display-internal> values */
display: table-row-group;
display: table-header-group;
display: table-footer-group;
display: table-row;
display: table-cell;
display: table-column-group;
display: table-column;
display: table-caption;
display: ruby-base;
display: ruby-text;
display: ruby-base-container;
display: ruby-text-container;

/* <display-box> values */
/* 是否显示 */
display: contents;
display: none;

/* <display-legacy> values */
display: inline-block;
display: inline-table;
display: inline-flex;
display: inline-grid;

/* Global values */
display: inherit;
display: initial;
display: unset;
```


