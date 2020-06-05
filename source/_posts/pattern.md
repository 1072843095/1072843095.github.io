---
title: JavaScript设计模式与开发实践笔记
date: 2018-01-29 14:33:19
categories: 学习笔记
---
js设计模式

### 单例模式

保证一个类仅有一个实例，并提供一个访问它的全局访问点。
#### 用代理实现单例模式

<!-- more -->
{% codeblock %}
var CreateDiv = function(html){
	this.html = html;
	this.init();
};

CreateDiv.prototype.init = function(){
	var div = document.createElement('div');
	div.innerHTML = this.html;
	document.body.appendChild(div);
};

//代理类
var ProxySingletonCreaterDiv = (function(){
	var instance;
	return function(html){
		if(!instance){
			instance = new CreateDiv(html);
		}
		return instance;
	}
})();

var a = new ProxySingletonCreaterDiv('sven1');
var b = new ProxySingletonCreaterDiv('sven2');
alert(a===b);  // true
{% endcodeblock %}

#### js中的单例模式

##### 使用命名空间,减少全局变量的数量
{% codeblock %}
var namespace1 = {
	a: function(){
		alert(1)
	   },
	b: function(){
		alert(2)
	   }
}
{% endcodeblock %}

##### 使用闭包封装私有变量，只暴露一些接口跟外界通信
{% codeblock %}
var user = (function(){
	var _name = 'sven',
		_age = 29;

	return{
		getUserInfo: function(){
			return _name+'-'+_age;
		}
	}
})();
{% endcodeblock %}

#### 惰性单例
惰性单例指的是在需要的时候才创建对象实例。
{% codeblock %}
//管理单例
var getSingle = function(fn){
	var result;
	return function(){
		return result || (result=fn.apply(this, argument));
	}
}

var createSingleIframe = getSingle( function(){
	var iframe = document.createElement ( 'iframe' );
	document.body.appendChild( iframe );
	return iframe;
});
document.getElementById( 'loginBtn' ).onclick = function(){
	var loginLayer = createSingleIframe();
	loginLayer.src = 'http://baidu.com';
};
{% endcodeblock %}

### 策略模式
定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。
{% codeblock %}
var strategies = {
	"S": function(salary){
		return salary*4;
	},
	"A": function(salary){
		return salary*3;
	},
	"B": function(salary){
		return salary*2;
	}
};

var calculateBonus = function(level, salary){
	return strategies[level](salary);
}

console.log(calculateBonus("S", 20000));   //80000
console.log(calculateBonus("A", 10000));   //30000
{% endcodeblock %}

#### 使用策略模式重构表单校验
{% codeblock %}
var strategies = {
	isNonEmpty: function(value, errorMsg){
		if(value === ''){
			return errorMsg;
		}
	},
	minLength: function(value, length, errorMsg){
		if(value.length < length){
			return errorMsg;
		}
	},
	isMobile: function(value, errorMsg){
		if(!/^1[3|5|8][0-9]{9}$/.test(value)){
			return errorMsg;
		}
	}
}

var Validator = function(){
	this.cache = [];//保存校验规则
};
Validator.prototype.add = function(dom, rules){
	var self = this;
	for(var i=0,rule; rule=rules[i++];){
		(function(rule){
			var strategyAry = rule.strategy.split(":"); // 把 strategy 和参数分开
			var errorMsg = rule.errorMsg;

			self.cache.push(function(){  // 把校验的步骤用空函数包装起来，并且放入 cache
				var strategy = strategyAry.shift(); // 用户挑选的 strategy
				strategyAry.unshift(dom.value);  // 把 input 的 value 添加进参数列表
				strategyAry.push(errorMsg);   // 把 errorMsg 添加进参数列表
				return strategies[strategy].apply(dom, strategyAry);
			})
		})(rule)
	}
};
Validator.prototype.start = function(){
	for(var i=0,validatorFunc; validatorFunc=this.cache[i++];){
		var msg = validatorFunc();  //开始校验，并取得校验后的返回信息
		if(msg){ //如果有确切的返回值，说明校验没有通过
			return msg;
		}
	}
}

