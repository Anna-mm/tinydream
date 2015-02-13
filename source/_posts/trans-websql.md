title: 【翻译】Web SQL数据库介绍
date: 2013-10-08 20:34:26
tags: WEB SQL 数据库
---

原文地址：[http://html5doctor.com/introducing-web-sql-databases/](http://html5doctor.com/introducing-web-sql-databases/)

Web SQL数据库事实上不是HTML5规范的一部分，而是允许开发者构建完全成熟的web应用程序的规范套件的一部分，所以它是关于我们深入浅出的时间的。（请注意截止到11月18日W3C在Web SQL数据库规范上不再有所动作。）

![websql数据库](/img/websql1.png)

##箱子里是什么

如果你没有从过于繁琐的规范标题中猜到，Web SQL数据库是将SQL引入到客户端的规范。如果你有后端开发的背景，那么你可能会熟悉SQL，如满身是泥的猪一样快乐。如果没有，你可能在伺处碰壁之前先了解一个SQL,Google就是你的朋友。

规范基于SQLite (3.1.19)，而且来源于MySQL本身，几乎是完全一样的（很抱歉用这种笼统绝对的说法）。有关Web SQL数据库工作的一个例子，看看Twitter HTML5 chatter demo，它使用SQL WHERE子句来缩小最近关于HTML5在Twitter上的聊天。（在Safari，Chrome和Opera 10.50上运行）

我要在这里说规范中的三个核心方法，：

**1、openDatabase**
**2、transaction**
**3、executeSql**

各个浏览器对此支持不一，目前只有Webkit（Safari、SafariMobile和Chrome）和Opera 10.50支持。Bruce Lawson研究员告诉我，火狐则持观望态度，因为他们觉得有一个比SQLite更好的实现（无论他们如何选择，我都希望这些方案是相似的，）。无论哪种方式，我肯定会建议你去浏览SQLite的文档中所有可用的方法。

目前支持的不一致性，Webkit对这种数据库规范实施了一段时间已是既定事实，W3C上的规范现在稍稍领先于Safari中的实现，同时其他WebKit仍在追赶。另一方面，尽管Opera也只是才支持，但是它更接近规范（后面我会提到这种差异化）。接下来，让我与其共舞吧！

##创建和打开数据库

如果你试图打开一个不存在的数据库，API会动态创建，你也不用担心关闭数据库。要创建和打开数据库时，使用下面的代码：

	var db = openDatabase('mydb', '1.0', 'my first database', 2 * 1024 * 1024);

该方法使用四个参数，第一个参数是数据库名称，第二个参数是版本号，第三个参数是数据库描述，第四个参数是数据库大小。还缺一项功能（不确定什么时候加上）就是第五个参数——回调方法。数据库创建之后调用回调方法。没有这项功能，数据库仍然被动态创建并版本正确。
该方法的返回值包括transaction方法，通过调用此方法执行SQL查询。

##估计数据库大小

通过测试，只有Safari浏览器会在创建数据库超过默认的数据库大小5MB时提示用户，提示见下图,询问是否要授予数据库权限扩展到下一级数据库大小 - 5，10，50，100和500MB。而Opera创建指定大小的数据库，可在后续修改。
![websql数据库](/img/websql.png)

##版本

我可能是错的，但到目前为止测试过的一切均说明，SQL数据库中的版本是borked。问题是这样的：如果数据库升级到2.0版本（,对于1.0版本，有一些重要的schema发生变化），你怎么知道，访客是在版本1.0还是2.0？版本号是OpenDatabase方法的必需参数，所以你在打开数据库之前必须知道版本号。否则，会抛出一个异常。

此外，changeVersion方法是用来改变数据库的版本的，但是在WebKit上还不完全支持，Chrome和Opera已支持。无论如何，如果我不能确定用户是在哪个版本的数据库，那么我不能升级。

一种可能的解决方法是保持状态数据库，像在MySQL数据库中的‘mysql’ 的。这样的话，在这种状态数据库只会有一个版本的，在此，你会记录控制你的应用程序的任何数据库的当前版本。这是一个hack，但它奏效。

##事务

现在，我们已经打开数据库，我们可以创建事务。为什么使用事务而不是只运行SQL？事务使我们能够回滚。这意味着，如果一个事——-其中可能包含一个或多个SQL语句——失败了（无论是SQL或在事务处理中的代码），对数据库的更新都没有提交，就像什么都没有发生。

事务中有失败和成功的回调，这样你就可以管理错误，但重要的是要明白，事务有能力回滚那些修改。事务是一个简单的功能，包含了一些代码：
	
	var db = openDatabase('mydb', '1.0', 'my first database', 2 * 1024 * 1024);
	db.transaction(function (tx) {
	//这里是一个事务
	//通过tx对象处理sql
	});

我最近在[html5demos.com](http://html5demos.com)上传了一个demo，演示事务回滚[Web SQL database rollback demo](http://html5demos.com/database-rollback)
每晚构建的浏览器，我们也有db.readTransaction，只允许读取在数据库上运行的语句。假设只读的readTransaction比读写的transaction有性能优势，最有可能就是在表锁定上。

现在，我们已经得到了我们的事务对象（如上例中的tx），我们将要运行一些SQL！

##执行SQL

这是所有SQL善良的爱的漏斗.ExecuteSQL用于读写，包括SQL注入映射，并提供一个回调方法处理任何查询结果。一旦我们有一个事务对象，我们可以调用EXECUTESQL：

	var db = openDatabase('mydb', '1.0', 'my first database', 2 * 1024 * 1024);
	db.transaction(function (tx) {
		tx.executeSql('CREATE TABLE foo (id unique, text)');
	});

这将在名为“mydb”数据库中创建一张表“foo”。需要注意的是，如果数据库中已经存在这张表，该事务失败，任何后续的SQL也不会运行。因此，我们可以用另一种事务，或者我们只能判断这张表不存在时创建，这是我现在要做的，这样就可以在同一事务中插入新的一行：

	var db = openDatabase('mydb', '1.0', 'my first database', 2 * 1024 * 1024);
	db.transaction(function (tx) {
		tx.executeSql('CREATE TABLE IF NOT EXISTS foo (id unique, text)');
		tx.executeSql('INSERT INTO foo (id, text) VALUES (1, "synergies")');
	});

现在，表中有一条记录。如果我们想从用户或外部获得文本呢？我们想要确保它不能危及数据库的安全性（使用像SQL注入这种讨厌的东西）。executeSql的第二个参数给出了字段的映射关系，如下：

	tx.executeSql('INSERT INTO foo (id, text) VALUES (?, ?)', [id, userValue]);

id和userValue是外部变量，分别映射到数组参数中的？。

最后，如果我们要获取表中的值，可以使用一个回调获取结果：

	tx.executeSql('SELECT * FROM foo', [], function (tx, results) {
		var len = results.rows.length, i;
		for (i = 0; i < len; i++) {
			alert(results.rows.item(i).text);
		}	
	});
（请注意，在这个查询中，有没有被映射的字段，但为了使用第三个参数，我需要传递一个空数组作为第二个参数。）

回调返回一个transaction对象和results对象。results对象包含一个rows对象，这是一个类数组但不是一个数组。它有长度属性，可以通过results.rows.item(i)访问到每一行，这里的i是当前行的索引。例如，如果表中有name和age字段，用results.rows.item(i).age访问age字段。

至此，你应该开始尝试Web SQL数据库了。我敢肯定，将会出现小型JavaScript库来帮助支持Web SQL数据库的工作。如果你想了解更多（无耻的自我推销）我刚完成了 Introducing HTML5一书的存储章节，这本书是我与同事Bruce合作的，也去看一下那个家伙！

Demos

[HTML5 demo showing simple database usage](http://html5demos.com/database)
[HTML5 demonstration of a transaction rolling back](http://html5demos.com/database-rollback)
[Demo showing time range selection using SQLite](http://rem.im/html5-tweet-time-range.html)


