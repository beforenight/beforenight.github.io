---
layout: post
title: "Android 的线程和线程池"
subtitle: "Thread and Thread pool in Android "
author: "beforenight"
header-img: "img/post-bg-alitrip.jpg"
header-mask: 0.4
tags:
  - Android
  - Thread
  - 线程
---
> 线程在 Android 中是一个很重要的概念，从用途上来说，线程分为主线程和子线程，主线程主要处理和界面相关的事情，而子线程则往往用于执行耗时任务。
除了 Thread 以外，在 Android 中可以扮演线程角色的还有很多，比如 AsyncTask 和 IntentService，同时 HandlerThread 也是一种特殊的线程。尽管 AsyncTask 、IntentService 以及 HnadlerThread 的表现都有别与传统的线程，但是它们的本质仍然是传统的线程。
对于 AsyncTask 来说，它的底层用到了线程池，对于IntentService 和 HandlerThread 来说，它们的底层则直接使用了线程。

不同的使用场景
- [ AsyncTask ] 封装了线程池和 Handler ，它主要是为了方便开发者在子线程中更新 UI。
- [ HandlerThread ] 是一个具有消息循环的线程，在它的内部可以使用 Handler。
- [ IntentService ] 是一个服务，系统对其进行封装使其可以更方便的执行后台任务，IntentService 内部采用HandlerThread 来执行任务，当任务执行完毕后会自动退出。而且，IntentService 是一种服务，不容易被系统杀死从而可以尽量保证任务的执行。

---
 

### 什么是线程？
在操作系统中，线程是操作系统调度的最小单元，同时线程又是一种受限的资源，即：线程不可能无限制的产生，并且线程的创建和销毁都会有相应的开销。
当系统中存在大量的线程时，系统会通过时间片轮转的方式调度每个线程，因此线程不可能做到绝对的并行，除非线程数量小于等于CPU的核心数，一般来说这是不可能的。就算是系统支持，在一个进程中频繁地创建和销毁线程，也会对系统资源造成极大浪费。正确地做法是采用线程池，一个线程池会缓存一定数量的线程，通过线程池就可以避免频繁创建和销毁线程所带来的系统开销。
在 Android 中的线程池来源于 Java，主要是通过 Executor 来派生特定类型的线程池，不同种类的线程池又具有各自的特征。

### 主线程和子线程

主进程是指进程所拥有的线程，在 Java 中默认情况下一个进程只有一个线程，这个线程就是主线程。主线程主要处理界面交互相关的逻辑，因为用户随时会和界面发生交互，因此主线程在任何时候都必须有较高的响应速度，否则就会产生一种界面卡顿的感觉。为了保持较高的响应速度，这就要求主线程中不能执行耗时任务，这个时候子线程就派上用场了。
子线程也叫工作线程，除了主线程以外的线程都是子线程。
Android 沿用了 Java 的线程模型，其中的线程也分为主线程和子线程，其中主线程也叫 UI 线程。主线程的作用是运行四大组件以及处理它们和用户的交互，而子线程的作用则是耗时任务，比如网络请求、I/O操作等。从 Android 3.0 开始系统要求网络访问必须在子线程中进行，否则网访问将会失败并抛出 NewworkOnMainThreadException 这个异常，这样做是为了避免主线程由于被耗时操作所阻塞而出现 ANR 现象。

