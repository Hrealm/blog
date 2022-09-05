---
title: 非跨域 Ajax 与 跨域 Jsonp
tags: 
  - ajax
  - jsonp
categories: JavaScript
abbrlink: 20068
date: 2018-11-29 21:38:05
---
Ajax 即“Asynchronous Javascript And XML”（异步 JavaScript 和 XML），是指一种创建交互式网页应用的网页开发技术。意思就是用JavaScript执行**异步网络请求**。

> 特点：在不刷新页面的前提下去传输数据

<!-- more -->

# Ajax 原生 js 写法

Ajax实现异步网络请求主要依靠`XMLHttpRequest`对象

## XMLHttpRequest 对象

XMLHttpRequest 对象用于在后台与服务器交换数据，能够在不重新加载页面的情况下更新网页，在页面已加载后从服务器请求数据，接收数据，在后台向服务器发送数据。所以**XMLHttpRequest对象是Ajax技术的核心所在。**

## 实现流程

1. 创建ajax对象
	
```js
 var xhr = new XMLHttpRequest();
```

2. 创建请求

```js
 xhr.open("get" , "/tongyuan/lujing" , true);
```

`open()`三个参数：方式，路径，是否异步

`true`表示脚本会在`send()`方法之后继续执行，而不等待来自服务器的响应。

3. 发送请求

```JS
 xhr.send();
```

4. 监听回调函数状态

```javascript
 xhr.onreadystatechange = function () { // 状态发生变化时，函数被回调
    if (xhr.readyState === 4) {
        // 判断响应结果:
        if (xhr.status === 200) {
            // 成功，通过responseText拿到响应的文本:
            return success(xhr.responseText);
        } else {
            // 失败，根据响应码判断失败原因:
            return fail(xhr.status);
        }
    } else {
        // HTTP请求还在继续...
    }
 }
```

## .readyState 请求状态码

- 0 未初始化 - 请求还没建立
- 1 启动 - 请求建立了，调用`open()`但未调用`send()`
- 2 发送 - 请求发送了，调用`send()`但未处理返回数据
- 3 接收 - 请求有部分返回数据可用了
- 4 完成 - 请求完成，所有数据可用

## .onreadystatechange 事件

当请求码`readyState`发生改变的时候触发

`status`响应的HTTP状态码

`.responseText`后台返回的文本内容

## HTTP 状态码

上面提到了`status`响应的HTTP状态码，那什么是HTTP状态码？当浏览者访问一个网页时，浏览者的浏览器会向网页所在服务器发出请求。当浏览器接收并显示网页前，此网页所在的服务器会返回一个包含HTTP状态码的信息头（server header）用以响应浏览器的请求。

常见的HTTP状态码大致有以下这些:

- 101：切换协议，服务器根据客户端的请求切换协议
- **200：请求成功。一般用于`GET`与`POST`请求**
- **301：永久重定向**
- **302：临时重定向**
- 303：与301类似。使用`GET`和`POST`请求查看
- **304：请求资源未修改，使用缓存**
- 307：与302类似。使用GET请求重定向
- **404：客户端请求失败**
- 408：请求超时
- **500：内部服务器错误，无法完成请求**
- 505：服务器不支持请求的HTTP协议的版本，无法完成处理

# JQuery 中的 Ajax

jQuery对原生Ajax进行了封装，使用方便快捷，写法也非常简洁。按照以下模式写好各个参数，就可以实现请求了。需要注意的是每个`API`所需的数据是不一样的，根据需要的数据进行填写参数。

```javascript
 $.ajax({
       type:    //数据的提交方式：get和post
       url:     //请求地址
       async:   //是否支持异步刷新，默认是true
       data:    //需要提交的数据
       dataType:   //服务器返回数据的类型，例xml,String,Json,jsonp等
       success:function(data){
		  //请求成功后的回调函数,参数data就是服务器返回的数据
       }   
       error:function(data){
		  //请求失败后的回调函数，根据需要可以不写，一般只写上面的success回调函数
       }   
    })
```

这里提一下`dataType`如果为`jsonp`&ensp;Ajax支持跨域调用

`jsonp`只支持`get`请求方式

# Jsonp 的工作原理

因为浏览器的同源策略，所以有些时候不得不需要跨域访问，这时就可以利用 `<script>` 标签没有跨域限制来达到与第三方通讯。

`Jsonp` 的工作原理：首先是利用 `<script>` 标签的 `src` 属性来实现跨域，通过将前端的一个 `callback` 函数作为参数传递到服务器端，然后服务器端将这个参数作为函数名来包裹数据再返回。

由于使用 `script` 标签的 `src` 属性，因此只支持 `GET` 请求