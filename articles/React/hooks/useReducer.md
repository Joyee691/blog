# useReducer

> useReducer 在复杂且有关联的组件状态变更场景中为我们提供了比 useState 更好的替代方案

## 基本用法

```jsx
const initialState = {
  count: 1
}

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error('Unknown action');
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState)
  return (
    <>
      <div>Count: {state.count}</div>
      <button onClick={() => dispatch({ type: 'incremen' })}>+1</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-1</button>
    </>
  )
}
```

可以看到，useReducer 的用法跟 useState 非常的相似，它接收两个参数，并且返回一个数组

- 第一个参数 reducer 是一个方法，利用 action 提供的信息处理 state 的更新
- 第二个参数 initialState 制定了初始化时 state 的值
- 返回的数组中，第一个参数是一个 state，第二个参数则是用来更新 state 的 dispatch 方法
- dispatch 方法有一个 action 对象作为参数，action 通常由 type 与 payload 构成，前者定义操作类型，后者携带操作参数

## 常见用法以及与 useState 的区别

看了上面的例子，我们不难发现 useReducer 与 useState 的功能相差无几，甚至把上面的例子换成 useState 不仅代码更短，还更好理解，但是存在即合理，我们先从它与 useState 的不同出发，慢慢挖掘 useReducer 的潜力

首先最直观的一点就是：有别于 useState 直接将 state 变更方式写在组件中的形式，useReducer 在组件中只关注要**对 state 做什么**

换言之，**useReducer 解耦了 what（做什么）和 how（怎么做）**

上面的例子可能没办法体现出 useReducer 的优势，我们再举一个例子：

假设现在我们需要一个表格，表格的数据是从后端接口获取的，且支持分页以及关键字搜索的操作，如果用 useState 来写我们需要先定义 state：

```jsx
const [pagination, setPagination] = useState({ page: 1, pageSize: 10 });
const [filter, setFilter] = useState({ keyword: "" });
const [isLoading, setIsLoading] = useState(false);
```

再来我们需要定义不同的操作：

```jsx	
// 设置当前的页数
const setPage = page => setPagination(prev => { return { ...prev, page } });
// 设置要搜索的关键词
const setKeyword = keyword => setFilter(prev => { return { ...prev, keyword } });
// 根据关键词搜索
const search = () => {
  setIsLoading(true);
  // 异步获取数据成功后更新
  fetchTableData(filter).then(() => {
    setPage(1);
    setKeyword('');
    setIsLoading(false);
  })
}
```

可以看到，我们在处理 state 的时候需要关心的细节太多了，尤其是成功获取数据之后，一口气需要更新三个 state，这个无疑加大了后期维护这段代码的心智负担，让我们再来看看 useReducer 的版本吧：

```jsx
const initialState = {
  pagination: { page: 1, pageSize: 10 },
  filter: { keyword: "" },
  isLoading: false,
}

function reducer(state, action) {
  const payload = action.payload;
  switch (action.type) {
    case 'CHANGE_PAGE':
      return {
        ...state,
        pagination: {
          ...state.pagination,
          page: payload.page
        }
      };
    case 'SET_KEYWORD':
      return {
        ...state,
        filter: {
          ...state.filter,
          keyword: payload.keyword
        }
      };
    case 'CLEAR_KEYWORD':
      return {
        ...state,
        filter: { keyword: '' },
        pagination: { ...state.pagination, page: 1 }
      };
    case 'SEARCH':
      return {
        ...state,
        isLoading: true,
      };
    case 'SUCCESS':
      return {
        ...state,
        isLoading: false,
        filter: {
          keyword: ''
        },
        pagination: {
          ...state.pagination,
          page: 1
        }
      };
    default:
      throw new Error('Unknown action');
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, initialState)

  // 一些 UI
	return <>...</>
}
```

可以看到，虽然代码变长了，但是所有的操作 state 的方法都被一个 action 给包装起来了，这样一来，我们只需要在需要的时候通过 dispatch 的方式告诉 reducer 我需要做什么并且把对应的值给到就行，比如：

```jsx
// 设置关键词  
dispatch({ type: 'SET_KEYWORD', payload: { keyword: "Shakespeare" } });

// 搜索
const search = () => {
  dispatch({ type: 'SEARCH' });
  fetchTableData(state.filter).then(() => {
    dispatch({ type: 'SUCCESS' });
  })
}
```

通过把“指令”与“操作”解耦的方式，我们将复杂的 state 操作抽象成了一个个指令，后面如果有遇到相关的操作，我们就只需要发出指令而不需要关注额外的细节了

所以，**对于复杂的 state 操作，或者是嵌套的 state 对象，推荐使用 useReducer**

<br />

除了上述的场景之外，还有一个场景是我们可以优先考虑使用 useReducer 的，那就是单元测试

首先，我们知道 react 的页面重渲染是由 props 和 state 驱动的，其中 useReducer 通过 action 很好的封装了 state 的变化

其次，在 react 中，state 的更新都必须要是 immutable 的，通过这个原则，我们可以得知 reduce 这个函数本质上就是一个纯函数，也就是说，对于任何相同的输入，我们都会得到相同的输出

基于以上两点，我们可以根据 reduce 函数的输出来判断 state 的变化，进而达到测试的目的

### 结论

