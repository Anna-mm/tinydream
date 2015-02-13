title: 一滚两动——固定显示的元素超出可视区域部分如何显示完整
date: 2013-11-07 20:07:58
tags:
---

在页面布局及交互中，遇到这样一个问题：在发表文章页面有两栏布局，左侧是编辑区域，右侧为辅助功能如是否公开、添加标签、所属分类等。左侧编辑区域要求根据用户输入自动加高即内容高度不固定，右侧功能要跟随跟随左侧固定显示以便操作。问题就出现了，fix显示的元素高度超出可视区域部分如何显示完整？
简单的示意图如下：
![跨域请求](/img/scrollfix1.png)

可见，右侧滚动条全权负责左侧内容的滚动，那如何同时负责左右两个区域的滚动呢？他们的调度顺序是怎样的？思考中。。。

是不是都是这样想的呢？大致分成以下三个阶段：
1、初始化，拖动滚动条左右侧正常滚动。
2、右侧滚动到顶端，固定显示（fixed），左侧继续滚动。
3、左右侧低端平行时，左右侧一起滚动。

第三点恰恰是问题的切入点。效果的实现无碍都是scrollTop、height、offsetTop的计算，用代码分析一下这个过程：

####1、当卷去高度大于右侧距顶端的高度，右侧固定显示，rightColFixFlag为阶段标志，值为0、1、2对应上述的三个阶段。

	if($window.scrollTop() > $rightColumn.offset().top){
	    if(rightColFixFlag == 0){
	        $rightColumn.css({"position": "fixed","left": offsetLeft,"top": DEFAULT_TOP});//DEFAULT_TOP=0
	            rightColFixFlag = 1;
	    }
	}
	else{
	    $rightColumn.css({"position":"static"});
	    rightColFixFlag = 0;
	     rightTop = DEFAULT_TOP;
	}
####2、 左右侧低端平行，两侧同时滚动。右侧仍为fixed显示，通过变化其top值完成滚动。
![跨域请求](/img/scrollfix2.png)
![跨域请求](/img/scrollfix3.png)

	if(option.rightColFixFlag == 1){
	    //计算左右低端之间的距离        
	    var distance = $leftColumn.height() + $leftColumn.offset().top - $window.scrollTop() - $rightColumn.height()
	    //左右侧低端平行           
	    if(distance  <= 0){
	        //计算左右侧低端平行时滚动条卷去高度。
	        //如果你的手劲刚刚好，某一次拖动恰好赶上这个接触点，那滚动条卷去高度就正好是$window.scrollTop()，一般情况下，你会错过这个点，使用distance修正。
	        fixScrollTop = $window.scrollTop() + distance;
	        rightColFixFlag = 2;
	      }
	}
	if(option.rightColFixFlag == 2){
	    //计算右侧的top值，为相邻两次滚动的差值。
	    rightTop -= ($window.scrollTop() - fixScrollTop);
	    fixScrollTop = $window.scrollTop();
	    //计算左侧低端距离页面低端的高度
	    offsetBot = $document.height() - $leftColumn.offset().top - $leftColumn.height();
	    //计算右侧top的下限值
	    rightColEndTop = $window.height() - $rightColumn.height() - offsetBot;
	    //判断右侧top值处于哪个区间，(-∞，rightColEndTop]或(rightColEndTop，DEFAULT_TOP)或(DEFAULT_TOP，+∞)
        if(rightTop <= rightColEndTop){
                //右侧top值达到下限
              option.$rightColumn.css({"top": option.rightColEndTop});
        }
        if(rightColEndTop < rightTop < DEFAULT_TOP){
                $rightColumn.css({"top": option.rightTop});
        }
        if(rightTop > DEFAULT_TOP){
            //右侧top值达到上限
            $rightColumn.css({"top": DEFAULT_TOP});
            rightColFixFlag = 1;
            rightTop = DEFAULT_TOP;
        }
        //rightTop==DEFAULT_TOP，disatance=0，滚动条正好滚动至左右侧底端平齐，可不做调整。
	}
![跨域请求](/img/scrollfix4.png)

