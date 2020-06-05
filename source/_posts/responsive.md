---
title: css响应相关
date: 2018-01-22 13:46:04
categories: css
---

css响应相关(移动端高清、多屏适配方案),只是学习笔记，还没有亲自验证。

<!-- more -->
### 媒体查询
1.
{% codeblock %}
@media screen and (max-width: :960px){
	body{
		background-color: red;
	}
}
{% endcodeblock %}
2.纵向放置显示屏：
{% codeblock %}
<link rel="stylesheet" type="text/css" media="screen and (orientation: portrait)" href="screen-styles.css" />
{% endcodeblock %}
非纵向放置显示屏：
{% codeblock %}
<link rel="stylesheet" type="text/css" media="not screen and (orientation: portrait)" href="screen-styles.css" />
{% endcodeblock %}
满足一个即可应用：
{% codeblock %}
<link rel="stylesheet" type="text/css" media="screen and (orientation: portrait), projection" href="screen-styles.css" />
{% endcodeblock %}
（projection为投影仪）
这样引入css文件也会下载，再根据media决定是否应用样式

3.
{% codeblock %}
@import url("phone.css") screen and (max-width:360px);
{% endcodeblock %}
使用css的@media方式会增加HTTP请求。

### 阻止移动浏览器自动调整页面大小
{% codeblock %}
<meta name="viewport" content="initial-scale=1,width=device-width" >
{% endcodeblock %}

### retina下，border: 1px问题
1.pt是一个物理长度单位，指的是72分之一英寸。
2.px是一个相对长度单位，是计算机系统的数字化图像长度单位，如果px要换算成物理长度，需要指定精度DPI(Dots Per Inch，每英寸像素数)，在扫描打印时一般都有DPI可选。Windows系统默认是96dpi，Apple系统默认是72dpi。在不同的屏幕上(普通屏幕 vs retina屏幕)，所呈现的大小(物理尺寸)是一致的。
3.DPI(Dots Per Inch)是指打印分辨率，针对于输出设备而言的。
4.PPI图像分辨率，所表示的是每英寸所拥有的像素数量。因此PPI数值越高，即代表显示屏能够以越高的密度显示图像。当然，显示的密度越高，拟真度就越高。
5.并不是所有手机浏览器都能识别border: 0.5px;，ios7以下，android等其他系统里，0.5px会被当成为0px处理.

### 位图像素
1.一个位图像素是栅格图像(如：png, jpg, gif等)最小的数据单元。每一个位图像素都包含着一些自身的显示信息(如：显示位置，颜色值，透明度等)。
2.理论上，1个位图像素对应于1个物理像素，图片才能得到完美清晰的展示。
3.在普通屏幕下是没有问题的，但是在retina屏幕下就会出现位图像素点不够，从而导致图片模糊的情况。对于图片高清问题，比较好的方案就是两倍图片(@2x)。
4.retina下，图片高清问题解决方案:两倍图片(@2x)，然后图片容器缩小50%。我们照常写border-bottom: 1px solid #ddd;，然后通过transform: scaleY(.5)缩小0.5倍来达到0.5px的效果。

### 多屏适配布局问题
DPR = 设备像素 / CSS像素(某一方向上)
针对不同手机屏幕尺寸和dpr动态的改变根节点html的font-size大小(基准值)。
这里我们提取了一个公式(rem表示基准值):rem = document.documentElement.clientWidth * dpr / 10
1.乘以dpr，是因为页面有可能为了实现1px border页面会缩放(scale) 1/dpr 倍(如果没有，dpr=1),。
2.除以10，是为了取整，方便计算(理论上可以是任何值)

{% codeblock %}
var dpr, rem, scale;
var docEl = document.documentElement;
var fontEl = document.createElement('style');
var metaEl = document.querySelector('meta[name="viewport"]');

dpr = window.devicePixelRatio || 1;
rem = docEl.clientWidth * dpr / 10;
scale = 1 / dpr;


// 设置viewport，进行缩放，达到高清效果
metaEl.setAttribute('content', 'width=' + dpr * docEl.clientWidth + ',initial-scale=' + scale + ',maximum-scale=' + scale + ', minimum-scale=' + scale + ',user-scalable=no');

// 设置data-dpr属性，留作的css hack之用
docEl.setAttribute('data-dpr', dpr);

// 动态写入样式
docEl.firstElementChild.appendChild(fontEl);
fontEl.innerHTML = 'html{font-size:' + rem + 'px!important;}';

// 给js调用的，某一dpr下rem和px之间的转换函数
window.rem2px = function(v) {
    v = parseFloat(v);
    return v * rem;
};
window.px2rem = function(v) {
    v = parseFloat(v);
    return v / rem;
};

window.dpr = dpr;
window.rem = rem;
{% endcodeblock %}