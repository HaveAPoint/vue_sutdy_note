# Map、Set 与 WeakMap

## 概述

ES6 引入的新数据结构，解决 Object 和 Array 无法优雅处理的场景。

| 数据结构 | 特点 | 用途 |
|----------|------|------|
| `Map` | 键值对，键可以是任意类型 | 复杂键的映射 |
| `Set` | 值的集合，自动去重 | 去重、集合运算 |
| `WeakMap` | 键必须是对象，弱引用 | 私有数据、缓存 |
| `WeakSet` | 值必须是对象，弱引用 | 对象标记 |

---

## Map

### 基本用法

```javascript
const map = new Map()

// 设置
map.set('name', 'Alice')
map.set(1, 'one')
map.set({ key: 'obj' }, 'object value')

// 获取
map.get('name')  // 'Alice'
map.get(1)       // 'one'

// 检查
map.has('name')  // true

// 删除
map.delete('name')

// 大小
map.size  // 2

// 清空
map.clear()
```

### Map vs Object

| 特性 | Map | Object |
|------|-----|--------|
| 键类型 | 任意类型 | 字符串/Symbol |
| 键顺序 | 插入顺序 | 不保证（ES6 后基本有序） |
| 大小 | `map.size` | `Object.keys(obj).length` |
| 迭代 | 直接可迭代 | 需要 Object.keys/values |
| 性能 | 频繁增删更优 | 静态数据更优 |

```javascript
// 对象作为键 - Object 做不到
const userPermissions = new Map()
const user1 = { id: 1, name: 'Alice' }
const user2 = { id: 2, name: 'Bob' }

userPermissions.set(user1, ['read', 'write'])
userPermissions.set(user2, ['read'])

userPermissions.get(user1)  // ['read', 'write']
```

### Map 遍历

```javascript
const map = new Map([
  ['a', 1],
  ['b', 2],
  ['c', 3]
])

// for...of
for (const [key, value] of map) {
  console.log(key, value)
}

// forEach
map.forEach((value, key) => {
  console.log(key, value)
})

// 转数组
const entries = [...map]           // [['a', 1], ['b', 2], ['c', 3]]
const keys = [...map.keys()]       // ['a', 'b', 'c']
const values = [...map.values()]   // [1, 2, 3]
```

---

## Set

### 基本用法

```javascript
const set = new Set()

// 添加（自动去重）
set.add(1)
set.add(2)
set.add(1)  // 重复，不会添加

console.log(set.size)  // 2

// 检查
set.has(1)  // true

// 删除
set.delete(1)

// 遍历
for (const value of set) {
  console.log(value)
}
```

### 数组去重

```javascript
const arr = [1, 2, 2, 3, 3, 3, 4]
const unique = [...new Set(arr)]
console.log(unique)  // [1, 2, 3, 4]

// 字符串去重
const str = 'aabbcc'
const uniqueChars = [...new Set(str)].join('')
console.log(uniqueChars)  // 'abc'
```

### 集合运算

```javascript
const setA = new Set([1, 2, 3, 4])
const setB = new Set([3, 4, 5, 6])

// 并集
const union = new Set([...setA, ...setB])
// Set(6) {1, 2, 3, 4, 5, 6}

// 交集
const intersection = new Set([...setA].filter(x => setB.has(x)))
// Set(2) {3, 4}

// 差集
const difference = new Set([...setA].filter(x => !setB.has(x)))
// Set(2) {1, 2}
```

---

## WeakMap

### 核心特点

1. **键必须是对象**（不能是基本类型）
2. **弱引用**：不阻止垃圾回收
3. **不可遍历**：没有 size、keys、values 等方法

### 为什么需要弱引用？

```javascript
// 普通 Map - 可能导致内存泄漏
const map = new Map()
let obj = { name: 'test' }
map.set(obj, 'data')

obj = null  // 对象仍被 map 引用，不会被回收

// WeakMap - 不会内存泄漏
const weakMap = new WeakMap()
let obj2 = { name: 'test' }
weakMap.set(obj2, 'data')

obj2 = null  // 对象可以被垃圾回收
```

### 应用场景

#### 1. 私有数据

```javascript
const privateData = new WeakMap()

class Person {
  constructor(name, age) {
    privateData.set(this, { name, age })
  }

  getName() {
    return privateData.get(this).name
  }

  getAge() {
    return privateData.get(this).age
  }
}

const person = new Person('Alice', 25)
person.getName()  // 'Alice'
// 无法直接访问 privateData
```

#### 2. DOM 节点关联数据

```javascript
const nodeData = new WeakMap()

function bindData(element, data) {
  nodeData.set(element, data)
}

function getData(element) {
  return nodeData.get(element)
}

const div = document.querySelector('#myDiv')
bindData(div, { clicks: 0 })

// 当 div 被移除时，关联数据自动被回收
```

#### 3. 对象缓存

```javascript
const cache = new WeakMap()

function expensive(obj) {
  if (cache.has(obj)) {
    return cache.get(obj)
  }

  const result = /* 耗时计算 */ obj.value * 2
  cache.set(obj, result)
  return result
}
```

---

## WeakSet

类似 WeakMap，但存储的是值而非键值对。

```javascript
const weakSet = new WeakSet()
let obj = { id: 1 }

weakSet.add(obj)
weakSet.has(obj)  // true

obj = null  // 对象可被回收
```

### 应用场景：对象标记

```javascript
const processed = new WeakSet()

function process(obj) {
  if (processed.has(obj)) {
    return  // 已处理过
  }

  // 处理对象
  console.log('Processing:', obj)
  processed.add(obj)
}
```

---

## 对比总结

| | Map | WeakMap | Set | WeakSet |
|---|-----|---------|-----|---------|
| 键类型 | 任意 | 对象 | - | - |
| 值类型 | 任意 | 任意 | 任意 | 对象 |
| 可遍历 | ✅ | ❌ | ✅ | ❌ |
| size 属性 | ✅ | ❌ | ✅ | ❌ |
| 弱引用 | ❌ | ✅ | ❌ | ✅ |
| 内存泄漏 | 可能 | 不会 | 可能 | 不会 |

## 面试要点

1. **Map 的键可以是任意类型**，Object 只能是字符串/Symbol
2. **Set 自动去重**，常用于数组去重
3. **WeakMap/WeakSet 是弱引用**，不阻止垃圾回收
4. WeakMap 适合存储私有数据和缓存
5. Weak* 系列不可遍历

## 深入阅读

- [MDN - Map](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map)
- [MDN - WeakMap](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)
