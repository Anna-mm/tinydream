title: 【翻译】Internet Explorer：“别再叫我IE”
date: 2013-07-03 18:00:13
tags: Internet Explorer 11 
---
原文地址：[http://www.nczonline.net/blog/2013/07/02/internet-explorer-11-dont-call-me-ie/](http://www.nczonline.net/blog/2013/07/02/internet-explorer-11-dont-call-me-ie/  at 2013-07-02) by Nicholas C. Zakas

![IE11预览图](/img/ie11.png)

上周，微软发布了针对Windows 8.1平台的Internet Explorer11的预览版，结束了之前关于这个备受争议的浏览器的诸多传言。我们现在知道，Internet Explorer 11有一些非常重要的细节，包括支持WebGL、预读取、预渲染、flexbox（一种布局）、突变观察者（MutationObserver，DOM4事件规范，给开发者提供一种能在某个范围内的DOM树发生变化时作出适当反应的能力）和其他的web标准。但是更有趣的是，虽然被叫为 Internet Explorer 11，但它已经不再是 IE 了。

长久以来，微软第一次实质性地从Internet Explorer中删除功能，更改了user-agent字符串，似乎微软已经离开了自己的轨迹，使得无论是javascript中还是服务器端的isIE()的判断均失效，返回false。比较乐观的看法是，Internet Explorer 11终于足够支持web标准，不再需要之前那些特有的行为了。

##user-agent的变化

在Internet Explorer 11中，user-agent比以前更短，还有一个有趣的变化：

	Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv 11.0) like Gecko

对比之前的：

	Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)

最关键的不同是移除了自古就有的“MSIE”字样，而且尾部附加了“like Gecko”。看来，Internet Explorer更愿意被识别成“Gecko”而不再是它自己。safari是首个标识“like Gecko”的浏览器。因此，所有嗅探“MSIE”的代码将不再有效，你可以继续嗅探“Trident”（在Internet Explorer 9中引入的）来区别Internet Explorer。Internet Explorer版本号出现在“rv”字样的后面。

另外，navigator对象也发生了变化，与现存的浏览器更加混淆不清了。

navigator.appName值为Netscape（之前是"Microsoft Internet Explorer"）

navigator.product值为Gecko（之前没有这个属性）

这像是跟开发人员耍了一个小把戏，但是这种行为确实是HTML5中指定的。navigator.product的属性值一定要是“Gecko”，navigator.appName的属性值或者是“Netscape”或者是其他指定值。奇怪的规定，但Internet Explorer 11却遵守了。此举导致通过navigator判断浏览器型号的javascript将不奏效，而且会把Internet Explorer 11认定为基于Gecko的浏览器。

##document.all 和 friends

从Internet Explorer 4开始，document.all在Internet Explorer中举足轻重。比起document.getElementById()，document.all是”IE式“获得元素引用的方法。尽管Internet Explorer 5 增加了对DOM的支持，document.all一直沿用到Internet Explorer 10。到了11，这个历史遗迹已经被废弃了，意味着基于document.all之上的代码可能会执行失败，尽管document.all现在还是可以工作的。（还没有被移除，但不推荐使用）

另一个遗迹就是用于事件绑定的attachEvent()方法，和取消事件绑定的detachEvent()，这两个方法已经从Internet Explorer 11中移除。那么，像这样的代码，关于attachEvent的那部分已经没有意义了。

	function addEvent(element, type, handler) {
	    if (element.attachEvent) {
	        element.attachEvent("on" + type, handler);
	    } else if (element.addEventListener) {
	        element.addEventListener(type, handler, false);
	    }
	}

当然，推荐你先在标准的浏览器上测试，确保attachEvent()的移除不会对功能造成影响。然而，互联网上充斥着各种垃圾的特性检测代码，在移除attachEvent()的情况下，上述这样不错的代码会确保执行逻辑进入标准模式，而不是IE模式。

##被删除的特性还有：

window.execScript()——IE下的eval()

window.doScroll——IE下的滚动窗口

script.onreadystatechange——IE下监听脚本是否加载完成的方法

script.readyState——IE下脚本的状态

document.selection——IE下获取当前页面上选中的文本

document.createStyleSheet——IE下创建样式表

style.styleSheet——IE下对style标签中样式的引用

所有这些IE特有的特性都应该用相应的标准特性来取代，如果你兼容浏览器的代码使用的是特性检测，那么这些代码将不需改变，继续工作。

##结论

Internet Explorer 11 看起来是Internet Explorer迄今为止最好的浏览器。通过移除了过去的错误，微软已经准备在标准浏览器上占据一席之地了。移除旧有特性、调整user-agent以确保所有网站能够继续工作，这是一个非常大的改变。如果web应用程序使用特性检测而不是浏览器嗅探的话，之前的代码应该能够在Internet Explorer 11中工作。对于服务器端使用的user-agent判断，在Internet Explorer 11对标准的完美支持下，用户也会看到一个功能完整的站点。

不需要针对Internet Explorer写特殊代码的时代已经来临，我非常期待。


