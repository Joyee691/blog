# React hooks

> 全文共 xx 字，建议阅读时间 xx min

## 背景知识

> 如果你只是一个 react 初学者并且想先知道 hooks 有哪些以及他们各自的用法，可以先跳过这一章节，这并不会影响你对 hooks 的使用；但是如果想要进一步了解 hooks 背后的一些思考，那么我强烈建议你在了解完 hooks 用法之后回来看看

在这一章中，我会从三个角度去简单介绍 hooks 背后的一些想法，看完这一章，你将能够回答这三个问题：

1. hook 是什么？
2. 如何优雅的使用 hook？
3. 为什么要使用 hook？

### 动机

> 这个部分简单的描述了 React 团队引入 hooks 背后的思考

#### 难以复用组件 state 行为

在以前，我们如果想要使用组件的 state，就必须要使用 class component。但是在使用 class component 的过程中，如果想要把多个组件共有行为抽象出来的话，就只能使用渲染属性（Render Props），或高阶组件（High-order Component）。这两个方法除了显著增加了开发者的心智负担之外，更重要的是造成了 React Devtool 中的“包裹地狱”（Wrapper Hell）

通过使用 hooks，我们能够优雅的对组件行为进行抽象，具体例子我们会在 [拓展性](###拓展性) 这一节展开

#### 难以理解的复杂组件

随着项目的发展，由 class 组成的业务组件很容易越发臃肿最后导致难以维护。

举个例子，我们知道，在编写 react 的过程中有很多“副作用”（Side Effect）比如数据获取（fetching）或注册事件（subscribing）是必须在组件渲染完成之后才能运行的，并且还要额外关注在组件更新时的表现。这些副作用一般会被我们在组件的生命周期 componentDidMount 或 conponentDidUpdate 中调用并且在 componentWillUnmount 中被清除。随着业务的发展，这些生命周期函数中往往会耦合了非常多没有关联的代码，这样做的代价不仅造成了代码更难以被理解，也导致 bug 更容易被引入组件之中。

随着 useEffect 的发布，上述的问题能够很好的被避免，首先我们允许在一个组件中使用多个 useEffect，这样我们就更容易提高相关逻辑的内聚性；再来，useEffect 集合了上述三个生命周期的功能，让我们更容易将重心从关注组件的运行时序转移到对业务场景的思考。

关于 useEffect 的详细介绍将在之后的 hooks 讲解中展开

#### class 的理解成本

相信用过 class component 的人都遭遇过 this 指向的问题，而且每次写事件处理函数时，都要提醒自己额外 bind 一下 this，这个无疑加大了所有 react 开发者的心智负担

### 规则

> 本小节讲述了使用 hooks 必须要遵守的两个原则

#### 只在 react 函数的顶层作用域中使用 hooks

由于 react 支持开发者在一个 function component 中使用多个 hook，甚至能使用多个同一种 hook，为了有效的区分不同 hook 的操作，react 团队选择了用函数内**语句执行的顺序**来区分不同的 hook

如果我们在条件或者循环语句中使用了 hook 的话，会导致这一机制的失灵，所以，react 团队定下了这一个规则

就像 Ryan Florence 在一场演说中讲的：If you love hook, you should use it unconditionally.😆

#### 只在 react 函数或自定义 hooks 中使用 hooks

我们知道 react 中最基础的两个 hook：useState 和 useEffect 都是基于 function conponent 独有机制而构造起来的，所以我们不能将其用在普通的 JS 函数中

当然，自定义 hooks 其实本质上也是一个 JS 函数，但是它不同之处在于最后它会被交给一个 react 函数执行，所以它也适用于这一条规则

### 拓展性

> 本小节用一个例子描述了 hooks 最核心的能力——组件逻辑抽象

通过上面的阅读，我们知道了 react hooks 被提出来的一个很大的功能就是可以让我们摆脱难以理解的渲染属性以及高阶函数，这就是我们接下来要介绍的自定义 hook

要了解自定义 hook，我们先来了解一下官网上对它的说明：自定义 hook 使得组件的逻辑得以被复用

<br />

为了说明自定义 hook 的功能，我们用一个例子来说明：

假设今天我们在开发一款游戏，需要一个组件来监听并展示好友是否在线，我们假设组件是这样子的

```jsx
import React, { useState, useEffect } from 'react';

function FriendListItem(props) {
  const [isOnline, setIsOnline] = useState(false);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    FriendAPI.subscribe(props.friend.id, handleStatusChange);
    return () => {
      FriendAPI.unsubscribe(props.friend.id, handleStatusChange);
    };
  });

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

当我们美滋滋的写好了这个组件之后，突然想起来我们游戏还有主播，需要也展示一下他们的直播状态

这个时候我们可以观察一下，主播与用户之间 state 的管理逻辑可以复用，消息订阅与取消订阅也可以复用，那么我们是不是可以把这些相同逻辑进行抽象呢？

```js
import { useState, useEffect } from 'react';

function useOnlineStatus(id, API) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    API.subscribeToFriendStatus(id, handleStatusChange);
    return () => {
      API.unsubscribeFromFriendStatus(id, handleStatusChange);
    };
  });

  return isOnline;
}
```

可以看到，我们通过抽象相同逻辑，把 state 跟 effect 抽离出了原来的组件，并且只返回了原来组件关注的 isOnline

另外我们注意到，useOnlineStatus 虽说是一个自定义 hook，但它也是一个函数，所以也能正常接收参数

在这个例子，我们把 id 跟 API 当成参数传入，使得自定义组件能够更加灵活的组成逻辑

如果想要使用它，我们只需要将一开始的 FriendListItem 稍作修改就可以了：

```jsx
import React from 'react';

function FriendListItem(props) {
  const isOnline = useOnlineStatus(props.friend.id, FriendAPI);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

这样一来，随着我们对 hooks 的了解以及业务逻辑的深入，我们可以用自定义 hooks 组合不同的 hook 来封装不同的逻辑，既提高了代码的复用性，也避免了之前高阶组件带来的包裹地狱问题

## Introducing hooks

> 本章开始将会分别介绍 react 官方提供的 hook 的一些基本使用方法与背后原理

### useState

> useState 就是为 function component 提供了 state 的能力的 hook

#### 基本用法

```tsx
import React, { useState } from 'react';

/**
 * useState hook
 * @param {S} initialState 初始化 state 的值
 * @return {S} state 
 * @return {Dispatch<SetStateAction<S>>} setState 更新 state 的方法
**/
const [state, setState] = useState<S>(initialState)
```

useState 是一个方法，它可以接收一个参数并且返回一个数组

- initialState 代表一开始初始化时 state 的值，只会在该组件第一次使用时被初始化
- 返回的数组中包含了 state 跟一个更新 state 的方法，我们使用数组解构赋值分别将他们赋值为 state 跟 setState
- useState 接受一个范型 S，代表 state 的类型，也约束了 setState 方法的入参类型

<br />

想要使用 state 只需要简单的在 jsx 语法中使用就好

```tsx
function App() {
  const [state, setState] = useState('')

  return (
    <div>
      {state}
    </div>
  );
}
```

#### 多个 state

在使用 class conponent 时，如果我们想要使用多个 state，必须将他们放在一个对象内，想要更新的话也必须深拷贝一次整个对象

```jsx
class App extends react.Component {
  constructor(props) {
    super(props);
    this.state = {
      state1: 1,
      state2: 2,
    };
  }

  render() {
    return (
      <div>
        <p onClick={() => this.setState({ ...this.state, state1: this.state.state1 + 1 })}>
          state1: {this.state.state1}
        </p>
        <p onClick={() => this.setState({ ...this.state, state2: this.state.state2 + 1 })}>
          state2: {this.state.state2}
        </p>
      </div>
    );
  }
}
```

但是对于 function conponent 中，可以通过多次调用 useState 来拥有多个 state，这样一来也能减少在赋值时的心智负担

```jsx
function App() {
  const [state1, setState1] = useState(1)
  const [state2, setState2] = useState(2)

  return (
    <div>
      <p onClick={() => setState1(state1 + 1)}>
        state1: {state1}
      </p>
      <p onClick={() => setState2(state2 + 1)}>
        state2: {state2}
      </p>
    </div>
  )
}
```

#### 修改 state

在上面修改 state 的值的时候我们说到，必须使用 useState 返回的 setState 方法来修改

究其根本，这个跟 react 团队设计 state 的原理有关，在这里，我们只需要知道 react 将 state 保存在一个 function component 之外的上下文中就好，这样以来才能保证在每次组件被更新的时候 state 的值不随之改变

既然 state 的值被保存在了其他地方，那么 useState 返回的 state 其实只是一个当前 state 的副本，它只是当前 state 的快照，并非真正的 state 本身，所以才需要一个方法 setState 来更新 state 的值

<br />

另外，上面在讲述 setState 的用法时，我们只用了一种用法，即直接传入一个新的 state 值，但是如果我们看看它的类型定义：

```jsx
type SetStateAction<S> = S | ((prevState: S) => S);
```

不难发现，它其实是有两种用法的，除了传入新的 state 之外，我们还可以传入一个函数

在函数中，我们可以做一些较为复杂的计算，但是这个函数最大的作用还是在于它的参数 prevState，也就是目前的 state

要了解这个 prevState 的作用，我们需要引入一个 [stack overflow](https://stackoverflow.com/questions/56404819/usestate-hook-setstate-function-accessing-previous-state-value) 的例子来说明：

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={() => setTimeout(() => setCount(count + 1), 2000)}>
        Delayed Counter (basic)
      </button>
      <button onClick={() => setTimeout(() => setCount(pervState => pervState + 1), 2000)}>
        Delayed Counter (functional)
      </button>
      <button onClick={() => setCount(count + 1)}>Immediate Counter</button>
    </div>
  );
}
```

可以看到，Delayed Counter (basic) 跟 Delayed Counter (functional) 都给点击事件加了延迟，区别在于一个用了 prevState，另外一个则是直接使用了当前的 count 快照

在这种情况下，如果我们点了 Delayed Counter (basic) 之后马上使用 Immediate Counter 更新 state 的值，等到前者实际更新时，state 还会被设置为 1

但是如果我们点击 Delayed Counter (functional) 之后不论用 Immediate Counter 更新了几次 state，最后前者实际更新时都是当前的 state 加一

造成这个区别的原因时因为**闭包**，当点击事件被 setTimeout 延迟后，实际上在 setTmeout 中产生了闭包，在闭包的环境中 count 的值一直都是点击那一瞬间的值，哪怕 count 后来被更新了，也不影响闭包中的值；相比之下，Delayed Counter (functional) 按钮在更新 state 的时候传入了一个 function，并且在其中得到了 prevState，这个 prevState 的值永远是当下最新的 state

<br />

或许上面的例子不够直观，下面我们再用两个例子来看看其中的区别：

```jsx
function A() {
  const [count, setCount] = useState(0);

  setCount(count + 1);
  setCount(count + 1);
  console.log('A: ', count);	// ？
}

