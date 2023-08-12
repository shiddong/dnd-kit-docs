# 安装

开始使用 **@dnd-kit** 前，请先通过 `npm` 或 `yarn` 安装核心库：

```
npm install @dnd-kit/core
```

你还需要确保已经安装了 peer dependencies，可能你已经在项目中安装了 `react` 和 `react-dom`，如果还没有则需要进行安装：

```bash
npm install react react-dom
```

## 安装包

{% hint style="info" %}
**@dnd-kit** 是一个 [monorepo](https://en.wikipedia.org/wiki/Monorepo)。根据你的需要，你可能还需要安装 `@dnd-kit` 命名空间下的其它子包。

{% endhint %}

### 核心库

为了保持核心库足够小，`@dnd-kit/core` 中仅包含了大部分用户在大多数时候使用拖拽功能的主要能力：

- [Context provider](../api-documentation/context-provider/)
- 支持以下功能的 Hooks:
  - [可拖动元素](../api-documentation/draggable/)
  - [可拖放区域](../api-documentation/droppable/)
- [Drag Overlay](../api-documentation/draggable/drag-overlay.md)
- Sensors:
  - [Pointer](../api-documentation/sensors/pointer.md)
  - [Mouse](../api-documentation/sensors/mouse.md)
  - [Touch](../api-documentation/sensors/touch.md)
  - [Keyboard](../api-documentation/sensors/keyboard.md)
- [无障碍功能](../guides/accessibility.md)

### Modifiers

Modifiers 可以让你动态地修改 sensors 检测到的移动坐标。它们可以应用于很多的使用场景，例如：

- 将移动限制在单一坐标轴上
- 将移动限制在 draggable 节点容器的边界矩形内
- 将移动限制在 draggable 节点的滚动容器边界矩形内
- 施加阻力或夹紧运动

Modifiers 代码库中包含了许多有用的 modifiers，可以应用于 [`DndContext`](../api-documentation/context-provider/) 以及 [`DraggableClone`](../api-documentation/draggable/drag-overlay.md)。

开始使用 modifiers 前，请先通过 `yarn` 或 `npm` 安装依赖包。

```
npm install @dnd-kit/modifiers
```

### 预置

#### [Sortable](../presets/sortable/)

如果你选择从头开始建立一个可排序的界面， `@dnd-kit/core` 提供了所有你需要的构建模块，但幸运地是你并不需要这样做。

如果你打算开发一个可排序的界面，强烈建议你试试 `@dnd-kit/sortable`，它是建立在 `@dnd-kit/core` 之上的一个薄层，而且为了达到丝滑、灵活和易于使用的可排序界面而做了很多优化。

```
npm install @dnd-kit/sortable
```

## 开发版本

合并到 `@dnd-kit` 主分支的每个 commit 都会触发一次开发构建，并在 npm 仓库中发布一个打上 `next` 标签的包。

想要在正式版本发布之前尝鲜一下，请安装每个 `@dnd-kit` 子包的 `@next` 版本。

```
npm install @dnd-kit/core@next @dnd-kit/sortable@next
```

{% hint style="info" %}
开发版本可能不稳定，如果你打算在生产环境中使用，建议你锁定特定的开发版本号。
{% endhint %}
