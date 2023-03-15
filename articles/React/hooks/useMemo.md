# useMemo

> useMemo 提供了一种通过 **保存方法计算结果** 来减少不必要的昂贵计算的性能优化方案

## 基本用法

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

## 使用场景

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