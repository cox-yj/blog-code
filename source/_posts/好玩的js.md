---
title: 好玩的js
date: 2018-06-23 12:06:20
tags: JavaScript
---
#### 1、&& 和 ||
```
console.log(1&&2)的结果是2
// 只要“&&”前面是false，无论“&&”后面是true还是false，结果都将返“&&”前面的值;
// 只要“&&”前面是true，无论“&&”后面是true还是false，结果都将返“&&”后面的值;

console.log(0||1)的结果是1
//只要“||”前面为false,不管“||”后面是true还是false，都返回“||”后面的值。
//只要“||”前面为true,不管“||”后面是true还是false，都返回“||”前面的值。
// ||短路表达式
var foo = a || b;
// 相当于
if(a){
    foo = a;
}else{
    foo = b;
}
 
// &&短路表达式
var bar = a && b;
// 相当于
if(a){
    bar = b;
}else{
    bar = a;
}
```
1

#### 2.讲述你对reflow和repaint的理解 (待学习)
#### 3.Array.prototype.filter()

```
function isBigEnough(value) {
  return value >= 10;
}

var filtered = [12, 5, 8, 130, 44].filter(isBigEnough);

// filtered is [12, 130, 44]

// 以下为ES6的写法

// 箭头函数
const isBigEnough = value => value >= 10;

// 等同于：let spraed = 浅克隆([12, 5, 8, 130, 44])
let [...spraed]= [12, 5, 8, 130, 44];

let filtered = spraed.filter(isBigEnough);

// filtered is [12, 130, 44]
```
#### 3.原型
首先js中原型比较绕，主要原因在于向java或c++这样的语言中类只是一个概念，但在js中类不光是一个概念它是有实体的(构造函数)，它本身也是一个对象
```
function objs(x){
　　this.x=x;
}
var obj=new objs(1);
```

#### 4.arguments
**arguments**对象是所有（非箭头）函数中都可用的局部变量。你可以使用**arguments**对象在函数中引用函数的参数。此对象包含传递给函数的每个参数的条目，第一个条目的索引从0开始

```
function fn(){
    console.log(arguments)
}

fn(1,2,3)

 // 结果 [1, 2, 3, callee: ƒ, length: 3, Symbol(Symbol.iterator): ƒ]
arguments.callee
指向当前执行的函数。
arguments.caller 
指向调用当前函数的函数。
arguments.length
指向传递给当前函数的参数数量。
```
#### 5. 转换为number a = +a 类型的

#### 6. a>=b不等价于!(b<a) --引自《你不知道的js》91页的错误
反例：
是它的解释不能适用于NaN的情况，只要有NaN，怎么比都是false，而按照它的解释就不是了

```
var a
a = parseInt(a)
console.log(a) //NaN
var b = 1,
// 这时候a和b怎么比都是false
a > b 

```
   只要比较中有不等于号就会发生类型转换，两边都是[object Object]，所以应该相等。这样a<b和a>b就是false。而>=是大于或等于，满足其中等于就得到true，<=同理。但是 == 两边都是对象，类型一致不转换，而两边对象引用不一样结果就是false

---
#### 7.判断是数组还是对象

```
Object.prototype.toString.call([]) // "[object Array]"
Object.prototype.toString.call({}) // "[object Object]"
```
#### 8.判断当前字符串是不是纯数字 

```
var a = '122'
+a+0 === +a  // true
value % value === 0 // true
value % value + 1 // true

var a = '12a2'
+a+0 === +a  // false
value % value === 0 // false
value % value + 1 // false



```

