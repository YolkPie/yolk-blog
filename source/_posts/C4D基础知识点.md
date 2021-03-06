---
title: C4D基础知识点
date: 2021-06-29 18:03:15
tags:
- 设计
- 色彩
categories: 设计
author: 李黎
keywords: 设计,色彩
description: 设计,色彩
cover: https://img12.360buyimg.com/imagetools/jfs/t1/186575/18/10907/170470/60db0befE22ca54e7/75de6ad5f7cea23c.png
top_img: https://img12.360buyimg.com/imagetools/jfs/t1/190063/32/10770/265458/60db0c2fE8d030c93/825d63004625cddd.png
---
### 基础功能点

1. **视图窗口中物体**的旋转、缩放和位移

物体旋转：Alt+鼠标左键

物体缩放：Alt+鼠标右键

物体推拉：Alt+鼠标滚轮（中键）

三维空间中的坐标系称为笛卡尔坐标系；

2. 基础快捷键

旋转：快捷键R、缩放：快捷键T、位移：快捷键E、快速找到当前物体：快捷键H&S、物体复制：command+鼠标左键往下拖动、物体和世界切换：快捷键W、场景的四视图：鼠标中键、图层群组：alt+g（或者全选右击-群租对象）关闭当前预置：ctrl+F4

3. 找到当前C4D文件：窗口右下角（可直接切换多个文件）

关闭当前文件：窗口左上角文件--关闭（全部关闭）

合并两个文件：在其中一个文件下，点击左上角文件--合并--选择你想要合并的文件

4. 等比旋转：先点击R调出旋转命令，把鼠标放在你想要旋转的轴上（此时你想旋转的轴将会点亮），右手按住鼠标左键不放，左手按住shift旋转。

5. 物体的可编辑：快捷键C(物体转换为可编辑对象是不可逆的，物体转换为可编辑对象以后可以激活右键中的属性）

6. 循环选择线：上方菜单栏--循环选择或用移动工具在循环线上双击

7. 所有C4D中绿色的图标都是包含级关系（即需要把你所想要做效果的物体放在效果下边）

8. 地面与平面的区别：地面不具备编辑属性，平面有限，地面无限

9. 世界边缘的透明度调整：shift+V，视图窗口的调整
10. 简单的一套渲染流程：给物体上材质--加上       天空（光照）--渲染效果设置--保存

11. 在选择挤压命令时要记得勾选层级

12. 在二维世界编辑点的时候，勾掉"仅选择可见元素"，编辑完点记得要勾上"仅选择可见元素

13. 倒角的四种方法：C掉之前直接倒角，C掉以后右击倒角线，将物体细分曲面，将上方紫色的倒角放在需要倒角的物体下面（上下级关系，此处可以与绿色对比理解）

14. 摄像机：默认摄像机默认打开，当你调整好角度后可以点击摄像机新建一个摄像机，这样当你移动后还可以点击创建的摄像机后面的十字准星回到原本设置好的角度

15. 在正视图的情况下，选择模式-视图设置就能调整背景素材的透明度。

16. 图形对称，需要先建立空白坐标系，然后将物体作为空白坐标系的子级，按住alt键，选择对称

17. 建模方式：

①Nurbs:曲面建模(Non-uniform Rational B Splines)

应用领域：工业产品表现(生活家电、汽车等) 、辅助视觉设计(抽象化曲线设计)、

电影特效(玻璃破碎等)

②多边形建模poly：应用领域：

特点：由点线面构成，操作简便，可控性高，自由度高，要求布线，对造型能力要求高；

18. C4Dnurbs主要特点：无点线面、主要依靠样条、操作简便

​      其他软件nurbs的主要特点：少数点控制多数点、命令复杂

19. 挤压模式下：可以勾选层级，挤压就对子级的所有层级起作用**。**

### 操作栏知识点     

![工具栏功能](https://img13.360buyimg.com/imagetools/jfs/t1/181588/20/11690/1223977/60dae859Ed698a00c/7c990f8fa2b1378f.jpg)

![工具栏功能3](https://img11.360buyimg.com/imagetools/jfs/t1/193551/9/9943/1223977/60dae859Eb7ef2b6c/acd60ff7c879338f.jpg)

![绿色功能](https://img12.360buyimg.com/imagetools/jfs/t1/174501/33/17242/56259/60dae85aE554639f8/f47fbee325ba7044.jpg)



绿色功能通常作为父级使用，物体为子级产生作用。

放样：两个样条点和线必须相同才能用放样，两条同样的竖线放样后形成矩形面

扫描：适合做管状的物体，线画一个曲线的线条，然后再画圆环，再将两个层作为扫描的子级，就形成管状。

![紫色功能](https://img10.360buyimg.com/imagetools/jfs/t1/178074/32/11820/159468/60dae85bEb713e879/3351f5ab6b7375da.jpg)

紫色功能通常作为子级使用，物体为父级产生作用。

### 关于材质 

1. 漫射通道：物体本身的颜色。
2. 发光通道：萤火虫的尾巴，自发光模式；
3. 透明通道：透明亮度：数值越大，物体越透明；

各种物体的折射率：真空:1.0、空气:1.0003、冰:1.309、水:1.3333、酒精:1.3600、玻璃:1.5000

菲涅尔反射：菲涅尔反射率数值越大，中心反射越小，外部反射越大；

吸收颜色：物体暗部反射环境的颜色；

模糊：制作毛玻璃的时候会使用模糊；

4. 反射通道：物体对周围环境的反射；
5. 环境通道：假反射；
6. 烟雾通道：物体表面雾蒙蒙；
7. 凹凸通道：噪波，物体表面假凹凸不平；
8. Alpha：理解为镂空，直接透明的算法，不会在中间计算折射，某些地方显示，某些地方不显示；
9. 高光：对光源点的反射；

   参数：宽度-高光范围的大小，范围越大表面越粗糙，参数越小表面越平滑；

   高度：可以理解为高光的强度；

   衰减：指的是中心到四周的衰减大小；

   模式：塑料、金属、固有色；

10. 辉光：理解为外发光；
11. 置换通道：把模型表面发生真变化；
12. 建筑材质球和默认材质球的区别：没有贴图功能；

### 关于输出

![渲染设置](https://img10.360buyimg.com/imagetools/jfs/t1/191530/37/10855/173922/60dae85aE592817ed/01e684cf67b7a1c9.jpg)

1. 物体的渲染和输出

Ctrl+R:在做好之前预览窗口渲染，即可以快速查看当前窗口渲染效果；Shift+R：输出渲染；Alt+R：预览跟窗口预览相似，但是它能一直停留；

2. 渲染设置中提高精度值：抗锯齿选项开到最优；环境吸收选项取样值拉大；

3. 灯光和阴影：灯光具有叠加属性，无上限，有下限；软阴影：计算机模拟贴图制作的假阴影（渲染速度块）；硬阴影：模拟边缘锐利的阴影；区域：通过给的灯光大小来模拟真实阴影；

4. 如何判断灯光的好坏:有高光，有阴影，有过度；

5. 摄像机的固定：摄像机右击第一个选项里面的保护；

6. 保存带有通道：在渲染设置里面勾选Alpha通道、保存png图片；

7. 渲染输出一般尺寸选择：1280*720，预置：HDV高清；

8. 渲染输出的帧频一定要和工程里边的帧频数保持一致；

9. 输出为视频格式：quitetime影片；