---
title: 'H5动画进阶教程之Createjs配合Adobe Animate快速生产'
abbrlink: V_J6xowt2dUvmYwM0Pc3BQ
date: 2017-11-03 17:50:05
tags:
---

突然看到Flash即将停止维护退出历史舞台的新闻，想起了从高中时代开始，就听说过flash能做动画，当时心心念念想自己做个帅气的动画，就专门跑去电脑城买了Macromedia Flash的光盘，光盘上还印着“闪客”的字样。但当时还是太天真了，不知道Flash只是个工具，想做出动画还需要很多相关知识，无奈只好放弃，做动画也从此成了我的一个小心愿。
最近重新研究了一下现代动画的制作方式，发现Adobe Animate CC就是Flash的替代产品，可以直接导出Canvas在Web中使用，Createjs也成了替代ActionScript3的脚本库。
本文将介绍Createjs与Adobe Animate CC的协作开发方式，对于无Flash基础的前端人员，建议先了解下Flash基础后再阅读本文。

<!-- more -->

## 舞台
打开AN，设置舞台的大小为 750*1206，为什么不是750*1334呢？因为微信浏览器的标题栏大小是128，所以1334-128=1206。

## 自适应
```html
<!-- 修改viewport全局缩放 -->
<meta name="viewport" content="width=device-width, initial-scale=0.5, maximum-scale=0.5, minimum-scale=0.5, user-scalable=no">
```

``` js
/**
 * 自适应函数
 */
function resize(){
  if(stageWidth != document.documentElement.clientWidth || stageHeight != document.documentElement.clientHeight){
    stageWidth=  document.documentElement.clientWidth;
    stageHeight= document.documentElement.clientHeight;
    // 外部元素自适应
    canvas.width= stageWidth ;
    canvas.height= stageHeight;
    // 内部元素自适应
    stageScale = stageHeight / 1206; // 高度自适应
    //stageScale = stageWidth / 750; // 宽度自适应(两者选一)
    container.scaleX= stageScale;
    container.scaleY= stageScale;
    // 保证图片居中 (高度自适应时需要)
    container.x= (stageWidth - 750*container.scaleX) / 2;
    // 按钮元素页面位置固定
    if(soundBtn){
      soundBtn.x= stageWidth - 50;
      soundBtn.y= 50;
    }
  }
}
```

```js
/**
 * 自适应函数使用方法
 */
createjs.Ticker.timingMode =  createjs.Ticker.RAF_SYNCHED;
createjs.Ticker.setFPS(30);
createjs.Ticker.addEventListener("tick", function(){
  resize(); // 调用自适应函数
  stage.update(); // 刷新舞台
});
```

## 面向对象 与 类链接的使用
```js
/**
 * createjs.extend: 继承createjs.Container
 * createjs.promote: 使this.Map_constructor可以调用父类的构造函数
 * this.Xxx_constructor: Xxx部分要和promote中传入的参数一致
 * 使用p来定义方法可以方便在构造之前重写,AN调出来的类不能继承,也不能重写
 */
var View = {};
;(function(){
  function LoadingView(){
    this.Map_constructor();
    this.content = new lib.view1();
    this.addChild(this.content);
  }
  var p = createjs.extend(LoadingView, createjs.Container);
  p.dongzuo = function(){ /* 写方法 */ }
  View.LoadingView = createjs.promote(LoadingView, "Map"); //这里的 "Map" 影响上面的 Xxx_constructor()
}());

var ss= new View.LoadingView();
ss.dongzuo();
```

使用面向对象的方式去调用cjs的方法：
`ss.content.gotoAndStop(1)`
使用最简类链接的方式调用cjs的方法：
`ss.gotoAndStop(1)`

`Uncaught TypeError: Cannot read property 'getNumFrames' of undefined`
这个错误是因为图片文件没有加载成功。

快速使用类链接： `var loadingView= new lib.view1();`，实例化后的MC并没有没有插入舞台


mc 拖入舞台，可以使用实例名称，未拖入舞台，使用类链接

mc 实例名称： 在mc属性栏

![图片](http://7xk7wj.com1.z0.glb.clouddn.com/anpic.png)



`new lib.view1().content.loadingBar.gotoAndStop(Math.floor(event.progress*99))`

## 类链接的mc通过代码加入场景
```js
	var stage= new createjs.Stage(this.opts.canvas);
	var nextPageBtn = new lib.nextPageBtn();
	nextPageBtn.x= this.opts.stageWidth/2-30; // 插入舞台的X轴位置
	nextPageBtn.y= this.opts.stageHeight-100; // 插入舞台的Y轴位置
	stage.addChild(nextPageBtn);
```

## createjs与AN之间的通信方式
在AN的帧上加入动作脚本dispatchEvent

```js
data.type= "mc"
this.dispatchEvent("play")
​````

在JS中
​```js
var data={};
var loadingView= new lib.view1(); //类链接

loadingView.addEventListener("play",function(){
	alert(data.type);
});
```

或者：
```js
	/**
	 * 帧动作脚本中
	 */	
	if(model) model.dispatchEvent("play");

	/**
	 * JS中
	 */
	model = new createjs.EventDispatcher();
	model.addEventListener("play",function(){
		alert(data.type);
	});
```