function B() {
  const [count, setCount] = useState(0);

  setCount(count => count + 1);
  setCount(count => count + 1);
  console.log('B: ', count);	// ？
}
```

相信看懂了上面的小伙伴应该已经知道了，A 的结果是 1；B 的结果是 2，但是为什么呢？

因为对于 A 来说，两次 setCount 的参数都是 "0+1"，也就是实际上两次 setCount 做了同一件事

对于 B 来说，每次的 count 拿的都是更新之后的值，也就是第一次 setCount 使用 1 更新了 count，第二次 setCount 使用 2 更新了 count

所以，**如果有涉及之前 state 的状态，都推荐使用传入函数的方式更新 state**

<br />

另一种不得不使用函数的方式是，如果 state 是 Object、Array、Map、Set 等的引用类型时，也需要 prevState 来配合使用

举个例子：

```jsx
function App() {
  const [songs, setSongs] = useState(['Natural', 'Sharks'])

  const add = (newSong) => {
    setSongs(prev => [...prev, newSong])
  }

  return (
    // something
  )
}
```

之所以每次都需要设置一个新的对象，是因为 react 在比较新旧 state 的值使用的方法是比较引用，这个方式能够大幅减少比较的开销，但是缺点就是如果我们只改变了对象中的值而非整个引用的话，对于 react 来说这是同一个 state，也就不会触发更新了

#### 惰性初始化 Lazy Initialization

除了修改 state 的方法可以传一个方法，useState 的初始化参数也可以是一个方法，而且通过这个方法，有时候可以大幅提升性能

假设我们有一个需要复杂计算的 state 初始值：

```jsx
const [result, setResult] = useState(expensiveComputation(props));
```

我们知道每次组件更新的时候都会重新执行一次 expensiveComputation 这无疑带来了非常大的计算开销

所以贴心的 react 为我们提供了一种解决方案：

```jsx
const [result, setResult] = useState(() => expensiveComputation(props));
```

通过传入一个经过函数包裹的 expensiveComputation 后，我们就能够利用 useState 只会在组件初次渲染执行 initialization 的机制成功规避之后渲染的无用计算了

#### 总结

- useState 为我们提供了一种在 function component 中使用 state 的方法
- 如果一个组件中需要多个 state，可以调用多个 useState
- 在修改涉及上一个状态的 state 时，优先使用传入函数的方式更新 state
- 如果 state 的初始化涉及到较大计算开销，可以将其包裹在一个函数中传给 initialization

### useEffect

> useEffect 为我们提供了一种在 function component 中消费副作用的方法

#### 基本用法

```tsx
import React, { useEffect } from 'react';

/**
 * useEffect hook
 * @param effect 副作用函数，可以通过 return 返回一个清理函数
 * @param deps 依赖数组，如果传了的话 effect 只有在其中的依赖项变更才会被调用
 * @return void 没有返回值
**/
useEffect(effect: EffectCallback, deps?: DependencyList)
```

useEffect 是一个函数，它接受两个参数，返回 void

- 第一个参数 effect 是一个函数，用来执行我们需要的副作用
- effect 可以选择返回一个函数，用来清理前一次渲染的副作用
- 第二个参数 deps 是一个数组，数组中存放 effect 的依赖项，只有依赖项发生变化 effect 才会被调用
- 如果 deps 传一个空数组就表示只有第一次渲染才需要执行 effect
- 如果 deps 不传则表示每次重渲染都会执行 effect

#### 常见用法与清理副作用

useEffect 顾名思义，它的用途就是来使用 effect 的，effect 就是我们俗称的副作用，在 react 中，常见的副作用有这几种：

- 数据获取（data fetching）
- 事件监听或消息订阅（event listening or message subscription）
- 改变 DOM（changing the DOM）
- 输出日志（logging）

<br />

在使用 effect 的时候，我们希望 effect 能够随着组件的卸载而被清理掉，这样除了能够删除一些不再需要的行为，也可以防止内存泄漏。

拿上述几种 effect 来举例，我们会希望在组件卸载前把未完成的 fetching 操作给中断掉：

```js
useEffect(() => {
    const controller = new AbortController();
    const signal = controller.signal;
    // 请求一些数据
    fetch(url, { signal: signal }).then(res => { })

    return () => {
      // 在组件被卸载前把未完成的任务中止掉
      controller.abort();
    }
}, [])
```

我们也希望能把之前订阅的消息在组件卸载之前给取消了：

```js
useEffect(() => {
  function handleStatusChange(status) {
    // do something
  }
  // subscribe
  API.subscribeToFriendStatus(id, handleStatusChange);
  return () => {
    // unsubscribe
    API.unsubscribeFromFriendStatus(id, handleStatusChange);
  };
}, [id]);
```

当然，修改 DOM 或者输出日志类型的 effect 就不需要清理了，自然也不需要返回任何东西：

```js
useEffect(() => {
    // 修改 DOM
    document.title = `You clicked ${count} times`;
    // 打印日志
    console.log(`You clicked ${count} times`);
  }, [count]);
```

<br />

上面我们了解到，cleanup 函数会在组件被卸载的时候被调用，但是其实 **cleanup 会在每次更新，执行 effect 之前被调用，用来清除上一次渲染的副作用**

举个例子：

```jsx
function App() {
  const [result, setResult] = useState(0)
  useEffect(() => {
    console.log('enter effect when result equals ', result);
    return () => {
      console.log('exit effect')
    }
  }, [result]);

  return (
    <div>
      <p onClick={() => {
        console.log('onclick handler')
        setResult(prev => prev + 1)
      }}>{result}</p>
    </div>
  )
}
```

在上面的例子中，每次点击 p 标签，日志的执行顺序如下：

```js
onclick handler
exit effect
enter effect when result equals  1
```

之所以要这么设计，是因为 react 团队考虑到随着 deps 数组元素的变化，上一次渲染中 effect 依赖的元素可能也会发生变化，如果没有在每次更新的时候调用 cleanup，会造成 effect 依赖的数据过时了，让我们重新看一下之前消息订阅的例子：

```jsx
useEffect(() => {
  function handleStatusChange(status) {
    // do something
  }
  // subscribe
  API.subscribeToFriendStatus(id, handleStatusChange);
  return () => {
    // unsubscribe
    API.unsubscribeFromFriendStatus(id, handleStatusChange);
  };
}, [id]);
```

随着每次 id 的变化，effect 都会重新绑定一个新的 id，如果没有在每次 effect 重新绑定之前把上一次绑定的 id 取消订阅了，会导致无效的 id 一直被订阅着，最终导致内存泄漏或者崩溃；每次更新都调用 cleanup 可以为我们避免这个问题

#### 模拟生命周期函数

如果你熟悉 class component 的生命周期函数，不难发现 useEffect 其实就是 `componentDidMount`、`componentDidUpdate` 和`componentWillUnmount` 的组合，下面我们就分别来说说 useEffect 要怎么分别实现三种生命周期的功能

##### componentDidMount

这个生命周期函数只会在组件初次渲染的之后被调用，所以我们只需要把 useEffect 的 deps 数组置空就好

```jsx
useEffect(() => {
    console.log('[componentDidMount]: log only after mount')
}, []);
```

##### componentDidUpdate

这个生命周期函数会在组件每次更新之后被调用，useEffect 有两种方式来模拟：

- 不依赖 state 或 prop——不传 deps 数组（effect 会在初次渲染与每次更新之后被触发）

```js
useEffect(() => {
    console.log('[componentDidMount]: log only after mount')
});
```

- 依赖 state 或 prop——将对应的 state 或 prop 放入 deps 数组（只会在依赖变化时才出发 effect）

```js
useEffect(() => {
  console.log('[componentDidUpdate]: log only after "result" change')
}, [result]);
```

<br />

当然，componentDidUpdate 还时常通过 if 语句来减少 effect 的调用，useEffect 也可以：

```js
// class component
componentDidUpdate() {
  if (condition) {
    console.log('[componentDidUpdate]: log only if condition is fulfilled')
  }
}

