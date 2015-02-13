title: 雅虎科技频道纯图片布局的实现（下）
date: 2014-06-30 13:06:38
tags: YahooTech 雅虎科技 纯图片 横向瀑布流
---

![雅虎科技](/img/yahooTech1.jpg)

上一篇主要从外观上介绍了YahooTech的布局方式，本篇着重代码实现。代码实现经历了两个版本，以下是V1.0的算法：

1、 每张图片的宽度都设置为百分比，当屏幕resize时不需要额外处理。
2、 受启发于媒体查询根据屏幕宽度设置多种区间匹配样式，故设置0~320、320~1024、1024～1600、1600~+∞四个区间。

0~320：手机设备，每行只显示一个

320~1024：平板设备，每行显示二个或三个。第一个宽度为30~60%，第二个宽度为20~30%，或是当第一个宽度大于50%时，第二个占满该行。否则第三个占满该行。

1024~1600：桌面，每行显示三个或四个，类似上面的随机取值。

1600~+∞：超大宽屏，每行显示width/400张图片，每张图片宽度为350~450的随机值。

3、根据每张图片的实际宽高和显示宽度百分比计算其显示高度，并在当前行布局完成时取得当前行所有图片的最小显示高度作为当前行的显示高度。

	insertItem : function(item)
    {
        var divItem = $(item);
        var randomWidth = 0;
        //手机端每行只显示一个
        if(G_layout_options.clientW <= 320){
            randomWidth = 100;
            this.isFullROW = true;
            this.fullNum = 1;
        }
        //pad端每行显示二个或三个
        else if(G_layout_options.clientW <= 1024){
            if(this.itemsWidth.length == 0){
                randomWidth = Math.random() * (60 - 30) + 30;
            }
            else if(this.itemsWidth.length == 1){
                if(parseInt(this.itemsWidth[0]) < 50){
                    randomWidth = Math.random() * (30 - 20) + 20;
                }
                else{
                    randomWidth = 100 - this.itemsWidth[0];
                    this.isFullROW = true;
                    this.fullNum = 2;
                }
            }
            else if(this.itemsWidth.length == 2){
                randomWidth = 100 - this.itemsWidth[0] - this.itemsWidth[1];
                this.isFullROW = true;
                this.fullNum = 3;
            }
        }
        //desktop显示三个或四个
        else if(G_layout_options.clientW <= 1600){
            if(this.itemsWidth.length == 0){
                randomWidth = Math.random() * (40 - 20) + 20;
            }
            else if(this.itemsWidth.length == 1){
                if(parseInt(this.itemsWidth[0]) < 30){
                    randomWidth = Math.random() * (30 - 20) + 20;
                }
                else{
                    randomWidth = Math.random() * (40 - 30) + 30;
                }
            }
            else if(this.itemsWidth.length == 2){
                if(parseInt(this.itemsWidth[0] + this.itemsWidth[1]) < 60){
                    randomWidth = Math.random() * (30 - 20) + 20;
                }
                else{
                    randomWidth = 100 - this.itemsWidth[0] - this.itemsWidth[1];
                    this.isFullROW = true;
                    this.fullNum = 3;
                }
            }
            else if(this.itemsWidth.length == 3){
                randomWidth = 100 - this.itemsWidth[0] - this.itemsWidth[1] - this.itemsWidth[2];
                this.isFullROW = true;
                this.fullNum = 4;
            }
        }
        //超大屏
        else{
            this.fullNum = parseInt(G_layout_options.contentW / 400);
            if(this.itemsWidth.length < this.fullNum - 1){
                randomWidth = (Math.random() * (450 - 350) + 350) / G_layout_options.contentW * 100;
            }
            else{
                var itemsWidthSumTmp = 0;
                for(var i = 0; i < this.itemsWidth.length; i++){
                    itemsWidthSumTmp += this.itemsWidth[i];
                }
                randomWidth = 100 - itemsWidthSumTmp;
                this.isFullROW = true;
            }
        }
        this.itemsWidth.push(randomWidth);
        divItem.css("width", randomWidth + "%");
        var that = this;
        divItem.appendTo(this.element);
        this.calcuRowHeight(divItem, randomWidth);
    },

	calcuRowHeight: function(divItem, randomWidth){
	        var renderHeight = parseFloat(divItem.find("img").attr("data-height")) * (randomWidth * (G_layout_options.contentW - 20) - 1000) / (parseFloat(divItem.find("img").attr("data-width")) * 100);
	        this.itemsHeight.push(renderHeight);
	        if(this.isFullROW && this.itemsHeight.length == this.fullNum){
	            //console.log(this.itemsHeight);
	            var minHeight = Math.min.apply(Math,this.itemsHeight);
	            this.element.css({"height":minHeight > 500? 500:minHeight});
	            //每行显示两个时重新取bigger类型图片
	            if(this.fullNum == 2){
	                this.element.find(".content-img").each(function(){
	                    $(this).attr("src",$(this).attr("src").replace("common","bigger"));
	                })
	            }

	        }
	    }

