# 使用 JS 实现并发数量控制

之前面试遇到一个问题，要求用 JS 来实现异步请求的并发控制，觉得蛮有意思的就记录下来了

## 要求

1. 可以随意设置并发请求的最大数量
2. 整个过程是自动进行的，用户只需要在想要的时候把请求加进去就好

## 测试方法

测试方法用了 setTimeout 来模拟异步请求：

```js
// info：请求返回的结果，用来标识当前的请求
// delay：请求的延时，用来模拟网络请求的情况
function request(info, delay) {
  return () => new Promise((res, rej) => {
    setTimeout(() => {
      res(info);
    }, delay);
  })
}
```

## 实现

```js
class ConcurrentControl {
  // 限制最大并发请求数量
  limit;
  // 用一个队列保存等待被执行的请求
  queue;
  // 用来判断 run 函数是否正在执行
  isRunning = false;
  // 记录当前正在请求的数量
  currentRequest = 0;

  constructor(limit = 3, queue = []) {
    this.limit = limit;
    this.queue = queue;
    if (this.queue.length > 0) {
      this.run();
    }
  }

  // 设置最大并发请求数
  setLimit(limit) {
    this.limit = limit;
    if (!this.isRunning) {
      this.run();
    }
  }

  // 追加请求
  addRequest(request) {
    this.queue.push(request);
    if (!this.isRunning) {
      this.run();
    }
  }

  // 核心并发控制函数
  run() {
    this.isRunning = true;
    // 只有当 queue 中还有等待的请求，且并发数量还没满的时候才会发送新请求
    while (this.queue.length > 0 && this.currentRequest < this.limit) {
      this.currentRequest++
      const r = this.queue.shift();
      r().then(res => {
        // 在新请求结束的时候，需要更新当前的请求数，并且重新跑一次当前函数
        console.log('request return', res);
        this.currentRequest--;
        this.run();
      })
    }
    this.isRunning = false;
  }
}
```

## 测试 case

```js
controller.addRequest(request(1, 200))
controller.addRequest(request(2, 300))
controller.addRequest(request(3, 500))
controller.addRequest(request(4, 50))
controller.addRequest(request(5, 150))
controller.addRequest(request(6, 250))
controller.addRequest(request(7, 30))
controller.addRequest(request(8, 10))

// 预期结果： 14257836
```

说明（为了方便，我们使用数组来描述当前请求的剩余时间，数组的 8 个元素分别按顺序对应 1-8 号的请求，`-` 表示请求还没开始）：

```js
0ms: [200, 300, 500, -, -, -, -, -] => 没有输出
200ms: [0, 100, 300, 50, -, -, -, -] => 输出 1，插入 4
250ms: [0, 50, 250, 0, 150, -, -, -] => 输出 4，插入 5
300ms: [0, 0, 200, 0, 100, 250, -, -] => 输出 2，插入 6
400ms: [0, 0, 100, 0, 0, 150, 30, -] => 输出 5，插入 7
430ms: [0, 0, 70, 0, 0, 120, 0, 10] => 输出 7，插入 8
430ms: [0, 0, 70, 0, 0, 120, 0, 0] => 输出 8
500ms: [0, 0, 0, 0, 0, 120, 0, 0] => 输出 3
620ms: [0, 0, 0, 0, 0, 120, 0, 0] => 输出 6
```

## 写在结尾

本来想让 chatgpt 帮我多写几个测试 case 的，奈何它听不懂我的话😑

如果有人知道怎么教 gpt 写测试 case，或者觉得我代码逻辑有哪里奇怪的欢迎留言（issue）

可以请喝☕️







