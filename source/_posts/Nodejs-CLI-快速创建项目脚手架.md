---
title: NodeJS CLI 快速创建项目脚手架
categories:
  - 前端工程
  - Node-Cli
tags:
  - 前端脚手架
abbrlink: 7sQGOgJRBhnLuqdeLyyPog
date: 2018-03-09 14:35:21
---

2年前写过一篇快速搭建项目的文章 [Yeoman 快速构建新项目目录结构脚手架工具](http://zkzhao.github.io/W4LWuSjK512T0L1Ni5Ws_w.html) 。最近借鉴vue-cli的思路，脱离Yeoman，重新写了一个CLI工具。本文将关于Node CLI的知识点记录下来，适合有NodeJS基础的读者。[源码地址](https://github.com/zkzhao/dogo-cli)

![gif](http://qncdn001.189che.com/blog_nodecli.gif)

## Shebang
```js
/* dogo.js */
#!/usr/bin/env node  // Shebang（”#!”）
```

package.json文件中加入

```json
"bin": {
	"dogo": "bin/dogo"
}
```

在开发环境中我们实际上使用 `npm link` 便利地将我们的 index.js 软链接到 path 变量的位置。

<!-- more -->

## Commander
> Angled brackets (e.g. <cmd>) indicate required input. Square brackets (e.g. [env]) indicate optional input.

尖括号（例如<cmd>）表示必须参数。 方括号（例如[env]）表示可选参数。

```js
const program = require('commander');
program
	.option('-a, --age <age>', 'your age', '33')
	.command('init') //设置的命令
	.action(function() {
		//用于设置命令执行的相关回调
	});
//最后调用，用于解析process.argv
program.parse(process.argv);
//可以直接获取用户输入的值
console.log(program.ages) 
```

* command – 定义命令行指令，后面可跟上一个name，用空格隔开，如` .command( ‘app [name] ‘)`
* alias – 定义一个更短的命令行指令 ，如执行命令`$ app m `与之是等价的
* description – 描述，它会在help里面展示
* option – 定义参数。它接受四个参数，在第一个参数中，它可输入短名字 -a和长名字–app ,使用 | 或者,分隔，在命令行里使用时，这两个是等价的，区别是后者可以在程序里通过回调获取到；第二个为描述, 会在 help 信息里展示出来；第三个参数为回调函数，他接收的参数为一个string，有时候我们需要一个命令行创建多个模块，就需要一个回调来处理；第四个参数为默认值
* action – 注册一个callback函数,这里需注意目前回调不支持let声明变量
* parse – 解析命令行

## Inquirer
```js
var inquirer = require('inquirer');
inquirer.prompt([/* Pass your questions in here */]).then(function (answers) {
    // Use user feedback for... whatever!! 
})
```

* input – 输入
* validate – 验证
* list – 列表选项
* confirm – 提示
* checkbox – 复选框 [等等](https://github.com/SBoudrias/Inquirer.js)

## Download-git-repo
```js
download('zkzhao/webpack-pitbull', tmp, function (err) {
  if(err){
    console.log(chalkchalk.red('下载出错！'));
  }else{
    console.log(chalkchalk.green('下载完成！'));
  }
});
```

## Metalsmith
metalsmith是一个静态网站生成器，可以用在批量处理模板的场景，类似的工具包还有Wintersmith、Assemble、Hexo。在这里主要用来处理模板，生成编译后的项目文件。
```js
const Metalsmith = require('metalsmith');
const rm = require('rimraf').sync;
Metalsmith(__dirname)
	.source('.')
	.destination(dest)
	.build((err, files) => {
		rm(src); //删除临时下载目录
		// done...
	})
```
## Handlebars
模板引擎，可替换的选择还有ejs，jade。

[基于node.js的脚手架工具开发经历 - 掘金](https://juejin.im/post/5a31d210f265da431a43330e?utm_source=tuicool&utm_medium=referral)
[记一次nodejs开发CLI的过程 - 掘金](https://juejin.im/post/5a90dd62f265da4e9a4973aa)