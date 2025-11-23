# 实现自己的 Promise.all

## 要求

请实现一个函数 **myPromiseAll(promises)**，功能和原生 Promise.all 相同

```js
function myPromiseAll(promises) {
    // your code here
}
```

**输入**：

- promises 是一个数组
- 内部的元素可以是：
  - Promise 对象
  - 普通值（自动视为已 resolve 的 Promise）

**输出**：返回一个新的 Promise

- 如果所有 Promise 都成功，resolve 一个数组（按顺序）
- 只要有一个失败，立即 reject

## 实现

```js
function myPromiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError('arguments must be array'))
    }
    const len = promises.length
    const result = new Array(len)
    let finishCount = 0
    promises.forEach((promise, idx) => {
      Promise.resolve(promise).then(res => {
        result[idx] = res
        if (++finishCount === len) {
          return resolve(result)
        }
      }, reject)
    })
  })
}
```

## 测试数据

```js
let p1 = Promise.resolve(1),
  p2 = Promise.resolve(2),
  p3 = Promise.resolve(3),
  p4 = 4
myPromiseAll([p1, p2, p3, p4]).then(function (results) {
  console.log(results);  // [1, 2, 3, 4]
}).catch(err => console.log('err', err))
```

```js
let p1 = Promise.resolve(1),
  p2 = Promise.reject(2),
  p3 = Promise.resolve(3),
  p4 = 4
myPromiseAll([p1, p2, p3, p4]).then(function (results) {
  console.log(results);  
}).catch(err => console.log('err', err))	// err 2
```







