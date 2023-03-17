# useId

>  useId 为我们提供了基于组件层级，保证唯一的字符串 ID
## 基本用法

```js
const id = useId();
```
useId 不接受任何参数，并且会返回一个保证唯一的字符串 ID
## 使用场景
1. 当作 html 元素的 id 属性（保证全局唯一）
2. 在使用 SSR 的时候 useId 能够保证服务端与客户端生成的 ID 是一致的
## 注意事项
1. useId 是一个 hook，所以 hook 那些基本原则 useId 也必须遵守
2. useId 创建的 ID 不应该被当成一个 list 的 key 属性