# Vue Router 路由

## 什么是前端路由？

前端路由是**单页应用 (SPA)** 的核心：URL 变化时，不刷新页面，而是由 JavaScript 控制显示不同的组件。

## 两种路由模式

### 1. Hash 模式（默认）

```javascript
// URL 示例：https://example.com/#/user/123
const router = createRouter({
  history: createWebHashHistory(),
  routes: [...]
})
```

**原理**：监听 `window.onhashchange` 事件。

**特点**：
- URL 带 `#` 号
- 兼容性好（支持 IE9+）
- 不需要服务器配置
- SEO 不友好

### 2. History 模式

```javascript
// URL 示例：https://example.com/user/123
const router = createRouter({
  history: createWebHistory(),
  routes: [...]
})
```

**原理**：使用 HTML5 History API (`pushState`, `replaceState`, `popstate`)。

**特点**：
- URL 干净美观
- 需要服务器配置（所有路由返回 index.html）
- SEO 相对友好

**Nginx 配置**：
```nginx
location / {
  try_files $uri $uri/ /index.html;
}
```

## 基本配置

### 路由定义

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: () => import('@/views/Home.vue') // 懒加载
  },
  {
    path: '/user/:id',  // 动态路由
    name: 'User',
    component: () => import('@/views/User.vue'),
    props: true  // 将 params 作为 props 传递
  },
  {
    path: '/dashboard',
    component: () => import('@/views/Dashboard.vue'),
    children: [  // 嵌套路由
      {
        path: 'profile',
        component: () => import('@/views/Profile.vue')
      },
      {
        path: 'settings',
        component: () => import('@/views/Settings.vue')
      }
    ]
  },
  {
    path: '/:pathMatch(.*)*',  // 404 页面
    name: 'NotFound',
    component: () => import('@/views/NotFound.vue')
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

### 在组件中使用

```vue
<script setup>
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()  // 路由实例
const route = useRoute()    // 当前路由信息

// 获取路由参数
console.log(route.params.id)  // 动态路由参数
console.log(route.query.page) // 查询参数 ?page=1

// 编程式导航
function goToUser(id) {
  router.push(`/user/${id}`)
  // 或
  router.push({ name: 'User', params: { id } })
}

function goBack() {
  router.go(-1)
}

function replaceRoute() {
  router.replace('/home')  // 不留历史记录
}
</script>

<template>
  <!-- 声明式导航 -->
  <router-link to="/">首页</router-link>
  <router-link :to="{ name: 'User', params: { id: 123 } }">用户</router-link>

  <!-- 路由出口 -->
  <router-view />
</template>
```

## 路由守卫

### 1. 全局守卫

```javascript
// 全局前置守卫
router.beforeEach((to, from, next) => {
  const isAuthenticated = localStorage.getItem('token')

  if (to.meta.requiresAuth && !isAuthenticated) {
    next('/login')  // 未登录跳转登录页
  } else {
    next()  // 放行
  }
})

// 全局后置钩子
router.afterEach((to, from) => {
  document.title = to.meta.title || '默认标题'
})
```

### 2. 路由独享守卫

```javascript
const routes = [
  {
    path: '/admin',
    component: Admin,
    beforeEnter: (to, from, next) => {
      if (userRole !== 'admin') {
        next('/403')
      } else {
        next()
      }
    }
  }
]
```

### 3. 组件内守卫

```vue
<script setup>
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'

// 路由更新时（如 /user/1 → /user/2）
onBeforeRouteUpdate((to, from) => {
  // 重新获取数据
  fetchUser(to.params.id)
})

// 离开当前路由前
onBeforeRouteLeave((to, from) => {
  if (hasUnsavedChanges) {
    const answer = window.confirm('有未保存的更改，确定离开？')
    if (!answer) return false  // 取消导航
  }
})
</script>
```

## 路由元信息 (meta)

```javascript
const routes = [
  {
    path: '/admin',
    component: Admin,
    meta: {
      requiresAuth: true,
      title: '管理后台',
      roles: ['admin', 'superadmin']
    }
  }
]

// 在守卫中使用
router.beforeEach((to, from, next) => {
  if (to.meta.requiresAuth) {
    // 检查权限...
  }
  next()
})
```

## 路由懒加载

```javascript
// 方式 1：动态 import
component: () => import('@/views/User.vue')

// 方式 2：webpackChunkName（webpack）
component: () => import(/* webpackChunkName: "user" */ '@/views/User.vue')

// 方式 3：分组打包
const UserDetails = () => import(/* webpackChunkName: "user" */ './UserDetails.vue')
const UserPosts = () => import(/* webpackChunkName: "user" */ './UserPosts.vue')
```

## 滚动行为

```javascript
const router = createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    // 返回上一页时恢复滚动位置
    if (savedPosition) {
      return savedPosition
    }
    // 有锚点时滚动到锚点
    if (to.hash) {
      return { el: to.hash }
    }
    // 默认滚动到顶部
    return { top: 0 }
  }
})
```

## 动态路由

### 添加路由

```javascript
// 动态添加路由（常用于权限控制）
router.addRoute({
  path: '/admin',
  name: 'Admin',
  component: Admin
})

// 添加嵌套路由
router.addRoute('Dashboard', {
  path: 'analytics',
  component: Analytics
})
```

### 删除路由

```javascript
// 方式 1：通过 addRoute 的返回值
const removeRoute = router.addRoute(route)
removeRoute()  // 删除

// 方式 2：通过 name 删除
router.removeRoute('Admin')
```

## 面试常见问题

### 1. Hash 和 History 模式的区别？

| 特性 | Hash | History |
|------|------|---------|
| URL | 带 `#` | 干净 |
| 兼容性 | IE9+ | IE10+ |
| 服务器配置 | 不需要 | 需要 |
| SEO | 不友好 | 相对友好 |
| 原理 | hashchange | pushState/popstate |

### 2. 如何实现路由权限控制？

```javascript
// 方案 1：路由守卫
router.beforeEach((to, from, next) => {
  const userRoles = store.state.user.roles
  const requiredRoles = to.meta.roles

  if (requiredRoles && !requiredRoles.some(r => userRoles.includes(r))) {
    next('/403')
  } else {
    next()
  }
})

// 方案 2：动态路由
async function initRoutes() {
  const routes = await fetchUserRoutes()
  routes.forEach(route => router.addRoute(route))
}
```

### 3. 导航守卫执行顺序？

```
1. 导航被触发
2. 调用离开组件的 onBeforeRouteLeave
3. 调用全局 beforeEach
4. 调用路由独享 beforeEnter
5. 解析异步组件
6. 调用进入组件的 onBeforeRouteUpdate（如果是复用组件）
7. 调用全局 afterEach
8. DOM 更新
```

## 面试要点

1. **Hash 用 `#`，History 用 HTML5 API**
2. **History 模式需要服务器支持**
3. **路由守卫**：全局、路由独享、组件内三种
4. **动态路由**用于权限控制
5. **路由懒加载**优化首屏性能

## 深入阅读

- [Vue Router 官方文档](https://router.vuejs.org/zh/)
