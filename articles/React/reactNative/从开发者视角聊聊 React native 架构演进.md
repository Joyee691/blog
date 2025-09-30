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

首先是 C++ 调用 JS 函数：我们知道 JS 是运行在由 C++ 编写而成的引擎中的（在 RN 初版架构的例子中，这个引擎是 JSCore）

JSCore 除了提供了一个运行 JS 的环境之外，它还对外暴露了一系列的 C API 来让开发者控制它的部份能力

RN 就是使用了其中的 [JSGlobalContextCreateInGroup](https://developer.apple.com/documentation/javascriptcore/jsglobalcontextcreateingroup(_:_:)?language=objc) 来创建了一个 **context**

>  context 代表了 JS 执行的上下文，当 JSCore 完成了一个 context 的创建，我们可以认为他完成了运行 JS 代码前的准备工作

除此之外，RN 还调用了 [JSContextGetGlobalObject](https://developer.apple.com/documentation/javascriptcore/jscontextgetglobalobject(_:)?language=objc) 来通过 context 获取 JS **Global** 对象的引用

有了 Global 的引用，C++ 侧就有了在任意时候读取 JS 放在 Global 对象上的属性/方法的能力

<br />

接下来我们来看看 JS 侧，为了能够让 C++ 顺利调用我们的方法，我们需要往 Global 上挂点东西：

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

至此我们完成了所有 C++ -> JS 的函数调用准备工作

<br />

接下来我们来看看 JS 是如何调用 C++ 函数的

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

可以看到，RN 使用了序列化-反序列化的操作将 JS 的 `Value` 类型转换成了可以在 C++ 中使用的 `folly:dynamic` 类型

这个方法无疑简单粗暴的解决了类型转换的问题，但同时也增加了 JS 调用 C++ 方法的性能开销，考虑到这条链路的触发频率，这个部份有较大概率会成为 RN 的性能瓶颈，所以才有了后来用 JSI 取代序列化-反序列化的新架构（这个我们后面聊）













#### Bridge in Native

上面我们分别介绍了 Android 与 IOS 进行 ITC 的方法，接下来是时候聊聊在 RN 的场景下是如何利用这些方法搭建出一套跨语言跨线程的通信体系（Bridge）了

为了能够更加清晰的脉络，本章节会以 `AppRegistry.runApplication` 方法的调用流程来进行讲解 Bridge 的工作流程

`AppRegistry.runApplication` 方法是所有 RN 应用在初始化完了之后由 Native 侧调用的方法，它的主要职责是告诉 JS 运行时：**嘿，我这边前置工作完成了，你快告诉我你要我做什么？**







### RN 的 UI 布局与绘制

### RN 的触摸事件处理机制

### RN 的运行时异常与异常捕获处理

### RN 应用的启动流程