// hook
useEffect(() => {
  if (condition) {
    console.log('[componentDidUpdate]: log only if condition is fulfilled')
  }
}, [condition]);
```

##### componentWillUnmount

这个生命周期函数只在组件要被卸载之前被调用，同样地，useEffect 可以通过在 effect 函数中返回一个 cleanup 函数来实现

区别在于 cleanup 函数会在每次 effect 调用前被调用，具体原因可以看一下 [常见用法与清理副作用](####常见用法与清理副作用)

#### 处理多个副作用

 在 class component 中，如果有多个副作用，只能把他们全部写进生命周期函数中，随着项目的发展，可能会导致组件的膨胀与难以维护；这个问题在使用 useEffect 的时候可以很好的避免

在 function component 中，如果有多个副作用，我们可以通过调用多次 useEffect 来分别处理不同的副作用

```js
// 把多个副作用放到一个 useEffect 中会导致逻辑耦合在一起不好维护
useEffect(() => {
  // manipulate DOM
  document.title = `You clicked ${count} times`;

  function handleStatusChange(status) {
    // do something
  }
  // subscribe
  API.subscribeToFriendStatus(id, handleStatusChange);
  return () => {
    // unsubscribe
    API.unsubscribeFromFriendStatus(id, handleStatusChange);
  };
}, [count, id]);

// ================================

// 将多个副作用分开多次调用 uesEffect 可以很好的分离逻辑，提升可读性
useEffect(() => {
  // manipulate DOM
  document.title = `You clicked ${count} times`;
}, [count])

useEffect(() => {
  function handleStatusChange(status) {
    // do something
  }
  // subscribe
  API.subscribeToFriendStatus(id, handleStatusChange);
  return () => {
    // unsubscribe
    API.unsubscribeFromFriendStatus(id, handleStatusChange);
  };
}, [id])
```

### useContext

> useContext 为我们提供了一种使用 Context.Consumer 的语法糖，它可以让我们跨越组件层级传递信息

#### 基本用法

```jsx
const StrContext = react.createContext('default string');

function Ancestor() {
  return (
    <StrContext.Provider value='set by Ancestor'>
      <Parent />
    </StrContext.Provider>
  )
}

function Parent() {
  return (
    <Child />
  )
}

function Child() {
  const str = useContext(StrContext);
  return (
    <div>
      <p>the context is *{str}*</p>
    </div >
  )
}
```

useContext 时一个比较特殊的 hook，如果熟悉 context 的小伙伴应该很快就能发现它其实就是 `Context.Consumer` 的语法糖

context 最大的用途在于摆脱 props 的层层传递关系，为我们提供了跨层组件之间的通信，我们将在后面的 [Context](####Context) 章节中详细介绍 context，让我们先了解它要怎么用吧：

- 首先我们需要调用 react 中的 `createContext` 方法，来创建一个 context
- `createContext` 的时候可以传入 context 的默认值，如果后面没有 provider 更新 context 的值的话一律使用这个默认值
- 有了 context 之后，我们可以在顶层的组件中使用 provider 来更新 context 的值
- 对于想要消费这个 context 的组件，我们可以用 `useContext` 并将 context 当作参数传入
- `useContext` 的返回值就是我们 context 中的值，在当前组件中我们可以把它当成 props 或者 state 来正常使用
- 如果一个 context 有多个 provider，context 的值由最近的一个 provider 提供

<br />

了解了基本用法之后，我们将会开始介绍 useContext 背后的概念 context，看完这一节你将会学到：

1. context 是什么？
2. 最适合 context 使用的场景有哪些？
3. 如何解决 context 多余渲染的问题？

#### Context

> context，顾名思义，就是上下文的意思，也就是说它提供了一种组件版本的 “全局变量”

想要了解我们为什么需要 context，以及什么时候用 context，我们必须先从目前的痛点入手

让我们回顾一下上面 “基本用法” 中的例子，如果我们不用 context，根据 react 组件由上自下的单向数据流，我们需要将字符串通过 props 层层传递，最后才能到达消费这个属性的子组件中

在这个过程中，Parent 组件完全不关心这个属性，但是要负责 “传话”，这个明显不符合组件之间 ”高内聚，低耦合“ 的原则

比较理想的情况应该是：**上层组件 Ancestor 把这个属性 “抛” 到一个空间中，再让想要用这个属性的子组件从空间中 “抓取” 下来**

上面这句话中，空间指的就是 context；抛这个动作可以是 createContext 的初始值，也可以是 Provider；抓取这个动作对应的就是 Consumer

接下来，我们开始介绍所有 context 的起点：react.createContext 方法

##### createContext

> createContext 是 react 提供的一个 api，它的作用就是创建一个 context

让我们来看一下 createContext 的函数原型：

```typescript
interface Context<T> {
  Provider: Provider<T>;
  Consumer: Consumer<T>;
  displayName?: string | undefined;
}
function createContext<T>(defaultValue: T,): Context<T>;
```

可以看到，createContext 接受一个初始值 defaultValue，并且返回一个 context 对象

这个**初始值只有在一个组件没有任何对应 provider 的时候才会被使用**，其他任何时候都会优先使用最近的 provider 提供的值（包括这个 provider 提供了 null、undefined 等一系列空值）

createContext 返回的 context 有三个属性，其中 provider 跟 consumer 我们后面介绍，displayName 可以控制 React Devtools 的 Components 面板展示 context 的名字：

```js
StrContext.displayName = 'NewContextName'
```

![8-1](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/8-1.jpg)

##### Provider

> Provider 是 context 的其中一个属性，它的作用是为它的后代指定 context 的值

让我们来看一下 Provider 的用法：

```jsx
<Context.Provider value={/* new value */}>
	{/* descendants */}
</Context.Provider>
```

可以看到，provider 一般用来包裹想要接收这个 context 的后代组件，它有一个属性 value 可以为所有的后代节点更新当前 context 的值

<br />

provider 可以被嵌套，**后代组件中 context 的值取决于离它最近的 provider 的值**，举个例子：

```jsx
<Context.Provider value={/* new value1 */}>
  <Context.Provider value={/* new value2 */}>
    {/* context's value equal to "new value2" */}
		{/* descendants */}
	</Context.Provider>
</Context.Provider>
```

<br />

当 provider 的值被更新的时候，其后代中有消费这个 context 值的组件都会被强制更新（强制更新指的是他们不受自己或自己祖先的 `shouldComponentUpdate` 影响），举个例子：

```
Ancestor(Provider)
	|						\
Parent			Uncle
	|
Child(Consumer)
```

上图是一个组件树，Ancestor 作为组件的共同祖先，同时也是 context 的 provider；除了 Ancestor 之外的所有组件都带上了

```js
shouldComponentUpdate() {
  return false;
}
```

的生命周期，但是当 context 中的值被改变的时候，除了 Ancestor 触发了重新渲染，Child 也会被触发重新渲染

##### Consumer

> Consumer 是 context 中的一个属性，它的作用是为组件声明该组件要消费这个 context

Consumer 的用法总共有三种，除了上述专为 function component 提供的 useContext 之外，还有一种专门给 class component 的用法：

```jsx
class App extends react.Component {
  // 将要使用的 context 赋值给 class component 的静态属性 contextType
  static contextType = Context;
  render() {
    return (
      // 就可以直接在代码中用 this.context.xxx 来使用了
      <p>The context is *{this.context.value}*</p>
    )
  }
}
```

还有一种通用用法：

```jsx
const App = () =>
  <Context.Consumer>
    {
      value =>
        <p>The context is *{value}*</p>
    }
  </Context.Consumer>

// or...
class App extends React.Component {
  render() {
    return (
      <Context.Consumer>
        {
          value =>
            <p>The context is *{value}*</p>
        }
      </Context.Consumer>
    );
  }
}
```

可以看的出来，最后一种通用写法会让代码显得有些浑浊，所以这就是为什么前面会说useContext 本质上是 Context.Consumer 的语法糖的原因了

另外需要注意的一点就是，如果有小伙伴好奇的把 context 打印出来，应该能看到一个 **_currentValue** 的属性，并且会发现 context 的值本质上就是取的这个属性，但是这并不代表我们就可以抛弃 consumer 直接从 context 中取出值来用

一个很重要的点是，consumer 并不只是获取 context 的值而已，它还为每一个使用它的组件上“订阅”了 context（这里使用括号是因为本质上 react 并不是用发送消息的方式来通知组件更新，在这里只是形象的一种描述而已）的更新，并在后续 context 的值发生变更时强制触发渲染

#### Context 的使用场景

> 在这一节我们将介绍 context 的最佳使用姿势以及不需要使用的场景

首先还是回到我们一开始的那个例子：

```jsx
function Ancestor() {
  const [value, setValue] = useState(0)
  return (
    <Parent value={value} />
  )
}

function Parent(props) {
  const { value } = props;
  return (
    <Child value={value} />
  )
}

function Child(props) {
  const { value } = props;

  return (
    <div> The value is *{value}*</div>
  )
}
```

 特别简单的三个组件，`Ancestor` 的 state 通过 `Parent` 组件的中转，传递给 `Child` 组件

上面我们拿这个来当例子是因为这个例子足够的简单以及好理解，但是实际上这种情况除了使用 context 之外，还有一种方式：

```jsx
function Ancestor() {
  const [value, setValue] = useState(0);
  // 直接将 Child 组件当作属性传递
  const child = <Child value={value} />
  return (
    <Parent child={child} />
  )
}

