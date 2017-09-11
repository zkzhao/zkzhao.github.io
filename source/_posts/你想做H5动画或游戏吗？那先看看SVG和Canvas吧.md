---
title: 你想做H5动画或游戏吗？那先看看SVG和Canvas吧
categories:
  - 动画特效
  - Canvas
tags:
  - SVG
  - Canvas
abbrlink: MVpA4qVVnJRpx9pDfxb7rQ
date: 2017-08-14 17:10:43
---

从高中时代开始听说flash能做动画，就一直心心念念想自己做出帅气的动画，当时还专门跑到电脑城，买了Macromedia Flash的光盘，光盘上还印着“闪客”的字样。但当时还是太天真了，flash只是个工具，想做出动画还需要很多相关知识，无奈只好放弃，做动画也成了我的一个小心愿。
最近突然看到flash要停止维护了，就又研究了一下现代动画的制作方式，发现Animate是flash的替代产品，可以直接导出Canvas在web中使用，趁着最近不是很忙，系统的学习一下动画知识吧。下面就把我学习过程中的疑问都记录下来，本文适合无任何基础的新人。

> 先上结论：经过这几天的摸索发现，动画或2D小游戏可以使用SVG、Canvas。游戏开发使用layaAir或Egret（白鹭），layaAir支持JS，更适合前端开发者。

<!-- more -->

## CSS3、SVG、Canvas介绍
1. css3主要用来做过渡动画，可以配合svg和canvas使用。关于css3之前有过一篇文章[Three.js+TweenMax帮你实现骚气的H5动画广告](http://zkzhao.github.io/6IeVPTJlGdtsxb0AAqk6ug.html)，有兴趣的同学可以看下。

2. SVG是保留模式（retained mode）的图形模型，将图形模型持久化在内存中。SVG基于图形，适合做静态图像，与线条、路径动画。现在移动端SVG icon图标的使用已经超过 iconfont 图标。

3. Canvas是一个底层、立即模式（immediate mode）的应用程序编程接口。基于像素，通过脚本以编程方式创建和修改视觉呈现。

SVG与Canvas比较，屏幕较大的话，Canvas性能欠佳，因为需要绘制更多的像素。对象数量较多的话，SVG性能欠佳，因为DOM加载较慢。另外还有js引擎速度，浏览器或者其他环境是否完全使用硬件加速等等也会左右着两者的选择。在浏览器，可以说Canvas这项H5新技术更容易为广大的Web开发人员所接受，很重要的一方面是因为它简单且高效。但是除去浏览器，SVG可用于CAD建筑，工程等要求高保真度的复杂矢量文档方面，它占有很明显的优势。


## SVG基础知识
SVG是一种可缩放矢量图形是基于可扩展标记语言（XML），用于描述二维矢量图形的图形格式。SVG由W3C制定，是一个开放标准。简单的理解它是图形的另一种格式例如它和常见的图片格式.png、.jpg、.gif等是一类。与图片对比支持矢量，容量小，与iconfont对比可以增加颜色。

### SVG的2个概念： 视窗、世界、视野
```html
<svg style=“height;width”></svg>   <!-- <svg>限制的是视窗 -->

<rect x y width height>    <!--x,y 坐标系; w,h 视野-->
```

世界：无限大，SVG代码定义,（只要里面有SVG代码则存在SVG世界）
视野（viewbox）：可视区域

### 坐标系
世界的坐标系叫**用户坐标系**，所有坐标系都是从用户坐标系开始，所以也叫原始坐标系。
图形或分组的坐标系叫**自身坐标系**
父容器的坐标系叫**前驱坐标系**
任意的坐标系叫**参考坐标系**

### 坐标变换
线性变换方程：
X=aX+cY+e
Y=bX+dY+f
实现方式有矩阵的形式和Transform的形式

## Canvas基础知识
大部分基础知识都和svg大同小异

位移 translate(x,y)
旋转 rotate(deg)
缩放 scale(sx,sy)

[a  c  e]           a 水平缩放（1）

[b  d  f]           b 水平倾斜（0）

[0  0  1]           c 垂直倾斜（0）

​			d 垂直缩放（1）

​			e 水平位移（0）

​			f 垂直位移（0）			


```js
// a,d 水平、垂直缩放
// b,c 水平、垂直倾斜
// e,f 水平、垂直位移
transform(a,b,c,d,e,f)
setTransform()

```

