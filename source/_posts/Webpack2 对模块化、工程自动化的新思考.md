---
title: Webpack2 对模块化、工程自动化的新思考
categories:
  - 前端工程
  - Webpack
tags:
  - Webpack
  - 模块化
abbrlink: LjQz9z4CQLQnrlh0VNKBJA
date: 2017-07-13 16:08:58
---
> 使用webpack差不多快一年了，已经稳定运行在多个生产项目中，趁着最近webpack官方版本更新，再整理一下自己模块化，工程化的相关知识。本文将介绍webpack分文件写法，以及html、ejs模板的使用方法。适合对模块化有初步了解，对webpack有基础的读者。

![图片](http://qncdn001.189che.com/blog_webpack.jpg)
<!-- more -->

首先回顾一下我对模块化、工程化学习的历程。最初（2014）对这些概念还是很混乱的，看到高手们使用requirejs和grunt做项目，能快速加载插件模块，还能合并压缩成一个文件，觉得diao炸天。于是照猫画虎，就用yeoman-generator-webapp脚手架快速构建了一个基于gulp的项目（2015）。但是它没有模块化加载的相关功能，插件脚本的依赖是通过bower（现在基本过时了）进行管理的。感觉仅适用于单页项目，多页项目使用起来不但不方便，还很麻烦。
接着开始尝试使用browserify（2015）来做模块化的功能，但它太小而美了，很多功能都要自己实现。官方给出的示例也需要在后台实时编译打包才能看到效果，页面多了实时编译速度会变慢，项目的工程化依然需要gulp来完成。当时就开始怀疑在自己的项目中加入模块化、工程化是不是真的有用了。总觉得是在拖后腿，影响开发进度，因为经常会出现一些不能理解的问题需要查资料去解决。
后来发现很多大厂都从seajs迁移到webpack了，我也就跟着再折腾一下（2016），大而全（Fis更大而全）的特性，让基础的插件功能已经可以满足所需，模块化和工程化一个工具全部搞定，还有各种黑魔法（之前有[文章](http://zkzhao.github.io/hJN-n3Pj0BAbWkD90DFlBg.html)介绍过）可以很容易的让生产、开发环境各自独立运行，已经可以说是比较完美的解决方案了（还有rollup）。

# webpack配置文件结构优化
在最近的项目中，我对webpack的配置文件引入了类似组件化的思想，因为如果区分开发环境和生产环境，配置文件会很长很繁杂很难懂。引入组件化加载方式后，使得开发、生产环境某些公用、独立功能都可以优雅的共存。比如css的引入方式，如果是外链导入文件的方式，则热更新功能无法使用，所以在开发环境中可以使用内嵌方式，生产环境再使用外链方式。又比如生产环境才需要进行代码的压缩，开发服务器只在开发环境中使用。
基于这个思路，首先创建2个配置文件如下：
```js
/**
 * webpack.dev.config.js （用于开发环境）
 */

//存放一些常用的路径配置
const dirPath= require('./config/dir.path.js');

module.exports= {
  entry: require('./config/entry.config.js'), //入口

  output: require('./config/output.config.js'), //出口

  module: require('./config/module.dev.config.js'), //模块

  plugins: require('./config/plugins.dev.config.js'), //插件

  resolve: require('./config/resolve.config.js'), //解析模块
	
	//开发服务器
  devServer: {
    inline: true, // 可以监控js变化
    hot: true, // 热更新
    compress: true,
    proxy: dirPath.serverProxy
  }
}
```

```js
/**
 * webpack.config.js （用于生产环境）
 */
module.exports= {
  entry: require('./config/entry.config.js'),

  output: require('./config/output.config.js'),

  module: require('./config/module.prod.config.js'),

  plugins: require('./config/plugins.prod.config.js'),

  resolve: require('./config/resolve.config.js'),
}
```

相应的需要在`package.json`中指定配置文件运行路径。

```js
"scripts": {
    "dev": "webpack --progress --colors --config ./webpack-config/webpack.dev.config.js", //开发环境
    "prod": "webpack --progress --colors --config ./webpack-config/webpack.config.js --bail", //生产环境
    "start": "webpack-dev-server --config ./webpack-config/webpack.dev.config.js --open --hot --inline" //启动开发服务器
}
```

完整代码可以star并clone[Github zkzhao/webpack-pitbull项目](https://github.com/zkzhao/webpack-pitbull)本地查看。计划后续会加入yeoman脚手架自动生成初始项目的功能。

# webpack引入模板的使用方法
多页应用大多会有一个需求，那就是共用头部或者尾部。以前只知道webpack支持jade和ejs模板，当时只想共用html代码段就可以，正好后台是基于python的，索性就使用了django的母版页。用下来到也没什么问题，但紧接着一个项目需要使用php，想了想，还是用前端模板，自己比较好掌控，就开始着手思考如何让webpack支持模板页了。
开始的想法是只需要在html页面文件中支持include功能，不影响html页面的结构就可以了。查了很多资料发现ejs-loader、ejs-compiled-loader等这样的loader都需要在脚本文件中先引入ejs模板页进行编译，然后再插入到html中，感觉这样很不灵活，还要在脚本文件引入一次，也不方便。
继续研究发现了`ejs-render-loader`，它可以不在js脚本中引入ejs模板文件，而是在配置文件中的`html-webpack-plugin`插件生成html的时候进行ejs的编译（虽然后来也没有用到），使用方法如下。
```HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Title</title>
  <link rel="stylesheet" href="styles/bundle.css">
</head>
<body>
  <% include includes/header %>
  <h1>index</h1>
  <% include includes/footer %>
</body>
</html>
```

```js
plugins: [
  new HtmlWebpackPlugin({
    filename: '../index.html',
    template:  'ejs-render-loader!./src/index.ejs'
  })
]
```

但同时也出现了新的问题，`html-loader`在对需要src路径的元素（图片）进行解析时会出错，无法根据路径去生成生产环境图片。仔细想想，是因为webpack的plugins是基于stream流的思想，通过entry放入文件，经过module、plugins对文件进行加工，最后在output产出文件。
上面plugins代码片段中`template:  'ejs-render-loader!./src/index.ejs'`就是在加工过程中改变了文件流，导致`html-loader`无法正确识别，那怎么办呢？
首先能想到的解决办法就是让`html-loader`识别被转换过后的文件流，那就仔细看一下文档，发现了文档中有这样一句话：
```
You can use interpolate flag to enable interpolation syntax for ES6 template strings, And if you only want to use require in template and any other ${} are not to be translated, you can set interpolate flag to require
```

原来**Interpolation**就是用于给模板插值的。

```js
require("html-loader?interpolate!./file.html");
```

```HTML
<div>${require('./components/gallery.html')}</div>
```
这刚好可以满足我的需求，之前的众多loader也不需要了。不过后来发现还是需要有传参的功能才行，准备日后再完善一下。

# webpack2、3的更新
1. 最开始看到webpack2的更新文档上说增加了对ES6模块的原生支持，就天真的以为可以不用Babel做转换了，实践了以后才发现，只是增加了`import` 和 `export` 的识别而已。

2. 编译压缩可以使用`new webpack.optimize.UglifyJsPlugin`

3. webpack3 更新文档上说可以与2完美兼容，实现了类似”Tree-shaking“的功能，暂时还没有实践。