//使用
var validataFunc = function(){
	var validator = new Validator();//创建一个validate对象
	/****添加一些校验规则****/
	validator.add(registerForm.userName, [{strategy:'isNonEmpty', errorMsg:'用户名不能为空'}, {strategy:'minLength:6', errorMsg:'用户名长度不能少于6位'}]);
	validator.add(registerForm.password, [{strategy:'minLength:6', errorMsg:'密码长度不能少于6位'}]);
	validator.add(registerForm.phoneNumber, [{strategy:'isMobile', errorMsg:'手机号码格式不正确'}]);

	var errorMsg = validator.start();//获取校验结果
	return errorMsg; //返回校验结果
}
var registerForm = document.getElementById('registerForm');
registerForm.onsubmit = function(){
	var errorMsg = validataFunc();   //如果errorMsg有确切的返回值，说明未通过校验
	if(errorMsg){
		alert(errorMsg);
		return false;//阻止表单提交
	}
}
{% endcodeblock %}

###  代理模式
代理模式是为一个对象提供一个代用品或占位符，以便控制对它的访问。

#### 虚拟代理实现图片预加载
在web开发中，图片预加载是一种常见的技术，如果直接给某个img标签节点设置src属性，由于图片过大或者网络不佳，图片的位置往往有段时间会是一片空白。常见的做法是先用一张loading图片站位，然后用异步的方式加载土坯阿尼，等图片加载好了再把它填充到img节点里，这种场景就很适合虚拟代理。
{% codeblock %}
var myImage = (function(){
	var imgNode = document.createElement('img');
	document.body.appendChild(imgNode);
	return{
		setSrc: function(src){
			imgNode.src = src;
		}
	}
})();

var proxyImage = (function(){
	var img = new Image();
	img.onload = function(){
		myImage.setSrc(this.src);
	}
	return {
		setSrc: function(src){
			myImage.setSrc( 'file:// /C:/Users/svenzeng/Desktop/loading.gif' );
			img.src = src;
		}
	}
})();
proxyImage.setSrc( 'http:// imgcache.qq.com/music/photo/k/000GGDys0yA0Nk.jpg' );
{% endcodeblock %}

#### 虚拟代理合并http请求
{% codeblock %}
var synchronousFile = function(id){
	console.log('开始同步文件，id为：'+id);
};

var proxySynchronousFile = (function(){
	var cache = [],   //保存一段时间内需要同步的id
		timer;    //定时器
	return function(id){
		cache.push(id);
		if(timer){ //确保不会覆盖已经启动的定时器
			return;
		}
		timer = setTimeout(function(){
			synchronousFile(cache.join(','));  //2秒后向本体发送需要同步的id集合
			clearTimeout(timer);  //清空定时器
			timer = null;
			cache.length = 0;  //清空id集合
		}, 2000);
	}
})();
var checkbox = document.getElementByTagName('input');
for(var i=0,c; c=checkbox[i++];){
	c.onclick = function(){
		if(this.checked === true){
			proxySynchronousFile(this.id);
		}
	}
}
{% endcodeblock %}

#### 用高阶函数动态创建代理
{% codeblock %}
/**************** 计算乘积 *****************/
var mult = function(){
	var a = 1;
	for ( var i = 0, l = arguments.length; i < l; i++ ){
		a = a * arguments[i];
	}
	return a;
};
/**************** 计算加和 *****************/
var plus = function(){
	var a = 0;
	for ( var i = 0, l = arguments.length; i < l; i++ ){
		a = a + arguments[i];
	}
	return a;
};
/**************** 创建缓存代理的工厂 *****************/
var createProxyFactory = function( fn ){
	var cache = {};
	return function(){
		var args = Array.prototype.join.call( arguments, ',' );
		if ( args in cache ){
			return cache[ args ];
		}
		return cache[ args ] = fn.apply( this, arguments );
	}
};
var proxyMult = createProxyFactory( mult ),
proxyPlus = createProxyFactory( plus );
alert ( proxyMult( 1, 2, 3, 4 ) ); // 输出：24
alert ( proxyMult( 1, 2, 3, 4 ) ); // 输出：24
alert ( proxyPlus( 1, 2, 3, 4 ) ); // 输出：10
alert ( proxyPlus( 1, 2, 3, 4 ) ); // 输出：10
{% endcodeblock %}

### 迭代器模式
迭代器模式是指提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。js内置了迭代器。forEach，for in等。
{% codeblock %}
var getActiveUploadObj = function(){
	try{
		return new ActiveXObject( "TXFTNActiveX.FTNUpload" ); // IE 上传控件
	}catch(e){
		return false;
	}
};
var getFlashUploadObj = function(){
	if ( supportFlash() ){ // supportFlash 函数未提供
		var str = '<object type="application/x-shockwave-flash"></object>';
		return $( str ).appendTo( $('body') );
	}
	return false;
};
var getFormUpladObj = function(){
	var str = '<input name="file" type="file" class="ui-file"/>'; // 表单上传
	return $( str ).appendTo( $('body') );
}; 

