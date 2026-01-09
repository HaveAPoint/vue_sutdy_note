# v-if 与 v-show 的区别

## 核心区别

| 特性 | v-if | v-show |
|------|------|--------|
| **编译过程** | 惰性渲染。条件为假时，**DOM 压根不存在**。 | **始终渲染 DOM**，只是 CSS 切换。 |
| **CSS 原理** | 节点的创建 / 销毁 | `display: none` / `display: block` |
| **切换开销** | **高** (涉及 DOM 插入删除) | **低** (只改样式) |
| **初始开销** | **低** (假就不渲染) | **高** (不管真假都渲染) |

## 适用场景

### 1. 推荐 v-if
- **条件很少改变**（如：用户登录状态、权限控制）。
- **由数据驱动的大块内容**（如渲染很大的组件树）。
- **组件需要触发生命周期**（v-if 切换会触发 mounted/unmounted，v-show 不会）。

### 2. 推荐 v-show
- **需要非常频繁地切换**（如：手风琴菜单、Tab 切换、折叠面板）。
- **元素初始化耗时昂贵**（如复杂的图表组件，用 v-show 隐藏，避免每次显示都重新初始化）。

## 经典面试题

### v-if 和 v-for 优先级问题

**Vue 2**: `v-for` > `v-if`
- **问题**：每次循环都会执行 if 判断，浪费性能。
- **例子**：`<div v-for="item in list" v-if="item.active">` ->哪怕只渲染一个，也会遍历整个 list。

**Vue 3**: `v-if` > `v-for`
- **问题**：if 拿不到 for 里的 item 变量，因为 if 先执行。

**最佳实践**：
**不要把 v-if 和 v-for 写在同一个元素上！**
请使用 `<template>` 标签包裹或计算属性 (computed) 过滤 list。

```vue
<!-- 推荐写法 1：计算属性 (最佳) -->
<div v-for="item in activeList" :key="item.id"></div>

<!-- 推荐写法 2：外层 template 包裹 -->
<template v-for="item in list">
  <div v-if="item.active" :key="item.id"></div>
</template>
```
