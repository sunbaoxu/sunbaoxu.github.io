---
published: true
title: Vue + Canvas炫酷效果
category: javascript
tags: 
  - canvas
  - vue+canvas
layout: post
---

github地址：[https://github.com/sunbaoxu/canvas-back](https://github.com/sunbaoxu/canvas-back) 
文件直接下载地址(vue项目可直接用)：[https://download.csdn.net/download/qq_38998250/12001308](https://download.csdn.net/download/qq_38998250/12001308)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127164028307.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4OTk4MjUw,size_16,color_FFFFFF,t_70)
相信大多数人看到这张图片便知道我要实现的效果了，一直觉得这个效果挺不错，心血来潮查了很多资料终于在项目的登录页用到了。废话不多说了，看代码：

###### 变量：
mainName : String   | 默认为空 ，画布不监听鼠标移动区域的标签ID
dotsMax ： Number | 默认15000 ，点与点之间的距离
RAF    : null 
canvas : null 
 ctx    : null 
 warea  : null | 鼠标粒子数据
 dots   : []    | 记录创建的粒子数据
 len    : 300 | 粒子个数

###### 1) 首先我们需要先创建一个画布
```javascript
// html
<canvas id="canvasId" class="canvas-box" />

// javascript
this.canvas = document.getElementById("cas");
this.ctx = this.canvas.getContext("2d");
```
###### 2) 设置一下画布大小
```javascript
this.canvas.width = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth;
this.canvas.height = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight;
```
###### 3) 根据设备宽高计算粒子个数 （可以写死）
```javascript
//len:Number | 为粒子总个数
let w = document.body.clientWidth;
let h = document.body.clientHeight;
if(Math.floor(w*h/6500) <=50){
  this.len = 100
} else if(Math.floor(w*h/6500) >=300){
  this.len = 300
} else{
  this.len = Math.floor(w*h/6500)
}

if(navigator.userAgent.indexOf("Firefox")>0){
  this.len = 150
}
```
###### 4）告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画
requestAnimationFrame中文文档：[https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame)
```javascript
//这一行代码非常非常重要，我也是最后动画不生效，踩坑才知道的。（小白的惯例，啥都不调研就干上手开发！！）
window.requestAnimationFrame = window.requestAnimationFrame || window.mozRequestAnimationFrame || window.webkitRequestAnimationFrame || window.msRequestAnimationFrame;
```
###### 5）鼠标移动时，获取鼠标坐标。（如果不希望点线随着鼠标移动，可以跳过这里）
```javascript
 this.warea = {x: null, y: null, max: 20000};
 //鼠标移动
 window.onmousemove = (e) =>{
   e = e || window.event;
   //界面是否有内容区域
   if(this.mainName){
     let main = document.getElementById(this.mainName),
       offTop = main.offsetTop,
       offLef = main.offsetLeft,
       offH   = main.offsetHeight,
       offW   = main.offsetWidth;
   
     if(!(e.clientX >=offLef && e.clientX <= offLef+offW)||!(e.clientY >=offTop && e.clientY <= offTop+offH) ){
       this.warea.x = e.clientX;
       this.warea.y = e.clientY;
     } else{
       this.warea.x = null;
       this.warea.y = null;
     }
   } else{
     this.warea.x = e.clientX;
     this.warea.y = e.clientY;
   }
 };
// 鼠标离开
 window.onmouseout = (e)=> {
   this.warea.x = null;
   this.warea.y = null;
 };
```
###### 6) 创建粒子
```javascript
// x，y为粒子坐标，xa, ya为粒子xy轴加速度，max为连线的最大距离
 for (let i = 0; i < this.len; i++) {
   let x = Math.random() * this.canvas.width,
       y = Math.random() * this.canvas.height,
       xa = Math.random() * 2 - 1,
       ya = Math.random() * 2 - 1;
   this.dots.push({
     x: x,
     y: y,
     xa: xa,
     ya: ya,
     max: this.dotsMax
   })
 }
```
###### 7) 渲染粒子位置
```javascript
 this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
 // 将鼠标坐标添加进去，产生一个用于比对距离的点数组
 let ndots = [this.warea].concat(this.dots);
 
 for(let i=0 ; i<this.dots.length;i++){
   let dot = this.dots[i];
   // 粒子位移
   dot.x += dot.xa;
   dot.y += dot.ya;
   // 遇到边界将加速度反向
   dot.xa *= (dot.x > this.canvas.width || dot.x < 0) ? -1 : 1;
   dot.ya *= (dot.y > this.canvas.height || dot.y < 0) ? -1 : 1;
   // 绘制点
   this.ctx.fillRect(dot.x - 0.5, dot.y - 0.5, 1, 1);
   this.ctx.fillStyle="#ffffff";
   //循环比对粒子间的距离
   this.spotLineSize(ndots,dot);
   // 将已经计算过的粒子从数组中删除
   ndots.splice(ndots.indexOf(dot), 1);
 }
 this.RAF = requestAnimationFrame(this.spotPosition);
```
###### 8) 循环比对粒子间距离
```javascript
for (let i = 0; i < ndots.length ; i++) {
  let d2 = ndots[i];
  if (dot === d2 || d2.x === null || d2.y === null) {
    continue
  };
  let xc  = dot.x - d2.x,
      yc  = dot.y - d2.y,
      // 两个粒子之间的距离
      dis = xc * xc + yc * yc,
      // 距离比
      ratio;
      // 如果两个粒子之间的距离小于粒子对象的max值，则在两个粒子间画线
      if (dis < d2.max) {
        // 如果是鼠标，则让粒子向鼠标的位置移动
        if (d2 === this.warea && dis > (d2.max / 2)) {
          dot.x -= xc * 0.03;
          dot.y -= yc * 0.03;
        }
        // 计算距离比
        ratio = (d2.max - dis) / d2.max;
        // 画线
        this.ctx.beginPath();
        this.ctx.lineWidth = ratio / 2;
        this.ctx.strokeStyle = 'rgba(49,90,148,' + (ratio + 0.2) + ')';
        this.ctx.moveTo(dot.x, dot.y);
        this.ctx.lineTo(d2.x, d2.y);
        this.ctx.stroke();
      }
}
```
###### 9) 离开当前页记得销毁window方法
```javascript
window.cancelAnimationFrame(this.RAF);
window.onresize = undefined;
window.onmousemove = undefined;
window.onmouseout  = undefined;
```