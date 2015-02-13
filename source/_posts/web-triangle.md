title: 跨浏览器纯CSS实现的三角之美
date: 2011-11-26 20:40:02
tags: 浏览器 三角 arrow 纯css
---
小小提示框几乎充斥在网站的每个角落，有时候固定显示用以标明网站的新增功能，有的是鼠标滑过显示提示信息，有的只是单纯的指向某一段内容。这种提示框眼熟到都不想看了。

百度知道:

![三角之美](/img/triangle1.png)

提示框在全站中应用很广泛，最简单的莫过于切图实现了。但是缺点也很明显，图片的大小一固定，内容就要受到限制，图片的颜色也会导致重用到其他页面时与当前色调不符。 所以我们要寻求一种跨浏览器的完全不用图片的实现方案。

提示框大致分成两部分箭头和内容框。内容框很容易，设置背景颜色、边框和圆角半径（IE下不支持，可忽略）即可。最重要的是箭头部分。

##（一）无边框正箭头

先看下豆瓣的同城活动：

![三角之美](/img/triangle2.png)

这个灰色的不带边框的小箭头是个图片。那怎么用纯CSS实现这个箭头呢。
CSS代码：

		.triangle{ 
			position:absolute;
			left:80px;
			top:-40px;
			width:0;
			height:0;
			border-right:20px solid transparent;
			border-left: 20px solid transparent;
			border-top:20px solid transparent;
			border-bottom:20px solid #d2d2d2;
		}

简写为
	
	border-width:20px;border-style:solid;border-color: transparent  transparent  #d2d2d2

**特别注意：某些IE6下不兼容，一定要将transparent中的solid改成dashed。简写繁写都没问题。主要背景色不是白色就要注意了。如果有背景色的话，transparent设置成背景色。**

效果图如下：

![三角之美](/img/triangle3.jpg)

不用惊讶，没错，高度和宽度都设为0，只是通过设置border实现的。我们可见想象一个基准点（由于宽度和高度都为0，不能视为盒子，只能视为基准点），上下左右边框均为20px，于是构成了一个正方形，如图上图所示，基准点即为红框的中心。

最关键的是，除了下边框为灰色外，其他边框均为透明。当把红框按照基准点分成四个三角形时，正好显示的是底部三角形。达到预期效果。同理，当我们想做成其他方向的箭头时，就只将那个方向的border设成颜色即可。

还有一点要注意，三角常常是对于下面的内容框绝对定位，定位的起始点是红框的左上顶点，所以，top=-40px，left=80px。

##（二）无边框斜箭头

有时候，我们嫌这种与内容框完全对称的箭头太中规中矩，那怎么做到斜向一边的呢。如图所示。（忽略掉所有红色的线可以看到不规则箭头）

![三角之美](/img/triangle4.jpg)

在之前的代码基础上把右边框由20px改为10px，可以看到正箭头右偏的部分已经有了缺失，底边框填充的部分也缩小到了30px。如果将右边框继续缩小，缩小到0，我们可以想象，变成了直角三角形，直角边在右底部，也就是方框的1/8。

##（三）有边框箭头

其实，这种纯色无边框的提示还是不够明显，常常不能满足我们需求，就像第一张百度的图片那样，浅黄色的背景外加灰色的边框，那能不能做到呢？你可能说，箭头已经是边框来模拟的了，怎么可能再加一层border呢。

点点：

![三角之美](/img/triangle5.png)

仔细看那个箭头，确实没有边框。由于白色和灰色的色差比较小，用户不容易发现，如果是百度的那张图，箭头没有边框那可就很丑了。（可见，使用安全色还是很重要的。可是我们的网站颜色都很不安全，很有视觉冲击力和不规则性，边边角角的要求就更高了。）

其实，这个也很好解决。既然能做一个箭头，就可以做两个。。。怎么样，想到了吧。把两个三角形偏移1px的重叠。OK，如图：

![三角之美](/img/triangle6.png)

结构代码：

	<div class="content" style="border:1px solid #666">
		<div class="triangle"></div>
		<div class="triangle-copy"></div>
	</div>

CSS代码：

	.triangle{
		position:absolute;
		left:80px;
		top:-40px;
		width:0;
		height:0;
		border-right:0px solid transparent;
		border-left: 20px solid transparent;
		border-top:20px solid transparent;
		border-bottom:20px solid #666;
	}

	.triangle-copy{
		position:absolute;
		left:79px;
		top:-38px;
		width:0;
		height:0;
		border-right:0px solid transparent;
		border-left: 20px solid transparent;
		border-top:20px solid transparent;
		border-bottom:20px solid #d2d2d2;
	}

注意看，两个三角形是兄弟关系，不是父子关系（父子结构的样式由于基准点的复杂性很难找出规律）。三角哥哥的20px深灰色边框被三角弟弟20px的浅灰边框覆盖，弟弟的绝对定位相对哥哥下移2px，左移1px。（如果都是1px，无法露出三角哥哥的边框）。

结语：看了国内几家当下很流行的网站，箭头几乎都是用图片做的。只有点点用了CSS样式实现，但也有缺憾。最后的解决方案从JQuery UI Widgets——wijmo中借鉴到的。看来，必须得多看看国外的网站和相关信息了。

