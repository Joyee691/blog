# useDebugValue

> useDebugValue 为我们提供了一个给自定义 hook 添加标签以快速定位问题的机制

## 基本用法

```jsx
function useFriendStatus() {
  const [isOnline, setIsOnline] = useState(false);

  // 使用 useDebugValue 在 react 的 devtool 中显示 isOnline 最新的值
  useDebugValue(isOnline);

  useEffect(() => {
    setIsOnline(true)
  }, []);

  return isOnline;
}
```

标签展示结果如下：

![8-2](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/8-2.jpg)

可以看到，在自定义 hook 的名称旁边，我们顺利标出了 state 的当前值，而且这个值是实时随着 state 的变化展示的

如果我们在一个自定义 hook 中使用了多个 useDebugValue，则标签会以数组的形式按顺序进行展示：

![8-3](https://raw.githubusercontent.com/Joyee691/image-hosting/main/blog/8-3.jpg)

## 自定义格式化方法

虽然直接使用 useDebugValue 足以应付大多数的问题了，但是有些时候我们会需要客制标签格式化的方法；或者有些数据的格式化可能会有额外的复杂度，我们只希望它在 hook 被 inspect 的时候被执行时，我们可以使用 useDebugValue 的第二个参数

useDebugValue 的第二个参数是一个格式化函数，它接收一个参数，这个参数的值时第一个参数的值

格式化函数只有在 react devtool 的 hook 被 inspect 时才会被执行：

```jsx
useDebugValue(date, date => date.toDateString());
```

