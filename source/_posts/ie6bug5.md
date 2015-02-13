title: IE6兼容性专题五——正确使用return false
date: 2011-11-17 22:16:24
tags:
---
在IE6下，进入个人时空的照片列表，点击“删除”，确认后页面没有反应。

	<a class="photo_list_delete" href="javascript:void(0);">删除</a>
	<form id="photoform" method="post" action="http://www.l99.com/EditPicture_deletePicture.action">
		......
	</form>

	$(".photo_list_delete").bind("click", function(){
		if(confirm('确认删除?')){
			$("#photoform").submit();
			return false;//解决方案
		}
	});

分析：执行submit语句后，继续执行href指定的跳转。尽管它并没有指定地址，void是一个操作符，会计算一个表达式，但不会返回值，void(0)就等于0，意思是不进行任何操作。所以IE6停住，不进行跳转，但照片已经删除了。 

方案一：加入return false，阻止默认的事件行为。
方案二：改为href=”#”,但页面也会回到顶部（注：”#”等同于#top,回到顶部，相当于浏览器中输入地址回车后的操作。）或者去掉默认事件，`href="javascript:void(0);"`，但需要额外加入样式`{cursor:pointer;}`，并且页面也会滚到起始位置。这种方案不采用。

正确使用return false，以下内容总结自[http://fuelyourcoding.com/jquery-events-stop-misusing-return-false/](http://fuelyourcoding.com/jquery-events-stop-misusing-return-false/)

return false实际上做了三件事情：
  1. `event.preventDefault();（或window.event.returnValue=false）`
  2. `sevent.stopPropagation();（或window.event.cancelBubble=true）`
  3. 停止执行立即返回

我们本想只阻止默认事件，但是它又额外地阻止了冒泡，并且立即返回了。举例说明这种用法可能出现的问题：
	
	<div class="post">
		<h2><a href="/path/to/page">My Page</a></h2>
		<div class="content">
			Teaser text...
		</div>
	</div>
	<div class="post">
		<h2><a href="/path/to/other_page">My Other Page</a></h2>
		<div class="content">
			Teaser text...
		</div>
	</div>

要求点击某一篇文章标题的时候，将内容加载到当前的“div.content”中。

	$("div.post h2 a").click(function () {
    	var a = $(this),
        	href = a.attr('href') ;
        	content  = a.parent().next();
		content.load(href + " #content");
		return false;
	});

至此一切顺利，如果我们继续要求加载内容之后，当前的div.post要增加样式.active,于是写道：

	var posts = $("div.post");
	posts.click(function(){
	    posts.removeClass("active");
	    $(this).addClass("active");
	});

这样就能正常work了么？不！这个click事件将不会被触发，因为a的click事件已经阻止了冒泡。如果加入live事件，情况将会变得更复杂。

总结：当我们确保在正确的位置上阻止默认事件和冒泡事件时可以使用return false，不要滥用避免出现问题。那最开始讲到的问题，有第三种完美版解决方案就是在IE6下直接阻止默认事件，
	
	window.event.returnValue = false;
