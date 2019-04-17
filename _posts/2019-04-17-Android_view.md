---
layout: post
title: "View 的事件体系(1)"
subtitle: "Android View Motion"
author: "beforenight"
header-img: "img/post-bg-android.jpg"
header-mask: 0.4
tags:
  - Android
  - View
  - 适配
---

View 不属于四大组件，但 是它的作用堪比四大组件，甚至比 Receiver 和 Provider 的重要性都要大。
> 在 Android 幵发 中，Activity承担这可视化的功能，同时 Android 系统提供了很多基础控件，常见的有Button、 TextView、CheckBox 等。很多时候仅仅使用系统提供的控件是不能满足需求的，因此我们 就需要能够根据需求进行新控件的定义，而控件的自定义就需要对 Android 的View体系有深入的理解，只有这样才能写出完美的自定义控件。同时 Android 手机属于移动设备，移动设备的一个特点就是用户可以直接通过屏幕来进行一系列操作，一个典型的场景就是屏幕的滑动，用户可以通过滑动来切换到不同的界面。很多情况下我们的应用都需要支持 滑动操作，当处于不同层级的View都可以响应用户的滑动操作时，就会带来一个问题， 那就是滑动冲突。如何解决滑动冲突呢？这的确是个头疼的问题，其实解决滑动冲突本不难，它需要对 View 的事件分发机制有一定的了解，在这个基础上，我们就可以利于这个特性从而得出滑动冲突的解决方法。


<a name="1e00523c"></a>
# View 基础知识
主要介绍 的内容有：View 的位置参数、MotionEvent 和 TouchSlop 对象、VelocityTracker、 GestureDetector和Scroller对象，通过对这些基础知识的介绍，可以方便读者理解更复杂的内容。
<a name="e1027d8f"></a>
## 什么是 View
View 是 Android 中所有控件的基类，不管是简单的 Button 和 TextView 还是复杂的 RelativeLayout 和 ListView. 它们的共同基类都是 View。所以说，View 是一种界面层的控件的一种抽象，它代表了一 个控件。除了 View,还有ViewGroup,从名字来看，它可以被翻译为控件组，言外之意是 ViewGroup 内部包含了许多个控件，即一组 View。在 Android 的设计中，ViewGroup 也继 承了 View,这就意味着 View 本身就可以是单个控件也可以是由多个控件组成的一组控件， 通过这种关系就形成了 View 树的结构，这和 Web前端中的 DOM 树的概念是相似的。根据这个概念，我们知道，Button 显然是个 View，而 LinearLayout 不但是一个 View 而且还 是一个 ViewGroup，而 ViewGroup 内部是可以有子 View 的，这个子 View 同样还可以是 ViewGroup，以此类推。

TextView&Button 的层次结构如下：<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/178685/1555469762680-4ae6bba1-25da-4c82-9202-188b745aad51.png#align=left&display=inline&height=403&name=image.png&originHeight=806&originWidth=1160&size=285248&status=done&width=580)<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/178685/1555469816681-2f65d7bc-2b50-4292-ad76-d7fbea1a447c.png#align=left&display=inline&height=352&name=image.png&originHeight=704&originWidth=762&size=182990&status=done&width=381)
<a name="7e5a79be"></a>
## View 的位置参数
View 的位置主要由它的四个顶点来决定，分别对应于 View 的四个属性:top、left、right、 bottom,其中 top 是左上角纵坐标，left 是左上角横坐标，right 是右下角横坐标，bottom 是右下角纵坐标。需要注意的是，这些坐标都是相对于 View 的父容器来说的，因此它是一种 相对坐标，View 的坐标和父容器的关系如下图所示。在 Android中，x轴和y轴的正方向分别为右和下，这点不难理解，不仅仅是Android，大部分显示系统都是按照这个标准来 定义坐标系的。<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/178685/1555469936248-8499a575-442d-45cb-84e7-05e5d9c436ea.png#align=left&display=inline&height=395&name=image.png&originHeight=790&originWidth=1000&size=204969&status=done&width=500)

View的宽高和View坐标的关系:
```java
// 宽度
width = right - left;
// 高度
height = bottom - top;
```
获取View的位置参数:
```java
// 相当简单吧, getXXX()系列方法.
left = getLeft();
right = getRight();
top = getTop();
bottom = getBottom();
```
Android 3.0 增加了几个额外的参数: x, y, translationX, translationY<br />它们都是相对于父控件而言的位置.
* (x, y) : 是View左上角坐标.**注意:此时的坐标系是父控件的左上顶点为原点**
* translationX : View相对于父控件在X轴上的位移量.初始值为 0.
* translationY : View相对于父控件在Y轴上的位移量.初始值为 0.

