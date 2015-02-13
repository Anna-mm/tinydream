title: 跨域请求如何破
date: 2013-11-29 16:46:33
tags: 跨域 jquery跨域
---
近日，前端在尝试跨域请求API，接受纯粹的JSON数据，绕过后端大婶们，他们总是这个不能做，那个影响性能。但是，请求一开始就报错，跨域总是这么屌，轻易让你搞定，就不是他的范儿。
先看一下错误信息：

Chrome：Uncaught SyntaxError: Unexpected token :

Firefox：![跨域请求](/img/crossdomain1.png)

代码：

    $.ajax({
            async: true,
            url: 'http://dbapi.xy.l99.com/dovebox/pintimes/mediatype',
            dataType: 'jsonp',
            data:{media_type: 0, limit: 20},
            method: "GET",
            error: function (jqXHR, textStatus, errorThrown){
                  //……;
            },
            success: function (data, textStatus, jqXHR) {
                   //……;
            }
    });
从Firefox的错误信息来看，是数据格式的问题，而返回的JSON数据肯定不存在问题。根据stackoverflow上的说法把dataType从“jsonp”替换成“json”，或在url后面加上callback=?等等都不能解决。我想到的种种错误可能都不是，那肯定是后端的问题了。但是，这个理由是不能拿出去跟后端程序员说的，除非告诉他错误在哪，继续调查下。

我们知道，跨域请求本质上并不是ajax，而是创建了script标签，将请求地址赋给src，因为script标签可以无条件执行，不受限于同源策略。于是，尝试在页面上增加：

    <script type="text/javascript" src="http://dbapi.xy.l99.com/dovebox/pintimes/mediatype" ></script>
猜猜结果如何？

报了同样的错误信息，不难得知，浏览器加载了这个假的“js文件”，但是执行的时候发现这段代码并不符合js语法，所以报了以上错误。

回到跨域请求的本质上来，后端在返回数据时需要在数据外层包装一个函数，函数名由前端传递，默认是jquery+随机数。包装函数的目的一方面是使其符合js语法，另一方面是将数据以参数形式传递过来，方便获取。除了包装函数，还有其他办法吗？把数据赋给一个变量岂不是更方便？有机会可以再实现一下跨域请求。

那到底是不是后端未处理的原因？我们用fiddler模拟返回数据看看。

    $.ajax({
            async: true,
            url: 'http://dbapi.xy.l99.com/dovebox/pintimes/mediatype',
            dataType: 'jsonp',
            jsonpCallback: "aa", 
            data:{media_type: 0, limit: 20},
            method: "GET",
            error: function (jqXHR, textStatus, errorThrown){
                  //……;
            },
            success: function (data, textStatus, jqXHR) {
                   //……;
            }
    });
由于在请求中添加callback=?会导致函数名随机生成，不易控制，所以自定义一个：jsonpCallback: "aa",

在fiddler中设置规则，当请求某个URL时，返回本地数据（特别增加了函数包装）。

    aa({"status":"1","code":"200","data":{......}});
![跨域请求](/img/crossdomain1.png)
果然，一切搞定。API啊，API，你竟然不支持跨域请求？？？！！！！

最后，我们还是从jquery源码中走一下跨域请求的流程~~
请求入口：

    transport.send( requestHeaders, done );
    // requestHeaders :"{"Accept":"text, text/javascript, application/javascript, application/ecmascript, application/x-ecmascript, */*; q=0.01"}"
发送请求的本质：

    send: function( _, callback ) {
        script = document.createElement("script");
        script.async = true;
        if ( s.scriptCharset ) {
            script.charset = s.scriptCharset;
        }
        script.src = s.url;
        // Attach handlers for all browsers
        script.onload = script.onreadystatechange = function( _, isAbort ) {                    
            if ( isAbort || !script.readyState || /loaded|complete/.test( script.readyState ) ) {
                // Handle memory leak in IE
                script.onload = script.onreadystatechange = null;
                // Remove the script
                if ( script.parentNode ) {
                    script.parentNode.removeChild( script );
                }
                alert(script.innerHTML);
                // Dereference the script
                script = null;
                // Callback if not abort
                if ( !isAbort ) {
                    callback( 200, "success" );
                }
            }
        };
        // Circumvent IE6 bugs with base elements (#2709 and #4378) by prepending
        // Use native DOM manipulation to avoid our domManip AJAX trickery
        head.insertBefore( script, head.firstChild );
    }
执行js代码：

    window[ callbackName ] = function() {
        responseContainer = arguments;
    };
预先注册了window[“aa”] = function(){};当假的js加载后立即执行此函数，将参数保存到变量responseContainer中。然后进入onload回调函数，后续会将responseContainer返回给response对象，以及错误处理等等。

结论：出现文章开头的错误信息，是由于后端没有做跨域处理（这句话是给看了开头直接看结尾的人的）。

