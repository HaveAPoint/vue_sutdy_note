# 手写 call / apply / bind / new / 深拷贝

## 实现 call

```javascript
Function.prototype.myCall = function(context, ...args) {
  context = context ?? globalThis // null/undefined 兜底为全局对象
  const fnKey = Symbol('fn')
  context[fnKey] = this            // 暂存要调用的函数
  const result = context[fnKey](...args)
  delete context[fnKey]
  return result
}
```

## 实现 apply

```javascript
Function.prototype.myApply = function(context, argsArray) {
  context = context ?? globalThis
  const fnKey = Symbol('fn')
  context[fnKey] = this
  const result = Array.isArray(argsArray)
    ? context[fnKey](...argsArray)
    : context[fnKey]()
  delete context[fnKey]
  return result
}
```

## 实现 bind

```javascript
Function.prototype.myBind = function(context, ...bindArgs) {
  const self = this
  function boundFn(...callArgs) {
    // new 优先级高于 bind：作为构造函数时 this 应该指向实例
    const thisArg = this instanceof boundFn ? this : context ?? globalThis
    return self.apply(thisArg, [...bindArgs, ...callArgs])
  }
  boundFn.prototype = Object.create(self.prototype) // 继承原型
  return boundFn
}
```

## 实现 new

```javascript
function myNew(Constructor, ...args) {
  const obj = Object.create(Constructor.prototype) // 1. 创建对象并挂上原型
  const result = Constructor.apply(obj, args)       // 2. 执行构造函数，绑定 this
  return (result !== null && typeof result === 'object') ? result : obj // 3. 返回
}
```

## 深拷贝

- 简单场景：`JSON.parse(JSON.stringify(obj))`，但会丢失函数/Symbol/循环引用。
- 推荐方案：递归 + WeakMap 记录循环引用，处理特殊对象（Date/RegExp/Map/Set）。完整实现见 `01-JavaScript基础/09-深拷贝与浅拷贝.md`。
