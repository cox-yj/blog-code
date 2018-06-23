---
title: vue项目webPack多页面配置
date: 2018-06-23 12:07:46
tags: vue
---
<!-- TITLE: Webpack -->
<!-- SUBTITLE: A quick summary of Webpack -->
​
# 什么是webpack
WebPack可以看做是**模块打包机**：它做的事情是，分析你的项目结构，找到JavaScript模块以及其它的一些浏览器不能直接运行的拓展语言（Scss，TypeScript等），并将其转换和打包为合适的格式供浏览器使用。
​
# 为什要使用WebPack
- 现今的很多网页其实可以看做是功能丰富的应用，它们拥有着复杂的JavaScript代码和一大堆依赖包。
- 模块化，我们可以将各功能拆分成小模块来实现。
- 扩展语言，使我们能够实现目前版本的JavaScript不能直接使用的特性，并且之后还能转换为JavaScript文件使浏览器可以识别。
​
# 开始使用webpack
- 安装nodejs
- npm i -g webpack全局安装
- npm init创建package.json文件（如果没有）
- npm i -D webpack安装到项目目录
- 创建webpack.config.js配置文件
- 使用webpack命令行即可自动使用webpack.config.js配置打包
- 或者使用webpack --config [path]来指定配置文件
​
# 配置文件详解
## entry: String|Object
主文件入口
​
## output: Object
输出配置对象有以下属性
### path: String
生成文件目录
### filename: String
生成文件名称
### libraryTarget: String
生成文件标准，如果只是简单的被html引用，不需要添加此属性。
如果是打包库需要被其他项目import， 改为umd为生成全兼容方案
### library: String
为require('xxx')的名称
## devtool: String
添加此属性值source-map可以生成方便调试的sourcemap文件，生产环境去掉
​
## resolve: Object
文件处理对象，有以下属性
### alias: Object
别名属性，可以为路径指定别名，如 components: './src/components'
### extensions: Array[String]
尝试处理的文件名后缀，比如配置了['.js', '.vue']在import的时候只写common，  编译器会尝试寻找common.js和common.vue
## module: Object
- 通过使用不同的loader，webpack有能力调用外部的脚本或工具，实现对不同格式的文件的处理，比如说分析转换scss为css，或者把下一代的JS文件（ES6，ES7)转换为现代浏览器兼容的JS文件。
- loader需要单独安装，并在webpack配置文件中的**module对象中的rules属性**配置，一个loader配置包含以下属性
### test: Reg
用来匹配文件名的正则表达式
### use: String|Array[String]|Object|Array[Object]
使用的loader名称，或loader对象，或一组多个loader名称或对象
### include: Reg
手动添加必须处理的文件（可选）
### exclude: Reg
手动屏蔽不处理的文件（可选）
## plugins: Array
loaders为在每个文件上执行的转换，而plugins为在整个打包过程里执行操作和自定义功能。有了插件，能在webpack中添加极其强大的定制化。
使用：在配置文件头里require()插件对象，将他添加到plugins数组里。使用option来配置插件，或者使用new来实例化插件对象。
​
# 常用loaders配置
## css-loader
- npm i -D style-loader css-loader安装
```js
module: {
  rules:[
    {
      test: /.css$/,
      use: ['style-loader', 'css-loader']
    }
  ]
}
​
```
## vue-loader
- npm i –D vue-loader vue-style-loader vue-template-complier安装
```js
{
  test: /.vue$/,
  use: 'vue-loader' 
}
​
```
## babel-loader
- 通过babel，可以将es6、es7等浏览器目前还不支持的新语法编译成浏览器可以执行的旧语法。
- npm i –D babel-core babel-loader babel-preset-es2015 babel-preset-stage-0安装
```js
{
  test: /.js$/,
  use: 'babel-loader',
  exclude: /node_modules/ //不编译依赖包
}
​
```
项目目录下创建babel的配置文件：
新建文件 改名为 .babelrc. 即可创建.babelrc配置文件
```json
{
  "presets": ["es2015", "stage-0"]
}
​
```
## url-loader
- url-loader可以将打包中的其他图片文件等输出到打包目录，并且可以设定文件大小小于设置值时转换成base64编码串来减少文件数量。
- npm i –D url-loader安装
```javascript
{
  test: /\.(png|jpg|gif|svg)$/,
  loader: 'url-loader?limit=8192&name=images/[hash:8].[name].[ext]'
  // 8192为设置的文件大小
}
```
# 常用plugins配置
## html-webpack-plugin
这个插件依据一个简单的html模版，自动生成一个新的模版，向里面添加了编译后的bundle文件，在每次生成的bundle文件名都不一样（比如添加了hash值）时非常有用
- 使用npm i –D html-webpack-plugin安装
- webpack.config.js中配置
```js
const HtmlWebpackPlugin = require('html-webpack-plugin')
plugins: [
  new HtmlWebpackPlugin({
    template: __dirname + '/index.html'// 模版文件路径
  })
]
```
## HotModuleReplacementPlugin
这个插件可以在修改组件后自动重载页面，在生产环境勿使用
```js
const webpack = require('webpack')
plugins: [
  new webpack.HotModuleReplacementPlugin()
]
```
​
# 多入口webpack
除了SPA，也有多页面应用需要用到webpack打包。
简单思路如下，
- 打包每个页面的index.js
- 单独打包公共库如vue，jquery
- 单独打包不常改动的公共组件
## 配置entry
entry除了上面的一个单路径之外，还可以接受一个对象，名字对应路径。
```js
entry: {
  home: __dirname+'/src/apps/home/index.js',
  map: __dirname+'/src/apps/map/index.js'
}
​
```
但是这样的话每增加一个页面都得手动去添加入口，下面我们借助glob来匹配入口文件
### 使用glob
npm i –D glob安装
配置文件里
```js
const glob = require(‘glob’)
const files = glob.sync(__dirname + '/src/apps/*/index.js')// 即可返回所有的匹配路径的入口文件
files.forEach(filepath => {
  const path = filepath.replace('/index.js', '')
  const name = path.slice(path.lastIndexOf('/') + 1)
  entry[name] = filepath
})
​
```
## 打包三方库
在entry中添加一个入口对应数组即可
如
```js
entry: {
  vendor: ['vue', 'jquery']
}
​
```
这样就会生成一个vendor.js文件为打包的所有库
还需要在plugins里添加一个对象，使得代码中import这些库的地方不再打包库进去
```js
plugins:[
  new webpack.optimize.CommonsChunksPlugin('vendor')
]
```
## 打包公共组件
同样的，我们使用glob来匹配组件，将他们打包成一个文件
```js
const comps = glob.sync(__dirname + '/src/components/*/*.vue')
​
entry: {
  components: comps
}
```
加入之前那个插件对象中
```js
plugins: [
  new webpack.optimize.CommonsChunkPlugin({
    names: ['vendor', 'components']
  })
]
```
## 生成功能的html页面
在之前的files.forEach中加入
```js
files.forEach(filepath => {
  const path = filepath.replace('/index.js', '')
  const name = path.slice(path.lastIndexOf('/') + 1)
  entry[name] = filepath
​
  const plug = new HtmlWebpackPlugin({
    filename: __dirname + '/dist/' + name + '.html',
    template: __dirname + '/src/apps/index.html',
    chunks: [name, 'vendor', 'components'],
    inject: true
  })
  config.plugins.push(plug)
})
​
```
## 生成每个页面对应的目录
如之前的配置，entry中的name的值如果包含路径，就可以实现。
如原map，改为map/index，则会生成map文件夹
配置中改为const name = path.slice(path.lastIndexOf('/') + 1) + '/index‘
​
## 多入口打包示例配置
```js
const glob = require('glob')
const webpack = require('webpack')
const HtmlWebpackPlugin = require('html-webpack-plugin')
​
const files = glob.sync(__dirname + '/src/apps/*/index.js')
const comps = glob.sync(__dirname + '/src/components/*/*.vue')
​
const entry = {
  vendor: ['vue', 'jquery']，
  components: comps
}
​
const config = {
  entry,
  output: {
    path: __dirname + '/dist',
    filename: '[name].js'
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({ names: ['vendor', 'components'] })
  ]
}
​
files.forEach(filepath => {
  const path = filepath.replace('/index.js', '')
  const name = path.slice(path.lastIndexOf('/') + 1) + '/index'
  entry[name] = filepath
​
  const plug = new HtmlWebpackPlugin({
    filename: __dirname + '/dist/' + name + '.html',
    template: __dirname + '/src/apps/index.html',
    chunks: [name, 'vendor', 'components'],
    inject: true
  })
  config.plugins.push(plug)
})
​
module.exports = config
​
```
Powered by Wiki.js.
首页
滚到顶部