### Android 中的线程形态
- AsyncTask：是一种轻量级的异步任务类，它可以在线程池中执行后台任务，然后把执行进度和最终结果传递到主线程并在主线程中更新 UI。从实现上来说，AsyncTask 封装了 Thread 和 Handler，通过 AsyncTask 可以更加方便地执行后台任务以及在主线程中访问 UI，但是 AsyncTask 并不适合进行特别耗时的后台任务，对于特别耗时的任务来说，建议使用线程池。
- HandlerThread：HandlerThread 继承了 Thread，它是一种可以使用Handler 的Thread，它的实现也很简单，就是在 run 方法中通过 Looper.prepare() 来创建消息队列，并通过 Looper.loop() 来开启消息循环，这样在实际使用中就允许在 HandlerThread 中创建 Handler。从HandlerThread的实现来看，它和普通的Thread有显著的不同之处。普通Thread主要用于在run方法中执行一个耗时任务，而HandlerThread在内部创建了消息队列，外界需要通过Handler的消息方式来通知HandlerThread执行一个具体的任务。HandlerThread是一个很有用的类，它在Android中的一个具休的使用场景是IntentService。由于HandlerThread的run方法是一个无限循环，因此当明确不需要 再使用HandlerThread时，可以通过它的quit或者quitSafely方法来终止线程的执行，这是一个良好的编程习惯。
- IntentService是一种特殊的Service，它继承了Service并且它是一个抽象类，因此必须创建它的子类才能使用IntentService。IntentService可用于执行后台耗时的任务，当任务执行后它会自动停止，同时由于IntentService是服务的原因，这导致它的优先级比单纯的线程要高很多，所以IntentService比较适合执行一些高优先级的后台任务，因为它优先级髙不容场被系统杀死。

### Android 中的线程池
线程池的优点可以 概括为以下三点：
- (1)重用线程池中的线程，避免因为线程的创建和销毁所带来的性能开销。
- (2)能有效控制线程池的最大并发数，避免大量的线程之间因互相抢占系统资源而导致的阻塞现象。
- (3)能够对线程进行简单的管理，并提供定时执行以及指定间隔循环执行等功能。

Android中的线程池的概念来源于Java中的Executor,Executor是一个接口，真正的线程池的实现为ThreadPoolExecutor。ThreadPoolExecutor提供了一系列参数来配置线程池，通过不同的参数可以创建不同的线程池，从线程池的功能特性上来说，Android的线程池主要分为4类，这4类线程池可以通过Executors所提供的工厂方法来得到。

ThreadPoolExecutor：ThreadPoolExecutor是线程池的真正实现，它的构造方法提供了一系列参数来配置线程池：
```
public ThreadPoolExecutor(int corePoolSize,
    int maximumPoolSize,
    long keepAliveTime,
    TimeUnit unit,
    BlockingQueue<Runnable> workQueue,
    ThreadFactory threadFactory)
```

Thread Pool Executor执行任务时大致遵循如下规则：
- (1)如果线程池中的线程数最未达到核心线程的数量，那么会直接启动一个核心线程来执行任务。
- (2)如果线程池中的线程数量已经达到或者超过核心线程的数量，那么任务会被插入到任务队列中排队等待执行。
- (3)如果在步骤2中无法将任务插入到任务队列中，这往往是由于任务队列已满，这个时候如果线程数量未达到线程池规定的最大值，那么会立刻启动一个非核心线程来执行任务。
- (4)如果步骤3中线程数量已经达到线程池规定的最大值，那么就拒绝执行此任务，ThreadPoolExecutor 会调用 RejectedExecutionHandler 的 rejectedExecution 方法来通知调用者。

```
    private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    private static final int CORE_POOL—SIZE = CPU_COUNT + 1;
    private static final int MAXIMUM_POOL_SI2E = CPU_COUNT * 2十 1;
    private static final int KEEP一ALIVE = 1;
    private static final ThreadFactory sThreadFactory = new ThreadFactory();
    private final Atomiclnteger mCount = new Atomiclnteger(1>;
    public Thread. newThread (Runnable r) {
    return new Thread(r, "AsyncTask #" + mCount.getAndlncrement());
    }}
    private static final BlockingQueue<Runnable> sPoolWorkQueue = new LinkedBiockingQueue<Runnable>(128);
    * An {@link Executor} that can be used to execute tasks in parallel.*/
    public static final Executor THREAD_POOL—EXECUTOR = new ThreadPoolExecutor(CORE_POOL一SIZE, MAXIMUM_POOL_SIZE, KEEPALIVE, TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);
```
从上面的代码可以知道，AsyncTask对THREAD_POOL_EXECUTOR这个线程池进行了配置，配置后的线程池规格如下：
- 核心线程数等于CPU核心数+ 1;
- 线程池的最大线程数为CPU核心数的2倍+1:
- 核心线程无超时机制，非核心线程在闲背时的超时时间为1秒：
- 任务队列的容量为128。


