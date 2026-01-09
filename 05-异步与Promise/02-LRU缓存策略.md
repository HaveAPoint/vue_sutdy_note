# LRU 缓存策略算法

## 什么是 LRU？

**LRU (Least Recently Used)**：最近最少使用。

**核心思想**：
如果一个数据在最近一段时间没有被访问过，那么在将来它被访问的可能性也很小。当空间满了，最先淘汰那个"最久没被用过"的数据。

## 数据结构设计

要实现高效的 LRU（最好 get 和 put 都是 O(1)），通常结合使用：

1. **哈希表 (Map)**：实现 O(1) 的查找。
2. **双向链表**：维护访问顺序。
   - **头部**：最近使用的（热数据）。
   - **尾部**：最久未使用的（冷数据，待淘汰）。

**为什么用 Map？**
JavaScript 的 ES6 `Map` 类底层正好维护了插入顺序（迭代时是按插入顺序），这让利用 Map 实现 LRU 变得非常简单！我们不需要自己手写双向链表。

## 算法实现 (使用 Map)

```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity
    this.cache = new Map() // map 的 key 是有序的
  }

  get(key) {
    if (!this.cache.has(key)) {
      return -1
    }
    
    // 关键点：访问了 key，需将其移动到"最近使用"位置（Map 的尾部）
    const value = this.cache.get(key)
    this.cache.delete(key) // 先删除
    this.cache.set(key, value) // 再重新 set，这就去到了 Map 的最后面
    
    return value
  }

  put(key, value) {
    if (this.cache.has(key)) {
      // 如果 key 已存在，更新值，并移动到最新
      this.cache.delete(key)
    } else if (this.cache.size >= this.capacity) {
      // 如果满了，删除"最久未使用"的（Map 的头部）
      // cache.keys().next().value 获取 Map 的第一个 key
      const oldestKey = this.cache.keys().next().value
      this.cache.delete(oldestKey)
    }
    
    // 插入新值（会放到 Map 的最后面，代表最近使用）
    this.cache.set(key, value)
  }
}
```

## 测试

```javascript
const lru = new LRUCache(2) // 容量为 2

lru.put(1, 1) // cache: {1=1}
lru.put(2, 2) // cache: {1=1, 2=2}

console.log(lru.get(1)) // 返回 1
// cache: {2=2, 1=1}  (1 变为最近使用)

lru.put(3, 3) 
// 容量满了，删除最久未使用的 2
// cache: {1=1, 3=3}

console.log(lru.get(2)) // 返回 -1 (已被淘汰)

lru.put(4, 4)
// 容量满了，删除最久未使用的 1
// cache: {3=3, 4=4}

console.log(lru.get(1)) // 返回 -1
console.log(lru.get(3)) // 返回 3
console.log(lru.get(4)) // 返回 4
```

## 应用场景

1. **浏览器缓存**：缓存最近访问的图片或页面资源。
2. **Vue `keep-alive`**：
   - Vue 的 `<keep-alive>` 组件内部就是用 LRU 算法来缓存组件实例。
   - `include` 匹配的组件会被缓存。
   - 当缓存数量超过 `max` 时，把很久没渲染的组件销毁。
3. **Redis 内存淘汰**：Redis 的内存淘汰策略之一就是 LRU。

## 面试要点

1. **数据结构**：Map + 双向链表（JS 中 Map 自带顺序，可简化）。
2. **O(1) 复杂度**：get 和 put 都必须快。
3. **操作逻辑**：
   - get：存在则移动到最新，不存在返回 -1。
   - put：存在则更新并移动；不存在且未满直接插；不存在且满了删最老再插。

## 深入阅读

- [LeetCode 146. LRU 缓存](https://leetcode.cn/problems/lru-cache/)
