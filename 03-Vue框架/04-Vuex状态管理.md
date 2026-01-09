# Vuex 状态管理

> **注意：Vue 3 官方推荐使用 Pinia，但 Vuex 4 依然被支持。本篇介绍 Vuex 核心概念。**

## 1. 五大核心概念

### State
**单一状态树**，存储应用的数据源。

```javascript
state: {
  count: 0
}
```

### Getters
**Store 的计算属性**。当 state 变化时自动重算，且有缓存。

```javascript
getters: {
  doubleCount: (state) => state.count * 2
}
```

### Mutations
**同步**修改 State 的唯一途径。**必须是同步函数**。
便于 Devtools 追踪状态变化快照。

```javascript
mutations: {
  increment(state, payload) {
    state.count += payload
  }
}
// 调用：store.commit('increment', 10)
```

### Actions
处理**异步**操作（API 请求），然后提交 mutation。

```javascript
actions: {
  async fetchUser({ commit }) {
    const user = await api.getUser()
    commit('setUser', user)
  }
}
// 调用：store.dispatch('fetchUser')
```

### Modules
将 Store 分割成模块，每个模块拥有自己的 state, mutations, actions。

```javascript
modules: {
  user: userModule,
  cart: cartModule
}
```

---

## 2. Vuex vs Pinia (面试必问)

| 特性 | Vuex 4 | Pinia |
|---|---|---|
| **API** | mutations, actions, getters | **actions, getters** (无 mutations) |
| **异步** | actions | actions (同步异步都行) |
| **层级** | 树状结构 (Modules 嵌套) | **扁平化** (独立的 Store) |
| **TS 支持** | 极其痛苦 | **原生 TS，类型推导极佳** |
| **体积** | 较大 | 极小 (1kb) |

### 为什么 Pinia 去掉了 Mutations？
Vuex 中 mutation 只是为了区分同步和异步，以便 Devtools 追踪。但现在 Devtools 已经通过监听 action 的各种钩子实现了同样的功能，所以 strict mode 的 mutation 变成了多余的样板代码。

---

## 3. 面试题：Vuex 页面刷新数据丢失怎么办？

**原因**：Vuex 的数据是保存在**内存**中的（JS 堆内存），刷新页面（浏览器进程重启）内存会被释放。

**解决方案**：**持久化存储**。

1. **手动存取**：在 mutation 中同步 `localStorage`。
2. **插件 (推荐)**：使用 `vuex-persistedstate` (Vuex) 或 `pinia-plugin-persistedstate`。

```javascript
// 原理伪代码
store.subscribe((mutation, state) => {
  localStorage.setItem('store', JSON.stringify(state))
})

// 初始化时
const savedState = JSON.parse(localStorage.getItem('store'))
if (savedState) store.replaceState(savedState)
```
