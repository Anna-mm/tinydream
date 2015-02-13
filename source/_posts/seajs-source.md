title: seajs源码解析
date: 2014-03-17 13:53:07
tags: seajs源码
---
关于seajs的源码解析玉伯自己也只写了两篇[http://chuansongme.com/n/124571](http://chuansongme.com/n/124571)、[http://chuansongme.com/n/126184](http://chuansongme.com/n/126184)，去年订阅他的微信号后就已拜读过。讲述了目录结构、大闭包等这些表面的细节，还未真正写到精髓，就没有了。最近项目中应用了seajs，模块化改造也并不复杂，腾出时间专门看看源码。
下载个seajs，src目录如下：

	src
	|----intro.js             -- 全局闭包头部
	|----sea.js               -- 基本命名空间

	|----util-lang.js         -- 语言增强
	|----util-events.js       -- 简易事件机制
	|----util-path.js         -- 路径处理
	|----util-request.js      -- HTTP 请求
	|----util-deps.js         -- 依赖提取

	|----module.js            -- 核心代码
	|----config.js            -- 配置
	|----outro.js             -- 全局闭包尾部

Dist目录中的sea-debug.js就是上述文件按顺序合并而成，未压缩代码行900+，按模块回顾下吧。

（一）intro.js，只是一个包装而且是个上包装，下包装见outro.js。

	(function(global, undefined) {
    	if (global.seajs) {
        	return
    	}

jQuery的包装器基本也是这样：

	(function(window, undefined) {
    	var jQuery ＝  ...
    	...
    	window.jQuery = window.$ = jQuery;
    })(window);

注意四点：
1、自调用匿名函数创建了一个闭包，该作用域中的代码不会破坏和污染全局变量，这是任何一个javascript库所必备的功能。
2、jQuery传递是window参数，seajs传递的是global参数，作者说，“在浏览器环境中，global 是 window 对象。在 Node.js 环境中，global 则是 node 环境中的 global 对象。这是一个跨平台的兼容式写法。”
3、传入undefined参数是为了确保undefined的值是undefined，避免被低版本浏览器通过window.undefined＝“”这样的赋值语句重写。
4、传入window、undefined参数有一个共通原因是可以使其变为局部变量，这样在jQuery代码块中访问它时，不需要将作用域回退到顶层作用域，更快地访问到window对象。同时局部变量可以进行压缩优化，压缩为一个字符，也省不少字节。


（二）sea.js，确保全局环境中只有一个seajs，events、。。。等均保存在data中。

	var seajs = global.seajs = {
		// The current version of Sea.js being used
		version: "2.1.1"
	}
	var data = seajs.data = {}

（三）util-lang.js

	function isType(type) {
		return function(obj) {
			return Object.prototype.toString.call(obj) === "[object " + type + "]"
		}
	}
	var isObject = isType("Object")
	var isString = isType("String")
	var isArray = Array.isArray || isType("Array")
	var isFunction = isType("Function")

	var _cid = 0
	function cid() {
		return _cid++
	}

先举例说明一下如何做类型检测：

	var a = "aa";
	console.log(Object.prototype.toString.call(a)) //[object String]
	var a = 123;
	console.log(Object.prototype.toString.call(a)) //[object Number]
	var a = ["aa"];
	console.log(Object.prototype.toString.call(a)) //[object Array]
	var a = function(){};
	console.log(Object.prototype.toString.call(a)) //[object Function]
	var a = {};
	console.log(Object.prototype.toString.call(a)) //[object Object]

原来这么简单，将变量转变成字符串就可以打印出其类型。有没有觉得isType函数有些特别，接受一个type参数，其内部返回一个接受obj参数的匿名函数。为什么不直接定义一个isType函数，传递type、obj两个参数，判断此obj是不是type类型，多好理解。哎，不要乱想，这可是javascript高级技巧——柯里化，柯里化就是把接受多个参数的函数换成接受单一参数的函数，并返回接受余下的参数的新函数的技术。以后写代码也试试这酷炫吊的技巧。

（四）util-events.js,简单事件机制包括绑定、解绑、触发。

	var events = data.events = {}
	seajs.on = function(name, callback) {
		var list = events[name] || (events[name] = []);
		list.push(callback);
		return seajs;
	}
	seajs.off = function(name, callback) {}
	var emit = seajs.emit = function(name, data) {}

1、将事件对象保存到seajs.data.events中；
2、每次绑定事件时在seajs.data.events中创建name/callback键值对保存事件的宿主元素和事件处理函数。
3、实际应用中主要使用seajs的模块化组织方式，未使用事件绑定功能

（五）util-path.js

	var doc = document
	var loc = location
	var cwd = dirname(loc.href)
	var scripts = doc.getElementsByTagName("script")

	// 得到最后一个script标签
	var loaderScript = doc.getElementById("seajsnode") ||
	    scripts[scripts.length - 1]
	// 最后一个script标签的请求路径，不包括文件名
	var loaderDir = dirname(getScriptAbsoluteSrc(loaderScript) || cwd)
	// 得到指定标签的全路径
	function getScriptAbsoluteSrc(node) {}
	//提取路径名，如dirname("a/b/c.js?t=123#xx/zz") ==> "a/b/"
	function dirname(path) {}
	function realpath(path) {}
	function normalize(path) {}
	function parseAlias(id) {}
	function parsePaths(id) {}
	function parseVars(id) {}
	function parseMap(uri) {}
	function addBase(id, refUri) {}
	function id2Uri(id, refUri) {}

	// 对外接口
	seajs.resolve = id2Uri

(六)util-request.js

	//创建<link>或<script>标签，预加载后插入到页面中。
	function request(url, callback, charset) {}
	function getCurrentScript() {}

（七）util-deps.js，只有一个函数，用于提取每个模块中require的文件

	function parseDependencies(code) {}

（八）module.js 模块化的核心策略

	seajs.use = function(ids, callback) {
		//加载config中的预加载项,如jQuery
		Module.preload(function() {
			// ids：依赖模块或是模块本身，第三个参数为模块uri，如http://www.l99.com/_use_4，为什么要拼凑这样一个uri。
			Module.use(ids, callback, data.cwd + "_use_" + cid())
		})
		return seajs
	}
	//获取模块缓存或创建一个新的模块
	Module.use = function (ids, callback, uri) {}
	//加载模块、并设置相应的状态、_waitings、_remain
	Module.prototype.load = function() {}
	//得到最终的调用模块的uris
	Module.prototype.resolve = function() {}
	//通过id和refUri得到最终的uri，如"dashboard/dashboard_inset_header" + 
	//"http://www.l99.com/_use_1"=> //"http://www.l99.com/jscmd/dashboard/dashboard_inset_header.js?v=20140305"
	Module.resolve = function(id, refUri) {}
	//模块的构造函数
	function Module(uri, deps) {
		this.uri = uri
		this.dependencies = deps || []
		this.exports = null
		this.status = 0
		// Who depends on me
		this._waitings = {}
		// The number of unloaded dependencies
		this._remain = 0
	}

。。。
这个地方还有很多实质内容没有分析，今天先到这，专注不下去了。
。。。

（九）outro.js，将this传入闭包体global，在浏览器环境中global 是 window 对象。在 Node.js 环境中global 则是 node 环境中的 global 对象

	})(this);




