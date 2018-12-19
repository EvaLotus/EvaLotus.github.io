---
layout: post
title: 「 CSS 」css布局规则和你不知道的BFC
date: 2017-04-18 10:11:15
tags: CSS
---
BFC的文章很多，但是大多翻译腔读起来佶屈聱牙，而且有些例子根本不符合场景。今天从一些常见基本问题入手，来全面的分析css布局的规则。大部分内容来自MDN及自己的理解。

<!-- more -->
### 0.什么是盒模型？

一个元素如何在页面中找准自己的位置，首先要知道定位，然后再根据padding，margin，border及内容来决定尺寸。

一个大盒子里放了小盒子，小盒子的定位首先是根据大盒子来的。外部发生什么只影响大盒子，但不会影响小盒子。

标准盒子模型：宽度=内容的宽度（content）+ border + padding + margin
低版本IE（&lt;6）盒子模型：宽度=内容宽度（content+border+padding）+ margin

box-sizing属性用来确定控制元素盒模型的解析模式，默认是content-box标准盒模型，定义了宽高是否包含border和padding。

\(IE&gt;=8需要加上前缀\)box-sizing:border-box

box-sizing:border-box没有广泛使用的原因？
IE6，7的标准模式下没有box-sizing，要使用box-sizing必须使用quirk mode
有了calc\(\)之后，box-sizing用来避免无意义标签的不可替代性消失了，而且还有些bug

