title: jQuery事件系统二
date: 2014-10-11 18:01:11
tags: jQuery event 事件 数据缓存 W3C 浏览器
---
jQuery事件系统并没有将事件监听函数直接绑定到DOM元素上，而是基于数据缓存系统管理监听函数的。举一个简单的栗子看一下何为数据缓存系统：

	<div class="box">
		<div class="content"></div>
	</div>
	<input type="button" name="submit" value="click me" class="btn" />

	<script type="text/javascript">
		$(".box").click(function(){
			console.log("box");
		})
		$(".content").click(function(){
			console.log("content click 1");
		})
		$(".content").click(function(){
			console.log("content click 2");
		}) 
		$(".content").hover(function(){
			console.log("hover in");
		},function(){
			console.log("hover out");
		}) 
		$(".btn").click(function(){
			console.log("btn");
		}) 
	</script>

如上述代码所示，页面上有一对父子元素box、content和一个按钮btn。
* 父元素box绑定了click事件
* 子元素content绑定了两个click事件，一个hover事件
* btn元素绑定了一个click事件。

我们看一下chrome下面打印jQuery.cache是什么情况：
![jQuery事件系统](/img/jsevent1.png)

由上图所示，
* jQuery.cache中存储了三个以ID为key值的对象，三个ID是为页面中的box、content、btn元素分配的唯一ID。
* 每个object中存储了events和handle两个对象。
* events对象存储了click、mouseout、mouseover三个事件对象，click对象的值是一个存储了两个元素的数组，对应content元素的两个click事件和hover事件。
* click[0]的数组元素中存储了一个guid，唯一标识事件处理函数的ID，以此类推jQuery.cache[1].events.click[0].guid == 1, jQuery.cache[2].events.click[1].guid == 3；存储了一个type＝“click”；存储了一个事件处理函数handler，参见上图底部，是content元素绑定得第一个click事件的处理函数。
* 那回到最外层，每个object的直接子元素handle是干什么用的？通过打印的内容可见，其方法体内调用了jQuery.events.dispatch事件。事实上，每个object对象都有一个handle作为入口监听函数，当浏览器触发事件时，入口监听函数被调用，该函数从事件缓存对象events中获取绑定到此元素上的对应的事件处理函数，然后执行。


问题来了，jQuery.cache中没有存储任何与页面元素有关的信息，其中的1、2、3是如何对应到DOM元素的？一定在什么地方给DOM元素设置了自定义属性存储了其ID值。

![jQuery事件系统](/img/jsevent2.png)

如上图所示，代码来源于jQuery.event.add方法，给派发函数传递的第一个参数当前的DOM元素已经有一个自定义属性jQueryXXXX==2，接下来寻找何处设置了这个自定义属性。

	jQuery.event = {

		add: function( elem, types, handler, data, selector ) {

			var elemData, eventHandle, events,
				t, tns, type, namespaces, handleObj,
				handleObjIn, handlers, special;

			//通过jQuery._data给当前DOM元素设置自定义数据
			if ( elem.nodeType === 3 || ... || !(elemData = jQuery._data( elem )) ) {
				return;
			}
			......
		}
	}

	jQuery.extend({
		data: function( elem, name, data, pvt) {

			var thisCache, ret,
				internalKey = jQuery.expando, //这个自定义属性来自与expando，这是什么东西？？
				getByName = typeof name === "string",

			if ( !id ) {
				// 给DOM元素设置一个unique ID，以获取全局的数据缓存对象中的对应数据
				if ( isNode ) {
					elem[ internalKey ] = id = jQuery.deletedIds.pop() || ++jQuery.uuid;
				} else {
					id = internalKey;
				}
			}
			......
		},
		_data: function( elem, name, data ) {
			return jQuery.data( elem, name, data, true );
		},
	},

好，我们知道了DOM元素的ID与jQuery.cache中的对象ID是一一对应的，在派发函数dispatch中通过种对应关系就能方便地找到当前DOM元素所绑定的事件从而执行。不过，顺着这个线索也遗留了一些问题，比如jQuery.expando是什么？jQuery.data除了DOM ID还设置了那些属性？事件派发函数中对于子元素的click事件冒泡到父元素是如何处理的？且听下回分解。

