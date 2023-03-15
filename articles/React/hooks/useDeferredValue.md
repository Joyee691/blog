# useDeferredValue

>  useDeferredValue 与 useTransition 类似，只不过它“推迟”的是导致变化的 value，从而达到推迟某部分 ui 渲染的目的
## 基本用法

```jsx
const [value, setValue] = useState();
const deferredValue = useDeferredValue(value);
```
可以看到，useDeferredValue 接收一个值（一般是 state 或 prop 这种能导致 ui 变化的值），然后返回一个被“拖延”的值
这个值只能是 js 的原始值（比如 number，string）或者在组件作用域外创建的对象。
因为如果我们传给 useDeferredValue 的是组件作用域内创建的对象，随着每次组件的重渲染都会导致传入一个新的对象，从而导致多余的重渲染

## 使用场景

想要了解使用场景，我们可以先从它的实现作为切入点，以下是 useDeferredValue 的简单实现：
```js
// 挂载阶段 useDeferredValue 简易实现
function useDeferredValue(value) {
    const [prevValue, setPrevValue] = useState(value);
    const [isPending, startTransition] = useTransition();
    useEffect(()=> {
        startTransition(() => setPrevValue(value));
    }, [value])
    return prevValue;
}
```
可以看到，useDeferredValue 本质上就是在组件渲染后使用 useEffect 执行了一个低优先级的 transition，这样有两点好处：
1. 被 deferred 的值保证会在原来的 value 之后被更改
2. 被 deferred 的值的更新是可以被更高优的任务打断的
与 useTransition 不同的是，useDeferredValue 不需要获取到对应值的 setState 方法，这使得它能够覆盖一些 useTransition 无法使用的场景，比如 props，或者从其他 hook 返回的值。
## 注意事项

对于想要用 useDeferredValue 优化的组件，最好是将需要昂贵计算的部分抽离出单独的组件，并且将 deferred 的 value 通过 props 传递并且将组件使用 memo 包裹起来，这样才能有效的降低不必要的重渲染
举个例子：

```jsx
function App() {
  const [value, setValue] = useState('');
  const deferredValue = useDeferredValue(value);

  const handleChange = (e) => {
    setValue(e.target.value);
  };

  return (
    <div className='container'>
      <input onChange={handleChange} />
      <div style={{ display: 'flex' }}>
        <div>
          {Array(10)
            .fill('a')
            .map(() => (
              <div>{value}</div>
            ))}
        </div>
        <div style={{ width: '20px' }}></div>
        <SlowList value={deferredValue} />
      </div>
    </div >
  );
}

const SlowList = memo((props) => {
  const { value } = props;
  return (
    <div>
      {Array(50000)
        .fill('a')
        .map(() => (
          <div>{value}</div>
        ))}
    </div>
  )
})
```