显示效果如下图：

![立方媒体](/img/yahooTech3.jpg)

其实这个算法很low，基本上就是一些随机值凑数，被吐槽都是卡卡西风格，这种不顾图片实际宽高而采用随机宽度的做法太不接地气，效果也不好。于是v2.0采用了新的解决方案：

1、 遍历待排列的数据块Blocks，取得一个availableRow，可能是新的一行，也可能是未满行，将当前数据块Block试插入此行。
2、 所谓试插入，就是计算新数据插入后的当前行宽度是否超出最大宽度，超出也没有关系，顺势按比例压缩计算试插入的行高。attempHeight = MaxWith/Sum(width/height)。此时设置一个高度边界值300，当行高小于300时影响效果，故试插入失败。其余情况皆为成功。
3、 试插入失败意味着当前行剩余空间过小，不适合再插入数据，故创建新行，新行变成了当前行。
4、 执行DOM插入操作。
5、 重新计算行高，并以此设置图片显示宽高，如果当前图片宽度与需要显示的宽度不符，为避免图片拉伸影响效果可重新获取对应尺寸的图片URL。排列完成后显示当前行。

在此方案中图片宽度使用固定值不再使用百分比，能够更大限度地因图制宜。只是窗口resize时需要重排。最大的亮点是行高的计算的方法：attempHeight = MaxWith/Sum(width/height)，真的是很简单的四则运算提供了一个很大的突破口。

	showBlocks : function(items)
    {   
        for(var i = 0; i < items.length; i++)
        {
            var block = items[i];
            var row = this.getAvailableRow();
            var rowIndex = row.getRowIndex();
            if(!row.attemptInsertBlock(block)){
                row = this.createNewRow();
            };
            row.insertBlock(block);
        }
    },

	getAvailableRow : function()
	    {
	        var lastRow = this.getLastRow();
	        if(lastRow == null)
	            lastRow = this.createNewRow();
	        else
	        {
	            var enough = lastRow.checkFullRow();
	            if(enough){
	                lastRow = this.createNewRow();
	            }
	        }
	        return lastRow;
	    },

	attemptInsertBlock : function(block)
	    {   
	        //边界测试
	        if(this.rowWidth + block.width - 10 > G_layout_options.contentW){
	            this.rowWidth = G_layout_options.contentW;
	            this.attempHeight = (G_layout_options.contentW - (this.blockWidths.length + 1) * 10 ) / (this.rowRadio + block.aspectRadio);
	            if(this.attempHeight < 300){
	                this.layoutBlock();
	                return false;
	            }
	            return true;
	        }
	        return true;
	    },

	insertBlock : function(block)
	    {
	        //更新视图
	        this.element.append(block.element);
	        //更新Model
	        this.blocks.push(block);
	        this.attempHeight = this.attempHeight == 0? block.height : this.attempHeight;
	        this.rowWidth += block.width + 10;
	        this.rowRadio += block.aspectRadio;
	        this.blockWidths.push(block.width);
	        this.blockHeights.push(block.height);
	        //边界测试
	        if(this.rowWidth - 10 > G_layout_options.contentW){
	            this.layoutBlock();
	        }
	    },

	layoutBlock : function()
    {   
        this.rowWidth = G_layout_options.contentW;
        this.attempHeight = (G_layout_options.contentW - this.blockWidths.length * 10 ) / this.rowRadio;
        //设置行高
        this.element.css({"height":this.attempHeight});
        for(var i = 0; i < this.blocks.length; i++){
            //更新Model
            var _block = this.blocks[i];
            _block.setContainer(this);
            _block.height = this.attempHeight;
            _block.width = _block.aspectRadio * this.attempHeight;
            if(_block.width > 400){
                _block.fetchImage = true;
                _block.src= _block.src.replace("common","bigger");
            }
            //更新视图
            _block.element.find("img").attr("width",_block.width);
            _block.element.find("img").attr("height",_block.height);
            if(_block.fetchImage){
                _block.element.find("img").attr("src",_block.src);
            }
        }
        this.isFullROW = true;
        //显示当前行
        this.show();
    },

效果图如下：

![立方媒体](/img/yahooTech4.jpg)

有没有觉得顿时高大上了很多，完整效果请移步：<a href="http://www.l99.com/media/sex" title="立方媒体" target="blank">http://www.l99.com/media/sex</a>

