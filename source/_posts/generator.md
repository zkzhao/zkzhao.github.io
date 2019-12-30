---
title: Yeoman 快速构建新项目目录结构脚手架工具
categories:
- 前端工程
- Yeoman
tags:
- 前端脚手架
abbrlink: W4LWuSjK512T0L1Ni5Ws_w
date: 2016-07-12 16:16:21
---

因为工作中经常需要创建新的项目目录（不要问我为什么），每次开始新的项目都需要新建一次前端目录结构、流程化构建等等，后来发现使用Yeoman新建别工程目录很方便，就想自己也定制一套，慢慢完善。

![yeoman脚手架](http://qncdn001.189che.com/blog_yeoman.jpg)

## 安装配置
*  全局安装yo和yeoman-generator： `npm install -g yo ` ， ` npm install --save yeoman-generator`
*  创建一个以 `generator-` 开头的文件夹，例如： `generator-name`
*  在命令行运行 `npm init` 来构建package.json文件，其中 `name` 属性必须以 `generator-` 前缀开头。

<!-- more -->

```javascript
  {
    "name": "generator-name",
    "version": "0.1.0",
    "description": "",
    "files": [
      "app",
      "router"
    ],
    "keywords": ["yeoman-generator"],
    "dependencies": {
      "yeoman-generator": "^0.20.2"
    }
  }
```
* generator把app作为默认的子generator,当你使用 `yo name` 调用的是app子generator。可以通过 `yo name:subcommand` 命令来调用子generator，它会调用与subcommand完全一样的文件夹即子generator。
```javascript
  ├───package.json
  ├───app/
  │   └───index.js
  └───router/
      └───index.js
```
* 实现generator的具体内容，可以在index.js中重写构造函数并添加自己的方法。
```javascript
  var generators = require('yeoman-generator');
  module.exports = generators.Base.extend({
    constructor: function () {
      generators.Base.apply(this, arguments);
      this.option('optionsName'); 
    },
    method1: function () {
      console.log('method just ran');
    },
  });
```
* 在 generator-name 项目跟路径下执行  `npm link` 安装node模块依赖，创建软连接指向你的当前项目。
* 创建新文件夹，进入该文件夹命令行执行 `yo name` ，即可构建出你自己定制的 generator-name 项目结构。

## 生命周期
Yeoman是按照优先级顺序依次执行所定义的方法。当你定义的函数名字是Yeoman定义的优先级函数名时，会自动将该函数列入到所在优先级队列中，否则就会列入到 default 优先层级队列中。

1. initializing - 初始化一些状态之类的，通常是和用户输入的 options 或者 arguments 打交道，这个后面说。
2. prompting - 和用户交互的时候（命令行问答之类的）调用。
3. configuring - 保存配置文件（如 .babelrc、.editorconfig  等）。
4. default - 如果函数名称如生命周期钩子不一样，则会被放进这个组。
5. writing - 在这里写一些模板文件，generator特殊的文件（路由、控制器、等）。
6. conflicts - 处理文件冲突，比如当前目录下已经有了同名文件。
7. install - 开始安装依赖（npm、bower）。
8. end - 安装结束、清除文件、设置good bye文案、等

## 可以使用的模块依赖、工具库
* [inquirer](https://github.com/SBoudrias/Inquirer.js)：提供命令行交互的工具
* [yosay](https://github.com/yeoman/yosay)：在命令行输出信息的时候，同时输出 Yeoman 的卡通人物
* [chalk](https://github.com/chalk/chalk)：名字叫粉笔，可以给命令行输出各种不同的颜色。
* ejs： 模板引擎。

## 构建思路
最基础的思路就是通过inquirer和用户交互，收集到需求后通过ejs模板引擎构建出需要的各类文件，最后安装npm、bower依赖。
这里有一个我写的[demo](https://github.com/zkzhao/generator-yi)，可以参考学习。

本人学习参考的文章：
1.  [使用YEOMAN创建属于自己的前端工作流](http://www.open-open.com/lib/view/open1460133960139.html)
2.  [写一个自己的 Yeoman Generator](https://leozdgao.me/write-yeoman-generator/)
3.  [yeoman-generator 入门教程](https://segmentfault.com/a/1190000005827971)