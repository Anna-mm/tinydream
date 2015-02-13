title: JSLint调研
date: 2013-10-16 20:19:00
tags: JSLint
---

##1、背景

随着富Web前端应用的出现,JavaScript正越来越流行。JavaScript受欢迎的部分原因是因为它的灵活便捷，可以快速上手。然而，这种灵活性在带来高效的同时，也成为初学或者经验不足的JavaScript开发人员的噩梦。形式各异的代码风格、隐含错误的代码行为，严重影响了整体代码的可读性和稳定性，成为 Web 项目中最为常见问题之一。

JSLint 正是 Douglas Crockford 同学为解决此类问题创建的工具，JSLint 除了能指出这些不合理的约定，还能标出结构方面的问题，有助于发现错误并教会开发人员一些好的编码实践。（Douglas Crockford是 web领域技术权威之一，他是JSON、JSLint、JSMin和ADSafe的创造者）

**同时也保证编码规范能够真正实施起来，避免流于形式。**
了解JSLint的工作原理[http://www.ibm.com/developerworks/cn/web/1105_linlin_jslint/](http://www.ibm.com/developerworks/cn/web/1105_linlin_jslint/)

##2、在eclipse中安装JSLint

Help -> Install New Software->add->输入以下地址
[https://svn.codespot.com/a/eclipselabs.org/mobile-web-development-with-phonegap/tags/jslint4java1/download](https://svn.codespot.com/a/eclipselabs.org/mobile-web-development-with-phonegap/tags/jslint4java1/download) ->OK

![JSLint调研](/img/jslint1.png)

勾选JavaScript Development Tools和jslint4java ->next

![JSLint调研](/img/jslint2.png)

如果遇到这个提示，不用管，点OK。
安装完成之后会提示重启eclipse，重启之后，进入window->preferences，会看到jslint4java，右侧为默认的检测规则。

![JSLint调研](/img/jslint3.png)

主要的规则如下：
bitwise 是否禁止使用位运算符
Continue 是否允许使用continue语句
Debug 是否允许debug语句
Evil 是否允许使用eval
Forin for-in声明中的key是否使用hasOwnProperty过滤
Indent 缩进的空格数，默认为4。
maxerr 每个JS文件允许最大的错误数，默认是50。 错误过多时，只返回文件名和错误总数，不返回具体错误。
Maxlen 每行代码的最大行数。建议80。
newcap 是否允许构造函数的首字母不大写。
nomen 是否允许在名称首部和尾部加下划线
plusplus 是否允许使用++或--
regexp 正则中是否允许使用.或者[^…]
Sloppy 是否允许不写编译指示语句“user strict”
Sub 是否允许使用下标获取属性值（一般情况下使用点.）
Todo 是否允许使用TODO开头的注释。
Unparam没有被使用的参数是否需要给出warning
Vars 一个function中是否允许多个var声明变量
White 是否需要遵守严格的空格用法
L06的JS经JSLint默认规则检查后代码警告无数：

![JSLint调研](/img/jslint4.png)

##3、在SVN hook中使用JSLint

更狠的是，可以在SVN服务器上配置相关信息，不符合规范的代码无法提交。
参考[http://www.amaxus.com/cms-blog/jslint-as-subversion-hook](http://www.amaxus.com/cms-blog/jslint-as-subversion-hook)

##4、使用JSLint代码检查遇到的问题

虽然JSLint的检查规则可以自己设置，但常常某个规则部分可取，部分过于严格。如果要普及开来，需要克服的问题如下：

####1、JSLint要求变量必须先定义后使用。
当check单个js文件时，会出现大量的“变量未定义”，原因是调用了其他js文件中的变量或方法，需要在当前js的头部预定义变量
如dashboard_ls.src.js中需要增加如下注释语句：
/global
scrollTimelineEvent: true,
resizeImg: true,
loadAnimation: true,
enableAnimation: true,
showcontrol: true,
showNopost: true,
ImageSourceShow: true,
DashboardMap: true,
loadAddrAnalyse: true,
blockImg:true,
loadMoreDashboard: true/
这个规则必须开启。否则真正的”变量未定义”的错误将无法检查出来。

####2、类型转换parseInt()方法必须给出第二个参数，指定进制数。JSLint没有单独针对这个规则的配置，所以无法关闭。

####3、JSLint的空格使用非常严格。

* 调用函数的时候，函数名与左括号之间没有空格
* 函数名与参数序列之间，没有空格
* 所有其他语法元素与左括号之间，都有一个空格

如果此规则开启现有代码会有大量的空格问题需要修改。可以关闭，将不对空格做任何检查。

####4、前端代码严重依赖于jquery，jquery很多地方不遵循此规则。

以上是尝试修改L06中几个JS以符合规则过程中发现的问题，仅列出这些，可能不限于此。

