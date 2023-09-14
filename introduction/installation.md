# 安装指南

开始使用 **@dnd-kit** 前，请先通过 `npm` 或 `yarn` 安装核心库：

```
npm install @dnd-kit/core
```

你还需要确保已经安装了对等依赖 (peer dependencies)，可能你已经在项目中安装了 `react` 和 `react-dom`，如果还没有则需要进行安装：

```bash
npm install react react-dom
```

## 安装包

{% hint style="info" %}
**@dnd-kit** 是一个 [monorepo](https://en.wikipedia.org/wiki/Monorepo)。根据使用需要，你可能还要安装 `@dnd-kit` 命名空间下的其它子包。
{% endhint %}

### 核心库

为了保持核心库足够小，`@dnd-kit/core` 中仅包含了大部分用户在大多数时候使用拖拽功能的主要能力：

- [Context provider](../api-documentation/context-provider/)
- 可用于下列概念的 Hooks:
  - [可拖拽 (Draggable)](../api-documentation/draggable/)
  - [可放置 (Droppable)](../api-documentation/droppable/)
- [拖拽浮层 (Drag Overlay)](../api-documentation/draggable/drag-overlay.md)
- 传感器 (Sensors):
  - [指针 (Pointer)](../api-documentation/sensors/pointer.md)
  - [鼠标 (Mouse)](../api-documentation/sensors/mouse.md)
  - [触摸 (Touch)](../api-documentation/sensors/touch.md)
  - [键盘 (Keyboard)](../api-documentation/sensors/keyboard.md)
- [无障碍功能](../guides/accessibility.md)

### 修改器

利用修改器可以动态地修改传感器检测到的移动坐标。它们的使用场景非常广泛，例如：

- 将移动限制在单一坐标轴上
- 将移动限制在可拖拽节点容器的边框内
- 将移动限制在可拖拽节点的滚动容器边框内
- 施加阻力或夹紧运动 (clamping the motion)

修改器的代码仓库中包含了许多有用的修改器，可以应用于 [`DndContext`](../api-documentation/context-provider/) 以及 [`DraggableClone`](../api-documentation/draggable/drag-overlay.md)。

使用修改器之前，请先通过 `yarn` 或 `npm` 安装依赖包。

```
npm install @dnd-kit/modifiers
```

### 预置能力

#### [Sortable](../presets/sortable/)

如果你打算从头开始建立一个可拖拽排序的界面， `@dnd-kit/core` 提供了所有你需要的构建模块，但幸运地是你并不需要这样做。

如果你打算直接开发一个可拖拽排序的界面，强烈建议你试试 `@dnd-kit/sortable`，它是建立在 `@dnd-kit/core` 之上的一个薄层，为了达到丝滑、灵活和易使用地可拖拽排序的界面，做了很多的优化。

```
npm install @dnd-kit/sortable
```

## 开发版本

合并到 `@dnd-kit` 主分支的每个 commit 都会触发一次开发构建，并在 npm 仓库中发布一个打上 `next` 标签的包。

如果你想要在正式版本发布之前尝鲜一下，请安装每个 `@dnd-kit` 子包的 `@next` 版本。

```
npm install @dnd-kit/core@next @dnd-kit/sortable@next
```

{% hint style="info" %}
开发版本可能不稳定，如果你打算在生产环境中使用，建议你锁定特定的开发版本号。
{% endhint %}
