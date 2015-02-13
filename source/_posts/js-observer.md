title: 走向观察者模式
date: 2013-11-13 20:01:04
tags: 设计模式 观察者
---

我们的JS开发模式基本以JQuery为主导，遍布众多插件式规模较小的js文件。在迭代开发中会遇到这样一种情况，有两个本来相对独立的文件A.js、B.js，此时二者需要通信或相互调用。
最简单的办法是，在B中添加一个方法someFunc()，在A中需要调用的地方调用B.someFunc()就行了。缺点是耦合性太高了，举个简单的情况，如果B的开发人员认为someFunc命名不太好修改了一下，谁知道A还在使用，还要去A中再改一遍。

	var A = {
	    funcName:"A",
	    init:function(){
	        …….this.someHappenedHandler();……
	    },
	    someHappenedHandler:function(){
	        B.someFunc();
	    },
	}
	var B = {
	    funcName:"B",
	    someFunc:function(){
	        document.write("调用B的someFunc方法");
	    },
	    init:function(){
	        ……
	    }
	}
	B.init();
	A.init();
那怎么样才能不把someFunc暴露给A呢？这个方法名是由B决定的,由B提前告知并写入A，直到某种情况发生了，A来调用B写入的方法签名？好，尝试一下：

	var A = {
	    funcName:"A",
	    listeners:{}, //用于接收B的写入
	    init:function(){
	        this.someHappenedHandler();
	    },
	    someHappenedHandler:function(){
	        //B.someFunc();
	        this.listeners["someHappened"]();
	    }           
	}
	var B = {
	    funcName:"B",
	    initListeners:function(){
	        A.listeners.someHappened = this.someFunc;
	    },
	    someFunc:function(){
	        document.write("调用B的someFunc方法");
	    },
	    init:function(){
	        this.initListeners();
	    }
	}
	B.init();
	A.init();
一切OK，此时A、B之间的耦合为共同约定了一个字符串“someHappened”用于标识监听的事件签名，以及A的内部属性listeners被B修改。
为了将A的listeners也不暴露出去，可以将上述代码抽象成事件注册、事件处理两个方法。

	var A = {
	    funcName:"A",
	    listeners:{},
	    init:function(){
	        this.someHappenedHandler();
	    },
	    someHappenedHandler:function(){
	        //B.someFunc();
	        //this.listeners["someHappened"]();
	        this.fireEvent("someHappened");
	    }
	}
	var B = {
	    funcName:"B",
	    initListeners:function(){
	        //A.listeners.someHappened = this.someFunc;
	        B.listen(A,"someHappened",this.someFunc);
	    },
	    someFunc:function(){
	        document.write("调用B的someFunc方法");
	    },
	    init:function(){
	        this.initListeners();
	    }
	}
	Object.prototype.listen = function(obj,eventName,handler){
	    //允许注册同名事件，方法签名保存到数组中
	    var arrayHandler = obj.listeners[eventName];
	    if(arrayHandler == null)
	    {
	        arrayHandler = [];
	        obj.listeners[eventName] = arrayHandler;
	    }
	    arrayHandler.push(handler);
	}
	Object.prototype.fireEvent = function(eventName){
	    var arrayHandler = this.listeners[eventName];
	    if(arrayHandler == null){
	        return;
	    }
	  //注册同名事件，顺序执行事件处理程序
	    for(var i = 0; i < arrayHandler.length; i++)
	    {
	        arrayHandler [i]();
	    }
	}
	B.init();
	A.init();
是不是很熟悉？这是常见的浏览器事件处理模式——观察者模式（又名发布订阅模式），类同于

	document.getElementById("XXX").addEventListener(
	"click",function(){
	……
	},false);
这种模式的实质是可以对程序中某个对象的状态进行观察，并且在其发生改变时得到通知。

