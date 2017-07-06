---
title: Webpack Development Server 开发服务器详解
categories:
  - 前端工程
  - Webpack
tags:
  - Webpack Server
  - Javascript
abbrlink: hJN-n3Pj0BAbWkD90DFlBg
date: 2016-09-03 15:39:19
---
![webpack](http://7xk7wj.com1.z0.glb.clouddn.com/blog_webpack.png)
曾经前端工程化的主流方案是使用grunt、gulp作为构建优化工具，使用requirejs、seajs、browserify作为模块化开发工具，构建工具与模块工具相互配合，实现了前后端分离开发的一种工程化解决方案。现如今只需要webpack就可以实现前端工程化所需的大多数功能（和npm script配合使用效果更佳），小而美**VS**大而全。

本文适合对模块化、前后端分离有一定了解的读者，最好懂得一些webpack、node、express的基础相关知识。需要深入了解webpack配置方法的读者可以进入文章末尾链接继续学习。
<!-- more -->
## 为什么需要Webpack开发服务器
### 实现前后端分离
前后端分离的概念已经出现很久了，曾经的主流做法是，前端代码和后端代码在同一个代码仓库，把前端代码所在目录设置为静态资源根目录，前端开发时为了在本地启动后端服务，还要搭建后端服务环境，还不得不去写后端模板代码。英国都脱欧了，前端也要脱离后端的魔爪掌控！

向往自由的前端程序员在nodejs出现后，连夜搭起了node服务，让数据请求通过node代理到后端服务，此方案大大降低了对后端服务的依赖，而且构建自由度也大大提高，没（shuai）有（diao）束（hou）缚（duan）的感觉真好！那么如何搭起这踹掉后端的node服务呢？最简单快捷的现成方法是使用gulp+browser-sync，当然，还可以根据自己狂拽酷炫吊炸天的需求，定制一套webpack-dev-server开发服务器（实现方法就在下下下面）。
### 热加载
关于热加载，网上已经有很多文章来介绍了，这里就说一下为什么因为热加载而使用Webpack开发服务器。
去年在一个项目中使用了browserify作为模块化打包工具，发现网上大多是单页面应用one bundle的方案，但我想实现多页面more bundles，很辛苦的看文档，才照猫画虎实现了这个功能（计划日后专门写一篇关于browserify more bundles的文章），但使用中发现页面多了以后，保存马上刷新页面有时看不到想要的效果，原因是页面引用的bundle需要经过模块化打包编译生成文件，这个过程中所用的时间导致不能马上刷新看到效果。
而webpack的热加载功能，不需要编译成文件后调用，这个步骤是在内存中完成，所以不存在上述情况。

## 实现开发服务器的三种方法
### 1. 命令行、config配置方式
使用命令行启动webpack development server是最简单的方式，只需要在cmd中输入`webpack-dev-server --hot --inline`，OK 完事收工。

使用webpack.config.js 配置的方式，需要注意output.publicPath，官方文档中有这样一句话：
> It’s important to specify a correct output.publicPath otherwise the hot update chunks cannot be loaded.
> 指定一个正确的output.publicPath否则热更新块无法加载这一点很重要。

**注意：**
output.publicPath的参数值是页面运行时bundle模块的访问路径，但它是在内存中的，实际并没有编译成文件。

配置好后来一条命令`webpack-dev-server` 搞定收工。
```javascript
//webpack.config.js 
var webpack = require('webpack');
module.exports = {
  entry: [
    'webpack/hot/dev-server',
    'webpack-dev-server/client?http://localhost:8080',
    './app/main.js'
  ],
  output: {
    filename: 'bundle.js', //输出的模块名
    publicPath: '/build/' //注意点
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ]
};
```
```
<!--index.html-->
<html>
  <body>
    <div id='root'></div>
    <script src="/build/bundle.js"></script> <!--这里的"/build/"就是publicPath-->
  </body>
</html>
```
### 2. webpack-dev-server 
`webpack-dev-server` 是一个静态资源WEB服务器，只用于开发环境，它会把编译后的静态文件全部保存在内存里，而不会写入到文件目录内，这样做就不用改变webpack输出目录。要启动 `webpack-dev-server` 需要创建一个`server.js`，并在命令行中输入`node server`来运行，代码如下：
```javascript
var config = require("./webpack.config.js");
var webpack = require('webpack');
var WebpackDevServer = require('webpack-dev-server');

config.entry.app.unshift("webpack-dev-server/client?http://localhost:8080/", "webpack/hot/dev-server"); //注意点
config.plugins.push(new webpack.HotModuleReplacementPlugin());
var compiler = webpack(config);
var server = new WebpackDevServer(compiler,{
  publicPath: config.output.publicPath, //这里必须有，不然404找不到脚本
  hot: true,
  stats: {colors: true} //命令行中增加颜色
});
server.listen(8080);
```

**注意1：**
可能会遇到config.entry.unshift数据类型错误的提示，这是因为webpack.config.js文件中entry的参数是字符串，而不是数组。可以通过如下方法解决：
```javascript
for (var i in config.entry) {
  if(typeof config.entry[i] === 'string'){
    config.entry[i]= ['webpack/hot/dev-server', 'webpack-dev-server/client?http://localhost:8080' , config.entry[i]];
  }else{
    config.entry[i].unshift('webpack/hot/dev-server', 'webpack-dev-server/client?http://localhost:8080');
  }
}
```
**注意2：**
Webpack热加载功能只对有`module.exports`封装的模块起作用，否则会导致热加载失败从而刷新浏览器，而对于入口js文件由于没有模块输出，就会发现总是刷新浏览器了。如果要禁止自动刷新浏览器，可以将server.js中的`webpack/hot/dev-server`改为`webpack/hot/only-dev-server`。

对webpack.config.js 进行如下修改：
```javascript
var webpack = require('webpack');
var path = require("path");
module.exports = {
  entry: {
    app: ["./app/main.js"]
  },
  output: {
    path: path.resolve(__dirname, "build"), //打包编译后文件存放的绝对路径，没有这个参数会报错
    publicPath: "/build/",
    filename: "bundle.js"
  },
  plugins:[
    //...
  ]
};
```
最后再加入最重要的proxy参数来转发服务器请求，前后端分离的大业就完成了。如果没有proxy，本地开发服务器对后端RESTful api的请求就会跨域错误，对server.js的修改如下：
```javascript
var server = new WebpackDevServer(compiler,{
  publicPath: config.output.publicPath,
  hot: true,
  stats: {colors: true}, //命令行种增加颜色
  //转发api接口请求
  proxy: {
    '/api/*': 'http://127.0.0.1:8000'
  }
});
```
业务脚本index.js将本地对后端RESTful api的请求转发至`http://127.0.0.1:8000`，代码如下：
```javascript
$.ajax({
  url: , //对api接口的请求会转发到http://127.0.0.1:8000服务器下
  type: 'post',
  dataType: 'json',
  data: {//***}
})
```
### 3. express中间件
> The webpack-dev-server is a little node.js Express server.

官方文档告诉我们`webpack-dev-server`实际上是使用Express实现的一个服务，那么如果你想高度定制化，就可以用`webpack-dev-middleware`和`webpack-hot-middleware`中间件来自己创建一个express服务，这样就可以实现比如页面自动刷新（热加载只在模块修改后重新加载）、router路由，模板引擎等等功能。基本用法如下：

```javascript
var app = express(),
    routes = require('./routes'),
    webpackDevMiddleware = require('webpack-dev-middleware'),
    webpackHotMiddleware = require('webpack-hot-middleware'),
    webpack = require('webpack'),
    webpackConf = require("./webpack.config.js"),
    compiler = webpack(webpackConf);
//router路由，做一些你想做的事情
app.use('/', routes);
app.use(webpackDevMiddleware(compiler, {
  publicPath: config.output.publicPath,
  hot: true,
  proxy: {
    '/w/start/*': 'http://127.0.0.1:8000'
  },
  stats: {colors: true}
});
app.use(webpackHotMiddleware(compiler));
```
`webpack-hot-middleware`是Github上的一个开源项目，专门用来实现`webpack-dev-middleware`中间件热加载功能的。

文章结尾，分享我学习webpack时候看到的对我有帮助的文章，未来会继续深入研究基于express，探索更多基于express中间件的功能。

* [webpack傻瓜式指南](https://vikingmute.gitbooks.io/webpack-for-fools/content/)
* [webpack模块编写](https://diamont1001.github.io/webpack-summary/)
* [webpack使用优化](http://www.alloyteam.com/2016/01/webpack-use-optimization/)
* [基于webpack搭建前端工程解决方案探索](https://segmentfault.com/a/1190000003499526#articleHeader14)
* [webpack多页面工程化](https://github.com/vhtml/webpack-MultiPage-static)
* [webpack-dev-middleware 原理](http://andyyou.github.io/webpack,/js/2016/05/30/webpack-dev-middleware-in-express.html)