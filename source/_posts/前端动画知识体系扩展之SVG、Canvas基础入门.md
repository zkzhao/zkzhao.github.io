---
title: 前端动画知识体系扩展之SVG、Canvas基础入门
categories:
  - 动画特效
  - Canvas
tags:
  - SVG
  - Canvas
abbrlink: fyKNDxcd18PZ6iZu8JaCzg
date: 2017-09-17 10:46:23
---

过去经常感慨，微信上那些所谓的H5营销推广页怎么都这么酷炫。也曾怀疑过，这真是前端做出的页面吗？为什么我看着一点思路也没有，为什么！？
深入研究才发现并想起，曾经有个职业叫Flash动画交互师。直到Flash被乔布斯宣判了死刑、Html5的兴起，这些曾经的Flasher才被迫向前端技术转变。他们的工作就是做酷炫的动画效果和互动效果。这就是为什么很多传统前端不知道动画H5是怎么做的了，毕竟隔行如隔山。
脱离Flash后的动画开始使用Div+Css来排版布局，用一些Css3+JS框架做逐帧动画（如：**Tweenmax 缓动库**）。
但由于使用代码控制动画非常不直观，想要处理复杂动画需要非常多的Tween叠加。慢慢开始出现了一些仿ActionScript3框架的JS引擎，如**Createjs，Egret（白鹭），layaAir**。和一些3D引擎，如**Three.js、A-Frame**等。
至此，出现了大量使用**canvas2d**和**webgl3d**的技术，但又不如游戏行业对于机制的要求那么高。

<!--more-->

## SVG、Canvas、CSS3动画基础概念介绍

- css3主要用来做过渡动画，可以配合svg和canvas使用。关于css3之前有过一篇文章[Three.js+TweenMax帮你实现骚气的H5动画广告](http://zkzhao.github.io/6IeVPTJlGdtsxb0AAqk6ug.html)，有兴趣的同学可以看下。
- SVG是保留模式（retained mode）的图形模型，将图形模型持久化在内存中。SVG基于图形，适合做静态图像，与线条、路径动画。现在移动端SVG icon图标的使用已经超过 iconfont 图标。
- Canvas是一个底层、立即模式（immediate mode）的应用程序编程接口。基于像素，通过脚本以编程方式创建和修改视觉呈现。

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

- canvas是画框，有自己固定的高宽，svg是不依赖分辨率的矢量，可以任意放大缩小。
- canvas能以.jpg的格式保存图像，svg是文本的格式保存图像
- canvas绘制的图像不占DOM，而svg的每个图像都是1个DOM元素
- canvas适合图像密集型的动画，而svg不适合大量使用，例如制作飘雪等
- canvas完全依赖脚本绘制作，而svg可直接使用矢量转存生成。

位移 translate(x,y)
旋转 rotate(deg)
缩放 scale(sx,sy)

```
[a  c  e]           a 水平缩放（1）
[b  d  f]           b 水平倾斜（0）
[0  0  1]           c 垂直倾斜（0）
                    d 垂直缩放（1）
                    e 水平位移（0）
                    f 垂直位移（0）
```

```js
// a,d 水平、垂直缩放
// b,c 水平、垂直倾斜
// e,f 水平、垂直位移
transform(a,b,c,d,e,f)
setTransform()
```

## CSS3 3D转换

transition : 从一个属性值平滑的过渡到另一个属性值

animation ：通过关键帧，产生复杂的动画效果

transform 同样适用2D 需要3D: transform-style:preserve-3d
-translate
-rotate

perspective: 800 (视口距离物体的远近)
perspective-origin: 50% 50% (视点的位置)

transform-origin 调整旋转中心