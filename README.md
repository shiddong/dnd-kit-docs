---
description: >-
  @dnd-kit – 一个轻量级、模块化、高性能、易使用且可扩展的 React 拖放工具包。
---

# 概述

- **功能丰富：** 支持定制化的碰撞检测算法、多种激活器 (activators)、可拖拽浮层 (draggable overlay)、拖拽手柄 (drag handles)、自动滚动、约束等等。
- **服务于 React：** 对外暴露了像 [`useDraggable`](api-documentation/draggable/usedraggable.md) 与 [`useDroppable`](api-documentation/droppable/usedroppable.md) 的 hooks，不需要你重构原来的应用或者添加额外的 DOM 包装节点。
- **支持广泛的使用场景：** 比如列表、网格、多容器、多层级嵌套、可变大小的元素、虚拟列表、2D 游戏等场景。
- **零依赖与模块化：** 核心库压缩后的大小约为 10kb，并且没有外部依赖。它仅仅使用了 React 内置的状态管理和 Context，非常的精简。
- **内置了对多种输入方式的支持能力：** 包括指针、鼠标、触摸和键盘传感器 (sensors)。
- **能够完全地自定义和可扩展：** 允许自定义任何细节：动画、过渡形式、行为和风格。你也可以创建自己的传感器 (sensors)、碰撞检测算法和自定义按键绑定等等。
- **无障碍：** 支持使用键盘进行操作，提供了合理的 aria 默认属性，内置了自定义的屏幕阅读器指示和动态更新区域。
- **高性能：** 从设计层面就考虑到了性能问题，能够提供丝滑的动画。
- **预置能力：** 如果你需要创建一个可拖拽排序的界面？可以看看 [`@dnd-kit/sortable`](presets/sortable/)，它是建立在 `@dnd-kit/core` 之上的一个薄层。未来还将会提供更多的预置能力。

![](.gitbook/assets/concepts-illustration-large.svg)

**dnd kit** 的核心库暴露了两个主要的概念:

- [可拖拽的 (draggable) 元素](api-documentation/draggable/)
- [可放置的 (droppable) 区域](api-documentation/droppable/)

使用 [`useDraggable`](api-documentation/draggable/usedraggable.md) 与 [`useDroppable`](api-documentation/droppable/usedroppable.md) 这两个 hooks 可以增强现有的组件，或者将它们结合起来创建既可拖拽也可放置的组件。

使用 [`<DndContext>`](api-documentation/context-provider/) provider 可以处理事件和自定义行为 —— 包括可拖拽元素与可放置区域。配置 [传感器](api-documentation/sensors/) 可以处理不同的输入方式（如指针、鼠标、触摸、键盘等）。

使用 [`<DragOverlay>`](api-documentation/draggable/drag-overlay.md) 组件可以为可拖拽元素渲染一个脱离正常文档流并相对于视口定位的浮层。

请点击下面的链接，了解如何开始使用：

{% content-ref url="introduction/getting-started.md" %}
[getting-started.md](introduction/getting-started.md)
{% endcontent-ref %}

### 可扩展性

可扩展性是 **dnd kit** 的核心。**dnd kit** 的目标是精简和可扩展。它提供了我们认为大多数人在大多数时候所需要的功能，并提供了扩展点，以便在 `@dnd-kit/core` 的基础上开发其他功能。

显示 **dnd kit** 可扩展性能力水平的一个典型例子是 [预置能力 Sortable](presets/sortable/)，它就是基于 `@dnd-kit/core` 所暴露的扩展点而开发出来的。

主要的扩展点包括:

- [传感器 (Sensors)](api-documentation/sensors/)
- [修改器 (Modifiers)](api-documentation/modifiers.md)
- [自定义碰撞检测算法](api-documentation/context-provider/collision-detection-algorithms.md#custom-collision-detection-strategies)

### 无障碍

构建一个所有人都能够无障碍使用的拖放界面是非常困难的，这需要我们经过深思熟虑熟虑的设计。

`@dnd-kit/core` 库中提供了一系列的切入点，可以帮助你创建更易于使用的拖放界面：

- 开箱即用的[支持键盘输入](api-documentation/sensors/keyboard.md)；
- [自定义的屏幕阅读器指示](guides/accessibility.md#screen-reader-instructions)，帮助了解如何与可拖拽元素进行交互；
- [自定义的动态区域更新](guides/accessibility.md#screen-reader-announcements-using-live-regions) ，将当前可拖拽元素与可放置区域的状态实时同步到屏幕阅读器；
- [合理的默认属性](api-documentation/draggable/usedraggable.md#attributes)，需要传入这些属性给可拖拽元素；

查看无障碍指南，了解更多关于如何创建一个更易于使用的拖放界面：

{% content-ref url="guides/accessibility.md" %}
[accessibility.md](guides/accessibility.md)
{% endcontent-ref %}

### 架构

与大多数的拖放库不同，**dnd kit** 是有意地没有基于 [HTML5 拖放 API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API) 所开发的。这是一个经过考量的架构决策，你在决定使用它之前，需要意识到其中的权衡。对于大多数的 Web 应用而言，我们认为这样做是利大于弊的。

HTML5 拖放 API 有一些严重的**不足之处**。它不支持触屏设备，这意味着基于它所开发的库需要提供一个完全不同的实现来支持这些触屏设备。通常，这样会增加代码库的复杂度和包的整体大小。此外，它还需要一些变通方案来实现一些常见的使用场景，比如自定义的拖拽预览 （drag preview），将拖拽操作限定在特定的坐标轴或容器的边界，以及拖拽元素被拖拽时的动画处理。

没有使用 HTML5 拖放 API 的主要**权衡**是假设你不需要在桌面或窗口之间进行拖拽。如果你想要实现包含这种场景的功能，应该使用基于 HTML5 拖放 API 所开发的库。强烈推荐 [react-dnd](https://github.com/react-dnd/react-dnd/)，它是一个底层基于 HTML5 拖放 API 的 React 拖放工具库。

### 性能

#### **最小化 DOM 突变**

**dnd kit** 能够让你在构建拖放界面时，不会在拖拽元素每次更新位置时都直接修改 DOM。

这是因为在触发一个拖拽操作之后，**dnd kit** 会惰性地计算和存储可拖放容器的初始位置与矩形边框信息。这些位置信息会传递给使用了 `useDraggable` 和 `useDroppable` 的组件，这样就可以在拖拽时计算出各个元素的新位置，并使用不触发重绘的高性能 CSS 属性（如 translate3d 与 scale）将它们移动到新位置。关于如何实现的示例，请查看 [`@dnd-kit/sortable`](presets/sortable/) 库中排序策略的实现。

这并不是说你不能在拖拽时改变可拖拽元素在 DOM 中的位置，这其实是**支持的**，有时甚至是不可避免的。在某些情况下，除非将可拖拽元素移动到具体的位置，否则我们是无法提前预测它在 DOM 中的新位置和布局的。要知道，在拖拽时对 DOM 的这种突变的代价是非常昂贵的，并且会导致重绘，所以尽可能使用 `translate3d` 与 `scale` 来计算新的位置。

#### 合成事件

在传感器中，所有传感器的激活器事件都采用了[合成事件监听器](https://reactjs.org/docs/events.html) ，与手动地为每个可拖放元素添加事件监听器而言，性能上得到了提高。
