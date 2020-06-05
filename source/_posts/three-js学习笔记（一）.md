---
title: three.js学习笔记（一）
date: 2016-12-02 16:27:03
categories: '学习笔记'
---

{% blockquote %}
[three.js在线中文文档](http://techbrood.com/threejs/docs/)
{% endblockquote %}
一个three.js程序至少包括Renderer（渲染器），Scene（场景），Camera（照相机），场景中的物体。

<!--more-->
## Renderer渲染器
WebGL的渲染需要HTML5Canvas元素，可以手动在HTML中定义或者用three生成。three使用的是右手坐标系（即Z轴向外）。

在HTML中定义canvas元素时，设置渲染器：
{% codeblock %}
var renderer = new THREE.WebGLRenderer({
    canvas: document.getElementById('mainCanvas')
});
{% endcodeblock %}

three生成canvas元素:
{% codeblock %}
var renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth,window.innerHeight);                     //设置canvas元素宽高
document.getElementsByTagName('body')[0].appendChild(renderer.domElement);  //将canvas元素添加到body中
{% endcodeblock %}

设置背景色（用于清除画面的颜色）
{% codeblock %}
renderer.setClearColor(0x000000);
{% endcodeblock %}

如果你的应用运行在一个低分辨率的情况下，但是你想保持程序视窗的大小，你可以在调用setSize时把updateStyle设置为false
{% codeblock %}
renderer.setSize(window.innerWidth/2, window.innerHeight/2, false);
{% endcodeblock %}

## Scene场景
在threejs中添加的物体都是添加到场景中的，因此他相当于一个大容器。一般说，场景里没有很复杂的操作，在程序最开始的时候进行实例化，然后将物体添加到场景中即可。
{% codeblock %}
var scene = new THREE.Scene();
{% endcodeblock %}

## Camera照相机

### 正交投影照相机

{% codeblock %}
THREE.OrthographicCamera(left, right, top, bottom, near, far);
{% endcodeblock %}
这六个参数分别代表正交投影照相机拍摄到的空间的六个面的位置，这六个面围成一个长方体，我们称其为视景体。只有在视景体内部的物体才可能显示在屏幕上，而视景体外的物体会在显示之前被裁减掉。

为了保持照相机的横竖比例，需要保证(right - left)与(top - bottom)的比例与Canvas宽度与高度的比例一致。

near与far都是指到照相机位置在深度平面的位置，而照相机不应该拍摄到其后方的物体，因此这两个值应该均为正值。为了保证场景中的物体不会因为太近或太远而被照相机忽略，一般near的值设置得较小，far的值设置得较大，具体值视场景中物体的位置等决定。

{% codeblock %}
camera.lookAt(new THREE.Vector3(0, 0, 0));
{% endcodeblock %}
照相机默认是沿z轴负方向观察的，我们可以通过lookAt函数指定它看着原点方向。

### 透视投影照相机

{% codeblock %}
THREE.PerspectiveCamera(fov, aspect, near, far);
{% endcodeblock %}
视景体，是可能被渲染的物体所在的区域。fov是视景体竖直方向上的张角（是角度制而非弧度制）。aspect等于width / height，是照相机水平方向和竖直方向长度的比值，通常设为Canvas的横纵比例。near和far分别是照相机到视景体最近、最远的距离，均为正值，且far应大于near。

## 几何物体

### 立方体

{% codeblock %}
THREE.CubeGeometry(width, height, depth, widthSegments, heightSegments, depthSegments);
{% endcodeblock %}
width是x方向上的长度；height是y方向上的长度；depth是z方向上的长度；后三个参数分别是在三个方向上的分段数，如widthSegments为3的话，代表x方向上水平分为三份。一般情况下不需要分段的话，可以不设置后三个参数，后三个参数的缺省值为1。其他几何形状中的分段也是类似的，下面不做说明。

### 平面

{% codeblock %}
THREE.PlaneGeometry(width, height, widthSegments, heightSegments);
{% endcodeblock %}
这里的平面其实是一个长方形，而不是数学意义上无限大小的平面。其中，width是x方向上的长度；height是y方向上的长度；后两个参数同样表示分段。如果需要创建的平面在x轴和z轴所在的平面内，可以通过物体的旋转来实现。

### 球体

{% codeblock %}
THREE.SphereGeometry(radius, segmentsWidth, segmentsHeight, phiStart, phiLength, thetaStart, thetaLength);
{% endcodeblock %}
radius是半径；segmentsWidth表示经度上的切片数；segmentsHeight表示纬度上的切片数；phiStart表示经度开始的弧度；phiLength表示经度跨过的弧度；thetaStart表示纬度开始的弧度；thetaLength表示纬度跨过的弧度。

### 圆形

{% codeblock %}
THREE.CircleGeometry(radius, segments, thetaStart, thetaLength);
{% endcodeblock %}
在x轴和y轴所在平面创建圆形或者扇形。

### 圆柱体

