title: IE6兼容性专题七——0ms延迟
date: 2011-11-20 17:00:20
tags: IE6 兼容 setTimeout setInterval 延迟
---

在IE6下处理透明图片后，“回到顶部”不见了。尝试着延迟处理确实可以解决问题，可是事实上不需要延迟，那就干脆0ms延迟吧，也同样达到目的。网上搜索之后才发现这已经成为js处理的“潜规则”之一了，也一度成为解决了很多前端疑难杂症的法宝，准确地说，不是0ms，而是16ms延迟。

>setTimeout的存在会导致CPU中断调度，如果两个中断之间时间太短会导致CPU性能消耗很高，同时影响能耗。于是微软和英特公司为了解决这个问题，就约定每个中断之间的间隔是15.6ms（64 fps）所以就是我们常见的约等于16ms的间隔，不过随着web的要求不断增加，大家对这个要求希望是放宽的态度，于是在高端浏览器，这个性能被提升了4倍左右，所以在chrome，IE10等浏览器，setTimeout的间隔缩短到了 4ms（250 fps）。

以下是透明图片处理的公用方法：

	//给所有PNG图片加上样式”fixpng”
	$("img[src$='.png'],img[src$='.PNG']").not($("#unfixpng")).addClass("fixpng");

	setTimeout(function(){
		//遍历所有div、a标签，当其有背景图片且背景图片为PNG图片时加上样式”fixpng”
		$("div,a").each(function(){
			var b = $(this).css("background-image");
			if(b!=undefined && /.png|.PNG/.test(b)){
				$(this).addClass("fixpng");
			}
		})
	},0)

另外一个用到此方法的实例是：礼尚往来，点击我收到的礼物，选择全选后，选中框不见了。

![ie6bug](/img/ie6bug71.png)

代码结构：

	<a class="all_button" href="javascript:void(0);" id="present_check_mark">全选</a>

样式：

	.all_button {
	    background: url("/skin/message/images/all_nochecked.png") no-repeat scroll right center transparent;
	}	

Checkbox框是一张背景图。当点击之后，更换为另外一张选中后的背景图。
JS代码：

	jQuery.fn.extend({
			//全选/全不选
			chooseAll:function(){
				return this.click(function(){
					var chooseFlag = $(this).hasClass("all_checked");
					if(chooseFlag){
						$(this).removeClass("all_checked");
					}else{
						$(this).addClass("all_checked");
		}								 $("input[type='checkbox'][name='presentIds']").each(function(){
						$(this).attr("checked", !chooseFlag);
					});
				});
	});

这个bug很不容易再现，它不是每次点击都消失，而且一旦checkbox出现再也不消失了。除非你关闭浏览器再重新打开，相当恶心。调试发现，如果在加入样式”all_checked”前随便alert个什么东西，问题就解决了。所以就想到了

	var that = $(this);
	setTimeout(function(){
		that.addClass("all_checked");
	},0);

不过，还要注意一点，setTimeout中的this指向的是window，不再是单击的事件源了，要先将$(this)保存一下。

总结：
1. 正常情况下javascript都是按照顺序执行的。但是我们可能让该语句后面的语句执行完再执行本身，这时就可以用到setTimeout延时0ms来实现了。
2. 在事件中，setTimeout 会在其完成当前任何延宕事件的事件处理器的执行，以及完成文档当前状态更新后，告诉浏览器去启用 setTimeout 内注册的函数。