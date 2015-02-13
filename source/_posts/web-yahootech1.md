title: 雅虎科技频道纯图片布局的实现（上）
date: 2014-05-14 00:25:12
tags: YahooTech 雅虎科技 纯图片 横向瀑布流
---
题外话：希望有一天做东西不再用“仿”这个字。
首次看到<a href="https://www.yahoo.com/tech" title="yahooTech" target="blank">雅虎科技</a>的页面，我想到的是这几个词，全屏、大胆、横向瀑布流、随机、没什么大不了的。仔细研究了一下它的页面布局之后，发现了几个细节：

1. 每行显示的图片数量和布局是否有规律？

	取样了近200条数据，无明显规律。但是大概可以推断一些边界值。如每行显示2张、3张（并列3张、左1右2）、5张（左2中2右1）、6张（左1上三下二），但不限于此。图片宽度范围为300px~1000px，高度范围为260px~620px。每个区块均绝对定位，动态设置宽高和位置。Pinterest的瀑布流很早就使用这种方式定位，对页面元素的控制性更好，而我们更习惯于浮动定位。

	举个形象的故事，就像组织同学们排队，小红你站在（25,25）这个点上，小绿你站在（25+10,25）这个点上，这是绝对定位；同学们按照学号依次排列，中间间隔10cm，这是浮动。浮动不用计算每个人的位置，实现简单，但是常出现一种现象：就是后面的人一拥而上看似排好了但肯定还会陆续往后退。这种体验不好，除非明确每个人的三围，站好了就不要乱动。

	于是，我让她们每个人回家量三围，排队前都贴到衣服后面（等同于让后端获取图片宽高写入html），这样后面的同学会主动预留位置。但是还是有好些同学没有完成任务，忘记啦，家里没有尺子啦，确实量不出来啊等等。我就无语了，先排队吧。小红学号在前面，先入队站好后现场量三围，发现胖了就把后面的同学往后挤挤。小绿学号在后面，还没轮到她的时候她就量好了。所以她就不影响。

	但这影响了我们的班级形象啊，排个队都拖拖拉拉的。小红说，我们家就是没有钱买尺子啊。。。（后端同学说无法取得图片尺寸，鬼才信呢）我只好决定，没有三围的同学不再入列。话扯得有点远了，拉回来。上述的边界值是2014年4月13日统计的，现在布局有些许变化，向着可视区域内图片数量增加的方向改进。

2. 无论如何改变屏幕大小，图片清晰依旧。

	响应式设计中，这种按需要尺寸加载图片的技术必然会普及，那Yahoo对每张图片都提供了那些尺寸呢？经调查，图片宽度从200px-1000px，每隔50／100提供一种尺寸。200、250、300、350、400、450、500、550、600、700、800、900、1000。（试着读一下这些数据，有没有想起《卖拐》里的情景）1000px的图片文件大小得多大啊？！随便看了一张88.4K，只有全屏的背景图我才舍得用100K左右的图片，人家不差网速呀。

3. 图片上的文字遮罩

	图片上显示文字为了显示清晰，常用的解决方案就是给文字增加一层遮罩。这种遮罩过于明显很不美观，而Yahoo使用的是根据图片的主色使用渐变白或渐变黑遮罩，正好与图片很好地融合到一起。

对比看一下YahooTech和两性（床上）版块的实现效果。

![雅虎科技](/img/yahooTech1.jpg)

![立方媒体](/img/yahooTech2.png)

实现上主要有三个类，简要分析一下代码：

	//Manager类，管理Row和DataPool
	Class("RowManager", com.lifeix.event.Listener, {
	    instance : null,  //单例
	    constructor : function(){},
	    //调用入口，从缓冲池DataPool中获取数据传递给showBlocks()
	    showMoreBlocks : function(requiredCount){}
	    //遍历数据，逐个插入到当前行
	    showBlocks : function(items){}
	    //获取当前Row，如果当前行数据已满则新建Row
	    getAvailableRow: function(items){}
	    //新建Row
	    createNewRow: function(items){}
	    ……
	}

	//Row类，实现行的基本操作
	Class("Row", com.lifeix.event.Listener, {
	    constructor : function(){},
	    //新建一行append到页面中
	    init: function(){}
	    //每行显示二条或三条数据，逐条append到页面中。
	    insertItem: function(items){}
	    //当前行数据已满时设置，根据图片宽高计算行高。
	    calcuRowHeight: function(items){}
	    ……
	}

	//DataPool类，实现缓冲池的基本操作
	Class("DataPool", com.lifeix.event.Listener, {
	    instance : null, //单例
	    constructor : function(){},
	    //将初始化得到的和ajax后续加载的数据均存储到缓冲池DataPool中。
	    refresh: function(){}
	    //发送ajax请求入口，过滤重复请求。
	    _loadMore: function(items){}
	    //发送ajax请求核心方法，请求后的数据存储到缓冲池DataPool中
	    _sendAjaxRequest: function(items){}
	    //从缓冲池DataPool中获取指定数量的数据
	    getTopItems: function(num){}
	    ……
	}

