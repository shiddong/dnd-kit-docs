# DndContext

## 应用结构

### Context provider

为了使 [Droppable](../droppable/) 与 [Draggable](../draggable/) 组件能够彼此交互，需要让它们在同一个父级 `<DndContext>` 组件下。`<DndContext>` provider 使用 [React Context API](https://reactjs.org/docs/context.html) 在 **draggable**、**droppable** 组件与 hooks 之间共享数据。

> React context 提供了一种在深层组件树中传递数据的方式，而不需要通过 props 逐级向下透传。

因此，使用 [`useDraggable`](../draggable/usedraggable.md)、 [`useDroppable`](../droppable/usedroppable.md) 或 [`DragOverlay`](../draggable/drag-overlay.md) 的组件需要嵌套在一个 `DndContext` provider 中。

它们可以不是 `<DndContext>` provider 的直接后代，只需要这个父级的 `<DndContext>` provider 处于组件树中的任一更高层级。

```jsx
import React from "react";
import { DndContext } from "@dnd-kit/core";

function App() {
  return (
    <DndContext>
      {/* Components that use `useDraggable`, `useDroppable` */}
    </DndContext>
  );
}
```

### 嵌套使用

你还可以将 `<DndContext>` provider 嵌套在其他的 `<DndContext>` provider 中，实现相互独立的嵌套式 **draggable/droppable** 界面。

```jsx
import React from "react";
import { DndContext } from "@dnd-kit/core";

function App() {
  return (
    <DndContext>
      {/* Components that use `useDraggable`, `useDroppable` */}
      <DndContext>
        {/* ... */}
        <DndContext>{/* ... */}</DndContext>
      </DndContext>
    </DndContext>
  );
}
```

需要注意的是，在嵌套的 DndContext providers 中，`useDroppable` 和 `useDraggable` 只能访问当前上下文中的其他 **draggable** 和 **droppable** 节点。

如果多个 `DndContext` provider 在监听同一事件，那么事件将会由包含了被该事件所激活的 [Sensor](../sensors/) 的第一个 `DndContext` 捕获，类似于 [DOM 中的事件冒泡](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events#Event_bubbling_and_capture)。

## Props

```typescript
interface Props {
  announcements?: Announcements;
  autoScroll?: boolean;
  cancelDrop?: CancelDrop;
  children?: React.ReactNode;
  collisionDetection?: CollisionDetection;
  layoutMeasuring?: Partial<LayoutMeasuring>;
  modifiers?: Modifiers;
  screenReaderInstructions?: ScreenReaderInstructions;
  sensors?: SensorDescriptor<any>[];
  onDragStart?(event: DragStartEvent): void;
  onDragMove?(event: DragMoveEvent): void;
  onDragOver?(event: DragOverEvent): void;
  onDragEnd?(event: DragEndEvent): void;
  onDragCancel?(): void;
}
```

### 事件处理函数

从上面的 props 列表中可以看出，有各种由 `<DndContext>` 触发的事件，你可以监听这些事件并决定如何处理它们。

可以监听的主要事件包括：

#### `onDragStart`

当发生一个满足 [sensor](../sensors/) [激活条件](../sensors/#concepts) 的拖动事件时会触发该事件，同时被触发的还有被选中的 **draggale** 元素的唯一标识。

#### `onDragMove`

当 [draggable](../draggable/) 元素在移动时会触发该事件。取决于被激活的 [sensor](../sensors/#activators)，例如，移动 [鼠标指针](../sensors/pointer.md)或按下 [键盘](../sensors/keyboard.md) 的移动键。

#### `onDragOver`

当一个 [draggable](../draggable/) 元素移动到一个 [droppable](../droppable/) 容器之上时会触发该事件。同时被触发的还有 **droppable** 容器的唯一标识符。

#### `onDragEnd`

当一个 **draggable** 元素被放置后会触发该事件。

该事件包含了激活的 **draggable** 元素的 `id`，以及它放置时的 `over` 元素信息。

当 **draggable** 元素在放置时没有 [检测到碰撞](collision-detection-algorithms.md)，`over` 属性则为 `null`。如果检测到了碰撞，`over` 属性则包含放置时的那个 **droppable** 元素 `id`。

{% hint style="info" %}

需要理解的是 `onDragEnd` 事件**并不意味着已经将** [**draggable**](../draggable/) **元素放置于** [**droppable**](../droppable/) **容器中**。

相反，它只是提供了关于被放置的 draggable 元素以及将其放置后可能存在的 droppable 容器的信息。

如何处理这些信息以及如何响应，是由 `DndContext` 的 **consumer** 决定的（对应于 provider），例如，通过更新（或不更新）其内部状态来响应该事件，从而声明式地使这些元素在不同的父级 droppable 容器中进行渲染。

{% endhint %}

#### `onDragCancel`

当拖动操作被取消时会触发该事件，例如，用户在拖动 draggable 元素时按下了 `escape` 键。

### 无障碍

有关 draggable 与 droppable 组件中无障碍性的更多细节和最佳实践，请阅读无障碍部分：

{% page-ref page="../../guides/accessibility.md" %}

#### Announcements

Use the `announcements` prop to customize the screen reader announcements that are announced in the live region when draggable items are picked up, moved over droppable regions, and dropped.

The default announcements are:

```javascript
const defaultAnnouncements = {
  onDragStart(id) {
    return `Picked up draggable item ${id}.`;
  },
  onDragOver(id, overId) {
    if (overId) {
      return `Draggable item ${id} was moved over droppable area ${overId}.`;
    }

    return `Draggable item ${id} is no longer over a droppable area.`;
  },
  onDragEnd(id, overId) {
    if (overId) {
      return `Draggable item was dropped over droppable area ${overId}`;
    }

    return `Draggable item ${id} was dropped.`;
  },
  onDragCancel(id) {
    return `Dragging was cancelled. Draggable item ${id} was dropped.`;
  },
};
```

尽管这些默认的公告是相对合理的默认值，应该涵盖了大多数简单的使用场景，但最了解你的应用程序的还是你自己，强烈建议你自定义这些公告，以便为正在构建的应用程序提供更合适的屏幕阅读体验。

#### 屏幕阅读器指令

使用 `screenReaderInstructions` 来自定义当焦点移动时读给屏幕阅读器的指令。

### 自动滚动

针对 `DndContext` 下的所有 sensors, 使用可选的布尔值类型 `autoScroll` 参数可以临时或永久地禁用自动滚动。

使用 sensor 的静态属性 `autoScrollEnabled` 也可以在单个 sensor 上禁用自动滚动。例如，[键盘 sensor](../sensors/keyboard.md) 在内部管理滚动，因此将其境台属性
`autoScrollEnabled` 设置为 `false`。

### 碰撞检测

使用 `collisionDetection` 参数可以自定义碰撞检测算法，对 `DndContext` provider 下的 draggable 节点与 droppable 区域进行碰撞检测。

The default collision detection algorithm is the [rectangle intersection](collision-detection-algorithms.md#rectangle-intersection) algorithm.

默认的碰撞检测算法是 [矩阵相交 rectangle intersection](collision-detection-algorithms.md#rectangle-intersection) 算法。

以下是内置的碰撞检测算法：

- [矩阵相交 Rectangle intersection](collision-detection-algorithms.md#rectangle-intersection)
- [最近中心点 Closest center](collision-detection-algorithms.md#closest-center)
- [最近邻角 Closest corners](collision-detection-algorithms.md#closest-corners)

你可以自定义一个碰撞检测算法，或对现有算法进行组合。

想要了解更多，可以阅读碰撞检测指南：

{% page-ref page="collision-detection-algorithms.md" %}

### Sensors

Sensors 是一个抽象的概念，用于检测不同的输入方式，以便触发拖动操作、响应移动、结束或取消操作。

`DndContext` 默认的 sensors 是 [指针](../sensors/pointer.md) 与 [键盘](../sensors/keyboard.md)

想要了解如何自定义 sensors 或者如何向 `DndContext` 传递不同的 sensors，清阅读 Sensors 指南：

{% page-ref page="../sensors/" %}

### Modifiers

Modifiers 可以让你动态地修改 sensors 检测到的移动坐标。它们可以应用于很多的使用场景，例如：

- 将移动限制在单一坐标轴上
- 将移动限制在 draggable 节点容器的边界矩形内
- 将移动限制在 draggable 节点的滚动容器边界矩形内
- 施加阻力或夹紧运动

想要了解更多关于如何使用 Modifiers，请阅读 Modifiers 指南：

{% page-ref page="../modifiers.md" %}

### 测量布局

使用 `layoutMeasuring` 参数可以配置 `DndContext` 应当在何时、多久测量一次 droppable 元素。

`frequency` 参数可以控制测量布局的频率。默认情况下，布局测量会被设置为 `optimized`，表示只基于 `strategy` 测量布局。

可以指定以下策略之一：

- `LayoutMeasuringStrategy.WhileDragging`: 默认行为，仅在拖动开始后测量 droppable 元素。

- `LayoutMeasuringStrategy.BeforeDragging`: 在拖动开始前和结束后测量 droppable 元素。

- `LayoutMeasuringStrategy.Always`: 在拖动开始前、开始后和结束后测量 droppable 元素。

使用案例：

```jsx
import { DndContext, LayoutMeasuringStrategy } from "@dnd-kit/core";

<DndContext layoutMeasuring={{ strategy: LayoutMeasuringStrategy.Always }} />;
```
