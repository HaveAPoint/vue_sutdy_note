# 手撕题：嵌套 Map 平铺

## 题目描述

将一个多层嵌套的对象（或 Map）平铺成一层。键名用 `.` 连接。

**输入**：
```javascript
const input = {
  a: 1,
  b: {
    c: 2,
    d: {
      e: 3
    }
  }
}
```

**输出**：
```javascript
{
  'a': 1,
  'b.c': 2,
  'b.d.e': 3
}
```

## 实现代码 (递归)

```javascript
function flattenObject(obj, prefix = '', result = {}) {
  // 遍历对象的所有 key
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      const value = obj[key]
      
      // 构造当前的 key (如果有前缀就要拼接)
      const newKey = prefix ? `${prefix}.${key}` : key
      
      if (typeof value === 'object' && value !== null && !Array.isArray(value)) {
        // 如果值是对象（且不是数组，不为null），递归调用
        flattenObject(value, newKey, result)
      } else {
        // 否则（是基本类型或数组），直接赋值
        result[newKey] = value
      }
    }
  }
  return result
}
```

### 测试

```javascript
const data = {
  a: 1,
  b: {
    c: 2,
    d: { e: 3 }
  }
}

console.log(flattenObject(data))
// Output: { a: 1, 'b.c': 2, 'b.d.e': 3 }
```

## 进阶：如何还原？(Unflatten)

将平铺的 key (`a.b.c`) 拆分并重建对象。

```javascript
function unflattenObject(obj) {
  const result = {}
  
  for (const key in obj) {
    const value = obj[key]
    const keys = key.split('.') // ['b', 'd', 'e']
    
    let current = result
    
    // 遍历路径，除了最后一个 key
    for (let i = 0; i < keys.length - 1; i++) {
      const k = keys[i]
      // 如果不存在，创建一个空对象
      if (!current[k]) {
        current[k] = {}
      }
      // 指针下移
      current = current[k]
    }
    
    // 最后一个 key 赋值
    current[keys[keys.length - 1]] = value
  }
  
  return result
}
```
