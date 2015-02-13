title: 主流地图API比较
date: 2012-05-25 21:17:03
tags: 地图 API
---
空白网页中引用各地图JS，查看性能。

<table><tr><td width="25%"> </td><td width="25%">文件大小（bytes）</td><td  width="25%">加载时间（ms）</td><td  width="25%">数据提供</td></tr><tr><td>Google</td><td>4,684</td><td>400</td><td></td></tr><tr><td>bing</td><td>1,041,… </td><td>4500</td><td></td></tr><tr><td>soso</td><td>156,688</td><td>200</td><td>四维(navinfo)</td></tr><tr><td>sogou</td><td>92,580</td><td>500</td><td></td></tr><tr><td>baidu</td><td>111,308</td><td>400</td><td>四维(navinfo)</td></tr></table>
				
###Google地图
API：[https://developers.google.com/maps/documentation/javascript/examples/?hl=zh-CN](https://developers.google.com/maps/documentation/javascript/examples/?hl=zh-CN)
引用：[http://maps.googleapis.com/maps/api/js?sensor=false](http://maps.googleapis.com/maps/api/js?sensor=false)
简介：2006年发布Maps API，v2版已废弃，现使用v3。
* 支持Chrome、iPhone Safari和Android手机上使用
* 不在需要API keys
* 基于MVC（Model-View-Controller）的框架，这将减少JavaScript的下载量，并且简单易用。
* 命名空间。所有的一切都在google.maps.*的命名空间，没有以“G”为前缀的全局变量。
![Google地图](/img/webmap1.png)

###bing地图
引用：[http://ecn.dev.virtualearth.net/mapcontrol/mapcontrol.ashx?v=6.2](http://ecn.dev.virtualearth.net/mapcontrol/mapcontrol.ashx?v=6.2)
![bing地图](/img/webmap2.png)

###soso地图
URL：[http://api.map.soso.com/](http://api.map.soso.com/)
引用：[http://api.map.soso.com/v1.0/main.js](http://api.map.soso.com/v1.0/main.js)
更新日志：2011-07-04 v1.0版本正式发布，最近一次修复在2012-05-15。
API特色：
* 基于MVC框架
* 内存占用量小
* 丰富流程的动画效果
* 接口开发程度高
* 兼容各种浏览器
现称为腾讯地图，最近一次更新日志为2012-07-19 。(2014-6-11注)
![soso地图](/img/webmap3.png)

###sogou地图
引用：[http://api.go2map.com/maps/js/api_v2.5.js](http://api.go2map.com/maps/js/api_v2.5.js)
更新日志：2011-07-22 API 2.0发布至2012-02-14 API 2.5发布
兼容性：
* IE 6.0+ (Windows) 以及ie内核的其他浏览器，如：搜狗高速浏览器、遨游浏览器、360浏览器、世界之窗浏览器等。
* Firefox 2.0+ (Windows|Mac|Linux)
* Safari 3.1+ (Mac|Windows)
* chrome谷歌浏览器 (Windows)
* opera 10+ (Windows)
![sogou地图](/img/webmap4.png)

###baidu地图
URL：[http://developer.baidu.com/map/index.html](http://developer.baidu.com/map/index.html)
引用：[http://api.map.baidu.com/api?v=1.3](http://api.map.baidu.com/api?v=1.3)
更新日志：有资料考证由2010-10-12 API 1.1 至 2012-04-11 API 1.3
简介：
* 兼容HTML4和HTML5，卡位移动互联网入口；
* 新增定位服务、手势操作、硬件加速等功能；
* 帮助开发者快速构建适应PC和移动环境的Web地图应用，数据更新及时、服务更加稳定；
* 全新界面样式。
从1.5开始需要先申请密钥。更新日志至2014-4-4。（2014-6-11注）
![baidu地图](/img/webmap5.png)

