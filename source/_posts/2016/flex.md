---
title: flex布局
date: 2016-11-30 15:31:13
tags:
categories:
---

布局的传统解决方案，基于[盒状模型](https://developer.mozilla.org/en-US/docs/Web/CSS/box_model)，依赖 display属性 + position属性 + float属性。它对于那些特殊布局非常不方便，比如，[垂直居中](https://css-tricks.com/centering-css-complete-guide/)就不容易实现。

Flex是Flexible Box的缩写，意为"弹性布局"，用来为盒状模型提供最大的灵活性。
任何一个容器都可以指定为Flex布局。
<!-- more -->
```css
.box{
  display: flex;
  // display: inline-flex;
}


.box{
  display: -webkit-flex; /* Safari */
  display: flex;
}
```
注意，设为Flex布局以后，子元素的float、clear和vertical-align属性将失效。
采用Flex布局的元素，称为Flex容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为Flex项目（flex item），简称"项目"。

### 容器的属性

1. flex-direction:
>- row（默认值）：主轴为水平方向，起点在左端。
>- row-reverse：主轴为水平方向，起点在右端。
>- column：主轴为垂直方向，起点在上沿。
>- column-reverse：主轴为垂直方向，起点在下沿。

2. flex-wrap:
 nowrap | wrap | wrap-reverse

3. flex-flow
flex-flow属性是flex-direction属性和flex-wrap属性的简写形式，默认值为row nowrap。

4. justify-content
>- flex-start（默认值）：左对齐
>- flex-end：右对齐
>- center： 居中
>- space-between：两端对齐，项目之间的间隔都相等。
>- space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。

5. align-items
>- flex-start：交叉轴的起点对齐。
>- flex-end：交叉轴的终点对齐。
>- center：交叉轴的中点对齐。
>- baseline: 项目的第一行文字的基线对齐。
>- stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。

6. align-content 属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。
>- flex-start：与交叉轴的起点对齐。
>- flex-end：与交叉轴的终点对齐。
>- center：与交叉轴的中点对齐。
>- space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
>- space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
>- stretch（默认值）：轴线占满整个交叉轴。

### 项目的属性

1. order 属性定义项目的排列顺序。数值越小，排列越靠前，默认为0。
2. flex-grow 定义项目的放大比例，默认为0，即如果存在**剩余空间**，也不放大。0 为默认不放大，为文本本身的长度，如果没有则不显示
3. flex-shrink 属性定义了项目的缩小比例，默认为1，即**如果空间不足**，该项目将缩小。1 为默认，为原始大小，
4. flex-basis  属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，**计算主轴是否有多余空间**。它的默认值为auto，即项目的本来大小
5. flex  flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 0 auto。后两个属性可选。
6. align-self  属性允许单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。
7. margin : auto 设置"margin"值为"auto"值，自动获取弹性容器中剩余的空间。所以设置垂直方向margin值为"auto"。可以使弹性子元素在弹性容器的两上轴方向都**完全集中**。
以下实例在第一个弹性子元素上设置了 margin-right: auto; 。 它将剩余的空间放置在元素的右侧：

flex ：默认	0 1 auto ; auto<=> " 1 1 auto "; none<=> "0 0  auto"
参考一下代码，分别改变flex的三个值，去理解其中三个值的含义。
当总长度-去flex-basis的值之后，根据剩余的空间或者缩小比例 来改变grow或者shrink的值。
```html
<style>
#main {
    width: 350px;
    height: 100px;
    border: 1px solid #c3c3c3;
    display: flex;
}
#main div {
    flex: 0 1 80px;
}

#main div:nth-of-type(5) {
  flex: 0 1 30px;
}
flex-grow : 
flex-shrink: 
</style>
</head>
<body>

<div id="main">
  <div style="background-color:coral;">1</div>
  <div style="background-color:lightblue;">2</div>
  <div style="background-color:khaki;">3</div>
  <div style="background-color:pink;">4</div>
  <div style="background-color:lightgrey;">5</div>
</div>
```

[示例](http://www.runoob.com/try/try.php?filename=trycss3_flexbox_margin)

### DEMO

[参考](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
[骰子](https://gist.github.com/changyuan/cce6e8818c038760d8cadccfd8f96824)
[圣怀布局](https://gist.github.com/changyuan/02c14ca2aabc7107950d0056fa654522)
[常用后台布局](https://gist.github.com/changyuan/77cfcd7344a8522b7e365865a0428c2d)