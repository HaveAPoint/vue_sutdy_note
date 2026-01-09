# Promise 状态与原理

## 1. 什么是 Promise？

**Promise 是异步编程的一种解决方案。** 它是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。

### 为什么需要 Promise？

**解决"回调地狱" (Callback Hell)**。

```javascript
// 回调地狱
doA(function(resA) {
  doB(resA, function(resB) {
    doC(resB, function(resC) {
      console.log(resC)
    })
  })
})

// Promise 链式调用
doA()
  .then(resA => doB(resA))
  .then(resB => doC(resB))
  .then(resC => console.log(resC))
```

## 2. 三种状态

Promise 对象代表一个异步操作，有三种状态：

| 状态 | 英文 | 含义 |
|------|------|------|
| **进行中** | `pending` | 初始状态，既没有成功，也没有失败 |
| **已成功** | `fulfilled` (或 `resolved`) | 操作成功完成，通常会带有一个 value |
| **已失败** | `rejected` | 操作失败，通常会带有一个 error |

**特点：状态改变不可逆。**
- 只能从 `pending` -> `fulfilled`
- 或者从 `pending` -> `rejected`
- 一旦改变，状态就固定了（Resolved）。

## 3. 基本用法

```javascript
const promise = new Promise((resolve, reject) => {
  // 执行异步操作
  setTimeout(() => {
    const success = true
    if (success) {
      resolve('成功的数据') // pending -> fulfilled
    } else {
      reject('失败的原因')  // pending -> rejected
    }
  }, 1000)
})

promise
  .then(value => {
    console.log('Success:', value)
  })
  .catch(error => {
    console.log('Error:', error)
  })
  .finally(() => {
    console.log('结束（无论成功失败）')
  })
```

## 4. 链式调用原理

`then` 方法返回的是一个**新的 Promise 实例**。

```javascript
p1.then(val => {
  // 返回值会决定下一个 then 的状态
  return val + 1
}).then(val => {
  console.log(val)
})
```

### 返回值处理规则

1. **返回普通值**：新的 Promise 变为 fulfilled，值为返回的值。
2. **返回 Promise**：新的 Promise 跟随这个 Promise 的状态。
3. **抛出错误**：新的 Promise 变为 rejected。

## 5. 常用静态方法

### `Promise.all([p1, p2, p3])`
- **全部成功**才算成功（返回所有结果数组）。
- **只要有一个失败**就立即失败（返回第一个失败原因）。
- 适用：并发请求，需要所有数据都准备好。

### `Promise.race([p1, p2, p3])`
- **谁跑得快**就用谁的结果（无论成功失败）。
- 适用：超时控制（请求 vs 5秒定时器）。

### `Promise.allSettled([p1, p2])` (ES11)
- 等所有都结束（无论成功失败）。
- 返回每个 Promise 的状态对象数组 `{ status: 'fulfilled', value: ... }`。

### `Promise.any([p1, p2])` (ES12)
- 只要有**一个成功**就算成功。
- 全部失败才算失败。

## 6. async / await (终极解决方案)

Generator 函数的语法糖，让异步代码像同步代码一样。

```javascript
async function getData() {
  try {
    const user = await getUser()      // 等待 Promise resolve
    const posts = await getPosts(user.id)
    console.log(posts)
  } catch (error) {
    console.log('出错啦', error)      // 捕获 reject
  }
}
```

## 7. 手写简易 Promise (面试核心)

```javascript
class MyPromise {
  constructor(executor) {
    this.state = 'pending'
    this.value = undefined
    this.reason = undefined
    this.onFulfilledCallbacks = []
    this.onRejectedCallbacks = []

    const resolve = (value) => {
      if (this.state === 'pending') {
        this.state = 'fulfilled'
        this.value = value
        this.onFulfilledCallbacks.forEach(fn => fn())
      }
    }

    const reject = (reason) => {
      if (this.state === 'pending') {
        this.state = 'rejected'
        this.reason = reason
        this.onRejectedCallbacks.forEach(fn => fn())
      }
    }

    try {
      executor(resolve, reject)
    } catch (err) {
      reject(err)
    }
  }

  then(onFulfilled, onRejected) {
    // 穿透处理
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v
    onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err }

    return new MyPromise((resolve, reject) => {
      if (this.state === 'fulfilled') {
        setTimeout(() => {
          try {
            const x = onFulfilled(this.value)
            resolve(x) // 简化版，未处理 x 是 Promise 的情况
          } catch (e) { reject(e) }
        }, 0)
      }

      if (this.state === 'rejected') {
        setTimeout(() => {
          try {
            const x = onRejected(this.reason)
            resolve(x)
          } catch (e) { reject(e) }
        }, 0)
      }

      if (this.state === 'pending') {
        this.onFulfilledCallbacks.push(() => {
          setTimeout(() => {
            try {
              const x = onFulfilled(this.value)
              resolve(x)
            } catch (e) { reject(e) }
          }, 0)
        })
        this.onRejectedCallbacks.push(() => {
          setTimeout(() => {
            try {
              const x = onRejected(this.reason)
              resolve(x)
            } catch (e) { reject(e) }
          }, 0)
        })
      }
    })
  }
}
```

## 深入阅读

- [Promise/A+ 规范](https://promisesaplus.com/)
- [MDN - Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
