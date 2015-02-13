title: IE6兼容性专题三——经典的内容复制bug
date: 2011-11-12 14:33:32
tags: IE6 兼容 莫名其妙 变态
---

##举例1: IE6注释bug
几个相连的浮动元素中间插入一段html注释代码，就会将最后一个容器中的最后几个字符复制到容器之外显示出来。俗称“IE6注释bug”。如果添加更多条的注释，会继续余出更多字符。

![内容复制bug](/img/ie6bug31.png)

结构代码

	<div class="left">浮动元素1</div>
	<div class="left">浮动元素2</div><!--注释-->
	<div class="left">浮动元素3</div>
	<div class="left">浮动元素4</div>

解决方案：去掉注释语句。

##举例2：流浪在外的“应”

图1：
![内容复制bug](/img/ie6bug32.png)

图2：
![内容复制bug](/img/ie6bug33.png)

图3：
![内容复制bug](/img/ie6bug34.png)

图1结构代码：

	<div class="tml_postcontrol" style=”float:right; width:300px;”>
		<input type="image" alt="如果您喜欢这条飞鸽，请点亮这颗心。" class=”right ml15” src=”#” />
	 	<a href="#" class=”right ml15”>回复</a>
	 	<a href="#" class=”right ml15”>重飞</a>
	 	<a href="#" class=”right ml15”>收藏</a>
	 	<a href="#" class=”right ml15”>
	 		<span class=”right”>0条回应</span>
	 	</a>
	</div>

为了方便分析，特别给每个部分加上边框。可以看到，图1多出了一个莫名其妙的“应”，并且不属于任何一个部分。图2少了一项内容，显示正常。图3增加了一项内容，多出来两个字“回应”。

解决方案1：**由于每个部分都是右浮动，很容易想到可能是没有设置width引起的，尽管图中对没有width的浮动元素的宽度渲染的很好，尽管图2也没有width仍然显示正常。但是，给“回应”的标签设置`{_width:100px;}`的确解决了问题。**

解决方案2：**“0条回应”外有和两层标签，而且都是右浮动。这个结构很有意思，它会导致每增加一个浮动项目都会多出来一个字。删掉或的右浮动可以解决。或者在任意个右浮动项目中外套div(除`<span>`外)。**

##举例3：多出一行的”立方偷袭”

![内容复制bug](/img/ie6bug35.png)

结构代码：

	<div>
		<a href="#" class=”left”>第九维<a>
		<a href="#" class=”left”>头像买卖<a>
		<a href="#" class=”left”>礼尚往来<a>
		<a href="#" class=”left”>立方PK<a>
		<a href="#" class=”left”>心理测试<a>
		<a href="#" class=”left” style=”margin-right:-3px;”>立方偷袭<a>
	</div>

解决方案：在最后一个浮动元素中添加负边距样式。

结语：无语，莫名其妙，变态


