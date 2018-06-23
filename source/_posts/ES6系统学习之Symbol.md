---
title: ES6系统学习之Symbol
date: 2018-06-23 12:01:00
tags: ES6
---
## 1.Symbol的定义和作用
### 1.定义： 
首先Symbol不属于现有的六种类型中的任何一种，而是一种新的数据类型，表示独一无二的值。
### 2.为什么要引入Symbol类型：
在ES5中对象的属性名都是字符串，当你想要给一个已有的对象添加一个新的属性时有可能会将原有的属性覆盖掉。

```
let obj = {
    a： 1,
    ...
}

//给obj添加新的属性a
obj.a = 3 // 这样就让原有的属性a给覆盖掉了

```
如果属性的名字都是独一无二的就不会有这种问题了，而Symbol就是用来解决这个问题的。

---
 这样一来对象的属性就可以有两种类型了，一种是原本的字符串，一种是Symbol类型，而通过上面对Symbol的定义我们可以知道，只要是属性名是Symbol类型的，那么他就是唯一的，不会跟其他的属性名冲突。



### 3.Symbol的使用
####  1.用Symbol函数生成无参数的。

```
let s1 = Symbol()
let s2 = Symbol()

typeof s1 // symbol

s1 === s2 // false
s1 == s2 // false

```
要注意的是Symbol函数前不用加new，因为生成Symbol的是一个原始类型的值，而不是一个对象。由于Symbol值不是一个对象，所以不能添加属性，他基本上可以说是一种类似字符串的类型。

**这里说明一下：String 和 new String ()的区别：**

```
let str = 'abc'

let str1 = 'abc'

let strObj = new String('abc')
let strObj1 = new String('abc')


str == str1 // true

str == strObj // false

strObj == strObj1 // false


typeof str // string

typeof strObj // object
```
1. str的创建过程：
    1. 先栈区创建str的引用
    2. 然后在String池中去找值为 'abc'的对象，如果没有就创建一个，如果有str就直接指向这里的 'abc'
1. str1的创建过程：
    1. 先栈区创建st1的引用
    2. 然后直接找到直接存在的'abc'的然后指向他。
1. strObj的创建过程：
    1. 先栈区创建strObj的引用
    2. 在堆中创建对象，strObj指向这个对象

1. strObj1的创建过程：
    1. 先栈区创建strObj1的引用
    2. 在堆中创建对象(而不是直接指向strObj创建的对象)，strObj1指向这个对象

#### 2.带参数的Symbol值
Symbol可以接受一个参数来作为对生成的Symbol进行描述。但是要注意的是就算参数一致他们也是不等的。
```
// 没有参数的情况
let s1 = Symbol();
let s2 = Symbol();

s1 === s2 // false

// 有参数的情况
let s1 = Symbol('foo');
let s2 = Symbol('foo');

s1 === s2 // false
```
Symbol函数的参数只是表示对当前 Symbol 值的描述，因此相同参数的Symbol函数的返回值是不相等的。

#### 3. 一些特性
1. Symbol 值不能与其他类型的值进行运算，会报错。

```
let sym = Symbol('My symbol');

"your symbol is " + sym
// TypeError: can't convert symbol to string
`your symbol is ${sym}`
// TypeError: can't convert symbol to string
```

2. Symbol 值可以显式转为字符串。


```
let sym = Symbol('My symbol');

String(sym) // 'Symbol(My symbol)'
sym.toString() // 'Symbol(My symbol)'
```

3. Symbol 值也可以转为布尔值，但是不能转为数值。


```
let sym = Symbol();
Boolean(sym) // true
!sym  // false

let trues = Symbol(true)
let falses = Symbol(false)
!trues === !falses // true

if (sym) {
  // ...
}

Number(sym) // TypeError
sym + 2 // TypeError
```
### 3.作为属性名
1.  用Symbol类型来作为对象key值多用于一个对象由多个模块构成的情况，这避免了某一个属性被覆盖或者改写。由于Symbol类型的特性，用其来做对象的key和常规的写法有一些区别：

```
let mySymbol = Symbol();

// 第一种写法
let a = {};
a[mySymbol] = 'Hello!';

// 第二种写法 如果不放在[] 中的话就会被解析为字符串
let a = {
  [mySymbol]: 'Hello!'
};

// 第三种写法
let a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });

// 以上写法都得到同样结果
a[mySymbol] // "Hello!"
```
2.  要注意的是在使用Symbol为key的属性时，不能用点运算符，对象在用[]获取属性的时候要区别key加引号和不加引号的区别。

```
// 情况1
const mySymbol = Symbol();
const a = {};

a.mySymbol = 'Hello!';
a[mySymbol] // undefined
a['mySymbol'] // "Hello!"

// 情况2
const mySymbol = Symbol();
const a = {
    [mySymbol]: 'Hello!'
};

a[mySymbol] // "Hello!"
a['mySymbol'] // undefined
```
看情况1点运算符后总是跟的字符串，这样的操作定义或修改的属性并没有用Symbol作为key。

3.  作为常量，Symbol还可以定义一组值都不想等的常量。

```
log.levels = {
  DEBUG: Symbol('debug'),
  INFO: Symbol('info'),
  WARN: Symbol('warn')
};
log(log.levels.DEBUG, 'debug message');
log(log.levels.INFO, 'info message');
```
同时因为Symbol类型的特性，在switch循环中使用Symbol类型更加的合理。


```
const COLOR_RED    = Symbol();
const COLOR_GREEN  = Symbol();

function getComplement(color) {
  switch (color) {
    case COLOR_RED:
      return COLOR_GREEN;
    case COLOR_GREEN:
      return COLOR_RED;
    default:
      throw new Error('Undefined color');
    }
}
```
### 4.属性名的遍历
在有Symbol作为属性的对象中，这个一类型的属性不会被for...in、for...of遍历出来，而且也不会被Object.keys()、 Object.getOwnPropertyNames()、JSON.stringify()返回。**但是它不是一个私有属性**，用Object.getOwnPropertySymbols方法，可以获取指定对象的所有 Symbol 属性名。

```
let foo = Symbol("foo");

const obj = {
    [foo]: 'foobar', 
    a: 1
};



for (let i in obj) {
  console.log(i); // 无输出
}

// Object.getOwnPropertyNames()方法返回一个由指定对象的所有自身属性的属性
// 名（包括不可枚举属性但不包括Symbol值作为名称的属性）组成的数组。
Object.getOwnPropertyNames(obj)
// []

Object.getOwnPropertySymbols(obj)
// [Symbol(foo)]

```
你也可以使用另一个新的API--Reflect.ownKeys获取所有的key，包括常规的和Symbol类型的。

```
let obj = {
  [Symbol('my_key')]: 1,
  enum: 2,
  nonEnum: 3
};

Reflect.ownKeys(obj)
//  ["enum", "nonEnum", Symbol(my_key)]
```
### 5.Symbol.for()，Symbol.keyFor() 
#### 1.Symbol.for()
有时候我们希望重新使用同一个Symbol值，Symbol.for()就用来做这个的，Symbol.for方法接受一个字符串作为参数，看是否有以该参数为名称的Symbol值，如果有就直接返回这个值，没有的话就创建一个并且返回。

```
let s1 = Symbol.for('foo');
let s2 = Symbol.for('foo');

s1 === s2 // true
```
但是要注意的是 Symbol方法和 Symbol.for方法是不互通的：

```
let s1 = Symbol('foo');
let s2 = Symbol.for('foo');

s1 === s2 // false
```
由于Symbol()写法没有登记机制，所以每次调用都会返回一个不同的值，而Symbol.for方法会登记。
