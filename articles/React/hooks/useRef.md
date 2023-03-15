# useRef

> useRef 为我们提供了一个在组件生命周期内持久化保存不会触发渲染的对象的方法

## 基本用法

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

## ref 属性

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

### useRef & createRef

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

### callback refs

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

### 字符串 ref（Legacy api）

最后一种方法是最不推荐使用的，在文档与代码中已经被标注为 Legacy API 并且在代码运行时会有警告提醒

如果像深入了解为什么这个不被推荐使用，可以看看这篇[文章](https://github.com/facebook/react/pull/8333#issuecomment-271648615)

<br />

最后，需要提醒的一点是，虽然我在前面说 ref 可以获取 react 组件或者 dom 元素的实例，但是其实 ref 是无法获取到 Function component 的实例的，因为 Function component 本身并没有实例，如果想在 Function component 中使用 ref，可以用我们之后会介绍的 [useImperativeHandle](https://reactjs.org/docs/hooks-reference.html#useimperativehandle) hook 搭配 [forwardRef](https://reactjs.org/docs/forwarding-refs.html) 使用，或者将你的组件转换为 Class component

## 使用场景

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