function Parent(props) {
  const { child } = props;
  return child;
}

function Child(props) {
  const { value } = props;
  return (
    <div> The value is *{value}*</div>
  )
}
```

可以看到，上面这种方式直接将 `Child` 组件提升到了 `Ancester`，并且当作 props 传递给 `Parent`，这种依赖反转的方式在需要传递的 props 比较多的情况下能够有效的减少 props 的数量，而且这种思路也可以进行拓展，比如说父组件可以留下多个插槽（slot）：

```jsx
function Page() {
  const [theme, setTheme] = useState('light');
  const left = <Left value={theme} />
  const right = <Right value={theme} />
  return (
    <Layout left={left} right={right} />
  )
}
```

当然，在工程的世界中没有银弹，这种方式会导致更高层级的组件变得更加复杂，并且对插入的组件的灵活性有着更高的要求，所以在选择任何方案之前一定要衡量好其中的利弊

<br />

 接下来我们来探讨一下最适合 context 的场景是什么

从上面的章节我们可以了解到，context 提供了一种组件的 “全局” 能力，并且通过 provider，我们可以精准控制任何组件树枝干的 context 值

根据上面的描述，我们可以总结出 context 的两个特点：

1. 对于子组件众多的大组件树，context 提供了一种无视层级的属性能力
2. 对于特殊的子组件，可以定制 context 的值

根据这两个特性，有两个场景就特别适合：全局的亮暗主题、全局的语言偏好

#### Context 多余渲染问题

> 这一节我们来讨论一下 context 在什么时候会存在多余渲染以及避免多余渲染的三种方式

首先我们先从一个例子入手：

```jsx
const Context = react.createContext({});

function Parent() {
  const [value, setValue] = useState({
    a: 1, b: 1
  })
  return (
    <Context.Provider value={value}>
      <button onClick={() => setValue(prev => { return { ...prev, b: prev.b + 1 } })}>hit me!</button>
      <Child />
    </Context.Provider >
  )
}

const Child = react.memo(() => {
  const { a } = useContext(Context);
  return (
    <VeryExpensiveTree value={a}>
  )
})
```

我们可以看到 `Child` 组件消费了 context 中的属性 a，而 `Parent` 组件提供了一个按钮可以更新 context 中的属性 b，在按下按钮更新的过程中，除了组件 `Parent` 触发重渲染之外，组件 `Child` 也触发了一次重渲染（react.memo 在这个 case 不生效）

从上述例子可以看到，组件 `Child` 每次重渲染都需要很昂贵的计算，这些多余的渲染明显造成了严重的性能损耗，这就是我所想要讨论的多余渲染问题

关于这个问题，有三种解决思路：

##### 拆分 context（推荐）

这种方法简单粗暴，如果一个 context 中耦合了太多的数据，我们直接把他们拆开成不同的 context 就完事了，比如说：

```jsx
const Context1 = react.createContext();
const Context2 = react.createContext();

function Parent() {
  const [value1, setValue1] = useState(1)
  const [value2, setValue2] = useState(1)
  return (
    <Context1.Provider value={value1}>
      <Context2.Provider value={value2}>
        <button onClick={() => setValue2(prev => prev + 1)}>hit me!</button>
        <Child />
      </Context2.Provider>
    </Context1.Provider>

  )
}

const Child = react.memo(() => {
  const value = useContext(Context1);
  return (
    <VeryExpensiveTree value={value}>
  )
})
```

在这个 case 里，`Child` 组件使用的 context 没有被更改，而且因为使用了 react.memo，从而 “屏蔽” 了重渲染

##### 拆分组件

如果我们在开发的项目对于拆开 context 有比较大的难度时，可以考虑拆分对应的组件，比如说：

```jsx
const Context = react.createContext({});

function Parent() {
  const [value, setValue] = useState({
    a: 1, b: 1
  })
  return (
    <Context.Provider value={value}>
      <button onClick={() => setValue(prev => { return { ...prev, b: prev.b + 1 } })}>hit me!</button>
      <Child />
    </Context.Provider >
  )
}

function Child() {
  const { a } = useContext(Context);
  return (
    <ExpensiveTreeWrapper value={a} />
  )
}

const ExpensiveTreeWrapper = react.memo(({ value }) => {
  return <VeryExpensiveTree value={value} />
})
```

从上面的例子可以看到，我们在 `VeryExpensiveTree` 组件的外面包了一层，并且将 context 用 props 的形式传给了 `VeryExpensiveTree`，通过 props，我们就可以完美的利用 memo 的机制来减少多余渲染啦

##### 使用 useMemo

最后，我们可以使用 useMemo 这个新的 hooks 来避免多余渲染（具体的介绍会在后面章节补充）：

```jsx
const Context = react.createContext({});

function Parent() {
  const [value, setValue] = useState({
    a: 1, b: 1
  })

  return (
    <Context.Provider value={value}>
      <button onClick={() => setValue(prev => { return { ...prev, b: prev.b + 1 } })}>hit me!</button>
      <Child />
    </Context.Provider >
  )
}

function Child() {
  const { a } = useContext(Context);
  return useMemo(() => {
    return (
      <VeryExpensiveTree value={a} />
    )
  }, [a])
}
```

这样一来，虽然每次更新的时候仍然会执行 Child 函数，但是 useMemo 里面的 `VeryExpensiveTree`  是不会触发重渲染的

### useReducer

> useReducer 在复杂且有关联的组件状态变更场景中为我们提供了比 useState 更好的替代方案

#### 基本用法

```jsx
const initialState = {
  count: 1
}

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error('Unknown action');
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState)
  return (
    <>
      <div>Count: {state.count}</div>
      <button onClick={() => dispatch({ type: 'incremen' })}>+1</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-1</button>
    </>
  )
}
```

可以看到，useReducer 的用法跟 useState 非常的相似，它接收两个参数，并且返回一个数组

- 第一个参数 reducer 是一个方法，利用 action 提供的信息处理 state 的更新
- 第二个参数 initialState 制定了初始化时 state 的值
- 返回的数组中，第一个参数是一个 state，第二个参数则是用来更新 state 的 dispatch 方法
- dispatch 方法有一个 action 对象作为参数，action 通常由 type 与 payload 构成，前者定义操作类型，后者携带操作参数

#### 常见用法以及与 useState 的区别

看了上面的例子，我们不难发现 useReducer 与 useState 的功能相差无几，甚至把上面的例子换成 useState 不仅代码更短，还更好理解，但是存在即合理，我们先从它与 useState 的不同出发，慢慢挖掘 useReducer 的潜力

首先最直观的一点就是：有别于 useState 直接将 state 变更方式写在组件中的形式，useReducer 在组件中只关注要**对 state 做什么**

换言之，**useReducer 解耦了 what（做什么）和 how（怎么做）**

上面的例子可能没办法体现出 useReducer 的优势，我们再举一个例子：

假设现在我们需要一个表格，表格的数据是从后端接口获取的，且支持分页以及关键字搜索的操作，如果用 useState 来写我们需要先定义 state：

```jsx
const [pagination, setPagination] = useState({ page: 1, pageSize: 10 });
const [filter, setFilter] = useState({ keyword: "" });
const [isLoading, setIsLoading] = useState(false);
```

再来我们需要定义不同的操作：

```jsx	
// 设置当前的页数
const setPage = page => setPagination(prev => { return { ...prev, page } });
// 设置要搜索的关键词
const setKeyword = keyword => setFilter(prev => { return { ...prev, keyword } });
// 根据关键词搜索
const search = () => {
  setIsLoading(true);
  // 异步获取数据成功后更新
  fetchTableData(filter).then(() => {
    setPage(1);
    setKeyword('');
    setIsLoading(false);
  })
}
```

可以看到，我们在处理 state 的时候需要关心的细节太多了，尤其是成功获取数据之后，一口气需要更新三个 state，这个无疑加大了后期维护这段代码的心智负担，让我们再来看看 useReducer 的版本吧：

```jsx
const initialState = {
  pagination: { page: 1, pageSize: 10 },
  filter: { keyword: "" },
  isLoading: false,
}

function reducer(state, action) {
  const payload = action.payload;
  switch (action.type) {
    case 'CHANGE_PAGE':
      return {
        ...state,
        pagination: {
          ...state.pagination,
          page: payload.page
        }
      };
    case 'SET_KEYWORD':
      return {
        ...state,
        filter: {
          ...state.filter,
          keyword: payload.keyword
        }
      };
    case 'CLEAR_KEYWORD':
      return {
        ...state,
        filter: { keyword: '' },
        pagination: { ...state.pagination, page: 1 }
      };
    case 'SEARCH':
      return {
        ...state,
        isLoading: true,
      };
    case 'SUCCESS':
      return {
        ...state,
        isLoading: false,
        filter: {
          keyword: ''
        },
        pagination: {
          ...state.pagination,
          page: 1
        }
      };
    default:
      throw new Error('Unknown action');
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, initialState)

  // 一些 UI
	return <>...</>
}
```

可以看到，虽然代码变长了，但是所有的操作 state 的方法都被一个 action 给包装起来了，这样一来，我们只需要在需要的时候通过 dispatch 的方式告诉 reducer 我需要做什么并且把对应的值给到就行，比如：

```jsx
// 设置关键词  
dispatch({ type: 'SET_KEYWORD', payload: { keyword: "Shakespeare" } });

