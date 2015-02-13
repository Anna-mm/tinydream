title: 悬停事件的常用方案
date: 2011-08-09 18:20:10
tags: 悬停 hover mouseover
---
经常遇到悬停事件的问题，比如弹出层不断隐藏显示，IE下的闪动等，很头疼。这与div结构和事件机制都有很大关系。故浅显地总结一下解决方案，供大家参考。

##举例一：包含结构（父子结构）
鼠标移动到右侧头像处，显示弹出菜单。

![悬停事件](/img/hover1.png)

结构代码：

	<div id="head_setting_item" >
	<!—头像-->
	<a id="nav_avatar" href="#"><img src="#"/> </a>
	<!—弹出层-->
	<span id="head_setting_menu" style="display: none;">
		<a href="#">◀ 我的飞鸽</a>
		<a href="#">上传头像</a>
		<a href="#">个人设置</a>
		<a href="#">充值中心</a>
	</span>
	</div>

原始的事件代码：

	$("#head_setting_item").mousemove(function(){
		$("#head_setting_menu").show();
	});
	$("#head_setting_item").mouseout(function(){
		$("#head_setting_menu").hide();
	});

可以看到，mouse事件是绑定在外层div上，当从外层div移动到弹出层时，会先触发外层div的mouseout事件，再触发其mouseover事件，这种情况在ie6下会出现弹出层闪动。况且，个人认为，mousemove更多应用在拖拽事件中，此处用不合适。

解决方案：

	$("#head_setting_item").hover(function(){
		$("#head_setting_menu").show();
	},function(){
		$("#head_setting_menu").hide();
	});

Jquery的hover模仿悬停事件，并且解决了mouseout事件中移动到子元素上仍然触发移出事件的错误。

##举例二：非包含结构
在处理悬停事件时尽可能使用上述的包含结构，简单无闪动。但实际上有些情况，并不能做成包含结构，比如事件源是`<a>`标签，其内部不能加`<div>`标签（这一点在HTML5中得到支持）。

![悬停事件](/img/hover2.png)

结构代码：

	<div>
		<!—-主题缩略图-->
		<a title=”点击预览主题” class=”prf_onelist” ><img src=”#”/></a>
		<span>默认主题</span>
		<a href=”#”>预览</a>
		<a href=”#”>使用</a>
		<a href=”#”>免费</a>
	</div>
	<!—-主题简介-->
	<div class=”mousebox” style=”display:none”></div>

事件代码：

	$(".prf_onelist").hover(function(){
		//鼠标悬停在主题缩略图上，显示弹出层
		$(". mousebox ").slideDown("fast");
	},function(e){
		//鼠标离开主题缩略图，当前往弹出层时终止事件。否则隐藏弹出层。
	if($(".mousebox")[0].contains(e.relatedTarget)){ return;}
		$(". mousebox ").slideUp("slow");
	});
	$(".mouseBox").hover(function(){
		//鼠标悬停于弹出层时不作处理
	},function(){
		//鼠标离开弹出层时隐藏
	$(". mousebox ").slideUp("slow");
	})

	//利用回溯为HTML元素定义是否包含某个特定元素的实例方法。
	if(typeof(HTMLElement)!="undefined"){
		HTMLElement.prototype.contains=function(obj){
			while(obj!=null&&typeof(obj.tagName)!="undefind"){ 
				if(obj==this) return true;
				obj=obj.parentNode;
			}
			return false;
		};
	}
