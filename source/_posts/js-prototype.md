title: 老调重弹：那些年一遍又一遍领悟过的原型链
date: 2014-02-12 15:22:48
tags: 原型链 继承
---
一过年吃了许多肉，见了许多人，又思考了许多人生大事，中了年后综合症的招，唯恐脑子不再灵光，荒废专业。特意把基础知识再梳理一下：

### 一、简单原型链：构造函数、原型、实例

![简单原型链](/img/prototype1.png)
这是Nicholas C. Zakas的《Javascript高级程序设计》P120的截图。有一个前端朋友说，他曾经怎么都不理解原型链直到看到此书此章节才如醍醐灌顶。我想，看一遍就领悟恐怕也是神人了。

这张图的重点总结如下：
1. 只要创建了一个新函数，就为该函数创建一个prototype属性
2. Prototype属性会自动获得一个constructor属性。
3. Constructor属性包含快一个指向prototype所在函数的指针。
4. 可为原型增加其他属性和方法
5. 创建新实例，其内部包含一个名为protyo (内部属性)的指针指向其构造函数的原型属性。

此外，还有两个特别的方法用来确定其内部关系：
1. 实例person1、person2与Person原型之间的关系
		alert(Person.prototype.isPrototypeOf(person1));//true
		alert(Person.prototype.isPrototypeOf(person2));//true来自实例本身
2. 我们知道，如果要读取person1的某个属性，搜索首先从实例本身开始，如果没有会继续搜索原型对象。那么如何知道搜索到的属性是来自实例本身还是原型？
		alert(person1.hasOwnProperty(“name”)) //true来自实例本身

**最后，原型对象有一个问题。**
原型中所有属性是被很多实例共享的，通过在实例上添加一个同名属性可以隐藏原型中的对应属性。但是如果这个属性是引用属性就有麻烦了。举例如下：

	function Person(){   }
	Person.prototype = {
	    constructor:Person,
	    name:"Nicholas",
	    age:29,
	    friends:["shelby","Court"],
	    sayName:function(){....}
	}
	var person1 = new Person();
	var person2 = new Person();
	person1.friends.push("Van");
	alert(person1.friends);//["shelby","Court","Van"]
	alert(person2.friends);//["shelby","Court","Van"]
	alert(person1.friends===person2.friends); //true

原因是friends数组存在于Person.prototype中，不是在person1中。如果是简单属性赋值比如person1.age = 30并不影响Person.prototype.age=29。

###二、继承原型链：

![简单原型链](/img/prototype2.png)
提示：所有函数的默认原型都是Object的实例，因此默认原型都会包含一个内部指针__proto指向Object.prototype。继承Object这部分未在图中体现。
下面我们研究继承是怎么实现的？

	function Animal(){
	    this.species = “动物”
	}
	Animal.prototype = {
	    say: function(){
	        alert(“I am a ”+ this.species +”,my name is XXX”);
	}
	}
	function cat(name,color){
	    this.name = name;
	    this.color = color;
	}

如何使”猫”继承“动物”？继承分两部分，分别是构造函数继承和原型继承。构造函数继承实现了对实例属性的继承，原型实现对原型属性和方法的继承。

####1. 构造函数继承

		function cat (name, color){
			Animal.call(this);
			this.name = name;
			this.color = color;
		}

####2. 原型继承

		Cat.prototype = new Animal();
		Cat.prototype.constructor = Cat;
		var cat1 = new Cat(“大毛”，“黄色”);
		alert（cat1.species）； //动物

此时，原型继承同时继承了构造函数中不变的属性species。事实上，不变的属性应该写入原型，作为原型属性。

####3. 直接原型继承
为什么要将Animal的实例作为Cat的原型呢？如果将Animal的原型赋值给Cat的原型如何呢？
改写Animal对象：

		function Animal(){}
		Animal.prototype.species = “动物”;
		Cat.prototype = Animal.prototype;
		Cat.prototype.constructor = Cat;
		var cat1 = new Cat(“大毛”，“黄色”);
		alert（cat1.species）； //动物

优点：不用执行和建立Animal实例，省内存
缺点：cat.prototype和animal.prototype都指向同一对象，对cat.prototype的修改都会反映到animal.prototype上。

####4. 对上个方案的改进——利用空对象作中介

		function extend(subclass, superclass) {
		        var F = function () {};
		        F.prototype = superclass.prototype;
		        subclass.prototype = new F();
		        subclass.prototype.constructor = subclass;
		}

####5. 拷贝继承
引颖同学曾大力推荐的自创类库中使用的是拷贝继承，将父对象的所有属性和方法拷贝进子对象。

		function extend （subclass, superclass）{
		        Var p = superclass.prototype;
		        Var c = subclass.prototype;
		        for(var i in p){
		            C[i] = p[i]
		        }
		}
