# 从开发者视角聊聊 React native 架构演进

## 前言

之前在项目中用 react native 实现了一个 native app，过程中查阅了大量的资料，也配合 ai 扒了一下 RN 的源码

本文是通过之前的学习资料整理而成，希望能够回答三个问题：

1. RN 的设计初衷是什么？
2. RN 的初版架构在实现过程中做了哪些权衡？
3. RN 的新架构是如何填坑的？

## 设计初衷

> 本节资料来自 RN 团队的第一篇博客：https://engineering.fb.com/2015/03/26/android/react-native-bringing-modern-web-techniques-to-mobile/

2013 年 Facebook 的团队第一次发布了 React。React 的成功极大提升了研发的效率，使得 Facebook 网站得以两天发一次版本，并以极快的速度验证 A/B 实验的结果。

但是在原生 APP 的场景下，情况有些变化：在更丝滑的手势交互、更丰富的原生控件、多线程支持下带来更好用户体验的背后，是双倍的研发投入，更长的研发与发版周期

在更好的用户体验与更好的研发体验之间，React 团队选择了全部都要😂

在尝试了 Webview 以及 [componentkit](https://componentkit.org/) 的方案之后，React 团队最后回到了他们最熟悉的领域：Javascript

他们的想法就是：**使用 JS 调用原生的 APIs**（Scripting native）

### JS 应该跑在哪里？

这个想法特别美好，如果成功了既可以直接套用现有的 JS 基建，又可以通过 JS 调用所有原生的能力

紧随其后的第一个问题是：JS 应该跑在哪里？

最直接的想法是主线程（又称 Main thread，是 IOS 与 Android 系统启动应用时开启的第一个线程，负责 UI 渲染与用户输入事件的处理）

不难想象，如果一个应用在主线程同时存在大量 JS 与原生语言的同步调用，最终的结果只会是卡到完全无法响应用户输入

所以**我们需要让 JS 运行在一个独立线程中**（后文称之为 JS thread）

### 独立线程之后的问题

将 JS 线程独立出来之后，我们还需要考虑到两个问题：

1. 资源争夺：当两个线程需要同时读写一块内存（一个特定资源，比如当前某个 View 的长宽）的时候，线程锁会导致线程之间的等待，进而导致用户可感知的卡顿
2. 跨语言跨线程通讯开销：在 JS 与原生语言之间通讯会产生固定的开销，随着通讯频率的增加，这块开销也会线性增长

通讯开销这个问题我们留在后面讨论 RN 旧架构的时候再来展开聊，我们先来看看资源争夺这个问题 RN 团队是如何解决的

在正常的多线程编程模型下，资源争夺一直都是让开发者非常头疼的问题，但是 React 的编程模型在特定场景下给出了一个完美的解决方案：

```
declarative -> async -> responsive
```

React 的组件有一个特点，就是他们每一个都是纯函数，并且每个组件都描述了一组元素

在浏览器中，React 的组件最终会被解析为一个个 dom 元素，但实际上，React 通过抽象解开了组件与 dom 之间的耦合关系

说白话就是，React 只负责**定义**组件是什么样子的，至于后面组件被**解析（响应）**为 `div`、`View`、`UIView` 都无所谓

由于 React 已经帮我们完成了大部分的工作，我们实际上只需要搞定通信的部份，然后把组件渲染成与原生平台相关的 UI 元素就好了

于是，RN 的初版就在 2015 年发布了

### 题外话：RN 的增量迁移

上文我们说到，RN 是通过在原生系统中开启了一个 JS 线程，然后通过主线程解析 JS 线程传来的组件而渲染出 UI

这么做还有另外一个好处：主线程除了渲染 JS 线程的组件之外，还可以渲染自己原生系统的 UI，因为主线程对 APP 依然拥有绝对的控制权

这意味着 RN 不只能从 0 到 1 构建一个原生 APP，还能够在现有的 APP 上添加 RN 来运行 JS 代码渲染跨端的 RN 组件

## RN 的初版架构

> RN 的新架构虽然从 0.68 开始实验，但是其实 IOS 中的 JSI 在 0.58 就开始投入使用了，所以我们选择用 0.57 作为本章节讲解的 RN 版本

想要讲清楚任何一个框架的架构绝对不是一个非常容易的事情，特别是 RN 这种跨了多种语言的框架，所以本章节会分成以下几个部份：

1. RN 的线程模型
2. RN 的通信机制
3. RN 的 UI 布局与绘制
4. RN 的触摸事件处理机制
5. RN 的运行时异常与异常捕获处理
6. RN 应用的启动流程

同时，我们在后续讨论中都会围绕着架构图来讲解，架构如下图所示：

![RN old architecture](https://github.com/Joyee691/image-hosting/blob/main/blog/RN-old-architecture.png?raw=true)

### RN 的线程模型

在上文中，我们聊了为什么 RN 要为 JS 独立出一个线程，但实际上，RN 在默认情况下至少会有 4 个线程，让我们来一一了解他们分别有什么作用：

#### Main thread

主线程，又被称之为 UI 线程，当用户开启一个 IOS 或 Android 应用时，它会是第一个被启动的线程

它负责处理 UI 的渲染以及用户事件的输入，所以如果它被阻塞了，会导致应用卡顿甚至无响应

#### JS thread

JS 线程，用于在 IOS/Android 中运行 JS 代码

RN 团队最终选用了 IOS 系统自带的 JSCore 来执行 JS 代码（个人猜测是因为 IOS 禁用第三方 JS 引擎所做的权衡）

> 这里稍微展开一点，在 rn 中 js 引擎的选择有几种情况：
>
> - IOS：使用系统自带的 JSCore 框架，只能用 Apple 公开的 C API，并且由于 JSCore 是跟 IOS 系统绑定的，所以 RN 不能决定 JSCore 的版本
> - Android：RN 团队把 android-jsc 打包到了 app 中，除了公开的 C API 之外， android-jsc 还对外暴露了一些 API，这个我们后面会稍微聊到
> - remote debug：由于当时的历史原因，jsc 无法提供一些开发调试工具，所以 RN 团队给调试代码开辟了一条用 chrome v8 运行 js 然后通过 chrome devtool 调试代码的路径，在这种情况下 v8 会用 web socket 跟 RN APP 通信

与 React 类似，RN 也需要一个打包器来解析 jsx 并将其打包为 JS 代码，所以 RN 会先将代码交给 Metro 打包，打包的结果才会被送给 JSCore 执行

#### Shadow thread

Shadow thread 又称 UI manager，主要负责管理原生的 UI 元素

在我们聊 Shadow thread 干了什么之前，我们先来看看为什么需要 Shadow thread

在传统原生 App 的开发中，常见的 UI 更新一般包含有四个步骤：事件响应、状态更新、布局、绘制，而这四个步骤都是在主线程中以同步的方式进行的

在 RN 中，由于 JS 与主线程不在同一个线程中，如果我们依然让主线程来承担元素 layout 的工作，那么每次主线程 UI 更新都需要等待 JS 线程处理完状态更新后，再将变更的部份通过 Bridge（这个我们后面会聊，先当他是一个线程间通信手段） 发给主线程去重新布局及绘制，而且在这个过程中，主线程必须等待 JS 线程的结果，这会导致主线程频繁的等待

此外，React 可能会在一次渲染中提交多次小的更新，这个操作会触发主线程多次的布局操作，进而拖垮整个应用的性能

<br />

UI manager 就是用来解决上述性能问题的，而他的解决思路很简单：**将布局的工作外包给另一个线程**

有了 UI manager 后，我们可以将管理 UI 元素与计算布局的任务交给他来计算，并将布局结果告知主线程绘制

这样一来，我们就能够在最小化主线程布局计算开销了

此外，由于 UI manager 能够控制何时提交布局结果，他可以将多个 React 的更新打包成一个操作集合，并在每帧绘制前提交给主线程

<br />

要做到上述两点（计算布局、打包操作）UI manager 需要做到两件事情：

1. 在 Shadow thread 使用原生语言（java/object-c）维护一个 Shadow tree
2. 定制一个布局引擎，用来计算元素的布局信息

关于这部份的内容，本文的 UI 布局与绘制细说明

#### Native module thread

由于 RN 本质上还是运行在原生平台之上的框架，所以在某些特定时刻，它会需要调用原生的 API 来实现特定功能

对同一类 API 的调用可以被我们抽象成一个个的模块（Module），比如 Platform 可以获取系统当前的信息、Clipboard 可以读写系统的剪贴板等等

由于这些 module 是基于 Native 语言与 API 实现的，所以 RN 将其称之为 Native module

同时，由于 Native module 对 JS 暴露了原生语言（Java/Objective-C/C++）的类实例，他们也是 JS 调用原生代码的唯一入口（甚至包括我们上文所述的 UIManager 也是一个 Native module）

除了使用系统自带以及社区提供的 Native module 之外，我们还可以自己编写符合自身需求的 Native module 以满足调用系统 API、复杂计算、网络请求等等需求，这无疑大大增加了 RN 的扩展性

值得一提的是，Native module thread 在大多数时候并不是指的是一个线程，更像是对一类线程的统称，具体 Native module thread 的线程数量取决于系统，以及具体的 Native module

<br />

以上，我们聊完了 RN 的线程模型以及每个线程的职责，下面我们来聊聊 RN 线程的通信机制，也就是 Bridge

### RN 的通信机制

我们知道 RN 是一个**跨语言跨线程**的框架，所以要了解 RN 的通信机制，我们有两个问题需要解决：

1. 如何跨线程？
2. 如何跨语言？

因为 RN 的初版架构是基于 Android 与 IOS 搭建的，所以我们要先来聊聊这两个系统是如何处理跨线程通信（ITC）的

#### Android 的跨线程通信

最原始的 Android 跨线程通信（后文简写为 ITC）包含了三个概念：`Looper`、`MessageQueue`、`Handler`

MessageQueue 可以简单理解为一个代办任务清单，而 Looper 是一个循环机制，每次循环都会从 MessageQueue 中取出任务来执行

Handler 则是负责将代办任务添加进 MessageQueue 中

下面是一个简单的例子：

```java
// 获取主线程的 Handler
Handler mainHandler = new Handler(Looper.getMainLooper());

// 模拟在背景线程中的操作
new Thread(new Runnable() {
    @Override
    public void run() {
        // 给主线程发送一段可执行程序
        mainHandler.post(new Runnable() {
            @Override
            public void run() {
                System.out.println("Runnable running on: " +
                        Thread.currentThread().getName());
            }
        });
    }
}).start();
```

在 Android 中的 RN 就是基于这三个概念封装了一套平台无关的跨线程通信方法

#### IOS 的跨线程通信

##### GCD

IOS 的 ITC 跟 Android 有比较大的差异，主要源于系统中的 **GCD**(Grand Central Dispatch) APIs

GCD 将跨线程的资源调度与任务分配抽象成了不同的队列（Queue）

默认情况下，IOS 系统会创建 1 个 Main queue 以及 5 个 Global queue

Main queue 是唯一一个与主线程绑定的队列，`dispatch_get_main_queue()` 方法会返回 Main queue

Global queue 有 5 个，分别代表了系统规划的 5 种 QoS（Quality of Service，代表任务的重要性，高优的任务有优先调度资源的权力），分别为：

1. `userInteractive`：优先级最高，代表用户交互的事件，比如触摸事件，动画事件，滚动事件
2. `userInitiated`：优先级次高，代表用户发起且希望能够马上获得结果的事件，比如打开一篇文档，加载另一个页面
3. `default`：优先级中等，代表默认的事件，如果不知道当前任务要放哪个优先级，就放这个
4. `utility`：优先级次低，代表用户不会期望能立刻获得结果的事件，这类事件通常会有一个进度条来表示，比如加载批量文件
5. `background`：优先级最低，代表用户不会感知发生的事件，比如内容预加载、清理内存、数据同步

GCD 优雅的地方在于，除了 Main queue 绑定了主线程之外，其他的 Queue 都没有绑定特定线程

开发者只要把任务塞给相对应优先级的 Queue，系统就会帮其分配线程资源来执行任务，这个期间开发者是无感知的（不用分配线程，也不需要销毁资源）

<br />

除了上述系统创建的两种队列之外，开发者还可以根据自身需求创建队列，创建队列方法如下：

```objc
/**
*	dispatch_queue_create 用于创建自定义队列
* label 一个字符串用于标识该队列，用于 debug
* attr 队列的属性，可以在这里设置队列的 QoS，以及设置队列为 Serial/Concurrent 模式
**/
dispatch_queue_t dispatch_queue_create(const char *label, dispatch_queue_attr_t attr);
```

系统创建的 Main queue 与 Global queue 都是 **Serial 模式**，Serial 模式下的队列一次只会在一个线程执行一个任务，保证队列中的任务会以 FIFO 的顺序执行

自定义的队列可以手动设置为 **Concurrent 模式**，改模式下队列一次可能会在多个线程中同步执行多个任务，好处就是能加快队列中任务处理速度，坏处就是无法保证该队列中任务执行完成的顺序

<br />

如果要将任务委托给 queue 的时候，可以使用 dispatch 方法：

```objc
// 同步方式，会冻结当前 queue 直到 block 中的任务结束
void dispatch_sync(dispatch_queue_t queue, dispatch_block_t block);

// 异步方式，会直接执行当前 queue 后面的代码，如果执行 block 的 queue 需要返回结果，则再使用一次 dispatch_async 就好
void dispatch_async(dispatch_queue_t queue, dispatch_block_t block);
```

##### NSThread

IOS 的 GCD 提供了一种从系统层面调度线程的方案，开发者无法感知也无法控制特定的线程

但是当我们从 RN 的视角看这个方案的时候，情况就有点尴尬了：RN 要求开发者至少能够控制 JS 线程的创建与销毁（这样才能保证 JS 代码始终有环境可以运行）

还好 IOS 提供了一种不用 GCD 控制线程的解决方案：NSThread

NSTread 简单来说就是完全受控于开发者的 IOS 线程，RN 在创建 NSThread 后做了四件事：

1. 创建了一个永不停止的循环任务（RunLoop）确保线程不会闲置后被清理
2. 用 `RCTMessageThread` 类包裹了这个 RunLoop，并对外提供了 `runOnQueue` 与 `runOnQueueSync` 方法来发送可执行代码给 RunLoop 执行（这俩方法底层调用了 IOS 系统提供的 `CFRunLoopPerformBlock` 方法）
3. 使用上述方法，让 JSC 引擎在 RunLoop 中创建并完成初始化
4. 使用上述方法，将打包好的 js 代码交给 JSC 执行

至此，我们获得了一个完全受我们控制的线程，并且完成了运行 js 代码的前置配置

而 `runOnQueue` 与 `dispatch_async` 则为其他线程与 NSThread 的通信提供了底层支持

#### 跨语言通信

了解完 IOS 与 Android 是如何跨线程通信之后，我们来聊聊 RN 是如何跨不同语言来通信的

我们知道 IOS 与 Android 的应用程序分别是使用 Objc 与 Java 来编写的，并且他们通过运行 JS 引擎（由 C++ 编写）来执行 JS 代码

所以，我们可以得到以下四种通信方向（以函数调用为例）：

- `Java -> C++ -> JS`：Android App 调用 JS 函数
-  `Objc -> C++ -> JS`：IOS App 调用 JS 函数
- `JS -> C++ -> Java`：JS 调用 Android 的 Native module
- `JS -> C++ -> Objc`：JS 调用 IOS 的 Native module

再总结一下，我们想要解决 RN 不同语言之间通信，只需要解决以下三个问题：

1. `Java <-> C++`：Java 与 C++ 函数如何互相调用
2. `Objc <-> C++`：Objc 与 C++ 函数如何互相调用
3. `JS <-> C++`：JS 与 C++ 函数如何互相调用

##### Java <-> C++

关于 Java 与 C++ 之间的通信，Java 社区已经有了解决方案：**JNI (Java Native Interface)**

JNI 是由 JVM（Java 虚拟机）提供的一系列 C 接口 API 组成的机制

C 系列语言（包括 C++）只需要通过调用这些 API（比如 `FindClass`, `CallVoidMethod` 等）就可以调用 Java 的方法

反之，Java 也可以调用一些 C++ 侧通过 JNI 注册的一些方法

JNI 甚至允许 Java 与 C++ 同时持有对同一块内存的引用

所以这个问题被完美解决了

##### Objc <-> C++

关于 Objc 与 C++ 之间的通信也有现成的解决方案：**Objective-C++**

Objective-C++（`.mm` 文件）允许在同一个文件中混写 C++ 与 Objc 代码（包括类型与方法等）也能调用一些 C API（比如上述 `dispatch_queue_t`）

当然，持有同一份引用以及互相调用方法也不在话下

这一切要归功于 Objc 的编译器，它支持将 `.mm` 文件中的 C++ 与 Objc 编译为同一份目标代码

所以这个问题也被完美解决啦

##### JS <-> C++

最后，我们来聊聊最关键的一环，也就是 Js 与 C++ 之间的通信

我们以函数调用为例，来分别聊聊这两者是如何互相调用对方的函数的

<br />

一般来说，要跨语言调用对方函数，有两个问题需要克服：

1. **调用方式**：要么取得对方函数的引用，要么将意图封装成某种 DSL(Domain specific language) 并让对方解析
2. **参数类型转换**：将调用方的调用参数转换成被调用方的类型以供被调用方执行

**首先是调用方式：**

我们知道 JS 是运行在由 C++ 编写而成的引擎中的（在 RN 初版架构的例子中，这个引擎是 JSCore）

JSCore 除了提供了一个运行 JS 的环境之外，它还对外暴露了一系列的 C API 来让开发者控制它的部份能力

RN 就是使用了其中的 [JSGlobalContextCreateInGroup](https://developer.apple.com/documentation/javascriptcore/jsglobalcontextcreateingroup(_:_:)?language=objc) 来创建了一个 **context**

>  context 代表了 JS 执行的上下文，当 JSCore 完成了一个 context 的创建，我们可以认为他完成了运行 JS 代码前的准备工作

除此之外，RN 还调用了 [JSContextGetGlobalObject](https://developer.apple.com/documentation/javascriptcore/jscontextgetglobalobject(_:)?language=objc) 来通过 context 获取 JS **Global** 对象的引用

有了 Global 的引用，C++ 侧就有了在任意时候读取 JS 放在 Global 对象上的属性/方法的能力

**其次是类型转换：**

类型转换分成两个情况：

- JS 调用 C++ 方法时，我们需要将 JS 的参数类型转换成 C++ 的类型
- 如果 C++ 调用 JS 方法时，我们则需要将 C++ 的类型转换成 JS 类型

```cpp
// in JSCExecutor.cpp

// JS 调用 C++ 方法
void JSCExecutor::flushQueueImmediate(Value&& queue) {
  // 将 JS 类型的 queue 通过 JSON 转换成 C++ 的 folly::dynamic 类型
  auto queueStr = queue.toJSONString();
  m_delegate->callNativeModules(*this, folly::parseJson(queueStr), false);
}

// C++ 调用 JS 方法
void JSCExecutor::callFunction(
    const std::string& moduleId,
    const std::string& methodId,
    const folly::dynamic& arguments) {
  // ... 略过部份代码
  
  // 将 C++ 的 string，folly::dynamic 类型转换成 JS 的类型
  m_callFunctionReturnFlushedQueueJS->callAsFunction(
          {Value(m_context, String::createExpectingAscii(m_context, moduleId)),
           Value(m_context, String::createExpectingAscii(m_context, methodId)),
           Value::fromDynamic(m_context, std::move(arguments))});
  
  // ... 略过部份代码
}
```

让我们先看看 JS 调用 C++ 方法的部份，可以看到 RN 选择了先将 JS 的类型转换成了 JSON，再转换成 C++ 的类型，那么问题来了，为什么要这么麻烦呢？为什么不直接转换呢？

我个人认为有两个因素：

1. RN 在真机上运行的时候，是否可以直接转换类型呢？其实答案是可以的，但是会非常局限：JSCore 提供的公开 API 中确实有类似于 [JSValueToBoolean](https://developer.apple.com/documentation/javascriptcore/jsvaluetoboolean(_:_:)?language=objc)，[JSValueToNumber](https://developer.apple.com/documentation/javascriptcore/jsvaluetonumber(_:_:_:)?language=objc) 能将原始类型转换成对应 C++ 类型的方法，但是面对 String，Object 类型的时候就有点力不从心了
2. 在聊 JS 线程的时候，我们有聊到 RN 支持在 chrome 上调试 js 代码，在这个场景下 JS 跟 Native 的沟通主要依赖 ws 的连接，在这种情况下，我们只能把数据通过 JSON 进行传送，这个是无法绕过的，再进一步考虑到 dev/prod 环境下的表现一致性，JSON 是权衡之下的较优解

当然，RN 也在这个背景下做了些优化，queue 中其实并不是一个方法调用，而是类似一个先进先出的栈，里面存放了多个调用，最大程度的减少 JSON 带来的性能损耗

<br />

接下来我们来看看 C++ 调用 JS 方法的部份，这个部份主要用了两个类型转换的方法：`String::createExpectingAscii`, `Value::fromDynamic`，我们来看看他们是怎么实现的：

```cpp
// in Value.h
static String createExpectingAscii(JSContextRef context, const char* ascii, size_t len) {
  #if WITH_FBJSCEXTENSIONS
      return String(context, JSC_JSStringCreateWithUTF8CStringExpectAscii(context, ascii, len), true);
  #else
      return String(context, JSC_JSStringCreateWithUTF8CString(context, ascii), true);
  #endif
}

// in Value.cpp
Value Value::fromDynamic(JSContextRef ctx, const folly::dynamic& value) {
  #if USE_FAST_FOLLY_DYNAMIC_CONVERSION
    JSDeferredGCRef deferGC = JSDeferGarbageCollection(ctx);
    JSLock(ctx);
    JSValueRef jsVal = Value::fromDynamicInner(ctx, value);
    JSUnlock(ctx);
    JSResumeGarbageCollection(ctx, deferGC);
    return Value(ctx, jsVal);
  #else
    auto json = folly::toJson(value);
    return fromJSON(String(ctx, json.c_str()));
  #endif
}
```

非常有意思的是，这两个方法都用了两套实现

还记得我们之前聊 JS thread 的时候提到的 android-jsc 吗？`WITH_FBJSCEXTENSIONS` 与 `USE_FAST_FOLLY_DYNAMIC_CONVERSION` 中的实现是专门为了它写的，不像在 ios 中的 jsc，RN 团队可以完全控制 Android 的 jsc，所以也可以用一些比较 “野” 的写法

先来看看 `createExpectingAscii` 这个方法的两个实现，区别在于在 Android 中用了 `JSStringCreateWithUTF8CStringExpectAscii` ，在 IOS 中用了 `JSStringCreateWithUTF8CString`

这两个方法的区别在于，前者保证传进去的参数是 ascii 编码的字符，所以当 JSC 在转换类型的时候，只需要考虑 7-bit 的 ascii 编码就好，速度会比需要考虑 UTF-8 的后者快，而 `createExpectingAscii` 方法传入的 ascii 参数是类似于 `AppRegistry` 之类的字符，且完全受 RN 所控制，所以能保证编码类型；至于为什么 IOS 不能用前者，自然是 Apple 没有公开这个 API 啦～

<br />

再来看看 `fromDynamic` 这个方法，Android 中的实现（if 的部份）做了三件事：

1. 暂停了 JSC 的垃圾回收 GC：这个目的是为了防止转换类型的中间产物被 GC 回收导致不可控的错误
2. 用 `JSLock` 锁住了当前的 context（这里有个小知识点，由于 JSC 是单线程的，为了防止多个线程同时调用其 API，JSC 会把大部分的 API 调用前后锁住，以免发生资源争夺的情况，由于 `fromDynamicInner` 内部会调用大量的 JSC API，所以这里提前锁上了避免大量额的加锁解锁资源损耗）
3. 使用 `fromDynamicInner` 转换类型：

```cpp
JSValueRef Value::fromDynamicInner(JSContextRef ctx, const folly::dynamic& obj) {
  switch (obj.type()) {
    // For primitive types (and strings), just create and return an equivalent JSValue
    case folly::dynamic::Type::NULLT:
      return JSC_JSValueMakeNull(ctx);

    case folly::dynamic::Type::BOOL:
      return JSC_JSValueMakeBoolean(ctx, obj.getBool());

    case folly::dynamic::Type::DOUBLE:
      return JSC_JSValueMakeNumber(ctx, obj.getDouble());

    case folly::dynamic::Type::INT64:
      return JSC_JSValueMakeNumber(ctx, obj.asDouble());

    case folly::dynamic::Type::STRING:
      return JSC_JSValueMakeString(ctx, String(ctx, obj.getString().c_str()));

    case folly::dynamic::Type::ARRAY: {
      // Collect JSValue for every element in the array
      JSValueRef vals[obj.size()];
      for (size_t i = 0; i < obj.size(); ++i) {
        vals[i] = fromDynamicInner(ctx, obj[i]);
      }
      // Create a JSArray with the values
      JSValueRef arr = JSC_JSObjectMakeArray(ctx, obj.size(), vals, nullptr);
      return arr;
    }

    case folly::dynamic::Type::OBJECT: {
      // Create an empty object
      JSObjectRef jsObj = JSC_JSObjectMake(ctx, nullptr, nullptr);
      // Create a JSValue for each of the object's children and set them in the object
      for (auto it = obj.items().begin(); it != obj.items().end(); ++it) {
        JSC_JSObjectSetProperty(
          ctx,
          jsObj,
          String(ctx, it->first.asString().c_str()),
          fromDynamicInner(ctx, it->second),
          kJSPropertyAttributeNone,
          nullptr);
      }
      return jsObj;
    }
    default:
      // Assert not reached
      LOG(FATAL) << "Trying to convert a folly object of unsupported type.";
      return JSC_JSValueMakeNull(ctx);
  }
}
```

至于为什么 IOS 用的是 JSON 转换呢～当然是因为阻止 GC 以及 JSLock 的 API 在 IOS 中并不是公开的 API 啦～

至此！我们讲完了 RN 跨语言通信的一些基础机制与准备，接下来我们看看 Bridge 是如何利用这些构建连接的桥梁

#### Bridge in Native

接下来我们来看看 bridge 做了什么，以及它是如何与其他模块交互的

首先我们需要介绍三个重要角色：

- `Instance`：代码实现在 `Instance.cpp`。
  - 可以看成 RN app 的大管家，它接受原生程序的委托，主要负责管理 Native module 注册、初始化 Bridge、加载 JS bundle 等事务，同时它也是原生程序调用 JS 方法的唯一入口
- `bridge`：代码实现在 `NativeToJsBridge.cpp`。
  - bridge 由两个部分组成：`NativeToJsBridge` 与 `JsToNativeBridge`。
  - 前者主要把 `Instance` 的委托的加载 JS bundle 以及调用 JS 方法的任务传递给 `JSCExecutor`（此外它还肩负着委托 `JSCExecutor` 启动 JS 引擎的重责大任）
  - 后者则是负责处理从 JS 传来的 Native module 调用请求
- `JSCExecutor`：代码实现在 `JSCExecutor.cpp`。
  - 接受 `NativeToJsBridge` 的请求启动 JS 引擎（用 [JSGlobalContextCreateInGroup](https://developer.apple.com/documentation/javascriptcore/jsglobalcontextcreateingroup(_:_:)?language=objc) 创建 JS context）
  - 负责消费在 JSC 中加载 JS bundle 的任务
  - 往 JS global 对象上挂方法/读取 JS global 对象上的方法，为相互调用做准备
  - 负责将从 bridge 传来的调用 JS 方法的请求通过 global 对象传递到 JS 侧
  - 负责将从 JS 侧传来的调用 Native module 的请求传递给 bridge

三个角色的关系如图所示：

![rn bridge](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/RN_bridge.png)

从图中可以看到，RN 在代码结构上大致分了三层，`Instance` 之下是原生代码写的 APP 的壳（包括但不限于 APP 启动的原生代码，Native modules 的原生代码实现等）这个部份是平台区分的

从 `Instance` 开始到 `Bridge` 再到 `JSCExecutor`，这三个部份都是通过 C++ 语言实现，这个部份提供了跨平台的实现（同一套代码运行在不同的原生平台上）

`JSCExecutor` 之上，就是 JS 的领域了，JS 跟 `JSCExecutor` 主要是通过 global 对象进行通信，上述类型转换的代码也大多被封装到 `JSCExecutor` 的实现中

从原生代码或者 Native module 的角度来看，调用 JS 方法只需要调用 `Instance` 提供的 `callJSFunction` 后面的流程他们不需要感知

从 JS 的角度来看，调用原生方法则只需要调用 global 对象上的 `nativeFlushQueueImmediate` 方法，后面的流程也无需感知了

其实到这里 RN 的通信机制也讲的差不多了，后面我写了一点关于 JSCExecutor 跟 JS 互相调用的核心代码解析，有兴趣的可以看看

#### JS <-> C++ 互相调用核心代码解析

- 首先是 C++ 调用 JS 函数：

我们来看看 JS 侧，为了能够让 C++ 顺利调用我们的方法，我们需要往 Global 上挂点东西：

```js
// in MessageQueue.js
class MessageQueue {
  	// ...
  	callFunctionReturnFlushedQueue(module: string, method: string, args: any[]) {
        const moduleMethods = this.getCallableModule(module);
				moduleMethods[method].apply(moduleMethods, args);
    		return this.flushedQueue();
  }
}

// in BatchedBridge.js
const BatchedBridge = new MessageQueue();
Object.defineProperty(global, '__fbBatchedBridge', {
  configurable: true,
  value: BatchedBridge,
});
```

这两个文件在 JS 侧提供了给 C++ 调用特定函数的途径（为了方便讲解我简化了调用，但是方法名不变）

首先是 `MessageQueue` 的 `callFunctionReturnFlushedQueue` 方法，它会根据 `module` 去查找是否有可被调用的 module，如果有，则将其初始化后返回，最后使用 `apply` 调用其方法（`flushedQueue` 我们会在后续的布局与绘制篇中讲解）

接着是 `BatchedBridge`，它将上述的 `MessageQueue` 作为 `__fbBatchedBridge` 的值挂上了 globa 对象（`configurable` 值设为 true 是为了方便框架测试）

<br />

让我们再看回 C++ 侧，有了 JS 的 global 之后，我们可以通过以下方式调用 JS 方法：
```cpp
// 根据 context 获得 global 对象引用
auto global = Object::getGlobalObject(m_context);
// 从 global 对象中获取 __fbBatchedBridge 属性得到 MessageQueue 的实例
auto batchedBridgeValue = global.getProperty("__fbBatchedBridge");
auto batchedBridge = batchedBridgeValue.asObject();
// 从 MessageQueue 实例中取得 callFunctionReturnFlushedQueue 方法的引用
m_callFunctionReturnFlushedQueueJS = batchedBridge.getProperty("callFunctionReturnFlushedQueue").asObject();
// 调用 JSC 的 callAsFunction api，传入对应 moduleId，methodId，arguments 以调用 callFunctionReturnFlushedQueue 方法
m_callFunctionReturnFlushedQueueJS->callAsFunction({
  	Value(m_context, String::createExpectingAscii(m_context, moduleId)),
		Value(m_context, String::createExpectingAscii(m_context, methodId)),
		Value::fromDynamic(m_context, std::move(arguments))
})
```

值得注意的是，由于 C++ 与 JS 的类型是不同的，所以在 `callAsFunction` 中，C++ 需要先进行类型转换

其中 `moduleId` 与 `methodId` 都是固定的 String 类型，可以直接转换，但是 `arguments` 类型不固定，所以 RN 在 `Value::fromDynamic` 方法中采用了最简单粗暴的方法：**序列化与反序列化**

```cpp
Value Value::fromDynamic(JSContextRef ctx, const folly::dynamic& value) {
  auto json = folly::toJson(value);
  return fromJSON(String(ctx, json.c_str()));
}
```

至此我们完成了所有 C++ -> JS 的函数调用准备工作

<br />

- 接下来我们来看看 JS 是如何调用 C++ 函数的：

与 C++ 调用 JS 函数类似，只不过这次变成了 C++ 往 Object 上挂东西：

```cpp
// in JSCExecutor.cpp

/** 
* installNativeHook 主要用于将特定 C++ 方法的钩子安到 global 对象中
* template 是 C++ 的语法，他会接收符合对应返回值（JSValueRef）与参数（size_t, const JSValueRef[]）的 JSCExecutor 实例方法并将其放到 installNativeHook 中（在这个例子里，他会将其放进 exceptionWrapMethod 这个方法的 template 中）
**/
template <JSValueRef (JSCExecutor::*method)(size_t, const JSValueRef[])>
void JSCExecutor::installNativeHook(const char* name) {
  installGlobalFunction(m_context, name, exceptionWrapMethod<method>());
}


/**
* exceptionWrapMethod 主要职责是调用传入的 method，并且处理其的 Error
**/
template <JSValueRef (
    JSCExecutor::*method)(JSObjectRef object, JSStringRef propertyName)>
inline JSObjectGetPropertyCallback exceptionWrapMethod() {
  struct funcWrapper {
    static JSValueRef call(
        JSContextRef ctx,
        JSObjectRef object,
        JSStringRef propertyName,
        JSValueRef* exception) {
      try {
        auto executor = Object::getGlobalObject(ctx).getPrivate<JSCExecutor>();
        if (executor &&
            executor->getJavaScriptContext()) { // Executor not invalidated
          // 核心代码，执行对应的 C++ 方法
          return (executor->*method)(object, propertyName);
        }
      } catch (...) {
        *exception = translatePendingCppExceptionToJSError(ctx, object);
      }
      return Value::makeUndefined(ctx);
    }
  };
	// 返回对于 call 方法的引用，这个 call 方法会执行传入的 method 方法并返回其返回值
  return &funcWrapper::call;
}

/**
*	installGlobalFunction 主要职责是将上述的 call 方法放进 JS 的 global 对象中，以便后续让 JS 调用
**/
void installGlobalFunction(
    JSGlobalContextRef ctx,
    const char* name,
    JSObjectCallAsFunctionCallback callback) {
  String jsName(ctx, name);
  JSObjectRef functionObj = JSC_JSObjectMakeFunctionWithCallback(
    ctx, jsName, callback);
  Object::getGlobalObject(ctx).setProperty(jsName, Value(ctx, functionObj));
}

// 最后 C++ 调用上面的所有方法把 nativeFlushQueueImmediate 方法放进了 global 对象中
installNativeHook<&JSCExecutor::nativeFlushQueueImmediate>("nativeFlushQueueImmediate");
```

从上述代码中，我们能看到 C++ 侧成功的将 `nativeFlushQueueImmediate` 方法放进了 global 对象中

接下来我们来看看 JS 是如何使用这个方法的：

```js
// in MessageQueue.js
class MessageQueue {
  	// ...
    // JS 侧调用 C++ 方法的入口，为了简化我删除了部份代码（包括开发模式、性能打点、注册回调函数等等），有兴趣的可以自己去看看：https://github.com/facebook/react-native/blob/v0.57.0/Libraries/BatchedBridge/MessageQueue.js#L168
  	enqueueNativeCall(
    moduleID: number,
    methodID: number,
    params: any[],
  	){
        // 将当前要调用的方法放入队列中
      	this._queue[MODULE_IDS].push(moduleID);
        this._queue[METHOD_IDS].push(methodID);
        this._queue[PARAMS].push(params);

        const now = new Date().getTime();
        // 如果符合清空队列的条件，则一股脑将当前队列全部发给 C++
        if (global.nativeFlushQueueImmediate &&
        now - this._lastFlush >= MIN_TIME_BETWEEN_FLUSHES_MS
        ) {
            const queue = this._queue;
            this._queue = [[], [], [], this._callID];
            this._lastFlush = now;
            // 调用 global 中 C++ 的方法
            global.nativeFlushQueueImmediate(queue);
        }
    }
}
```

JS 传入 `nativeFlushQueueImmediate` 方法的是一个队列，它的类型是：`_queue: [number[], number[], any[], number];`

- 第一个元素是一个数字数组，代表需要调用的 native module 的 id
- 第二个元素也是一个数字数组，代表需要调用的方法的 id
- 第三个元素也是数组，代表调用方法的参数
- 最后一个元素是一个数字，唯一标识了当前的调用 id
- 如果我们需要从 queue 中取出方法调用，我们只需要对前三个元素取出同一下标的元素即可

```js
// 举个例子
module = _queue[0][0][0]
method = _queue[0][1][0]
args = _queue[0][2][0]
module[method](...args)
```

最后，我们来看看 C++ 是如何处理这个 queue 以及处理 JS -> C++ 的类型转换的：

```cpp
// in JSCExecutor.cpp
JSValueRef JSCExecutor::nativeFlushQueueImmediate(
    size_t argumentCount,
    const JSValueRef arguments[]) {
        Value queue = Value(m_context, arguments[0])
        // 序列化 queue
        auto queueStr = queue.toJSONString();
    	// 反序列化 queue 得到 folly:dynamic
        m_delegate->callNativeModules(*this, folly::parseJson(queueStr), false);
        return Value::makeUndefined(m_context);
}
```

可以看到，RN 使用了**序列化-反序列化**的操作将 JS 的 `Value` 类型转换成了可以在 C++ 中使用的 `folly:dynamic` 类型

这个方法无疑简单粗暴的解决了类型转换的问题，但同时也增加了 JS 与 C++ 方法互相调用的性能开销，考虑到这条链路的触发频率，这个部份有较大概率会成为 RN 的性能瓶颈，所以才有了后来用 JSI 取代序列化-反序列化的新架构（这个我们后面聊）

### RN 的 UI 布局与绘制

我们知道 RN 之所以受欢迎的其中一个原因就是把之前只有在 React 中有的 jsx 带进了 Native 开发的世界

在这一个篇章中，我们会深入了解 RN 是如何将 `<View>`、`<Text>` 标签转换成 `UIView`（IOS）、`ViewGroup`（Android）

当然，还有 `yoga` 究竟在其中做了什么？以及为什么要有 `yoga`

<br />

但是，在深入之前，我们要先聊聊一个方法：`runApplication`

`runApplication` 顾名思义，这是一个启动应用的方法，但这里启动的不是原生应用，而是在 JS bundle 在加载完之后，由 RN 在原生应用中**启动 React 应用**，它的调用过程涵盖了三个线程，其调用流程如下：

![](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/RN_runApplication_workflow.png)

可以看到，我们的 RN 程序启动以后，会在客户端的 `RootView`  中调用 `runApplication` 方法，这个方法的调用会通过我们在通信机制中讲到的 `Instance -> Bridge -> JSCExecutor` 这条通道一路走进 JS 程序

当 JS 接收到 `AppRegistry.runApplication` 的调用后，它会去找到我们 RN 项目根目录的 `index.js` 注册的组件（默认在 `App.js`），最后调用 `ReactNative.js` 的 `render` 方法

`ReactNative.js` 中包含着 RN 在 JS 侧的核心代码，他的主要任务是将 React diff 完的 fiber 转换成为一系列的 `UIManager.xxxxx` 调用

这些调用最后会触发 Native 中的 UIManager（UIManager 也是一个 Native module） 的逻辑生成原生元素（`UIView`,`ViewGroup` 等等） ，最后在 yoga 这个布局引擎的帮助下完成原生页面的渲染

#### createView, setChildren 与 yoga 布局

接下来我们以一个简单的例子来聊聊我们写的 RN jsx 是如何最后转变为原生元素并显示在屏幕中的

```html
<View>
	<Text>Hello world!</Text>
</View>
```

当我们这个组件被 `ReactNative.js` 的 `render` 执行后，会有以下方法被调用：

> 题外话，在具体的场景中，上述例子可能不止有下述方法被调用了，被调用的方法也可能会有区别，但是他们的目的与功能是类似的，本文为了方便理解做了部份简化

1. `UIManager.createView(tagV, 'RCTView', rootTag, {})`
2. `UIManager.createView(tagT, 'RCTText', rootTag, { text: 'Hello world!' })`
3. `UIManager.setChildren(tagV, [tagT])`

<br />

`createView` 接受 4 个参数：

- 第一个参数是一个自增的数字，会唯一标识一个创建的元素
- 第二个参数是需要创建的元素类型，因为我们需要的是 `View` 元素，其对应的是 `RCTView` （在原生平台中，它是一个继承自各自平台 View 元素的类，其中定义了一些 RN 需要的方法）
- 第三个参数是根容器（root container）的唯一标识符，根容器在 native 侧创建，是 RN 创建的元素的根结点。由于一个 APP 中可以创建多个根容器，`createView` 需要确保当前创建的元素被归类到正确的容器中
- 最后一个参数代表元素的属性

`setChildren` 接受 2 个参数：

- 第一个参数与 createView 一致，唯一标识着一个父元素
- 第二个参数是一个数组，其中包含子元素的标签

<br />

当这两个方法通过 bridge 最后进入 Native 侧的 `UIManager` 时，会根据 IOS 与 Android 的平台特性区分为两套实现，分别是：

- `RCTUIManager.m`：IOS 中 `UIManager` 的实现
- `UIManagerModule.java`：Android 中 `UIManager` 的实现

下面我们分别聊聊这两者都做了些什么

##### UIManager in IOS

在 IOS 的 `createView` 实现中，主要做了 3 件事：

1. 根据 `RCTView` 这个类型分别创建了一个 `shadowView` 以及一个离屏的 UIKit `UIView`（`RCTText` 类也同理，后不赘述）
2. 根据 `RCTView` 这个类型的规则，从元素的属性中筛选了部份 `shadowView` 需要的属性赋值给 `shadowView` 的 `props`
3. 将当前的 `shadowView` 放进 `_shadowViewsWithUpdatedProps` 中等待后续消费

其中，`shadowView` 是 RN 为了方便 yoga 计算布局而设计的类型，而 `UIView` 是 IOS 正儿八经在屏幕上渲染的元素

两者的区别在于 `shadowView` 负责接受元素布局相关的属性（如 `width`, `height`, `border`, `margin` 等），然后交给 yoga 计算布局；`UIView` 只需要处理布局之外的 `backgroundColor`, `opacity`, `shadow` 等等属性就好

> 属性的分类依据每个类型不同而不同，比如 RCTView 的定义在 RCTViewManager.m 中

这样做的好处在于可以将计算量较大的布局工作交给另外一个线程防止 IOS 的主线程阻塞

<br />

在 IOS 的 `setChildren` 实现中，主要做了 3 件事：

1. 将子元素的 `shadowView` 插入成为父元素 `shadowView` 的 `subView`
2. 将子元素插入成为父元素的 `subView`
3. 将当前的 `shadowView` 放进 `_shadowViewsWithUpdatedChildren` 中等待后续消费

<br />

最后，我们在之前讲 `runApplication` 的调用流程的时候留了一个伏笔：在 ` JSCExecutor.cpp` 中调用 JS 的方法用的是 `callFunctionReturnFlushedQueue` ，以下是它的实现：

```js
callFunctionReturnFlushedQueue(module: string, method: string, args: any[]) {
    this.__guard(() => {
      // 调用对应的 js 方法
      this.__callFunction(module, method, args);
    });

  	// 返回到目前为止积压在 queue 中的 native module 调用请求
    return this.flushedQueue();
  }
```

可以看到在执行完 js 侧的 `runApplication` 后，该方法会将执行过程中累积的 native module 调用一下子清空，明确告知 native 侧：**我这个方法调用过程中发生的请求已经全部给你了**

当 native 侧接受到这个信息之后，它会去轮询所有注册过 `batchDidComplete` 方法的 native module（`UIManager` 也是其中一员）并执行他们的 `batchDidComplete` 方法

在 `UIManager` 中 `batchDidComplete` 调用了最重要的一个方法：`_layoutAndMount`

我们来看看实现：

```objc
// in RCTUIManager.m

- (void)_layoutAndMount
{
  // 消费上述 _shadowViewsWithUpdatedProps：把有变化的 props 经过转换后赋值给 yogaNode（后续 yoga 会根据这些节点的属性来计算布局
  [self _dispatchPropsDidChangeEvents];
  // 消费上述 _shadowViewsWithUpdatedChildren：根据不同的 view 类型做不同处理（shadowView 场景的话什么都不做）
  [self _dispatchChildrenDidChangeEvents];

  // 遍历所有的 root container（reactTag）
  for (NSNumber *reactTag in _rootViewTags) {
    // 找到每一个 root container 的 shadowView（也就是 rootShadowView），由于 view 跟 shadowView 是一一对应的关系，所以 rootShadowView 也有可能有多个）
    RCTRootShadowView *rootView = (RCTRootShadowView *)_shadowViewRegistry[reactTag];
    // 触发 yoga 的布局计算，并且把布局结果包装到一个代码片段中返回，返回的代码片段会被加到一个等待队列中等待被主线程执行（因为在 ios 中只有主线程能操纵 UIKit）
    [self addUIBlock:[self uiBlockWithLayoutUpdateForRootView:rootView]];
  }

  // 执行上述的代码片段，将计算好的布局应用给元素
  [self flushUIBlocksWithCompletion:^{}];
}
```

补充一点，我们说到 `uiBlockWithLayoutUpdateForRootView` 方法除了计算新的元素布局之外，还会返回一个代码片段，这个代码片段除了在普通情况下将计算好的布局应用给元素之外，还负责判断该元素是否需要动画，如果需要的话，还会将对应的动画效果应用给对应元素

至此，我们完成了对 IOS 中 UIManager 的部份方法与核心机制讲解

##### UIManager in Android

UIManager 在 Android 中的目标跟在 IOS 中是一致的，主要区别在于加入了一个 `NativeViewHierarchyOptimizer` 的优化机制

至于加入的原因我们会在后文描述，现在我们先来看看 Android 是如何实现 `createView`, `setChildren`, `batchDidComplete` 的

在 Android `createView` 的实现中，RN 也做了三件事：

1. 根据 `RCTView` 这个类型创建了一个 `shadowView` ，并将其保存至 `mShadowNodeRegistry`（一个用来保存所有 shadowView 的类）
2. 将元素属性中 `shadowView`  需要的属性赋值给新创建的 `shadowView` 
3. 将创建原生 View 元素的任务交给 `NativeViewHierarchyOptimizer`，它会在符合条件的情况下创建原生元素

`NativeViewHierarchyOptimizer` 就是 Android 与 IOS 在 UIManager 中最大的区别，它的工作主要就是将元素用是否为**布局专用元素**进行区分：**如果是布局专用元素它将不会创建真正的原生元素**；反之则会跟 IOS 一样创建原生元素

<br />

在 `setChildren` 中，则是：

1. 将子元素的 `shadowView` 插入成为父元素 `shadowView` 的 `mChildren`（对应 IOS 中的 `subView`）
2. 将插入原生子元素的任务交给 `NativeViewHierarchyOptimizer`，在其中会判断父元素是否为**布局专用元素**，如果是，则会将子元素插入到最近的不是**布局专用元素**的父元素上

<br />

最后，在 JS 侧所有请求结束后，Android 会执行 `dispatchViewUpdates` 方法（对应 IOS 中的 `_layoutAndMount`）

```java
// in UIImplementation.java

public void dispatchViewUpdates(int batchId) {
    try {
      // 1. 调用 yoga 计算布局
      // 2. 将布局结果转换成一些对元素的操作并将这些操作入栈等待执行
      // 3. 执行 JS 侧的 onLayout 回调
      updateViewHierarchy();
      // 清理布局过程中使用到的一些标识
      mNativeViewHierarchyOptimizer.onBatchComplete();
      // 将操作一一出栈并应用布局（调用元素的 measure 以及 layout 方法）
      mOperationsQueue.dispatchViewUpdates(batchId, commitStartTime, mLastCalculateLayoutTime);
    }
  }
```

##### Android vs IOS

在上文中，我们说到 Android 会比 IOS 多一个 `NativeViewHierarchyOptimizer` 用来防止为一些**布局专用元素**创建真正的元素，为什么呢？

首先，什么是布局专用元素？布局专用元素需要同时满足个条件：

1. 该元素是 `RCTView`
2. 该元素的 [collapsable](https://reactnative.dev/docs/view#collapsable) 元素是 true（也就是默认值）
3. 该元素所有属性都是**布局专用属性**（LAYOUT_ONLY_PROPS），包含：

```java
// in ViewProps.java

private static final HashSet<String> LAYOUT_ONLY_PROPS =
      new HashSet<>(
          Arrays.asList(
              ALIGN_SELF,
              ALIGN_ITEMS,
              COLLAPSABLE,
              FLEX,
              FLEX_BASIS,
              FLEX_DIRECTION,
              FLEX_GROW,
              FLEX_SHRINK,
              FLEX_WRAP,
              JUSTIFY_CONTENT,
              ALIGN_CONTENT,
              DISPLAY,

              /* position */
              POSITION,
              RIGHT,
              TOP,
              BOTTOM,
              LEFT,
              START,
              END,

              /* dimensions */
              WIDTH,
              HEIGHT,
              MIN_WIDTH,
              MAX_WIDTH,
              MIN_HEIGHT,
              MAX_HEIGHT,

              /* margins */
              MARGIN,
              MARGIN_VERTICAL,
              MARGIN_HORIZONTAL,
              MARGIN_LEFT,
              MARGIN_RIGHT,
              MARGIN_TOP,
              MARGIN_BOTTOM,
              MARGIN_START,
              MARGIN_END,

              /* paddings */
              PADDING,
              PADDING_VERTICAL,
              PADDING_HORIZONTAL,
              PADDING_LEFT,
              PADDING_RIGHT,
              PADDING_TOP,
              PADDING_BOTTOM,
              PADDING_START,
              PADDING_END));
```

在这种情况下，`NativeViewHierarchyOptimizer` 将不会创建真正的原生元素

为什么要在 Android 中应用这个优化呢？这个我们要从 Android 的 [Choreographer](https://developer.android.com/reference/android/view/Choreographer) 开始说起：

对于非 RN 的 Android app来说，当 app 接受到硬件传来的 **vsync** 信号之后，他会启动 choreographer 程序：

```js
Choreographer
  → ViewRootImpl.performTraversals()	// 开始从程序根节点向下遍历所有元素
      → performMeasure()   // 执行元素 measure 方法
      → performLayout()    // 计算元素布局
      → performDraw()      // 绘制元素
```

其中 measure 以及 layout 这两步只有当元素本身判断需要（元素调用了 [requestLayout](https://developer.android.com/reference/android/view/View#requestLayout()) ）之后才会启动，由于 RN 引入了 yoga 引擎来计算布局（取代了 performMeasure 与 performLayout 的功能），所以 RN 的目标就是让 Android 本身的 performMeasure 以及 performLayout 尽可能少的被启动

所以在 Android 中才需要 `NativeViewHierarchyOptimizer` 来尽可能减少多余的节点被挂在渲染树上

那么为什么 IOS 不需要呢？

因为 IOS 用的是完全不同的机制，IOS 提供了两种渲染机制：Frame-Based Layout 和 Constraint-Based Layout

RN 选用了 Frame-Based Layout，它的好处就是：系统会直接根据我们计算好的结果来渲染下一帧，不会有多余的操作

### RN 的触摸事件处理机制



### RN 的运行时异常与异常捕获处理

### RN 应用的启动流程

