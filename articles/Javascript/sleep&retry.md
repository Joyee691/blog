# 实现 sleep 与 retry

## 实现一个 sleep

### 要求

编写一个函数：

```js
function sleep(ms) {
    // ...
}
```

它返回一个 Promise，会在 ms 毫秒之后 resolve。

### 示例

```js
const test = async () => {
  console.log("start")
  await sleep(1000)
  console.log("end after 1 second")  // 一秒后输出：1 秒后打印
}
test()
```

### 实现

```js
function sleep(ms) {
  return new Promise((res => {
    setTimeout(() => {
      res()
    }, ms)
  }))
}
```

##  实现一个 retry 方法

### 要求

编写一个异步函数：

```js
async function retry(asyncFn, times, delay) {
    // ...
}
```

retry 函数：

1. 执行一次 `asyncFn()`（asyncFn 返回 Promise）
2. 如果 fn 成功（resolve），则 retry 也立即成功
3. 如果 fn 失败（reject），则：
   - 等待 delay ms
   - 重新执行 fn
4. 最多重试 times 次
5. 如果全部尝试都失败，必须抛出（reject）最后一次的错误

### 示例

```js
let count = 0
retry(() => {
    return new Promise((resolve, reject) => {
        if (++count >= 3) resolve("OK")
        else reject("Fail")
    })
}, 5, 1000).then(console.log)
// 前两次失败，最后输出 OK
```

### 实现

```js
async function retry(asyncFn, times, delay) {
  for (let count = 1; count <= times; count++) {
    try {
      return await asyncFn()
    } catch (err) {
      if (count === times) {
        throw err
      }
      await sleep(delay)
    }
  }
}
```

