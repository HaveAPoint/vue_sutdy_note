# Flex 与 Grid 布局

## Flex 布局 (弹性盒子)

**一维布局**：主要处理一行或一列的布局。

### 1. 容器属性 (Container)

```css
.container {
  display: flex;
  
  /* 主轴方向：水平 row (默认) | 垂直 column */
  flex-direction: row | column;
  
  /* 换行：不换行 nowrap (默认) | 换行 wrap */
  flex-wrap: nowrap | wrap;
  
  /* 主轴对齐：起点 | 终点 | 居中 | 两端对齐 | 均匀分布 */
  justify-content: flex-start | flex-end | center | space-between | space-around;
  
  /* 交叉轴对齐：拉伸 | 起点 | 终点 | 居中 | 基线 */
  align-items: stretch | flex-start | flex-end | center | baseline;
}
```

### 2. 项目属性 (Item)

```css
.item {
  /* 放大比例：默认为 0 (空间有剩余也不放大) */
  flex-grow: 0;
  
  /* 缩小比例：默认为 1 (空间不足时会自动缩小) */
  flex-shrink: 1;
  
  /* 基础大小：默认 auto，设置后优先级高于 width */
  flex-basis: auto;
  
  /* 简写：flex: grow shrink basis */
  /* 常用：flex: 1 (1 1 0%) -> 自动撑满剩余空间 */
  flex: 1; 
}
```

### 3. 经典场景：垂直水平居中

```css
.center-box {
  display: flex;
  justify-content: center; /* 主轴居中 */
  align-items: center;     /* 交叉轴居中 */
}
```

---

## Grid 布局 (网格布局)

**二维布局**：同时处理行和列，像表格一样强大。

### 1. 容器属性

```css
.grid-container {
  display: grid;
  
  /* 划分行和列 */
  /* 三列：100px 200px 剩余自动 */
  grid-template-columns: 100px 200px auto;
  
  /* 两行：各 50% */
  grid-template-rows: 50% 50%;
  
  /* repeat 语法：重复 3 次 1fr */
  grid-template-columns: repeat(3, 1fr);
  
  /* 间距 */
  gap: 10px;
}
```

### 2. 单位 `fr`
`fr` (fraction) 是 Grid 专属的单位，代表剩余空间的比例。
`1fr 2fr` 表示把剩余空间分成 3 份，前者占 1 份，后者占 2 份。

---

## Flex vs Grid 区别

| 特性 | Flex | Grid |
|------|------|------|
| **维度** | **一维** (线) | **二维** (面) |
| **核心思想** | 内容驱动 (Content-first) | 布局驱动 (Layout-first) |
| **重叠** | 难 (通常不重叠) | 易 (可指定区域重叠) |
| **适用场景** | 导航栏、列表、卡片对齐 | 复杂的网页整体布局、图片墙 |
| **对齐** | 比较简单 |极其强大 |

### 怎么选？

- **UI 是条状的**（一排按钮、一列文章列表） -> 用 **Flex**。
- **UI 是块状网格的**（后台仪表盘、复杂的报表、不对称的图片墙） -> 用 **Grid**。
- **大部分时候**：Flex 够用了，Grid 作为补充。

---

## 面试题：flex: 1 代表什么？

`flex: 1` 是 `flex: 1 1 0%` 的简写。

- `flex-grow: 1`：有剩余空间就放大。
- `flex-shrink: 1`：空间不足就缩小。
- `flex-basis: 0%`：**忽略内容本身的大小**，直接平分空间。

**对比 `flex: auto`** (`1 1 auto`)：
`flex-basis: auto` 会考虑内容原本的大小，会导致内容多的元素占更宽，而不是绝对平均。

**想要绝对平均分？用 `flex: 1`。**
