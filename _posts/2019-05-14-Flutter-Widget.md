---
layout: post
title: "Flutter快速上车之Widget"
subtitle: "Action in Flutter Widget"
author: "beforenight"
header-img: "img/post-bg-android.jpg"
header-mask: 0.4
tags:
  - Flutter
  - Widget
  - 入门
---
# Flutter快速上车之Widget

作者：闲鱼技术-意境

Flutter作为一种全新的响应式，跨平台，高性能的移动开发框架。从开源以来，已经得到越来越多开发者的喜爱。闲鱼是最早一批与谷歌展开合作，并在重要的商品详情页中使用上线的公司。一路走来，积累了大量的开发经验。虽然越来越多的技术大牛在flutter世界中弄得风声水起，但是肯定有很多的flutter小白希望能快速上手，享受flutter编程的乐趣。本文就是面向刚刚踏上futter的同学，从Flutter体系中最基本的一个概念widget入手学习Flutter。希望能助力每一位初学者。

**可能大家要问的第一个问题是为什么从Widget开始？**

![](https://gw.alicdn.com/tfs/TB1ASvQm7omBKNjSZFqXXXtqVXa-1354-706.png#width=)

从flutter的架构图中不难看出widget是整个视图描述的基础。Flutter 的核心设计思想便是

```
Everything’s a Widget
```
即一切即Widget。在flutter的世界里，包括views,view controllers,layouts等在内的概念都建立在Widget之上。widget是flutter功能的抽象描述。所以掌握Flutter的基础就是学会使用widget开始。

本文会从大家熟悉的UI绘制视角来介绍flutter组件和布局的基础知识。首先罗列了UI开发中最为常用，最为基础的组件。下面逐一进行介绍。

![](https://gw.alicdn.com/tfs/TB1wXH5nsUrBKNjSZPxXXX00pXa-1108-542.png#width=)

<a name="renyus"></a>
# [](#renyus)1 组件篇

<a name="uuppxm"></a>
## [](#uuppxm)1.1 Text
Text几乎是UI开发中最为重要的组件之一了，UI上面文字的展示基本上都要靠Text组件来完成。Flutter提供了原生的Text组件。Text的配置属性是很丰富的,属性主要分为两个部分一个是对齐&显示控制相关的在Text类的属性中，另一类是样式相关的属性使用单独的类TextStyle进行控制。跟native控件相比（以android为例），Text的组件基本上提供了同等的能力，并且提供了更加丰富的样式装饰能力。详细的属性可以参考官方文档[flutter text](https://docs.flutter.io/flutter/widgets/Text-class.html).

<a name="n4p0yz"></a>
### [](#n4p0yz)1.1.1 实践Coding

设置文字&文字大小&颜色&行数限制&文本对齐

```
const Text(  "hello flutter!",
            textAlign: TextAlign.center,
            maxLines: 1,
            overflow: TextOverflow.ellipsis, // 溢出显示。。。
            style: TextStyle(fontSize: 30.0,// 文字大小
               color: Colors.yellow),// 文字颜色
          ),
```

效果如下：

![](https://gw.alicdn.com/tfs/TB1PrthpcIrBKNjSZK9XXagoVXa-335-78.png#width=)

<a name="67tzxc"></a>
## [](#67tzxc)1.2 Image

图片也是UI部分开发最为重要的组件之一。在能看图随看文字的年代，图片是页面展示的重中之重！Flutter同样原生提供了Image组件。下面重点介绍一下几个重点:

<a name="9g8ift"></a>
### [](#9g8ift)1.2.1 缩放
怎样设置图片显示的缩放方式呢？<br />Flutter中的图片缩放是fit字段来控制的。这是对最终图片展示效果影响很大的一个参数，也是容易出错的点。下面逐个分析一下flutter Image组件提供的缩放方式。

**缩放属性值在BoxFit枚举中**

下面列出的图片是flutter官方对各种缩放做的图片示例。基本上都表述很清楚了，就整理出来供大家查阅。

| 属性 | 缩放效果 |
| --- | --- |
| fill | ![](https://flutter.github.io/assets-for-api-docs/assets/painting/box_fit_fill.png#width=) |
| contain | ![](https://flutter.github.io/assets-for-api-docs/assets/painting/box_fit_contain.png#width=) |
| cover | ![](https://flutter.github.io/assets-for-api-docs/assets/painting/box_fit_cover.png#width=) |
| fitWidth | ![](https://flutter.github.io/assets-for-api-docs/assets/painting/box_fit_fitWidth.png#width=) |
| fitHeight | ![](https://flutter.github.io/assets-for-api-docs/assets/painting/box_fit_fitHeight.png#width=) |
| none | ![](https://flutter.github.io/assets-for-api-docs/assets/painting/box_fit_none.png#width=) |
| scaleDown | ![](https://flutter.github.io/assets-for-api-docs/assets/painting/box_fit_scaleDown.png#width=) |


<a name="2pguqz"></a>
### [](#2pguqz)1.2.2 图片获取

怎样从各种来源加载图片？<br />默认的Image组件不能直接显示图片，他需要一个ImageProvider来提供具体的图片资源的（即Image中的image字段需要赋值）。咋一看这确实非常麻烦，但是实际上ImageProvider并不需要完全重新自己实现。在Image类中目前提供了一下几个实现好的ImageProvider，基本能满足常见的需求。

| ImageProvider | 用途 |
| --- | --- |
| Image.asset | 从asset资源文件中获取图片 |
| Image.network | 从网络获取图片 |
| Image.file | 从本地file文件中获取图片 |
| Image.memory | 从内存中获取图片 |


**Image同样支持GIF图片**

网络请求Image是大家最常见的操作。这里重点说明两个点：

- 缓存


ImageCache是ImageProvider默认使用的图片缓存。ImageCache使用的是LRU的算法。默认可以存储1000张图片。如果觉得缓存太大，可以通过设置ImageCache的maximumSize属性来控制缓存图片的数量。也可以通过设置maximumSizeBytes来控制缓存的大小（默认缓存大小10MB）。

- CDN优化


如果想要使用cdn优化，可以通过url增加后缀的方式实现。默认实现中没有这个点，但是考虑到cdn优化的可观收益，建议大家利用好这个优化。

<a name="avzynt"></a>
### [](#avzynt)1.2.3 FadeInImage

在实际开发中，考虑到图片加载速度可能不能达到预期。所以希望能增加渐入效果&增加placeHolder的功能。Flutter同样提供的这样的组件——FadeInImage。

FadeInImage也提供了从多种渠道加载图片的能力。这块跟上面所说差异不大。这里不再赘述。

<a name="p741pw"></a>
### [](#p741pw)1.2.4 实践Coding

- 从网络获取图片保持图片比例并尽可能大的放入


```
new Image.network(
            'https://gw.alicdn.com/tfs/TB1CgtkJeuSBuNjy1XcXXcYjFXa-906-520.png',
            fit: BoxFit.contain,
            width: 150.0,
            height: 100.0,
          ),
```

- 效果如下：


![](https://gw.alicdn.com/tfs/TB1YskCpljTBKNjSZFwXXcG4XXa-154-105.png#width=)
<a name="7btbgh"></a>
## [](#7btbgh)1.3 Container

Flutter的设计思想就是完全的widget化。这也就是说连最基本的padding,Center都是widget。设想一下如果每次写view，连padding,Center都要自己包一个组件是一种怎样的体验？作为一个工程师，别给只给我谈思想，实际操作的操作效率也同样非常重要。flutter 官方也意识到了这个问题，他们从实际编写效率的角提供了一个友好高效的封装，这就是Container！首先没有任何疑问，Container 本身也是一个widget。但是他却提供了对基础widget的封装，提高了UI基础装饰能力的表达效率。Container类似于android中的ViewGroup。

相信大部分的属性大家都会感觉非常亲切，结合代码注释都比较容易理解，这里就不再赘述。其中需要重点解释一下的是：Decoration和BoxConstraints。

<a name="wks6pu"></a>
### [](#wks6pu)1.3.1 装饰

Decoration是对Container进行装饰的描述。其概念类似与android中的shape。一般实际场景中会使用他的子类BoxDecoration。BoxDecoration提供了对背景色，边框，圆角，阴影和渐变等功能的定制能力。<br />需要注意几个点：

- BoxDecoration的image属性相当于设置的是背景图。但是image会绘制在color 和gradient之上。

- image是需要一个DecorationImage类的实现。DecorationImage的属性和Image组件比较类似，可以复用Image组件中的ImageProvider。


<a name="atltga"></a>
### [](#atltga)1.3.2 大小

BoxConstraints其实是对Container组件大小的描述。BoxConstraints属性比较简单。如果不太清楚可以研究一下盒子模型。这里有个点需要重点说明一下：

- 如何表达尽可能大这样的意思？（类似于android中的match_parent）Flutter中可以使用**double.infinity**来做出类似的表达。


<a name="3cgmtg"></a>
### [](#3cgmtg)1.3.3 实践Coding

- 设置边框&padding&margin&圆角&背景图


```
new Container(
         alignment: Alignment.center,
         padding: const EdgeInsets.all(15.0),
         margin: const EdgeInsets.all(15.0),
         decoration: new BoxDecoration(
           border: new Border.all(
             color: Colors.red,
           ),
           image: const DecorationImage(
             image: const NetworkImage(
               'https://gw.alicdn.com/tfs/TB1CgtkJeuSBuNjy1XcXXcYjFXa-906-520.png',
             ),
             fit: BoxFit.contain,
           ),
           //borderRadius: const BorderRadius.all(const Radius.circular(6.0)),
           borderRadius: const BorderRadius.only(
             topLeft: const Radius.circular(3.0),
             topRight: const Radius.circular(6.0),
             bottomLeft: const Radius.circular(9.0),
             bottomRight: const Radius.circular(0.0),
           ),
         ),
         child: Text(''),
       ),
```

- 效果如下：


![](https://gw.alicdn.com/tfs/TB14Zd8pmYTBKNjSZKbXXXJ8pXa-328-60.png#width=)

<a name="qegvfg"></a>
## [](#qegvfg)1.4 手势操作

手势操作是最常见的UI交互操作。在Flutter中手势识别也是一个widget！这点对新人来说又是一个新鲜的地方。通常来说可以通过GestureDetector类来完成点击事件的处理。使用时只需要将GestureDetector包裹在目标widget外面，再实现对应事件的函数即可。从点击到长按，从缩放到拖动，这个类基本上都有相应的实现。具体可以参见组件文档。

<a name="77c5dr"></a>
# [](#77c5dr)2. 布局

页面布局应该是UI编写最为根本的知识，其主要的描述的是父子组件子子组件之间的位置关系。首先我们理解一下官方文档的逻辑：

![](https://gw.alicdn.com/tfs/TB1N.o7nCMmBKNjSZTEXXasKpXa-1438-600.png#width=)

将布局分为单孩子和多孩子是Flutter布局的一大特色。这点对native研发同学来说会比较新鲜。单孩子组件主要继承自SingleChildRenderObjectWidget。这些组件能提供丰富的装饰能力（例如container），也能提供部分特定的布局能力（例如center）。多孩子组件继承自MultiChildRenderObjectWidget，能提供更加丰富的布局能力（Flex,Stack,flow）,但几乎没有装饰的能力。下面介绍几个重点布局：

<a name="24r4zc"></a>
## [](#24r4zc)2.1 Flex
Flutter在布局上也提供了完整的Flex布局能力。但是在Flutter官方文档中[Layout Widgets](https://flutter.io/widgets/layout/)，是看不到任何Flex的影子的。映入眼帘的却是Row，Column，这些是什么鬼？其实不难发现类似Row，Column 这样的组件，他们的基类都是Flex。Row和Column差别是设置了不同的flex-direction。而之所这么设计，是因为Flutter的widget从开始设计之初就考虑到UI布局语义保持的重要性。这块应该部分借鉴了前端的经验，极力避免一个div搞定全部页面的尴尬（当然flutter也可以使用Flex来做同样的事情，但是并不建议这么做）。<br />Flutter使用的Flex模型基本上跟传统的Css类似。这块前端同学可以快速上手。但是Flex对于客户端同学来说是一种全新的布局方式。Flex的基础知识可以参看[flex布局基础](http://www.runoob.com/w3cnote/flex-grammar.html)。由于篇幅有限这里不展开叙述。这里只重点强调一个点：<br />如下图flex布局概念如下：<br />![](https://gw.alicdn.com/tfs/TB1fJaXH4SYBuNjSsphXXbGvVXa-961-349.png#width=)<br />flex通过direction设置了flex的主轴方向即main axis。和主轴垂直的方向叫做cross axis。flex布局中对子布局的控制是从main axis 和cross axis两个方向上进行的。例如居中有main axis居中和cross axis居中。两者都居中才是容器的完全居中。这点是客户端同学可能会容易弄混的地方。重点关注一下。

<a name="46buxk"></a>
### [](#46buxk)2.1.1 实践Coding

ok，看完这些知识，我们实际需求角度实际操作几个case来熟悉一下Flex。

- 居中


```
new Flex(direction: Axis.horizontal,
              mainAxisAlignment: MainAxisAlignment.center,
              crossAxisAlignment: CrossAxisAlignment.center,
              children: <Widget>[
              new Container(
                  width: 40.0,
                  height: 60.0,
                  color: Colors.pink,
                  child: const Center(
                    child: const Text("left"),
                  )),
              new Container(
                  width: 80.0,
                  height: 60.0,
                  color: Colors.grey,
                  child: const Center(
                    child: const Text("middle"),
                  )),
              new Container(
                  width: 60.0,
                  height: 60.0,
                  color: Colors.yellow,
                  child: const Center(
                    child: const Text("right"),
                  )),
              ],
            ),
```

- 效果如下：


![](https://gw.alicdn.com/tfs/TB1n0A2pbArBKNjSZFLXXc_dVXa-366-107.png#width=)

- weight left:right=2:1 通过设置Flexible的flex值大小完成比例设置


```
new Flex(
         direction: Axis.horizontal,
         mainAxisAlignment: MainAxisAlignment.spaceEvenly,
         crossAxisAlignment: CrossAxisAlignment.center,
         children: <Widget>[
           new Flexible(
             flex: 2,
             fit: FlexFit.loose,
             child: new Container(
               color: Colors.blue,
               height: 60.0,
               alignment: Alignment.center,
               child: const Text('left!',
                   textAlign: TextAlign.center,
                   style: TextStyle(color: Colors.black),
                   textDirection: TextDirection.ltr),
             ),
           ),
           new Flexible(
             flex: 1,
             fit: FlexFit.loose,
             child: new Container(
               color: Colors.red,
               height: 60.0,
               alignment: Alignment.center,
               child: const Text('right',
                   textAlign: TextAlign.center,
                   style: TextStyle(color: Colors.black),
                   textDirection: TextDirection.ltr),
             ),
           ),
         ],
       )
```

- 效果如下：


![](https://gw.alicdn.com/tfs/TB1qtcZpiMnBKNjSZFzXXc_qVXa-345-104.png#width=)

<a name="kh0xcf"></a>
## [](#kh0xcf)2.2 stack
在实际开发中，还是需要在一些Widgets的上面再覆盖上新的Widgets。这时候就需要层式布局了。这种布局在Native上，以android为例，类似于relativeLayout 或者FrameLayout。在Flutter中使用的是Stack。

实际使用中Stack中的子Widgets分为两种：

- positioned

  - 是包裹在组件Positioned中的组件

  - 可以通过Positioned属性灵活定位

- non-positioned

  - 没有包裹在Positioned组件中

  - 需要通过父Widget Stack 的属性来控制布局


对于non-positioned children， 我们通过控制Stack的alignment属性来控制对齐方式。Positioned的布局方式类似于H5&weex中的position布局中的absolute布局方式。通过设置距离父组件上下左右的距离，Positioned对象能在Stack布局中更加灵活的控制view的展现方式。

<a name="4gf3at"></a>
### [](#4gf3at)2.2.1 实践Coding

- 层叠布局


```
new Container(
            color: Colors.yellow,
            height: 150.0,
            width: 500.0,
            child: new Stack(children: <Widget>[
              new Container(
                color: Colors.blueAccent,
                height: 50.0,
                width: 100.0,
                alignment: Alignment.center,
                child: Text('unPositioned'),
              ),
              new Positioned(
                  left: 40.0,
                  top: 80.0,
                  child: new Container(
                    color: Colors.pink,
                    height: 50.0,
                    width: 95.0,
                    alignment: Alignment.center,
                    child: Text('Positioned'),
                  )),
            ]))
```

- 效果如下：


![](https://gw.alicdn.com/tfs/TB1BAAupbZnBKNjSZFKXXcGOVXa-367-151.png#width=)

<a name="ibexrv"></a>
# [](#ibexrv)3. Visibility
当你看完Flutter Widge文档的时候，我们突然发现一个略显尴尬的问题：组件是否显示怎么控制？貌似所有的组件中都没有这个属性！这不坑了，咋办？

目前看方法无非如下几个：

<a name="o159tt"></a>
## [](#o159tt)3.1 删除法

核心将该真实widget或者widget树从renderTree中移除。

具体到实践级别主要分为两个：

- 单个组件‘隐藏’自己。在build方法中返回一个空的Container.


```
@override
Widget build(BuildContext context) {
  return isVisible
      ? Widget //真的Widget
      : new Container(); //空Widget 仅仅占位 并不显示
}
```

- 多个child


在父容器的children字段的list中，删除掉对应的cell。

<a name="nsaazp"></a>
## [](#nsaazp)3.2 Offstage

Offstage 是一个widget。Offstage的offstage属性设置为true，那么Offstage以及他的child都将不会被绘制到界面上。<br />sample code如下：

```
@override
Widget build(BuildContext context) {
  return new Offstage(
          offstage: !isVisible,
          child:child);
}
```
<a name="1ptuxi"></a>
## [](#1ptuxi)3.3 透明度
设置widget的透明度，使之不可见。但是这样的方法是副作用的。因为这个对应的widget树是已经经过了完整的layout&paint过程，成本高。同时设置透明度本身也要耗费一定的计算资源，造成了二次浪费。需要注意的是即便变透明了，占据的位置还在。大家慎重选择使用。<br />sample code如下：
```
@override
Widget build(BuildContext context) {
  return new AnimatedOpacity(
        duration: Duration(milliseconds: 10),
        opacity: isVisible ? 1.0 : 0.0,
          child:child);
}
```

visibility的控制还是比较麻烦的。这是Flutter设计上不符合正常习惯的一个点，需要大家重点关注。

<a name="dp3ixb"></a>
# [](#dp3ixb)4 生命周期

<a name="pg3msl"></a>
## [](#pg3msl)4.1 state 生命周期
widget是immutable的，发生变化的时候需要重建，所以谈不上状态。StatefulWidget 中的状态保持其实是通过State类来实现的。State拥有一套自己的生命周期，下面做一个简单的介绍。

| 名称 | 状态 |
| --- | --- |
| initState | 插入渲染树时调用，只调用一次 |
| didChangeDependencies | state依赖的对象发生变化时调用 |
| didUpdateWidget | 组件状态改变时候调用，可能会调用多次 |
| build | 构建Widget时调用 |
| deactivate | 当移除渲染树的时候调用 |
| dispose | 组件即将销毁时调用 |


生命周期状态图如下：

![](https://gw.alicdn.com/tfs/TB1GrRfpsUrBKNjSZPxXXX00pXa-1394-1314.png#width=)

**几个注意点**

- didChangeDependencies有两种情况会被调用。

  - 创建时候在initState 之后被调用

  - 在依赖的InheritedWidget发生变化的时候会被调用

- 正常的退出流程中会执行deactivate然后执行dispose。但是也会出现deactivate以后不执行dispose，直接加入树中的另一个节点的情况。

- 这里的状态改变包括两种可能：1.通过setState内容改变 2.父节点的state状态改变，导致孩子节点的同步变化。


<a name="t2kaur"></a>
## [](#t2kaur)4.2 App生命周期

需要指出的是如果想要知道App的生命周期,那么需要通过WidgetsBindingObserver的didChangeAppLifecycleState 来获取。通过该接口可以获取是生命周期在AppLifecycleState类中。常用状态包含如下几个：

| 名称 | 状态 |
| --- | --- |
| resumed | 可见并能相应用户的输入 |
| inactive | 处在并不活动状态，无法处理用户相应 |
| paused | 不可见并不能相应用户的输入，但是在后台继续活动中 |


一个实际场景中的例子：

在不考虑suspending的情况下：从后台切入前台生命周期变化如下:

- AppLifecycleState.inactive->AppLifecycleState.resumed;


从前台压后台生命周期变化如下：

- AppLifecycleState.inactive->AppLifecycleState.paused;


<a name="hlrqub"></a>
# [](#hlrqub)5 初学者的困惑

<a name="gwqgwo"></a>
## [](#gwqgwo)5.1 为什么使用dart语言？

Dart语言对大部分开发者而言是很陌生的一种语言。google为啥会选择如此'冷门'的语言来开发flutter？主要原因如下：

- 

  1. dart具有jit&Aot双重编译执行方式。这样就能利用JIt进行开发阶段的hot reload开发，提升研发效率。同时在最终release版本中使用aot将dart代码直接变成目标平台的指令集代码。简单高效，最大限度保障了性能。

  2. dart针对flutter中频繁创建销毁Widget的场景做了专门的gc优化。通过分代无锁垃圾回收器，将gc对性能的影响降至最低。

  3. dart语言在语法上面是类java的，易学易用。


<a name="muswql"></a>
## [](#muswql)5.2 为什么widget都是immutable？

个人认为是两个主要的点：

- 提高渲染效率<br />flutte在页面渲染上面的核心思想是simple is fast！将widget设计成immutable，所以在数据变化时，flutter选择重建widget树的方式进行数据更新。采用这样方式的好处是框架不需要关心数据影响的范围，简单高效。缺点就是对GC会造成压力。

- 组件描述的复用<br />既然widget都是不可变的。那widget可以以较低成本进行复用。在一个真实的渲染树中可能存在同一个widget渲染树中不同节点的情况。


<a name="37mays"></a>
## [](#37mays)5.3 widget是view么？

可能刚开始接触flutter的同学最疑惑的一个问题就是widget和view的关系了。那么简单分析一下：<br />widget是对页面UI的一种描述。他功能类有点似于android中的xml，或者web中的html。widget在渲染的时候会转化成element。Element相比于widget增加了上下文的信息。element是对应widget，在渲染树的实例化节点。由于widget是immutable的，所以同一个widget可以同时描述多个渲染树中的节点。但是Element是描述固定在渲染书中的某一个特定位置的点。简单点说widget作为一种描述是可以复用的，但是element却跟需要绘制的节点一一对应。那element是最终渲染的view么？抱歉，还不是。element绘制时会转化成rendObject。RendObject才是真正经过layout和paint并绘制在屏幕上的对象。在flutter中有三套渲染相关的tree，分别是：widget tree， element tree & rendObject tree。三者的渲染流程如下：

![](https://gw.alicdn.com/tfs/TB1E4ltm26TBKNjSZJiXXbKVFXa-1638-940.png#width=)

那可能有人会问，为什么需要增增加中间这层的Element tree？<br />![](https://cdn.yuque.com/lark/0/2018/png/30196/1531404370708-5838a6c4-0c92-4210-9771-36d1ee2d0f83.png#width=)<br />flutter是响应式的框架。在某一时刻页面的布局，可能受不同的输入源的影响。Element这层实际上做了对某一时刻事件的汇总，在将真正需要修改的部分同步到真实的rendObject tree上。这么做有两个好处：

- 1.不需要直接操作UI，改为通过数据驱动视图。代码表达可以更加精炼。

- 2.最大层度降低对最终真实视图(rendObject tree)的修改，提高页面渲染效率。


<a name="nfi7tm"></a>
## [](#nfi7tm)5.4 StatelessWidget 和 StatefulWidget的区别
StatelessWidget是状态不可变的widget。初始状态设置以后就不可再变化。如果需要变化需要重新创建。StatefulWidget可以保存自己的状态。那问题是既然widget都是immutable的，怎么保存状态？其实Flutter是通过引入了State来保存状态。当State的状态改变时，能重新构建本节点以及孩子的Widget树来进行UI变化。注意：如果需要主动改变State的状态，需要通过setState()方法进行触发，单纯改变数据是不会引发UI改变的。

<a name="rlq4hz"></a>
# [](#rlq4hz)6.联系我们

本文详细解释了基础组件的用法，也解答了一些初学者的疑惑。希望能给刚踏上flutter学习之路的人一些帮助。如果对文本的内容有疑问或指正，欢迎告知我们。

闲鱼技术团队是一只短小精悍的工程技术团队。我们不仅关注于业务问题的有效解决，同时我们在推动打破技术栈分工限制（android/iOS/Html5/Server 编程模型和语言的统一）、计算机视觉技术在移动终端上的前沿实践工作。作为闲鱼技术团队的软件工程师，您有机会去展示您所有的才能和勇气，在整个产品的演进和用户问题解决中证明技术发展是改变生活方式的动力。

<a name="icsmgm"></a>
# [](#icsmgm)7.引用：

- 1.[https://flutter.io/widgets-intro/](https://flutter.io/widgets-intro/)

- 2.[https://flutter.io/technical-overview/](https://flutter.io/technical-overview/)

- 3.[css 盒子模型简介](http://www.runoob.com/css/css-boxmodel.html)

- 4.[flutter Layout Widgets目录](https://flutter.io/widgets/layout/)

- 5.[flex布局基础](http://www.runoob.com/w3cnote/flex-grammar.html)

- 6.[A Visual Guide to CSS3 Flexbox Properties](https://scotch.io/tutorials/a-visual-guide-to-css3-flexbox-properties)

- 7.[为什么说Flutter是革命性的？](https://www.sohu.com/a/192998605_635110)

- 8.[深入了解Flutter界面开发](https://mp.weixin.qq.com/s/z2r2OmnY7r7dQrkO8ndkFQ)


