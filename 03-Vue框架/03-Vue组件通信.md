# Vue 组件通信方式全解

## 1. props / emit (父子通信)

最基础、最常用的方式。

- **父传子**：`props`
- **子传父**：`$emit`

```vue
<!-- 父组件 -->
<Child :msg="message" @change="handleChange" />

<!-- 子组件 -->
<script setup>
defineProps(['msg'])
const emit = defineEmits(['change'])
emit('change', 123)
</script>
```

## 2. v-model (父子双向)

**Vue 3** 支持多个 v-model。

```vue
<!-- 父组件 -->
<Child v-model="count" v-model:title="pageTitle" />

<!-- 子组件 -->
<script setup>
defineProps(['modelValue', 'title'])
defineEmits(['update:modelValue', 'update:title'])
// 更新
emit('update:modelValue', newValue)
</script>
```

## 3. provide / inject (依赖注入)

**适用**：祖先组件 -> 后代组件（跨多层）。
**特点**：非响应式数据需要包装成 `ref` 才能响应。

```vue
<!-- 祖先 -->
<script setup>
import { provide, ref } from 'vue'
provide('theme', ref('dark'))
</script>

<!-- 后代 (任意层级) -->
<script setup>
import { inject } from 'vue'
const theme = inject('theme')
</script>
```

## 4. ref / defineExpose (父调子)

父组件直接访问子组件实例。

**注意**：Vue 3 `<script setup>` 默认是**封闭**的，必须用 `defineExpose` 暴露属性。

```vue
<!-- 子组件 -->
<script setup>
const count = ref(0)
defineExpose({ count }) // 暴露出去
</script>

<!-- 父组件 -->
<script setup>
const childRef = ref(null)
// onMounted 后访问
console.log(childRef.value.count)
</script>

<template>
  <Child ref="childRef" />
</template>
```

## 5. attrs (透传属性)

**适用**：父组件传递的属性，子组件没有在 `props` 中声明，会自动绑定到子组件的根节点上（如 class, style, id）。

```vue
<!-- 子组件中访问 -->
<script setup>
import { useAttrs } from 'vue'
const attrs = useAttrs()
</script>
```

## 6. Pinia / Vuex (全局状态)

**适用**：任意组件间通信，复杂应用的状态管理。

```javascript
const store = useUserStore()
store.userInfo = { name: 'New' } // 修改
```

## 7. EventBus (Vue 3 已移除)

Vue 2 中常用的 `new Vue()` 作为总线。
**Vue 3 推荐使用**：`mitt` 等第三方库，或者直接用 Pinia。

---

## 总结：通信方式选择

| 场景 | 推荐方案 |
|------|----------|
| **父子** | `props` + `emit` |
| **父子 (表单)** | `v-model` |
| **祖孙 (跨层)** | `provide` + `inject` |
| **兄弟/全局** | `Pinia` (首选) |
| **父调子方法** | `ref` + `defineExpose` |