// 搜索
const search = () => {
  dispatch({ type: 'SEARCH' });
  fetchTableData(state.filter).then(() => {
    dispatch({ type: 'SUCCESS' });
  })
}
```

通过把“指令”与“操作”解耦的方式，我们将复杂的 state 操作抽象成了一个个指令，后面如果有遇到相关的操作，我们就只需要发出指令而不需要关注额外的细节了

所以，**对于复杂的 state 操作，或者是嵌套的 state 对象，推荐使用 useReducer**

<br />

除了上述的场景之外，还有一个场景是我们可以优先考虑使用 useReducer 的，那就是单元测试

首先，我们知道 react 的页面重渲染是由 props 和 state 驱动的，其中 useReducer 通过 action 很好的封装了 state 的变化

其次，在 react 中，state 的更新都必须要是 immutable 的，通过这个原则，我们可以得知 reduce 这个函数本质上就是一个纯函数，也就是说，对于任何相同的输入，我们都会得到相同的输出

基于以上两点，我们可以根据 reduce 函数的输出来判断 state 的变化，进而达到测试的目的

##### 结论

- 对于复杂的 state 操作，或者是嵌套的 state 对象，推荐使用 useReducer
- 如果对组件有单测的需求，推荐使用 useReducer

#### 惰性初始化 Lazy initialization

跟 useState 一样，useReducer 也支持惰性初始化，我们可以通过给 useReducer 传入第三个参数 init 方法来实现：

```jsx
function init(initialCount) {
  return {
    count: initialCount,
  }
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init)
  return <>...</>
}
```

可以观察到，init 方法会自动将第二个参数 initialCount 传入，并返回一个初始化好的 state

使用惰性初始化的场景主要有两个：

1. 如果初始化这个操作本身是一个昂贵的操作，而你不希望它在卖次 rerender 的时候都被执行一次的时候，可以用惰性初始化，它只会在第一次渲染的时候被执行
2. 我们可以利用 init 函数来还原 state 的初始值，比如

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'reset':
      return init(action.payload);
    default:
      throw new Error('Unknown action');
  }
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init)
  return (
    <button
      onClick={() => dispatch({type: 'reset', payload: initialCount})}>
      Reset
    </button>
  )
}
```

#### 自己写一个 redux

redux 是诞生于 react 社区，为了解决 react 状态管理问题而设计开发的一个库

对于 redux 不熟悉的小伙伴，我这里一句话总结 redux 的功能：

**redux 提供了一个应用程序级别的全局状态管理，用户可以通过 UI dispatch 一个 action 给 reducer，reducer 会对 store 中的数据进行更新操作，而更新了的数据会触发 UI 的重渲染，从而实现页面数据的更新**，整个流程如下图所示。

![redux](https://www.freecodecamp.org/news/content/images/2022/06/2.png)

相信认真看完我上面对 useReducer 介绍的小伙伴这个时候已经想到了，这个流程不就跟 useReducer 的流程一模一样吗？

对此我只能说，很像，但不完全一样，区别在于 **useReducer 没有提供一个全局状态管理的功能**

既然没有我们就自己写一个呗，至于怎么写？答案是 Context

我们来一句话回顾一下 Context 的功能：它提供了一种组件版本的 “全局变量”，允许我们跨越组件层级传递参数，并且能通过参数的变更触发重渲染逻辑

那有了 Context 跟 useReducer，我们可以来着手写一个自己的 redux 啦～

秉持这最简化原则，我们还是那一开始的 Counter 组件来举例子，毕竟谁能拒绝一个简单的 Counter 组件呢 ;)

```jsx
const Context = react.createContext();

function Counter() {
  // 我直接省略了 reducer initialState 的实现，这两者的实现在一开始的基本用法已经给出
  const [state, dispatch] = useReducer(reducer, initialState)
  return (
    <>
      <Context.Provider value={{ state, dispatch }}>
        <div>Count: {state.count}</div>
        <Operator />
      </Context.Provider>
    </>
  )
}

function Operator() {
  const dispatch = useContext(Context);
  return (
    <>
      <button onClick={() => dispatch({ type: 'increment' })}>+1</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-1</button>
    </>
  )
}
```

可以看到，我们通过将 state 跟 dispatch 方法放入 Context（类比 redux 中的 store）模仿了一个 redux 出来：

我们提供了一个跨组件的全局状态管理（Context），用户可以通过 UI dispatch 一个 action 给到 reducer，reducer 会对 store（这里是 Context）中的数据进行更新操作，而更新了的数据会触发 UI 的重渲染（通过 Context 的机制），从而实现页面数据的更新

完美对上了前面对 redux 的描述。

#### redux 能被取代吗

在上一个小节，我们说了如何用 useReduer 跟 Context 实现一个简易版的 redux

紧接而来的是好几个问题：我们实现的 redux 能够完美替代 redux 吗？这两者之间到底有什么区别呢？

首先第一个区别还是要从 Context 的缺点开始讲起——**Context 有多余渲染的问题**

当然，我们也在 useContext 的介绍中提到了三种避免多余渲染问题的解决方案，但是不论是拆分 context、拆分组件，抑或是使用 useMemo，都无法避免的对原来的代码进行侵入性修改

反观 redux，它将组件分为了容器型组件以及展示型组件，对于容器型组件需要使用 connect 函数生成，并通过 mapStateToProps 来过滤全局状态，在 connect 函数内部，对生成的容器型组件提供了一个在 shouldComponentUpdate 中对 state 跟 props 的浅比较，杜绝了多余渲染的发生

第二个区别，则是在于**我们自己实现的 redux 不能支持副作用（side effect）**

首先必须声明一点， redux 本身也是不支持副作用的，但是它提供了一个中间件机制，让社区开发者来帮它拓展功能，现在也有很多优秀的副作用处理解决方案了，比如redux-thunk、redux-promise、redux-saga等。

那么，我们自己写的 redux 就完全没有意义了吗？我们应该直接用 redux 吗？

俗话说的好，梭哈是一种艺术，但是这个时候显然不是展现这种艺术的时候

毕竟连人家 redux 的共同开发者 Dan Abramov 都说了：

> People often choose Redux before they need it.

在这里我给出我个人的观点：**建议在需要管理大型应用的全局状态时，再使用 redux；如果只需要处理小范围内的组件状态，可以考虑优先使用 Context + useReducer 的组合**

### useCallback

> useCallback 提供了一种通过 **保存方法引用** 来减少不必要的渲染的性能优化方案

#### 基本用法

```jsx
const memorizedFn = useCallback(fn, []);
```

可以看到，useCallback 主要接收两个参数，并返回一个新的方法

- 第一个参数是一个方法，它就是需要被保存引用的方法
- 第二个参数则是一个数组，数组中存放的是第一个参数中方法的依赖项，只有当数组中的依赖项变更时，该方法的引用才会被重新更新
- 返回值则是一个被保存了引用的方法

#### 使用场景

想要了解为什么我们需要在 react 中保存方法的引用，我们用一个例子来回顾一下 react 触发重新渲染的逻辑

```jsx
function Counter() {
  const [count, setCount] = useState(1)
  const handleClick = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <div>{count}</div>
      <button onClick={() => handleClick()}>+1</button>
    </div>
  );
}
```

这个是一个我们的老朋友 Counter 组件，当 button 每次被按下的时候，由于 count 状态被更新了，所以会触发整个组件的重渲染，在这个过程中，会重新执行一次 Counter 方法，于是，第三行的 handleClick 方法就会在每次的重渲染之前被执行并创建一个新的方法

而这个过程，也使得 setCount 方法中的 count 变量每次都能变成上次更新后的 count 值的原因

如果我们用 useCallback 方法把 handleClick 包一层的话，就会变成这样

```jsx
function Counter() {
  const [count, setCount] = useState(1)
  // 用了 useCallback 保存了方法的引用，并且将第二个参数置空
  const handleClick = useCallback(() => {
    setCount(count + 1);
  }, []);

  return (
    <div>
      <div>{count}</div>
      <button onClick={() => handleClick()}>+1</button>
    </div>
  );
}
```

在这个例子中，由于我们用 useCallback 保存了原来的 handleClick 方法的引用，我们会发现当这个组件点击了第一次成功更新 count 值之后，无论怎么点击 button 都不会更新 count 了

这个是因为当第一次点击按钮并触发重渲染之前，执行到 handleClick 方法时，因为 useCallback 的依赖项没有发生变化，所以返回了第一次创建的方法，而在之前的方法中， count 这个变量的值是 1，而不是更新后的 2

相当于在 handleCallback 方法中的 setCount 一直执行的都是 setCount(2) 这个语句，所以无论之后点击了多少次按钮，都不会出发点重新渲染了

<br />

在知道了 useCallback 保存引用的效果之后，我们用一个具体的例子来说明如何使用这个特性来减少子组件不必要的渲染

```jsx
function Counter() {
  const [count, setCount] = useState(1)
  // 不用 useCallback
  const handleClick = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <div>{count}</div>
      <button onClick={() => handleClick()}>+1</button>
      {/* 将 handleClick 的引用传入 Child 组件，因为每次更新的时候引用都会发生改变，所以 Child 组件在每次点击之后都会触发重渲染 */}
      <Child handleClick={handleClick} />
    </div>
  );
}

// 使用 memo 对 props 进行浅比较
const Child = memo(({ handleClick }) => {
  console.log('Child')
  return <button onClick={handleClick}>Child</button>
})
```