比如FF和IE在getComputedStyle得到的width/height是按照标准模式计算出来的），以及混合多种box model在开发中的理解成本（要随时记得width/height的表现取决于box-sizing的计算值——其实left/top也有类似的问题，调试的时候你得确定position的计算值），使用box-sizing的好处就不多了。
参考：[https://www.zhihu.com/question/20691294](https://www.zhihu.com/question/20691294)

房子是一砖一瓦搭起来的，这一个个砖块就是一个个盒子。所以说我们就是搬砖工？

### 1.盒（box）和元素（element）有什么关系？

看MDN上有时候提到块级盒，有时候又称为块级元素，本来简单的以为这两者是一致的，大多数情况下每个元素都可以被看作是一个盒子。但实际上两者并不是一一对应的关系。

多个元素可以组成一个盒。比较好理解，在css世界中本来就是大盒嵌套小盒。

一个元素也可以生成多个盒。主要的两种情况：

* 比如`li`，一个元素会生成2个盒，前面的项目符号也是一个盒，主盒是`li`中的内容。

* **匿名盒**（anonymous box），`<div>Some inline text <p>followed by a paragraph</p> followed by more inline text.</div>`

`Some inline text`这部分不能被选择器精确的选中，所有的属性都是`inherit`，整个`div`会生成一个盒，`Some inline text`这部分也会生成一个独立的盒，称作匿名盒。所以这种情况下一个元素生成了多个盒。匿名盒所有可继承的 CSS 属性值都为 `inherit` ，而所有不可继承的 CSS 属性值都为 `initial`。

元素只是对于HTML来说的，我们谈到**布局（layout）时，都是在说盒的布局**。可以说盒是css世界的基础，是css世界的最小单位。

大部分情况下盒和元素的关系是对应的，我们说属性时通常都是元素的属性，~~盒的类型由最外层元素的~~`display`~~属性决定~~。

盒主要有块级盒、行内级盒、匿名盒以及一些实验性的盒（未来可能添加到规范中）。我们主要分析块级盒和行内级盒。

### **2.何为行内（inline），何为块（block），各有什么特性？**

#### 块级

* **块级元素 **\(block-level element\)：`display`属性为`block`、`list-item`、`table` 的元素。主要对应的标签有:`div`，`p`，`h1`等。

* **块级盒 **\(block-level box\)：最外层元素是块级元素生成，对内部的子元素没有要求，子元素可以是行内元素。这个概念用来解释布局。【？？？又说内部元素要么都是块级盒，要么都是行内盒？？？】

* **块容器盒**（block containing box）：只是强调当前盒有包含其他的盒。

* **块盒**（block box）：既是块级盒又是块容器盒。

#### 行内

* **行内级元素**（inline-level element）：`display`属性 为 `inline`、`inline-block`、`inline-table` 的元素。

* **行内级盒**（inline-level box）：由行内级元素生成。行内级盒包括行内盒子和原子行内级盒子两种，区别在于该盒子是否参与行内格式化上下文（IFC，inline formatting context）的创建。

* **行内盒**（inline box）：参与行内格式化上下文创建的行内级盒称为行内盒。与块盒类似，行内盒也分为具名行内盒和匿名行内盒（anonymous inline box）两种。

* **原子行内级盒**（atomic inline-level box）：不参与行内格式化上下文创建的行内级盒。原子行内级盒子一开始叫做原子行内盒（atomic inline box），后被修正。原子行内级盒的内容不会拆分成多行显示。

空元素（empty element）：不存在子节点（子元素或元素内的内容）；

行内元素：`a b span img input select strong（强调的语气）`

块级元素有：`div ul ol li dl dt dd h1 h2 h3 h4…p`

常见的空元素： `<br> <hr> <img> <input> <link> <meta>`用的比较少的：`<area> <base> <col> <command> <embed> <keygen> <param> <source> <track> <wbr>`

### **3.页面是如何布局的？**

谈布局时，实际上说的都是盒的定位。

主要有普通流（normal flow）、绝对定位\(absolute\)、浮动布局\(float\)三种情况。布局根据盒的**display**，**postion**和**float**属性来决定。

#### 普通流

元素的默认属性是`position:static,float:none`称为普通流，主要规则是：

> 块级盒 垂直依次排列。行内盒水平依次排列。

postion为static时，每个盒根据普通流所计算出的确切位置来定位。

position为relative时，相对定位，此时每个盒还根据top，bottom，left，right在**原本的位置**基础上进行偏移，也就是说不设置position为非static，四个偏移属性是不生效的。

普通流中的块级盒有一些规则：

* 普通流中块级盒会独占一行，在垂直方向上，一个接一个放置。

* 块级盒**垂直方向**的距离由margin决定，相邻两个块级盒的会发生**垂直方向的margin重叠**，注意是垂直方向，水平方向不会有这个问题。

解决margin重叠的方法在之后的BFC的规则中会解释。

让两个元素不属于一个BFC，层级不一样就可以避免这个问题。但是两个元素是平级的话还是会出现margin重叠，其实解决的方式是遵循了：BFC是独立的，不受到外部的影响。

#### 浮动布局

当position为static或relative，且float不为none时，为浮动定位，生成了**浮动盒**（floated/floating box）。浮动盒在垂直方向上会脱离文档流，水平方向上，还是一个个按顺序排列，有的靠左，有的靠右。

对于块级盒，本来是独占一行，宽度是100%，但是变成浮动盒之后，自身的宽度就开始变成由内部撑起来。

对周围其他盒的影响：

外层盒如果是行内盒，会伸缩来适应内部浮动盒的大小。

相邻盒会环绕在浮动盒周围。

#### 绝对定位

当元素的position为absolute或fixed时，为绝对定位。元素脱离文档流，位置上和其父元素及相邻元素不再有关系，其位置会使用 top、bottom、left 和 right 相对其包含块进行计算。

对固定位置的元素来说，其包含块为整个视口，该元素相对视口进行绝对定位，因此滚动时元素的位置并不会改变。

### 4.BFC和IFC是什么？

除了上文提到的**定位**规则，视觉格式化模型（Visual Formatting Model）是页面布局的更细粒度的算法机制。

Formatting context 是 W3C CSS2.1 规范中的一个概念。它是页面中的一块**独立的**渲染区域，有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素的关系和相互作用。核心作用在于这块**区域内部独立，不受外部的影响也不影响外部**。

最常见的 Formatting context 有 块级格式化上下文 Block fomatting context \(简称BFC\)和 内联格式化上下文Inline formatting context \(简称IFC\)。CSS2.1中只有BFC和IFC，CSS3中增加了GFC和FFC。

#### 形成BFC的条件

一个BFC包含创建该上下文的所有子盒，但不包括创建了新BFC的子盒的内部元素，也是为了强调BFC的**独立**原则。

自身满足下列条件之一就可产生一个BFC

* root element 根元素

* float:not none 存在浮动

* position:absolute，fixed 绝对定位

* display:inline-block, table-cell, table-caption, flex, inline-flex

* overflow:not visible

**那对于子元素有要求吗？**

我们知道context在英文中的含义是上下文，可以引申为是一种氛围，一种组成，一项规则。内部的盒形成了上下文，参与到上下文中，遵循上下文的规则。

BFC中的block-level box参与在BFC中，遵循BFC的规则。

IFC的inline-level box参与在IFC中，遵循IFC的规则。

父元素产生了一个BFC，虽然BFC的子元素中有span等行内元素，依然不影响这个BFC的存在，所以说内部盒可以不是块级盒的，但是只有块级盒会遵循BFC的规则。Formatting Context最重要的观念是强调**内部独立，不受外界影响**。

#### BFC规则

* 从左到右布局中，BFC的各个内部元素的左margin与包含块的左border内相接触，对于rtl布局相反。

* BFC不与浮动盒重叠。

* BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。（但是容器本身还是会受到影响）。

* 计算整个BFC的高度时，浮动元素也参与计算（可以用来解决浮动情况下的高度崩塌，清浮动）。

我们可以使用BFC和IFC的**独立**原则来解决一些问题。

#### BFC的使用：

##### 1.清浮动

```
<style>
.parent{width: 200px;border: 1px solid #e8e8e8;}
.child{float: left;width:100px;height: 100px;border: 1px solid #666;}
</style>
<div class="parent">
<div class="child"></div>
<div class="child"></div>
</div>
```

可以发现parent高度崩塌，根据**计算BFC的高度时，浮动元素也参与计算，**使parent成为BFC，加上`.parent{overflow:hidden}或float:not none;display:inline-block;position:absolute,fixed`都可清浮动

##### 2.解决margin重叠

```
<style>
.parent{width: 200px;border: 1px solid #e8e8e8;}
.child{margin:100px;width:100px;height: 100px;border: 1px solid #666;}
</style>
<div class="parent">
<div class="child"></div>
<div class="child"></div>
</div>
```

可以发现预期上下child之间的距离应该是400px但是实际是200px，所以想办法使其中一个child成为一个独立的BFC，不受到外界的影响，将其中一个child包一层`<div class="wrap"></div>,.wrap{overflow:hidden;}`

##### 3.两栏自适应实现

```
<style>
.content{width: 200px;border: 1px solid #e8e8e8;}
.main{height:200px;background: #fcc;}
.aside{width: 100px; height: 150px; float: left; background: #f66;}
</style>
<div class="content">
<div class="aside"></div>
<div class="main"></div>
</div>
```

现在的显示是content占一行，符合**从左到右布局中，BFC的各个内部元素的左margin与包含块的左border内相接触**。aside是浮动盒，想要实现两栏自适应，可以根据**BFC不与浮动盒重叠可以**加上`.main{overflow:hidden}`使main成为新的BFC即可。

以上所有的例子都体现了BFC最重要的**独立**原则。

BFC和浮动的关系：

浮动定位和清除浮动时只会应用于同一个BFC内的元素。

浮动不会影响其它BFC中元素的布局，而清除浮动只能清除同一BFC中在它前面的元素的浮动。

margin重叠也只会发生在属于同一BFC的块级元素之间。

#### IFC

在IFC中，盒子从顶部开始一个接一个水平排列。

盒模型在IFC的元素中应用的不是很完全，给行内元素设置水平方向的padiding，border，margin会生效，但是垂直方向的margin并不会生效，而且垂直方向的padding和border看起来生效了，但是不会影响周围的元素，因为在IFC中，垂直方向上其他的inline box不会被padding和border推开。

一个包含着一些排成一条线的盒子的外层盒称为line box。（float元素也是排成一行呀？？？）

vertical-align对齐方式也多种多样：baseline，middle，top，bottom等。

IFC的规则很复杂，所以经常遇到vertical-align根本不生效的情况，根本没有text-align那么好用。

```
<style>
strong {
margin: 20px;
padding: 20px;
border: 5px solid #ff6700;
}
</style>
<p>即使给
<strong>这里</strong>
设置了垂直方向上的margin也不会生效，设置垂直方向的padding和border也不会影响外部
</p>
```

IFC的规则

[https://segmentfault.com/a/1190000004246731](https://segmentfault.com/a/1190000004246731)

[https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Visual\_formatting\_model](#)

#### 回顾与总结：

1.行内元素和块级元素等基础知识总结。

2.定位规则有三种：1.普通流，2.浮动布局，3.绝对定位。

3.BFC乃至Formatting Context 最重要的特性就是**内部独立，不受外部影响**。可以用来解决**高度崩塌**，**清浮动**，**margin重叠**，**两栏自适应**等常见问题。

#### 常见的面试问题与细节思考：

1.position的四种属性有啥区别

2.BFC是什么

3.怎么清除浮动

4.两栏自适应和多栏自适应的实现方式

5.水平居中和垂直居中的实现方式

现在可以回答上来了吧。
