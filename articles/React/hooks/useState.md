# useState

> useState 就是为 function component 提供了 state 的能力的 hook

## 基本用法

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

## 多个 state

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

## 修改 state

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

## 惰性初始化 Lazy Initialization

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

## 总结

- useState 为我们提供了一种在 function component 中使用 state 的方法
- 如果一个组件中需要多个 state，可以调用多个 useState
- 在修改涉及上一个状态的 state 时，优先使用传入函数的方式更新 state
- 如果 state 的初始化涉及到较大计算开销，可以将其包裹在一个函数中传给 initialization