在上面的例子中，我们就算用 memo 对 Child 组件进行了包裹，还是无法阻止子组件的重渲染，这是因为 memo 只是对传入组件中的 props 进行了一个浅比较的操作，而 Counter 组件每次的重渲染操作都会重新创建一个 handleClick 方法，从而导致传入 Child 组件的引用发生了改变，因而造成了子组件的重渲染

而解决方法也很简单，只需要给 handleClick 加上 useCallback 就好

```jsx
function Counter() {
  const [count, setCount] = useState(1)
  // 用 useCallback 保存了方法的引用
  const handleClick = useCallback(() => {
    // 用传入方法的方式更新 state 避免了上面无法触发更新的尴尬局面
    setCount(prev => prev + 1);
  }, []);

  return (
    <div>
      <div>{count}</div>
      <button onClick={() => handleClick()}>+1</button>
      <Child handleClick={handleClick} />
    </div>
  );
}

const Child = memo(({ handleClick }) => {
  console.log('Child')
  return <button onClick={handleClick}>Child</button>
})
```

完美解决～

但是！在结束这一小节之前，我还必须要知道一个概念：**任何性能优化总是伴随着额外的成本的**。

就拿上面的例子来说，虽然我们用 useCallback + memo 的方式减少了子组件没必要的渲染，但与此同时，我们增加了记忆函数引用的成本以及浅比较的成本

而单论上面的例子而言，其实完全没必要使用这一套优化方案，因为 setState 方法本身就是 memorized 了：

```jsx
function Counter() {
  const [count, setCount] = useState(1)
  const handleClick = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);

  return (
    <div>
      <div>{count}</div>
      <button onClick={() => handleClick()}>+1</button>
      {/* 直接传递 setState 也能达到一样的效果 */}
      <Child handleClick={setCount} />
    </div>
  );
}

const Child = memo(({ handleClick }) => {
  console.log('Child')
  return <button onClick={() => handleClick(prev => prev + 1)}>Child</button>
})
```

关于什么时候可以使用 useCallback 性能优化，可以看看这篇文章：[When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)

#### 总结

- useCallback 可以保存方法引用，在需要引用相等的场景下，如 memo，pureComponent，以及 shouldComponentUpdate 时可以避免重复渲染进而优化性能
- 在使用 useCallback 优化性能之前，请谨慎评估并实际测量，因为贸然使用而产生的成本很可能最终大于优化后的收益

### useMemo

> useMemo 提供了一种通过 **保存方法计算结果** 来减少不必要的昂贵计算的性能优化方案

#### 基本用法

```jsx
const memorizedValue = useMemo(fn, []);
```

咋看之下，useMemo 好像跟 useCallback 没什么两样，它们最大的区别在于，useMemo 保存的是方法执行的结果

- 第一个参数依旧是一个方法，但与 useCallback 不同的是，在**第一次调用的时候它就会被执行，并返回结果**
- 第二个参数依旧是一个数组，保存着第一个参数方法的依赖项，只有依赖项发生变更时，才会导致第一个方法被重新执行
- 返回值则是第一个参数的返回结果，这个结果被保存了起来，只有当依赖项发生变更时才会被重新计算

<br />

简单来说，假设有这么一个 useCallback 方法：

```jsx
const memorizedFn = useCallback(fn, []);
```

与其等价的 useMemo 方法可以写成：

```jsx
const memorizedFn = useMemo(() => fn, []);
```

#### 使用场景

在上一节 useCallback 中我们聊到，useCallback 保存方法引用的特性使得它适合用在需要浅比较的场景中

与之相对的，useMemo 保留方法执行结果的特性无疑是非常**适合需要昂贵的计算代价的场景**

举个例子：

```jsx
function Counter() {
  const [count, setCount] = useState(1)
  const doSomethingExpensive = () => {
    // 假设这里有一个很昂贵的计算
    return { result: 'done! ;)' }
  }

  return (
    <div>
      <div>{count}</div>
      <button onClick={() => setCount(prev => prev + 1)}>+1</button>
      {/* 这里会调用那个昂贵的计算并且将计算结果返回给子组件 */}
      <Child value={doSomethingExpensive()} />
    </div>
  );
}

const Child = memo(({ value }) => {
  console.log('Child')
  return <div>{value.result}</div>
})
```

上面的例子在我们每次点击 +1 按钮的时候都会在 13 行重新计算一次 doSomethingExpensive 的值并且返回一个全新的对象，因此 Child 组件也会被不断的重渲染

但是如果我们使用了 useMemo：

```jsx
function Counter() {
  const [count, setCount] = useState(1)
  // 用 useMemo 保存了计算结果
  const doSomethingExpensive = useMemo(() => {
    // 假设这里有一个很昂贵的计算
    return { result: 'done! ;)' }
  }, []);

  return (
    <div>
      <div>{count}</div>
      <button onClick={() => setCount(prev => prev + 1)}>+1</button>
      {/* 这个时候 doSomethingExpensive 就不是一个方法了，而是被记忆了的计算结果 */}
      <Child value={doSomethingExpensive} />
    </div>
  );
}

const Child = memo(({ value }) => {
  console.log('Child')
  return <div>{value.result}</div>
})
```

在使用了 useMemo 后，我们返回了一个对象给 doSomethingExpensive，并且只要 useMemo 的依赖项不发生改变，doSomethingExpensive 的引用也不会发生改变，所以点击按钮再也不会造成子组件渲染了

完美解决～

<br />

但是！老话一句：**任何性能优化总是伴随着额外的成本的**。

使用 useMemo 需要的成本主要有两个：保存缓存，以及计算依赖项变化

所以如果真的在代码里写了很多像上面那种 “昂贵” 的例子，很可能带来的是负优化

真正适合使用 useMemo 的场景主要有两个：

- 缓存真正昂贵的计算结果
- 当子组件被 memo 包裹并且依赖于某个父组件的计算属性时

其中第二点需要展开一下，并不是所有满足第二点条件的情况使用 useMemo 都能得到正向优化，在确定要使用之前，请谨慎评估并实际测量优化的效果

### useRef

> useRef 为我们提供了一个在组件生命周期内持久化保存不会触发渲染的对象的方法

#### 基本用法

```jsx
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  console.log('1', inputEl)
  const onButtonClick = () => {
    console.log('2', inputEl)
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

useRef 是一个方法，它接受一个参数，并返回一个对象：

- 接收的参数被称为 initialValue，用来初始化返回的对象的内容
- 返回的对象比较特殊，它有且仅有一个属性 current，所有的 initialValue 都会被赋值给 current 属性，所以第三行的输出会是 `{current: null}`
- 当代码执行到第 10 行时，我们将 inputRef 传入了 ref 属性中，ref 属性会将 inputEl 的 current 属性赋值为当前真实挂载的 input dom 元素，所以我们按下按钮后得到的输出会是 input 元素的 dom属性

需要特别注意的一点是，useRef 返回的 ref.current 不像 props 或 state，它是 **mutable** 的，且**改变它的值并不会触发重渲染**

#### ref 属性

在 react 中，有着典型的单向数据流

但是在有些时候我们希望能绕过这个机制自己去操作组件实例或甚至是 dom 元素实例

而 ref 属性，就是帮我们达到这一目的的工具

<br />

以下几个典型案例是使用 ref 的好时机：

- 管理焦点，文字选择或控制媒体播放
- 触发强制动画
- 集成第三方 dom 库

<br />

ref 属性总共有三种使用方法，这三种方法都能让我们获取到组件或 dom 的实例，下面我们按照推荐程度来一一介绍：

##### useRef & createRef

我们先从例子入手，看看该如何使用这两个方法，后面再介绍他们两之间的区别

useRef 的例子在基本用法中有说明了，我们来看看 createRef 要如何使用：

```jsx
class Counter extends react.Component {
  constructor() {
    super();
    //  创建一个对象，对象中有唯一一个属性 current，current 目前为 null
    this.inputEl = react.createRef();
  }

  render() {
    return (
      <div>
        {/* 通过 ref 属性，给对象中的 current 属性赋值为当前 dom 的实例 */}
        <div ref={this.inputEl} onClick={() => console.log('ref', this.inputEl)}>welcome</div>
      </div>
    )
  }
}
```

可以看到，两者的使用方法不说完全一样，也是如出一辙

那么就有一个有趣的问题了：这两个有什么区别呢？我们可以在 Function Component 中使用 createRef；在 Class Component 使用 useRef 吗？

答案是不行。

<br />

首先先来看看 Function component 的表现吧：

```jsx
function Counter() {
  const [count, setCount] = useState(1)

  const ref = react.createRef();
  console.log('1', ref);

  return (
    <div>
      <div ref={ref}>{count}</div>
      <button onClick={() => {
        setCount(prev => prev + 1)
        console.log('2', ref)
      }} >+1</button>
    </div>
  );
}
```

可以看到，随着每次 button 的 click，ref 都会因为第 5 行的执行被清空，并且在第 9 行时被重新赋值

由此可知，createRef 会随着 Counter 组件的 rerender 被重复执行；但 useRef 由于有 react 团队提供的机制，保证了**在组件生命周期内只会被调用一次**

而在 Class component 中调用 hooks 本身就是不被允许的。

所以，当我们要使用 ref 获取组件或 dom 实例时，如果**使用的是 Class component 推荐使用 createRef**；如果**使用的是 Function component 则推荐使用 useRef**

##### callback refs

除了一个对象之外，ref 属性还接收一个方法，这个方法会在 ref 属性所在组件或 dom 被挂载后或卸载前被调用

此外，react 保证了这个方法会在 componentDidMount 或 componentDidUpdate 之前被调用

该方法接收一个参数，代表组件或 dom 的实例

看看例子：

```jsx
class Counter extends react.Component {
  constructor() {
    super();
    this.state = {
      count: 1,
    }
    console.log('constructor', this.inputEl);
  }

