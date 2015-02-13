title: jQuery事件系统三
date: 2014-10-13 15:34:49
tags: jQuery event 事件 数据缓存 W3C 浏览器
---
上一节通过给DOM元素添加自定义数据引出jQuery.data这个数据缓存模块，这是数据缓存系统的另一大重头戏，这一节就好好说说jQuery.data。再看jquery源码之前大概大概明确几个要点：

1、jQuery.expando = "jQuery" + ( jQuery.fn.jquery + Math.random() )，即jQuery＋版本号＋随机数；

2、jQuery.data方法通过参数的不同，可以有几种用法：为DOM元素或Javascript对象设置任意类型的数据，或返回指定名称的数据，或返回关联的数据缓存对象。

3、数据缓存对象上区分内部数据和自定义数据，避免jQuery内部使用的数据和用户自定义的数据发生冲突。
方法._data()设置的数据为内部数据，直接存储在关联的数据存储对象上，如事件缓存数据。
方法.data()设置的数据为自定义数据，存储在关联的数据缓存对象的属性data上；如用户通过data-*自定义数据。

 在上节栗子中设置content元素的自定义属性如下，存储位置如下：

 	$(".content").data({"tagName":"div","className":"content"});

 ![数据缓存系统](/img/jsevent3.png)

 总结一下：上节中通过jQuery._data设置DOM元素的内部属性，事实上是通过jQuery.data实现的，只是传递pvt参数为true（用于内部属性或自定义属性的标识），在jQuery.cache上顺序挂上key值为1、2、3...依此类推的数据缓存对象。同时设置DOM元素本身的属性elem[ jQuery.expando ] = id；以此来关联数据缓存系统。

 ![数据缓存系统](/img/jsevent4.png)

源码如下：

	jQuery.extend({
		cache: {},//事件缓存系统
		deletedIds: [],
		uuid: 0,// uuid初始化
		// 生成类似于jQuery18005268001211807132这样的随机数（），避免与用户的自定义属性名冲突
		expando: "jQuery" + ( jQuery.fn.jquery + Math.random() ).replace( /\D/g, "" ),
		//jQuery.noData中存放了不支持扩展属性的embed、object、applet元素的节点名称。
		noData: {
			"embed": true,
			"object": "clsid:D27CDB6E-AE6D-11cf-96B8-444553540000",//用于检查object元素是否是flash
			"applet": true
		},

		//判断一个DOM元素或javascript对象是否有关联的数据
		hasData: function( elem ) {
			elem = elem.nodeType ? jQuery.cache[ elem[jQuery.expando] ] : elem[ jQuery.expando ];
			return !!elem && !isEmptyDataObject( elem );
		},

		//为DOM元素或javascript对象设置任何类型的数据
		data: function( elem, name, data, pvt /* Internal Use Only */ ) {
			//是否可以附加数据，不可以的话直接返回
			if ( !jQuery.acceptData( elem ) ) {
				return;
			}
			var thisCache, ret,
				//jQuery.expando是一个唯一的字符串，jquery对象产生的时候就生成
				internalKey = jQuery.expando,
				getByName = typeof name === "string",
				// 区分DOM对象和javascript对象，如果是DOM元素，为了避免javascript和DOM元素之间循环引用导致的浏览器（IE6、7）垃圾回收机制不起作用，要把数据存储在全局缓存对象jQuery.cache中；对于Javascript对象，垃圾回收机制自动发生，数据可以之间存储在javascript对象上。
				isNode = elem.nodeType,
				......
			//如果关联id不存在，则分配一个。
			if ( !id ) {
				if ( isNode ) {
					elem[ internalKey ] = id = jQuery.deletedIds.pop() || ++jQuery.uuid;
				} else {
					id = internalKey;
				}
			}
			//如果数据缓存对象不存在，就初始化为空对象
			if ( !cache[ id ] ) {
				cache[ id ] = {};
				......
			}
			// 如果参数name是对象或函数，则批量设置数据
			if ( typeof name === "object" || typeof name === "function" ) {
				if ( pvt ) {
					cache[ id ] = jQuery.extend( cache[ id ], name );
				} else {
					cache[ id ].data = jQuery.extend( cache[ id ].data, name );
				}
			}
			thisCache = cache[ id ];
			// 如果参数pvt为true，则设置或读取内部数据，内部数据存储在关联的数据存储对象上；
			// 如果参数pvt为false，则设置或读取自定义数据，自定义数据存储在关联的数据缓存对象的属性data上。
			if ( !pvt ) {
				if ( !thisCache.data ) {
					thisCache.data = {};
				}
				thisCache = thisCache.data;
			}
			//如果参数data不是undefined，则把参数data设置到属性name上。把name统一换成驼峰式。
			if ( data !== undefined ) {
				thisCache[ jQuery.camelCase( name ) ] = data;
			}
			// 参数name为字符串，data为null时读取单个数据。两次读取，使用name读取一次，使用驼峰式再读取一次。
			if ( getByName ) {
				ret = thisCache[ name ];
				if ( ret == null ) {
					ret = thisCache[ jQuery.camelCase( name ) ];
				}
			} else {
				//未传入参数name、data、则返回数据缓存对象。
				ret = thisCache;
			}
			return ret;
		},
		//用于移除通过jQuery.data()设置的数据
		removeData: function( elem, name, pvt /* Internal Use Only */ ) {
			......
		},
		// 设置或读取内部数据时使用
		_data: function( elem, name, data ) {
			return jQuery.data( elem, name, data, true );
		},
		// 判断DOM元素是否可以设置数据，通过检查DOM元素的节点名称在对象jQuery.noData中是否存在
		acceptData: function( elem ) {
			var noData = elem.nodeName && jQuery.noData[ elem.nodeName.toLowerCase() ];
			return !noData || noData !== true && elem.getAttribute("classid") === noData;
		}
	});

本节内容参考《jQuery技术内幕》第五章数据缓存Data