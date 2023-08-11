---
description: >-
  @dnd-kit – 一个 React 的轻量级、模块化、高性能、可访问且可扩展的拖放工具包。
---

# 概述

- **功能丰富：** 支持定制化的碰撞检测算法、多种激活器、可拖动元素的浮层、拖动手柄、自动滚动、约束等等。
- **服务于 React：** 对外暴露了像 [`useDraggable`](api-documentation/draggable/usedraggable.md) 与 [`useDroppable`](api-documentation/droppable/usedroppable.md) 的 hooks，不需要你去重构原来的应用或者额外的包装 DOM 节点。
- **支持广泛的使用场景：** 比如列表、网格、多容器、多层级嵌套、可变大小的元素、虚拟列表、2D 游戏等场景。
- **零依赖与模块化：** 核心库压缩后的大小约为 10kb，而且没有外部依赖。它仅仅使用了 React 内置的状态管理和 context，保持的非常精简。
- **内置了对多种输入方式的支持能力：** 包括指针、鼠标、触摸和键盘传感器。
- **完全可定制和可扩展：** 定制了每一个细节：动画、过渡、行为和风格。你可以创建自己的传感器、碰撞检测算法和自定义按键绑定等。
- **无障碍：** 支持使用键盘进行操作，包含了合理的 aria 默认属性，内置了可定制的屏幕阅读器指示和动态更新区域。
- **性能：** 它从设计层面就考虑到了性能，以便支持丝般顺滑的动画。
- **预置：** 你需要创建一个可排序的界面？可以看看 [`@dnd-kit/sortable`](presets/sortable/)，它是建立在 `@dnd-kit/core` 之上的一个薄层。未来将会提供更多的预置。

![](.gitbook/assets/concepts-illustration-large.svg)

**dnd kit** 的核心库暴露了两个主要的概念:

- [可拖动元素](api-documentation/draggable/)
- [可拖放区域](api-documentation/droppable/)

使用 [`useDraggable`](api-documentation/draggable/usedraggable.md) 与 [`useDroppable`](api-documentation/droppable/usedroppable.md) hooks 可以增强现有的组件，或者将这两个 hooks 结合起来创建既可拖动也可拖放的组件。

使用 [`<DndContext>`](api-documentation/context-provider/) provider 可以处理事件和定制行为 ———— 包括可拖动的元素与可拖放的区域。配置 [sensors](api-documentation/sensors/) 可以处理不同的输入方式（如指针、鼠标、触摸、键盘等）。

使用 [`<DragOverlay>`](api-documentation/draggable/drag-overlay.md) 组件可以为可拖动的元素渲染一个相对位置并脱离正常文档流的浮层。

查看快速入门指南，了解如何开始：

{% content-ref url="introduction/getting-started.md" %}
[getting-started.md](introduction/getting-started.md)
{% endcontent-ref %}

### 可扩展性

可扩展性是 **dnd kit** 的核心。**dnd kit** 的目标就是精简和可扩展。它提供了我们认为大多数人在大多数时候所需要的功能，并提供了扩展点，以便在 `@dnd-kit/core` 的基础上开发其他功能。

突出 **dnd kit** 可扩展性能力的一个典型例子是 [ Sortable preset](presets/sortable/)，它就是基于 `@dnd-kit/core` 所暴露的扩展点而开发出来的。

The primary extension points are:
主要扩展点包括:

- [传感器 Sensors](api-documentation/sensors/)
- [修改器 Modifiers](api-documentation/modifiers.md)
- [自定义碰撞检测算法 Custom collision detection algorithms](api-documentation/context-provider/collision-detection-algorithms.md#custom-collision-detection-strategies)

### 无障碍

构建一个每个人都能够无障碍使用的拖放界面是非常难的，这需要经过深思熟虑熟虑的设计。

`@dnd-kit/core` 库中提供了一系列的切入点，可以帮助你创建更容易使用的拖放界面：

- 开箱即用的 [键盘支持](api-documentation/sensors/keyboard.md)；
- [可定制的屏幕阅读器指示](guides/accessibility.md#screen-reader-instructions)，帮助了解如何与可拖动的元素进行交互；
- [可定制的动态区域更新](guides/accessibility.md#screen-reader-announcements-using-live-regions) ，为屏幕阅读器提供实时的通知，说明当前可拖放元素所发生的情况；
- 需要传递给可拖放元素的[合理的默认属性](api-documentation/draggable/usedraggable.md#attributes)

查看无障碍指南，了解更多关于如何创建一个更易于使用的拖放界面：

{% content-ref url="guides/accessibility.md" %}
[accessibility.md](guides/accessibility.md)
{% endcontent-ref %}

### 架构

与大多数的拖放库不同，**dnd kit** 是有意没有基于 [HTML5 拖放 API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API) 所开发的。这是一个经过考量的架构决策，在决定使用它之前，你需要意识到其中的权衡。而对于大多数的 Web 应用而言，我们认为这样做是利大于弊的。

HTML5 拖放 API 有一些严重的不足之处。它不支持触屏设备，这意味着基于它开发的库需要暴露一个完全不同的实现来支持这些触屏设备。通常这样会增加代码库的复杂度和包的整体大小。此外，它还需要一些变通方案来实现常见的使用场景，比如自定义的拖动预览，将拖动元素限定在特定的坐标轴或容器的边界，以及对所选择拖动元素的动画处理。

不使用 HTML5 拖放 API 的主要缺点是无法在桌面或窗口之间进行拖动。如果你想要包含这种场景的功能，你应该使用基于 HTML5 拖放 API 所开发的库。强烈推荐你看看 [react-dnd](https://github.com/react-dnd/react-dnd/)，它是一个底层基于 HTML5 拖放 API 的 React 拖放工具库。

### 性能

#### **最小化 DOM 突变**

**dnd kit** lets you build drag and drop interfaces without having to mutate the DOM every time an item needs to shift position.&#x20;
使用 **dnd kit** 能够让你在构建拖放界面时，不会在每次变换位置时都修改 DOM。

这是因为 **dnd kit** 在拖动操作启动后，会惰性计算与存储可拖放容器的初始位置和布局。这些位置会传递给使用 useDraggable 和 useDroppable 的组件，这样就可以在拖动时计算拖动元素的新位置，并使用不触发重绘的高性能 CSS 属性（如 translate3d 与 scale）将它们移动到新位置。关于如何实现的示例，请查看 [`@dnd-kit/sortable`](presets/sortable/) 库中的排序策略的实现。

This isn't to say that you can't shift the position of the items in the DOM while dragging, this is something that **is supported** and sometimes inevitable. In some cases, it won't be possible to know in advance what the new position and layout of the item until you move it in the DOM. Just know that these kind of mutations to the DOM while dragging are much more expensive and will cause a repaint, so if possible, prefer computing the new positions using `translate3d` and `scale`.

这并不是说你不能在拖动时改变拖动元素在 DOM 中的位置，这其实是**支持的**，有时甚至是不可避免的。在某些情况下，除非将拖动元素移动到具体的位置，否则我们是无法提前预测它在 DOM 中的新位置和布局的。要知道，在拖动时对 DOM 的这种突变的代价是非常昂贵的，并且会导致重绘，所以如果可能的话，最好使用 `translate3d` 与 `scale` 来计算新的位置。

#### 合成事件

在 Sensors 中，所有传感器的激活器事件都采用了 [合成事件的监听器](https://reactjs.org/docs/events.html) ，与手动的为每个可拖放节点添加事件监听器而言，性能得到了提高。