**注意 : 对于translationX 和 translationY来说如果控件没有发生移动他们就都是0,只有View发生了位移translationX和translationY的值才会有变化.**<br />三种位置参数关系:
```java
x = left + translationX;
y = top + translationY;
```
在上面的公式中,**left 和 top** 的值是始终不变的. 在平移过程中x, y ,translationX, translationY 变化如下 :<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/178685/1555470023858-42b11e24-8a23-4d93-87cf-7a72b4dd8be6.png#align=left&display=inline&height=378&name=image.png&originHeight=756&originWidth=1000&size=174920&status=done&width=500)

**注意1 : 以下变化都是通过属性动画改变View的位置**<br />**注意2 : 移动过程中top, left, bottom, right 是不会不变化的.**
* 当View向 x轴负方向移动时(向左):
  * x 减小.
  * translationX 减小.
* 当View向 x轴正方向移动(向右):
  * x 增大.
  * translationX 增大


![image.png](https://cdn.nlark.com/yuque/0/2019/png/178685/1555470042465-118fa14a-dd96-4018-8d02-f7f62b9fa63a.png#align=left&display=inline&height=377&name=image.png&originHeight=753&originWidth=1000&size=240930&status=done&width=500)

* 当View向 y轴负方向移动时(向上):
  * y 减小.
  * translationY 减小.
* 当View向 y轴正方向移动(向下):
  * y 增大.
  * translationY 增大


<br />
![image.png](https://cdn.nlark.com/yuque/0/2019/png/178685/1555470062559-6b2e8177-a9d2-4377-a857-618baebbf4a2.png#align=left&display=inline&height=372&name=image.png&originHeight=744&originWidth=1000&size=232697&status=done&width=500)

**总结 :**
* 上述位置参数都是相对于父控件的而言的.
* 只有在程序运行过程中发生位移x , y, translationX , translationY才会变化.
* 移动过程中 top, left bottom, right不会发生变化。

<a name="1fc5ccdf"></a>
## View 的 MotionEvent 和 TouchSlop
**MotionEvent** 在手指接触屏幕后所产生的一系列事件中，典型的事件类型有如下几种：<br />• ACTION_DOWN——手指刚接触屏幕;<br />• ACTION_MOVE——手指在屏幕上移动;<br />• ACTION_UP——手机从屏幕上松开的一瞬间。

正常情况下，一次手指触摸屏幕的行为会触发一系列点击事件，考虑如下几种情况： 
* 点击屏幕后离开松开，事件序列为DOWN-> UP;
* 点击屏幕滑动一会再松开，事件序列为DOWN -> MOVE ->...> MOVE-> UP。

上述三种情况是典型的事件序列，同时通过 MotionEvent 对象我们可以得到点击事件 发生的x和y坐标。为此，系统提供了两组方法：getX/getY和getRawX/getRawY。它们的 区 别 其 实 很 简 单 ，getX/getY 返回的是相对于当 前 View 左上角的x和y坐标，而 getRawX/getRawY 返回的是相对于手机屏幕左上角的x和y坐标。

**TouchSlop**是系统所能识别出的被认为是滑动的最小距离，换句话说，当手指在屏幕上 滑动时，如果两次滑动之间的距离小于这个常景，那么系统就不认为你是在进行滑动操作， 原因很简单：滑动的距离太短，系统不认为它是滑动。这是一个常暈，和设备有关，在不 同设备上这个值可能是不同的，通过如下方式即可获取这个常量：**ViewConfiguration. get(getContext()).getScaledTouchSlop()**。这个常量有什么意义呢？当我们在处理滑动时，可 以利用这个常量来做一些过滤，比如当两次滑动事件的滑动距离小于这个值，我们就可以 认为未达到滑动距离的临界值，因此就可以认为它们不是滑动，这样做可以有更好的用户 体验。其实如果细心的话，可以在源码中找到这个常量的定义，在**frameworks/base/core/ res/res/values/config.xml **文件中，如下所示。这个"**config_viewConfigurationTouchSlop**" 对应的就是这个常量的定义。

```xml
<! -- Base "touch slop" value used by ViewConf iguration as a movement thresholdwhere scrolling should begin.-->
<dimen name=nconfig_viewConfigurationTouchSlopH>8dp</dimen>
```







