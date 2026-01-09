# this 指向与输出题

## 本质

**this 是函数执行时的上下文对象**，它的值取决于函数的**调用方式**，而不是定义位置。

## this 绑定规则（优先级从高到低）

### 1. new 绑定

使用 `new` 调用构造函数时，`this` 指向新创建的对象。

```javascript
function Person(name) {
  this.name = name
}

const person = new Person('Tom')
console.log(person.name)  // 'Tom'
```

### 2. 显式绑定 (call/apply/bind)

使用 `call`、`apply`、`bind` 明确指定 `this`。

```javascript
function greet() {
  console.log(`Hello, ${this.name}`)
}

const user = { name: 'Alice' }

greet.call(user)   // Hello, Alice
greet.apply(user)  // Hello, Alice

const boundGreet = greet.bind(user)
boundGreet()       // Hello, Alice
```

### 3. 隐式绑定

函数作为对象方法调用时，`this` 指向该对象。

```javascript
const obj = {
  name: 'Bob',
  sayName() {
    console.log(this.name)
  }
}

obj.sayName()  // 'Bob'
```

### 4. 默认绑定

独立函数调用时，`this` 指向全局对象（严格模式下为 `undefined`）。

```javascript
function fn() {
  console.log(this)
}

fn()  // window (浏览器) 或 global (Node.js)
      // 严格模式下为 undefined
```

## 箭头函数的 this

**箭头函数没有自己的 this**，它继承定义时外层作用域的 `this`。

```javascript
const obj = {
  name: 'Arrow',
  regularFn: function() {
    console.log(this.name)  // 'Arrow'
  },
  arrowFn: () => {
    console.log(this.name)  // undefined (继承外层，这里是全局)
  },
  nested: function() {
    const inner = () => {
      console.log(this.name)  // 'Arrow' (继承 nested 的 this)
    }
    inner()
  }
}

obj.regularFn()  // 'Arrow'
obj.arrowFn()    // undefined
obj.nested()     // 'Arrow'
```

## 经典输出题

### 题目 1：隐式绑定丢失

```javascript
const obj = {
  name: 'obj',
  fn: function() {
    console.log(this.name)
  }
}

const fn = obj.fn
fn()  // 输出什么？

// 答案：undefined (或全局 name)
// 原因：fn 是独立调用，丢失了 obj 上下文
```

### 题目 2：回调函数中的 this

```javascript
const obj = {
  name: 'obj',
  fn: function() {
    setTimeout(function() {
      console.log(this.name)
    }, 100)
  }
}

obj.fn()  // 输出什么？

// 答案：undefined
// 原因：setTimeout 回调是独立调用，this 指向全局

// 修复方案1：箭头函数
setTimeout(() => {
  console.log(this.name)
}, 100)

// 修复方案2：保存 this
const self = this
setTimeout(function() {
  console.log(self.name)
}, 100)
```

### 题目 3：链式调用

```javascript
const obj = {
  a: 1,
  fn1: function() {
    console.log(this.a)
    return this
  },
  fn2: function() {
    console.log(this.a + 1)
    return this
  }
}

obj.fn1().fn2()  // 输出什么？

// 答案：1, 2
// 原因：每次调用后返回 this (obj)，形成链式调用
```

### 题目 4：构造函数中的 this

```javascript
function Foo() {
  this.name = 'Foo'
  return { name: 'Bar' }  // 返回对象
}

const foo = new Foo()
console.log(foo.name)  // 输出什么？

// 答案：'Bar'
// 原因：构造函数返回对象时，new 的结果是返回的对象
```

```javascript
function Foo() {
  this.name = 'Foo'
  return 'Bar'  // 返回基本类型
}

const foo = new Foo()
console.log(foo.name)  // 输出什么？

// 答案：'Foo'
// 原因：返回非对象时，忽略返回值，使用 new 创建的对象
```

### 题目 5：综合题

```javascript
var name = 'global'

const obj = {
  name: 'obj',
  fn1: function() {
    console.log(this.name)
  },
  fn2: () => {
    console.log(this.name)
  },
  fn3: function() {
    return function() {
      console.log(this.name)
    }
  },
  fn4: function() {
    return () => {
      console.log(this.name)
    }
  }
}

obj.fn1()        // ?
obj.fn2()        // ?
obj.fn3()()      // ?
obj.fn4()()      // ?

// 答案：
// obj.fn1()   -> 'obj'    (隐式绑定)
// obj.fn2()   -> 'global' (箭头函数继承外层，指向全局)
// obj.fn3()() -> 'global' (返回的函数独立调用)
// obj.fn4()() -> 'obj'    (箭头函数继承 fn4 的 this)
```

### 题目 6：call/apply/bind

```javascript
const obj1 = { name: 'obj1' }
const obj2 = { name: 'obj2' }

function fn() {
  console.log(this.name)
}

fn.call(obj1)              // 'obj1'
fn.apply(obj2)             // 'obj2'
fn.bind(obj1).call(obj2)   // 'obj1' (bind 优先级更高)
```

### 题目 7：类中的 this

```javascript
class Counter {
  constructor() {
    this.count = 0
  }

  increment() {
    this.count++
    console.log(this.count)
  }
}

const counter = new Counter()
counter.increment()  // 1

const inc = counter.increment
inc()  // 报错！this 是 undefined

// 修复：绑定 this
const boundInc = counter.increment.bind(counter)
boundInc()  // 2
```

## 总结表格

| 调用方式 | this 指向 | 示例 |
|----------|-----------|------|
| 独立调用 | 全局对象/undefined | `fn()` |
| 对象方法 | 调用对象 | `obj.fn()` |
| call/apply | 指定对象 | `fn.call(obj)` |
| bind | 指定对象 | `fn.bind(obj)()` |
| new | 新创建的对象 | `new Fn()` |
| 箭头函数 | 继承外层 this | `() => {}` |

## 面试要点

1. **this 是动态的**，取决于调用方式
2. **箭头函数是静态的**，取决于定义位置
3. **优先级**：new > bind > call/apply > 隐式 > 默认
4. **严格模式**下默认绑定为 `undefined`

## 深入阅读

- [MDN - this](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)
- [You Don't Know JS - this & Object Prototypes](https://github.com/getify/You-Dont-Know-JS)