  componentDidMount() {
    // 在 componentDidMount 中每次都能获取到最新的 ref
    console.log('componentDidMount', this.inputEl);
  }

  componentDidUpdate() {
		// 在 componentDidUpdate 中每次都能获取到最新的 ref
    console.log('componentDidUpdate', this.inputEl);
  }

  render() {
    return (
      <>
      	{/* 当 count >= 3 时，div 元素就会被卸载 */}
        {this.state.count < 3 && 
      		<div ref={(el) => {
    				this.inputEl = el
    				console.log('ref', this.inputEl)
  				}}>
        		{this.state.count}
      		</div>
      	}
        <button onClick={() => {
          // 通过更新 state，我们在每次按钮被点击后都会将上面的 div 重新渲染
          this.setState({ ...this.state, count: this.state.count + 1 })
        }} >+1</button>
      </>
    )
  }
}
```

下面展示的是从开始渲染到点击一次按钮后的日志输出：

```
constructor 				undefined
ref 								<div>1</div>
componentDidMount 	<div>1</div>
// =======> button clicked
ref 								null
ref 								<div>2</div>
componentDidUpdate 	<div>2</div>
// =======> button clicked
ref 								null
componentDidUpdate 	null
```

首先在一开始的构造函数中，inputEl 肯定是 undefined（日志第 1 行），然后 render 函数被执行，当 div 元素被挂载到 dom 树之后，callback 被执行了，inputEl 也被成功赋值为 div 的 dom 实例（日志第 2 行），在 callback 执行完之后，componentDidMount 才匆匆来迟的被执行，打印出 inputEl 获取到的实例（日志第 3 行）

第二个部分就是我们第一次点击按钮后了：点击按钮后首先通过 setState 更新了 state 的值，这个操作触发了 div 元素的重渲染操作，重新执行了一次 render 方法，在这个方法中，由于我们 callback 用的是内联函数，所以函数的实例被销毁，inputEl被赋值为 null（日志第 5 行），然后被重新创建重新赋值 dom 实例（日志第 6 行），最后更新完之后 componentDidUpdate 被调用（日志第 7 行）

在这里插一个题外话，如果不像让日志 5，6 行被执行，我们可以把内敛函数换成跟 class 实例绑定的函数，这样在执行 render 函数的时候就不会被销毁或重新创建了，当然这个实际上对代码表现跟性能都不会有太大影响，实际开发中还是比较常用内联函数的写法

第三个部分就是我们第二次点击按钮，这次跟第二次最大的区别在于，点击完按钮 count 的值会被更新到 3 ，而上面实例代码的第 24 行会在 count 大于或等于 3 的时候将 div 元素从 dom 树上销毁，所以我们可以看到 callback 在销毁之前被执行了，且 inputEl 被赋值为 null（日志第 9 行），最后的 componentDidUpdate 打印出来的自然也是 null（日志第 10 行）

##### 字符串 ref（Legacy api）

最后一种方法是最不推荐使用的，在文档与代码中已经被标注为 Legacy API 并且在代码运行时会有警告提醒

如果像深入了解为什么这个不被推荐使用，可以看看这篇[文章](https://github.com/facebook/react/pull/8333#issuecomment-271648615)

<br />

最后，需要提醒的一点是，虽然我在前面说 ref 可以获取 react 组件或者 dom 元素的实例，但是其实 ref 是无法获取到 Function component 的实例的，因为 Function component 本身并没有实例，如果想在 Function component 中使用 ref，可以用我们之后会介绍的 [useImperativeHandle](https://reactjs.org/docs/hooks-reference.html#useimperativehandle) hook 搭配 [forwardRef](https://reactjs.org/docs/forwarding-refs.html) 使用，或者将你的组件转换为 Class component

#### 使用场景

通过上面 ref 属性的介绍，我们其实顺便介绍了 useRef 其中一个常见的使用场景：获取 ref

但是 useRef 返回的对象还有三个十分特别的特性：

- 它的生命周期与 Function component 保持一致
- 它是 mutable 的
- 对象中 current 值的变化并不会导致组件重渲染

这三个特性其实在 Class component 中有一个非常贴切的名词：实例属性 Instance variables

所以，**我们可以把 useRef 返回的对象当成  Class component 中的实例属性来用**

举个例子：

```jsx
function Counter() {
  const [count, setCount] = useState(1)
  const intervalId = useRef();

  useEffect(() => {
    intervalId.current = setInterval(() => {
      console.log('invoke setCount')
      setCount(prev => prev + 1);
    }, 1000);
    return () => {
      console.log('clear automatically', intervalId.current)
      clearInterval(intervalId.current);
    }
  }, [])

  return (
    <div>
      <div>{count}</div>
      <button onClick={() => {
        console.log('clear manually')
        // 直接拿取对应的 id 取消计时器
        clearInterval(intervalId.current);
      }}>clear interval</button>
    </div>
  );
}
```

在上面的例子中，我们在第一次渲染组件时用 useEffect 注册了一个 intervalTimer，并且保存了它的 id 为我们将来清理它做准备，除了在组件销毁时主动清除这个计时器，我们希望在组件展示的过程中能够让用户随时通过点击按钮来暂停这个计时器，这个时候就非常适合用 useRef 了，使用这个 “实例变量” 我们有几个好处：

- intervalId 随着组件的卸载其占用的内存会被释放
- 组件的重渲染并不会更改 intervalId 的值
- intervalId 值的变更不会引发组件的重渲染

<br />

如果说上面的例子为我们展示了 useRef 在宽度（组件作用域）纬度上的应用，那么下面的例子为我们展示了 useRef 在深度（组件渲染时间轴）纬度上的运用

还记得我们上面说的其中一个特性么？

useRef 返回的对象生命周期与 Function component 保持一致

所以，useRef 返回的对象可以被看成一条贯穿组件一次次重渲染的一条线，自然的，它也就可以将上一次渲染的内部属性值通过闭包的方式带到下一次渲染

举个例子：

```jsx
const usePrevData = (data) => {
  const prevProps = useRef(0);
  useEffect(() => {
    prevProps.current = data;
  })
  return prevProps;
}

function Counter() {
  const [count, setCount] = useState(1)
  // 在这里拿到上一次的 count 值
  const prevCount = usePrevData(count);

  const intervalId = react.useRef();

  useEffect(() => {
    intervalId.current = setInterval(() => {
      console.log('invoke setCount')
      setCount(prev => prev + 1);
    }, 1000);
    return () => {
      console.log('clear automatically', intervalId.current)
      clearInterval(intervalId.current);
    }
  }, [])

  return (
    <div>
      <div>{count}</div>
      <div>{prevCount.current}</div>
      <button onClick={() => {
        console.log('clear manually')
        clearInterval(intervalId.current);
      }}>clear interval</button>
    </div>
  );
}
```

可以看到这个例子与上一个差不多，唯一的区别在于多了一个 usePrevData 的自定义 hook，这个 hook 的作用是，每次重渲染的时候传入一个当前的 count 值，我们来分析一下这个 hook 的执行步骤：

1. useRef 初始值为 0，一开始直接返回初始值，此时的作用域内 data 的值为 1
2. 组件渲染完之后，useEffect 中的副作用被调用，此时 data.current 被更新为作用域中的值 1，但因为 current 的改变不会引发重渲染，所以此时在 Counter 组件中的 30 行的值依然是 1
3. 计时器更新了 count 的值，触发了重渲染，此时 Counter 组件中 count 的值为 2 ， prevCount.current 的值为 1（同时 hook 中作用域保存着 data 为 2）
4. 重复步骤 2，3……

在这个过程中，我们成功获取到了上一次渲染的值～

### useImperativeHandle

> useImperativeHandle 提供了一种控制提供给父组件的 ref 中函数的方法，一般与 forwardRef 搭配使用

#### 基本用法

```js
useImperativeHandle(ref, createHandle, [deps])
```

useImperativeHandle 是一个方法，它接受三个参数：

- 第一个参数是一个 ref，它一般由父组件传递进来，负责承载 createHandle 返回的对象
- createHandle 是一个方法，它返回一个对象，对象中的属性就是要提供给 ref 的方法
- 最后一个参数是一个可选的数组，类似 useEffect 中的第二个参数，表示函数中的依赖变量

useImperativeHandle 的作用是“勾住”子组件中的一些方法供父组件使用

要了解上面那句话，我们需要从一个问题出发：React 中的父组件如何调用子组件中的方法呢？

答案是使用 ref 搭配 useImperativeHandle 以及 forwardRef

它的思路其实很简单：子组件将自身的方法添加到父组件使用 useRef 定义的对象中

虽然思路很简单，但是实现起来还是不容易的，来看看我们第一个例子：

```jsx
function Parent() {
  const counterRef = useRef();
  return (
    <>
      <button onClick={() => counterRef.current.click()}>Parent add one</button>
      <Counter ref={counterRef} />
    </>
  )
}

