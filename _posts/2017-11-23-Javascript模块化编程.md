---
layout: post
title: "Javascript模块化编程"
date: 2017-11-23   
tag: javascript 
---


### Javascript模块化编程

* 原始写法

```javascript
function m1(){
}

function m2(){

}
```
模块就是实现特定功能的一组方法。只要把不同的函数简单的放在一起，就算是一个模块。这种做法的缺点就是，污染了全局变量，无法保证不与其他模块发生变量名冲突，而且模块成员之间看不出直接关系。

* 在原始写法上进行改进，原始写法变为对象写法

```javascript
var module = new Object({
	_count:0,
	m1:function(){
		console.log("执行m1函数");
	},
	m2:function(){
		console.log("执行m2函数");
	}
});
```
所以调用m1或者m2的时候直接调用对象的属性。但这样的写法会暴露所有模块成员，内部状态可以被外部改写。

* 在对象写法上进行改进，使用立即执行函数写法

使用`立即执行函数`，可以达到不暴露私有成员的目的

```javascript
var module=(function(){
	var _count=0;
	var m1 = function(){
		console.log("执行m1函数");
	};
	var m2 = function(){
		console.log("执行m2函数");
	};

	return {
		m1:m1,
		m2:m2
	};
})();
```

* 放大模式

如果一个模块很大，必须分成几个部分，或者一个模块需要继承另一个模块，这时就有必要采用`放大模式`

```javascript
var module = (function(mod){
	mod.m3 = function(){
		console.log("添加了模块方法3");
	};

	return mod;
})(module);
```

* 宽放大模式

在浏览器环境中，模块的各个部分通常都是从网上获取的，有时无法知道哪个部分会先加载。如果采用`放大模式`,有可能加载一个不存在空对象，这时就要采用`宽放大模式`

```javascript
var module = (function(mod){
	//.....
	return mod;
})(window.module||{});
```

* 输入全局变量，模块内部用全局变量的别名

```javascript
var module = (function($){
	//....
})(jQuery);
```

### 模块的规范

因为有了模块，我们就可以更方便地使用别人的代码，想要什么功能，就加载什么模块。但是，这样做有一个前提，那就是大家必须以同样的方式编写模块，否则你有你的写法，我有我的写法，就完全乱了。目前通行的Javascript模块规范共有两种:CommonJS与AMD。

```javascript
//require是一个全局加载模块函数
var math = require('math');
math.add(2,3);//5
```

这在浏览器端就碰到一个巨大的问题，要执行`math.add(2,3)` 就必须等math.js加载完成，也就是说，如果加载时间很长，整个应用就会停在那里等，浏览器会出现假死。但是服务器端的node.js是可以同步加载的，所以浏览器端只能用异步加载。所以AMD就出现了`Asynchronous Module Definition`，它采用异步方式加载模块，模块的加载不影响它后面语句的运行，所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。

AMD 也采用require()语句加载模块，但是不同于CommonJS，它要求两个参数`reuire([module],callback)`，第一个参数`[module]`是一个数组，里面的成员就是要加载的模块，第二个参数`callback`，则是加载成功之后的回调函数。

```javascript
require(['math'],function(math){
	math.add(2,3);
});
```

### 为什么需要require.js

最早的时候，所有JavaScript代码都写在一个文件里面，只要加载这个文件就够了。后来代码越来越多，一个文件不够了，就必须分成多个文件，依次加载，如下面的模式

```html
<script src="1.js"></script>
<script src="2.js"></script>
<script src="3.js"></script>
<script src="4.js"></script>
```

这段代码依次加载多个js文件，这样的写法有很大的缺点。缺点一:加载的时候，浏览器会停止网页渲染，加载文件越多，网页失去响应的时间就会越长;缺点二:由于js文件之间存在依赖关系，因此必须严格保证加载顺序，比如`2.js`依赖于`1.js`那么`1.js`必须放在`2.js`前面，依赖性最大的模块一定要放到最后加载，当依赖关系很复杂的时候，代码的编写和维护都会越来越难维护了。

