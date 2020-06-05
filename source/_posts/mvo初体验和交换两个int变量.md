---
title: mvo初体验和交换两个int变量
date: 2018-05-25 16:17:48
categories: 学习笔记
---

### 交换两个int变量
交换两个int变量的值(var a=10, b=20;)的方法
##### 利用一个临时变量去存储
{% codeblock %}
	int tmp=a;
	a=b;
	b=tmp;
{% endcodeblock %}
<!-- more -->
##### ES6解构赋值 
{% codeblock %}
	[a,b]=[b,a]
{% endcodeblock %}
##### 利用加减法得到：
{% codeblock %}
a=a-b;//计算ab的距离 
b=b+a;//b+距离自然等于a 
a=b-a;//a-距离自然等于b，这里的b已经等于原来的a值
{% endcodeblock %}
{% codeblock %}
a=a+b;//计算ab的和  
b=a-b;//ab的和减去b自然等于a  
a=a-b;//ab的和减去a自然等于b，这里的b已经等于原来的a
{% endcodeblock %}
##### 利用连续计算
{% codeblock %}
a=a+b-(b=a);
{% endcodeblock %}
##### 利用异或运算
  (1)以两个数为角度(按位异或)：如果a、b两个值不相同，则异或结果为1。如果a、b两个值相同，异或结果为0。
  (2)以‘用一个数b去异或一个数a’角度：
  (3)a^b=b^a   a^b^b=a  a^b^a=b
{% codeblock %}
a=a^b;
b=a^b;
a=a^b
{% endcodeblock %}


### mvo初体验
{% codeblock %}
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	<div id='button-box'>
		
	</div>

	<div id='img-box'>
		<img src="" />
	</div>

	<div>
		第<span id='timer'>0</span>次点击
	</div>

	<script type="text/javascript">
		//mov model view octopus
		//model跟view分离。通过octopus沟通。
		var model = {
			currentImg: null,
			data: [
				{src: '1.png', clickNum: 0, name:'第一张'},
				{src: '2.png', clickNum: 0, name:'第二张'},
				{src: '3.jpg', clickNum: 0, name:'第三张'},
				{src: '4.jpg', clickNum: 0, name:'第四张'}
			]
		}

		var octopus = {
			init:function(){
				viewList.init();
				viewImg.init();
			},
			getCurrentImg: function(){
				return model.currentImg?model.currentImg:(model.currentImg = model.data[0]);
			},
			getAllData:function(){
				return model.data;
			},
			setCurrentImg: function(obj){
				model.currentImg = obj;
				viewImg.render();
			},
			addClickNum: function(){
				++model.currentImg.clickNum;
				viewImg.render();
			}
		}

		var viewList = {
			init: function(){
				this.buttonBox = document.querySelector('#button-box');
				this.render();
			},
			render: function(){
				var allData = octopus.getAllData();
				var fragment = document.createDocumentFragment();
				for(var i=0;i<allData.length;i++){
					var element = document.createElement('button');
					element.appendChild(document.createTextNode(allData[i].name));
					fragment.appendChild(element);
					element.addEventListener('click', function(i){
						return function(){
							octopus.setCurrentImg(allData[i]);
						}
					}(i))
				}
				this.buttonBox.appendChild(fragment);
			}
		}

		var viewImg = {
			init: function(){
				document.querySelector('#img-box img').addEventListener('click', function(){
					octopus.addClickNum();
				})
				this.imgElem = document.querySelector('img');
				this.timer = document.querySelector('#timer');
				this.render();
			},
			render: function(){
				var currentImg = octopus.getCurrentImg();
				this.timer.innerHTML = currentImg.clickNum;
				this.imgElem.src = currentImg.src;
			}
		}

		octopus.init();
	</script>
</body>
</html>
{% endcodeblock %}