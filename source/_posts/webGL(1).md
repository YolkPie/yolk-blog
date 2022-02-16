---
title: webGL相关知识介绍（1）
date: 2021-12-28
author: 7
---
# webGL（1）
## 基础：关于CPU与GPU
![这是图片](https://pic2.zhimg.com/80/918367f36e34c18dc1f92bd16760dae1_1440w.jpg?source=1940ef5c)
大家都知道在计算机中存在CPU和GPU，而我们这次要讲到的知识和GPU息息相关，所以需要简单了解一些相关知识。  
CPU与GPU都是用于处理数据的工具，但是却又大不相同，CPU需要很强的通用性来处理各种不同的数据类型，同时又要逻辑判断又会引入大量的分支跳转和中断的处理。这些都使得CPU的内部结构异常复杂。而GPU面对的则是类型高度统一的、相互无依赖的大规模数据和不需要被打断的纯净的计算环境。  

简单来说，就是CPU用来处理各种复杂的运算，而GPU更适合处理简单但数量级较高的运算。

## 什么是webGL？
WebGL是一种3D绘图标准，这种绘图技术标准允许把JavaScript和OpenGL ES 2.0结合在一起，通过增加OpenGL ES 2.0的一个JavaScript绑定，WebGL可以为HTML5 Canvas提供硬件3D加速渲染（部分计算GPU），这样Web开发人员就可以借助系统显卡来在浏览器里更流畅地展示3D场景和模型了，还能创建复杂的导航和数据视觉化。总结一下，WebGL的本质 —— JavaScript操作OpenGL接口。
（笔者个人理解，webGL即通过js操作浏览器来直接调用GPU绘制图画，如有错误欢迎指正。）
![这是图片](https://pic3.zhimg.com/80/v2-837a35848cf0d034dcbfa563d52cd472_1440w.jpg)

## 开始使用
为了使用 WebGL 进行 3D 渲染，首先需要一个 canvas 元素。下面的 HTML 片段用来建立一个 canvas 元素并设置一个 onload 事件处理程序来初始化我们的 WebGL 上下文 。  
```html
<body onload="main()">
  <canvas id="glcanvas" width="640" height="480">
    你的浏览器似乎不支持或者禁用了HTML5 <code>&lt;canvas&gt;</code> 元素.
  </canvas>
</body>
```
### 准备webGL上下文

main() 函数将会在文档加载完成之后被调用。它的任务是设置 WebGL 上下文并开始渲染内容。
```js
// 从这里开始
function main() {
  const canvas = document.querySelector("#glcanvas");
  // 初始化WebGL上下文
  const gl = canvas.getContext("webgl");

  // 确认WebGL支持性
  if (!gl) {
    alert("无法初始化WebGL，你的浏览器、操作系统或硬件等可能不支持WebGL。");
    return;
  }

  // 使用完全不透明的黑色清除所有图像
  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  // 用上面指定的颜色清除缓冲区
  gl.clear(gl.COLOR_BUFFER_BIT);
}
```
我们所要做的第一件事就是是获取 canvas 的引用，把它保存在 ‘canvas’ 变量里。

当我们获取到 canvas之后，我们会调用getContext 函数并向它传递"webgl"参数，来尝试获取WebGLRenderingContext。如果浏览器不支持webgl,getContext将会返回null，我们就可以显示一条消息给用户然后退出。

如果WebGL上下文成功初始化，变量 ‘gl’ 会用来引用该上下文。在这个例子里，我们用黑色清除上下文内已有的元素。（用背景颜色重绘canvas）。

WebGLRenderingContext:提供基于 OpenGL ES 2.0 的绘图上下文，用于在 HTML canvas 元素内绘图。
gl.clear:用预设颜色清空绘图区域，擦除原有内容  

现在，我们会得到一个这样黑色的矩形画布。接下来，就可以在画布上绘制我们想要的。

首先让我们了解一下webGL能绘制的基本图形。

### webGL中的基本图形
接下来我们来讲讲在 WebGL 中如何绘制一些基本图形

在 WebGL 的世界中，只有三种基本图形：点、线、三角形  
我们所见到的各种复杂模型，如一些游戏人物都是由这三种基本图形产生。
![如图](https://static001.geekbang.org/infoq/03/03e0defaa0490ab3d7ec9ab4db262557.png)
如上图所示，这样的许多三角形近似的构成了一个圆，如果我们把这些三角形划分的足够小，那么这个圆也就会越光滑，越逼真。对于 3D 图形，也同样如此，如下图所示。
![如图](https://static001.geekbang.org/infoq/53/5377657c3db653a47c2187d18290a1c7.png)
我们是如何告诉 WebGL 要选择什么样的基本图形呢？我们通过 gl.drawArray(mode, first, count)这个 API 在告诉 WebGL 我们要绘制什么样的基本图形，其中 mode 参数就是绘制的基本图形类型，具体的方式如下图所示（参考《WebGL 编程指南》）
![如图](https://static001.geekbang.org/infoq/48/48904d0996ba097fed0655fce92ae7ae.png)
![如图](https://static001.geekbang.org/infoq/fe/fe438dd875d9ff1ad005682d07239bc5.png)

### 利用webGL绘制一个点
```js
//顶点着色器
const vsSource = `
    attribute vec4 aVertexPosition;

    void main() {
      gl_Position = aVertexPosition;
      gl_PointSize = 5.0;
    }
  `

//片元着色器
const fsSource = `
    void main() {
      gl_FragColor = vec4(1.0,0.0,0.0,1.0);
    }
  `

 // 从这里开始
function main() {
  const canvas = document.querySelector("#glcanvas");
  // 初始化WebGL上下文
  const gl = canvas.getContext("webgl");

  // 确认WebGL支持性
  if (!gl) {
    alert("无法初始化WebGL，你的浏览器、操作系统或硬件等可能不支持WebGL。");
    return;
  }

  // 创建着色器，上传代码并编译
  function loadShader(gl, type, source) {
    // 创建着色器对象
    const shader = gl.createShader(type);
  
    // 向着色器对象中填充着色器程序的源代码
    gl.shaderSource(shader, source);

    // 编译着色器（gl.compileShader()）
    gl.compileShader(shader);

    // 检查编译状态，第二个参数可以传入不同值代表不同操作
    // gl.COMPILE_STATUS：编译是否通过
    // gl.SHADER_TYPE：顶点还是片元着色器
    // gl.DELETE_STATUS：删除是否成功
    if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
      gl.deleteShader(shader);
      return null;
    }

    return shader;
  }

  //  初始化着色器程序，让WebGL知道如何绘制我们的数据
  function initShaderProgram(gl, vsSource, fsSource) {
    const vertexShader = loadShader(gl, gl.VERTEX_SHADER, vsSource);
    const fragmentShader = loadShader(gl, gl.FRAGMENT_SHADER, fsSource);

    // 创建着色器程序
    const shaderProgram = gl.createProgram();
    // 为程序对象分配着色器
    gl.attachShader(shaderProgram, vertexShader);
    gl.attachShader(shaderProgram, fragmentShader);
    // 链接程序对象
    gl.linkProgram(shaderProgram);

    // 创建失败， alert
    if (!gl.getProgramParameter(shaderProgram, gl.LINK_STATUS)) {
      alert('Unable to initialize the shader program: ' + gl.getProgramInfoLog(shaderProgram));
      return null;
    }

    return shaderProgram;
  }

  //初始化着色器
  if(!initShaderProgram(gl, vsSource, fsSource)){
      console.log("初始化着色器失败");
      return;
  }

  const shaderProgram = initShaderProgram(gl, vsSource, fsSource);

  //指定一个覆盖（清空）canvas的颜色
  gl.clearColor(0.0, 0.0, 0.0, 1.0);

  //执行清空
  gl.clear(gl.COLOR_BUFFER_BIT);

  // 使用哪个程序
  gl.useProgram(shaderProgram);

  //绘制一个点
  gl.drawArrays(gl.POINTS, 0, 1);

}
```
这样我们就可以在画布上绘制出一个点。  
其中有几个重要概念。
### 着色器
在这不得不说一下webGL的渲染原理，如果想渲染 3D 图形，就需要经过一系列的步骤，这些步骤称为渲染管线。在开发 WebGL 程序时，我们就需要通过着色器语言跟GPU进行沟通，用来设定我们需要渲染和显示的图形。

大家可以认为 WebGL 是一套复杂的电路。我们将其中的一些变量连接起来，然后为其加上电源，那么 WebGL 就可以正常的工作起来了。
![如图](https://static001.geekbang.org/infoq/98/98a4761dd3ab7fef713afc3fb6b1785d.png)

所谓的渲染管线，实际上就是渲染过程流水线，指的不是具体某一样东西，而是一个流程。因为渲染管线的流程中总是将上一步的结果作为下一步的输入，就像水管一样接起来，管线的名字也因此得来。

下图简要的展示了渲染管线的一个流程
![如图](https://static001.geekbang.org/infoq/c1/c1e8c04c034d8eef1e9f4cda6838bd95.png)

接下来介绍着色器，
着色器是编写 WebGL时最重要的一点（没有之一）。我们之所以能生成并操作 3D图像，都是因为着色器在起作用。WebGL中着色器分为两种。顶点着色器和片元着色器。

```js
// 顶点着色器
const vsSource = `
    attribute vec4 aVertexPosition;

    void main() {
      gl_Position = aVertexPosition;
      gl_PointSize = 5.0;
    }
`
```
顶点着色器的任务就是产生投影矩阵的坐标。
我们著行解释一下它的含义  
1、声明了一个全局 vec4 类型的全局变量，attribute 是一个存储限定符，被它所修饰的变量表示是接受外部传入的顶点属性的。它是 WebGL 外部顶点信息传入 WebGL 内部的桥梁变量。vec4 类型是 4 维向量类型，用于表示顶点的坐标信息。这时有同学就会问了坐标信息不就是 x, y, z 吗，这明明只需要 3 维向量就可以了呀，哪里来的第四个参数呢？这是因为在 WebGL 中是采用的齐次坐标表示的坐标信息，我们用这样的形式(x, y, z, w) 来表示一个坐标的位置。大家可以将它等同于这样的一个三位坐标(x / w, y / w, z / w)  
2、void main 表示顶点着色器的 main 函数，类似于 C 语言的 main 函数，他是顶点着色器中的唯一入口函数。  
3、gl_Position 是顶点着色器中的内置变量，它就表示了当前顶点的实际位置，所以我们需要将从外界接受信息的 aVertexPosition 的值赋给 gl_Position 这个变量。
4、gl_PointSize  类型：float  表示点的尺寸（像素数） 如果不设置，默认为1.0

```js
    // 片元着色器
    const fsSource = `
    void main() {
      gl_FragColor = vec4(1.0,0.0,0.0,1.0);
    }
  `
```
片段着色器的任务就是为当前被栅格化的像素提供颜色。
1、main 函数，与顶点着色中的含义相同  
2、将 color 的值赋给 WebGL 的内置变量 gl_FragColor， gl_FragColor 表示的就是当前片元的颜色值

除了上文中出现的
```js
attribute
```
外，还存在 uniform, varying 这两种存储限定符，

attribute: 只能出现在顶点着色器中，被用来从外部向 WEBGL 内部中传递顶点信息

uniform: 可以出现在顶点着色器和片元着色器中，表示统一的值

varying:（光栅化阶段）可以出现在顶点着色器和片元着色器中，表示变化的值，是顶点着色器和片元着色器之间的连接桥梁

因暂时还未用到，暂时不做讲解。

现在，我们已经定义好了两个着色器，我们需要将他们传递给webGL，编译并连接到一起。首先，我们通过loadShader()，为着色器指定类型和来源，创建两个着色器，接着，初始化一个着色器程序，他的作用是让webGL知道如何绘制我们的数据，在初始化操作中，将两个链接到一起，如代码所示
```js
// 创建着色器，上传代码并编译
  function loadShader(gl, type, source) {
    // 创建着色器对象
    const shader = gl.createShader(type);
  
    // 向着色器对象中填充着色器程序的源代码
    gl.shaderSource(shader, source);

    // 编译着色器（gl.compileShader()）
    gl.compileShader(shader);

    // 检查编译状态，第二个参数可以传入不同值代表不同操作
    // gl.COMPILE_STATUS：编译是否通过
    // gl.SHADER_TYPE：顶点还是片元着色器
    // gl.DELETE_STATUS：删除是否成功
    if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
      gl.deleteShader(shader);
      return null;
    }

    return shader;
  }

  //  初始化着色器程序，让WebGL知道如何绘制我们的数据
  function initShaderProgram(gl, vsSource, fsSource) {
    const vertexShader = loadShader(gl, gl.VERTEX_SHADER, vsSource);
    const fragmentShader = loadShader(gl, gl.FRAGMENT_SHADER, fsSource);

    // 创建着色器程序
    const shaderProgram = gl.createProgram();
    // 为程序对象分配着色器对象
    gl.attachShader(shaderProgram, vertexShader);
    gl.attachShader(shaderProgram, fragmentShader);
    // 链接程序对象
    gl.linkProgram(shaderProgram);

    // 创建失败， alert
    if (!gl.getProgramParameter(shaderProgram, gl.LINK_STATUS)) {
      alert('Unable to initialize the shader program: ' + gl.getProgramInfoLog(shaderProgram));
      return null;
    }

    return shaderProgram;
  }
```

之后，我们就可以像这样调用他
```js
const shaderProgram = initShaderProgram(gl, vsSource, fsSource);
```
并告诉webGL我们要使用的程序
```js
gl.useProgram(shaderProgram);
```

接着，就可以绘制出一个点
```js
gl.drawArrays(gl.POINTS, 0, 1);
```

至此webGL第一节结束。。。