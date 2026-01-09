# 常用 Hooks (Composables)

Vue 3 组合式 API 的精髓在于**逻辑复用**。以下是高频实用的 Composables。

## 1. useMouse (鼠标追踪)
入门级示例，复用事件监听逻辑。

```javascript
import { ref, onMounted, onUnmounted } from 'vue'

export function useMouse() {
  const x = ref(0)
  const y = ref(0)

  function update(event) {
    x.value = event.pageX
    y.value = event.pageY
  }

  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))

  return { x, y }
}
```

## 2. useRequest (异步请求)
封装 loading、error 和数据状态。

```javascript
import { ref } from 'vue'

export function useRequest(apiFn) {
  const data = ref(null)
  const loading = ref(false)
  const error = ref(null)

  const run = async (...args) => {
    loading.value = true
    error.value = null
    try {
      data.value = await apiFn(...args)
    } catch (err) {
      error.value = err
    } finally {
      loading.value = false
    }
  }

  return { data, loading, error, run }
}
```

## 3. useLocalStorage (持久化状态)
自动同步 localStorage 的 ref。

```javascript
import { ref, watch } from 'vue'

export function useLocalStorage(key, defaultValue) {
  const storedValue = localStorage.getItem(key)
  const data = ref(storedValue ? JSON.parse(storedValue) : defaultValue)

  watch(data, (newValue) => {
    if (newValue === null || newValue === undefined) {
      localStorage.removeItem(key)
    } else {
      localStorage.setItem(key, JSON.stringify(newValue))
    }
  }, { deep: true }) // deep 确保对象内部变化也能触发

  return data
}
```

## 4. useEventListener (自动清理事件)
不用手动在 onUnmounted 里 removeEventListener。

```javascript
import { onMounted, onUnmounted } from 'vue'

export function useEventListener(target, event, callback) {
  onMounted(() => target.addEventListener(event, callback))
  onUnmounted(() => target.removeEventListener(event, callback))
}
```

## 5. VueUse 库 (推荐)

不要重复造轮子！**VueUse** 是 Vue 3 生态必备的工具库，提供了 200+ 个 Hooks。

- `useTitle`: 动态修改网页标题
- `useDark`: 黑暗模式切换
- `useClipboard`: 剪贴板操作
- `useDebounceFn`: 防抖函数
- `useInterSectionObserver`: 元素可见性检测（懒加载核心）
