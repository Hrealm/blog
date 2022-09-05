---
title: CSS 水平垂直居中
tags: css
categories: div+css
abbrlink: 29610
date: 2017-07-21 13:01:58
---

如何灵活使用`CSS`实现水平垂直居中？

今天总结一下，最近常见常用的几种`CSS`居中的方法

那么怎样使下面的子元素在父元素内实现水平垂直居中呢？

<!-- more -->

```html
<div class="parent">
	<div class="children"></div>		
</div>
```

## 若 已知子元素宽高+定位

可采用：通过设置父元素为相对定位，子元素为绝对定位。并设置子元素`top`与`left`各偏移`50%`，再通过`margin-*`来调整居中位置

```css
.parent{
    position: relative;
}

.children{
    position: absolute;
    width: $width;
    height: $height;
    left: 50%;
    top: 50%;
    margin-left: -$width/2;
    margin-top: -$height/2;
}
```

使用条件：需要知道子元素的宽高



## 若 未知子元素宽高+定位

可采用：通过设置父元素为相对定位，子元素为绝对定位。并设置子元素`top`与`left`各偏移`50%`，再通过`transform: translate(-50%,-50%);`来实现子元素自身的偏移

```css
.parent{
    position: relative;
}

.children{
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);
}
```

实用性非常高，在知道或者不知道的前提下均可使用



## 若 不用定位

在不使用定位的情况下，块级元素是否常用`margin：0 auto`来实现水平居中，同样道理，在不使用定位的情况下，我们可以通过`margin: 50% auto 0;`来实现水平居中，再通过刚刚所说的`transform`属性来使元素向上偏移自身高度的一半，来实现垂直居中

```css
.children{
    margin: 50% auto 0;
    transform: translateY(-50%);
}
```

在不使用定位的情况下，可采用这种方法



## 绝对居中

通过设置父元素为相对定位，子元素为绝对定位。并设置子元素`top`、`right`、`bottom`、`left`均偏移`0`，再通过`margin: auto;`来实现绝对居中

```css
.parent{
    position: relative;
}

.children{
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    margin: auto;
}
```

可根据所需情况来使用真正的绝对居中



## FlexBox 居中

使用弹性盒子布局，通过设置父元素`display`为`flex`，再通过子元素设置`margin: auto;`，也可以实现居中

```css
.parent{
    display: flex;
}

.children{
    margin: auto;
}
```



> 提一下一些比较不常用的方法



## table-cell 居中

表格布局现在基本少用了，不过仍可以了解一下，主要将父元素变成表格布局`display: table-cell;`，再利用表格的样式设置`vertical-align: middle;`，再通过子元素设置`margin: 0 auto;`，也可以实现居中

```css
.parent{
    display: table-cell;
    text-align: center;
    vertical-align: middle;
}

.children{
    margin: 0 auto;
}
```



## vertical-align: middle 居中

还可以通过`vertical-align: middle;`，也能实现居中。不过这种方法需要在`.children`中添加一个兄弟元素做为参照，并且`.children`与兄弟元素均需要设置为行内块元素，这样还不行，因为两个行内元素之间会解析一个空格，因此需要在`.parent`中设置`font-size: 0;`

```css
.parent{
    text-align: center;
    font-size: 0;
}
.middle{
    display: inline-block;
    height: 100%;
    vertical-align: middle;
}
.children{
    display: inline-block;
    vertical-align: middle;
}
```

另外还有一些，也是不常用的就不一一总结了

