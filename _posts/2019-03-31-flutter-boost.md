---
layout: post
title: "Flutter 混合开发 —— FlutterBoost"
subtitle: "FlutterBoost is a Flutter plugin"
author: "闲鱼技术-福居"
header-img: "img/post-bg-android.jpg"
header-mask: 0.4
tags:
  - Android
  - Flutter
  - FlutterBoost
---

> A next-generation Flutter-Native hybrid solution. FlutterBoost is a Flutter plugin which enables hybrid integration of Flutter for your existing native apps with minimum efforts.The philosophy of FlutterBoost is to use Flutter as easy as using a WebView. Managing Native pages and Flutter pages at the same time is non-trivial in an existing App. FlutterBoost takes care of page resolution for you. The only thing you need to care about is the name of the page(usually could be an URL). 


作者：闲鱼技术-福居
```
开源地址: https://github.com/alibaba/flutter_boost
```

<a name="d2a3dd8c"></a>
## 为什么需要混合方案

具有一定规模的App通常有一套成熟通用的基础库，尤其是阿里系App，一般需要依赖很多体系内的基础库。那么使用Flutter重新从头开发App的成本和风险都较高。所以在Native App进行渐进式迁移是Flutter技术在现有Native App进行应用的稳健型方式。闲鱼在实践中沉淀出一套自己的混合技术方案。在此过程中，我们跟Google Flutter团队进行着密切的沟通，听取了官方的一些建议，同时也针对我们业务具体情况进行方案的选型以及具体的实现。

<a name="df18d6fe"></a>
## 官方提出的混合方案

<a name="1e079232"></a>
### 基本原理

Flutter技术链主要由C++实现的Flutter Engine和Dart实现的Framework组成（其配套的编译和构建工具我们这里不参与讨论）。Flutter Engine负责线程管理，Dart VM状态管理和Dart代码加载等工作。而Dart代码所实现的Framework则是业务接触到的主要API，诸如Widget等概念就是在Dart层面Framework内容。

一个进程里面最多只会初始化一个Dart VM。然而一个进程可以有多个Flutter Engine，多个Engine实例共享同一个Dart VM。

我们来看具体实现，在iOS上面每初始化一个FlutterViewController就会有一个引擎随之初始化，也就意味着会有新的线程（理论上线程可以复用）去跑Dart代码。Android类似的Activity也会有类似的效果。如果你启动多个引擎实例，注意此时Dart VM依然是共享的，只是不同Engine实例加载的代码跑在各自独立的Isolate。

<a name="7bd2a77e"></a>
### 官方建议

<a name="d602a26e"></a>
#### 引擎深度共享

在混合方案方面，我们跟Google讨论了可能的一些方案。Flutter官方给出的建议是从长期来看，我们应该支持在同一个引擎支持多窗口绘制的能力，至少在逻辑上做到FlutterViewController是共享同一个引擎的资源的。换句话说，我们希望所有绘制窗口共享同一个主Isolate。

但官方给出的长期建议目前来说没有很好的支持。

<a name="9d50d214"></a>
#### 多引擎模式

我们在混合方案中解决的主要问题是如何去处理交替出现的Flutter和Native页面。Google工程师给出了一个Keep It Simple的方案：对于连续的Flutter页面（Widget）只需要在当前FlutterViewController打开即可，对于间隔的Flutter页面我们初始化新的引擎。

例如，我们进行下面一组导航操作：

```
Flutter Page1 -> Flutter Page2 -> Native Page1 -> Flutter Page3
```

我们只需要在Flutter Page1和Flutter Page3创建不同的Flutter实例即可。

这个方案的好处就是简单易懂，逻辑清晰，但是也有潜在的问题。如果一个Native页面一个Flutter<br />页面一直交替进行的话，Flutter Engine的数量会线性增加，而Flutter Engine本身是一个比较重的对象。

<a name="03a6f9d9"></a>
#### 多引擎模式的问题