var iteratorUploadObj = function(){
	for ( var i = 0, fn; fn = arguments[ i++ ]; ){
		var uploadObj = fn();
		if ( uploadObj !== false ){
			return uploadObj;
		}
	}
};
var uploadObj = iteratorUploadObj( getActiveUploadObj, getFlashUploadObj, getFormUpladObj );
{% endcodeblock %}
获取不同上传对象的方法被隔离在各自的函数里互不干扰，try 、 catch 和 if 分支不再纠缠在一起，使得我们可以很方便地的维护和扩展代码。比如，后来我们又给上传项目增加了 Webkit控件上传和 HTML5上传，我们要做的仅仅是下面一些工作

### 发布-订阅模式（观察者模式）
它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。
{% codeblock %}
var event = {
	clientList: [],
	listen: function( key, fn ){
		if ( !this.clientList[ key ] ){
			this.clientList[ key ] = [];
		}
		this.clientList[ key ].push( fn ); // 订阅的消息添加进缓存列表
	},
	trigger: function(){
		var key = Array.prototype.shift.call( arguments ), // (1);
		fns = this.clientList[ key ];
		if ( !fns || fns.length === 0 ){ // 如果没有绑定对应的消息
			return false;
		}
		for( var i = 0, fn; fn = fns[ i++ ]; ){
			fn.apply( this, arguments ); // (2) // arguments 是 trigger 时带上的参数
		}
	},
	remove: function( key, fn ){
		var fns = this.clientList[ key ];
		if ( !fns ){ // 如果 key 对应的消息没有被人订阅，则直接返回
			return false;
		}
		if ( !fn ){ // 如果没有传入具体的回调函数，表示需要取消 key 对应消息的所有订阅
			fns && ( fns.length = 0 );
		}else{
			for ( var l = fns.length - 1; l >=0; l-- ){ // 反向遍历订阅的回调函数列表
				var _fn = fns[ l ];
				if ( _fn === fn ){
					fns.splice( l, 1 ); // 删除订阅者的回调函数
				}
			}
		}
	};
};
var installEvent = function( obj ){
	for ( var i in event ){
		obj[ i ] = event[ i ];
	}
};

//使用
var salesOffices = {};
installEvent( salesOffices );
salesOffices.listen( 'squareMeter88', fn1 = function( price ){ // 小明订阅消息
	console.log( '价格= ' + price );
});
salesOffices.listen( 'squareMeter88', fn2 = function( price ){ // 小红订阅消息
	console.log( '价格= ' + price );
});
salesOffices.remove( 'squareMeter88', fn1 ); // 删除小明的订阅
salesOffices.trigger( 'squareMeter88', 2000000 ); // 输出：2000000
{% endcodeblock %}

#### 真实的例子——网站登录
{% codeblock %}
$.ajax( 'http:// xxx.com?login', function(data){ // 登录成功
	login.trigger( 'loginSucc', data); // 发布登录成功的消息
});

//各模块监听登录成功的消息：
var header = (function(){ // header 模块
	login.listen( 'loginSucc', function( data){
		header.setAvatar( data.avatar );
	});
	return {
		setAvatar: function( data ){
			console.log( '设置 header 模块的头像' );
		}
	}
})();
var nav = (function(){ // nav 模块
	login.listen( 'loginSucc', function( data ){
		nav.setAvatar( data.avatar );
	});
	return {
		setAvatar: function( avatar ){
			console.log( '设置 nav 模块的头像' );
		}
	}
})();

{% endcodeblock %}

### 命令模式
用闭包实现对的命令模式
{% codeblock %}
var setCommand = function(button, command){
	button.onclick = function(){
		command.execute();
	}
};

var MenuBar = {
	refresh: function(){
		console.log('刷新界面');
	}
};

var RefreshMenuBarCommand = function(receiver){
	return {
		execute:function(){
			receiver.refresh();
		},
	}
};

var refreshMenuBarCommand = RefreshMenuBarCommand(MenuBar);

setCommand(button1, refreshMenuBarCommand);
{% endcodeblock %}

