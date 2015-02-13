title: 字符串连接方法在不同浏览器中的性能比较
date: 2011-9-16 13:50:44
tags: 字符串连接 浏览器 数组 join
---
字符串连接方法在不同浏览器中的性能比较：

1. +=连接符：

		var str = '';
		str += '<div style="display:none;" id="head_app_menu">';
		str += '</div>';

2. +连接符

		var str = '';
		str = str + '<div style="display:none;" id="head_app_menu">';
		str = str + '</div>';

3. Array.prototype.join()
	
		var str = [];
		str.push('<div style="display:none;" id="head_app_menu">');
		str.push('</div>');
		str.join(“”);

![字符串连接](/img/strjoin.png)

分析：我们常用的join方法只是在IE6和IE7下有很大的性能提升，在更高版本的IE、FF、chrome下反而是降低性能的。不过，按我们现在情况下的字符串连接数（肯定是1000次循环以下），在其他浏览器有微小的消耗外，用join提升IE6、7还是最优方案。