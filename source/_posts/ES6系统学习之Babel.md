---
title: ES6系统学习之Babel
date: 2018-06-23 12:00:18
tags: ES6
---
# 1.Babel转码器
### 为什么要使用Babel?
在当代环境中虽然对ES6的支持越来越好，但是大多数还是不支持ES6，这时候我们就需要把已经写好的ES6代码转换成ES5，而Babel就是将ES6转化成ES5的转码器，被广泛使用。
举个栗子：

```
// 转码前
input.map(item => item + 1);

// 转码后
input.map(function (item) {
  return item + 1;
});
```
### 配置文件 .babelrc
.babelrc是babel的配置文件，放在项目的根目录下，比如现有的项目：

```
{
  // presets 是用来设置转码规整的，es2015就是ES6,
  // stage-2其实是一个系列，它是ES7的编码规则，有stage-0[,1,2,3], 四选一
  "presets": ["es2015", "stage-2"],
  "plugins": ["lodash", "transform-runtime"],
  "comments": false
}
```
基本用法如下。

```
# 转码结果输出到标准输出
$ babel example.js

# 转码结果写入一个文件
# --out-file 或 -o 参数指定输出文件
$ babel example.js --out-file compiled.js
# 或者
$ babel example.js -o compiled.js

# 整个目录转码
# --out-dir 或 -d 参数指定输出目录
$ babel src --out-dir lib
# 或者
$ babel src -d lib

# -s 参数生成source map文件
$ babel src -d lib -s
```

你也可以将babel-cli安装到项目中去：

```
$ npm install --save-dev babel-cli
```
然后，改写package.json。

```
{
  // ...
  "devDependencies": {
    "babel-cli": "^6.0.0"
  },
  // 这里build只是随便启的名字，运行是质询要npm run build就行了
  "scripts": {
    "build": "babel src -d lib"
  },
}
```
### babel-polyfill

Babel默认只转换新的JavaScript句法（syntax），而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如Object.assign）都不会转码。
举例来说，ES6在Array对象上新增了Array.from方法。Babel就不会转码这个方法。如果想让这个方法运行，必须使用babel-polyfill，为当前环境提供一个垫片。
安装命令如下。

```
$ npm install --save babel-polyfill
然后，在脚本头部，加入如下一行代码。
import 'babel-polyfill';
// 或者
require('babel-polyfill');
```
### 浏览器环境

当你不想做这些操作，或者现有的项目没有webpack等构架工具时，这一步转码过程可以放在浏览器中，只不过从Babel 6.0开始，不再直接提供浏览器版本，而是要用构建工具构建出来。如果你没有或不想使用构建工具，可以通过安装5.x版：
```
$ npm install babel-core@5
```

运行上面的命令以后，就可以在当前目录的node_modules/babel-core/子目录里面，找到babel的浏览器版本browser.js（未精简）和browser.min.js（已精简）。
然后，将下面的代码插入网页。

```
<script src="node_modules/babel-core/browser.js"></script>
<script type="text/babel">
// Your ES6 code
</script>
```

上面代码中，browser.js是Babel提供的转换器脚本，可以在浏览器运行。用户的ES6脚本放在script标签之中，但是要注明**type="text/babel"。**

另一种方法是使用babel-standalone模块提供的浏览器版本，将其插入网页。

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.4.4/babel.min.js"></script>
<script type="text/babel">
// Your ES6 code
</script>
```

但是要注意的是，如果这样做浏览器会是是转换ES6代码，会使性能下降。