* 冗余的资源问题.多引擎模式下每个引擎之间的Isolate是相互独立的。在逻辑上这并没有什么坏处，但是引擎底层其实是维护了图片缓存等比较消耗内存的对象。想象一下，每个引擎都维护自己一份图片缓存，内存压力将会非常大。
* 插件注册的问题。插件依赖Messenger去传递消息，而目前Messenger是由FlutterViewController（Activity）去实现的。如果你有多个FlutterViewController，插件的注册和通信将会变得混乱难以维护，消息的传递的源头和目标也变得不可控。
* Flutter Widget和Native的页面差异化问题。Flutter的页面是Widget，Native的页面是VC。逻辑上来说我们希望消除Flutter页面与Naitve页面的差异，否则在进行页面埋点和其它一些统一操作的时候都会遇到额外的复杂度。
* 增加页面之间通信的复杂度。如果所有Dart代码都运行在同一个引擎实例，它们共享一个Isolate，可以用统一的编程框架进行Widget之间的通信，多引擎实例也让这件事情更加复杂。

因此，综合多方面考虑，我们没有采用多引擎混合方案。

<a name="0446c8e4"></a>
## 现状与思考

前面我们提到多引擎存在一些实际问题，所以闲鱼目前采用的混合方案是共享同一个引擎的方案。这个方案基于这样一个事实：任何时候我们最多只能看到一个页面，当然有些特定的场景你可以看到多个ViewController，但是这些特殊场景我们这里不讨论。

我们可以这样简单去理解这个方案：我们把共享的Flutter View当成一个画布，然后用一个Native的容器作为逻辑的页面。每次在打开一个容器的时候我们通过通信机制通知Flutter View绘制成当前的逻辑页面，然后将Flutter View放到当前容器里面。

老方案在Dart侧维护了一个Navigator栈的结构。栈数据结构特点就是每次只能从栈顶去操作页面，每一次在查找逻辑页面的时候如果发现页面不在栈顶那么需要往回Pop。这样中途Pop掉的页面状态就丢失了。这个方案无法支持同时存在多个平级逻辑页面的情况，因为你在页面切换的时候必须从栈顶去操作，无法再保持状态的同时进行平级切换。

举个例子：有两个页面A，B，当前B在栈顶。切换到A需要把B从栈顶Pop出去，此时B的状态丢失，如果想切回B，我们只能重新打开B之前页面的状态无法维持住。这也是老方案最大的一个局限。

如在pop的过程当中，可能会把Flutter 官方的Dialog进行误杀。这也是一个问题。

而且基于栈的操作我们依赖对Flutter框架的一个属性修改，这让这个方案具有了侵入性的特点。这也是我们需要解决的一个问题。

