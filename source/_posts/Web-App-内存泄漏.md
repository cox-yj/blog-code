---
title: Web App 内存泄漏
date: 2018-06-23 12:21:11
tags: vue
---
##  一、写在前面
js中的内存垃圾回收机制：垃圾回收器会定期扫描内存，当某个内存中的值被引用为零时就会将其回收。当前变量已经使用完毕但依然被引用，导致垃圾回收器无法回收这就造成了内存泄漏。传统页面每次跳转都会释放内存，所以并不是特别明显。

Web App 与 传统Web的区别，因为Web   App是单页面应用页面通过路由跳转不会刷新页面，导致内存泄漏不断堆积，导致页面卡顿。

## 二、泄漏点
1.  DOM/BOM 对象泄漏
1. script 中存在对DOM/BOM 对象的引用导致
1. Javascript 对象泄漏
1. 通常由闭包导致，比如事件处理回调，导致DOM对象和脚本中对象双向引用，这个时常见的泄漏原因

## 三、代码关注点
1. DOM中的addEventLisner 函数及派生的事件监听， 比如Jquery 中的on 函数， vue 组件实例的 $on 函数，第三方库中的初始化函数
1. 其它BOM对象的事件监听， 比如websocket 实例的on 函数
1. 避免不必要的函数引用
1. 如果使用render 函数，避免在html标签中绑定DOM/BOM 事件

## 四、如何处理
1. 如果在mounted/created 钩子中绑定了DOM/BOM 对象中的事件，需要在beforeDestroy 中做对应解绑处理

1. 如果在mounted/created 钩子中使用了第三方库初始化，需要在beforeDestroy 中做对应销毁处理

1.  如果组件中使用了定时器，需要在beforeDestroy 中做对应销毁处理

1. 模板中不要使用表达式来绑定到特定的处理函数，这个逻辑应该放在处理函数中？

1. 如果在mounted/created 钩子中使用了$on，需要在beforeDestroy 中做对应解绑($off)处理

1. 某些组件在模板中使用 事件绑定可能会出现泄漏，使用$on 替换模板中的绑定



#### 如何在vue 组件中处理addEventListener
    
    created/mounted 生命期钩子函数中定义事件响应函数为对象实例的方法，使用 => 函数来绑定作用域
       调用 addEventListener 添加事件监听
       beforeDestroy 中调用 removeEventListener 移除对应的事件监听，注意前面定义的响应函数方法需要作为第二个参数传入
       然后用 delete 从对象实例移除定义的响应方法，或者将属性设置为 null/undefined
       
    
#### 为什么不能使用匿名函数或者已有的函数的绑定来直接作为事件监听函数
      这样无法准确移除监听，函数每次绑定返回的是不同的函数实例

具体例子请参考 如下代码

 
```
mounted() {
    const box = document.getElementById('time-line')
    this.width = box.offsetWidth
    this.resizefun = () => {
      this.width = box.offsetWidth
    }
    window.addEventListener('resize', this.resizefun)
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.resizefun)
    this.resizefun = null
  }
```
