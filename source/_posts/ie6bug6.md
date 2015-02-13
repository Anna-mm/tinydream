title: IE6兼容性专题六——PNG图片的处理
date: 2011-11-18 16:55:29
tags: IE6 兼容 setTimeout setInterval 延迟
---
最近一直在做IE6的兼容，毕竟14.3%的用户是怎么都无法忽略的。而在IE6下网站的丑恶之源就是PNG图片的蓝色背景。尝试了如下解决方案：

##一、CSS解决

背景图片可以通过加入如下CSS解决：

	.png_filter{
		background:url(../img/layer.png) 0 0 no-repeat;
		_background:none;
		_filter: progid:DXImageTransform.Microsoft.AlphaImageLoader(enabled=true, sizingMethod=scale, src="img/layer.png");
	}

如果是图片的话，仍然需要JS来解决。

##二、JS解决

这个相对简单，引入一个外部的JS：DD_belatedPNG_0.0.8a-min.js
在内部JS中加入代码：`DD_belatedPNG.fix('.fixpng')`;
JS将会对所有class为fixpng的图片或背景图片进行处理，可惜这也不是完美的方案。

缺点一：JS是对所有页面加载完成的图片进行处理，如果图片本来设置了width或height需要进行缩放，js处理完成之后就变成了原始大小。以前指定的width或height失效。重新设置大小不起作用。

针对这个问题，要么就是图片不缩放，显示原始大小（IE6下的登录页飞出效果就是这样做的）。要么就是不进行透明处理（加入unfixpng样式，在JS中过滤掉）。

缺点二：实际效果仍然可以用丑陋来形容。

![ie6透明图片](/img/ie6bug61.png)

用户最先看到的是左图，1s-5s后看到的是右图。就好像一个丑女在大家面前化妆，化成一个美女，说：我就是这样变漂亮的。分明是一大忌讳嘛。

三、PNG图片本身

带着这些疑问看看同行们是怎么做到的。用IE6打开点点和人人，从开始到加载完成完全没有蓝色背景，看不出任何CSS或JS处理的痕迹，滴水不漏呀。从人人上找一张PNG图片保存到本地，同样保存一张我们的图片，新建一个静态文件在IE6下显示它们。

![ie6透明图片](/img/ie6bug62.png)

左边是人人网的home_icon.png,完全没有蓝色背景。右侧是我们的timeline_icon_content.png。对点点的图片也做了这样的比较，结果是一样的。它们到底在PNG图片上做了什么处理呢？如果能解决了这个问题，我们就不用加入更多的CSS或JS，让缓慢的IE6雪上加霜了。

解决：png图片保存为索引模式。
