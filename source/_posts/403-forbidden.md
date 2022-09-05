---
title: 403 访问被拒绝
tags: api
categories: JavaScript
abbrlink: a2ab
date: 2019-09-14 15:57:40
---

在 [FreeMusic](https://github.com/Hrealm/freemusic) 项目中调用酷狗 MP4 资源的时候，无法播放，报403错误。然而直接在浏览器地址栏回车则可以正常播放。

报403错误则是访问被拒绝，既然我做了跨域处理还被拒绝这里就要提及浏览器的防盗链机制。

<!-- more -->

## 防盗链机制

当你的项目和需要访问的地址不在同一个域内，这时浏览器的防盗链机制就发挥作用了。其中防盗链是利用 `HTTP header` 中的 `referer` 来实现的。当浏览器向服务器发送请求时会带上 `referer` ，来告诉服务器从哪个页面链接过来的。

服务器通过识别 `referer` 来判断请求是否是自己的域名，如果不是自己的域名就会拦截，不会将请求发送出去，如果是自己域名就可以继续访问。

请求发送成功的请求头：

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="a2ab\1.jpg" width="90%" height="90%"/></div>
请求未发送成功的请求头：（会有Referer字段）

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="a2ab\2.jpg" width="90%" height="90%"/></div>
## 解决方法

这里我采用的是比较暴力的方法，直接在标签里加 `meta` 

`<meta name="referrer" content="never">`

在某些情况网站想要控制页面发送给服务器的 `referer` 信息时，可以使用 `referer metadata` 参数。

`referer` 的 `metadata` 属性可以设置 `content` 属性值为以下：

- default
- never
- always
- origin

`default` ：若当前页面使用的是 https 协议，而正要加载资源使用的是普通的 http 协议，则将 http header 中的 referer 置空；

`never` ：删除 http header 中的 referer，所有从当前页面发起的请求将不会携带 referer；

`always` ：不改变 http header 中的 referer 的值；

`origin` ：只发送 origin 部分；

