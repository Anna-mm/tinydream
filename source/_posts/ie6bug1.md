title: IE6兼容性专题一——z-index
date: 2011-10-10 16:31:21
tags: 绝对 相对 定位 IE6 兼容
---
##z-index
设置一个定位元素沿 z 轴的位置，z轴为垂直延伸到显示区的轴。我们可以想象从眼睛处向屏幕作一条垂直线，即为Z轴。z-index设置了元素的堆叠顺序，值越大就离用户更近，反之离用户更远。

1. z-index只对定位元素（设置position:relative/absolute/fixed）有效。
2. 正常情况下同级元素后者居上，父子之间子大于父。
3. 在IE下缺省值为0，FF下缺省为auto。
4. 对于定位元素，非同级关系或非父子关系元素之间比较大小，须要回溯到其为兄弟关系的两个祖先元素上，先比较这两个元素的z-index值，值大的元素所有的后代元素，均超过值小的元素及其后代元素。

举例：结构代码

	<div class=” father1” style=”z-index:1”></div>
	<div class=” father2”>
		<div class=”son2” style=”z-index:5”></div>
	</div>

FF下效果（希望看到的）：

![ie6bug](/img/ie6bug11.png)
 

IE6下效果（不希望看到的）：

![ie6bug](/img/ie6bug12.png)

分析：
根据④，son2与father1比较，就要先比较father1和father2的z-index。
根据③，IE6下father2的缺省值为0，那么father2及其所有后代元素都不可能位于father1上。而FF下，father2的缺省值为auto，可以理解为“随便，不关我事”，没有严格的层级制度，son2就位于了father1之上。

解决方案：给Father2设置z-index大于等于1的值即可。

