---
title: ES6系统学习之解构赋值
date: 2018-06-23 12:00:50
tags: ES6
---
## 1.数组的解构赋值
#### a.基本的解构赋值
```
ES5:
var a = 1
var b = 2

ES6:
var [a, b] = [1, 2]

```
本质上，ES6这种写法属于“模式匹配”，只要等号两边的数据结构一样，左边的变量就能赋上右边同位置的值。栗子：

```
// 给基本类型的变量赋值
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

// 如果左侧对应位置上没变量，则跳过(不赋值)
let [ , , third] = ["foo", "bar", "baz"];
third // "baz"
// 只给对应位置赋值
let [x, , y] = [1, 2, 3];
x // 1
y // 3

// 如果在右边没有对应的值，那么左边的变量只是定义，并没有赋值
let [one, two, three] = [1, 2];
one // 1
two // 2
three // undefined
let ones = []
ones // undefined

// 给数组赋值
let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

// 如果是要给数组赋值而右边没有，则默认为空数组。
let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```
只找结构对应的位置的：

```
let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
d // 4
```
如果等号右边不是可以遍历的结构会报错：

```
// 报错
// 下面1-5 转为对象以后不具备 Iterator 接口
let [foo] = 1;
let [foo] = false;
let [foo] = NaN;
let [foo] = undefined;
let [foo] = null;
// 本身就不具备 Iterator 接口
let [foo] = {};
```
实际上只要某种数据结构具有Iterator接口，都可以使用数组形式解构。

```
// fibs是一个 Generator 函数(参见《Generator 函数》)
function* fibs() {
  let a = 0;
  let b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

let [first, second, third, fourth, fifth, sixth] = fibs();
sixth // 5
```
#### b.默认值
解构赋值可以有默认值：

```
当右边没有值时，如果有默认值就用默认值
let [foo = true] = [];
foo // true
let [x, y = 'b'] = ['a']; // x='a', y='b'

// 当值为undefined时用默认值，没有默认值时为undefined。这个过程可以这样理解：
// 如果右边对应位置是undefined，那么这个位置上就相当与没有值。
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'

// 如上面的解释，只要有值且不是undefined，那么就可以赋值，而不是使用默认值。
let [x = 1] = [null];
x // null
```
如果默认值是一个表达式, 那么这个表达式只会在用到的时候执行。在阮大牛的书里把它叫做惰性求值。

```
// 这里f并没有执行，因为右边有值，所以就没有用到默认值。
function f() {
  console.log('aaa');
}

let [x = f()] = [1];

// 反之右边没有值，那么y就会使用默认值，这时候f()才会执行(虽然执行后y依然是undefined)。
function f() {
  console.log('aaa');
}

let [y = f()] = []; // aaa
y // undefined
```
默认值可以引用解构赋值的其他变量，但该变量必须已经声明。

```
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError
```
左边定义的变量是不能放在右边当值用的。

```
let str = '123'
let [x, y] = [1, str]; 
x // 1
y // 123

let [a, b] = [1, a]; // ReferenceError
```
## 2.对象的结构赋值
结构赋值不光可以用数组还可以使用对象，与数组不同的是数组是根据次序排列对应，而对象是要两边的属性名一致。

```
let { foo, bar } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"

// 不同的顺序只要变量名和属性名保持一致
let { bar, foo } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"

// 其实上面的{ bar, foo }是es6的简写，也可以写成{ bar: bar, foo: foo } 即：
let { foo: foo, bar: bar } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"

// 这样我们就能很好的理解，并且延伸：
let { foo: bar, bar: foo } = { foo: "aaa", bar: "bbb" };
foo // "bbb"
bar // "aaa"

```
与数组一样，解构也可以用于嵌套结构的对象：

```
// 只要保证前面的
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p: [x, { y }] } = obj;
x // "Hello"
y // "World"

```
对象解构赋值中走左边的对象中属性是可以重复的：

```
let {a: a,a: b,a: c} = {a: 1}
a // 1
b // 1
c // 1
```
那么对于上上个例子我们可以这样写：

```
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p: arr, p: [x, { y }] } = obj;
x // "Hello"
y // "World"
arr // [ 'Hello', { y: 'World' } ]

// 延伸下： 
const node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

let { loc, loc: { start }, loc: { start: { line }} } = node;
line // 1
loc  // Object {start: Object}
start // Object {line: 1, column: 5}
// 上面代码有三次解构赋值，分别是对loc、start、line三个属性的解构赋值。
// 注意，最后一次对line属性的解构赋值之中，只有line是变量，loc和start都是模式，不是变量。

// 玩个花的：
let obj = {};
let arr = [];

({ foo: obj.prop, bar: arr[0] } = { foo: 123, bar: true });

obj // {prop:123}
arr // [true]

```
像数组结构赋值一样，对象组结构赋值可以有默认值，当右边对象的属性值严格等于undefined时默认值才会生效：


```
let {x, y = 5} = {x: 1};
x // 1
y // 5

let {x: y = 3} = {};
y // 3

let {x: y = 3} = {x: 5};
y // 5


let {x, y = 5} = {x: 1, y: null};
x // 1
y // null

let {x, y = 5} = {x: 1, y: undefined};
x // 1
y // 5
```
如果左边的属性名在右边没有那么这个变量就会使undefined，反之右边的属性名在左边没有，不会有任何影响

```
let {foo} = {bar: 'baz'};
foo // undefined

// 报错
let {foo: {bar}} = {baz: 'baz'};
// 上面代码中，等号左边对象的foo属性，对应一个子对象。该子对象的bar属性，解构时会报错。
// 原因很简单，因为foo这时等于undefined，再取子属性就会报错，请看下面的代码。
```
如果你想给已经定义过的变量进行解构赋值，那么就要小心{}引发的错误。

```
// 错误的写法, 此时JavaScript引擎会将{x}理解成一个代码块从而报错。
let x;
{x} = {x: 1};

正确的写法： 
let x;
({x} = {x: 1});
```
因为数组也是对象，所以就可以有以下花式写法： 

```
let arr = [1,2,3]
let {length} = arr
length // 3

let arr = [1, 2, 3];
let {0 : first, [arr.length - 1] : last} = arr;
first // 1
last // 3
```
## 3.字符串的解构赋值
字符串的解构赋值赋值中字符串会被转换成一个类数组的对象。

```
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"

const {0: a, 1: b} = 'hello';
a // "h"
b // "e"
```
## 4.数值和布尔值的解构赋值
跟字符串一样如果等号右边是数值和布尔值，则会先转为对象。

```
// 跟字符串一样如果等号右边是数值和布尔值，则会先转为对象。
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true
```
## 5. 函数参数的解构赋值
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

