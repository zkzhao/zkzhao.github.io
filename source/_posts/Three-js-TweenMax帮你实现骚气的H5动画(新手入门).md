---
title: Three.js+TweenMax帮你实现骚气的H5动画(新手入门)
categories:
  - 动画特效
  - 3D
tags:
  - Three.js
  - TweenMax
abbrlink: 6IeVPTJlGdtsxb0AAqk6ug
date: 2016-11-02 15:30:18
---
> 本文未涉及详细的代码步骤来深入讲解H5动画的实现过程，只提供一些技术思路和方向，以供深入研究和学习。本文适合对css3与jquery动画库有一定了解的初学者。

公司突然要做一个H5广告，老板只有一个要求，就是要有3D效果，要向《穿越故宫来看你》、《淘宝造物节》等大神级作品看齐。回头看下只有1个策划2个美工不到3个前端的团队，额，臣妾做不到啊。但是老板要求了，别说要搞3D动画，就算他想搞原子弹，我们也要想办法弄清楚原子弹的设计原理，做不出大神级别H5，做个草根山寨版的也行啊，劳动人民的智慧总是无穷无尽的.....
<!-- more -->
## 视频动画
要做H5的3D动画广告，最先想到的就是视频了，这种方式当然对我们前端程序员来说是最省事的，只需要找相关视频制作的专业人事，做出一个狂拽酷炫吊炸天的视频，然后嵌入H5页面就搞定了。其中唯一需要担心的是网页视频在安卓设备上播放的时候，会默认冒出播放器的进度条。最简单粗暴的解决办法就是把视频的尺寸放大，让进度条飞出显示屏，嗯，完美。当然还有唯二需要担心的事情，比如公司没有这么一个会做视频的人.....

## 幻灯片图片动画
相信大家小时候都玩过书角动画（也叫手翻书），每一页的动画都有细微的差别，快速翻过由于视觉停留的缘故，看起来就像连续不间断的动画。H5动画广告也使用这种方法，*在视频软件中制作特效后再导出为序列帧图片*，设置时间间隔后让它们一张一张连续播放，再加一段音频字幕，完全可以以假乱真，假装自己是视频了。有的同学要问，那为什么不直接用视频呢？原因很简单，视频不能和用户交互呀！视频前端程序员不会做呀！对呀！不会做呀，那怎么导出连贯的静态图片？这个方案也不行，放弃。继续往下看吧。

## 补间动画和CSS3 3D转换
既然前2种方法中的视频制作对我们来说有些难度，那我们还是回归本质，运用自己最擅长的css和js来解决问题。

![](http://7xk7wj.com1.z0.glb.clouddn.com/blog_zaowujie.png)

看到这张**淘宝造物节作者**的聊天记录截图，瞬间感觉被开启了新世界的大门，原来造物节广告是利用css3 3d转换实现的全景伪3d，不得不佩服这位设计师大神的智慧，把css3玩出了新的高度。他使用动画库jstween、css3d、陀螺仪orienter.js（用于横竖屏重力感应）等技术完成了这部作品。那我们先从简单的开始研究，jstween动画库是造物节作者自己封装的一个js动画库，适合设计大师的不一定适合自己，我们还是选个文档齐全点的来玩吧。要做补间动画最重要的就是timeline，GSAP的TimeLineMax库可以很方便的管理时间轴动画，但是GSAP提供了TweenLite.js、TweenMax.js、TimelineLite.js和TimelineMax.js 4个文件，不用一脸懵逼，下面就介绍一下这些引用库文件都是做什么的。

* **TweenLite** 是GSAP的主体核心，它用于创建基本动画，是其他各类模块的基础。一般都会搭配plugins/CSSPlugin以实现DOM元素的动画（也就是我们最熟悉的动画了）。
* **TimelineLite** 是一个叫做时间轴的动画容器，它用于对多个动画进行有序的组织与控制。
* **TimeLineMax** 是TimelineLite的升级版，在TimelineLite的基础之上，可以有更高级的组织与控制。
* **TweenMax** 是GSAP集合包，除前面3个之外，还包括plugins里的常用插件以及easing里的缓动函数补充。

因此，如果想要简单地引入GSAP的主体功能，使用TweenMax.js这一个文件即可。而如果要争取更小的库文件大小，应该使用TweenLite.js（必需）+ 其他文件的组合。下面是它们的重叠包含关系图。

![](http://7xk7wj.com1.z0.glb.clouddn.com/blog_2259286548-5748f6ff2ca90_articlex.png)

搞明白这些概念之后，剩下的就简单了，只需要打开 [文档](http://greensock.com/tweenlite) ，做几个Deom就什么都会了（毕竟更深的东西暂时也用不到）。下面给出一个简单的时间轴实现代码，只需要to to to to.....就完成了，是不是很容易。
```javascript
var tl = new TimelineLite();
tl.to($box, 1, {x:50,y:0})
  .to($box, 1, {x:50,y:50})
  .to($box, 1, {x:-50,y:50})
  .to($box, 1, {x:-50,y:0});
```

GSAP还非常贴心的给出了一份包含丰富参考代码的 [备忘单（Cheat Sheet）](https://ihatetomatoes.net/wp-content/uploads/2016/07/GreenSock-Cheatsheet-4.pdf)。

## 真正的3D动画webGL库---threejs
在H5内想实现空间变换和推拉的效果，面对很多素材的大型场景时，灵巧的css3 3d空间变换显得十分吃力。为了更好的效率和流畅的体验，可以选用webGL来绘制3D场景，通过3DMAX、玛雅等工具建模后导入3d模型进行外部模型加载。

制作三维场景的基本要素：

1. 场景（Scene）：模型、灯光等；
  场景就是要在三维中显示的内容，好比电影中的场地、演员、道具，有了场景就知道要在三维中显示什么内容。

2. 相机（Camera）：观察场景的视角；
  相机是用来观察场景的视角，好比电影中的摄像机，有了相机元素才能把场景中的内容获取下来，相机控制是场景交互和动画的基础。

3. 渲染器（Renderer）：场景渲染输出的目标；
4. 渲染（render）：执行渲染操作；

下面有一个简单的使用three.js实现3d模型的详细例子：

```javascript
// 创建场景元素对象
var scene= new THREE.Scene();  
// 模型对象（100,100,100分别为长宽高）
var geometry= new THREE.BoxGeometry(100,100,100);  
//材质对象 （红色的材质）
var material= new THREE.MeshLamberMaterial({color: 0xff0000}); 
// 添加网格模型
var mesh= new THREE.Mesh(geometry,material); 
//将mesh对象加入场景中
scene.add(mesh) ;
// 添加灯光（白色的点光源）
var light= new THREE.PointLight(0xffffff) ;
// 设置光源的位置
light.position.set(300,400,200);
// 将光源加入场景之中
scene.add(light);
// 创建相机对象（透视相机）
var camera= new THREE.PerspectiveCamera(40,800/600,1,1000);
// 设置相机的位置
camera.position.set(200,200,200);
// 将相机朝向场景的中心
camera.lookAt(scene.position);
// 创建WebGL渲染器
var renderer= new THREE.WebGLRenderer();
// 设置渲染器的尺寸
renderer.setSize(800,600);
document.body.appendChild(renderer.domElement);
//渲染 场景和相机
renderer.render(scene,camera);
```

总结：实际上H5广告的技术并不算难，工作量基本耗费在美术素材上，有点人海战术的意思，依靠大量元素的视觉轰炸来达到惊艳的视觉体验。

