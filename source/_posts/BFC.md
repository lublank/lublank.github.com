---
title: CSS中的BFC
date: 2017-04-13 19:47:06
tags: CSS
category: 前端
status: publish
summary: 
---

BFC: 全称：Block Formatting Context (块级格式化上下文)
盒模型布局的一种CSS渲染模式。

<!--more-->

产生BFC布局条件：
浮动、绝对定位，inline-blocks, table-cells, table-captions,和overflow的值不为visible的元素。

- float为left，right，inherit，initial
- position为absolute，fixed，inherit，initial
- display为 table-cell，table-caption，inline-block，flex，inline-flex
- overflow为auto，hidden，inherit，initial，overlay，scroll 

想创建一个BFC盒子只需添加上面的任何一个样式即可。

![BFC-图1](http://oxt0lpfdn.bkt.clouddn.com/BFC-1.png)

上图中，我们可以看到，所有属于同一个BFC的盒子都左对齐（左至右的格式），他们的左外边框紧贴着包含块的左边框。在最后一个盒子里我们可以看到尽管那里有一个浮动元素（棕色）在它的左边，另一个元素（绿色）仍然紧贴着包含块的左边框。

**BFC导致外边距折叠**
![BFC-图2](http://oxt0lpfdn.bkt.clouddn.com/BFC-2.png)

理论上两个p元素之间的边距应该是两个外边距之和80px，但实际上是40px
这就是`外边距折叠`
当相邻块元素外边距不一样时，将以最大的外边距为准。
原因：在同一层BFC里的块盒子会发生外边距折叠的问题

**BFC来防止外边距折叠**
![BFC-图3](http://oxt0lpfdn.bkt.clouddn.com/BFC-3.png)

既然是因为在同一层BFC里才导致外边距折叠的，那么新建一个BFC就可以防止外边距折叠了
新建一个新的BFC包住第三个p元素，则与它相邻的块元素就不会外边距折叠了。

**BFC清浮动**
浮动元素的父容器将不会有任何的高度（高度坍塌），它将不会包含已经浮动的子元素。
脱离页面的常规流，为了解决这个问题，通常我们使用清除浮动来解决这问题，最常用的就是使用clearfix的伪类元素。
但是，我们通过添加overflow: hidden，在容器中创建一个新的BFC，也可以达到这个目的。
在新的这个BFC中，这些浮动元素将会回到常规的文档流中。
![BFC-图4](http://oxt0lpfdn.bkt.clouddn.com/BFC-4.png)

**BFC防止文字环绕**
![BFC-图5](http://oxt0lpfdn.bkt.clouddn.com/BFC-5.png)

首先理解为什么文字会环绕？
一个元素浮动时盒子模型是如何工作的？
浮动元素兼并了块元素和内联元素的优点，使得元素不仅可以设置宽度和高度，也可以在水平方向进行排列布局。
脱离标准流进行排列布局，脱离标准流后的元素就不和块元素相处在同一个流不居中，使得浮动元素漂浮在正常块元素上面。但是 浮动元素虽然脱离了正常的文档流，但是依然占据正常文档流的文本空间。于是在其后面写的文本并不会被浮动元素所覆盖而是继续水平排列超出换行。除了浮动元素之外的空间为排列基准，形成了文本环绕的效果。
![BFC-图6](http://oxt0lpfdn.bkt.clouddn.com/BFC-6.png)


**总结：**
BFC特性表现就是，内部的子元素再怎么翻江倒海，怎样布局都不会影响外部的元素，所以，避免margin重叠，清除浮动什么的就能理解了吧。

**优势：**
* 自适应内容由于封闭，更健壮，容错性强。比方说，内部clear:both不会与兄弟float产生矛盾。而纯流体布局，clear:both会让后面内容无法和float元素在一个水平上，产生布局问题。
* 自适应内容自动填满浮动以为区域，无需关心浮动元素宽度，可以整站大规模应用。而纯流体布局，需要大小不确定的margin/padding等值撑开合适间距，无法CSS组件化。


**参考：**
1.	https://www.w3cplus.com/css/understanding-block-formatting-contexts-in-css.html
2.	http://www.zhangxinxu.com/wordpress/2015/02/css-deep-understand-flow-bfc-column-two-auto-layout/comment-page-1/
3.	https://segmentfault.com/q/1010000002645174/a-1020000002646638
4.	http://div.io/topic/834


