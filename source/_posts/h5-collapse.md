title: HTML5移动开发中菜单如何左右折叠
date: 2014-03-29 13:02:33
tags: collapse html5 bootstrap
---

最近的工作都转移到移动开发中，针对主站www.l99.com开发一套移动端版本，像之前写的《webapp立方没体诞生之初》、《单页应用中页面转场的实现》都是针对此项目。

借此谈谈对原生APP和HTML5 WEBAPP的看法，关于各有千秋、根据项目需求的说法谁都会说，不能否认的是，HTML5在一段时间内无法与原生APP抗衡，在用户体验、速度、流量上都只能望其项背。但其表现出来的优势也越来越显著，无需审核、无需安装、跨平台（不需开发iphone和android两中版本）。在之前的PC时代，不也是安装各种管理信息系统、杀毒软件、音乐播放工具、输入法等等，而现在越来越倾向于访问网站、在线收听、云服务。这是一个趋势，但是需要多长时间达到成熟的状态还未可知。毕竟Facebook在2012年期间舍弃原本实行的HTML5架构，转而打造原生APP。不过，你可以看到这个趋势的推进，ios7的safari增加了隐藏地址栏和底部导航的功能，使WEBAPP拥有了与原生APP同样的空间。


从开发角度来看，webapp和浏览器依然扮演着同PC端相同的角色，一个负责布局样式功能的代码实现，一个负责解析渲染和交互，这些都只停留在单个页面内，一个url访问一个页面，点击链接跳转到另一页面。而对于页面间的布局和转场还鲜有涉及，没有系统或浏览器接口可供调用。要想达到原生app的体验效果，需要类似一种中间层的解决方案。其实这应该是HTML5持续推进的方向，它增加了`<section`>`<article`>`<video`>`<audio`>等各种语义化的标签，为什么没有增加`<page`>?这个标签一出现将会改变单一页面的格局，一个页面可能拥有多个page，并定义多个page的切换效果，浏览器就去实现吧。开发人员就又轻松了。好吧，这些都是意淫，话说回来，现在什么都没有，还是得自己开发呀。今天说说菜单如何左右折叠，效果嘛，原生APP中处处可见。这种把导航菜单折叠后隐藏起来的信息组织方式几乎成了原生APP的设计范式。

![facebook](/img/collapse1.jpg)
facebook中菜单折叠

####bootstrap中有没有此项功能:
bootsrap是webapp的开发利器，它通过在html上设置自定义属性定制出常见的页面布局和效果，最近的项目也是在bootstrap基础上开发的，所以顺便查找bootstrap是否支持此项功能。事实上，其官网上[http://v3.bootcss.com/javascript/#collapse](http://v3.bootcss.com/javascript/#collapse)提供了collapse插件，垂直方面上可以折叠和展开。通过调试器能够看出其大体实现原理：
1、默认情况下折叠：每个导航包括标题和内容两部分，内容默认隐藏：display：none；height：0；
2、点击某个导航：给内容增加动画样式 addClass(".collapsing")
		.collapsing{height:0;-webkit-transition:height .35s ease;}
	同时获取内容高度并设置高度，内容将按照动画样式用时0.35s以慢快慢的速度完成高度从0到指定值的过程。动画完成后删除此样式。

实际上，是CSS3流畅地完成了需要显示的内容的高度变化，将其下方内容挤走。那如何转移为横向的折叠呢？

####只有垂直折叠，如何横行折叠
bootstrap的官网实例中没有横向折叠，但是网上也能搜索出利用bootstrap的某个版本（链接地址到github）或者变种（链接到twitter）能够实现横向折叠。如果项目中已经引用了不同的版本或是自定义版本，为了一个效果便替换了新的版本是不可取的，花时间去研究两个版本的不同也不值得，尤其是自定义版本，既然选择了自定义就意味着你得了解它的代码，能够随时添加或修改。实现上也比较简单，可以自己做些修改：

![实例图](/img/collapse2.jpg)


	.viewport {
	    overflow: hidden; //使页面占满屏幕避免出现横向滚动条
	}
	.viewport .frame {
	  width: 200%;  //一唱
	  height: auto;
	}
	
	.viewport .frame .menu {
	  height: auto;	
	}
	.viewport .frame .menu.collapse {
	  float: left;
	  height: 100% !important;
	  width: auto; 
	}
	
	//折叠状态：设置宽度动画，初始宽度为0
	.viewport .frame .menu.collapse.width {
	   position: relative;
	  width:0;
	  overflow: hidden;
	  -webkit-transition: width 0.35s ease;
	  -moz-transition: width 0.35s ease;
	  -o-transition: width 0.35s ease;
	  transition: width 0.35s ease; 
	}
	//展开状态：动画结束宽度无250px
	.viewport .frame .menu.collapse.widthauto{
		width:250px;
	}
	
	//左侧导航内容
	.viewport .frame .menu .collapse-inner {
	  position: relative;
	  width: 250px;
	  height: 100%; 
	}
	//右侧内容即屏幕中心内容
	.viewport .frame .view {
	  float:left; 
	  width:50%;   //一和：外层容器为200%，内层为50%，一唱一和正好是原始大小100%。
	  height: auto; 
	  overflow: hidden; 
	  box-shadow:0 -12px 15px #999;
	  min-height: 400px;
	}

这是实例演示了如何实现左侧折叠，具体可见demo（还未添加链接），那如何实现左右侧双向折叠呢？设置300%和33.3333333%吗？且听下回分解。

