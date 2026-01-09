# keep-alive 原理与使用

## 作用

`<keep-alive>` 是 Vue 的内置抽象组件，用于**缓存组件实例**，在组件切换时保留状态，避免重复销毁/创建带来的性能消耗。

## 基本用法

```vue
<keep-alive include="User,Post" :max="10">
  <router-view />
</keep-alive>
```

- `include` / `exclude`：根据组件 `name`（优先）或 VNode key 过滤需要缓存的组件。
- `max`：缓存实例的上限，超过时使用 LRU 淘汰最久未使用的实例。

## 生命周期

被缓存的组件不会触发 `unmounted`，而是：
- 首次渲染：`created` → `mounted` → `activated`
- 重新显示：只触发 `activated`
- 被切走：触发 `deactivated`

可在 `activated/deactivated` 做数据刷新或清理定时器。

## 缓存键规则

- 默认 key：`componentVNode.type.__name`（组件 name）。
- 手动指定：在组件使用处加 `:key`，变化时会销毁旧实例并创建新实例（常用来强制重置状态）。

## 使用场景

- 频繁切换的路由页面或标签页，需要保留输入和滚动位置。
- 表单/富文本/图表等初始化开销大但数据量不大的组件。

## 避坑

- 不需要缓存时不要盲目套 keep-alive，会占用内存。
- 组件内要在 `deactivated` 清理定时器、监听器；在 `activated` 恢复。
- 依赖路由参数的组件，切换 params 时要配合 `watch` 或改变 key 触发刷新。
