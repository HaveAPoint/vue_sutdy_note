# v-on 事件处理

## 1. 基础用法

`v-on` 用于监听 DOM 事件，并在触发时执行 JavaScript 代码。

- **全写**：`v-on:click="handler"`
- **缩写**：`@click="handler"` (强烈推荐)

### 内联处理器 vs 方法处理器

```vue
<!-- 内联：适合简单逻辑 -->
<button @click="count++">加 1</button>

<!-- 方法：适合复杂逻辑 -->
<button @click="greet">打招呼</button>

<!-- 传参 -->
<button @click="say('Hello', $event)">传参</button>
```

```javascript
function greet(event) {
  // 方法处理器会自动接收原生 DOM 事件对象
  alert(event.target.tagName)
}

function say(msg, event) {
  // 如果内联调用需要 event，需显式传入 $event
  console.log(msg, event)
}
```

## 2. 事件修饰符 (Modifiers)

Vue 提供了点缀符来处理常见的 DOM 事件细节（如阻止冒泡），让我们专注于逻辑代码。

| 修饰符 | 含义 | 原生等价 |
|--------|------|----------|
| `.stop` | 阻止冒泡 | `event.stopPropagation()` |
| `.prevent` | 阻止默认行为 | `event.preventDefault()` |
| `.capture` | 捕获模式 | `addEventListener(..., true)` |
| `.self` | 仅当 event.target 是元素本身时触发 | - |
| `.once` | 只触发一次 | - |
| `.passive` | 滚动性能优化 | `{ passive: true }` |

### 示例

```vue
<!-- 阻止链接跳转 -->
<a href="http://baidu.com" @click.prevent="doSomething">不跳转</a>

<!-- 阻止点击冒泡 -->
<div @click="parentClick">
  <button @click.stop="childClick">不会触发父级</button>
</div>

<!-- 链式调用 -->
<a @click.stop.prevent="doThat">阻止冒泡且阻止默认行为</a>
```

## 3. 按键修饰符

监听键盘事件时常用。

```vue
<!-- 只有按 Enter 键时触发 -->
<input @keyup.enter="submit" />

<!-- Ctrl + Enter -->
<input @keydown.ctrl.enter="submit" />
```

常用别名：`.enter`, `.tab`, `.delete`, `.esc`, `.space`, `.up`, `.down`, `.left`, `.right`。

## 4. 为什么要在 HTML 里监听事件？

有人觉得把 `@click` 写在 HTML 里违背了"关注点分离"（结构与行为分离），但实际上：

1. **便于定位**：扫一眼 HTML 模板就能知道这个元素在这个状态下会干什么。
2. **自动销毁**：ViewModel 销毁时，所有的事件监听器会自动被移除，不用手动 `removeEventListener`，避免内存泄漏。
3. **纯粹的逻辑**：JavaScript 代码里不需要写 `document.querySelector` 去找 DOM，只关注数据逻辑。

## 5. 组件事件

在组件上使用 `v-on` 监听的是**子组件触发的自定义事件**。

```vue
<!-- 监听子组件 emit 的 'delete' 事件 -->
<UserTable @delete="handleDelete" />
```

如果想要监听组件根元素的原生事件（如 click），需要使用 `.native` 修饰符（**Vue 2**）。
**注意：Vue 3 移除了 `.native` 修饰符**。Vue 3 会自动判断：如果组件 `emits` 选项里没声明这个事件，就把它当作原生事件监听。
