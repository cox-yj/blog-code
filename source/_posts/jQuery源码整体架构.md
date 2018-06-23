---
title: jQuery源码整体架构
date: 2018-06-23 11:52:45
tags: jQ
---
# 1.沙箱
#####  jQuery中沙箱？
1. 用一个函数域包起来，就是所谓的沙箱在这里边 var 定义的变量，属于这个函数域内的局部变量，避免污染全局

2. 把当前沙箱需要的外部变量通过函数参数引入进来只要保证参数对内提供的接口的一致性，你还可以随意替换传进来的这个参数

##### jQuery源码： 

```
(function (window, undefined) {
})(window);
```


##### 在生产环境中：

在生产环境中，我们为了减少代码包的大小会压缩代码，而jQuery中压缩后是：

```
// w -> windwow , u -> undefined
(function(w, u) {
})(window);
```

由于undefined已经在一参数的形式传入了在代码中所有用到undefined的地方都会被u代替，可以在一定程度上减少代码量，
从而减少丫说版本的体积。

##### 总结：
1.  jQuery的所有代码都是包在这样一个自执行的匿名函数中，作用是为了防止变量污染，最终只将jQuery和$挂在在window上。

1.  匿名函数接收两个参数window和undefined，而传参只有window，这里是为了保证undefined是undefined，因为在早期的环境中undefined是可以赋值的，这里是为了防止某些二货给undefined赋值造成的未知报错。

1. 压缩代码后更大程度上减少代码量。


# 常用方法保存
##### 技术准备：
###### 1. call和apply:
　　
- 他两的作用都是一样的，改变this的指向，第一个参数都是this要改变的指向，从第二个参数起, call方法参数将依次传递给借用的方法作参数, 而apply直接将这些参数放到一个数组中再传递, 最后借用方法的参数列表是一样的.<br />

- call, apply都属于Function.prototype的一个方法,它是JavaScript引擎内在实现的,因为属于Function.prototype,所以每个Function对象实例,也就是每个方法都有call, apply属性.既然作为方法的属性,那它们的使用就当然是针对方法的了.这两个方法是容易混淆的,因为它们的作用一样,只是使用方式不同.

图片－－－－－－－－－－－－－－－
###### ２．类数组的定义：
- 拥有length属性，其它属性（索引）为非负整数(对象中的索引会被当做字符串来处理，这里你可以当做是个非负整数串来理解)
- 不具有数组所具有的方法


##### jQuery源码：

```
class2type = {},
core_deletedIds = [],
core_version = "1.9.0",
// 保存一写常用方法
core_concat = core_deletedIds.concat,
core_push = core_deletedIds.push,
core_slice = core_deletedIds.slice,
core_indexOf = core_deletedIds.indexOf,
core_toString = class2type.toString,
core_hasOwn = class2type.hasOwnProperty,
core_trim = core_version.trim,
```
##### 总结：
1. 类数组也可使用数组的方法。
1. 减少计算步骤，提高性能；
<br />例：<br />
    　　调用数组实例的concat时：<br />
	　　　1.先要判断实例的类型是不是Array，<br />	
	　　　2.其次在内存空间中找到Array的concat入口，<br />
	　　　3.把当前的实例指针和参数加入栈，<br />
	　　　4.然后跳转到concat地址开始执行；<br />
    　　而保存了concat方法后，可以省去前两步

# jQuery 无 new 构造
##### 技术准备：

1. $或者jQuery: 构造函数，我们叫它--类

1. $(“app”):  正经的对象，它是$的实例。

1. 在JavaScript中类也是一种特殊的对象。

1. 类原型上的属性实例可以直接使用

1. 类本身的属性实例是不能接直接调用的


##### 使用：
图片---------------------------------

##### 源码：

```
1.
jQuery = function (selector, context) {
	// The jQuery object is actually just the init constructor 'enhanced'
	return new jQuery.fn.init(selector, context, rootjQuery);
},

2.
jQuery.fn = jQuery.prototype = {
	// 版本
	jquery: core_version,

	constructor: jQuery,
	init: function (selector, context, rootjQuery) {
	...
	}
}

3.
jQuery.fn.init.prototype = jQuery.fn
```

##### 总结：
1. 根据以上的代码$('#app')实际jQuery()是构造了jQuery.fn.init对象

2. 通过代码中的第二段我们能够知道jQuery.fn = jQuery.prototype，在第三段中我们把jQuery的原型传递给了jQuery.fn.init

3. 在jQuery.fn.init的this仍然指向jQuery.fn

4. 那么new jQuery.fn.init 实际就等于 new jQuery

5. 从而调用jQuery()方法 return new jQuery.fn.init，$('#app')等价于new $('#app')，因此实现无new 构造jQuery对象

# jQuery.fn.extend 与  jQuery.extend

##### 使用：

图片---------------------------------

##### 源码


```
// 方法定义
jQuery.extend = jQuery.fn.extend = function () {
    ...
}

// 给jQuery类添加属性
jQuery.extend({
    ...
})

// 给jQuery类的原型添加属性
jQuery.fn.extend({
    ...
})

```



##### 总结
1.在函数中this的指向：函数挂载在那个对象上就指向谁。

2.jQuery.fn.extend 与  jQuery.extend在实现时共用了一个方法， 同一个方法却实现了不同的功能，这主要归功于JavaScript中this的强大了。
 
3.前面说过，jQuery.fn = jQuery.prototype添加方法， jQuery.fn.extend 是给jQuery类的原型上去添加方法，也就是jQuery实例对象上添加方法。

4.jQuery.extend是给jQuery这个类去添加方法。

# jQuery 的链式调用及回溯

##### 使用

```
<body>
  <div class="a">1</div>
  <div>2</div>
  <div>3</div>
</body>
<script>
    $('div').eq(0).show().end().eq(1).hide();
</script>
```
图片-------------------------
##### 源码

```
jQuery.fn = jQuery.prototype = {
	...
	eq: function (i) {
		var len = this.length,
			j = +i + (i < 0 ? len : 0);
		return this.pushStack(j >= 0 && j < len ? [this[j]] : []);
	},

	end: function () {
		return this.prevObject || this.constructor(null);
	},
	...
};
```

##### 总结
1.实现链式调用其实就是在调用的方法中返回this就可以了

2.回溯的实现主要归功于jQuery实例中有prevObject 属性，它记录了调用前的对象。

3.prevObject是由pushStack() 方法生成，他将一个DOM集合放入了jQuery内部管
理的一个栈中，就是prevObject

4.当我们调用end()方法时，直接返回当前对象的prevObject就可以了。
