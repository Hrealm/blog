---
title: vue 使用 axios 跨域处理
tags:
  - vue
  - axios
categories: JavaScript
abbrlink: 30727
date: 2019-08-20 21:56:31
---

小学期在做一个音乐网 [FreeMusic](https://github.com/Hrealm/freemusic) 需要很多跨域访问的 `API` ，所以随之而来就是如何解决跨域问题同时还支持本地 `API` ，网上方法很多不过大部分会遭受 [CORB](https://juejin.im/post/5cc2e3ecf265da03904c1e06) ，或者需要在服务器端设置跨域。

<!-- more -->

因为这个项目采用的是 `vue` 框架 + `axios` 进行 `http` 请求，所以结合 `vue` +  `axios` 进行跨域处理，先说说在服务器端设置跨域处理：

```json
header(“Access-Control-Allow-Origin:*”); 
header(“Access-Control-Allow-Headers:content-type”); 
header(“Access-Control-Request-Method:GET,POST”); 
```

服务器端设置跨域主要在于对请求头进行访问控制即可，很显然对于我本次访问的 `API` 是酷狗音乐的，我们无法操作服务器端。

结合 `vue` 里面的配置文件可以自己设置一个代理服务器，使用 `proxyTable` 进行配置，同时还可以配置本地 `API` ，在 `config/index.js` 里面找到 `proxyTable: {}` , 然后在里面进行如下配置：

```json
proxyTable: {
    "/kugouApi":{
        target:'http://m.kugou.com/',
        changeOrigin:true,
        pathRewrite:{
            '^/kugouApi':'/'
        }
    }，
    "/api":{
        target: 'http://localhost:8008/',
        changeOrigin: true,
        pathRewrite:{
            '^/api': '/'
        }
    }
}
```

`/kugouApi` 可自定义命名，`target` 设置你将要调用的接口域名和端口号，`changeOrigin` 将主机头的原点更改为目标URL，所以要设为 `true`，`pathRewrite` 路径重写，即可直接使用 `/kugouApi` 代替 `target` 里面的接口。如果要设置多个代理，同理直接在后面继续添加，注意这里设置了多个代理，`Axios.defaults.baseURL` 就不能在设置了。

使用：`http://m.kugou.com/?json=true`   =>  `/kugouApi/?json=true`

```javascript
this.axios.get('/kugouApi/?json=true').then(res=>{
	console.log(res);
}).catch(err=>{})
```

还可以在 `main.js` 中将代理挂载到 `vue` 的原型上

`Vue.prototype.host = '/api' `  请求的时候只需要使用 `this.host` + "路由参数地址" 即可