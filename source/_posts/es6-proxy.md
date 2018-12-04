---
title: 使用 ES6 Proxy 代理的一些问题记录
date: 2018-12-04 23:10:22
tags: ES6 Proxy
---


# 使用 ES6 Proxy 代理的一些问题记录

最近在项目里使用了 Proxy，遇到一些问题记录一下

### Proxy 简介
简单来说 Proxy 是对`对象`设置一个拦截，直接上代码
``` javascript
let obj = {
    attr: 1
};
// 对 obj 进行拦截
obj = new Proxy(obj, {
  get: function (target, key, receiver) {
    //如果 obj 有这个属性，则直接返回
    if(key in target) {
        return target[key];
    } 
    //如果 obj 没有这个属性，则统一返回 '没有这个值'
    return '没有这个值';
  },
  set: function (target, key, value, receiver) {
    // 想设置对象属性，直接返回（不让设置）
    return true;
  }
});

// 使用
console.log(obj.attr); // 输出 1
console.log(obj.abc); // 输出 '没有这个值', 

obj.attr = 2;//
console.log(obj.attr); // 输出 1, 不能设置对象属性

```
可以进行的拦截类型不止 `get, set`，还有很多，例如：（来自[阮一峰老师的书 ES6 Proxy](http://es6.ruanyifeng.com/#docs/proxy)）：

* set(target, propKey, value, receiver)：拦截对象属性的设置，比如proxy.foo = v或proxy['foo'] = v，返回一个布尔值。
* has(target, propKey)：拦截propKey in proxy的操作，返回一个布尔值。
* deleteProperty(target, propKey)：拦截delete proxy[propKey]的操作，返回一个布尔值。
* ownKeys(target)：拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性。
* getOwnPropertyDescriptor(target, propKey)：拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。
* defineProperty(target, propKey, propDesc)：拦截Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。
* preventExtensions(target)：拦截Object.preventExtensions(proxy)，返回一个布尔值。
* getPrototypeOf(target)：拦截Object.getPrototypeOf(proxy)，返回一个对象。
* isExtensible(target)：拦截Object.isExtensible(proxy)，返回一个布尔值。
* setPrototypeOf(target, proto)：拦截Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
* apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。
construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如new proxy(...args)。

如上这些都是介绍都是直接拷贝的 [阮一峰老师的书 ES6 Proxy](http://es6.ruanyifeng.com/#docs/proxy)


### 遇到的问题

#### 对原生的浏览器 HTMLElement 对象进行拦截
1、拦截时的 `this` 指向问题 [阮老师书里也有提及](http://es6.ruanyifeng.com/#docs/proxy#this-%E9%97%AE%E9%A2%98)

在拦截方法里，`this` 指向 Proxy 对象，所以在调用原对象的方法时，需要注意，直接看代码
``` javascript
let div = document.querySelector('div'); // 随便拿一个页面的 div, 对他进行代理

// 第一种代理方法
let divProxy = new Proxy(div, {
  get: function (target, key, receiver) {
    // 访问这个 div 的任何属性，都直接返回
    return target[key];
  }
});
// 调用 div 的 querySelector 方法，拿他下边的 a 标签
console.log(divProxy.querySelector('a')); // chrome 上会报错：Uncaught TypeError: Illegal invocation

```
如上的代码，猜测因为 querySelector 方法内部有访问 `this` 指向，导致报错。修改为如下代码，则可以修复这个问题：

``` javascript
let div = document.querySelector('div'); // 随便拿一个页面的 div, 对他进行代理

// 第二种代理方法
let divProxy = new Proxy(div, {
  get: function (target, key, receiver) {
    if( !!target[key] && !!target[key].bind) {
        // 使用 bind 绑定 this 指向
        return target[key].bind(target);
    } else {
        return target[key];
    }
  }
});
console.log(divProxy.querySelector('a')); // <a href='https://t.tt'>来买呀</a>

```

另外还有一个问题，通过代理设置对象属性时，也会有问题相同的，看代码吧
``` javascript
let div = document.querySelector('div');
let divProxy = new Proxy(div, {});
// 设置 div 的 innerHTML 属性，设置对象属性默认其实是调用 `div.innerHTML.__defineSetter__` 方法
divProxy.innerHTML = '<a href="https://t.tt">来买呀</a>'; //chrome 报错： Uncaught TypeError: Illegal invocation

```

直接上解决办法
``` javascript
let div = document.querySelector('div'); 
let divProxy = new Proxy(div, {
    // 拦截所有的 set 行为
    set: function (target, key, value, receiver) {
        if(key in target) {
            target[key] = value; // 直接调用原对象属性进行设置（target[key]）
        }
        return true;
    }
});
divProxy.innerHTML = '<a href="https://t.tt">来买呀</a>';
```


### 结尾
一般 `Proxy` 会和 `Reflect` 一起使用，本文不做介绍了，直接上[阮老师的书](http://es6.ruanyifeng.com/#docs/reflect)；

JavaScript 的语法越来越规范，之前的一些坑，新的标准规范也开始慢慢填，加上`babel`的普及使用，我们可以多多拥抱新语法、特性，简洁代码，愉悦自己。



