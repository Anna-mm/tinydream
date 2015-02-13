title: 网页头部向下滚动隐藏向上滚动显示
date: 2014-05-29 13:45:56
tags: 向上滚动 向下滚动
---
这种效果在原生APP中处处可见。向下滚动时意味着用户在寻求更多的内容，同时手机端可视区域有限，故将头部隐藏以展示更多的内容真是一件体验美好的事；向上滚动时，用户可能在寻找其他操作，故显示头部。这本身也是响应式设计的体现。今天，我们尝试把这种效果迁移到web端。

**首先、头部固定**
	
	body {
	  padding-top: 80px; // 头部高度固定时CSS中设置
	}
	header {
	  background:  #062D52;
	  position: fixed;
	  top: 0;
	  transition: top 0.2s ease-in-out;
	  width: 100%;
	  box-shadow: 0 3px 3px 0 rgba(6, 45, 82, 0.5); 
	  z-index: 999;
	}
	.nav-up {
	  top: -80px; //头部高度固定时CSS中设置. 
	}


**其次、滚动事件**

	var $header = $("header");
    var headerHeight =  $header.height();
    var lastScrollTop = 0;
    var delta = 5;
    var didScroll;
    //$("body").css({"padding-top": headerHeight});头部高度不固定时JS中设置
    $(window).scroll(function() {
        didScroll = true;
    })
    setInterval(function() {
        if (didScroll) {
            scrollHeaderStickyEvent();
            didScroll = false;
        }
    }, 250);

    function scrollHeaderStickyEvent(){
        var st = $(this).scrollTop();
        // 滚动距离过小，未达到最小间距时不处理。
        if(Math.abs(lastScrollTop - st) <= delta){
            return;
        }
        // 向下滚动超过header部分时增加样式nav-up以隐藏header部分.
        if (st > lastScrollTop && st > headerHeight){
            // 向下滚动
        $header.addClass("nav-up");
        //$header.css({"top": "-" + headerHeight + "px"}); 高度动态设置   
        } else {
            // 向上滚动
            if(st + $(window).height() < $(document).height()) {
                $header.removeClass("nav-up");
                    //$header.css({"top": 0}); 高度动态设置
            }
        }
        lastScrollTop = st;
    }

如此使用250ms的定时器来检测是否在scroll过程中的方式我是第一次使用，而且与我的思考方向正好相反。没有想通它在性能上有什么优势，传统的做法如下：

	var $header = $("header");
	    var headerHeight =  $header.height();
	    var lastScrollTop = 0;
	    var stickyTimer = null;
	    var delta = 5;
	    var didScroll;
	    //$("body").css({"padding-top": headerHeight});头部高度不固定时JS中设置
	    $(window).scroll(function() {
	        if(stickyTimer != null){
	            clearTimeout(stickyTimer);
	            stickyTimer = setTimeout(function(){
	                scrollHeaderStickyEvent();
	            },250)
			}
			else{
			    stickyTimer = setTimeout(function(){
			        scrollHeaderStickyEvent();
				},250)
			}
	    }
	    function scrollHeaderStickyEvent(){
	        //同上…….
	    }

参考资料：<a href="https://medium.com/design-startups/67bbaae9a78c">https://medium.com/design-startups/67bbaae9a78c</a>