require.js的出现就是解决:1,实现js文件的异步加载，避免网页失去响应；2,管理模块之间的依赖性，便于代码的编写和维护。

### require.js的写法

```html
//require.js加载

//普通常用写法
<script src="js/require.js"></script>
<script src="js/business.js"></script>

//异步加载写法，避免阻塞网页Dom渲染,IE支持`defer`,其他浏览器支持 `async="true"`
<script src="js/require.js" defer async="true" data-main="js/business.js"></script>

```

* 业务模块写法

下面说说`main.js`的写法

```javascript
//main.js
require(['moduleA','moduleB','moduleC'],function(moduleA,moduleB,moduleC){
	//main.js中的业务代码
});

//真实中
require(['jquery','underscore','backbone'],function($,_,Backbone){
	//main.js中的业务代码
});

```

* 模块名与模块路径定义

看到上面的写法，我们发现`jquery`,`underscore`,`backbone` 这些是从哪里来的，我们知道require.js就是解决异步加载与模块依赖问题，想到了估计跟定义`模块名称`与`模块的js文件路径`有关，下面就给出如何定义模块。

```javascript
//直接单独定义路径,路径默认的文件后缀名为`.js`,所以`jquery.min.js` 可以写成 `jquery.min`
//这是相对路径写法，相对于main.js文件的路径
require.config({
	paths:{
		"jquery":"jquery.min",
		"underscore":"underscore.min",
		"backbone":"backbone.min"	
	}
});
//绝对路径写法
require.config({
	paths:{
		"jquery":"/js/lib/jquery.min",
		"underscore":"/js/lib/underscore.min",
		"backbone":"/js/lib/backbone.min"	
	}
});
//基目录写法
require.config({
	baseUrl:"/js/lib",
	paths:{
		"jquery":"jquery.min",
		"underscore":"underscore.min",
		"backbone":"backbone.min"	
	}
});
```

* 功能模块写法

```javascript
//math功能模块，math.js，不依赖于其他模块的情况写法
define(funciton(){
	var add = function(x,y){
		return x+y;
	};
	return{
		add:add
	};
});

//依赖于`commonLib`模块的写法
define(['commonLib'],function(commonLib){
	function isTelphoneNumber(phoneNumber){
	    var isPhoneNumber=commonLib.checkPhoneNumber(phoneNumber);
	    return isPhoneNumber;
	}
	return {
		isTelPhoneNumber:isTelphoneNumber
	};
});
```

* 加载非规范的模块

理论上,require.js加载的模块,必须是按照AMD规范、用define()函数定义的模块。但是实际上，虽然已经有一部分流行的函数库(比如jQuery)符合AMD规范，更多的库并不符合规范。鉴于那种情况，require.js还是支持非规范的模块。

这样的模块在用require()加载前，要先用require.config()方法，定义它们的一些特征。

```javascript
//比如 underscore与backbone这两个库，都没有采用AMD规范编写。如果需要加载他们的话，就必须定义它们的特征。
require.cnfig({
	shim:{
		'underscore':{
			exports:'_'
		},
		'backbone':{
			deps:['underscore','jquery'],
			exports:'backbone'
		}
	}
});
```

属性`shim` 专门用来配置不兼容的模块。1：每个模块的`exports`值的输出变量名，表明这个模块外部调用时的名称；2：`deps`数组，表明该模块的依赖性。

* require.js插件

require.js还提供一系列插件，实现一些特定的功能。
domready插件，可以让回调函数在页面Dom结构加载完成后再运行
```javascript
require(['domready!'],function(doc){
	//called once the dom is ready
});
```
text、image插件，则是允许require.js加载文本与图片文件。

```javascript
define([
	'text!review.txt',
	'image!cat.jpg'],
	function(review,cat){
		console.log(review);
		document.body.appendChild(cat);
	}
);
```
类似的插件还有Json、mdown，用于加载json文件与markdown文件。