宏命令:宏命令是一组命令的集合，通过执行宏命令的方式，可以一次执行一批命令。宏命令是命令模式与组合模式的联用产物.
{% codeblock %}
var closeDoorCommand = {
	execute: function(){
		console.log( '关门' );
	}
};
var openPcCommand = {
	execute: function(){
		console.log( '开电脑' );
	}
};
var openQQCommand = {
	execute: function(){
		console.log( '登录 QQ' );
	}
};
var MacroCommand = function(){
	return {
		commandsList: [],
		add: function( command ){
			this.commandsList.push( command );
		},
		execute: function(){
			for ( var i = 0, command; command = this.commandsList[ i++ ]; ){
				command.execute();
			}
		}
	}
};
var macroCommand = MacroCommand();
macroCommand.add( closeDoorCommand );
macroCommand.add( openPcCommand );
macroCommand.add( openQQCommand );
macroCommand.execute();
{% endcodeblock %}

### 组合模式
组合模式就是用小的子对象来构建更大的对象，而这些小的子对象本身也许是由更小的“孙对象”构成的。
{% codeblock %}
var MacroCommand = function(){
	return {
		commandsList: [],
		add: function(command){
			this.commandsList.push(command);
		},
		execute: function(){
			for(var i=0,command; command=this.commandsList[i++];){
				command.execute();
			}
		}
	}
};

var openAcCommand = {
	execute: function(){
		console.log('打开空调')；
	}
}；

/*************家里的电视和音响是连接在一起的，所以可以用一个宏命令来组合打开电视和打开音响的命令*****************/
var openTvCommand = {
	execute:function(){
		console.log('打开空调');
	}
};
var openSoundCommand = {
	execute: function(){
		console.log('打开音响');
	}
};
var macroCommand1 = MacroCommand();
macroCommand1.add(openTvCommand);
macroCommand1.add(openSoundCommand);

/*******关门，打开电脑和登录qq的命令**********/
var closeDoorCommand = {
	execute: function(){
		console.log('关门')
	}
};
var openPcCommand = {
	execute: function(){
		console.log('开电脑');
	}
};
var openQQCommand = {
	execute: function(){
		console.log('登录qq')
	}
};
var macroCommand2 = MacroCommand();
macroCommand2.add(closeDoorCommand);
macroCommand2.add(openPcCommand);
macroCommand2.add(openQQCommand);

/*********现在把所有的命令组合成一个“超级命令”**********/
var macroCommand = MacroCommand();
macroCommand.add( openAcCommand );
macroCommand.add( macroCommand1 );
macroCommand.add( macroCommand2 );
/*********最后给遥控器绑定“超级命令”**********/
var setCommand = (function( command ){
	document.getElementById( 'button' ).onclick = function(){
		command.execute();
	}
})( macroCommand )

{% endcodeblock %}

组合模式的例子---扫描文件夹
{% codeblock %}
/****Folder***/
var Folder = function(name){
	this.name = name;
	this.parent = null; //增加 this.parent 属性
	this.files = [];
};
Folder.prototype.add = function(file){
	file.parent = this; //设置父对象
	this.files.push(file);
};
Folder.prototype.scan = function(){
	console.log('开始扫描文件夹：'+this.name);
	for(var i=0,file,files=this.files; file=files[i++];){
		file.scan();
	}
}
Folder.prototype.remove = function(){
	if ( !this.parent ){ //根节点或者树外的游离节点
		return;
	}
	for ( var files = this.parent.files, l = files.length - 1; l >=0; l-- ){
		var file = files[ l ];
		if ( file === this ){
			files.splice( l, 1 );
		}
	}
};
/******File*****/
var File = function(name){
	this.name = name;
	this.parent = null;
};
File.prototype.add = function(){
	throw new Error('文件下面不能再添加文件');
};
File.prototype.scan = function(){
	console.log('开始扫描文件:'+this.name);
}
File.prototype.remove = function(){
	if ( !this.parent ){ //根节点或者树外的游离节点
		return;
	}
	for ( var files = this.parent.files, l = files.length - 1; l >=0; l-- ){
		var file = files[ l ];
		if ( file === this ){
			files.splice( l, 1 );
		}
	}
};


var folder = new Folder('学习资料');
var folder1 = new Folder('JavaScript');
var folder2 = new Folder('jQuery');

var file1 = new File('JavaScript设计模式与开发实战');
var file2 = new File('精通jQuery');
var file3 = new File('重构与模式');

//现有文件目录结构
folder1.add(file1);
folder2.add(file2);
folder.add(folder1);
folder.add(folder2);
folder.add(file3);

//需要增加的新的数据
var folder3 = new Folder( 'Nodejs' );
var file4 = new File( '深入浅出 Node.js' );
folder3.add( file4 );
var file5 = new File( 'JavaScript 语言精髓与编程实践' );

//把这些文件都添加到原有的树中
folder.add( folder3 );
folder.add( file5 );

//移除文件夹
folder1.remove();

//扫描整个文件夹
folder.scan();
{% endcodeblock %}

