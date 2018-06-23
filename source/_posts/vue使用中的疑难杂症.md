---
title: vue使用中的疑难杂症
date: 2018-06-23 12:08:37
tags: vue
---

#### 1.当用view中的数据对store中某一个数据进行修改时需要深度克隆
```
// 当在view对store中某一个数据进行修改时，不能写成下面这种情况
this.$store.commit('MUTATIONS', this.data)
// 应当写成下面这种情况 必须对this.data进行深度克隆
this.$store.commit('MUTATIONS', cloneDeep(this.data))
```
**原因**：当写成第一种情况的时候，view中的变量和store中的变量都是指向一个对象的，而store中的数据只能通过mutations同步修改，但在view中却可以随时修改从而产生冲突。

---
#### 2.当用view中需要对store中的某一个值(以下称为dataX)进行修改时，如果dataX中的数据挺大时当分为多个mutations。
```
// 不能写成这样(不能说错了，只能说不推荐)：
this.$store.commit('MUTATIONS', cloneDeep(this.data))
// 应当写成下面这种情况 
this.$store.commit('MUTATIONS_NAME', cloneDeep(this.data.name))
this.$store.commit('MUTATIONS_AGE', cloneDeep(this.data.age))
...
```
**原因**：和Vuex的设计理念不符(具体怎么不符有待进一步的研究)


---

#### 3.actions在调用时只能传一个参数
引用文档：

```
在 store 上注册 action。处理函数总是接受 context(commit, state) 作为第一个参数，payload 作为第二个参数（可选）。
```
###### 根据文档，当引用action的时候**只能传一个参数**

#### 4.组件角色
在vue中组件作用在于解耦合，分为功能组件和容器组件，不同种类的组件的功能也是不一样的：
1. 功能组件：
    <br/>不予外界发生交互(调接口、本地存储和vuex)，所需数据从props中获取，抛出数据通过emit；实现某种功能,可复用，比如说 input组件或 包括input和select的筛选组件
2. 容器性组件：
    处理子组件抛出的方法或值，与外界交互

---
误区：
1. 功能组件中也可以包含其他功能组件，不能说包含其他的组件的组件就是容器组件，比如我上边说到的包括input组件和select组件的筛选组件
2. 


#### 5.npm install 后面的参数
那 package.json 文件里面的 **devDependencies**  和 **dependencies** 对象有什么区别呢？

在 package.json 文件里面提现出来的区别就是，使用 --save-dev 安装的 插件，被写入到 devDependencies 对象里面去，而使用 --save 安装的插件，责被写入到 dependencies 对象里面去。


**devDependencies**：里面的插件只用于开发环境
```
npm install XX --save-dev
```


**dependencies**：里面的插件只用于生产环境
```
npm install XX --save
```
#### 6.刷新与vue-route冲突导致刷新无效
我创建和编辑的页面使用的是同一个component,默认情况下当这两个页面切换时并不会触发vue的created或者mounted钩子，官方说你可以通过watch $route的变化来做处理，但其实说真的还是蛮麻烦的。后来发现其实可以简单的在 router-view上加上一个唯一的key，来保证路由切换时都会重新渲染触发钩子了。这样简单的多了。
###### 解决：
++对于嵌套了多个子组件，这个方法一劳永逸++
```
<router-view :key="key"></router-view>

computed: {
    key() {
        return this.$route.name !== undefined? this.$route.name + +new Date(): this.$route + +new Date()
    }
 }
```
