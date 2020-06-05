---
title: three.js学习笔记（二）
date: 2016-12-02 19:11:21
categories: '学习笔记'
---

{% blockquote %}
[three.js在线中文文档](http://techbrood.com/threejs/docs/)
{% endblockquote %}
## 文字形状

可以用来创建三维的文字形状。使用文字形状需要下载和引用额外的字体库。
<!--more-->
### 加载文字

{% codeblock %}
var loader = new THREE.FontLoader();
loader.load('../lib/helvetiker_regular.typeface.json', function(font) {
    var mesh = new THREE.Mesh(new THREE.TextGeometry('Hello', {
        font: font,
        size: 1,
        height: 1
    }), material);
    scene.add(mesh);

    // render
    renderer.render(scene, camera);
});
{% endcodeblock %}

### 使用

{% codeblock %}
THREE.TextGeometry(text, parameters);
{% endcodeblock %}
text是文字字符串，parameters是以下参数组成的对象：
size：字号大小，一般为大写字母的高度
height：文字的厚度
curveSegments：弧线分段数，使得文字的曲线更加光滑
font：字体，默认是'helvetiker'，需对应引用的字体文件
weight：值为'normal'或'bold'，表示是否加粗
style：值为'normal'或'italics'，表示是否斜体
bevelEnabled：布尔值，是否使用倒角，意为在边缘处斜切
bevelThickness：倒角厚度
bevelSize：倒角宽度
## 材质

材质是独立于物体顶点信息之外的与渲染效果相关的属性。通过设置材质可以改变物体的颜色、纹理贴图、光照模式等。

### 基本材质

使用基本材质的物体，渲染后物体的颜色始终为该材质的颜色，而不会由于光照产生明暗、阴影效果。如果没有指定材质的颜色，则颜色是随机的。其构造函数是：
{% codeblock %}
THREE.MeshBasicMaterial(opt);
{% endcodeblock %}
opt可以缺省，或者为包含各属性的值。
visible：是否可见，默认为true
side：渲染面片正面或是反面，默认为正面THREE.FrontSide，可设置为反面THREE.BackSide，或双面THREE.DoubleSide
wireframe：是否渲染线而非面，默认为false
color：十六进制RGB颜色，如红色表示为0xff0000
map：使用纹理贴图

### Lambert材质

只考虑漫反射而不考虑镜面反射的效果，因而对于金属、镜子等需要镜面反射效果的物体就不适应，对于其他大部分物体的漫反射效果都是适用的。
{% codeblock %}
THREE.MeshLambertMaterial(opt);
{% endcodeblock %}
color：是用来表现材质对散射光的反射能力，也是最常用来设置材质颜色的属性
ambient：表示对环境光的反射能力，只有当设置了AmbientLight后，该值才是有效的，材质对环境光的反射能力与环境光强相乘后得到材质实际表现的颜色。
emissive：是材质的自发光颜色，可以用来表现光源的颜色。

### Phong材质

考虑了镜面反射的效果，因此对于金属、镜面的表现尤为适合
{% codeblock %}
THREE.MeshPhongMaterial(opt);
{% endcodeblock %}
color：是用来表现材质对散射光的反射能力，也是最常用来设置材质颜色的属性
ambient：表示对环境光的反射能力，只有当设置了AmbientLight后，该值才是有效的，材质对环境光的反射能力与环境光强相乘后得到材质实际表现的颜色。
emissive：是材质的自发光颜色，可以用来表现光源的颜色。单独使用红色的自发光：
specular：指定镜面光
shininess：值越大时，高光的光斑越小，默认值为30

### 法向材质

{% codeblock %}
THREE.MeshNormalMaterial();
{% endcodeblock %}
可以将材质的颜色设置为其法向量的方向，有时候对于调试很有帮助。材质的颜色与照相机与该物体的角度相关。

### 材质的纹理贴图

导入图像作为纹理贴图，并添加到相应的材质中
{% codeblock 一个面%}
var texture = THREE.ImageUtils.loadTexture('../img/0.png', {}, function() {
    renderer.render(scene, camera);
});
var material = new THREE.MeshLambertMaterial({
    map: texture
});
{% endcodeblock %}
{% codeblock 六个面%}
var materials = [];
for (var i = 0; i < 6; ++i) {
    materials.push(new THREE.MeshBasicMaterial({
        map: THREE.ImageUtils.loadTexture('../img/' + i + '.png',
                {}, function() {
                    renderer.render(scene, camera);
                }),
        overdraw: true
    }));
}

var cube = new THREE.Mesh(new THREE.CubeGeometry(5, 5, 5),
        new THREE.MeshFaceMaterial(materials));
scene.add(cube);
{% endcodeblock %}
{% codeblock 棋盘格%}
var texture = THREE.ImageUtils.loadTexture('../img/chess.png', {}, function() {
    renderer.render(scene, camera);
});
texture.wrapS = texture.wrapT = THREE.RepeatWrapping;//重复方式为两个方向（wrapS和wrapT）都重复
texture.repeat.set(4, 4);//设置两个方向上都重复4次
{% endcodeblock %}

## 网格：网格是由顶点、边、面等组成的物体
{% codeblock 棋盘格%}
Mesh(geometry, material);
{% endcodeblock %}
如果不指定material，则每次会随机分配一种wireframe为true的材质，每次刷新页面后的颜色是不同的.

除了在构造函数中指定材质，在网格被创建后，也能对材质进行修改。
{% codeblock 棋盘格%}
var material = new THREE.MeshLambertMaterial({
    color: 0xffff00
});
var geometry = new THREE.CubeGeometry(1, 2, 3);
var mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);

