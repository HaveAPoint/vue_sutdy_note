# key 的作用与 Diff 算法

## key 是什么？

`key` 是 Vue 用于**标识 VNode 唯一性**的特殊属性，主要用于 `v-for` 列表渲染。

```vue
<template>
  <li v-for="item in list" :key="item.id">
    {{ item.name }}
  </li>
</template>
```

## 为什么需要 key？

### 没有 key 的问题

```vue
<template>
  <!-- 不推荐：使用 index 作为 key -->
  <li v-for="(item, index) in list" :key="index">
    <input type="text" />
    {{ item.name }}
  </li>
</template>
```

假设原列表是 `[A, B, C]`，删除 B 后变成 `[A, C]`：

**使用 index 作为 key**：
```
原来：0-A, 1-B, 2-C
删除 B 后：0-A, 1-C

Vue 的理解：
- key=0 的内容从 A 变成 A（不变）
- key=1 的内容从 B 变成 C（更新）
- key=2 消失（删除）

结果：复用了错误的 DOM，input 的值可能错位！
```

**使用唯一 id 作为 key**：
```
原来：id1-A, id2-B, id3-C
删除 B 后：id1-A, id3-C

Vue 的理解：
- key=id1 不变
- key=id2 删除
- key=id3 不变

结果：正确删除 B 对应的 DOM
```

## Diff 算法与 key

### Vue 的 Diff 策略

Vue 使用**同层比较**的 Diff 算法，不会跨层级比较节点。

```
旧 VNode 树          新 VNode 树
    A                    A
   / \                  / \
  B   C       →        B   D
 / \                  /
E   F                E

只比较同层：A-A, B-B, C-D, E-E, F-删除
```

### key 在 Diff 中的作用

**没有 key 时**：
- Vue 使用"就地更新"策略
- 尽可能复用相同位置的 DOM 元素
- 只更新元素内容，不移动元素

**有 key 时**：
- Vue 可以准确识别哪些节点是相同的
- 可以**移动**和**复用**正确的 DOM 元素
- 最小化 DOM 操作

### Diff 过程（简化版）

```javascript
// 新旧子节点数组
oldChildren: [A, B, C, D]
newChildren: [D, A, B, C]

// 有 key 时的处理：
// Vue 发现 D 只是位置变了，直接移动 DOM
// 而不是更新 A→D, B→A, C→B, D→C（4 次更新）

// 无 key 时的处理：
// Vue 按位置比较，每个都要更新内容
```

## 正确使用 key

### 推荐做法

```vue
<!-- 使用唯一且稳定的 id -->
<li v-for="item in list" :key="item.id">
  {{ item.name }}
</li>

<!-- 组合多个字段生成唯一 key -->
<li v-for="item in list" :key="`${item.type}-${item.id}`">
  {{ item.name }}
</li>
```

### 不推荐做法

```vue
<!-- 不推荐：使用 index -->
<li v-for="(item, index) in list" :key="index">

<!-- 不推荐：使用随机数（每次渲染都不同） -->
<li v-for="item in list" :key="Math.random()">

<!-- 不推荐：使用可能重复的值 -->
<li v-for="item in list" :key="item.name">
```

## key 的其他用途

### 1. 强制重新渲染组件

```vue
<template>
  <!-- 改变 key 会销毁旧组件，创建新组件 -->
  <UserProfile :key="userId" :user-id="userId" />
</template>

<script setup>
// 当 userId 变化时，组件会完全重新创建
// 而不是复用旧组件（只更新 props）
</script>
```

### 2. 重置组件状态

```vue
<template>
  <form :key="formKey">
    <input v-model="name" />
    <button @click="reset">重置</button>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const formKey = ref(0)

function reset() {
  formKey.value++ // 改变 key，表单组件重新创建，状态重置
}
</script>
```

### 3. 触发过渡动画

```vue
<template>
  <transition name="fade">
    <!-- key 变化时触发离开和进入动画 -->
    <p :key="message">{{ message }}</p>
  </transition>
</template>
```

## 面试经典题

### 题目 1：为什么不推荐用 index 作为 key？

**答案**：

1. **列表顺序改变时**（插入、删除、排序），index 会变化，导致 key 与实际数据不对应
2. **性能问题**：本可以复用的 DOM 被错误地更新
3. **状态错乱**：表单输入、选中状态等可能绑定到错误的元素上

### 题目 2：key 可以重复吗？

**答案**：

- 在**同一个 v-for 循环中不能重复**，否则 Vue 会警告
- 不同的 v-for 循环可以有相同的 key（它们在不同的作用域）

### 题目 3：不写 key 会怎样？

**答案**：

- Vue 3 会自动使用 index 作为 key
- 对于**静态列表**（不会增删改排序）没问题
- 对于**动态列表**可能导致渲染错误和性能问题

## 总结

| 场景 | 推荐 key |
|------|----------|
| 静态列表（不变） | 可以用 index |
| 动态列表（增删改排序） | 必须用唯一 id |
| 组件需要完全重建 | 改变 key 值 |
| 触发过渡动画 | 改变 key 值 |

## 面试要点

1. **key 是 VNode 的唯一标识**，帮助 Vue 识别节点
2. **不推荐用 index**，因为列表变化时会导致 key 与数据不对应
3. **key 影响 Diff 算法**，正确的 key 能优化性能
4. **key 变化会导致组件重新创建**，可用于重置状态
5. **同一 v-for 中 key 不能重复**

## 深入阅读

- [Vue 官方文档 - key](https://cn.vuejs.org/api/built-in-special-attributes.html#key)
- [Vue 3 Diff 算法源码](https://github.com/vuejs/core/blob/main/packages/runtime-core/src/renderer.ts)