function Counter(props) {
  console.log(props);   // {}
  const [count, setCount] = useState(1);
  return (
    <div>
      <div>{count}</div>
      <button
        ref={props.ref}
        onClick={() => setCount(prev => prev + 1)}
      >
        Counter add one
      </button>
    </div>
  );
}
```

这个例子创建了两个组件，一个 Parent 组件中创建了一个 ref 然后想要通过 props 的方式传递给子组件，子组件在接收到 ref 之后将其绑定到 button 元素中，这样一来只要我们点击 Parent 组件中的按钮，Counter 组件中的 count 也会加一

但是事实真的如此吗？我们通过第 12 行打印 Counter 组件中的 props 的结果（一个空对象），可以知道 ref 属性并没有被传递下来，并且在控制台中，我们也能看到 React 给出的警告：

```
Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
```

这个是因为我们在 Counter 组件上使用 ref 的意思是：我想要把 counterRef 的 current 属性指向 Counter 组件的实例，但是 Function components 本身是没有实例的，所以它不能够像 Class components 一样使用 ref 属性，更别提传递 ref 了

事实上，Function component 想要传递 ref 有两种方法，下面我们分别来聊聊这两种方法

#### 传递 ref 的两种方法

##### forwardRef

在 React 16.3 以上的版本，提供了一个名叫 forwardRef 的方法来帮助 Function component 传递 ref

```jsx
function Parent() {
  const counterRef = useRef();

  return (
    <>
      <button onClick={() => counterRef.current.click()}>Parent add one</button>
      <Counter ref={counterRef} />
    </>
  )
}

// 加上 forwardRef
const Counter = forwardRef((props, ref) => {
  const [count, setCount] = useState(1);
  return (
    <div>
      <div>{count}</div>
      <button
        ref={ref}
        onClick={() => setCount(prev => prev + 1)}
      >
        Counter add one
      </button>
    </div>
  );
})
```

可以看到 forwardRef 的作用就是创建一个高阶组件，该组件能够将其接收的 ref 属性转发到内部的一个组件中

##### 自定义 props

上面我们说到，没办法将组件中的 ref 属性透传，那么我们可以用自定义的 props 来传递 ref：

```jsx
function Parent() {
  const counterRef = useRef();

  return (
    <>
      <button onClick={() => counterRef.current.click()}>Parent add one</button>
    	{/* 使用 props 来传递 ref */}
      <Counter customRef={counterRef} />
    </>
  )
}

const Counter = (props) => {
  const [count, setCount] = useState(1);

  return (
    <div>
      <div>{count}</div>
      <button
        ref={props.customRef}
        onClick={() => setCount(prev => prev + 1)}
      >
        Counter add one
      </button>
    </div>
  );
}
```

这里的 customRef 可以是除了 ref 之外的任何名称，这种方法的好处是更加灵活，且不会受到 react 版本的限制

备注：如果想要了解这两种方法各自的利弊，可以看看这篇 [文章](https://stackoverflow.com/questions/58578570/value-of-using-react-forwardref-vs-custom-ref-prop/60237948#60237948)

#### 限制父组件 ref 的功能

通过上述传递 ref 的方法，我们可以让父组件中的 ref 取得子组件元素的引用了，但是随之而来有一个问题：我们并不想要让父组件太过“自由”地操纵子组件

换言之，我们希望有一种方法，可以限制 ref 中存在我们希望被使用的方法，而非暴露整个子组件元素

useImperativeHandle 就是为了这个场景而生的：

```jsx
const Counter = (props) => {
  const [count, setCount] = useState(1);

  useImperativeHandle(
    props.customRef,
    () => ({
      click: () => setCount(prev => prev + 1)
    }),
  )

  return (
    <div>
      <div>{count}</div>
      <button
        onClick={() => setCount(prev => prev + 1)}
      >
        Counter add one
      </button>
    </div>
  );
}
```

通过使用 useImperativeHandle，我们不再是将 button 元素的 dom 节点直接暴露给 Parent 组件，而是只暴露 click 事件，这一层限制杜绝了开发者通过 ref 取到 dom 后，执行不该被使用的 API，出现 ref 失控的情况

#### 使用场景

虽然传递 ref 使得父组件能够使用子组件的功能非常强大，但是这个举动无疑破环了 react 的单向数据流原则，也使得代码更加不容易理解与调试，所以原则上来说，如果不是真的确定必要的情况下，不建议使用这个能力

### useLayoutEffect

> useLayoutEffect 为我们提供了一个渲染函数执行完成后，屏幕绘制前同步执行语句的方法

#### 基本用法

```jsx
/**
 * useLayoutEffect hook
 * @param effect 副作用函数，可以通过 return 返回一个清理函数
 * @param deps 依赖数组，如果传了的话 effect 只有在其中的依赖项变更才会被调用
 * @return void 没有返回值
**/
useLayoutEffect(effect: EffectCallback, deps?: DependencyList)
```

可以看到，这个方法的用法与 useEffect 基本一模一样

#### useLayoutEffect 与 useEffect 区别

- useLayoutEffect 的执行时机在渲染函数执行完成后，屏幕绘制前**同步执行**，与 componentDidMount 和 componentDidUpdate 保持一致
- useEffect 执行时机则是在屏幕绘制完成后**异步执行**

下面的例子展示了两个方法的执行顺序：

```jsx
function App() {
  useEffect(() => {
    console.log('useEffect fn')
    return () => {
      console.log('useEffect return')
    }
  }, []);

  useLayoutEffect(() => {
    console.log('** useLayoutEffect fn **')
    return () => {
      console.log('** useLayoutEffect return **')
    }
  }, []);

  return (
    <>
      <div></div>
    </>
  )
}

// log:
// ** useLayoutEffect fn **
// useEffect fn
// ** useLayoutEffect return **
// useEffect return
```

#### 使用场景

由于 useLayoutEffect 同步执行的特性，如果在它里面执行太多计算或者需要等待的方法，会导致整个页面无法响应任何操作甚至渲染不出东西

所以建议 **99% 的场景都优先使用 useEffect**

只有使用 useEffect 时因为异步更改变量触发重渲染导致画面闪烁时，才应该考虑使用 useLayoutEffect

举个例子：

```jsx
function Counter() {
  const [text, setText] = useState('******************')
  useEffect(() => {
    let i = 0;
    while (i <= 100000000) {
      i++;
    };
    setText('&&&&&&&&&&&&&');
  }, []);

  // useLayoutEffect(() => {
  //   let i = 0;
  //   while (i <= 100000000) {
  //     i++;
  //   };
  //   setText('&&&&&&&&&&&&&');
  // }, []);

  return (
    <>
      <h1>{text}</h1>
    </>
  )
}
```

使用 useEffect 的版本可以看到画面一开始会闪过星星符号，对比 useLayoutEffect 的版本则不会出现画面的闪烁

根本原因在于使用 useEffect 的方法执行时机是在屏幕绘制后，所以一开始屏幕显示的是 *，等到 * 被渲染出来之后，程序经历了一亿次循环，才把 & 显示在屏幕上；使用 useLayoutEffect 的方法执行时机是在屏幕绘制前同步执行，所以还没等 * 被绘制到屏幕上，会先执行一亿次循环，把 * 设置成 & 才进行屏幕的绘制，所以用户看到的只有最后的 &，并不会看到闪烁的 *

#### 注意事项

- 使用 useLayoutEffect 有可能阻塞屏幕更新
- useLayoutEffect 的调用时机等价于使用 componentDidMount 和 componentDidUpdate
- 当使用服务端渲染（SSR）时，需要注意的是在 js 代码被客户端下载并执行之前，useEffect 与 useLayoutEffect 都无法被执行，所以在 SSR 使用 useLayoutEffect 会有警告提醒，详情可见 [官网](https://reactjs.org/docs/hooks-reference.html#uselayouteffect)

### useDebugValue

> useDebugValue 为我们提供了一个给自定义 hook 添加标签以快速定位问题的机制

#### 基本用法

```jsx
function useFriendStatus() {
  const [isOnline, setIsOnline] = useState(false);

  // 使用 useDebugValue 在 react 的 devtool 中显示 isOnline 最新的值
  useDebugValue(isOnline);

  useEffect(() => {
    setIsOnline(true)
  }, []);

  return isOnline;
}
```

标签展示结果如下：

![8-2](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/8-2.jpg)

可以看到，在自定义 hook 的名称旁边，我们顺利标出了 state 的当前值，而且这个值是实时随着 state 的变化展示的

如果我们在一个自定义 hook 中使用了多个 useDebugValue，则标签会以数组的形式按顺序进行展示：

![8-3](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/8-3.jpg)

#### 自定义格式化方法

虽然直接使用 useDebugValue 足以应付大多数的问题了，但是有些时候我们会需要客制标签格式化的方法；或者有些数据的格式化可能会有额外的复杂度，我们只希望它在 hook 被 inspect 的时候被执行时，我们可以使用 useDebugValue 的第二个参数

useDebugValue 的第二个参数是一个格式化函数，它接收一个参数，这个参数的值时第一个参数的值

格式化函数只有在 react devtool 的 hook 被 inspect 时才会被执行：

```jsx
useDebugValue(date, date => date.toDateString());
```