### 模板方法模式
模板方法模式是一种典型的通过封装变化提高系统扩展性的设计模式。但在JavaScript中，我们很多时候都不需要依样画瓢地去实现一个模版方法模式，高阶函数
是更好的选择。
{% codeblock %}
var Beverage = function(){};
Beverage.prototype.boilWater = function(){
	console.log( '把水煮沸' );
};
Beverage.prototype.brew = function(){
	throw new Error( '子类必须重写 brew 方法' );
};
Beverage.prototype.pourInCup = function(){
	throw new Error( '子类必须重写 pourInCup 方法' );
};
Beverage.prototype.addCondiments = function(){
	throw new Error( '子类必须重写 addCondiments 方法' );
};
Beverage.prototype.customerWantsCondiments = function(){
	return true; // 默认需要调料
};
Beverage.prototype.init = function(){
	this.boilWater();
	this.brew();
	this.pourInCup();
	if ( this.customerWantsCondiments() ){ // 如果挂钩返回 true，则需要调料
		this.addCondiments();
	}
};
var CoffeeWithHook = function(){};
CoffeeWithHook.prototype = new Beverage();
CoffeeWithHook.prototype.brew = function(){
	console.log( '用沸水冲泡咖啡' );
};
CoffeeWithHook.prototype.pourInCup = function(){
	console.log( '把咖啡倒进杯子' );
};
CoffeeWithHook.prototype.addCondiments = function(){
	console.log( '加糖和牛奶' );
};
CoffeeWithHook.prototype.customerWantsCondiments = function(){
	return window.confirm( '请问需要调料吗？' );
};
var coffeeWithHook = new CoffeeWithHook();
coffeeWithHook.init();
{% endcodeblock %}

### 享元模式
是一种用于性能优化的模式,核心是运用共享技术来有效支持大量细粒度的对象。如果系统中因为创建了大量类似的对象而导致内存占用过高，享元模式就非常有用了。享元模式要求将对象的属性划分为内部状态与外部
状态（状态在这里通常指属性）。
{% codeblock %}
1.内部状态储存于对象内部。
2.内部状态可以被一些对象共享。
3.内部状态独立于具体的场景，通常不会改变。
4.外部状态取决于具体的场景，并根据场景而变化，外部状态不能被共享。
{% endcodeblock %}
对象池维护一个装载空闲对象的池子，如果需要对象的时候，不是直接 new，而是转从对象池里获取。如果对象池里没有空闲对象，则创建一个新的对象，当获取出的对象完成它的职责之后， 再进入池子等待被下次获取。

### 职责链模式
一系列可能会处理请求的对象被连接成一条链，请求在这些对象之间依次传递，直到遇到一个可以处理它的对象，
{% codeblock %}
var order500 = function(orderType, pay, stock){
	if(orderType===1 && pay===true){
		console.log('500元定金预购，得到100优惠券');
	}else{
		return 'nextSuccessor';  //我不知道下一个节点是谁，反正把请求往后传递
	}
};
var order200 = function(orderType, pay, stock){
	if(orderType===2 && pay===true){
		console.log('200元定金预购，得到100优惠券');
	}else{
		return 'nextSuccessor';  //我不知道下一个节点是谁，反正把请求往后传递
	}
};
var orderNormal = function(orderType, pay, stock){
	if(orderType===2 && pay===true){
		console.log('普通购买，无优惠券');
	}else{
		console.log('手机库存不足');
	}
};

//Chain.prototype.setNextSuccessor 指定在链中的下一个节点
//Chain.prototype.passRequest  传递请求给某个节点
var Chain = function(fn){
	this.fn = fn;
	this.successor = null;
};
Chain.prototype.setNextSuccessor = function(successor){
	return this.successor = successor;
};
Chain.prototype.passRequest = function(){
	var ret = this.fn.apply(this, arguments);
	if(ret === 'nextSuccessor'){
		return this.successor && this.successor.passRequest.apply(this.successor, arguments);
	}
	return ret;
};

var chainOrder500 = new Chain(order500);
var chainOrder200 = new Chain(order200);
var chainOrderNormal = new Chain(orderNormal);
chainOrder500.setNextSuccessor( chainOrder200 );
chainOrder200.setNextSuccessor( chainOrderNormal );
chainOrder500.passRequest( 1, true, 500 ); // 输出：500 元定金预购，得到 100 优惠券
chainOrder500.passRequest( 2, true, 500 ); // 输出：200 元定金预购，得到 50 优惠券
chainOrder500.passRequest( 3, true, 500 ); // 输出：普通购买，无优惠券
chainOrder500.passRequest( 1, false, 0 ); // 输出：手机库存不足
{% endcodeblock %}
异步职责链.异步的职责链加上命令模式（把 ajax请求封装成命令对象，详情请参考第 9章），我们可以很方便地创建一个异步 ajax队列库。
{% codeblock %}
Chain.prototype.next= function(){
	return this.successor && this.successor.passRequest.apply( this.successor, arguments );
};

