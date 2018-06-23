---
title: ES6值得一看的总结
date: 2018-06-23 11:59:24
tags: ES6
---

# 1. let 
## 1. 块级作用域在ES6浏览器中失效
看一下代码：

```
function f() { console.log('I am outside!'); }

(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }

  f();
}());
```
如果按照ES5去执行它的话他会打印I am inside!，理论上ES6会打印I am outside!。但是在ES6 浏览器实际执行中会报错： *f is not a function*。
这是为什么呢？ 实际上他的运行书序是这样的：
1. 先自执行函数中检测有没有函数、变量的声明，这时候if块作用域中的f声明也会被检测出来。
2. 代码开始前定义f（变量提升）
3. 开始执行函数体，因为f只是做了定义，并没有赋值，所以f是undefined。

<div style="background: #eee">     　　原来，如果改变了块级作用域内声明的函数的处理规则，显然会对老代码产生很大影响。为了减轻因此产生的不兼容问题，ES6 在附录 B里面规定，浏览器的实现可以不遵守上面的规定，有自己的行为方式。

- 允许在块级作用域内声明函数。
- 函数声明类似于var，即会提升到全局作用域或函数作用域的头部。
- 同时，函数声明还会提升到所在的块级作用域的头部。

.　　注意，上面三条规则只对 ES6 的浏览器实现有效，其他环境的实现不用遵守，还是将块级作用域的函数声明当作let处理。
</div>
# 2. const定义值为复合类型的变量
const声明值为复合类型的变量，只要保证当前变量的地址指向不变，改变其属性是被允许的。

```

const foo = {};
// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only

const a = [];
a.push('Hello'); // 可执行
a.length = 0;    // 可执行
a = ['Dave'];    // 报错


```
如果你想获得一个不能改变的对象：

```

// Object.freeze 作用是冻结一个对象 只能冻结该对象属性的简单类型
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;

// 除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数。

var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, i) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};
```
# 3. 函数参数的解构赋值
函数的参数也可以使用解构赋值。

```
[[1, 2], [3, 4]].map(([a, b]) => a + b);
```
有默认值的参数：

```
function move({x = 0, y = 0} = {}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]

// 但是这个默认值是为函数move的参数指定默认值，而不是为变量x和y指定默认值，所以会得到与前一种写法不同的结果。
// 调用时有传对象就不使用{ x: 0, y: 0 }
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]

// 如果你想要给变量x和y默认值，就这样写：
function move({x=0, y= 0}={ x: 0, y: 0}) {
	console.log(arguments)
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3,y: undefined}); // [3, 0]
move({}); // [undefined, 0]
move(); // [0, 0]

```
我们常用的：

```
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```
