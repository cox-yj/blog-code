---
title: ES6系统学习之Set 和 Map 数据结构
date: 2018-06-23 12:01:10
tags: ES6
---
## 1. set
set是ES6中的一种新的数据结构，类似于数组结构但是他其中的元素都是唯一的。

### 1.Set是一个构造函数，用来创建Set类型的数据。

```
const s = new Set()

// 使用Set的add方法给s添加成员，重复的值不会再次添加
[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));


for (let i of s) {
    console.log(i)
}

// 2, 3, 5, 4

// Set函数也可以接受一种数组或者具有iterable接口的其他数据结构作为参数来初始化一个Set。

const set = new Set([1, 2, 3, 4, 4]);
[...set]
// [1, 2, 3, 4]

// 对象
function divs () {
  return [...document.querySelectorAll('div')];
}

const set = new Set(divs());
```

根据Set的特性，可以用来给数组去重。

```
[...new Set(array)]
```

### 2.有些**特殊**的地方需要注意的：

```
// 在加入值时，不会进行类型转换， 在Set内部判断两个值是否相同，
// 使用的算法叫做 **Same-value equality**(后面会详细解释)  
// 它跟 === 的区别是NaN等不等于自身   大家都知道 === 判断中NaN和NaN是不相等的

// 例1
let set = new Set();
let a = 5;
let b = '5';
set.add(a);
set.add(b);
set // Set(2) {5, "5"}


// 例2
let set = new Set();
let a = NaN;
let b = NaN;
set.add(a);
set.add(b);
set // Set {NaN}

// 两个对象是不相等的
let set = new Set();

set.add({});

set.add({});

set // Set(2) {{…}, {…}}

```
### 3.Set 的属性和方法
#### 1.先来对数组的增删改查
- add(value)：添加某个值，返回 Set 结构本身。
- delete(value)：删除某个值，返回一个布尔值，表示删除是否成功。
- clear()：清除所有成员，没有返回值。
- has(value)：返回一个布尔值，表示该值是否为Set的成员。
- Set.prototype.size：返回Set实例的成员总数。


```
s.add(1).add(2).add(2);
// 注意2被加入了两次

s.size // 2

s.has(1) // true
s.has(2) // true
s.has(3) // false

s.delete(2);
s.has(2) // false
```
####  2.遍历操作
- keys()：返回键名的遍历器
- values()：返回键值的遍历器
- entries()：返回键值对的遍历器
- forEach()：使用回调函数遍历每个成员

```
let set = new Set(['red', 'green', 'blue']);

set.keys // SetIterator {"red", "green", "blue"}
for (let item of set.keys()) {
  console.log(item);
}
// red
// green
// blue

set.values() // SetIterator {"red", "green", "blue"}
for (let item of set.values()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.entries()) {
  console.log(item);
}
// ["red", "red"]
// ["green", "green"]
// ["blue", "blue"]
```