//使用
var fn1 = new Chain(function(){
	console.log( 1 );
	return 'nextSuccessor';
});
var fn2 = new Chain(function(){
	console.log( 2 );
	var self = this;
	setTimeout(function(){
		self.next();
	}, 1000 );
});
var fn3 = new Chain(function(){
	console.log( 3 );
});
fn1.setNextSuccessor( fn2 ).setNextSuccessor( fn3 );
fn1.passRequest();
{% endcodeblock %}
AOP实现.用 AOP 来实现职责链既简单又巧妙，但这种把函数叠在一起的方式，同时也叠加了函数的作用域，如果链条太长的话，也会对性能有较大的影响。
{% codeblock %}
Function.prototype.after = function( fn ){
	var self = this;
	return function(){
		var ret = self.apply( this, arguments );
		if ( ret === 'nextSuccessor' ){
			return fn.apply( this, arguments );
		}
		return ret;
	}
};
var order = order500.after( order200 ).after( orderNormal );
order( 1, true, 500 ); // 输出：500 元定金预购，得到 100 优惠券
order( 2, true, 500 ); // 输出：200 元定金预购，得到 50 优惠券
order( 1, false, 500 ); // 输出：普通购买，无优惠券
{% endcodeblock %}

### 中介者模式
{% codeblock %}
function Player(name, teamColor){
	this.name = name; //角色名字
	this.teamColor = teamColor; //队伍颜色
	this.state = 'alive';  //玩家生存状态
};
Player.prototype.win = function(){
	console.log(this.name+' won');
};
Player.prototype.lose = function(){
	console.log(this.name+' lost');
};
/*******玩家死亡*********/
Player.prototype.die = function(){
	this.state = 'dead';
	playerDirector.receiveMessage('playerDead', this); //给中介者发送信息，玩家死亡
};
/*******移除玩家******/
Player.prototype.remove = function(){
	playerDirector.receiveMessage('removePlayer', this);  //给中介者发送信息，移除一个玩家
};
/*******玩家换队********/
Player.prototype.changeTeam = function(color){
	playerDirector.receiveMessage('changeTeam', this, color);  //给中介者发消息，玩家换队
};

var playerFactory = function(name, teamColor){
	var newPlayer = new Player(name, teamColor);  //创建一个新的玩家对象
	playerDirector.receiveMessage('addPlayer', newPlayer);  //给中介者发送消息，新增玩家
	return newPlayer;
};

var playerDirector = (function(){
	var players = {},  //保存所有玩家
		operations = {}; //中介者可以执行的操作
	/*****新增一个玩家*****/
	operations.addPlayer = function(player){
		var teamColor = player.teamColor;  //玩家的队伍颜色
		players[teamColor] = players[teamColor] || [];  //如果该颜色的玩家还没有成立队伍，则新成立一个队伍
		players[teamColor].push(player);  //添加玩家进队伍
	};
	/***移除一个玩家****/
	operations.removePlayer = function(player){
		var teamColor = player.teamColor,   //玩家的队伍颜色
			teamPlayers = players[teamColor] || []; //该队伍所有成员
		for(var i=teamPlayers.length-1; i>=0; i--){  //遍历删除
			if(teamPlayers[i] === player){
				teamPlayers.splice(i,1);
			}
		}
	};
	/****玩家换队*****/
	operations.changeTeam = function(player, newTeamColor){  //玩家换队
		operations.removePlayer(player); //从原队伍中删除
		player.teamColor = newTeamColor;  //改变队伍颜色
		operations.addPlayer(player);   //增加到新队伍中
	};
	operations.playerDead = function(player){  //玩家死亡
		var teamColor = player.teamColor,
			teamPlayers = players[teamColor];  //玩家所在队伍
		var all_dead = true;
		for(var i=0,player; player=teamPlayers[i++];){
			if(player.state !=== 'dead'){
				all_dead = false;
				break;
			}
		}
		if(all_dead === true){  //全部死亡
			for(var i=0,player; player=teamPlayers[i++];){
				player.lose();  //本队所有玩家lose
			}
			for(var color in players){
				if(color !=== teamColor){
					var teamPlayers = players[color]; //其他队伍玩家
					for(var i=0,player; player=teamPlayers[i++];){
						player.win();  //其他队伍所有玩家win
					}
				}
			}
		}
	};
	var receiveMessage = function(){
		var message = Array.prototype.shift.call(arguments);  //arguments的第一个参数为消息名称
		operations[message].apply(this, arguments);
	};
	return {
		receiveMessage: receiveMessage
	}
})()


