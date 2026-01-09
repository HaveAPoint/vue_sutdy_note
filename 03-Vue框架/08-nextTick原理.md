# nextTick 原理

## 为什么需要 nextTick？

Vue 的 DOM 更新是**异步**的。当你修改响应式数据后，DOM 不会立即更新，而是在下一个"tick"中批量更新。

```vue
<script setup>
import { ref, nextTick } from 'vue'

const count = ref(0)
const buttonRef = ref(null)

function increment() {
  count.value = 1
  console.log(buttonRef.value.textContent) // 还是 0！DOM 还没更新

  nextTick(() => {
    console.log(buttonRef.value.textContent) // 1，DOM 已更新
  })
}
</script>

<template>
  <button ref="buttonRef" @click="increment">{{ count }}</button>
</template>
```

## 异步更新队列

### Vue 的更新机制

```
数据变化 → 触发 setter → 通知 Watcher → 推入异步队列 → 下一个 tick 批量更新 DOM
```

**为什么要异步更新？**

```javascript
// 如果同步更新，这里会触发 100 次 DOM 更新
for (let i = 0; i < 100; i++) {
  count.value++
}

// 实际上 Vue 只会更新一次 DOM
// 因为多次修改会被合并到同一个"tick"中
```

### nextTick 的本质

`nextTick` 就是把回调函数推入微任务队列，等 DOM 更新后执行。

## 使用场景

### 1. 获取更新后的 DOM

```vue
<script setup>
import { ref, nextTick } from 'vue'

const list = ref([])
const listRef = ref(null)

async function addItem() {
  list.value.push('new item')

  await nextTick()
  // 现在可以安全地操作 DOM
  listRef.value.scrollTop = listRef.value.scrollHeight
}
</script>
```

### 2. 在 mounted 后操作 DOM

```vue
<script setup>
import { ref, onMounted, nextTick } from 'vue'

const inputRef = ref(null)

onMounted(async () => {
  await nextTick()
  inputRef.value.focus()
})
</script>
```

### 3. 动态组件加载后操作

```vue
<script setup>
import { ref, nextTick } from 'vue'

const showModal = ref(false)
const modalRef = ref(null)

async function openModal() {
  showModal.value = true

  await nextTick()
  // Modal 组件已渲染，可以操作
  modalRef.value.focus()
}
</script>
```

## 两种使用方式

### 1. 回调函数方式

```javascript
import { nextTick } from 'vue'

count.value++
nextTick(() => {
  console.log('DOM 已更新')
})
```

### 2. Promise / async-await 方式

```javascript
import { nextTick } from 'vue'

count.value++
await nextTick()
console.log('DOM 已更新')
```

## 实现原理

### Vue 3 的实现（简化版）

```javascript
const resolvedPromise = Promise.resolve()
let currentFlushPromise = null

export function nextTick(fn) {
  const p = currentFlushPromise || resolvedPromise
  return fn ? p.then(fn) : p
}
```

Vue 3 直接使用 `Promise.resolve().then()` 实现，利用微任务的特性。

### Vue 2 的实现（降级策略）

Vue 2 会根据环境做降级处理：

```javascript
// 优先级从高到低
1. Promise.then        // 微任务
2. MutationObserver    // 微任务
3. setImmediate        // 宏任务（IE/Node）
4. setTimeout          // 宏任务（兜底）
```

## 经典面试题

### 题目 1：输出顺序

```vue
<script setup>
import { ref, nextTick } from 'vue'

const count = ref(0)

function test() {
  console.log('1')

  count.value++

  console.log('2')

  nextTick(() => {
    console.log('3')
  })

  console.log('4')

  Promise.resolve().then(() => {
    console.log('5')
  })

  console.log('6')
}
</script>
```

**输出**：`1 2 4 6 3 5`（或 `1 2 4 6 5 3`，取决于 nextTick 和 Promise 的入队顺序）

**分析**：
- 1, 2, 4, 6 是同步代码，先执行
- 3 和 5 都是微任务，在同步代码后执行
- nextTick 内部也是 Promise，所以 3 和 5 的顺序取决于谁先入队

### 题目 2：nextTick 和 setTimeout 的区别

```javascript
count.value++

nextTick(() => {
  console.log('nextTick')  // 微任务，在 DOM 更新后立即执行
})

setTimeout(() => {
  console.log('setTimeout') // 宏任务，在下一轮事件循环执行
}, 0)
```

**输出**：`nextTick` 先于 `setTimeout`

### 题目 3：多次 nextTick

```javascript
nextTick(() => console.log('1'))
nextTick(() => console.log('2'))
nextTick(() => console.log('3'))

// 输出：1 2 3（按入队顺序执行）
```

## 与其他 API 的关系

### nextTick vs watchEffect

```javascript
// watchEffect 会在依赖变化后自动执行，包含 DOM 更新后的时机
watchEffect(() => {
  console.log(count.value)
}, { flush: 'post' })  // 'post' 表示在 DOM 更新后执行
```

### nextTick vs onUpdated

```javascript
// onUpdated 是生命周期钩子，每次 DOM 更新后都会执行
onUpdated(() => {
  console.log('DOM 已更新')
})

// nextTick 是一次性的，手动调用
```

## 最佳实践

1. **需要获取更新后的 DOM 时使用 nextTick**
2. **优先使用 async/await 语法**，代码更清晰
3. **避免过度使用**，如果可以用 `watch` 或 `watchEffect` 解决，优先使用它们
4. **注意 nextTick 返回的是 Promise**，可以配合 `try/catch`

## 面试要点

1. **Vue 的 DOM 更新是异步的**，在下一个 tick 批量更新
2. **nextTick 将回调推入微任务队列**
3. **Vue 3 直接用 Promise.then**，Vue 2 有降级策略
4. **使用场景**：获取更新后的 DOM、动态组件操作、滚动定位等
5. **nextTick 返回 Promise**，支持 async/await

## 深入阅读

- [Vue 3 官方文档 - nextTick](https://cn.vuejs.org/api/general.html#nexttick)
- [Vue 源码 - nextTick 实现](https://github.com/vuejs/core/blob/main/packages/runtime-core/src/scheduler.ts)
