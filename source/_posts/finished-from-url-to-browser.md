---
title: 从 URL 到浏览器渲染完成
tags:
  - DOM渲染
  - TCP连接
categories: How-browsers-work
abbrlink: 41560
date: 2019-05-12 23:38:51
---

学完计算机网络到现在（感觉已经全部还给老师了），对于输入`URL`到浏览器渲染完成显示页面这个过程一直都只是一个比较模糊的概念，今天有空总结一下。

<!-- more -->

## 模糊概念

当你输入指定的`URL`（Uniform Resource Locator统一资源定位符）并按下回车，浏览器就开始干活了

1. `DNS`域名解析
2. 建立`TCP`连接
3. 发送`HTTP`请求
4. 服务器处理请求
5. 返回响应结果
6. 关闭`TCP`连接
7. 浏览器解析`HTML`
8. 浏览器渲染页面

以上8个过程，我大致分为两个部分：网络连接通信和页面渲染，进行详细了解

# 网络连接通信

每个网络设备间的通信都遵循`TCP/IP`协议，利用`TCP/IP`协议族进行网络通信时，会通过分层顺序与对方进行通信。`TCP/IP`模型分层自上而下分别为：应用层、传输层、网络层、数据链路层、物理层。发送端从应用层往下走，接收端从底层往上走。

## 输入 URL

用户输入的`URL`可以为域名或`IP`地址，使用域名是为了方便记忆，但是为了让计算机理解这个地址还需要把它解析为`IP`地址。上面我说到，当用户输入`URL`并按下回车，浏览器才开始工作其实是不正确的，现在主流浏览器引入了`DNS`预取技术。 浏览器为了加快域名`DNS`解析速度，会对网页的所有链接先做域名解析。所以，当用户在输入指定`URL`的时候，浏览器其实已经在本地缓存模糊搜索`URL`了，然后智能提示并助于用户补全`URL`，并且在用户还没有按下回车键的时候，浏览器就已经开始使用`DNS`预取技术解析该域名了。

## DNS 域名解析

- 浏览器客户端收到你输入的域名地址后，首先浏览器搜索自己的`DNS`缓存，其次是系统缓存，接着到本地的`hosts`文件查找有没有对应的域名映射，如果有，则向其`IP`地址发送请求
- 以上都没有找到对应的域名映射，再去找`DNS`服务器
- 浏览器客户端向本地`DNS`服务器发送一个含有域名`www.baidu.com`的`DNS`查询报文
- 本地`DNS`服务器把查询报文转发到根`DNS`服务器，根`DNS`服务器分析其后缀，然后向本地`DNS`服务器返回其后缀的服务器的`IP`地址，这里返回`comDNS`服务器的`IP`地址
- 本地`DNS`服务器再次向`comDNS`服务器发送查询请求，`comDNS`服务器注意到其`www.baidu.com`后缀并用负责该域名的权威`DNS`服务器的`IP`地址作为回应
- 负责该域名的权威`DNS`服务器解析该域名映射的`IP`地址并返回本地`DNS`服务器
- 最后，本地`DNS`服务器将含有`www.baidu.com`的`IP`地址的响应报文发送给客户端

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="41560\dns-server-queries.png" width="80%" height="80%"/></div>
> 从客户端到本地服务器属于**递归查询**，而`DNS`服务器之间的交互属于**迭代查询**
>
> 正常情况下，本地`DNS`服务器的缓存中已有`comDNS`服务器的地址，因此请求根域名服务器这一步不是必需的。

## 建立 TCP 连接

`TCP` 是一种面向有连接的传输层协议。它可以保证两端（发送端和接收端）通信主机之间的通信可达。它还能够处理在传输过程中丢包、传输顺序乱掉等异常情况，以及有效利用宽带，缓解网络拥堵。

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="41560\tcp.png" width="63%" height="63%"/></div>
客户端发送一个带有`SYN`标志的数据包给服务端，服务端收到后，回传一个带有`SYN/ACK`标志的数据包以示传达确认信息，最后客户端再回传一个带`ACK`标志的数据包，代表握手结束，连接成功。

> **SYN** ：在连接建立时用来同步序号。当`SYN=1`而`ACK=0`时，表明这是一个连接请求报文。对方若同意建立连接，则应在响应报文中使`SYN=1`和`ACK=1`。 因此，` SYN`置1就表示这是一个连接请求或连接接受报文。 
>
> **ACK**： 即是确认字符，在数据通信中，服务器发给客户端的一种传输类控制字符。表示发来的数据已确认接收无误。`TCP`协议规定，只有`ACK=1`时有效，也规定连接建立后所有发送的报文的`ACK`必须为1。
>
> **FIN （finis）**：完，终结的意思， 用来释放一个连接。当`FIN = 1 `时，表明此报文段的发送方的数据已经发送完毕，并要求释放连接。

## 发送 HTTP 请求

与服务器建立连接后，客户端就向服务器发起请求，产生`HTTP`请求报文。

一个`HTTP`请求报文由请求行`request line`、请求头部`header`、空行和请求数据4个部分组成。

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="41560\request-message.png" width="55%" height="55%"/></div>
在浏览器（Google）中查看报文首部：

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="41560\Message.jpg" width="80%" height="80%"/></div>
请求行，用来说明请求类型，要访问的资源以及所使用的HTTP版本；

请求头部，紧接着请求行之后的部分，用来说明服务器要使用的附加信息；

空行，请求头部后面的空行是必须的；

请求体，可以添加任意的其他数据；

通过报文还可以看到更多具体信息，这里就不进一步分析报文了。