// 红队：
var player1 = playerFactory( '皮蛋', 'red' ),
player2 = playerFactory( '小乖', 'red' ),
player3 = playerFactory( '宝宝', 'red' ),
player4 = playerFactory( '小强', 'red' );
// 蓝队：
var player5 = playerFactory( '黑妞', 'blue' ),
player6 = playerFactory( '葱头', 'blue' ),
player7 = playerFactory( '胖墩', 'blue' ),
player8 = playerFactory( '海盗', 'blue' );
player1.die();
player2.die();
player3.die();
player4.die();
{% endcodeblock %}
选手机
{% codeblock %}
var goods = { // 手机库存
	"red|32G": 3,
	"red|16G": 0,
	"blue|32G": 1,
	"blue|16G": 6
};
var mediator = (function(){
	var colorSelect = document.getElementById( 'colorSelect' ),
	memorySelect = document.getElementById( 'memorySelect' ),
	numberInput = document.getElementById( 'numberInput' ),
	colorInfo = document.getElementById( 'colorInfo' ),
	memoryInfo = document.getElementById( 'memoryInfo' ),
	numberInfo = document.getElementById( 'numberInfo' ),
	nextBtn = document.getElementById( 'nextBtn' );
	return {
		changed: function( obj ){
			var color = colorSelect.value, // 颜色
			memory = memorySelect.value,// 内存
			number = numberInput.value, // 数量
			stock = goods[ color + '|' + memory ]; // 颜色和内存对应的手机库存数量
			if ( obj === colorSelect ){ // 如果改变的是选择颜色下拉框
				colorInfo.innerHTML = color;
			}else if ( obj === memorySelect ){
				memoryInfo.innerHTML = memory;
			}else if ( obj === numberInput ){
				numberInfo.innerHTML = number;
			}
			if ( !color ){
				nextBtn.disabled = true;
				nextBtn.innerHTML = '请选择手机颜色';
				return;
			}
			if ( !memory ){
				nextBtn.disabled = true;
				nextBtn.innerHTML = '请选择内存大小';
				return;
			}
			if ( ( ( number - 0 ) | 0 ) !== number - 0 ){ // 输入购买数量是否为正整数
				nextBtn.disabled = true;
				nextBtn.innerHTML = '请输入正确的购买数量';
				return;
			}
			nextBtn.disabled = false;
			nextBtn.innerHTML = '放入购物车';
		}

	}
})();
// 事件函数：
colorSelect.onchange = function(){
	mediator.changed( this );
};
memorySelect.onchange = function(){
	mediator.changed( this );
};
numberInput.oninput = function(){
	mediator.changed( this );
};
{% endcodeblock %}

### 装饰着模式
装饰者模式能够在不改变对象自身的基础上，在程序运行期间给对象动态地添加职责。跟继承相比，装饰者是一种更轻便灵活的做法，这是一种“即用即付”的方式。
用AOP装饰函数
{% codeblock %}
Function.prototype.before = function( beforefn ){
	var __self = this; // 保存原函数的引用
	return function(){ // 返回包含了原函数和新函数的"代理"函数
	beforefn.apply( this, arguments ); // 执行新函数，且保证 this 不被劫持，新函数接受的参数
	// 也会被原封不动地传入原函数，新函数在原函数之前执行
	return __self.apply( this, arguments ); // 执行原函数并返回原函数的执行结果，
	// 并且保证 this 不被劫持
	}
}
Function.prototype.after = function( afterfn ){
	var __self = this;
	return function(){
		var ret = __self.apply( this, arguments );
		afterfn.apply( this, arguments );
		return ret;
	}
};

document.getElementById = document.getElementById.before(function(){
	alert (1);
});
var button = document.getElementById( 'button' );
console.log( button );

window.onload = function(){
	alert (1);
}
window.onload = ( window.onload || function(){} ).after(function(){
	alert (2);
}).after(function(){
	alert (3);
}).after(function(){
	alert (4);
});
{% endcodeblock %}