mesh.material = new THREE.MeshLambertMaterial({
    color: 0xff0000
});
{% endcodeblock %}

THREE.Vector3有x、y、z三个属性，如果只设置其中一个属性，则可以用以下方法
{% codeblock %}
mesh.position.z = 1;
{% endcodeblock %}
如果需要同时设置多个属性，可以使用以下两种方法
{% codeblock %}
mesh.position.set(1.5, -0.5, 0);
mesh.position = new THREE.Vector3(1.5, -0.5, 0);
{% endcodeblock %}

## 动画

{% codeblock %}
<script type="text/javascript" src="stats.min.js"></script><!--实时的FPS信息，从而更好地监测动画效果。它占据屏幕中的一小块位置（如左上角）。单击后显示每帧渲染时间-->
var stat = null;
function init() {
    stat = new Stats();
    stat.domElement.style.position = 'absolute';
    stat.domElement.style.right = '0px';
    stat.domElement.style.top = '0px';
    document.body.appendChild(stat.domElement);

    // Three.js init ...
}
{% endcodeblock %}
然后，在上一节介绍的动画重绘函数draw中调用stat.begin();与stat.end();分别表示一帧的开始与结束：最终就能得到FPS效果了。requestAnimationFrame跟js一样。

## 光与影

### 环境光

环境光是指场景整体的光照效果，是由于场景内若干光源的多次反射形成的亮度一致的效果，通常用来为整个场景指定一个基础亮度。因此，环境光没有明确的光源位置，在各处形成的亮度也是一致的。

在设置环境光时，只需要指定光的颜色：THREE.AmbientLight(hex)其中，hex是十六进制的RGB颜色信息，如红色表示为0xff0000。
{% codeblock %}
var light = new THREE.AmbientLight(0xffffff);//如果想让环境光暗些，可以将其设置为new THREE.AmbientLight(0xcccccc)等
scene.add(light);
{% endcodeblock %}
环境光并不在乎物体材质的color属性，而是ambient属性。ambient属性的默认值是0xffffff。
不透明物体的颜色其实是其反射光的颜色，而ambient属性表示的是物体反射环境光的能力。对于0x00ff00的物体，红色通道是0，而环境光是完全的红光，因此该长方体不能反射任何光线，最终的渲染颜色就是黑色；而对于0xffffff的白色长方体，红色通道是0xff，因而能反射所有红光，渲染的颜色就是红色。
当环境光不是白色或灰色的时候，渲染的效果往往会很奇怪。因此，环境光通常使用白色或者灰色，作为整体光照的基础。

