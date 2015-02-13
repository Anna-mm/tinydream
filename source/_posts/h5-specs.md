title: 移动端浏览器下页面制作规范
date: 2014-03-01 21:47:27
tags: h5 webapp iPhone Android viewport media query 
---
##要求：
兼容各种分辨率下的iPhone和Android设备。
##要点：
###1. 视口viewport
头部声明与其他页面一致，使用HTML5的写法<!DOCTYPE html>，但需要在head中增加一行：

	<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">

此处width不要固定宽度，不要后端传值，使用设备宽度即可，并且不允许缩放，以避免页面出现横向滚动条。

###2. 尺寸设置
一般情况下，设计师提供的是适配iphone尺寸的设计图，通常为640*960，但是iPhone4+在解析网页的时候依旧把自己当作一台横向分辨率为320px的设备，所以针对设计图中标识的横向尺寸按比例缩小，纵向尺寸不变，字体大小减小到2/3，背景图按比例缩小，背景图不能拼接。 尽量不使用固定宽度，使用百分比无法除尽时可以精确到小数点后七位，基本保证无误差。若使用固定宽度，请放在媒体查询中：

	@media screen and (min-width: 320px){}
	@media screen and (min-width: 480px){}
	@media screen and (min-width: 640px){}

注意使用min-width，而不是min -device-width。

###3. 从桌面端向下设计VS从移动端向上设计
注意媒体查询的顺序。
从桌面端向下设计：

	@media screen and (min-width: 640px){}
	@media screen and (max-width: 640px){}
	@media screen and (max-width: 480px){}
	@media screen and (max-width: 320px){}

从移动端向上设计：

	@media screen and (min-width: 320px){}
	@media screen and (min-width: 480px){}
	@media screen and (min-width: 640px){}

推荐使用**从移动端向上设计**，所需的CSS代码更少，代码结构更加清晰。

###4. 浏览器模拟测试
测试时Chrome的UA配置如下：

![移动APP制作规范](/img/h5specs.png)

###5. 改变盒模型
标准盒模型下width属性只是内容的宽度，在响应式设计中大部分width均为百分比，如果再设置padding和border会导致内容溢出或换行，所以通常在CSS顶级样式中加入：

	*, *:before, *:after {-webkit-box-sizing:border-box;-moz-box-sizing:border-box;box-sizing:border-box;}，

使width属性包含padding和border。
###6. 手机端测试
①可将静态页面放置本地目录下:

	C:\Program Files (x86)\Apache Software Foundation\Apache2.2\htdocs
打开手机浏览器访问：
	
	http://192.168.50.241/m/cbs-download.html(本机ip+项目目录)
②若访问本地java项目如lifeix-web中的页面，可手动修改手机上的HTTP代理，服务器为本机ip，端口为80，打开手机浏览器通过域名访问。

针对简单的APP下载页面，了解以上内容即可。