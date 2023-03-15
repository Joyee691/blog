# useEffect

> useEffect 为我们提供了一种在 function component 中消费副作用的方法

## 基本用法

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

## 常见用法与清理副作用

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

## 模拟生命周期函数

如果你熟悉 class component 的生命周期函数，不难发现 useEffect 其实就是 `componentDidMount`、`componentDidUpdate` 和`componentWillUnmount` 的组合，下面我们就分别来说说 useEffect 要怎么分别实现三种生命周期的功能

### componentDidMount

这个生命周期函数只会在组件初次渲染的之后被调用，所以我们只需要把 useEffect 的 deps 数组置空就好

```jsx
useEffect(() => {
    console.log('[componentDidMount]: log only after mount')
}, []);
```

### componentDidUpdate

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

### componentWillUnmount

这个生命周期函数只在组件要被卸载之前被调用，同样地，useEffect 可以通过在 effect 函数中返回一个 cleanup 函数来实现

区别在于 cleanup 函数会在每次 effect 调用前被调用，具体原因可以看一下 [常见用法与清理副作用](##常见用法与清理副作用)

## 处理多个副作用

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

