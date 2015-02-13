title: 献给2014年的第一场雪
date: 2013-12-12 16:22:52
tags: js动画 css3 requestAnimationFrame 雪花动画
---

当我查找MVC、MVP、MVVM这些设计模式相关资料时，发现了这个背景效果——飘雪——做得还不错的网站，[http://nirajrules.wordpress.com/2009/07/18/mvc-vs-mvp-vs-mvvm/](http://nirajrules.wordpress.com/2009/07/18/mvc-vs-mvp-vs-mvvm/)，如果你不能打开链接，就悲剧了，“墙”里难见“墙”外花。于是，从该站点剥离出纯粹实现此动画的代码，制作demo如链接（忙啊！！还没传上去）。

我们会发现一些细节，鼠标移动的轨迹控制了风向和风速，从而控制雪花的方向和速度；当前tab不是活动tab时动画停止，避免过度绘制。说说你实现该效果的思路，或是关键词。定时器？运动轨迹？边际检测？我们先按照自己的思路一步一步走，再结合它的代码修正。
1. 制造雪花
2. 执行定时器

哦，就这样？好像也很简单~

###1、采用何种模式？
前提是基于原生javascript来考虑，不要受jquery的影响。首先这里不涉及继承，这就简单很多。javascript中最基本又最有用的模式就是单体，用来划分命名空间并将一批相关属性和方法组织在一起，同时也对外提供了一个访问内部属性和方法的访问点。为了避免被外界访问，将单例的“值部分”闭包起来，在内部返回当前活动对象（绑定到this上的属性和方法仍然可以被外部访问）。好，外壳做好了~

	var snowStorm = (function(window, document) {
	   ...
	   return this;
	}(window, document));;

###2、制造雪花
如何构造雪花，它的构造函数需要哪些属性？位置坐标必须有，用来记录和控制轨迹？关联DOM对象必须有， 方便设置view相关属性? 它需要哪些方法呢？setPosition（设置位置）？setVelocities（设置速度）？move（移动的入口函数）？根据效果需要可能还需要melt（融化）、stick（堆成雪堆）等。多个雪花实例是随机生成的，运动也互不影响，不必有共享的属性和方法，所以不考虑原型。抽取出的代码结构如下：

	this.SnowFlake = function(type,x,y) {
	    var s = this;
	    this.type = type;
	    this.x = x||parseInt(rnd(screenX-20),10);
	    this.y = (!isNaN(y)?y:-rnd(screenY)-12);
	    this.vX = null;
	    this.vY = null;
	    this.active = 1;
	    this.fontSize = (10+(this.type/5)*10);
	    this.o = document.createElement('div');
	    this.o.innerHTML = storm.snowCharacter;
	    this.o.style.color = storm.snowColor;
	    this.o.style.position = (fixedForEverything?'fixed':'absolute');
	    this.o.style.width = storm.flakeWidth+'px';
	    this.o.style.height = storm.flakeHeight+'px';
	    this.o.style.fontFamily = 'arial,verdana';
	    this.o.style.cursor = 'default';
	    this.o.style.overflow = 'hidden';
	    this.o.style.fontWeight = 'normal';
	    this.o.style.zIndex = storm.zIndex;
	    docFrag.appendChild(this.o);

	    this.refresh = function() {
	      ......
	      storm.setXY(s.o, s.x, s.y);
	    };

	    this.vCheck = function() {
	      if (s.vX>=0 && s.vX<0.2) {
	        s.vX = 0.2;
	      } else if (s.vX<0 && s.vX>-0.2) {
	        s.vX = -0.2;
	      }
	      if (s.vY>=0 && s.vY<0.2) {
	        s.vY = 0.2;
	      }
	    };

	    this.move = function() {
	        ...
	    };

	    this.setVelocities = function() {
	      s.vX = vRndX+rnd(storm.vMaxX*0.12,0.1);
	      s.vY = vRndY+rnd(storm.vMaxY*0.12,0.1);
	    };

	    this.setOpacity = function(o,opacity) {
	      if (!opacitySupported) {
	        return false;
	      }
	      o.style.opacity = opacity;
	    };

	    this.melt = function() {
	      if (!storm.useMeltEffect || !s.melting) {
	        s.recycle();
	      } else {
	        if (s.meltFrame < s.meltFrameCount) {
	          s.setOpacity(s.o,s.meltFrames[s.meltFrame]);
	          s.o.style.fontSize = s.fontSize-(s.fontSize*(s.meltFrame/s.meltFrameCount))+'px';
	          s.o.style.lineHeight = storm.flakeHeight+2+(storm.flakeHeight*0.75*(s.meltFrame/s.meltFrameCount))+'px';
	          s.meltFrame++;
	        } else {
	          s.recycle();
	        }
	      }
    };
构造函数完成，可以制造雪花，插入到页面了。

	this.createSnow = function(limit,allowInactive) {
	    var i;
	    for (i=0; i<limit; i++) {
	      storm.flakes[storm.flakes.length] = new storm.SnowFlake(parseInt(rnd(flakeTypes),10));
	      if (allowInactive || i>storm.flakesMaxActive) {
	        storm.flakes[storm.flakes.length-1].active = -1;
	      }
	    }
	    storm.targetElement.appendChild(docFrag);
	};
关于move方法的实现，也就是移动轨迹相关的细节，我曾一度想了解动画原理，仔细分析下这个方法：

	this.move = function() {
		// rnd: 返回[param2,param1)之间的随机数
		// plusMinus: 以相同概率返回±param
		// storm.vMaxX=2.5：水平方向最大速度
		// 水平风速vRndX = plusMinus(rnd(storm.vMaxX,0.2)); --from randomizeWind（）
		// vRndX∈±[0.2,2.5)
		// 水平速度s.vX = vRndX+rnd(storm.vMaxX*0.12,0.1) –-from setVelocities（）
		// s.vX = ±[0.2,2.5) + [0.1,2.5*0.12);
		// 水平速度≠水平风速？为什么要修正[0.1,2.5*0.12)，0.12是怎么来的？
		// 风偏速windOffset默认为1，并根据鼠标移动的位置计算。处于屏幕中间为0，最左侧为-2，最右侧为2；windOffset∈[-2,2]
		var vX = s.vX*windOffset, yDiff;
		// 单位时间运动的终点值=初始值+水平速度
		s.x += vX;
		// 同理s.vX = [0.2,2.5) + [0.1,2.5*0.12);
		// 之前忽略了一点，雪花和雪花是不一样的。尤其是轻重不同，那下落速度必定不同。
		// this.vAmpTypes = [1,1.2,1.4,1.6,1.8];
		// this.type ∈[0,5)的随机整数
		// this.vAmp = this.vAmpTypes[this.type];
		s.y += (s.vY*s.vAmp);
		// 边际检测
		// 右侧出屏幕，设置到左边,左侧出屏幕，设置到右边
		if (s.x >= screenX || screenX-s.x < storm.flakeWidth) { // X-axis scroll check
		s.x = 0;
		} else if (vX < 0 && s.x-storm.flakeLeftOffset < -storm.flakeWidth) {
		s.x = screenX-storm.flakeWidth-1; // flakeWidth;
		}
		// 设置到页面中
		s.refresh();
		// yDiff 当前雪花位置距离屏幕底部的高度
		// 以下实现雪花是否成堆、融化的效果，不再赘述
		yDiff = screenY+scrollY-s.y+storm.flakeHeight;
		if (yDiff<storm.flakeHeight) {
			s.active = 0;
			if (storm.snowStick) {
			  s.stick();
			} else {
			  s.recycle();
			}
		} else {
			if (storm.useMeltEffect && s.active && s.type < 3 && !s.melting && Math.random()>0.998) {
			  // ~1/1000 chance of melting mid-air, with each frame
			  s.melting = true;
			  s.melt();
			  // only incrementally melt one frame
			  // s.melting = false;
			}
			if (storm.useTwinkleEffect) {
			  if (s.twinkleFrame < 0) {
			    if (Math.random() > 0.97) {
			      s.twinkleFrame = parseInt(Math.random() * 8, 10);
			    }
			  } else {
			    s.twinkleFrame--;
			    if (!opacitySupported) {
			      s.o.style.visibility = (s.twinkleFrame && s.twinkleFrame % 2 === 0 ? 'hidden' : 'visible');
			    } else {
			      s.o.style.opacity = (s.twinkleFrame && s.twinkleFrame % 2 === 0 ? 0 : 1);
			    }
			  }
			}
		}
	};
###3、执行定时器ergrg
那么，遍历所有的雪花，让她们飘去吧。要么setInterval、要么递归调用setTimeout。

	this.snow = function() {
		var active = 0, flake = null, i, j;
		for (i=0, j=storm.flakes.length; i<j; i++) {
		  if (storm.flakes[i].active === 1) {
		    storm.flakes[i].move();
		    active++;
		  }
		  if (storm.flakes[i].melting) {
		    storm.flakes[i].melt();
		  }
		}
		……
		if (storm.timer) {
		  features.getAnimationFrame(storm.snow);
		}				
	};

这里引入了一个新的专业名词“requestAnimationFrame”。

我们知道，setInterval、setTimeout在实现动画的流畅性上总是不理想。动画比较棘手的问题是延迟应该多少，一方面要必须短，从而使动画流畅地进行，另一方面还要足够长，使得浏览器可以完成渲染。大多数浏览器的刷新频率为60Hz，即每秒60次刷新，那流畅动画的最佳间隔是1000ms/60约为17ms。其次的问题是无法精确，第二个参数指定的延迟表示代码何时会添加到浏览器的UI线程队列中。如果UI线程处于繁忙状态，那代码不会被马上执行。再次，即使看不到网页，或是处于背景选项卡中的页面，动画都会频繁出现，导致过度绘制。

实际上，CSS transitions 和 animations的动画都非常平滑，优势在于浏览器知道哪些动画将会发生。而javascript动画，浏览器不知道动画正在发生。所以一个独特的方案就是创建一个requestAnimationFrame（）方法来告诉浏览器哪些javascript代码正在执行，而计时由系统处理，与浏览器的绘制时间间隔保持一致。此方法接受一个参数，是一个动画函数，并需要在函数最后再次调用requestAnimationFrame（）方法。

requestAnimationFrame（）API是W3C起草的一个新议案，目前chrome10+、firefox 4+、IE10+已支持。上述代码中getAnimationFrame方法针对不同浏览器对requestAnimationFrame进行了封装，的实现如下：

	function timeoutShim(callback) {
		window.setTimeout(callback, 1000/(storm.animationInterval || 20));
    }

    var _animationFrame = (window.requestAnimationFrame ||
        window.webkitRequestAnimationFrame ||
        window.mozRequestAnimationFrame ||
        window.oRequestAnimationFrame ||
        window.msRequestAnimationFrame ||
        timeoutShim);

	// apply to window, avoid "illegal invocation" errors in Chrome
	getAnimationFrame = _animationFrame ? function() {
		return _animationFrame.apply(window, arguments);
	} : null;
最后，总结一下代码结构如下：
![雅虎科技](/img/snowflake.png)

