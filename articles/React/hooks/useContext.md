# useContext

> useContext 为我们提供了一种使用 Context.Consumer 的语法糖，它可以让我们跨越组件层级传递信息

## 基本用法

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

context 最大的用途在于摆脱 props 的层层传递关系，为我们提供了跨层组件之间的通信，我们将在后面的 [Context](##Context) 章节中详细介绍 context，让我们先了解它要怎么用吧：

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

## Context

> context，顾名思义，就是上下文的意思，也就是说它提供了一种组件版本的 “全局变量”

想要了解我们为什么需要 context，以及什么时候用 context，我们必须先从目前的痛点入手

让我们回顾一下上面 “基本用法” 中的例子，如果我们不用 context，根据 react 组件由上自下的单向数据流，我们需要将字符串通过 props 层层传递，最后才能到达消费这个属性的子组件中

在这个过程中，Parent 组件完全不关心这个属性，但是要负责 “传话”，这个明显不符合组件之间 ”高内聚，低耦合“ 的原则

比较理想的情况应该是：**上层组件 Ancestor 把这个属性 “抛” 到一个空间中，再让想要用这个属性的子组件从空间中 “抓取” 下来**

上面这句话中，空间指的就是 context；抛这个动作可以是 createContext 的初始值，也可以是 Provider；抓取这个动作对应的就是 Consumer

接下来，我们开始介绍所有 context 的起点：react.createContext 方法

### createContext

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

### Provider

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

### Consumer

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

## Context 的使用场景

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

## Context 多余渲染问题

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

### 拆分 context（推荐）

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

### 拆分组件

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

### 使用 useMemo

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