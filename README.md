### requestAnimationFrame--写出性能更好的WEB动画

> 前言：要实现动画效果，我们一般会借助JS的setTimeout或setInterval这两个函数。而在CSS3动画出现后，我们便可以使用它的transitions或animation来实现动画，这在性能与流畅度上有了很大的提升。然并卵，CSS3的动画有很多局限性，并不是所有的属性都支持动画，另外缓动效果也比较少，无法完全控制动画过程。更多时候，还是不得不使用JS的原始写法来实现动画，可是setTimeout与setInterval存在严重的性能问题，虽然某些现代浏览器对这两个函数进行了一些优化，但是无法跟CSS3的动画性能相提并论。这个时候，requestAnimationFrame的出现在很大程度上解决了这个问题。
> 

#### 实现动画的常用方法
- 1、window.setTimeout() / window.setInterval()
- 2、transitions
- 3、animation+keyframes

Jquery动画的实现考虑到兼容与易用一般采用SetInterval来不断绘制新的属性值，从而达到动画效果。大部分浏览器的显示频率是16.7ms，由于浏览器的特性，setInterval会有丢帧的问题。即使向其传递毫秒为单位的参数，也不到达到ms的准确性。因为JS是单线程的，可能会发生阻塞。Jquery会有一个全局设置jQuery.fx.nterval = 13设置动画每秒运行帧数。默认是13毫秒，这属性值越小，在速度较快的浏览器中（如：Chrome），动画执行的越流畅，但是会影响程序的性能且占用更多的CPU资源。
那么开发者并不知道下一刻绘制动画的最佳时机是什么时候


#### 什么是requestAnimationFrame
The window.requestAnimationFrame() method tells the browser that you wish to perform an animation and requests that the browser call a specified function to update an animation before the next repaint. The method takes as an argument a callback to be invoked before the repaint.

window.requestAnimationFrame() 将告知浏览器你马上要开始动画效果了，后者需要在下次动画前调用相应方法来更新画面。这个方法就是传递给window.requestAnimationFrame()的回调函数。
requestAnimationFrame是专门为实现高性能帧动画而设计的API。


#### requestAnimationFrame的优势
- 1、requestAnimationFrame会把每一帧中的所有DOM操作集中起来，在一次重绘或回流中就完成，并且重绘或回流时间间隔跟随浏览器的刷新频率（一般为60帧/秒）；
- 2、在隐藏或不可见的元素中，requestAnimationFrame将不会进行重绘或回流，这意味着将耗费更少的CPU、GPU及内存；
- 3、requestAnimationFrame会像setTimeout一样有一个返回值ID用于取消，可以把它作为参数值入cancelAnimationFrame函数来取消requestAnimationFrame的回调；


#### 基本语法
与setTimeout/setInterval类似，requestAnimationFrame是一个全局函数。调用requestAnimationFrame后，它会要求浏览器根据自己的频率进行一重绘，它接收一个回调函数作为参数。在即将开始的浏览器重绘时，会调用这个函数。由于requestAnimationFrame的动效是一次性的，所以想要达到动画效果，需要连续不断的调用requestAnimationFrame。同时，在调用回调函数时返回有一个ID值，通过把这个ID值伟给window.cancelAnimationFrame()可以取消该次动画。
> requestAnimaitonFrame(callback)   //callback为回调函数


#### 兼容性
目前，各个支持requestAnimationFrame的浏览器有些还是自己的私有实现，所以需要添加前缀。对于不支持requestAnimationFrame的浏览器，我们可以使用setTimeout，由于两者使用方式类似，所以优雅降级并不难实现。如：
```
window.requestAnimFrame = (function(){
  return  window.requestAnimationFrame       ||
          window.webkitRequestAnimationFrame ||
          window.mozRequestAnimationFrame    ||
          function( callback ){
            window.setTimeout(callback, 1000 / 60);
          };
})();
```


#### 统一封装
```
var lastTime = 0;
var vendors = ['ms', 'moz', 'webkit', 'o'];    //统一各个浏览器前缀
var len = vendors.length;
for(var i=0; i<len && !window.requestAnimationFrame; ++i){
	window.requestAnimationFrame = window[vendors[i] + 'RequestAnimationFrame'];
	//webkit中取消方法有变
	window.cancelAnimationFrame = window[vendors[i] + 'CancelAnimationFrame'] || window[vendors[i] + 'CancelRequestAnimationFrame']
} 
//浏览器没有requestAnimationFrame方法的，使用setTimeout
if(!window.requestAnimationFrame){ 
	window.requestAnimationFrame = function(callback, element){
		var curTime = new Date().getTime();
		var timeToCall = Math.max(0, 16.7-(curTime - lastTime));
		var id = window.setTimeout(function(){
			callback(curTime + timeToCall);
		}, timeToCall)
		lastTime = curTime+timeToCall;
		return id;
	}
}
if(!window.cancelAnimationFrame){
	window.cancelAnimationFrame = function(){
		clearTimeout(id);
	}
}

```

#### 动画形式
- Linear：无缓动效果
- Quadratic：二次方的缓动（t^2）
- Cubic：三次方的缓动（t^3）
- Quartic：四次方的缓动（t^4）
- Quintic：五次方的缓动（t^5）
- Sinusoidal：正弦曲线的缓动（sin(t)）
- Exponential：指数曲线的缓动（2^t）
- Circular：圆形曲线的缓动（sqrt(1-t^2)）
- Elastic：指数衰减的正弦曲线缓动
- 超过范围的三次方缓动（(s+1)*t^3 – s*t^2）
- 指数衰减的反弹缓动

> 每个效果都分三个缓动方式，分别是（可采用后面的邪恶记忆法帮助记忆）：
- easeIn：从0开始加速的缓动；
- easeOut：减速到0的缓动；
- easeInOut：前半段从0开始加速，后半段减速到0的缓动；

缓动封装：
详见：[tween.js](https://github.com/zhangxinxu/Tween/blob/master/tween.js)

#### 案例演示
弹动的小球：[预览](http://www.zhangxinxu.com/study/201309/requestanimationframe-tween-easeoutbounce.html)
```
funFall = function() {
    var start = 0, during = 100;
    var _run = function() {
         start++;
         var top = Tween.Bounce.easeOut(start, objBall.top, 500 - objBall.top, during);
         ball.css("top", top);
         shadowWithBall(top);    // 投影跟随小球的动
         if (start < during) requestAnimationFrame(_run);
    };
    _run();
};
```

#### 参考文章
- http://www.zhangxinxu.com/wordpress/2013/09/css3-animation-requestanimationframe-tween-%E5%8A%A8%E7%94%BB%E7%AE%97%E6%B3%95/
- http://www.cnblogs.com/Wayou/p/requestAnimationFrame.html
- http://www.cnblogs.com/2050/p/3871517.html
