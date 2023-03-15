# useTransition

> useTransition 为我们提供了使用 react 18 Concurrent 模式的方法
>
> 背景知识：react 18 的 Cocurrent 模式、优先级调度

## 基本用法

```jsx
const [isPending, startTransition] = useTransition();
```

可以看到 useTransition 不接受任何参数，但是返回有两个值的数组：

- isPending 是一个 boolean，表示 transition 更新的状态，为 true 表示还未更新；反之则表示已经更新
- startTransition 是一个方法，接收一个可以触发更新的函数，比如：

```jsx
startTransition(() => setState(xxx));
```

当我们通过 startTransition 触发更新时，react 会采用 Concurrent 模式来协调 fiber tree

## startTransition

要了解 startTransition 有什么用，首先需要了解一下什么是 transition

transition 的中文翻译是过渡，也就是从一个状态转换成另一个状态的过程

我们知道 Concurrent 模式与 Lagecy 模式最大的区别在于它支持了根据优先级中断任务的能力

transition 既然是一种过渡，那就表示它的优先级是比较低的

实际上，通过 startTransition 触发的更新优先级为 **NormalPriority**，而在它之上，还有 **ImmediatePriority** 与  **UserBlockingPriority** 两种更高优先级别的更新

通常情况下，高优先级的更新会被优先处理，所以通过 startTransition 触发的更新在有更高优先级的情况下并不会在第一时间被处理，而是会处于 pending 状态，直到没有比它更高优的任务时，才会被处理

所以 startTransition 的作用在于：**降低某些较为不重要的更新的优先级，使得其他更高优的任务能够被更快的执行**

startTransition 除了通过 useTransition 在 Function Component 中使用之外，还能直接在 Class Component 中使用（只是在 Class Component 中无法使用 isPending 的值）

## 使用场景

上面说到，使用 startTransition 本质上是为了手动降低某些不重要的更新的优先级，那么我们首先需要知道什么更新是不那么重要的更新

一个比较常见的使用场景就是避免高频的页面重绘

举个例子：我们有一个搜索框以及一个长列表，长列表显示的数据会根据搜索框的输入不断地重新渲染

```jsx
function App() {
  const [value1, setValue1] = useState("");
  const [value2, setValue2] = useState("");
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    setValue1(e.target.value);
    startTransition(() => setValue2(e.target.value));
  };

  return (
    <div className="container">
      <input onChange={handleChange} />
      <div style={{ display: 'flex' }}>
        <div>
          {Array(10)
            .fill("a")
            .map(() => (
              <div>{value1}</div>
            ))}
        </div>
        <div style={{ width: '20px' }}></div>
        <div>
          {Array(50000)
            .fill("a")
            .map(() => (
              <div>{value2}</div>
            ))}
        </div>

      </div>
    </div>
  );
}
```

上面的例子可以分成两个部分来看：第一个部分是 value1，它使用了原来的 setState 方法直接触发更新；第二个部分是 value2，它用 startTransition 来触发低优先级的更新

当我们在输入框中随意输入一串字符时，可以看到首先被绘制出来的是输入框以及左边的列表，这两个部分对应的是直接使用 setState 更新的部分；在等待一会之后，右边的列表才会被绘制在屏幕上，这是因为使用 startTransition 触发的低优先级更新会在高优先级更新处理完之后才被执行

这么做的目的在于，如果今天要渲染的是一个长列表，如此高频的重渲染不仅会消耗大量资源，同时用户关注的是最后输入完之后的查询结果，并不是过程中高频的查询，这种情况下，我们就可以通过 startTransition 来告诉 react：这个任务并不高优，你可以先去处理别的高优任务。从而最大化用户体验

## 防抖与节流

既然解决的是高频卡顿的问题，那么可不可以用传统的防抖与节流来解决呢？
首先来看看防抖（debounce），防抖的原理是延迟执行——只有某次操作后在指定时间内没有再操作才会执行更新
防抖能够稍微解决交互卡顿的问题，但是用户在每次输入之后仍然会感受到一点点的延迟。此外，它还有两点不足：

1. 会出现用户长时间输入而得不到响应的情况
2. 等到正式更新开始后，渲染引擎仍然会被长时间阻塞，造成页面卡死的情况
   接下来看看节流（throttle），节流的原理是规定一段时间，只有到达规定时间才会开始处理对应更新
   节流能够解决防抖中用户长时间输入得不到响应的情况，但是仍然有两个缺点：
3. 与防抖相似，正式更新开始后渲染引擎仍然会被长时间阻塞导致页面卡死
4. 节流的时间一般依赖于开发人员的设置，这个配置的时间对于性能跨度较大的机型可能不符合预期
   所以，针对支持了 concurrent 模式的 react 来说，最有效预防此类问题的解决方案依旧是使用 useTransition，但是针对使用了不支持 concurrent 模式的 react，可以使用节流或防抖来提升用户体验