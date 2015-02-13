title: JS压缩方案升级
date: 2013-08-29 21:28:59
tags: JS 压缩 YUI compressor UglifyJS Google closure compiler
---
##1、现状

使用javascript Obfuscator压缩。
缺点：手动执行、界面操作。
目的：选择更方便的压缩工具，支持脚本处理。

##2、备选方案

方案一：YUI compressor。
简称YC，L06 JS代码经Obfuscator压缩后10.5M，YUI compressor压缩后8.5M，混淆后8.49M。

方案二：Google closure compiler
简称CC，不仅是一个compressor，YC只做了词法上扫描，CC更是一个compiler。它的高级优化方式提出了各种“激进”的方法破坏代码，达到压缩的目的。如果能掌控它的压缩规则，代码可以压缩至极小。同时也有很强的代码约束和检查。

方案三： UglifyJS
jQuery v1.5已将压缩工具从Google closure compiler切换到UglifyJS，jQuery的青睐使其大红大紫，它的压缩策略比CC更简单安全些。UglifyJS基于node，避免不了安装node、npm、uglifyjs。
从安全平稳、易实施的角度看，还是使用YUI compressor + ant结合的方式，已更新了测试服务器上的ant脚本，具体参见
`/root/develop/lifeix_product/build.xml`。

注：混淆压缩后，我们的代码也会变得很屌，一堆a、b、c、d、e、f、g神马的，哇咔咔^_^！

##3、本地使用YUI compressor

###方法一：命令行

	$ java -jar yuicompressor-x.y.z.jar

用法: `java -jar yuicompressor-x.y.z.jar [options] [input file]`
下面的命令将压缩myfile.js文件并输出为myfile-min.js (x.y.z 为具体的版本号):
`java -jar yuicompressor-x.y.z.jar myfile.js -o myfile-min.js`

	Global Options
	-h, --help              显示帮助信息
	--type <js|css>         指明需要压缩的文件是js还是css。
	--charset <charset>     指明需要压缩的文件的 <charset>
	--line-break <column>   在指定的列换行
	-o <file>               指定输出文件 <file>。默认为标准输出（屏幕）。

	JavaScript Options
	--warn                  打印代码中的错误信息
	--nomunge             只压缩，不混淆
	--preserve-semi         保留所有分号
	--preserve-strings      Do not merge concatenated string literals, Use this option to specify that concatenated string literals should never be merged.

注：命令行也支持批处理，但是据本人实践，只针对同一级目录下的多文件处理，不支持嵌套目录。

###方法二：ant脚本
与测试服务器的更新脚本相似。

	<target name="compress"  description="Compress">
		<echo message ="begin to compress the js file." />
		<apply executable="java" dest="js" parallel="false" failonerror="true" append="false" force="true">
	       <fileset dir="js" includes="**/*.js" />
	       <arg line="-jar" />
	       <arg path="lib/yuicompressor-2.4.6.jar" />
	       <arg line="--charset utf-8" />
	       <srcfile />
	       <arg line="-o" />
	       <mapper type="glob" from="*.js" to="*.js" />
	       <targetfile />
			<!--<arg line="--nomunge " />-->
	    </apply>
		<echo message ="compress the js end." />
	</target>

注意`dest="js"`、`dir="js"` 、`lib/yuicompressor-2.4.6.jar`是文件相对路径。设置Dest为生成文件的目标路径，设置dir为待处理文件的路径。Arg path为YUI compressor路径。

###方法三：右键压缩

1. 制作批处理文件

		@echo off
		::设置YUI Compressor启动目录
		SET YUIFOLDER=D:\tool\js_compress
		::设置你的JS和CSS根目录，脚本会自动按树层次查找和压缩所有的JS 
		SET JSFOLDER=D:\tool\js_compress\js

		::Check Directories.
		IF NOT EXIST %YUIFOLDER% ECHO Error: Can't find directory "YUIFOLDER". 
		IF NOT EXIST %JSFOLDER% ECHO Error: Can't find directory "JSFOLDER".

		::Compress.
		for /r . %%a in (*.js) do ( 
			for /f "usebackq tokens=*" %%i in ("%%a") do (
				set line = %%i
				set "line = !line:SPEED=wangfj!"
			)
			echo compress %%~a ...
			java -jar %YUIFOLDER%\yuicompressor-2.4.6.jar --charset UTF-8 %%~fa -o %%~fa
		)
		echo compress finished!
		pause & exit

2. 制作注册表文件

将批处理文件写入右键菜单的“Compress JS using YUI Compressor”。

	Windows Registry Editor Version 5.00
	[HKEY_CLASSES_ROOT\AllFilesystemObjects\shell]
	[HKEY_CLASSES_ROOT\AllFilesystemObjects\shell\Compress JS using YUI Compressor]
	[HKEY_CLASSES_ROOT\AllFilesystemObjects\shell\Compress JS using YUI Compressor\command]
	@="D:\\tool\\js_compress\\yuicompressor.bat %1"

双击此文件导入信息到注册表。
