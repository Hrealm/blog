---
title: 请求酷狗 API 身份认证
tags:
  - cookie
  - api
categories: JavaScript
abbrlink: 716d
date: 2019-09-03 23:26:36
---

在 [FreeMusic](https://github.com/Hrealm/freemusic) 项目中成功跨域访问酷狗 `API` 后，成功获取到了酷狗的很多数据，包括歌手的信息，歌曲列表等等，但今天在获取歌曲的播放链接的时候，请求失败返回：`{“status”:0,"err_code":20010,"data":[]}`

<!-- more -->

排查问题后发现，若在浏览器上输入酷狗歌曲详情 `API` ：

`http://www.kugou.com/yy/index.php?r=play/getdata&hash=1035269c05791f1665e36dffe478326c` 

显示请求失败， `{“status”:0,"err_code":20010,"data":[]}` ；然后我在浏览器上访问酷狗官网，[https://www.kugou.com](https://www.kugou.com) 然后随便播放一首歌,，首次会出现一个滑动验证，才可以播放；我再次尝试刚刚那个 `API` 请求，结果请求成功了，成功得到数据 ` {“status”:1,"err_code":0,"data":{20}}` 

显然，问题就在刚刚的滑动验证，这其实就是一个设备的身份认证，所以直接排查新增的 `cookie` 。在测试后发现，在原有的基础上需要携带 `cookie` 里面的 `kg_mid` 参数才能正常获取，而且是在认证后生成的，所以 `kg_mid` 参数是浏览器生成的而不是服务器返回的。

## 添加 cookie

既然请求需要带上 `cookie` 身份认证参数才能成功获取，尝试在项目初始化时默认添加 `kg_mid` 参数

```javascript
document.cookie = "kg_mid=1923c6069c305133056773d8fce002ce;expires = Session";
```

这里的 `kg_mid` 参数是直接在酷狗认证的浏览器上拿下来测试。再次请求

```javascript
 let url = '/playApi/yy/index.php?r=play/getdata&hash='+ hash;
 this.axios.get(url).then(res=>{
 	console.log(res.data)
 };
```

成功得到数据 ` {“status”:1,"err_code":0,"data":{20}}` 

## 分析 kg_mid

首先我查看了所有的请求，然后筛选后只保留了获取 `JS` 文件的请求，确定具体是哪个文件最死板的方法是一个个打开然后去查有没有 `kg_mid` 这个参数的相关生成方法，但是这里我们根据 `JS` 文件的名字推测出应该是这个 `kguser_min.js` 文件，因为 `Cookie` 就是相当于用户的身份，所以根据 `user` 能推测出应该是这个文件

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="716d\1.jpg" width="90%" height="90%"/></div>
然后将这个 `JS` 文件下载并格式化，因为是压缩后的文件，接下来就是查找 `kg_mid` ，猜测没有错误

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="716d\2.jpg" width="90%" height="90%"/></div>
根据源码，`kg_mid` 是 `KgMid` 对象的一个属性值，根据 `KgMid` 对象继续查找，因为上下文出现 `KgMid` 对象比较多，所以在边找时候要联系上下文看一下，直到找到这个

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="716d\3.jpg" width="90%" height="90%"/></div>
这段代码里面，需要注意两个方法：`KgUser.Guid()` 和 `KgUser.Cookie.write()` ， `Guid()`方法很明显是按照一定规则返回一个生成的参数，这里 `S4()` 返回的是一个长度为4的随机字符串，每个字符都属于 **a-z / 0-9** 的范围，最后 `Guid()` 返回的是8个 `S4()` 拼接的字符串，再经过Md5加密

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="716d\6.jpg" width="90%" height="90%"/></div>
再来看看 `Cookie.write()` ，其中第一和第二个参数是最主要的，对应 `KgUser.KgMid.name` 和 `KgUser.Md5(n)` 

<div style="width:100%;height:100%;text-align:center;font-size: 0;"><img src="716d\5.jpg" width="90%" height="90%"/></div>
到这里大概知道 `kg_mid` 是如何生成的了，既然知道是如何生成，不妨在项目初始化的时候仿造同样的方式来生成 `kg_mid` 。

## 生成 kg_mid

其实 `Cookie.write()` 主要的参数就是前两个 **name，value**  ，这里 name 取默认即可，然后根据生成 value 所需的方法添加到项目 `methods` 中

```javascript
/* 函数具体内容直接复制源码部分即可 */
methods: {
    Guid: function() {},
    Md5: function(str) {},
}
```

接下来就是在项目初始化的时候生成 `kg_mid` 并添加到 `cookie` 中

```javascript
created(){
    var n = this.Guid();
    document.cookie = `kg_mid=${this.Md5(n)};expires = Session`;
},
```

