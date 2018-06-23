---
title: ES6系统学习之proxy
date: 2018-06-23 12:01:21
tags: ES6
---
# 1.定义

.　　proxy用于修改某些操作中的默认行为，等同于在语言层面上做出修改，所以属于一种"元编程"，相当于是对编程语言进行编程。

.　　举个栗子，现在有一个生产某种产品的流水线，这个流水线上每一个节点都有一个工人，现在原来的某个节点已经不满足当前的需求了，现在老板请了一个叫proxy的技术来指导工作，他可以在某个工人开始前先对产品进行一次处理，或者直接代替他的工作。

*当我阅读完文档后我严重怀疑vue内部的数据变化监听就是采用proxy的方式实现的(当然我没有阅读过vue源码)， 在文中我会说下我脑子中的vue监听数据实现逻辑。*

# 2.用法
看个例子，这里我们对一个空对象的get和set方法进行了重新定义

```
var obj = new Proxy({}, {
  // 这里是对get的方法进行了重写，先进行了自己的操作(console)，然后执行默认的get方法，相当于在执行get之前进行了自己的操作
  get: function (target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },
  // set同get
  set: function (target, key, value, receiver) {
    console.log(`setting ${key}!`);
    return Reflect.set(target, key, value, receiver);
  }
});

// 在这里使用obj的set方法，会执行修改后的set方法
obj.a = 1 
// setting a!

// 在这里使用obj的get方法，会执行修改后的get方法
console.log(obj.a)
// getting a!
// 1
```
**Reflect 是一个内置的对象，它提供拦截 JavaScript 操作的方法。这些方法与处理器对象的方法相同。Reflect不是一个函数对象，因此它是不可构造的。*

看以上的代码说明在进行obj的读写前proxy重载了点运算符，也就是所覆盖了原始的语言定义。

ES6提供了原始的proxy构造函数，用于构造proxy实例。
```
// target 指的是即将拦截的对象
// handler 也是一个对象，用来定制拦截行为
var proxy = new Proxy(target, handler);
```




```

// 这里直接对proxy对象的get进行了修改，所有获取到的值都是固定值
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

proxy.time // 35
proxy.name // 35
proxy.title // 35
```
.　　上面代码中，作为构造函数，Proxy接受两个参数。第一个参数是所要代理的目标对象（上例是一个空对象），即如果没有Proxy的介入，操作原来要访问的就是这个对象；第二个参数是一个配置对象，对于每一个被代理的操作，需要提供一个对应的处理函数，该函数将拦截对应的操作



如果handler没有设置任何拦截，那就等同于直接通向原对象。

```

// 这时handler是一个空对象
var target = {};
var handler = {}
var proxy = new Proxy({}, {});
proxy.a = 'b';
target.a // "b"

```

