title: 单页应用中页面转场的实现
date: 2014-02-06 15:42:46
tags: jquery mobile SAP 转场动画
---
在上上篇文章《webapp立方没体诞生之初》中提到，移动框架只使用了Boostrap，但后来迫于页面转场效果，自定义build了jquery mobile的Animated page Transitions部分功能，60多k的代码也让人唏嘘。于是，剖析源码，尝试简化。

![页面转场](/img/pageTransition1.png)

在看源码之前思考一下自己如何实现：
1. 页面切换效果：如何让上一个页面移开，让下个页面进入
2. 当上个页面的滚动条向下滚动一段距离后，如何平滑地移动到下个页面的顶部。
3. 浏览器中如何回退？

###一、 页面初始化和事件绑定
**HTML：**

		<div data-role="page" id="page_content" class="">
		    ......
		    <a class="p_notes left" href="#page_notes" data-transition="slide">8条回应</a>
			......
		</div>
		<div data-role="page" id="page_notes" class="lst_notescontainer">
		    ......
		</div>
**Jquery.mobile.custom.js：**

![页面转场](/img/pageTransition2.png)

页面初始化中主要做了以下事情，暂不涉及hash存储：
1. 记录所有pages对象(含有data-role="page"的DOM)；
2. 将第一个page设为当前page；
3. 让当前page入场。

代码简化如下：

	var PageTransition = (function() {
	    var $pages; //记录所有page对象
	    var pageArray = []; //记录所有pageId
	    return {
	        init: function(){
	            $pages = $("*[data-role='page']");
	            $("body").addClass("ui-mobile");
	            for(var i = 0; i < $pages.length; i++){
	                pageArray.push($pages.eq(i).attr("id"));
	                //标记首个page为当前活动page
	                if(i == 0){
	                    $pages.eq(i).addClass("ui-page-active");
	                }
	            }
	        }
	}
	}());

页面初始化完成后，向锚点绑定事件来完成页面转场：
![页面转场](/img/pageTransition3.png)
那这个神秘的changePage方法是如何实现的?

###二、页面转场的外衣

辗转经历各种call之后进入change方法，别以为这就登堂入室了，还要一层层剥开迷雾~~
![页面转场](/img/pageTransition4.png)
对_cssTransition望文生义，难道是CSS3的transition实现的？那岂不是对上下页add或remove 相应动画的class即可。耶！确实如此，不过还得耐得性子读一读这像老太太裹脚布一样的代码~~~

###三、页面转场的核心代码

![页面转场](/img/pageTransition5.png)

**CSS:**

	@-webkit-keyframes slideinfromright {
	    from {-webkit-transform: translate3d(100%, 0, 0);}
	    to {-webkit-transform: translate3d(0, 0, 0);}
	}

	@-webkit-keyframes slideouttoleft {
	    from {-webkit-transform: translate3d(0, 0, 0);}
	    to {-webkit-transform: translate3d(-100%, 0, 0);}
	}
	.slide.out,.slide.in {
	    -webkit-animation-timing-function: ease-out;
	    -webkit-animation-duration: 350ms;
	    -moz-animation-timing-function: ease-out;
	    -moz-animation-duration: 350ms;
	    animation-timing-function: ease-out;
	    animation-duration: 350ms;
	}
	.slide.in {
	    -webkit-transform: translate3d(0, 0, 0);
	    -webkit-animation-name: slideinfromright;
	    -moz-transform: translateX(0);
	    -moz-animation-name: slideinfromright;
	    transform: translateX(0);
	    animation-name: slideinfromright;
	}
	.slide.out {
	    -webkit-transform: translate3d(-100%, 0, 0);
	    -webkit-animation-name: slideouttoleft;
	    -moz-transform: translateX(-100%);
	    -moz-animation-name: slideouttoleft;
	    transform: translateX(-100%);
	    animation-name: slideouttoleft;
	}
	.ui-page-pre-in {
	    opacity: 0;
	}
	.ui-page-active {
	    display: block;
	    overflow: visible;
	    overflow-x: hidden;
	}
不得不用文字描述一下这个过程，尽管繁琐得要命。
![页面转场](/img/pageTransition6.png)

上图为控制台输出，翻译后意思是：
1：下页藏到最后面（z-index=-10）
2：下页显示又隐藏（display:block && opacity=0）
3：下页不用藏到最后面了，原来在哪就在哪
4：下页显示，开始入场
5：当前页离场
6：动画完成后当前页成为了上页隐藏、并移除动画样式
7：动画完成后下页成为了当前页移除动画样式。
真是搞不懂1、2、3在墨迹啥。

自行实现后代码如下：
![页面转场](/img/pageTransition7.png)
文章开头提出了三个问题，至此才说清楚一个。真是想得太远，走得太慢啦。Zepto.js号称是jquery mobile的简化版，有时间再看看它是怎么实现的吧！