{% codeblock %}
THREE.CylinderGeometry(radiusTop, radiusBottom, height, radiusSegments, heightSegments, openEnded);
{% endcodeblock %}
radiusTop与radiusBottom分别是顶面和底面的半径，由此可知，当这两个参数设置为不同的值时，实际上创建的是一个圆台；height是圆柱体的高度；radiusSegments与heightSegments可类比球体中的分段；openEnded是一个布尔值，表示是否没有顶面和底面，缺省值为false，表示有顶面和底面。

### 正四面体、正八面体、正十二面体

{% codeblock %}
THREE.TetrahedronGeometry(radius, detail);
THREE.OctahedronGeometry(radius, detail);
THREE.IcosahedronGeometry(radius, detail);
{% endcodeblock %}
radius是半径；detail是细节层次（Level of Detail）的层数，对于大面片数模型，可以控制在视角靠近物体时，显示面片数多的精细模型，而在离物体较远时，显示面片数较少的粗略模型。这里我们不对detail多作展开，一般可以对这个值缺省。

### 圆环面

{% codeblock %}
THREE.TorusGeometry(radius, tube, radialSegments, tubularSegments, arc);
{% endcodeblock %}
radius是圆环半径；tube是管道半径；radialSegments与tubularSegments分别是两个分段数，详见上图；arc是圆环面的弧度，缺省值为Math.PI * 2。

### 圆环结

{% codeblock %}
THREE.TorusKnotGeometry(radius, tube, radialSegments, tubularSegments, p, q, heightScale);
{% endcodeblock %}
前四个参数在圆环面中已经有所介绍，p和q是控制其样式的参数，一般可以缺省，如果需要详细了解，请学习圆环结的相关知识；heightScale是在z轴方向上的缩放。

### 自定义形状

{% codeblock THREE.Geometry()%}
// 初始化几何形状
var geometry = new THREE.Geometry();

// 设置顶点位置
// 顶部4顶点
geometry.vertices.push(new THREE.Vector3(-1, 2, -1));
geometry.vertices.push(new THREE.Vector3(1, 2, -1));
geometry.vertices.push(new THREE.Vector3(1, 2, 1));
geometry.vertices.push(new THREE.Vector3(-1, 2, 1));
// 底部4顶点
geometry.vertices.push(new THREE.Vector3(-2, 0, -2));
geometry.vertices.push(new THREE.Vector3(2, 0, -2));
geometry.vertices.push(new THREE.Vector3(2, 0, 2));
geometry.vertices.push(new THREE.Vector3(-2, 0, 2));

// 设置顶点连接情况
// 顶面
geometry.faces.push(new THREE.Face3(0, 1, 3));
geometry.faces.push(new THREE.Face3(1, 2, 3));
// 底面
geometry.faces.push(new THREE.Face3(4, 5, 6));
geometry.faces.push(new THREE.Face3(5, 6, 7));
// 四个侧面
geometry.faces.push(new THREE.Face3(1, 5, 6));
geometry.faces.push(new THREE.Face3(6, 2, 1));
geometry.faces.push(new THREE.Face3(2, 6, 7));
geometry.faces.push(new THREE.Face3(7, 3, 2));
geometry.faces.push(new THREE.Face3(3, 7, 0));
geometry.faces.push(new THREE.Face3(7, 4, 0));
geometry.faces.push(new THREE.Face3(0, 4, 5));
geometry.faces.push(new THREE.Face3(0, 5, 1));
{% endcodeblock %}

### 画线

{% codeblock %}
LineBasicMaterial( parameters );
{% endcodeblock %}
Parameters是一个定义材质外观的对象，它包含多个属性来定义材质，这些属性是：
Color：线条的颜色，用16进制来表示，默认的颜色是白色。
Linewidth：线条的宽度，默认时候1个单位宽度。
Linecap：线条两端的外观，默认是圆角端点，当线条较粗的时候才看得出效果，如果线条很细，那么你几乎看不出效果了。
Linejoin：两个线条的连接点处的外观，默认是“round”，表示圆角。
VertexColors：定义线条材质是否使用顶点颜色，这是一个boolean值。意思是，线条各部分的颜色会根据顶点的颜色来进行插值。
Fog：定义材质的颜色是否受全局雾效的影响。
{% codeblock %}
//画线
var geometry = new THREE.Geometry();                                                 //声明几何体
var material = new THREE.LineBasicMaterial( { vertexColors: THREE.VertexColors} );   //定义线条材质
var color1 = new THREE.Color( 0x444444 ), color2 = new THREE.Color( 0xFF0000 );

// 线的材质可以由2点的颜色决定
var p1 = new THREE.Vector3( -100, 0, 100 );
var p2 = new THREE.Vector3(  100, 0, -100 );
geometry.vertices.push(p1);
geometry.vertices.push(p2);
geometry.colors.push( color1, color2 );

var line = new THREE.Line( geometry, material, THREE.LinePieces );                  //定义线条，使用THREE.Line类
scene.add(line);
{% endcodeblock %}