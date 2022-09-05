---
title: 区分 node / es6 不同的导入导出方式
tags: node
categories: JavaScript
abbrlink: 5fee
date: 2020-04-19 23:21:02
---

在前端开发中，书写 `js` 现在很倡导以模块导入导出的方式来定义不同功能的代码块，可能在很多代码中，会看到类似 `exports`, `module.exports`, `export`, `import`, `require()` 这些导入导出的方式，不知你是否能清楚的区分他们？下面逐一描述一下，这些方式应用的方法和环境。

各自的使用环境

<!-- more -->

> require : node / es6
> import/export : es6
> module.exports/exports : node

##  Node.js 中导入导出： module.exports / exports

`module.exports / exports` 属于 `Node.js` 的写法, `module.exports / exports = {}` 初始化时都是一个空对象，他们不同之处在于，`module.exports` 导出的内容会被 `require` 识别，但 `exports` 导出的内容不会被 `require` 识别，是因为 `exports` 其实是 `module.exports` 的一个引用。

`module.exports / exports` 导出的内容属于 `Object` 引用类型，传讲一个引用类型时，实际是定义了一个对象，并声明该对象指向该对象内存地址的一个引用。

```javascript
module.exports.name = '我是 module.exports'
exports = module.exports

console.log(module.exports) // {name: '我是 module.exports'}
console.log(exports)       //  {name: '我是 module.exports'}

exports.name = '我改名了'

console.log(module.exports) // {name: '我改名了'}
console.log(exports)       //  {name: '我改名了'}

var obj = {
  name: '我是一个对象',
  func: function(x) {
    return x
  }
}

exports = obj

console.log(module.exports) // {name: '我改名了'}
console.log(exports)       // {name: '我是一个对象', func: [Function]}
```

可以看出，初始化时，`exports = {}` 指向了 `module.exports = {}` ，他们的 `name` 属性指向同一个内存地址，所以在修改 `exports.name` 的时候，`exports.name` 和 `module.exports.name` 都被改变了；当 `exports = obj` 时，`obj` 在内存中指向了一个新的地址，`exports` 作为 `obj `的引用，也指向了这个新地址，所以 `exports.name `和 `module.exports.name` 值不相等了。

##  ES6 中导入导出：export / export default / import

`ES6` 语法导入导出还是十分灵活的，我们项目里也使用 `ES6` 写法，可以记住以下几点帮助你理解：

1. `export / export default` 可以导出常量，对象，模块，函数等；

2. `export` 可以导出多个变量，`export default` 只能导出一个变量；

3. `export` 导出的内容，需要 `import {}` 导入；`export default` 导出的内容，直接 `import` xxx 即可；

4. `export` 能导出变量表达式，`export default `不可以；

```javascript
// demo 1
export const num = 100
export const name = 'muzidx'

import { num, name } from 'demo1'


// demo 2
const num = 100
export default num

import num from 'demo2'


// demo 3
function hello(x) {
  console.log(x)
}

export { hello }

import { hello } from 'demo3'

// demo 4
export const a = 100
export const b = 200

import * as num from 'demo4'
num.a
num.b
```

