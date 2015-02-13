title: jQuery事件系统一
date: 2014-10-10 17:16:59
tags: jQuery event 事件 数据缓存 W3C 浏览器
---
说到Javascript事件，脑海中闪现的是最初做简单网页时写个button，注册个onclick事件，就出现了简单的交互。那么onclick，和后续接触到的原生JS中的DOM0、DOM2、addEventListener、attachEvent、jQuery中的bind、live、delegate、on等是什么关系，如何演化的。一个简单的onclick为什么会发展成复杂的jQuery事件系统以及数据缓存系统。今天只是一个引子：

##DOM0级事件系统

	document.getElementById("btn").onclick = function(){
		alert(arguments.length);
	}

这是是最简单的绑定事件的方式。再早之前就是元素标签内绑定事件，不推荐这种样式行为混在一起的写法。

优势：
简单稳定，浏览器兼容；处理事件时，this关键字处理的是当前元素，这很有帮助。
劣势：
1、只允许元素每次绑定一个回调，重复绑定会覆盖之前的绑定。
2、在IE下回调没有参数，在其他浏览器下回调的第一个参数是事件对象。上述代码在IE下弹出0，而在Firefox下弹出1，这个参数就是event对象。
3、只能在事件冒泡中运行，而非捕获或冒泡。

##DOM2级事件系统
IE：

	element.attachEvent("on" + type, callback); //绑定事件
	element.detachEvent("on" + type, callback); //解除绑定
	document.createEventObject(); //创建事件
	element.fireEvent(type, event); //派发事件

优势：
可绑定多个事件，不会覆盖
劣势：
1、回调函数中this不是指向被绑定元素，而是window
2、同种事件绑定多个回调时，回调并不是按照绑定时的顺序依次触发（这还真是头一次听说）
3、event事件对象仅存在于window.event参数中，其属性与W3C的有很大差异，比如currentTarget
4、事件必须以onXXX形式，仅IE可用
5、只支持冒泡

W3C：
	
	element.addEventListener(type, callback, [phase]); //绑定事件
	element.removeEventListener(type, callback, [phase]); //解除绑定
	element.createEvent(types); //创建事件
	event.initEvent(); //初始化事件
	element.dispatchEvent(event); //派发事件

优势：
1、同时支持事件处理的捕获和冒泡阶段，事件阶段取决于参数设置
2、事件处理函数内部，this引用当前元素
3、事件对象总是可以通过处理函数的第一个参数获取
4、可以绑定多个事件

劣势：其他标准浏览器的实现也有不一致的地方，比如firefox不支持focusin、focus事件，第三四五个参数的使用，事件成员对象的不稳定，如safari下event.target可能时返回文本节点。

为了兼容浏览器，我们通常会创建一个统一的方法，在方法内部通过特性检测分别调用不用的事件模型。

	//绑定事件
	function addEvent(el, type, callback, useCapture){
		if(el.addEventListener){//W3C方式优先
			el.addEventListener(type, callback, !!useCapture);
		}
		else{
			el.attachEvent("on" + type, callback);
		}
	}
	//移除事件
	function removeEvent(el, type, callback, useCapture){
		if(el.removeEventListener){//W3C方式优先
			el.removeEventListener(type, callback, !!useCapture);
		}
		else{
			el.detachEvent("on" + type, callback);
		}
	}
	//派发事件
	function fireEvent(el, type){
		if(el.createEvent){
			event = document.createEvent("HTMLEvents");
			event.initEvent(type, true, true);
			el.dispatchEvent(event);
		}
		else{
			event = document.createEventObject();
			el.fireEvent("on" + type, event);
		}
	}
	//延伸
	//阻止冒泡的通用函数
	function stopBubble(e){
		if(e && e.stopPropagation){ //W3C方式
			e.stopPropagation();
		}
		else{
			window.event.cancelBubble = true; //IE
		}
	}
	//阻止浏览器默认行为的通用函数
	function stopDefault(){
		if(e && e.preventDefault){ //W3C方式
			e.preventDefault();
		}
		ele{
			window.event.returnValue = false; //IE
		}
	}

fireEvent与调用onClick的区别：
派发事件fireEvent模拟用户行为触发事件，如触发一个button的onclick事件，如果该button未注册onclick事件也不会报错，并且会引发冒泡，触发其父类中的onclick事件，更贴近用户真实的触发行为。那如果直接调用onclick()呢？如果在未注册onclick事件时调用onclick将会报错“对象不支持此属性或方法”。

##Dean Edward && event.js
鉴于DOM2级事件系统的缺陷，Dean Edward提出了更完美的解决方案，成为jQuery事件系统的源头，看它是不是长了三头六臂，代码来自[http://dean.edwards.name/weblog/2005/10/add-event/](http://dean.edwards.name/weblog/2005/10/add-event/)

	function addEvent(element, type, handler) {
		// 为每一个事件处理函数分派一个唯一的ID，方便移除
		if (!handler.$$guid) handler.$$guid = addEvent.guid++;
		// 为元素的事件类型创建一个空对象，保存所有类型的回调
		if (!element.events) element.events = {};
		// events对象包含多个type/handlers这样的键值对
		var handlers = element.events[type];
		if (!handlers) {
			handlers = element.events[type] = {};
			// 如果元素之前以onXXX的形式绑定过事件，则存储起来
			if (element["on" + type]) {
				handlers[0] = element["on" + type];
			}
		}
		// 保存当前的事件处理函数
		handlers[handler.$$guid] = handler;
		// 指定一个全局的事件处理函数来做所有的工作
		element["on" + type] = handleEvent;
	};
	// 事件处理函数ID计数器
	addEvent.guid = 1;

	function removeEvent(element, type, handler) {
		// 从events对象移除当前事件处理函数/函数类型
		if (element.events && element.events[type]) {
			delete element.events[type][handler.$$guid];
		}
	};

	//统一的事件处理函数入口
	function handleEvent(event) {
		var returnValue = true;
		// 获取原生的事件对象
		event = event || fixEvent(window.event);
		// 从元素的事件对象上获取事件处理函数
		var handlers = this.events[event.type];
		// 遍历执行事件处理函数
		for (var i in handlers) {
			this.$$handleEvent = handlers[i];
			if(this.$$handleEvent(event)===false){
				returnValue = false;
			};
		}
	};
	//为IE的事件对象做简单的修复
	function fixEvent(event) {
	    //添加标准的W3C方法
	    event.preventDefault = fixEvent.preventDefault;
	    event.stopPropagation = fixEvent.stopPropagation;
	    return event;
	};
	fixEvent.preventDefault = function() {
	    this.returnValue = false;
	};
	fixEvent.stopPropagation = function() {
	    this.cancelBubble = true;
	};

特点：
1、没有对象检测，因为使用最通用的原始的onXXX绑定，不使用addEventListener/attachEvent
2、保持正确的this指向
3、传递了正确的event对象
4、完全跨浏览器包括IE4或NS4
5、不会引发内存泄漏（使用者发现onXXX在IE存在不可消弭的内存泄漏）

jQuery在这个版本基础上吸收了“每个处理函数分配一个unique ID，所有回调放到一个对象上存储”的建议，出现了jQuery的数据缓存系统，同时舍弃了onXXX方式，仍然使用addEventListener/attachEvent绑定事件。

