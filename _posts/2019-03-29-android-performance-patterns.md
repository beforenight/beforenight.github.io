---
layout: post
title: "Android性能优化典范(第一篇)"
subtitle: "android-performance-patterns"
author: "beforenight"
header-img: "img/post-bg-alitrip.jpg"
header-mask: 0.4
tags:
  - Android
  - Performance
  - Google
---

> Android 设备作为一种移动设备，不管是内存还是 CPU 的性能都受到了一定的限制。无法做到像 PC 设备那样具有超大内存和高性能的 CPU。鉴于这一点，这也意味着 Android 程序不可能无限制地使用内存和 CPU 资源，过多的使用内存会导致程序内存溢出，即 OOM。而过多地使用 CPU 资源，一般是指大量的耗时任务，会导致手机变得卡顿甚至出现程序无法响应的情况，即 ANR。因此，Android 程序的性能问题就变得异常突出了，这对于开发人员提出了更高地要求。为了提高应用程序的性能，这里介绍一些有效的性能优化方法，主要内容包括布局优化、绘制优化、内存泄漏/检测优化、响应速度优化、ListView 优化、Bitmap 优化、线程优化以及一些其它优化建议等。

---
性能优化中一个很重要的问题就是内存泄漏，内存泄漏并不一定会导致程序功能异常，但是它会导致 Android 程序的内存占用过大，这将提高内存溢出的发生几率。如何避免写出内存泄漏的代码，这和开发人员的水平和意识有很大关系，甚至很多情况下内存泄漏的原因是很难发现的，这时就需要借助一些内存泄漏分析工具，对内存泄漏进行检测和分析，如 内存泄漏分析工具——MAT。

在做程序设计时，除了要完成功能开发、提高程序性能以外，还有一个问题也是不可忽视的，就是代码的可维护性和可扩展性。如果一个程序的可维护性和可扩展性很差，那就意味着后续代码维护代价是相当高的，比如需要对一个功能进行调整，这可能会出现牵一发而动全身的尴尬局面。

关于代码的可维护性和可扩展性，看起来是一个很抽象的问题，其实它并不抽象，它是可以通过一些合理的设计原则去完成的，比如良好的代码风格、清晰地代码层级、代码的可扩展性和合理的使用设计模式，这将在一定程度上提高程序的可维护性和可扩展性。

## Android 性能优化的方法

- 布局优化
- 绘制优化
- 内存泄漏/检测优化
- 响应速度优化
- ListView 优化
- Bitmap 优化
- 线程优化

#### 1.布局优化
> 布局优化的思想主要是：尽量减少布局文件的层级，避免过度绘制。

如何进行布局优化呢？ 首先删除布局中无用的控件和层级，其次有选择地使用性能比较低的 ViewGroup，比如 RelativeLayout；如果布局中使用 LinearLayout 也可以使用 RelativeLayout ，那么就采用 LinearLayout ，这是因为 RelativeLayout 的功能比较复杂，它的布局过程需要花费更多的 CPU 时间。FrameLayout 和 LinearLayout 一样都是一种简单高效的 ViewGroup ，因此在不增加层级的情况下，可以考虑使用它们。
布局优化的另一种方式是采用 <include> 标签 、 <merge> 标签和ViewStub。<include> 标签主要用于布局重用，<merge> 标签一般和<include>配合使用，它可以降低布局的层级，而 ViewStub 则提供了按需加载功能，当需要时才会将 ViewStub 中的布局加载到内存，这提高了程序的初始化效率。

#### 2.绘制优化
> 绘制优化是指 View 的 onDraw 方法要避免执行大量的操作。
这主要体现在两个方面，首先，onDraw 中不要创建新的局部对象，这是因为 onDraw 方法很可能会被频繁调用，这样会在一瞬间产生大量的临时对象，这不仅占用了过多的内存而且还会导致系统更加频繁的 gc,降低了程序
的执行效率。

另一方面，onDraw 方法中不要做耗时的任务，也不能执行成千上万的循环操作，尽管每次循环都很轻量，但是大量的循环仍然十分抢占 CPU 的时间片，这会造成 View 的绘制过程不流畅。按照Google给的性能优化典范标准：View 的绘制帧率保证 60fps 是最佳的，这就要求每帧的绘制时间不超过 16ms(16ms = 1000/60)，虽然程序很难保证 16ms 这个时间，但尽量降低 onDraw 方法的复杂度总是切实有效的。

#### 3.内存泄漏/检测优化
内存泄漏的优化分为两个方面，一方面是在开发过程中尽量避免写出有内存泄漏的代码，另一方面是通过分析工具比如 MAT 来找出潜在的内存泄漏继而解决。

场景一：静态变量导致的内存泄漏；
场景二：单例模式导致的内存泄漏；
场景三：属性动画导致的内存泄漏；

#### 4.响应速度优化
> 响应速度优化的核心思想是避免在主线程中做耗时任务。
可以采用将这些耗时任务放在子线程中去执行，即采用异步的方式执行耗时操作。响应速度更多地体现在 Activity 的启动速度上，如果在主线程操作太多事情会导致 Activity 启动时出现黑屏现象，甚至出现 ANR。

Android 规定：
- 超过 5s —— Activity 无法响应屏幕触摸事件就会出现ANR；
- 超过 10s —— BroadCast 还未执行完操作就会出现ANR；

#### 5.ListView 优化
ListView 主要分为三个方面：
- 采用 ViewHolder 并且避免在 getView 中执行耗时操作；
- 要根据列表的滑动状态来控制任务的执行频率；
- 尝试开启硬件加速来提升 ListView 绘制效果，使 ListView 的滑动更加流畅；

#### 6.Bitmap 优化
Bitmap 主要有：
- 通过 BitmapFactory.Options 来根据需要对图片进行采样；
- 采样过程中主要用到了 BitmapFactory.Options 的 inSampleSize 参数

#### 7.线程优化
> 线程优化的思想是采样线程池，避免程序中频繁的创建和销毁 Thread & 存在大量的 Thread。
线程池可以重用内部的线程，从而避免了线程的创建和销毁带来的性能开销，同时线程池还能有效地控制线程池的最大并发数量，避免大量的线程因互相抢占系统资源从而导致阻塞现象的发生。因此开发过程中尽量采用线程池，而不是每一次都要创建一个 Thread 对象。

#### 8.一些性能优化的建议

- 避免创建过多的对象；
- 不要过多使用枚举，枚举占用的内存空间比整型大；
- 常量请使用 static  final 来修饰；
- 使用一些 Android 特有的数据结构，比如 SparseArray 和 Pair 等，它们都具有更好的性能；
- 适当使用软引用和弱引用；
- 尽量采用静态内部类，这样可以避免潜在的由于内部类而导致的内存泄漏。