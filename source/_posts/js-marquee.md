title: 无缝滚动能有多麻烦
date: 2013-11-23 19:49:49
tags: 无缝滚动 marquee
---
实现无缝滚动是前端的必备技能，每次实现也都是小打小闹，这次是在N1、N2、N3的条件下实现，下次在N2、N3、N4的条件下实现，套路大同小异。近期做了个内容动态加载、图文混排的无缝滚动，比以前多了些看点，遂由浅入深，一一道来。

###一、marquee标记

	<marqueen>标记可以实现文字滚动，但是不能实现无缝滚动。
	<marquee direction=left scrollamount=2 onmouseover="this.stop();" 
	onmouseout="this.start();" width=100 height=100>
	    中文为什么就不行
	</marquee>
Direction设置活动字幕的滚动方向，scrollamount设置文字的滚动速度。
onMouseOut="this.start()" ：用来设置鼠标移出该区域时继续滚动
onMouseOver="this.stop()"：用来设置鼠标移入该区域时停止滚动
behavior设置滚动方式，align设置对齐方式等，属性很多，但是致命伤是不能实现无缝滚动。

###二、JS障眼法

仍然以横向滚动为例，通常情况下，显示范围的宽度肯定小于内容的宽度，才有滚动显示的必要。红色框为内容显示框。为了使滚动文字的首尾相接，要使用两个放有相同内容的div。
![跨域请求](/img/marquee1.png)

红色框固定不动，每次当第二个div内容滚动到末尾时，要模拟成第一个div滚动到末尾的情况，这样，第二个div就可以无缝跟上。
![跨域请求](/img/marquee2.png)

模拟成：

![跨域请求](/img/marquee3.png)

结构代码：

	<div id="dvvv" style="overflow:hidden;height:27px;width:100px;
	                      border:1px solid red;white-space:nowrap">
	        <div id="dvvv1" style="display:inline">实现文字无缝滚动。</div>
	        <div id="dvvv2" style="display:inline"></div>
	</div>
JS代码：

	var interval=30;
	var $dvvv = document.getElementById("dvvv");
	var $dvvv1 = document.getElementById("dvvv1");
	var $dvvv2 = document.getElementById("dvvv2");   
	//内容复制
	$dvvv2.innerHTML = $dvvv1.innerHTM;
	function Marqpuee(){
	    //此时滚动到第二个div的末尾处，100为显示宽度
	    if($dvvv.scrollLeft >= 2 * $dvvv1.offsetWidth - 100 ){ 
	//障眼法来啦！！模拟成第一个div滚到到末尾处。瞬间发生，人眼不太能感受到这种变化
	$dvvv.scrollLeft = $dvvv1.offsetWidth - 100;
	    }
	     else{  
	        $dvvv.scrollLeft++;
	     }
	}
	var MyMar = setInterval(Marqpuee,  interval);

###三、实战

前面都是热身，这时来了一个实时新闻页面，要求向下滚动播放，并不时从后台推送新的战况，如下：
![跨域请求](/img/marquee4.png)

####1、稍微复杂的图文混排新闻块如何进行DOM拷贝实现障眼法？

假设，页面中固定显示区域为10个block，那初始加载可能为20个block，那如果将20个block都进行DOM拷贝，无疑给页面增大了负荷。那么，我们可以只拷贝前10个block，等待20个block都滚动一遍之后，并没有新的数据载入时，继续滚动这10个拷贝，留出时间让前20个block复位。
![跨域请求](/img/marquee5.png)
初始化
![跨域请求](/img/marquee6.png)
滚动到拷贝数据，复位

####2、如何在不刷新页面的情况下，显示后台推送的新闻？（前端可以设置定时器，定时请求新数据）

考虑一下，新数据应该插在什么位置？显然，拷贝数据是每一轮滚动结束后才显示，以供原始数据复位，那么新数据应该插在拷贝数据之前。又有新问题了，如果刚好滚动到拷贝数据，那用户肯定能看到新数据的插入，页面就不平滑了。所以插入要寻找时机，插入点一定处于可视区域之外：

	// scrollNum:底部滚出数
	// sum:总数量，包括拷贝数
	// visibleNum = bakNum:可视数=拷贝数
	if(scrollNum < sum - visibleNum * 2){
	    //append html
	}
	else{
	    setTimeout(function(){
	            //append html;s
	    },marqueeTimer.interval * visibleNum)
	}
当插入点处于可视区域时，等待拷贝数据滚动完成之后再插入。但要考虑拷贝数据滚动完成所需时间内，没有新的数据推送到前端。否则，需要增加缓冲池缓存数据。

####3、每次插入数据后，还要及时修正容器的scrollTop或margin-top，保证不跳动。

