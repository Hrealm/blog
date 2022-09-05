---
title: 一文了解 apply、call、bind
abbrlink: e171
date: 2018-10-06 23:43:10
categories: JavaScript
---

改变函数执行时的上下文，具体一点就是改变函数运行时的 `this` 指向，这就是 `apply()`、`call()`、`bind()` 存在的意义。具体什么时候使用哪一个函数来改变 `this` 指向，就需要了解它们之间的区别和应用。

<!-- more -->

## apply() 与 call()

`apply()` 和 `call()` 两者改变 `this` 指向的方式类似，在改变函数的 `this` 上下文后便执行该函数，区别在于两者接收的参数的方式不一致。

`apply()` 方法接收两个参数，第一个是指定函数运行的上下文 `this`，第二个参数是一个数组，数组第一个值对应原函数的第一个参数，数组第二个值以此类推。

`call()` 方法的第一个参数跟 `apply()` 相同，从第二个参数开始分别对应原函数的形参。

```javascript
let sum = {
    name:'sum',
    computed: function(num1,num2){
        console.log(`${this.name}:${num1+num2}`)
    }
}
let obj ={
    name: 'obj'
}
//不改变函数上下文
sum.computed(1,2);  //sum:3

//使用 apply()
sum.computed.apply(obj,[1,2]);  //obj:3

//使用 call()
sum.computed.call(obj,1,2);  //obj:3

```

`apply()` 和 `call()` 两者都改变了函数运行的上下文，如果调用的函数不用传参，`apply()` 和 `call()` 的效果都是一样的。当调用函数不知道需要传递参数个数时，则可以使用 `apply()` 。

## bind()

`bind()` 同样是可以改变函数内 `this` 指向，但与 `apply()`、`call()` 不同的是，`bind()` 方法不会立即调用，而是创建返回一个新函数，称为绑定函数，其内的 `this` 指向为创建它时传入 `bind()` 的第一个参数，而传入的第二个及以后的参数作为原函数的参数来调用原函数。

```javascript
//使用 bind()
let _bind = sum.computed.bind(obj,1,2);
_bind();  //obj:3

//简写
sum.computed.bind(obj,1,2)();  //obj:3
```

当使用 `bind()` 改变函数的 `this` 上下文后，则返回改变了上下文后的新函数。也就是说，在我们需要使用一个新上下文的旧函数时候，可以使用 `bind()` 绑定一个新的上下文到函数中，从而产生新的绑定函数，然后再运行这个有指定 `this` 对象的函数。

 ## 关于变量保存 this

函数在被调用时都会自动取得两个特殊变量：`this` 和 `arguments` 。内部函数在搜索这两个变量时，只会搜索到其活动对象为止，因此永远不可能直接访问外部函数中的这两个变量。不过，把外部作用域中的 `this` 对象保存在一个闭包能够访问到的变量里，就可以让闭包访问该对象了。

```javascript
let obj = {
    name: 'hrealm',
    getName: function(){
        //使用变量保存this
        let _this = this;
        return function(){
            return _this.name;
        };
    }
};
console.log(obj.getName()());  //'hrealm'
```

这里可以直接使用 `bind()` 来完成，并把外部函数 `this` 指向绑定到内部函数。

```
let obj = {
    name: 'hrealm',
    getName: function(){
        return function(){
            return this.name;
        }.bind(this);
    }
};
```

## apply 与 call 的性能

在知道调用函数的参数个数时，使用 `call()` 的性能会优于 `apply()` ,编译的的效率更高。因为在 `apply()` 实现过程中，需要对第二个参数进行类数组的长度判断等额外的操作。

所以在当有 `this` 指向或者执行参数时， `call()` 的性能要优于 `apply()`。

参考：[call和apply性能对比]( https://blog.csdn.net/zhengyinhui100/article/details/7837127 )