![](https://cdn.nlark.com/yuque/0/2019/png/123320/1552968436247-d1dbeb9f-cc09-4644-9a2b-f3f4448af945.png#align=left&display=inline&height=368&originHeight=368&originWidth=741&size=0&status=done&width=741)

具体细节，大家可以参考老方案开源项目地址：

[https://github.com/alibaba-flutter/hybrid_stack_manager](https://github.com/alibaba-flutter/hybrid_stack_manager)

<a name="ae4f45b2"></a>
## 新一代混合技术方案 FlutterBoost

<a name="aa162336"></a>
### 重构计划

在闲鱼推进Flutter化过程当中，更加复杂的页面场景逐渐暴露了老方案的局限性和一些问题。所以我们启动了代号FlutterBoost（向C++ Boost致敬）的新混合技术方案。这次新的混合方案我们的主要目标有：

* 可复用通用型混合方案
* 支持更加复杂的混合模式。比如支持主页Tab这种情况
* 无侵入性方案：不再依赖修改Flutter的方案
* 支持通用页面生命周期
* 统一明确的设计概念

跟老方案类似，新的方案还是采用共享引擎的模式实现。主要思路是由Native容器Container通过消息驱动Flutter页面容器Container，从而达到Native Container与Flutter Container的同步目的。我们希望做到Flutter渲染的内容是由Naitve容器去驱动的。

简单的理解，我们想做到把Flutter容器做成浏览器的感觉。填写一个页面地址，然后由容器去管理页面的绘制。在Native侧我们只需要关心如果初始化容器，然后设置容器对应的页面标志即可。

<a name="d8fa8aef"></a>
### 主要概念

![](https://cdn.nlark.com/yuque/0/2019/png/123320/1552968436255-e781d85b-cc08-4dad-8267-a4bb94c7229c.png#align=left&display=inline&height=451&originHeight=623&originWidth=1031&size=0&status=done&width=746)

<a name="db232311"></a>
#### Native层概念

* Container：Native容器，平台Controller，Activity，ViewController
* Container Manager：容器的管理者
* Adaptor：Flutter是适配层
* Messaging：基于Channel的消息通信

<a name="121b24b3"></a>
#### Dart层概念

* Container：Flutter用来容纳Widget的容器，具体实现为Navigator的派生类-
* Container Manager：Flutter容器的管理，提供show，remove等Api
* Coordinator: 协调器，接受Messaging消息，负责调用Container Manager的状态管理。
* Messaging：基于Channel的消息通信

<a name="fbbefb32"></a>
#### 关于页面的理解

在Native和Flutter表示页面的对象和概念是不一致的。在Native，我们对于页面的概念一般是ViewController，Activity。而对于Flutter我们对于页面的概念是Widget。我们希望可统一页面的概念，或者说弱化抽象掉Flutter本身的Widget对应的页面概念。换句话说，当一个Native的页面容器存在的时候，FlutteBoost保证一定会有一个Widget作为容器的内容。所以我们在理解和进行路由操作的时候都应该以Native的容器为准，Flutter Widget依赖于Native页面容器的状态。

那么在FlutterBoost的概念里说到页面的时候，我们指的是Native容器和它所附属的Widget。所有页面路由操作，打开或者关闭页面，实际上都是对Native页面容器的直接操作。无论路由请求来自何方，最终都会转发给Native去实现路由操作。这也是接入FlutterBoost的时候需要实现Platform协议的原因。

另一方面，我们无法控制业务代码通过Flutter本身的Navigator去push新的Widget。对于业务不通过FlutterBoost而直接使用Navigator操作Widget的情况，包括Dialog这种非全屏Widget，我们建议是业务自己负责管理其状态。这种类型Widget不属于FlutterBoost所定义的页面概念。

理解这里的页面概念，对于理解和使用FlutterBoost至关重要。

<a name="2f65d4d2"></a>
### 与老方案主要差别

前面我们提到老方案在Dart层维护单个Navigator栈结构用于Widget的切换。而新的方案则是在Dart侧引入了Container的概念，不再用栈的结构去维护现有的页面，而是通过扁平化key-value映射的形式去维护当前所有的页面，每个页面拥有一个唯一的id。这种结构很自然的支持了页面的查找和切换，不再受制于栈顶操作的问题，之前的一些由于pop导致的问题迎刃而解。同时也不再需要依赖修改Flutter源码的形式去进行实现，除去了实现的侵入性。

那这是如何做到的呢？

<a name="26c7eeef"></a>
#### 多Navigator的实现

Flutter在底层提供了让你自定义Navigator的接口，我们自己实现了一个管理多个Navigator的对象。当前最多只会有一个可见的Flutter Navigator，这个Navigator所包含的页面也就是我们当前可见容器所对应的页面。

Native容器与Flutter容器（Navigator）是一一对应的，生命周期也是同步的。当一个Native容器被创建的时候，Flutter的一个容器也被创建，它们通过相同的id关联起来。当Native的容器被销毁的时候，Flutter的容器也被销毁。Flutter容器的状态是跟随Native容器，这也就是我们说的Native驱动。由Manager统一管理切换当前在屏幕上展示的容器。

我们用一个简单的例子描述一个新页面创建的过程：

1. 创建Native容器（iOS ViewController，Android Activity or Fragment）。
1. Native容器通过消息机制通知Flutter Coordinator新的容器被创建。
1. Flutter Container Manager进而得到通知，负责创建出对应的Flutter容器，并且在其中装载对应的Widget页面。
1. 当Native容器展示到屏幕上时，容器发消息给Flutter Coordinator通知要展示页面的id.
1. Flutter Container Manager找到对应id的Flutter Container并将其设置为前台可见容器。

这就是一个新页面创建的主要逻辑，销毁和进入后台等操作也类似有Native容器事件去进行驱动。

<a name="25f9c7fa"></a>
## 总结

目前FlutterBoost已经在生产环境支撑着在闲鱼客户端中所有的基于Flutter开发业务，为更加负复杂的混合场景提供了支持。同时也解决了一些历史遗留问题。

我们在项目启动之初就希望FlutterBoost能够解决Native App混合模式接入Flutter这个通用问题。所以我们把它做成了一个可复用的Flutter插件，希望吸引更多感兴趣的朋友参与到Flutter社区的建设。我们的方案可能不是最好的，这个方案距离完美还有很大的距离，我们希望通过多分享交流以推动Flutter技术社区的发展与建设。我们更希望看到社区能够涌现出更加优秀的组件和方案。

在有限篇幅中，我们分享了闲鱼在Flutter混合技术方案中积累的经验和代码。欢迎兴趣的同学能够积极与我们一起交流学习。

<a name="beb431c0"></a>
## 扩展补充

<a name="42202600"></a>
### 性能相关

在两个Flutter页面进行切换的时候，因为我们只有一个Flutter View所以需要对上一个页面进行截图保存，如果Flutter页面多截图会占用大量内存。这里我们采用文件内存二级缓存策略，在内存中最多只保存2-3个截图，其余的写入文件按需加载。这样我们可以在保证用户体验的同时在内存方面也保持一个较为稳定的水平。

页面渲染性能方面，Flutter的AOT优势展露无遗。在页面快速切换的时候，Flutter能够很灵敏的相应页面的切换，在逻辑上创造出一种Flutter多个页面的感觉。

<a name="18a5e27f"></a>
### Release 1.0支持

项目开始的时候我们基于闲鱼目前使用的Flutter版本进行开发，而后进行了Release 1.0兼容升级测试目前没有发现问题。

<a name="ca1fc273"></a>
### 接入

只要是集成了Flutter的项目都可以用官方依赖的方式非常方便的以插件形式引入FlutterBoost，只需要对工程进行少量代码接入即可完成接入。<br />详细接入文档，请参阅GitHub主页官方项目文档。

<a name="cf999933"></a>
### 现已开源

目前，新一代混合栈已经在闲鱼全面应用。我们非常乐意将沉淀的技术回馈给社区。欢迎大家一起贡献，一起交流，携手共建Flutter社区。

项目开源地址：[https://github.com/alibaba/flutter_boost](https://github.com/alibaba/flutter_boost)<br />

# FlutterBoost 的使用

新一代Flutter-Native混合解决方案。 FlutterBoost是一个Flutter插件，它可以轻松地为现有原生应用程序提供Flutter混合集成方案。FlutterBoost的理念是将Flutter像Webview那样来使用。在现有应用程序中同时管理Native页面和Flutter页面并非易事。 FlutterBoost帮你处理页面的映射和跳转，你只需关心页面的名字和参数即可（通常可以是URL）。


# 前置条件
在继续之前，您需要将Flutter集成到你现有的项目中。

# 安装

## 在Flutter项目中添加依赖项。

打开pubspec.yaml并将以下行添加到依赖项：

Release 1.0 之前的版本

```json
flutter_boost: ^0.0.400
```

## Dart代码的集成
将init代码添加到App App

```dart
void main() => runApp(MyApp());

class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  @override
  void initState() {
    super.initState();

    ///register page widget builders,the key is pageName
    FlutterBoost.singleton.registerPageBuilders({
      'sample://firstPage': (pageName, params, _) => FirstRouteWidget(),
      'sample://secondPage': (pageName, params, _) => SecondRouteWidget(),
    });

    ///query current top page and load it
    FlutterBoost.handleOnStartPage();
  }

  @override
  Widget build(BuildContext context) => MaterialApp(
      title: 'Flutter Boost example',
      builder: FlutterBoost.init(), ///init container manager
      home: Container());
}
```

## iOS代码集成。

使用FLBFlutterAppDelegate作为AppDelegate的超类

```objectivec
@interface AppDelegate : FLBFlutterAppDelegate <UIApplicationDelegate>
@end
```


为您的应用程序实现FLBPlatform协议方法。

```objectivec
@interface DemoRouter : NSObject<FLBPlatform>

@property (nonatomic,strong) UINavigationController *navigationController;

+ (DemoRouter *)sharedRouter;

@end


@implementation DemoRouter

- (void)openPage:(NSString *)name
          params:(NSDictionary *)params
        animated:(BOOL)animated
      completion:(void (^)(BOOL))completion
{
    if([params[@"present"] boolValue]){
        FLBFlutterViewContainer *vc = FLBFlutterViewContainer.new;
        [vc setName:name params:params];
        [self.navigationController presentViewController:vc animated:animated completion:^{}];
    }else{
        FLBFlutterViewContainer *vc = FLBFlutterViewContainer.new;
        [vc setName:name params:params];
        [self.navigationController pushViewController:vc animated:animated];
    }
}


- (void)closePage:(NSString *)uid animated:(BOOL)animated params:(NSDictionary *)params completion:(void (^)(BOOL))completion
{
    FLBFlutterViewContainer *vc = (id)self.navigationController.presentedViewController;
    if([vc isKindOfClass:FLBFlutterViewContainer.class] && [vc.uniqueIDString isEqual: uid]){
        [vc dismissViewControllerAnimated:animated completion:^{}];
    }else{
        [self.navigationController popViewControllerAnimated:animated];
    }
}

@end
```



在应用程序开头使用FLBPlatform初始化FlutterBoost。

```的ObjectiveC
 [FlutterBoostPlugin.sharedInstance startFlutterWithPlatform：router
                                                        onStart：^（FlutterViewController * fvc）{
                                                            
                                                        }];
```

## Android代码集成。

在Application.onCreate（）中初始化FlutterBoost

```java
public class MyApplication extends FlutterApplication {
    @Override
    public void onCreate() {
        super.onCreate();
        FlutterBoostPlugin.init(new IPlatform() {
            @Override
            public Application getApplication() {
                return MyApplication.this;
            }

            /**
             * get the main activity, this activity should always at the bottom of task stack.
             */
            @Override
            public Activity getMainActivity() {
                return MainActivity.sRef.get();
            }

            @Override
            public boolean isDebug() {
                return false;
            }

            /**
             * start a new activity from flutter page, you may need a activity router.
             */
            @Override
            public boolean startActivity(Context context, String url, int requestCode) {
                return PageRouter.openPageByUrl(context,url,requestCode);
            }

            @Override
            public Map getSettings() {
                return null;
            }
        });
    }
```

# 基本用法
## 概念

所有页面路由请求都将发送到Native路由器。Native路由器与Native Container Manager通信，Native Container Manager负责构建和销毁Native Containers。

## 使用Flutter Boost Native Container用Native代码打开Flutter页面。

```objc
 FLBFlutterViewContainer *vc = FLBFlutterViewContainer.new;
        [vc setName:name params:params];
        [self.navigationController presentViewController:vc animated:animated completion:^{}];
```

Android

```java
public class FlutterPageActivity extends BoostFlutterActivity {

    @Override
    public void onRegisterPlugins(PluginRegistry registry) {
        //register flutter plugins
        GeneratedPluginRegistrant.registerWith(registry);
    }

    @Override
    public String getContainerName() {
        //specify the page name register in FlutterBoost
        return "sample://firstPage";
    }

    @Override
    public Map getContainerParams() {
        //params of the page
        Map<String,String> params = new HashMap<>();
        params.put("key","value");
        return params;
    }
}
```

或者用Fragment

```java
public class FlutterFragment extends BoostFlutterFragment {
    @Override
    public void onRegisterPlugins(PluginRegistry registry) {
        GeneratedPluginRegistrant.registerWith(registry);
    }

    @Override
    public String getContainerName() {
        return "sample://firstPage";
    }

    @Override
    public Map getContainerParams() {
        Map<String,String> params = new HashMap<>();
        params.put("key","value");
        return params;
    }
}
```


## 使用Flutter Boost在dart代码打开页面。
Dart

```java
 FlutterBoost.singleton.openPage("pagename", {}, true);
```


## 使用Flutter Boost在dart代码关闭页面。

```java
FlutterBoost.singleton.closePageForContext(context);
```

# Examples
更详细的使用例子请参考Demo


# 作者
阿里巴巴闲鱼终端团队

# 许可证
该项目根据MIT许可证授权 - 有关详细信息，请参阅[LICENSE.md]（LICENSE.md）文件
<a name="Acknowledgments"> </a>
# 致谢
- Flutter