#### 数据统计上报
{% codeblock %}
Function.prototype.after = function( afterfn ){
	var __self = this;
	return function(){
		var ret = __self.apply( this, arguments );
		afterfn.apply( this, arguments );
		return ret;
	}
};
var showLogin = function(){
	console.log( '打开登录浮层' );
}
var log = function(){
	console.log( '上报标签为: ' + this.getAttribute( 'tag' ) );
}
showLogin = showLogin.after( log ); // 打开登录浮层之后上报数据
document.getElementById( 'button' ).onclick = showLogin;
{% endcodeblock %}
插件式的表单验证
{% codeblock %}
Function.prototype.before = function( beforefn ){
	var __self = this;
	return function(){
		if ( beforefn.apply( this, arguments ) === false ){
			// beforefn 返回 false 的情况直接 return，不再执行后面的原函数
			return;
		}
		return __self.apply( this, arguments );
	}
}
var validata = function(){
	if ( username.value === '' ){
		alert ( '用户名不能为空' );
		return false;
	}
	if ( password.value === '' ){
		alert ( '密码不能为空' );
		return false;
	}
}
var formSubmit = function(){
	var param = {
		username: username.value,
		password: password.value
	}
	ajax( 'http:// xxx.com/login', param );
}
formSubmit = formSubmit.before( validata );
submitBtn.onclick = function(){
	formSubmit();
}
{% endcodeblock %}

### 状态模式
允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。将状态封装成独立的类，并将请求委托给当前的状态对象，当对象的内部状态改变时，会带来不同的行为变化。
{% codeblock %}
var Light = function(){
	this.offLightState = new OffLightState( this ); // 持有状态对象的引用
	this.weakLightState = new WeakLightState( this );
	this.strongLightState = new StrongLightState( this );
	this.superStrongLightState = new SuperStrongLightState( this );
	this.button = null;
};
Light.prototype.init = function(){
	var button = document.createElement( 'button' ),
	self = this;
	this.button = document.body.appendChild( button );
	this.button.innerHTML = '开关';
	this.currState = this.offLightState; // 设置默认初始状态
	this.button.onclick = function(){ // 定义用户的请求动作
		self.currState.buttonWasPressed();
	}
};
var OffLightState = function( light ){
	this.light = light;
};
OffLightState.prototype.buttonWasPressed = function(){
	console.log( '弱光' );
	this.light.setState( this.light.weakLightState );
};
...
{% endcodeblock %}

JavaScript 版本的状态机
{% codeblock %}
var Light = function(){
	this.currState = FSM.off; // 设置当前状态
	this.button = null;
};
Light.prototype.init = function(){
	var button = document.createElement( 'button' ),
	self = this;
	button.innerHTML = '已关灯';
	this.button = document.body.appendChild( button );
	this.button.onclick = function(){
		self.currState.buttonWasPressed.call( self ); // 把请求委托给 FSM 状态机
	}
};
var FSM = {
	off: {
		buttonWasPressed: function(){
			console.log( '关灯' );
			this.button.innerHTML = '下一次按我是开灯';
			this.currState = FSM.on;
		}
	},
	on: {
		buttonWasPressed: function(){
			console.log( '开灯' );
			this.button.innerHTML = '下一次按我是关灯';
			this.currState = FSM.off;
		}
	}
};
var light = new Light();
light.init();

var delegate = function( client, delegation ){
	return {
		buttonWasPressed: function(){ // 将客户的操作委托给 delegation 对象
			return delegation.buttonWasPressed.apply( client, arguments );
		}
	}
};
var FSM = {
	off: {
		buttonWasPressed: function(){
			console.log( '关灯' );
			this.button.innerHTML = '下一次按我是开灯';
			this.currState = this.onState;
		}
	},
	on: {
		buttonWasPressed: function(){
			console.log( '开灯' );
			this.button.innerHTML = '下一次按我是关灯';
			this.currState = this.offState;
		}
	}
};
var Light = function(){
	this.offState = delegate( this, FSM.off );
	this.onState = delegate( this, FSM.on );
	this.currState = this.offState; // 设置初始状态为关闭状态
	this.button = null;
};
Light.prototype.init = function(){
	var button = document.createElement( 'button' ),
	self = this;
	button.innerHTML = '已关灯';
	this.button = document.body.appendChild( button );
	this.button.onclick = function(){
		self.currState.buttonWasPressed();
	}
};
var light = new Light();
light.init();
{% endcodeblock %}

### 适配器模式(包装器)
适配器模式的作用是解决两个软件实体间的接口不兼容的问题。使用适配器模式之后，原本由于接口不兼容而不能工作的两个软件实体可以一起工作。
