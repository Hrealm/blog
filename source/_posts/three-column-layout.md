---
title: CSS 三栏布局
tags: css
abbrlink: 8955
date: 2017-08-08 14:43:56
categories: div+css
---

> 三栏布局：指左右两栏宽度固定，中间一栏自适应的布局方法
>
> 本文将总结几种常见常用的 `CSS` 三栏布局方法

<!-- more -->

## 浮动布局

左右模块各自向左右浮动，并设置中间模块的 `margin` 值使中间模块宽度自适应

优点：布局简单，兼容性良好

缺点：中间一栏主要内容无法最先加载，当页面内容较多时会影响用户体验；`DOM` 结构必须是先写浮动部分，在到中间一栏；浮动布局是有局限性的，浮动元素脱离文档流，要做好清除浮动；


```CSS
<div class="box">
    <div class="left"></div>
    <div class="right"></div>
    <div class="center"></div>
</div>

<style type="text/css">
    .box{
    	height: 500px;
    }
    .left{
        float:left;
        width: 200px;
        height: 100%;
        background-color: red;
    }
    .right{
        float: right;
        width: 300px;
        height: 100%;
        background-color: green;
    }
    .center{
        height: 100%;
        margin-left: 220px;
        margin-right: 320px;
        background-color: blue;
    }
</style>
```



##  BFC 布局

块格式化上下文 `（Block Formatting Context，BFC）` 是 Web 页面的可视化 `CSS` 渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域。

利用 `BFC` 不与浮动元素叠加的特点，让中间一栏通过设置 `overflow` 属性拥有 `BFC` 特性，实现三栏布局

由于 `DOM` 结构并未改变，同样中间一栏并不能先加载

```CSS
<div class="box">
    <div class="left"></div>
    <div class="right"></div>
    <div class="center"></div>
</div>

<style type="text/css">
    .box{
        height: 500px;
    }
    .left{
        float:left;
        width: 200px;
        height: 100%;
        margin-right: 20px;
        background-color: red;
    }
    .right{
        float: right;
        width: 300px;
        height: 100%;
        margin-left: 20px;
        background-color: green;
    }
    .center{
        height: 100%;
        overflow: auto;
        background-color: blue;
    }
</style>
```



## 绝对布局

只要设置左右两栏分别绝对定位到两端，中间一栏通过 `margin` 创建安全输入区即可。

优点：简单实用，并且内容区可优先加载

缺点：脱离文档流，使用时需要注意

```CSS
<div class="box">
    <div class="center"></div>
    <div class="left"></div>
    <div class="right"></div>
</div>

<style type="text/css">
    .box{
        position: relative;
        height: 500px;
    }
    .left{
        position: absolute;
        left: 0;
        top: 0;
        width: 200px;
        height: 100%;
        background-color: red;
    }
    .right{
        position: absolute;
        right: 0;
        top: 0;
        width: 300px;
        height: 100%;
        background-color: green;
    }
    .center{
        height: 100%;
        margin-left: 220px;
        margin-right: 320px; 
        background-color: blue;
    }
</style>
```



## Flexbox 布局

`flex` 弹性盒模型布局，`flex-direction` 默认 `row` ，即按行排列。再配合 `CSS` [order](https://developer.mozilla.org/zh-CN/docs/Web/CSS/order) 属性，规定弹性容器中的可伸缩项目在布局时的顺序。元素按照 `order` 属性的值的增序进行布局。拥有相同 `order` 属性值的元素按照它们在源代码中出现的顺序进行布局。

优点：`DOM` 结构可以是中间一栏再到左右两栏，主要内容可优先加载

缺点：可能需要考虑兼容性问题

```CSS
<div class="box">
    <div class="center"></div>
    <div class="left"></div>
    <div class="right"></div>
</div>

<style type="text/css">
    .box{
        display: flex;
        height: 500px;
    }
    .left{
        order: -1;
        width: 200px;
        height: 100%;
        background-color: red;
    }
    .right{
        order: 1;
        width: 300px;
        height: 100%;
        background-color: green;
    }
    .center{
        flex: 1;
        order: 0;
        margin: 0 20px;
        height: 100%;
        background-color: blue;
    }
</style>
```



## 圣杯布局

- DOM 结构：center，left，right 依次布局
- 将 `center` 的宽度设为 100%，center，left，right 均向左浮动，即 `float: left;`
- 在通过设置 `margin-left` 将左右两栏分别重叠到中间一栏，左边一栏为 `-100%` ，右边一栏为 `-$width` 

通过以上布局，三栏布局已成初形，不过，中间一栏的安全输入区被左右两栏遮盖住，所以还需：

- 将三栏布局的最外层 box 设置左右 `padding` 为左右两栏的宽度 + 中间间隙
- 再通过 `position: relative` 将左右两栏的 `left: -$width` ，`right: -$width` 移至对应位置

由于 DOM 结构，中间一栏优先加载；缺点：注意清除浮动

```CSS
<div class="box">
    <div class="center"></div>
    <div class="left"></div>
    <div class="right"></div>
</div>

<style type="text/css">
    .box{
        padding-left: 220px;
        padding-right: 320px;
        height: 500px;
    }
    .left{
        position: relative;
        left: -220px;
        float: left;
        width: 200px;
        height: 100%;
        margin-left: -100%;
        background-color: red;
    }
    .right{
        position: relative;
        right: -320px;
        float: left;
        width: 300px;
        height: 100%;
        margin-left: -300px;
        background-color: green;
    }
    .center{
        float: left;
        width: 100%;
        height: 100%;
        background-color: blue;
    }
</style>
```



## 双飞翼布局

步骤基本同圣杯布局，不过双飞翼布局在中间一栏里面再嵌套一个块级元素，并且通过设置 `margin` 使这个块级元素的输入内容不会被左右两栏给遮盖。同时不用再设置左右两栏的 `position` ，和不用设置外层 box 的 `padding` ，即可实现三栏布局。

由于 DOM 结构并未改变，所以中间一栏内容区域优先加载；缺点：注意清除浮动

```CSS
<div class="box">
    <div class="center"><div class="main"></div></div>
    <div class="left"></div>
    <div class="right"></div>
</div>

<style type="text/css">
    .box{
        height: 500px;
    }
    .left{
        float: left;
        width: 200px;
        height: 100%;
        margin-left: -100%;
        background-color: red;
    }
    .right{
        float: left;
        width: 300px;
        height: 100%;
        margin-left: -300px;
        background-color: green;
    }
    .center{
        float: left;
        width: 100%;
        height: 100%;
    }
    .main{
        height: 100%;
        margin-left: 220px;
        margin-right: 320px;
        background-color: blue;
    }
</style>
```



## 总结

除了以上几种布局均可以实现三栏布局外，还有 **Table 布局**也可以实现，不过无法设置栏间距，并且中间一栏无法优先加载；其次还有**网格布局** `display: grid;` 也可以实现，且存在一定的兼容性问题，这两种布局均不在代码举例。以上代码举例布局方法都介绍了各自的优缺点，具体使用哪种方法可根据具体情况选择。