## 服务器处理请求

服务器接受到客户端发送的HTTP请求后，查找客户端请求的资源，并返回响应报文。

访问资源可以是访问静态资源，这个就直接根据`URL`地址去服务器里找即可；如果访问动态资源的话要经过一个叫 [CGI](https://baike.baidu.com/item/CGI/607810) 的东西，再用服务端脚本处理，再返回给客户端。

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="41560\processing-requests.png"/></div>
## &ensp; 返回响应结果

服务器在处理完请求后，返回一个`HTTP`响应报文。

`HTTP`响应报文和请求报文的结构差不多，也是由四个部分组成：状态行、消息报头、空行、响应体。

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="41560\response-message.png" width="40%" height="40%"/></div>
响应报文中包括一个重要的信息一一状态码。状态码由三位数字组成，第一个数字定义了响应的类别，且有五种可能取值：

- 1xx：指示信息 – 表示请求已接收，继续处理
- 2xx：成功 – 表示请求已被成功接收、理解、接受
- 3xx：重定向 – 要完成请求必须进行更进一步的操作
- 4xx：客户端错误 – 请求有语法错误或请求无法实现
- 5xx：服务器端错误 – 服务器未能实现合法的请求

[**常见HTTP状态码**](https://hrealm.github.io/posts/20068.html#HTTP-状态码)

## 关闭 TCP 连接

为了避免服务器与客户端双方的资源占用和损耗，当双方没有请求或响应传递时，任意一方都可以发起关闭请求。与创建`TCP`连接的3次握手类似，不过关闭`TCP`连接，需要4次握手。

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="41560\closeTCP.png" width="63%" height="63%"/></div>
> **FIN**： 完，终结的意思， 用来释放一个连接。表明此报文段的发送方的数据已经发送完毕，并要求释放连接。
>
> **ACK**：即是确认字符，在数据通信中，服务器发给客户端的一种传输类控制字符。表示发来的数据已确认接收无误。

# 页面渲染

在服务器返回的响应报文中，响应体主要由`HTML`、`CSS`、`JS`，以及图片、视频等各种媒体资源。如果说响应的内容是`HTML`文档的话，就需要浏览器进行解析和渲染最后再呈现给用户。整个过程需涉及两个方面：解析和渲染。

## 浏览器解析 HTML

当浏览器获得一个`HTML`文件时，浏览器会将`HTML`解析成一个`DOM`树，`DOM`树的构建过程是一个深度遍历过程，当前节点的所有子节点都构建好后才会去构建当前节点的下一个兄弟节点。解析`CSS`成`CSS`规则数，`CSS`选择符是从右到左进行匹配的。然后通过`DOM`树和`CSS`规则树`Rule Tree`来构造渲染树`Rendering Tree`。

`Rendering Tree`渲染树与`DOM`树不同，渲染树中并没有`head`、`display:none`等不必显示的节点。

> 注意 `visibility: hidden` 与 `display: none` 是不一样的。前者隐藏元素，但元素仍占据着布局空间（即将其渲染成一个空框），而后者 (`display: none`) 将元素从渲染树中完全移除，元素既不可见，也不是布局的组成部分。

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="41560\render-tree-construction.png"/></div>
- 渲染树只包含渲染网页所需的节点
- 布局计算每个对象的精确位置和大小
- 最后一步是绘制，使用最终渲染树将像素渲染到屏幕上

## 浏览器渲染页面

有了`Render Tree`，浏览器已经能知道网页中哪些节点应该是可见的以及它们的计算样式，但尚未计算它们在设备视口内的确切位置和大小---这就是“布局”阶段，也称为“自动重排”。

为弄清每个对象在网页上的确切大小和位置，浏览器从渲染树的根节点开始进行遍历。

**需要注意的是**：上述过程是逐步完成的，为了更好的用户体验，渲染引擎会尽可能早的将内容呈现到屏幕上，并不会等到所有的`html`都解析完成之后再去构建和布局`render`树。这个过程是循序渐进的，浏览器解析完一部分内容就显示一部分内容，同时还可能通过网络下载其他资源内容。

根据渲染树布局，计算`CSS`样式，即每个节点在页面中的大小和位置等几何信息。`HTML`默认是流式布局的，`CSS`和`JS`会打破这种布局，改变`DOM`的外观样式以及大小和位置。这就触发浏览器进行`Repaint`重绘、`Reflow`回流。

> Repaint（重绘）：只改变某个元素的背景颜色、文字样式，不影响整体布局，浏览器将Repaint，重画屏幕某一部分
>
> Reflow（回流）：当某个元素发生了变化影响了整体布局，浏览器将需要倒回去重新渲染

`Reflow`比`Repaint`更花费时间，也就是更影响性能。所以在写代码的时候尽量避免过多的`Reflow`。

**减少 Reflow / Repaint**

- 采用`ClassName`修改`DOM`，减少重绘
- 对于动画的`HTML`，可使其脱离文档流，修改`CSS`不会导致回流
- 避免使用`table`布局，避免修改布局导致回流

最后浏览器绘制各个节点，将页面渲染到屏幕上。

# 总结

边写边思考，不断在补充。我发现这个话题可以延伸的太多太多了，无论是广度还是深度都还可以继续扩展，总结完全篇感觉还差很多，我仅限站在自己理解的角度上去堆积式的总结，以后待更新。

参考：[How browsers work](http://taligarsiel.com/Projects/howbrowserswork1.htm)、[渲染树构建、布局及绘制](https://developers.google.cn/web/fundamentals/performance/critical-rendering-path/render-tree-construction)