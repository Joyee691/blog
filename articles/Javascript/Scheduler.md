# 实现最大并发任务控制器 Scheduler

## 目标

实现一个任务调度器，用来控制异步任务（Promise）的并发数量。

同一时间最多只能有 N 个任务在执行，多出来的排队等前面的完成。

```js
class Scheduler {
  // maxConcurrency 代表最大并发数
  constructor(maxConcurrency) { /* ... */ }
  // taskFn 是一个函数，调用时会返回一个 promise
  // add 方法必须返回一个 promise
  // 当 taskFn() 返回的 Promise resolve 时：add 返回的 Promise 也要 resolve（值要一致地透传）
  // 当 taskFn() 返回的 Promise reject 时：add 返回的 Promise 也要 reject（错误要透传）
  add(taskFn) { /* ... */ }
}
```

## 测试 case

```js
function createTask(id, time) {
  return () => new Promise(resolve => {
    console.log('start', id)
    setTimeout(() => {
      console.log('end', id)
      resolve(id)
    }, time)
  })
}

const scheduler = new Scheduler(2)

scheduler.add(createTask(1, 1000))
scheduler.add(createTask(2, 500))
scheduler.add(createTask(3, 300))
scheduler.add(createTask(4, 400))

// 预期结果：
// start 1
// start 2
// end 2
// start 3
// end 3
// start 4
// end 1
// end 4
```

## 实现

```js
class Scheduler {
  constructor(maxConcurrency) {
    this.limit = maxConcurrency
    this.tasks = []
    this.wipCount = 0
  }

  add(taskFn) {
    return new Promise((resolve, reject) => {
      this.tasks.push({ taskFn, resolve, reject })
      this.checkAndExecute()
    })
  }

  checkAndExecute() {
    while (this.wipCount < this.limit && this.tasks.length > 0) {
      const { taskFn, resolve, reject } = this.tasks.shift()
      this.wipCount++
      // 保证 taskFn 中的 throw 能被 catch 到
      Promise.resolve(taskFn()).then(res => {
        resolve(res)
      }).catch(err => {
        reject(err)
      }).finally(() => {
        this.wipCount--
        this.checkAndExecute()
      })
    }
  }
}
```

