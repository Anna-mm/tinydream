title: 从safari下的缺字符说起
date: 2011-12-12 20:33:01
tags: 字体 缺字符 safari opera 兼容 浏览器
---
又开始做网站的opera和safari的兼容，苦难随之而来。今收了一小鬼，不料却动不得，眼看其在safari下恣意，也只好任其发展。话说一非主流MM通过QQ登录了立方网，其他浏览器都显示正常，但在safari下其个性的名字就成了乱码。

![safari的缺字符](/img/safaricode.png)

不要说试试改变浏览器的文本编码，unicode(UTF-8)、简体中文（GB2312）等等，一个比一个乱。再来看字体设置：`font-family："Microsoft Yahei"`。难道是缺乏备用字体，陆续加了多个字体都不起作用。

那我们从font-family的解释方式说起，常见的字体设置如下，

	font-family:"Comic Sans MS","幼圆","黑体",sans-serif;

按照W3C的规范，浏览器在使用这个font-family显示一个字符时，首先应寻找Comic Sans MS字体，如果找不到Comic Sans MS字体，顺序搜寻下一个字体，即“幼圆”。如果找到Comic Sans MS字体，那么浏览器在Comic Sans MS 字体中寻找这个字符，就使用Comic Sans MS 字体来显示这个字符。如果没有找到这个字符，或者这个字符对应一个缺字符（缺字符是字体文件中的一种特殊字符，用来表示字体文件中没有这个字符，通常就是显示一个方块），那么就要到下一个字体，也就是“幼圆”中继续搜寻这个字符。如此搜索整个字体表，直至搜索到这个字符为止。如果在通用字体sans-serif中也找不到这个字符的话，那么浏览器就应该显示这个字符的缺字符。那么，在一个正常的windows XP系统上，所有的中文字符都会被显示为幼圆，英文字符都被显示为Comic Sans MS字体。另外，像双引号“”这样的字符在这两种字体中都有，浏览器应该优先按照Comic Sans MS中的双引号显示。

说了半天那也只是规范，“应该”，事实呢。很多事情都有“潜规则”的，IE和opera在搜索“Comic Sans MS”失效后，理应搜索幼圆字体，可实际上，并没有搜索下一个字体，甚至也没有搜索后面的黑体和sans-serif，而是直接跳到系统默认字体了——请注意，是系统默认字体，也没有使用浏览器自设的字体。Comic Sans MS明明存在双引号，也没有在opera下正确地显示，IE7至少还认了双引号。那safari呢，在第一个字体中寻找不到中文字符时，干脆显示了缺字符，于是网页就都变成了框框。据说，牛人们多次和Apple交涉，他们才最终修正了这个bug。FireFox呢，也并不完美，它不支持字体别名。于是幼圆你只能写成“幼圆”，黑体你只能写成“黑体”，而不能用他们在系统中的正式名称——YouYuan和SimHei。借此，想到一个怪异的事，opera竟然不能识别“Microsoft Yahei”，一定要是“微软雅黑”。

 可见，我们是遇到了safari的bug，不是说解决了么，怎么还是这样，不解的是，为啥其他浏览器都可以在Microsoft Yahei字体中找到要显示的字符，而safari就是找不到呢，也许只有他们的工程师知道吧。

还有safari的确认框——confirm，竟不支持方向键，点击"取消"也要点两次才行。看来在windows下果然不是本土作战，弱呀~

