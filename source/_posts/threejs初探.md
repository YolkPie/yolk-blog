---
title: SameSiteForCookie
date: 2021-05-03 15:20:23
tags:
- cookie

author: YYY
---

## threejs初探

#### **Three.js， WebGL 与 OpenGL**

提到 Three.js，就必须说一下 OpenGL 和 WebGL。
OpenGL 大概许多人都有所听闻，它是最常用的跨平台图形处理开源库。
WebGL 就是基于 OpenGL 设计的面向 web 的 3D 图形标准，它提供了一系列 JavaScript API，通过这些 API 进行图形渲染，系统硬件会加速 3D 渲染，从而获得较高性能。
而 Three.js 是 JavaScript 编写的 WebGL 第三方库，通过对 WebGL 接口的封装与简化而形成的一个易用的图形库。

#### **WebGL 与 Three.js 对比**
通过上面的简介，我们知道 WebGL 和 Three.js 都可以进行 Web 端的 3D 图形开发。那问题来了，既然我们有了 WebGL，为什么还需要 Three.js？
这是因为前端工程师想要短时间上手 WebGL 还是挺有难度的。
WebGL 门槛相对较高，计算机图形学需要相对较多的数学知识。一个前端程序员或许还熟悉解析几何，但是还熟悉线性代数的应该寥寥无几了（比如求个逆转置矩阵试试？），更何况使用中强调矩阵运算中的物理意义，这在教学中也是比较缺失。
于是，Three.js 对 WebGL 提供的接口进行了非常好的封装，简化了很多细节，大大降低了学习成本。并且，几乎没有损失 WebGL 的灵活性。
因此，从 Three.js 入手是值得推荐的，这可以让你在较短的学习后就能面对大部分需求场景。


#### **three.js的整体结构**

![](../assets/global.svg)

- 首先有一个渲染器(Renderer)。这可以说是three.js的主要对象。你传入一个场景(Scene)和一个摄像机(Camera)到渲染器(Renderer)中，然后它会将摄像机视椎体中的三维场景渲染成一个二维图片显示在画布上。

- 其次有一个场景图 它是一个树状结构，由很多对象组成，比如图中包含了一个场景(Scene)对象 ，多个网格(Mesh)对象，光源(Light)对象，群组(Group)，三维物体(Object3D)，和摄像机(Camera)对象。一个场景(Scene)对象定义了场景图最基本的要素，并包了含背景色和雾等属性。这些对象通过一个层级关系明确的树状结构来展示出各自的位置和方向。子对象的位置和方向总是相对于父对象而言的。比如说汽车的轮子是汽车的子对象，这样移动和定位汽车时就会自动移动轮子。你可以在场景图的这篇文章中了解更多内容。

- 注意图中摄像机(Camera)是一半在场景图中，一半在场景图外的。这表示在three.js中，摄像机(Camera)和其他对象不同的是，它不一定要在场景图中才能起作用。相同的是，摄像机(Camera)作为其他对象的子对象，同样会继承它父对象的位置和朝向。在场景图这篇文章的结尾部分有放置多个摄像机(Camera)在一个场景中的例子。

- 网格(Mesh)对象可以理解为用一种特定的材质(Material)来绘制的一个特定的几何体(Geometry)。材质(Material)和几何体(Geometry)可以被多个网格(Mesh)对象使用。比如在不同的位置画两个蓝色立方体，我们会需要两个网格(Mesh)对象来代表每一个立方体的位置和方向。但只需一个几何体(Geometry)来存放立方体的顶点数据，和一种材质(Material)来定义立方体的颜色为蓝色就可以了。两个网格(Mesh)对象都引用了相同的几何体(Geometry)和材质(Material)。

- 几何体(Geometry)对象顾名思义代表一些几何体，如球体、立方体、平面、狗、猫、人、树、建筑等物体的顶点信息。Three.js内置了许多基本几何体 。你也可以创建自定义几何体或从文件中加载几何体。

- 材质(Material)对象代表绘制几何体的表面属性，包括使用的颜色，和光亮程度。一个材质(Material)可以引用一个或多个纹理(Texture)，这些纹理可以用来，打个比方，将图像包裹到几何体的表面。

- 纹理(Texture)对象通常表示一幅要么从文件中加载，要么在画布上生成，要么由另一个场景渲染出的图像。

- 光源(Light)对象代表不同种类的光。



![](../assets/helloworld.svg)

