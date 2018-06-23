---
title: ES6系统学习之let和const
date: 2018-06-23 12:00:37
tags: ES6
---
# 2.let和const命令
## 1. let
let的用法类似于var,但只在当前代码块中有效

```
1.
    {
      let a = 10;
      var b = 1;
    }
    
    console.log(a) // ReferenceError: a is not defined.
    console.log(b) // 1

2.
    var a = [];
    for (var i = 0; i < 10; i++) {
      a[i] = function () {
        console.log(i);
      };
    }
    a[6](); // 10
    
    var a = [];
    for (let i = 0; i < 10; i++) {
      a[i] = function () {
        console.log(i);
      };
    }
    a[6](); // 6
```
##### 不存在变量提升
什么叫做变量提升：
首先，在执行一个方法时，会先查找方法体中的function定义，然后找变量的定义；这些都找完后才开始执行,不理解的走[这里](https://github.com/mqyqingfeng/Blog/issues/8)，现在我们只说变量的声明：
- ES5
  1. 在方法体前先查看在方法中有没有变量声明，
  2. 执行语句，在执行前如果在1中发现有变量声明(这里举例为 var a = 1),那么实际执行顺序会先执行一步 *var a*, 然后再去从上到下去执行方法中的语句
- ES6　而ES6也会有ES5中的第一步，但它会假装没有执行第一步，直接按顺序执行方法中语句，后声明的变量会被当成没有声明处理。
   
举例：

```
console.log(foo); // 输出undefined
console.log(bar); // 报错ReferenceError

var foo = 2;
let bar = 2;
```

##### 暂时性死区
只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

```
// 存在全局变量tmp，但是块级作用域内let又声明了一个局部变量tmp，导致后者绑定这个块级作用域，所以在let声明变量前，对tmp赋值会报错。
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```
总之，在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称TDZ）。

```
ES6:
// 变量x使用let命令声明，所以在声明之前，
// 都属于x的“死区”，只要用到该变量就会报错。
// 因此，typeof运行时就会抛出一个ReferenceError。
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}

ES5:
if (true) {
  // TDZ开始
  tmp = 'abc'; 
  console.log(tmp); // abc

  var tmp; // TDZ结束
  console.log(tmp); // abc

  tmp = 123;
  console.log(tmp); // 123
}
```
**值得注意的一点是，在ES6之前typeof是百分百安全的，但是ES6中在let定义该变量前使用typeof该变量会报错。**

###### 有些“死区”比较隐蔽，不太容易发现：


```
function bar(x = y, y = 2) {
  return [x, y];
}

// 调用bar函数之所以报错，是因为参数x默认值等于另一个参数y，
// 而此时y还没有声明，属于”死区“
bar(); // 报错
```
##### 不允许重复声明

```
ES6:
// 报错
function () {
  let a = 10;
  var a = 1;
}

// 报错
function () {
  let a = 10;
  let a = 1;
}

ES5:
// 不报错
function () {
  var a = 10;
  var a = 1;
}
```
## 2.块级作用域
首先这是ES6新提出的一个规则

---

#### 1.在ES5中
在ES5中只有全局作用域和函数作用域，如果只有这两个作用域会造成很多不合理的场景： 
1. 内部变量可能会覆盖外层变量。

```
var tmp = 'bulabula';

function f(){
// 在执行这个方法前，先检查出了有tmp的定义，
// 所以在f的作用域中不使用外部的tmp
  console.log(tmp);
  if (false){
    var tmp = "hello world";
  }
}

f() // undefined
```
2. 用来计数的变量会泄漏为全局变量。
 
```
var s = 'hello';

for (var i = 0; i < s.length; i++){
  console.log(s[i]);
}

// 实际i的作用只是用来控制循环，但循环结束后他并没有被回收
console.log(i); // 5
```
#### 2.在ES6中
在ES6中不只有全局作用域和函数作用，let实际上为JavaScript新增了块级作用域

```
function f1() {
  var n = 5;
  if (true) {
    var n = 10;
  }
  console.log(n);
}

function f2() {
  let n = 5;
  if (true) {
    // 这里定义的n只能在if判断中使用
    let n = 10;
  }
  console.log(n);
}

f1() // 10
f2() // 5
```
ES6允许块级作用域({})的单独使用,而且可以嵌套多层。

```
// 这里外面四层{}只是为了说明可以嵌套多层
{{{{
  {let insane = 'Hello World'}
  console.log(insane); // 报错
}}}}
```
##### 但在ES6浏览器中会出现一下错误：
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

## 3.const
．　　从本质上来说用const定义的变量要保证指向的内存地址不变，对于简单的数据类型(比如数值、字符串、布尔值等)，值就保存在那个地址，改变值就会改变内存指向；如果是复合类型(数组、对象)只要保证当前的对象地址不变，改变他其中的属性是没有问题的。

---

### 1.基本类型
const声明的变量如果赋值为基本类型的值。一旦声明，常量的值就不能改变。

```
const PI = 3.1415;
PI // 3.1415

PI = 3; // TypeError: Assignment to constant variable.

// const定义的变量必须当场赋值
const foo; // SyntaxError: Missing initializer in const declaration

// 通let一样  const定义的变量必须先定义再使用
if (true) {
  console.log(MAX); // ReferenceError
  const MAX = 5;
}
```
### 2.复合类型
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
