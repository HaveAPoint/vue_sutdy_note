# Vue 2 与 Vue 3 对比

## 1. 核心响应式原理 (面试必问)

### Vue 2 - Object.defineProperty
- **原理**：通过 `Object.defineProperty` 劫持对象属性的 getter/setter。
- **缺陷**：
  1. **无法检测对象属性的添加或删除**（需要用 `Vue.set` / `Vue.delete`）。
  2. **无法检测数组索引和长度的变化**（通过重写 push/pop 等 7 个方法变相实现，但无法劫持 `arr[0] = 1`）。
  3. **即使没用到的属性也会被初始化劫持**（递归遍历），对象层级深时性能差。

### Vue 3 - Proxy
- **原理**：使用 ES6 `Proxy` 代理整个对象。
- **优势**：
  1. **全方位劫持**：支持属性增删、数组索引修改、Map/Set 等数据结构。
  2. **懒代理**：只有访问对象深层属性时才递归代理，初始化性能更好。

## 2. API 风格

### Vue 2 - Options API (选项式)
代码按 `data`, `methods`, `computed`, `watch` 分类。
- **缺点**：逻辑分散。实现一个功能（如"搜索"），代码要跳来跳去写在 data 和 methods 里，难以维护（"反复横跳"）。

### Vue 3 - Composition API (组合式)
代码按**功能逻辑**组织。
- **优点**：同功能的代码（数据+方法）写在一起（配合 setup），容易复用（Composables）。
- **Tree Shaking**：按需引入 API，打包体积更小。

## 3. 生命周期变更

| Vue 2 (Options API) | Vue 3 (Setup) |
|---|---|
| beforeCreate | (setup 即创建) |
| created | (setup 即创建) |
| beforeMount | onBeforeMount |
| mounted | onMounted |
| beforeUpdate | onBeforeUpdate |
| updated | onUpdated |
| beforeDestroy | **onBeforeUnmount** |
| destroyed | **onUnmounted** |

## 4. 其他重要区别

### Fragment (片段)
- **Vue 2**：组件模板**必须**有一个根节点。
- **Vue 3**：支持多个根节点（自动包裹在 Fragment 中），减少了无意义的 wrapper div。

### Teleport (传送门)
- **Vue 3**：原生支持将组件（如 Modal 弹窗）渲染到 Body 下，解决 `z-index` 层级和 `overflow: hidden` 问题。

### TypeScript 支持
- **Vue 2**：对 TS 支持较差（混入、this 推导困难）。
- **Vue 3**：源码用 TS 重写，对 TS 支持极好。

### Diff 算法优化
- **静态标记 (PatchFlag)**：Vue 3 会标记静态节点（永远不变的 div），Diff 时直接跳过，只对比动态节点。
- **事件缓存**：缓存事件处理函数，避免不必要的组件更新。

## 总结

| 特性 | Vue 2 | Vue 3 |
|---|---|---|
| 响应式 | Object.defineProperty | Proxy |
| API | Options API | Composition API + Options API |
| 根节点 | 必须 1 个 | 可以多个 |
| TS 支持 | 弱 | 强 |
| 性能 | 较好 | 更好 (快 1.2~2 倍) |
| 兼容性 | IE9+ | 不支持 IE11 (因 Proxy) |
