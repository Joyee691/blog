# React hooks

## 背景知识

> 如果你只是一个 react 初学者并且想先知道 hooks 有哪些以及他们各自的用法，可以先跳过这一章节，这并不会影响你对 hooks 的使用；
>
> 但是如果想要进一步了解 hooks 背后的一些思考，那么我强烈建议你在了解完 hooks 用法之后回来看看

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