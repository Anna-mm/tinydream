title: IE6兼容性专题四——div元素不能覆盖select元素
date: 2011-11-15 14:14:26
tags: IE6 兼容 select 划破 iframe
---
在IE6下，当div下方有下拉列表框select元素的时候，下拉列表框会划破div显示在div之上，不论z-index设为何值均会出现此问题，可能由于下拉列表控件的弹出式下拉列表的原因导致Z轴高度失控。但是IE6还有一个逻辑，div 无法覆盖select,但是iframe 可以覆盖select，而div可以覆盖iframe。

**所以解决办法就是用Z轴高度更高的Iframe元素，（包裹或）覆盖住下拉列表框控件，使其回到正常的Z轴高度上来！**

举例：发起PK时选择好友的弹出层被select划破

![ie6bug](/img/ie6bug41.png)

结构代码：

	<!—弹出层-->
	<div style="width: 525px; " id="friend_pop_wrap">
		<!—弹出层内容省略-->
		<!—用于覆盖背后层的select元素-->
		<iframe class="fixCrossIframe" frameborder="0"></iframe>
	</div>

在弹出层的直接子元素中加入iframe，并设置样式为：

	.fixCrossIframe{
		position:absolute;
		top:-10px;/*考虑到10px的边框设置起始位置为-10px*/
		left:-10px;
		width:545px;/*如果父元素显式定义了width和height，并且没有border，可简单写为100%
		height:565px;
		z-index:-1;/ *-1保证iframe显示在div下方*/
		border:none; /*去掉iframe的默认边框,无法去掉ie6下左上3D边框，在html中加入frameborder=0*/
	}

据之前的描述，可使用iframe包裹或覆盖select元素，刚才使用的是覆盖的方法。我们再看网上介绍的包裹的方法

	<iframe style="z-index:1" id=”fixIframe”> 
		PK时间：
		<select name="challenge.chaTime">
			<option value="1">一天</option>
			<option value="2">二天</option>
			<option value="3">三天</option>
		</select>
	</iframe>

Iframe中不允许直接写入HTML元素，会被吃掉（通过firebug可以看到iframe中的body中没有任何内容）。所以采用JS动态设置iframe的内容，代码如下:

	window.open("","fixIframe","")
	fixIframe.document.write('PK时间：'+ 
		'<select name="challenge.chaTime">'+
		'<option value="1">一天</option>'+
		'<option value="2">二天</option>'+
		'<option value="3">三天</option>'+
		'</select>');

实践证明，这种方案并没有解决问题。

那么干脆引入带有select的网页。下图是引入淘宝首页，定位到右侧的“淘宝旅行”。

![ie6bug](/img/ie6bug42.png)

点击选择好友，弹出层仍然覆盖不了select元素。

![ie6bug](/img/ie6bug43.png)

至此证明此方案无效。确实是个人尝试后的结果，如有问题，欢迎指正。