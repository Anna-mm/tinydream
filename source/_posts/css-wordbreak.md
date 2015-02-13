title: CSS3中文本自动换行的样式说明
date: 2011-12-22 21:31:00
tags: CSS3 文本换行
---
浏览器本身都自带着让文本自动换行的功能，会让文本在浏览器或div元素的右端自动实现换行。
默认换行规则：
① 对于西方文字，在半角空格或连字符的地方换行。
② 对于中文，可以在任意一个中文字后换行（除非这个文字后面是标点符号，这样通常将标点符号以及它前面的一个文字作为一个整体来统一换行）。

在CSS3中，有两个属性涉及自动换行的处理：
###1、word-break属性：

<table><tr><td width="20%">值</td><td width="20%">换行规则</td><td width="20%">IE5以上版本</td><td width="25%">Safari3与chrome6</td><td width="15%">FF（待补充）</td></tr><tr><td>normal</td><td>默认换行规则</td><td>支持</td><td>支持</td><td></td></tr><tr><td>keep-all</td><td>只能半角空格、连字符处换行<br/>中文之间不能换行</td><td>支持</td><td>不支持</td><td></td></tr><tr><td>break-all</td><td>允许在单词内换行</td><td>支持<br/>不允许标点符号在行首</td><td>支持<br/>允许标点符号在行首</td><td></td></tr>
</table>


个人实践中，这个属性很少使用。通过以上说明可以看出，keep-all中文之间不能换行，没有想到会在什么情况下有这种需求。break-all允许在单词内换行，这种体验实在不敢恭维。

###2、word-wrap属性：让长单词和URL自动换行

对于西方文字来说，浏览器在半角空格或连字符的地方进行换行。因此，浏览器不能给较长的单词自动换行。当浏览器窗口比较窄的时候，文字会超出浏览器的窗口，底部会出现横向滚动条，让用户通过拖动滚动条来查看全部文字。这种体验很不好。

word-wrap属性值：
normal采用默认规则。
break-word在长单词内部换行。如24位的用户名设置成这样WWWWWWWWWWWWWWWWWWWWWWWW，就必须用break-word了。（此种情况下是否可以使用word-break：break-all？）

**补充一下字号和宽度的关系：**
例如用户名的设置规则是24个字符，一个汉字占两个字符，字体均为Microsoft Yahei。
① 12个汉字在12px的情况下宽度为144px
② 24个字符iiiiiiiiiiiiiiiiiiiiiiii 12px的情况下宽度为76px
③ 24个字符WWWWWWWWWWWWWWWWWWWWWWWW 12px的情况下宽度为293，宋体下是144px。
貌似也没什么关系。