### 线程池的分类
四类线程池分别是：
- FixedThreadPool
- CachedThreadPool
- ScheduledThreadPool
- SingleThreadExecutor


### FixedThreadPool
通过Executors的newFixedThreadPool方法来创建。它是一种线程数量固定的线程池，当线程处于空闲状态时，它们并不会被回收，除非线程池被关闭了。当所有的线程都处沪活动状态时，新任务都会处于等待状态，直到有线程空闲出来。由于HxedThreadPool只有核心线程并且这些核心线程不会被回收，这意味着它能够更加快速地响应外界的请求。newFixedThreadPool方法的实现如下，可以发现FixedThreadPool中只有核心线程并且这些核心线程没有超时机制，另外任务队列也是没有大小限制的。

### CachedThreadPool
通过Executors的newCachedThreadPool方法来创建。它是一种线程数M不定的线程池，它只有非核心线程，并且其最大线程数为Integer.MAX_VALUE。由于Integer.MAX_VALUE是一个很大的数，实际上就相当于最大线程数可以任意大。当线程池中的线程都处于活动状态时，线程池会创建新的线程来处理新任务，否则就会利用空闲的线程来处理新任务。线程池中的空闲线程都有超时机制，这个超时时长为60秒，超过60秒闲置线程就会被回收。和FixedThreadPool不同的是，CachedThreadPool的任务队列其实相当于一个空集合，这将导致任何任务都会立即被执行，因为在这种场景下SynchronousQueue是无法插入任务的。SynchronousQueue是一个非常特殊的队列，在很多情况下可以把它简单理解为一个无法存储元素的队列，由于它在实际中较少使用，这里就不深入探讨它了。从CachedThreadPool的特性来看，这类线程池比较适合执行大量的耗时较少的任务。当粮个线程池都处于闲置状态时，线程池中的线程都会超时而被停止，这个时候CachedThreadPool之中实际上是没有任何线程的，它几乎是不占用任何系统资源的。

### ScheduledThreadPool
通过Executors的newScheduledThreadPool方法来创建。它的核心线程数量是固定的，而非核心线程数是没有限制的，并且当非核心线程闲置时会被立即回收。ScheduledThreadPool这类线程池主要用于执行定时任务和具有固定周期的重复任务。

### SingleThreadExecutor
通过Executors的newSingleThreadExecutor方法来创建-这类线程池内部只有一个核心线程，它确保所有的任务都在同一个线程中按顺序执行。SingleThreadExecutor的意义在于统一所有的外界任务到一个线程中，这使得在这些任务之间不需要处理线程同步的问题。

示例代码：
```
Runnable command = new Runnable() {
  @Override
  public void run () {
  SystemClock.sleep(2000);)
  }
  ExecutorService fixedThreadPool = Executors.newFixedThreadPool(4);
  fixedThreadPool.execute(command);
  
  ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
  cachedThreadPool.execute(command);
  
  ScheduledExecutorService scheduledThreadPool = Executors.newSchedule.ThreadPool(4);// 2000ms 后执行 commandscheduledThreadPool.schedule(command, 2000, TimeUnit.MILLISECONDS);//延迟10ms后，每隔1000ms执行一次
  commandscheduledThreadPool.scheduleAtFixedRate(command, 10, 1000, TimeUnit.MILLISECONDS)；
  
  ExecutorService singleThreadExecutor = Executors.newSingleThread.Executor();
  singleThreadExecutor.execute(command);
```