- 对于复杂的 state 操作，或者是嵌套的 state 对象，推荐使用 useReducer
- 如果对组件有单测的需求，推荐使用 useReducer

## 惰性初始化 Lazy initialization

跟 useState 一样，useReducer 也支持惰性初始化，我们可以通过给 useReducer 传入第三个参数 init 方法来实现：

```jsx
function init(initialCount) {
  return {
    count: initialCount,
  }
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init)
  return <>...</>
}
```

可以观察到，init 方法会自动将第二个参数 initialCount 传入，并返回一个初始化好的 state

使用惰性初始化的场景主要有两个：

1. 如果初始化这个操作本身是一个昂贵的操作，而你不希望它在卖次 rerender 的时候都被执行一次的时候，可以用惰性初始化，它只会在第一次渲染的时候被执行
2. 我们可以利用 init 函数来还原 state 的初始值，比如

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'reset':
      return init(action.payload);
    default:
      throw new Error('Unknown action');
  }
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init)
  return (
    <button
      onClick={() => dispatch({type: 'reset', payload: initialCount})}>
      Reset
    </button>
  )
}
```

## 自己写一个 redux

redux 是诞生于 react 社区，为了解决 react 状态管理问题而设计开发的一个库

对于 redux 不熟悉的小伙伴，我这里一句话总结 redux 的功能：

**redux 提供了一个应用程序级别的全局状态管理，用户可以通过 UI dispatch 一个 action 给 reducer，reducer 会对 store 中的数据进行更新操作，而更新了的数据会触发 UI 的重渲染，从而实现页面数据的更新**，整个流程如下图所示。

![redux](https://www.freecodecamp.org/news/content/images/2022/06/2.png)

相信认真看完我上面对 useReducer 介绍的小伙伴这个时候已经想到了，这个流程不就跟 useReducer 的流程一模一样吗？

对此我只能说，很像，但不完全一样，区别在于 **useReducer 没有提供一个全局状态管理的功能**

既然没有我们就自己写一个呗，至于怎么写？答案是 Context

我们来一句话回顾一下 Context 的功能：它提供了一种组件版本的 “全局变量”，允许我们跨越组件层级传递参数，并且能通过参数的变更触发重渲染逻辑

那有了 Context 跟 useReducer，我们可以来着手写一个自己的 redux 啦～

秉持这最简化原则，我们还是那一开始的 Counter 组件来举例子，毕竟谁能拒绝一个简单的 Counter 组件呢 ;)

```jsx
const Context = react.createContext();

function Counter() {
  // 我直接省略了 reducer initialState 的实现，这两者的实现在一开始的基本用法已经给出
  const [state, dispatch] = useReducer(reducer, initialState)
  return (
    <>
      <Context.Provider value={{ state, dispatch }}>
        <div>Count: {state.count}</div>
        <Operator />
      </Context.Provider>
    </>
  )
}

function Operator() {
  const dispatch = useContext(Context);
  return (
    <>
      <button onClick={() => dispatch({ type: 'increment' })}>+1</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-1</button>
    </>
  )
}
```

可以看到，我们通过将 state 跟 dispatch 方法放入 Context（类比 redux 中的 store）模仿了一个 redux 出来：

我们提供了一个跨组件的全局状态管理（Context），用户可以通过 UI dispatch 一个 action 给到 reducer，reducer 会对 store（这里是 Context）中的数据进行更新操作，而更新了的数据会触发 UI 的重渲染（通过 Context 的机制），从而实现页面数据的更新

完美对上了前面对 redux 的描述。

## redux 能被取代吗

在上一个小节，我们说了如何用 useReduer 跟 Context 实现一个简易版的 redux

紧接而来的是好几个问题：我们实现的 redux 能够完美替代 redux 吗？这两者之间到底有什么区别呢？

首先第一个区别还是要从 Context 的缺点开始讲起——**Context 有多余渲染的问题**

当然，我们也在 useContext 的介绍中提到了三种避免多余渲染问题的解决方案，但是不论是拆分 context、拆分组件，抑或是使用 useMemo，都无法避免的对原来的代码进行侵入性修改

反观 redux，它将组件分为了容器型组件以及展示型组件，对于容器型组件需要使用 connect 函数生成，并通过 mapStateToProps 来过滤全局状态，在 connect 函数内部，对生成的容器型组件提供了一个在 shouldComponentUpdate 中对 state 跟 props 的浅比较，杜绝了多余渲染的发生

第二个区别，则是在于**我们自己实现的 redux 不能支持副作用（side effect）**

首先必须声明一点， redux 本身也是不支持副作用的，但是它提供了一个中间件机制，让社区开发者来帮它拓展功能，现在也有很多优秀的副作用处理解决方案了，比如redux-thunk、redux-promise、redux-saga等。

那么，我们自己写的 redux 就完全没有意义了吗？我们应该直接用 redux 吗？

俗话说的好，梭哈是一种艺术，但是这个时候显然不是展现这种艺术的时候

毕竟连人家 redux 的共同开发者 Dan Abramov 都说了：

> People often choose Redux before they need it.

在这里我给出我个人的观点：**建议在需要管理大型应用的全局状态时，再使用 redux；如果只需要处理小范围内的组件状态，可以考虑优先使用 Context + useReducer 的组合**