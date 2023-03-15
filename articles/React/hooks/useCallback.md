# useCallback

> useCallback 提供了一种通过 **保存方法引用** 来减少不必要的渲染的性能优化方案

## 基本用法

```jsx
const memorizedFn = useCallback(fn, []);
```

可以看到，useCallback 主要接收两个参数，并返回一个新的方法

- 第一个参数是一个方法，它就是需要被保存引用的方法
- 第二个参数则是一个数组，数组中存放的是第一个参数中方法的依赖项，只有当数组中的依赖项变更时，该方法的引用才会被重新更新
- 返回值则是一个被保存了引用的方法

## 使用场景

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

## 总结

- useCallback 可以保存方法引用，在需要引用相等的场景下，如 memo，pureComponent，以及 shouldComponentUpdate 时可以避免重复渲染进而优化性能
- 在使用 useCallback 优化性能之前，请谨慎评估并实际测量，因为贸然使用而产生的成本很可能最终大于优化后的收益