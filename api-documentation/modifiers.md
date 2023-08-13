# Modifiers

Modifiers 可以让你动态地修改 sensors 检测到的移动坐标。它们可以应用于很多的使用场景，例如：

- 将移动限制在单一坐标轴上
- 将移动限制在 draggable 节点容器的矩形边框内
- 将移动限制在 draggable 节点的滚动容器矩形边框内
- 施加阻力或夹紧运动

## 安装

开始使用 modifiers 前，请先通过 `yarn` 或 `npm` 安装依赖包。

```
npm install @dnd-kit/modifiers
```

## 使用方式

`@dnd-kit/modifiers` 包中提供了很多有用的 modifiers，可以应用到 [`DndContext`](context-provider/) 和 [`DragOverlay`](draggable/drag-overlay.md) 中。

```jsx
import { DndContext, DragOverlay } from "@dnd-kit";
import {
  restrictToVerticalAxis,
  restrictToWindowEdges,
} from "@dnd-kit/modifiers";

function App() {
  return (
    <DndContext modifiers={[restrictToVerticalAxis]}>
      {/* ... */}
      <DragOverlay modifiers={[restrictToWindowEdges]}>{/* ... */}</DragOverlay>
    </DndContext>
  );
}
```

从上面的示例中可以看出，在 `DndContext` 和 `DragOverlay` 中都可以使用不同的 modifiers。

## 内置的 modifiers

### 将移动限制在单一坐标轴上

#### `restrictToHorizontalAxis`

将移动限制在横向坐标轴上。

#### `restrictToVerticalAxis`

将移动限制在纵向坐标轴上。

### 将移动限制在一个容器的矩形边框内

#### `restrictToWindowEdges`

将移动限制在窗口的边缘处。该 modifier 可以有效地阻止 `DragOverlay` 移出窗口的边界。

#### `restrictToParentElement`

将移动限制在被选中的 draggale 元素的父元素内。

#### `restrictToFirstScrollableAncestor`

将移动限制在被选中的 draggable 元素的第一个可滚动的上层元素内。

### 网格对齐

#### `createSnapModifier`

该函数的作用是创建 modifiers 以实现对齐给定大小的网格。

```javascript
import { createSnapModifier } from "@dnd-kit/modifiers";

const gridSize = 20; // pixels
const snapToGridModifier = createSnapModifier(gridSize);
```

## 创建自定义的 modifiers

参考 `@dnd-kit/modifiers` 内置的 modifiers 实现，可以创建自定义的 modifiers: [https://github.com/clauderic/dnd-kit/tree/master/packages/modifiers/src](https://github.com/clauderic/dnd-kit/tree/master/packages/modifiers/src)

例如，下面的代码就创建了一个 modifier 可以实现网格对齐：

```javascript
const gridSize = 20;

function snapToGrid(args) {
  const { transform } = args;

  return {
    ...transform,
    x: Math.ceil(transform.x / gridSize) * gridSize,
    y: Math.ceil(transform.y / gridSize) * gridSize,
  };
}
```
