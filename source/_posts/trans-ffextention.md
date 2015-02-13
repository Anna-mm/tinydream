title: 【翻译】开发Firefox扩展
date: 2012-02-04 17:46:29
tags: Firefox扩展
---
译自[https://developer.mozilla.org/en/Building_an_Extension](https://developer.mozilla.org/en/Building_an_Extension)，意译非直译

##简介
这篇教程通过一些必要的步骤指导你开发一个简单的扩展——在火狐浏览器的状态面板（status bar panel）中加入一项“hello,World”。（ajiao注：事实上是加在了工具菜单中）
从firefox4（和其他的基于Mozilla 2版本的应用程序）开始有两种类型的扩展：
1. 传统的XUL扩展比较强大，但麻烦的是安装后需要重启浏览器；
2. 免重启的扩展，不需要安装后重启但是相比于传统扩展受限得多。[Add-on SDK and the Add-on Builder](https://addons.mozilla.org/en-US/developers/tools/builder)可以容易地用于开发免重启扩展。

这篇文章说明如何开发传统的扩展。

##快速开始
这个扩展在[another tutorial from MozillaZine Knowledge Base](kb.mozillazine.org/Getting_started_with_extension_development)上有逐行的代码解释

###1、构建开发环境
扩展打包成zip文件，或使用XPI文件系统("zippy"发布的)打包。
典型的XPI文件如下：

	my_extension.xpi：                       //扩展名
	/install.rdf                            //扩展的信息（包括id、名称、版本号等）
	/chrome.manifest              			//通过chrome引擎注册，负责扩展的运行
	/chrome/
	/chrome/content/                        //包括XUL and JavaScript 文件
	/chrome/icons/default/*                 //扩展icon
	/chrome/locale/*                        //本地化
	/defaults/preferences/*.js
	/plugins/*
	/components/*
	/components/cmdline.js

我们要创建一个类似的文件结构，随便在某一个硬盘上新建一个文件夹（比如C:\extensions\my_extension\ or ~/extensions/my_extension/），里面新建一个名为chrome的文件夹，再在里面建一个名为content的文件夹。在文件夹的根目录下新建两个空文件，分别命名为chrome.manifest and install.rdf。在chrome/content文件夹中，新建一个文件，命名为sample.xul。那么，这个文件结构是这样的：

	install.rdf
	chrome.manifest
	chrome\
		content\
			sample.xul

关于构建开发环境的其他信息请参考[Setting up extension development environment](https://developer.mozilla.org/en/Setting_up_extension_development_environment)

###2、编写instal.rdf

打开文件，将以下代码放到里面.

	<?xml version="1.0"?>
	<RDF xmlns="http://www.w3.org/1999/02/22-rdf-syntax-ns#" xmlns:em="http://www.mozilla.org/2004/em-rdf#">    
		<Description about="urn:mozilla:install-manifest">
			<em:id>sample@example.net</em:id>
			<em:version>1.0</em:version>
			<em:type>2</em:type>
			<!-- Target Application this extension can install into,with minimum and maximum supported versions. -->
			<em:targetApplication>
				<Description>
				<em:id>{ec8030f7-c20a-464f-9b0e-13a3a9e97384}</em:id>
					<em:minVersion>1.5</em:minVersion>
					<em:maxVersion>4.0.*</em:maxVersion>
				</Description>
			</em:targetApplication>
			<!-- Front End MetaData -->
			<em:name>sample</em:name>
			<em:description>A test extension</em:description>
			<em:creator>Your Name Here</em:creator>
			<em:homepageURL>http://www.example.com/</em:homepageURL>
		</Description>
	</RDF>

1、sample@example.net：扩展ID，Email格式的字符串用来唯一确定你的扩展。或者是一个GUID。
2、`<em:type`>2`</em:type`>：2是说这是一个扩展，如果要安装主题就是4，关于更多type参见[Install Manifests#type](https://developer.mozilla.org/en/Install_Manifests#type) 。
3、{ec8030f7-c20a-464f-9b0e-13a3a9e97384}：firefox的ID
4、1.5：扩展能够work的firefox的最低版本，需要确切的版本号，而不能用*代替。
5、4.0.*：扩展能够work的firefox的最高版本，4.0.*意味着扩展能够在firefox4.0和4.0.x以下的各版本中work。这个值必须小于等于 firefox已发布的版本。默认地，firefox10和以后版本不受这个最高版本限制，你可以通 过<em:strictCompatibility>强制限制。

注：如果提示此文件有问题，在firefox中通过文件->打开文件命令可以看到返回的xml错误信息。
扩展至少要在Firefox 2.0.0.x上work的话，最高版本应该设置成"2.0.0.*"。
扩展至少要在Firefox 1.5.0.x上work的话，最高版本应该设置成"1.5.0.*"。
（ajiao注：此言应该是为了保险起见，避免firefox不兼容）

想了解更多的required和optional属性请查看[Install Manifests](https://developer.mozilla.org/en/Install_Manifests)

###3、编写XUL文件

Firefox的用户接口（user interface）是用XUL和JS写的。XUL遵循XML语法，提供button、menu、toolbars、trees等组件（widgets），用户行为通过JS实现。

为了扩展浏览器，我们增加或修改组件来改变浏览器的部分UI。通过插入新的XUL DOM 元素到浏览器窗口来增加组件，用script和绑定事件来修改组件。

浏览器是通过一个名为browser.xul ($FIREFOX_INSTALL_DIR/chrome/browser.jar contains content/browser/browser.xul)的文件运行的（ajiao注：事实上，我没有找到这个文件），这个文件中有如下代码：

	<statusbar id="status-bar">
		... <statusbarpanel> ...
	</statusbar>

<statusbar id="status-bar">是一个"覆盖点"（merge point），可以被XUL Overlay文件覆盖的地方。

XUL Overlays是一种在运行时（at run time）把其他的UI组件结合到一个XUL文件的方法，一个XUL Overlay是一个XUL文件，它把一些XUL片段插入到一个主（master）文件的特定覆盖点。这些XUL片段指定组件是应该insert、 remove还是modified。

XUL Overlay 文件示例：

	<?xml version="1.0"?>
	<overlay id="sample" xmlns="http://www.mozilla.org/keymaster/gatekeeper/there.is.only.xul">
		<statusbar id="status-bar">
			<statusbarpanel id="my-panel" label="Hello, World"  />
		</statusbar>
	</overlay>

这个叫status-bar的<statusbar>指定了一个覆盖点，这个覆盖点是浏览器窗口中我们想要追加的地方。子节点<statusbarpanel>是想插入覆盖点的一个新组件（widget）。把上面的代码保存到chrome/content下的sample.xul文件中。

关于更多的插入组件和通过覆盖修改用户接口的内容，请往下看。

###Chrome URIs
ajiao注：Chrome不是Google Chrome，可以理解为普通意义上的浏览器。
XUL 文件是"[Chrome Packages](https://developer.mozilla.org/en/Chrome_Registration)" 的一部分，"Chrome Packages"涵盖了用户接口（user interface）的所有部件（components），这些部件可以通过chrome://URIs来访问（load）。而不是通过file://URI从硬盘上访问浏览器（尽管Firefox在系统中的位置可以在平台（platform）间和系统（system）间改变），MOzilla开发者提出了这个解决方案：创建指向XUL内容——这些是安装程序知道的——的URIs，[chrome://browser/content/browser.xul](chrome://browser/content/browser.xul)，把这个放到地址栏中。

Chrome URIs包括以下部件：
1、chrome://：URI scheme (chrome)，它告诉Firefox的网络库（networking library）那是一个chrome URI。它指明URI 内容应该被视为（chrome）来处理，而不是视为一个网页（http）。
2、browser：包名本身确定了这是用户接口部件，它应该区别于你的应用程序，尽可能避免扩展之间的冲突。
3、content：请求的数据类型。有三种类型：content (XUL，JavaScript，XBL bindings等决定了应用程序UI的结构和行为)，locale (DTD，.properties files等包含了UI的本地化的字符串)，and skin (CSS and images等确定了UI的theme主题)。
4、browser.xul：要访问的文件路径。

那么，[chrome://foo/skin/bar.png](chrome://foo/skin/bar.png)就是访问foo主题的skin下的bar.png。

当你通过Chrome URI访问资源时，Firefox通过Chrome注册信息（Registry）把这些URIs转换成硬盘上（或jar包中）实际的资源文件。

###4、编写chrome.manifest文件
关于Chrome Manifests和其支持的属性的更多信息，请参看Chrome Manifest。
打开文件，加入以下代码：
	
	content     sample    chrome/content/

不要忘记结尾的斜杠“/”，没有它这个包不能被注册。有以下三点说明：
1、content ：chrome package的内容类型（type of material within a chrome package）。
2、sample ：chrome package名（包名一定要用小写字母，Firefox的早期版本不支持大写或驼峰命名）。
3、chrome/content/：chrome package文件的位置。
这一行代码就是说明有一个名为sample的chrome package，可以在相对于chrome.manifest所在目录的chrome/content位置下找到这些资源。

你还需要告诉Firefox把你的覆盖点合并到浏览器窗口中，加入以下代码：
overlay [chrome://browser/content/browser.xul](chrome://browser/content/browser.xul) [chrome://sample/content/sample.xul](chrome://sample/content/sample.xul)
这就是告诉Firefox加载browser.xul时要把sample.xul合并进去。

###5、测试
首先，应该告诉Firefox有你这么一个扩展。在Firefox 2.0或更高版本中，你可以把Firefox指向你开发的那个扩展对应的文件夹，那每次重启时就会加载这个扩展。

1、找到[profile folder](kb.mozillazine.org/Profile_folder)（这个文件夹是第一个启动Mozilla时自动创建到你本地的，保存了程序的默认信息，在win7下，可以通过命令行%APPDATA%\Mozilla\Firefox\Profiles打开文件），你的工作就是在这个文件夹里。
2、打开extensions/这个文件夹，如果需要的话自建一个。
3、创建一个文本文件，把你的扩展文件的全路径(e.g. C:\extensions\my_extension\ or ~/extensions/my_extension/)写到里面。（windows用户应该保留操作系统的斜杠，并且要以斜杠结尾，去掉末尾的空格。）
4、保存文件，命名为你的扩展ID （e.g. [sample@example.net](sample@example.net)），没有文件后缀。

现在就可以测试你的扩展了。
启动Firefox，检测到有文本链接到你的扩展目录下，安装扩展。当浏览器完全显示的时候，你可以在状态面板的右侧（on the right side of the status bar panel）看到“Hello, World!”（ajiao注，事实上是在工具菜单中）
现在，你可以回去编写.XUL文件了，关闭然后启动Firefox，就能看到你想看到的。

###6、打包
你的扩展正常work了，就可以打包了。
把你扩展文件夹里面的内容打成zip包（注意是文件夹里面的内容，不是文件夹本身。ajiao注：开始就卡在这个盲点上了。），然后重命名，以.xpi作为后缀。
* 在XP中，选中扩展文件夹里的所有文件包括子文件，右击选择“发送到-->压缩zipped文件夹”。一个zip文件就创建好了。
* 在Mac OS X下，选择扩展文件夹里的所有文件，右击选择“Create Archive of...”就可以了。然后，自从Mac OS X增加了用来追踪文件元数据的隐藏文件夹，你就不应该使用终端，而是删除隐藏文件夹（那些以时间命名的）在命令行中敲入zip命令来创建zip文件。
* 在Linux下，同样通过命令完成。

如果你安装了“Extension Builder”这个扩展，它能够帮你编译成.xpi文件（Tools -> Extension Developer -> Extension Builder）。指定生成目录为扩展所在的目录（install.rdf 等），然后敲击Build Extension按钮。这个扩展有很多工具让开发变得更容易。

现在，上传你的.xpi文件到你的服务器上，确保它被服务为application/x-xpinstall。你可以link到它，让别人也可以下载和安装。如果只是测试我们的.xpi文件，可以直接拖到工具->附加组件中。（ajiao注：直接拖到浏览器中就可以）

###使用addons.mozilla.org（ajiao注：这里是广告么？）

Mozilla Add-ons是一个分布式（distribution）站点，可以免费寄宿（host）你的扩展。你的扩展可能非常受欢迎，也可以寄宿在Mozilla的映射（mirror）网络上来保证下载。Mozilla站点提供给用户更简单的安装方式，只有你上传了新的版本，它就会自动地到达（available to）使用者那里。另外，Mozilla Add-ons允许用户评论，并为你的扩展提供反馈。强烈推荐你用Mozilla Add-ons发布扩展。访问Visit [http://addons.mozilla.org/developers/](addons.mozilla.org/developers/)，创建一个账户就可以发布你的扩展了。

注：如果有一个更好的描述和截图，你的扩展会很快通过和被下载的。

##MORE
###7、更多的XUL Overlays
除了把UI 组件加到覆盖点（merge point），你可以在Overlays里面用如下XUL片段：
* 修改覆盖点的属性，例如隐藏状态条。
		<statusbar id="status-bar" hidden="true" />

* 从主文件中移出覆盖点，例如

		<statusbar id="status-bar" removeelement="true" />

* 控制插入组件的位置：

		<statusbarpanel position="1" ...  />
		<statusbarpanel insertbefore="other-id" ...  />
		<statusbarpanel insertafter="other-id" ...  />

###8、创建新的User Interface Components
你可以用单独的.xul文件创建自己的窗体和对话框，在JS文件中执行用户行为来提供功能，用DOM方法来操作用户组件，在.css文件中加入图片，设置样式。关于XUL开发的更多文档请参考[XUL](https://developer.mozilla.org/en/XUL)。
（ajiao注：XUL：XML User Interface Language，一种基于XML的语言，可以开发多功能的跨平台的应用程序，无论是否联网都可以运行。这些程序很容易通过文本，图片，布局来自定义。）

###9、默认文件
用来构成用户的基本信息（seed a user's profile）的默认文件，位于扩展文件夹的根目录下。默认偏好（preferences）的JS文件应该在defaults/preferences/下，当Firefox的偏好系统（defaults/preferences/）开始启动的时候，它们会被自动载入。访问[Preferences API](https://developer.mozilla.org/en/Preferences_API).了解更多信息。
默认偏好文件的实例：

	pref("extensions.sample.username", "Joe"); //a string pref
	pref("extensions.sample.sort", 2); //an int pref
	pref("extensions.sample.showAdvanced", true); //a boolean pref

###10、XPCOM Components
Firefox在扩展中支持[XPCOM](https://developer.mozilla.org/en/XPCOM) components。你可以用JS 或C++（用[Gecko SDK](https://developer.mozilla.org/en/Gecko_SDK)）很容易地创建自己的components（ajiao注：XPCOM类似于Microsoft COM，一种跨平台的component object model）。把你的.js、.dll文件都放到components/目录下，安装扩展后的Firefox在第一次运行的时候就会自动注册它们。

更多的信息参考 [How to Build an XPCOM Component in Javascript](https://developer.mozilla.org/en/How_to_Build_an_XPCOM_Component_in_Javascript), [How to build a binary XPCOM component using Visual Studio](https://developer.mozilla.org/en/How_to_build_a_binary_XPCOM_component_using_Visual_Studio) and the [Creating XPCOM Components](https://developer.mozilla.org/en/Creating_XPCOM_Components) book.

**Application Command Line**

自定义的XPCOM components的一个可能的用法就是增加一个命令行的处理到Firefox中，你可以用这种方法把扩展作为一个应用程序来运行。

		firefox.exe -myapp

更多信息请参考 [Chrome: Command Line](https://developer.mozilla.org/en/Chrome/Command_Line) and a [forum discussion](forums.mozillazine.org/viewtopic.php?t=365297)

**Localization**

为了支持多语言，你可以用[entities](https://developer.mozilla.org/en/XUL_Tutorial/Localization)和[string bundles](https://developer.mozilla.org/en/XUL_Tutorial/Property_Files)把字符串从你的内容中拆分出来。在开发过程中做这件事更容易，比起回过头来做。

本地化信息存储在扩展的locale目录下。比如，你要在扩展里加一个locale，在chrome目录里（即content的同级目录下）嵌套创建出locale/en-US这个目录，然后把下面这行加到chrome.manifest文件里。
	
	locale sample en-US chrome/locale/en-US/

要在XUL中创建本地化的属性，你可以把这些值放在一个.dtd文件里，.dtd文件放在locale目录下，代码如下：

	<!ENTITY  my-panel.label  "Hello, World">
		在XUL文件的顶部(在<?xml version"1.0"?>的下面)加入如下代码：
	<?xml version="1.0"?>
	<!DOCTYPE window SYSTEM "chrome://packagename/locale/filename.dtd">
	...

XUL文档的根节点的[localName](https://developer.mozilla.org/En/DOM/Node.localName) 值是window, “SYSTEM”属性的值是指向entity file（ajiao注：就是DTD文件）的chrome URI.

在我们的示例扩展中，把根节点window换成overlay，packagename写成sample，filename.dtd写成sample.dtd。怎么用这个entities呢，修改XUL如下：

	<statusbarpanel id="my-panel" label="&my-panel.label;"  />

Chrome注册信息会确保entity file被加载。
也可以给script中使用的字符串创建一个属性文件，就是一个每一行都是这种格式的文本文件。
key=value
然后用[nsIStringBundleService](https://developer.mozilla.org/en/XPCOM_Interface_Reference/nsIStringBundleService) /[nsIStringBundle](https://developer.mozilla.org/en/XPCOM_Interface_Reference/nsIStringBundle) or the [stringbundle](https://developer.mozilla.org/en/XUL/stringbundle)标签把相应的值load到script中。

###11、了解浏览器
用[DOM Inspector](https://developer.mozilla.org/en/DOM_Inspector)（ajiao注：一个扩展）检查你想扩展的浏览器窗口或是其他的XUL窗口。
提示：Firefox在默认情况下不安装DOM Inspector，从Firefox 3 Beta 4版本起，它单独作为一种扩展。如果你的浏览器是早期版本，工具菜单中没有“DOM Inspector”，那就用自定义安装方式选择DOM Inspector重新安装。

在DOM Inspector工具条的左上方有个点选式（point-and-click）按钮，在XUL窗口点击一个节点来选择它，DOM树就会跳到指定的节点。

用DOM Inspector的右侧面板可以发现覆盖点的ID，你可以用来插入你的元素。如果你没有发现你可以插入（merge into）的带ID的元素，你可能需要在overlay（ajiao注：overlay.xul）中加入一个script，当load事件运行到主XUL窗口时（the load event fires on the master XUL window）插入你的元素。

**Debugging Extensions**

Debug分析工具
* [DOM Inspector](https://developer.mozilla.org/en/DOM_Inspector)——检测已生效的属性、DOM结构、CSS样式（一个找出为什么样式没有生效的工具）
* [Venkman](https://developer.mozilla.org/en/Venkman)——在JS中设置断点，并inspect call stacks
* JS中的[arguments.callee.caller](https://developer.mozilla.org/en/JavaScript/Reference/Functions_and_function_scope/arguments/callee)——访问一个方法的call stack

函数调试
* 用[dump](https://developer.mozilla.org/en/DOM/window.dump)("string")（详情看链接，它需要一些配置才能正常work）
* 用[Components.utils.reportError()](https://developer.mozilla.org/en/Components.utils.reportError) or [nsIConsoleService](https://developer.mozilla.org/en/XPCOM_Interface_Reference/nsIConsoleService)输出（log to）到控制台。

高级调试
* 在Firefox中设置断点运行，对于有经验的开发者来说，这是诊断问题的最快方法。详情请看[Build Documentation](https://developer.mozilla.org/En/Developer_Guide/Build_Instructions) and [Developing Mozilla](https://developer.mozilla.org/En/Developer_Guide)
* 更多的帮助提示请看[Debugging a XULRunner Application](https://developer.mozilla.org/en/Debugging_a_XULRunner_Application)

###12、更多信息
[Extension FAQ](https://developer.mozilla.org/en/Extension_Frequently_Asked_Questions)
[Extensions](https://developer.mozilla.org/en/Extensions)