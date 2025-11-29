# useLayoutEffect

> useLayoutEffect 为我们提供了一个渲染函数执行完成后，屏幕绘制前同步执行语句的方法

## 基本用法

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

## useLayoutEffect 与 useEffect 区别

- useLayoutEffect 的执行时机在渲染函数执行完成后，屏幕绘制 paint 前**同步执行**，与 componentDidMount 和 componentDidUpdate 保持一致
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

## 使用场景

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

## 注意事项

- 使用 useLayoutEffect 有可能阻塞屏幕更新
- useLayoutEffect 的调用时机等价于使用 componentDidMount 和 componentDidUpdate
- 当使用服务端渲染（SSR）时，需要注意的是在 js 代码被客户端下载并执行之前，useEffect 与 useLayoutEffect 都无法被执行，所以在 SSR 使用 useLayoutEffect 会有警告提醒，详情可见 [官网](https://reactjs.org/docs/hooks-reference.html#uselayouteffect)