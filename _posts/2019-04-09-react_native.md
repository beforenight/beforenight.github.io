---
layout: post
title: "使用JavaScript和React编写原生移动应用（1）"
subtitle: "Build native mobile apps using JavaScript and React"
author: "beforenight"
header-img: "img/post-react-native.png"
header-mask: 0.4
tags:
  - React
  - JavaScript
  - Android
---
## 使用JavaScript和React编写原生移动应用
> React Native使你只使用JavaScript也能编写原生移动应用。 它在设计原理上和React一致，通过声明式的组件机制来搭建丰富多彩的用户界面。

```
import React, { Component } from 'react';
import { Text, View } from 'react-native';

class WhyReactNativeIsSoGreat extends Component {
  render() {
    return (
      <View>
        <Text>
          如果你喜欢在Web上使用React，那你也肯定会喜欢React Native.
        </Text>
        <Text>
          基本上就是用原生组件比如'View'和'Text'
          来代替web组件'div'和'span'。
        </Text>
      </View>
    );
  }
}

```
## React Native应用是真正的移动应用

> React Native产出的并不是“网页应用”， 或者说“HTML5应用”，又或者“混合应用”。 最终产品是一个真正的移动应用，从使用感受上和用Objective-C或Java编写的应用相比几乎是无法区分的。 React Native所使用的基础UI组件和原生应用完全一致。 你要做的就是把这些基础组件使用JavaScript和React的方式组合起来。

```
import React, { Component } from 'react';
import { Image, ScrollView, Text } from 'react-native';

class AwkwardScrollingImageWithText extends Component {
  render() {
    return (
      <ScrollView>
        <Image
          source={{uri: 'https://i.chzbgr.com/full/7345954048/h7E2C65F9/'}}
          style={{width: 320, height:180}}
        />
        <Text>
          在iOS上，React Native的ScrollView组件封装的是原生的UIScrollView。
          在Android上，封装的则是原生的ScrollView。

          在iOS上，React Native的Image组件封装的是原生的UIImageView。
          在Android上，封装的则是原生的ImageView。

          React Native封装了这些基础的原生组件，使你在得到媲美原生应用性能的同时，还能受益于React优雅的架构设计。 
        </Text>
      </ScrollView>
    );
  }
}
```


## 环境搭建

[React Native 中文文档社区](!https://reactnative.cn/docs/getting-started.html)

> Follow these instructions if you need to build native code in your project. For example, if you are integrating React Native into an existing application, or if you "ejected" from Create React Native App, you'll need this section.

环境：Macos(其他环境请参考上面的中文文档地址)
### 安装依赖
必须安装的依赖有：Node、Watchman 和 React Native 命令行工具以及 Xcode。

虽然你可以使用*任何编辑器*来开发应用（编写 js 代码），但你仍然必须安装 Xcode 来获得编译 iOS 应用所需的工具和环境。

### Node, Watchman
我们推荐使用Homebrew来安装 Node 和 Watchman。在命令行中执行下列命令安装：
```
brew install node
brew install watchman
```

如果你已经安装了 Node，请检查其版本是否在 v8.3 以上。安装完 Node 后建议设置 npm 镜像以加速后面的过程（或使用科学上网工具）。

> 注意：不要使用 cnpm！cnpm 安装的模块路径比较奇怪，packager 不能正常识别！

```
pm config set registry https://registry.npm.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global
```
[Watchman](!https://facebook.github.io/watchman)
则是由 Facebook 提供的监视文件系统变更的工具。安装此工具可以提高开发时的性能（packager 可以快速捕捉文件的变化从而实现实时刷新）。

### Yarn、React Native 的命令行工具（react-native-cli）
[Yarn](!http://yarnpkg.com/) 是 Facebook 提供的替代 npm 的工具，可以加速 node 模块的下载。React Native 的命令行工具用于执行创建、初始化、更新项目、运行打包服务（packager）等任务。
```
npm install -g yarn react-native-cli
```

安装完 yarn 后同理也要设置镜像源：
```
yarn config set registry https://registry.npm.taobao.org --global
yarn config set disturl https://npm.taobao.org/dist --global
```
安装完 yarn 之后就可以用 yarn 代替 npm 了，例如用yarn代替npm install命令，用yarn add 某第三方库名代替**npm install 某第三方库名**。

## Xcode
React Native 目前需要Xcode 9.4 或更高版本。你可以通过 App Store 或是到[Apple 开发者官网](!https://developer.apple.com/xcode/downloads/)上下载。这一步骤会同时安装 Xcode IDE、Xcode 的命令行工具和 iOS 模拟器。

### Xcode 的命令行工具
启动 Xcode，并在**Xcode | Preferences | Locations**菜单中检查一下是否装有某个版本的Command Line Tools。Xcode 的命令行工具中包含一些必须的工具，比如**git**等。

![Xcode](https://reactnative.cn/docs/assets/GettingStartedXcodeCommandLineTools.png)

## 创建新项目
使用 React Native 命令行工具来创建一个名为"AwesomeProject"的新项目：

> ！！！注意！！！：init 命令默认会创建最新的版本，而目前最新的 0.45 及以上版本需要下载 boost 等几个第三方库编译。这些库在国内即便翻墙也很难下载成功，导致很多人无法运行iOS项目！！！中文网在论坛中提供了这些库的国内下载链接。如果你嫌麻烦，又没有对新版本的需求，那么可以暂时创建0.44.3的版本。
```
react-native init AwesomeProject
```
提示：你可以使用--version参数（注意是两个杠）创建指定版本的项目。例如react-native init MyApp --version 0.44.3。注意版本号必须精确到两个小数点。

如果你是想把**React Native***集成到现有的原生项目中，则步骤完全不同，请参考集成到现有原生应用。

## 编译并运行 React Native 应用
在你的项目目录中运行**react-native run-ios**：
```
cd AwesomeProject
react-native run-ios
```

提示：如果 run-ios 无法正常运行，请使用 Xcode 运行来查看具体错误（run-ios 的报错没有任何具体信息）。

很快就应该能看到 iOS 模拟器自动启动并运行你的项目。

![ios-app](https://reactnative.cn/docs/assets/GettingStartediOSSuccess.png)

react-native run-ios只是运行应用的方式之一。你也可以在 Xcode 或是Nuclide中直接运行应用。

## 在真机上运行
上面的命令会自动在 iOS 模拟器上运行应用，如果你想在真机上运行，则请阅读在设备上运行这篇文档。


## 修改项目
现在你已经成功运行了项目，我们可以开始尝试动手改一改了：

- 使用你喜欢的编辑器打开App.js并随便改上几行。
- 在 iOS 模拟器中按下 **⌘-R** 就可以刷新 APP 并看到你的最新修改！（如果没有反应，请检查模拟器的 Hardware 菜单中，**connect hardware keyboard** 选项是否选中开启）

## 完成了！
恭喜！你已经成功运行并修改了你的第一个 React Native 应用。
![success-app](https://reactnative.cn/docs/assets/GettingStartedCongratulations.png)