**branch page1**

> Three.js需要使用canvas标签来绘制,如果你没有给three.js传canvas，three.js会自己创建一个 ，但是你必须手动把它添加到文档中。

> 摄像机默认指向Z轴负方向，上方向朝向Y轴正方向。我们将会把立方体放置在坐标原点，所以我们需要往后移一下摄像机才能显示出物体。

**branch page2**
![](../assets/page2.svg)

**branch page3**
![](../assets/page3.svg)


#### Style

**branch page3**

> 解决拉伸的问题。为此我们要将相机的宽高比设置为canvas的宽高比。 我们可以通过canvas的clientWidth和clientHeight属性来实现。

> HD-DPI代表每英寸高密度点显示器(视网膜显示器)。它指的是当今大多数的Mac和windows机器以及几乎所有的智能手机。

浏览器中的工作方式是不管屏幕的分辨率有多高使用CSS像素设置尺寸会被认为是一样的。 同样的物理尺寸浏览器会渲染出字体的更多细节。



#### **Three.js 中的一些概念**
想在屏幕上展示 3D 物体，大体上的思路是这样的：
1. 创建一个三维空间，Three.js 称之为场景（ Scene ）
2. 确定一个观察点，并设置观察的方向和角度，Three.js 称之为相机（ Camera ）
3. 在场景中添加供观察的物体，Three.js 中有很多种物体，如 Mesh、Group、Line 等，他们都继承自 Object3D 类。
4. 最后我们需要把所有的东西渲染到屏幕上，这就是 Three.js 中的 Renderer 的作用。


下面来仔细看看这些概念吧。

**图元**

> Three.js 有很多图元。图元就是一些 3D 的形状，在运行时根据大量参数生成。

[图元手册](https://threejsfundamentals.org/threejs/lessons/zh_cn/threejs-primitives.html)

**branch material**

**场景图**

![](../assets/scene.svg)

**branch scene1**

> 地球绕着太阳转，月球绕着地球转，月球绕着地球转了一圈。从月球的角度看，它是在地球的 "局部空间 "中旋转。尽管它相对于太阳的运动是一些疯狂的像螺线图一样的曲线，但从月球的角度来看，它只需要关注自身围绕地球这个局部空间的旋转即可。


**branch scene2**

![](../assets/scene2.svg)

> 创建地球并加入太阳场景,到底发生了什么？为什么地球和太阳一样大？为什么离太阳这么远？我居然要把摄像机从 50 单位移到 150 单位以上才能看到地球。


**branch scene3**

![](../assets/scene3.svg)

> sunMesh 和 earthMesh 都是 solarSystem 的子网格。三者都在旋转，现在因为 earthMesh 不是 sunMesh 的子网格，所以不再按 5 倍比例缩放。

**branch scene4**
![](../assets/scene4.svg)

> 可以看到月亮照着螺线图形式旋转，但我们不必手动计算它。我们只需要设置我们的场景图来为我们做这件事。


**材质**

![](../assets/material.jpg)

- MeshBasicMaterial 不受光照的影响。
- MeshLambertMaterial 只在顶点计算光照。
- MeshPhongMaterial 则在每个像素计算光照。


**纹理**

``` js
loader.load('https://threejsfundamentals.org/threejs/resources/images/wall.jpg', (texture) => {
    const material = new THREE.MeshBasicMaterial({
      map: texture,
    });
    const cube = new THREE.Mesh(geometry, material);
    scene.add(cube);
    cubes.push(cube);  // add to our list of cubes to rotate
  });
```

> 纹理支持Promise加载


**摄像机**

[摄像机轮廓](https://threejsfundamentals.org/threejs/lessons/zh_cn/threejs-cameras.html)













## Q&A

- Uncaught TypeError: THREE.CSS3DRenderer is not a constructor

> CSS3DRenderer 并没有包含在 three.js 中，你需要单独引入一个 CSS3DRenderer.js（文件可以在源码的examples/js/renderer中找到）



[](https://www.scaugreen.cn/posts/30679/)

[](https://zhuanlan.zhihu.com/p/27296011)

[Three.js 基础](https://threejsfundamentals.org/threejs/lessons/zh_cn/)

[three js 手册](https://threejs.org/docs/index.html#manual/zh/introduction/Creating-a-scene)

[WebGL 跨域图像](https://webglfundamentals.org/webgl/lessons/zh_cn/webgl-cors-permission.html)