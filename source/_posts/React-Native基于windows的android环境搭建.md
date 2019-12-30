---
title: React-Native基于windows的android环境搭建疑难点
categories:
- 手机APP开发
- React Native
tags:
- React
- React Native
abbrlink: WgwKKbS8Bx1ODUkoxn0EeQ
date: 2016-06-27 15:15:52
update: 2016-09-11 10:35:19
---
![reactnative](http://qncdn001.189che.com/blog_react-native.png)

对于React Native来说，IOS才是亲儿子，如果你是一名初学者，想按照教程在Windows系统下搭建Android环境，根本不用担心它会一次性成功让你没有折腾的机会，你只需要担心那么多问题你是不是能一一解决，在开始前就应该有它会报错报错报错不停报错的觉悟，下面我就把自己遇到的问题与解决的方法记录下来，希望对大家能有所帮助。
本文适合对node、android开发有一定基础的读者。
<!--more-->

## 注意点一 ：翻墙

下载安装 Android Studio、Android SDK等很多地方都需要翻墙，当出现错误的时候首先想想是不是被墙了。

解决办法：

sdk可以使用国内镜像：http://www.androiddevtools.cn/  （推荐使用腾讯的镜像，如果加速不明显，可以尝试重启电脑）

安装npm 包可以使用淘宝的cnpm镜像 。

## 注意点二 ： SDK 安装不完整

下载SDK包不完整会带来很多错误，根据官方文档的介绍，需要在SDK Manager中安装：
1. Extras/HAXM installer加速驱动
2. Extras/Android Support Repository
3. Tools/Android SDK Tools
4. Tools/Android SDK Platform-tools
5. Tools/Android SDK Build-tools (23.0.1)（这个必须版本严格匹配23.0.1）
6. Android 6.0 (API23)/SDK Platform (这个下面最好全装，使用模拟器需要Atom的相关包)

结论：SDK只装API23版的就可以，剩下的小工具包，最好全装，不然有莫名的错误很难找到原因，反正用国内镜像速度快。

##  注意点三 ： android需要的各种环境变量

JAVA和Android的环境变量，一个都不能少，不然会提示找不到SDK

1. JAVA_HOME
2. ANDROID_HOME
3. %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;    
4. %ANDROID_HOME%\tools;%ANDROID_HOME%\platform-tools;

##  注意点四 ： react-native init MyProject 初始化项目，提示node-gyp rebuild错误

至今依然在纠结这个坑，虽然报错，但是最后还是能成功运行起来，个人猜测，应该是安装npm包的时候出现了编译c++的错误。

查了一下node-gyp的资料，发现这是一个新版本node自带的c++编译扩展包，它需要c++和python的编译环境，我的工作电脑上有Microsoft Visual C++ 2013 Redistributable和python2.7，还安装了Cygwin（初期windows平台安装node需要借助Cygwin来安装，后来微软和node的作者一起合作，才有了现在的node安装包）,依然没有解决报错的问题，计划日后深入学习node后再来填这个坑，错误如下：

![init error](http://qncdn001.189che.com/blog_gyperror.jpg)

> 更新： 已经找到解决方案：https://github.com/nodejs/node-gyp#installation
> 1、全局安装node-gyp，`npm install -g node-gyp`
> 2、CMD安装 windows-build-tools ，`npm install --global --production windows-build-tools`
> 最后在init初始化项目就不会报错了

## 注意点五 ： Android模拟器

经过“注意点4”的折腾，已经身心俱疲，结果还是没搞定，算了，先试一下项目能不能跑起来吧

`react-native start` 起一个端口服务

`react-native run-android` 这里报错，主要是“注意点2”和“注意点3”，漫长的等待之后，提示我没有找到安卓模拟器。

解决办法：

打开Genymotion，在CMD命令行输入`adb devices` 查看当前可使用的安卓虚拟器列表，结果为空，说明确实没有，网上查了下，需要在setting的ADB下选中 `Use custom Android SDK tools` ，然后选择自己的SDK目录，换个姿势再来一次，什么？还是报错！准备放弃治疗了，重启电脑再试最后一次，没想到成功运行起来了。

![adb -l](http://qncdn001.189che.com/blog_adblist.jpg)

都还没开始写代码，只是配置环境，就这么麻烦，所以还是建议有条件的朋友，在mac平台上进行开发，毕竟react native的团队都是使用mac的，不怎么管windows的死活。

最后，关于Genymotion模拟器调试的一些小技巧

* 刷新界面： 按RR
* 调出菜单： 在命令行输入adb shell input keyevent 82 或者 Ctrl+M
* chrome调试：　`http://localhost:8081/debugger-ui`

最后，有问题可以底部留言与我交流。
