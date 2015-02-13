title: IE6兼容性专题二——消失的绝对定位
date: 2011-11-10 16:11:25
tags: IE6 兼容 绝对 定位 消失
---
这是【个人设置】里选择精灵的页面，直接上图啦!

FF下效果：

![ie6bug](/img/ie6bug21.png)

IE6下效果：

![ie6bug](/img/ie6bug22.png)

赶快找不同啊~~除了弹出层的位置稍稍偏移之外，最重要的右上角【关闭】按钮不见啦。

结构代码：

	<div class="spirit_popbox" style="width:900px; ">
		<div style="width:44px; float:left;"><!--前一个--></div>
		<div style="width:810px; float:left;"><!--精灵列表--></div>
		<div style="width:44px; float:left;"><!--后一个--></div>
		<div><!--解决方案--></div>
		<a style="position:absolute; right:5px; top:5px; background:url('#')" href="javascript:void(0)">
			<!--关闭-->
		</a>
	</div>

行内元素没有设置为块状显示？透明度？z-index？JS处理过？使用文字代替图片？a标签换成div？统统看不见。

终于找到一个不成道理的道理：相对定位（或绝对定位）里的绝对定位挨着浮动元素的时候，而且浮动元素宽度之和等于父元素，或者比父元素少2px的情况下，都会出现绝对等位的元素消失！这个恰恰少了2px。

**解决办法：在浮动元素后加入空层！**