### 点光源

点光源是不计光源大小，可以看作一个点发出的光源。点光源照到不同物体表面的亮度是线性递减的，因此，离点光源距离越远的物体会显得越暗。
{% codeblock %}
THREE.PointLight(hex, intensity, distance);
{% endcodeblock %}
hex是光源十六进制的颜色值；intensity是亮度，缺省值为1，表示100%亮度；distance是光源最远照射到的距离，缺省值为0。
{% codeblock %}
var light = new THREE.PointLight(0xffffff, 2, 100);
light.position.set(0, 1.5, 2);
scene.add(light);
{% endcodeblock %}

### 平行光

对于任意平行的平面，平行光照射的亮度都是相同的，而与平面所在位置无关。
{% codeblock %}
THREE.DirectionalLight(hex, intensity);
{% endcodeblock %}
hex是光源十六进制的颜色值；intensity是亮度，缺省值为1，表示100%亮度。此外，对于平行光而言，设置光源位置尤为重要。

### 聚光灯

聚光灯是一种特殊的点光源，它能够朝着一个方向投射光线。聚光灯投射出的是类似圆锥形的光线。
{% codeblock %}
THREE.SpotLight(hex, intensity, distance, angle, exponent);
{% endcodeblock %}
相比点光源，多了angle和exponent两个参数。angle是聚光灯的张角，缺省值是Math.PI / 3，最大值是Math.PI / 2；exponent是光强在偏离target的衰减指数（target需要在之后定义，缺省值为(0, 0, 0)），缺省值是10。
在调用构造函数之后，除了设置光源本身的位置，一般还需要设置target：
{% codeblock %}
light.position.set(x1, y1, z1);light.target.position.set(x2, y2, z2);
{% endcodeblock %}
除了设置light.target.position的方法外，如果想让聚光灯跟着某一物体移动（就像真的聚光灯！），可以target指定为该物体：
{% codeblock %}
var cube = new THREE.Mesh(new THREE.CubeGeometry(1, 1, 1),
        new THREE.MeshLambertMaterial({color: 0x00ff00}));

var light = new THREE.SpotLight(0xffff00, 1, 100, Math.PI / 6, 25);
light.target = cube;
{% endcodeblock %}

### 阴影

在Three.js中，能形成阴影的光源只有THREE.DirectionalLight与THREE.SpotLight；而相对地，能表现阴影效果的材质只有THREE.LambertMaterial与THREE.PhongMaterial。

我们需要在初始化时，告诉渲染器渲染阴影：renderer.shadowMapEnabled = true;然后，对于光源以及所有要产生阴影的物体调用：xxx.castShadow = true;对于接收阴影的物体调用：xxx.receiveShadow = true;

通常还需要设置光源的阴影相关属性，才能正确显示出阴影效果。
对于聚光灯，需要设置shadowCameraNear、shadowCameraFar、shadowCameraFov三个值，类比我们在第二章学到的透视投影照相机，只有介于shadowCameraNear与shadowCameraFar之间的物体将产生阴影，shadowCameraFov表示张角。

对于平行光，需要设置shadowCameraNear、shadowCameraFar、shadowCameraLeft、shadowCameraRight、shadowCameraTop以及shadowCameraBottom六个值，相当于正交投影照相机的六个面。同样，只有在这六个面围成的长方体内的物体才会产生阴影效果。

为了看到阴影照相机的位置，通常可以在调试时开启light.shadowCameraVisible = true。

如果想要修改阴影的深浅，可以通过设置shadowDarkness，该值的范围是0到1，越小越浅。而如果想实现软阴影的效果，可以通过renderer.shadowMapSoft = true;方便地实现。

## 着色器
使用着色器可以更灵活地控制渲染效果，结合纹理，可以进行多次渲染，达到更强大的效果。着色器是屏幕上呈现画面之前的最后一步，用它可以对先前渲染的结果做修改，包括对颜色、位置等等信息的修改，甚至可以对先前渲染的结果做后处理，实现高级的渲染效果。