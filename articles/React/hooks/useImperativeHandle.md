# useImperativeHandle

> useImperativeHandle 提供了一种控制提供给父组件的 ref 中函数的方法，一般与 forwardRef 搭配使用

## 基本用法

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

## 传递 ref 的两种方法

### forwardRef

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

### 自定义 props

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

## 限制父组件 ref 的功能

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

## 使用场景

虽然传递 ref 使得父组件能够使用子组件的功能非常强大，但是这个举动无疑破环了 react 的单向数据流原则，也使得代码更加不容易理解与调试，所以原则上来说，如果不是真的确定必要的情况下，不建议使用这个能力