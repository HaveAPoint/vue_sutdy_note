# 虚拟 DOM 与 MVVM

## 1. 为什么引入虚拟 DOM？

### 什么是虚拟 DOM (Virtual DOM)？
它是用**JavaScript 对象**来描述真实 DOM 结构的一种抽象。

```javascript
// 真实 DOM
<div class="box">Hello</div>

// 虚拟 DOM (VNode)
const vnode = {
  tag: 'div',
  props: { class: 'box' },
  children: 'Hello'
}
```

### 引入原因

1. **解决跨平台问题**
   - 真实 DOM 强绑定浏览器。
   - 虚拟 DOM 是纯对象，可以渲染到浏览器（web）、手机 App（Weex/React Native）、甚至服务端（SSR）。

2. **提高渲染效率（避免无效的重绘重排）**
   - **误区**："虚拟 DOM 一定比真实 DOM 快" —— **错！**
   - 直接手动操作 DOM（精细化操作）通常最快，但开发成本太高。
   - 虚拟 DOM 的意义在于：**它保证了性能下限**，并在**可维护性**和**性能**之间达到了很好的平衡。
   - **合并更新**：将多次数据修改合并成一次 DOM 操作。

### 核心流程
1. **Compile**：模板编译成渲染函数（Render Function）。
2. **Mount**：初次渲染，生成 Virtual DOM 树，再转为真实 DOM。
3. **Patch**：数据变化 -> 生成新的 VNode 树 -> Diff 算法比较新旧树 -> 找出差异 -> 局部更新真实 DOM。

---

## 2. MVVM 模式

### 概念

**Model-View-ViewModel** 的缩写。

- **Model (模型)**：数据层（JS 对象），负责业务逻辑和数据存储。
- **View (视图)**：UI 层（DOM / HTML），负责展示。
- **ViewModel (视图模型)**：中间层（Vue 实例），连接 View 和 Model 的桥梁。

### 图解

```
View (DOM)  <─── 绑定 ───>  ViewModel (Vue)  <─── 读写 ───>  Model (Data)
```

1. **DOM Listeners**：监听 View 层事件，修改 Model。
2. **Data Bindings**：监听 Model 数据变化，更新 View。

### MVVM 的好处

1. **低耦合**：View 和 Model 彻底分离，互不依赖。
2. **可复用**：逻辑（Model）可以复用，不依赖具体视图。
3. **自动化**：双向绑定让开发者无需手动操作 DOM，专注数据逻辑。

### 与 MVC 的区别

- **MVC (Model-View-Controller)**：
  - Controller 负责业务逻辑，手动操作 DOM 更新 View。
  - 数据流通常是单向的。
- **MVVM**：
  - ViewModel **自动**处理 DOM 更新（响应式系统）。
  - 主要区别在于**数据绑定的自动化**。

---

## 3. 面试相关

### 为了提升性能，一定需要虚拟 DOM 吗？
不一定。Svelte 框架就没有虚拟 DOM，它在编译阶段直接生成操作 DOM 的代码，性能也非常强悍。Vue 和 React 使用虚拟 DOM 是为了灵活性（运行时动态性）和跨平台能力。

### 为什么 Vue 2 的 Diff 算法比 React 简单？
Vue 是**响应式**的，它知道具体哪个组件变了，Diff 范围可以精确到组件级。React（Fiber 之前）通常需要从根节点 Diff 或配合 shouldComponentUpdate 优化。
