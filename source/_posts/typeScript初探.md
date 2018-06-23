---
title: typeScript初探
date: 2018-06-23 12:05:24
tags: TypeScript
---

```
// 变量定义  undefined和null是所有类型的子类

// 定义一个Number
let num: number = 4;
console.log(num);

// 定义一个String
let str: string = '5';
console.log(str);

// String中加变量可以用 ``
let strN: string = `houmianjiage${num}dadfdada`;
console.log(strN);

// 定义一个元素是Number的数组
let arr: Array<number> = [1, 2, 3];
console.log('arr', arr);

// 定义一个元素是String的数组
let arr1: string[] = ['1', '545'];
console.log(arr1);

// 元组 数组中的每一项类型可以是不一样的  但是要注意如下前三个的类型和顺序必须和定义是相同 之后的元素的类型要在前三个类型的范围内
let arr2: [number, string, number] = [1, 'dsds', num, 'dsds', 1, 1, 1];
console.log(arr2); // [ 1, 'dsds', 4, 'dsds', 1, 1, 1 ]

// enum定义枚举    
enum Color { a = 100, b, c }
console.log(Color) // { '100': 'a', '101': 'b', '102': 'c', a: 100, b: 101, c: 102 }
console.log(Color[100]) // a
console.log(Color.a) // 100

// 定义一个any类型  any类型表示可以是任何类型
let anys: any = 111;
console.log(anys) // 111

anys = 'string';
console.log(anys) // '111'

anys = false;
console.log(anys) // false

// void 空类型  function无返回值时可以使用此类型。
// let voids: void = 1 报错
let voids: void
console.log(voids) // undefined

voids = undefined
console.log(voids) // undefined

voids = null
console.log(voids) // null

//       never类型表示的是那些永不存在的值的类型。
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
  throw new Error(message);
}

// 推断的返回值类型为never
function fail() {
  return error("Something failed");
}

// 返回never的函数必须存在无法达到的终点
function infiniteLoop(): never {
  while (true) {
  }
}


/*
*关于void与never的区别 
* void的值是空值(null或者 undefined)
* never表示该函数没有尽头(报错或者死循环) 
*/
```
