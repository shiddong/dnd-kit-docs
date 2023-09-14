---
description: >-
  急于开始？本文可以帮助你快速熟悉 dnd kit 的核心概念。
---

# 快速开始

{% hint style="info" %}
在开始之前，请确保你已经遵循了[安装指南](installation.md) 中的操作步骤。
{% endhint %}

### Context provider

首先，我们将创建应用程序的整体结构。为了使 [`useDraggable`](broken-reference) 和 [`useDroppable`](broken-reference) 能够正常工作，使用它们的组件需要被包裹在 [`<DndContext />`](../api-documentation/context-provider/) 组件中。

{% tabs %}
{% tab title="App.jsx" %}

```jsx
import React from "react";
import { DndContext } from "@dnd-kit/core";

import { Draggable } from "./Draggable";
import { Droppable } from "./Droppable";

function App() {
  return (
    <DndContext>
      <Draggable />
      <Droppable />
    </DndContext>
  );
}
```

{% endtab %}
{% endtabs %}

### Droppable

![](../.gitbook/assets/droppable-large.svg)

接下来，我们开始创建第一个 **Droppable** 组件。为此，需要使用到 `useDroppable` hook.

useDroppable 对应用程序的结构没有要求。但至少需要传递一个 [ref](https://reactjs.org/docs/refs-and-the-dom.html) 给将要变成**可放置的** DOM 元素，同时还需要为所有的**可放置**组件提供一个唯一的 id。

当一个**可拖拽**元素移动到**可放置**元素之上时，`isOver` 属性就会变成 `true`.

{% tabs %}
{% tab title="Droppable.jsx" %}

```jsx
import React from "react";
import { useDroppable } from "@dnd-kit/core";

function Droppable(props) {
  const { isOver, setNodeRef } = useDroppable({
    id: "droppable",
  });
  const style = {
    color: isOver ? "green" : undefined,
  };

  return (
    <div ref={setNodeRef} style={style}>
      {props.children}
    </div>
  );
}
```

{% endtab %}
{% endtabs %}

### Draggable

![](../.gitbook/assets/draggable-large.svg)

下面，我们看看如何实现第一个 **Draggable** 组件，为此，需要使用到 `useDraggable` hook.

同样，`useDraggable` hook 对应用程序的结构也没有要求。它只需要将事件监听器和一个 ref 附加在你希望成为**可拖拽的** DOM 元素上。同时，你也需要为所有的**可拖拽**组件提供一个唯一的 id。

当一个**可拖拽的**元素被拖动后，`transform` 属性会被填充为需要在屏幕上移动该元素的 `translate` 坐标。

`transform` 对象遵循如下形式：`{x: number, y: number, scaleX: number, scaleY: number}`

{% tabs %}
{% tab title="Draggable.jsx" %}

```jsx
import React from "react";
import { useDraggable } from "@dnd-kit/core";

function Draggable(props) {
  const { attributes, listeners, setNodeRef, transform } = useDraggable({
    id: "draggable",
  });
  const style = transform
    ? {
        transform: `translate3d(${transform.x}px, ${transform.y}px, 0)`,
      }
    : undefined;

  return (
    <button ref={setNodeRef} style={style} {...listeners} {...attributes}>
      {props.children}
    </button>
  );
}
```

{% endtab %}
{% endtabs %}

正如上例所示，实际上只需要几行代码就可以将现有的组件转化为一个**可拖拽的**组件。

{% hint style="success" %}
**建议:**

- 出于性能方面的考虑，建议使用 **`transform`** 而不是其他的 CSS 位置属性来移动被拖拽的元素；
- 改变 Draggable 组件的 **`z-index`**，可以确保它出现在其他元素的上层；
- 如果一个元素需要从一个容器移动到另一个容器，建议使用 [`<DragOverlay>`](../api-documentation/draggable/drag-overlay.md) 组件。

  {% endhint %}

将 `transform` 对象转换为字符串可能感觉很繁琐。放心，你可以通过从 `@dnd-kit/utilities` 包中导入 `CSS` 工具来避免手工操作。

```jsx
import { CSS } from "@dnd-kit/utilities";

// Within your component that receives `transform` from `useDraggable`:
const style = {
  transform: CSS.Translate.toString(transform),
};
```

### 组装所有的代码片段

当创建了 **Droppable** 和 **Draggable** 组件，就可以回到使用 [`<DndContext>`](../api-documentation/context-provider/) 组件的地方，添加事件监听器以便响应不同的事件。

在这个示例中，假设你想要将 `<Draggable>` 组件从外部移入到 `<Droppable>` 组件中：

![](../.gitbook/assets/example.png)

为此，需要监听 `<DndContext>` 组件的 `onDragEnd` 事件，来查看**可拖拽**元素是否拖到了**可放置**元素的上层：

{% tabs %}
{% tab title="App.jsx" %}

```jsx
import React, { useState } from "react";
import { DndContext } from "@dnd-kit/core";

import { Droppable } from "./Droppable";
import { Draggable } from "./Draggable";

function App() {
  const [isDropped, setIsDropped] = useState(false);
  const draggableMarkup = <Draggable>Drag me</Draggable>;

  return (
    <DndContext onDragEnd={handleDragEnd}>
      {!isDropped ? draggableMarkup : null}
      <Droppable>{isDropped ? draggableMarkup : "Drop here"}</Droppable>
    </DndContext>
  );

  function handleDragEnd(event) {
    if (event.over && event.over.id === "droppable") {
      setIsDropped(true);
    }
  }
}
```

{% endtab %}

{% tab title="Droppable.jsx" %}

```jsx
import React from "react";
import { useDroppable } from "@dnd-kit/core";

export function Droppable(props) {
  const { isOver, setNodeRef } = useDroppable({
    id: "droppable",
  });
  const style = {
    color: isOver ? "green" : undefined,
  };

  return (
    <div ref={setNodeRef} style={style}>
      {props.children}
    </div>
  );
}
```

{% endtab %}

{% tab title="Draggable.jsx" %}

```jsx
import React from "react";
import { useDraggable } from "@dnd-kit/core";

export function Draggable(props) {
  const { attributes, listeners, setNodeRef, transform } = useDraggable({
    id: "draggable",
  });
  const style = transform
    ? {
        transform: `translate3d(${transform.x}px, ${transform.y}px, 0)`,
      }
    : undefined;

  return (
    <button ref={setNodeRef} style={style} {...listeners} {...attributes}>
      {props.children}
    </button>
  );
}
```

{% endtab %}
{% endtabs %}

就是这样！你已经成功创建了第一个 [**Droppable**](../api-documentation/droppable/) 和 [**Draggable**](../api-documentation/draggable/) 组件了。

### 更复杂一点的示例

上面示例稍微有点简单。在现实世界的例子中，你可能有多个**可放置**容器，而且你可能还想在元素被拖入到**可放置**容器后能够将其拖回。

下面是一个稍微复杂的例子，它包含了多个 **Droppable** 容器：

{% tabs %}
{% tab title="App.jsx" %}

```jsx
import React, { useState } from "react";
import { DndContext } from "@dnd-kit/core";

import { Droppable } from "./Droppable";
import { Draggable } from "./Draggable";

function App() {
  const containers = ["A", "B", "C"];
  const [parent, setParent] = useState(null);
  const draggableMarkup = <Draggable id="draggable">Drag me</Draggable>;

  return (
    <DndContext onDragEnd={handleDragEnd}>
      {parent === null ? draggableMarkup : null}

      {containers.map((id) => (
        // We updated the Droppable component so it would accept an `id`
        // prop and pass it to `useDroppable`
        <Droppable key={id} id={id}>
          {parent === id ? draggableMarkup : "Drop here"}
        </Droppable>
      ))}
    </DndContext>
  );

  function handleDragEnd(event) {
    const { over } = event;

    // If the item is dropped over a container, set it as the parent
    // otherwise reset the parent to `null`
    setParent(over ? over.id : null);
  }
}
```

{% endtab %}

{% tab title="Droppable.jsx" %}

```jsx
import React from "react";
import { useDroppable } from "@dnd-kit/core";

export function Droppable(props) {
  const { isOver, setNodeRef } = useDroppable({
    id: props.id,
  });
  const style = {
    color: isOver ? "green" : undefined,
  };

  return (
    <div ref={setNodeRef} style={style}>
      {props.children}
    </div>
  );
}
```

{% endtab %}

{% tab title="Draggable.jsx" %}

```jsx
import React from "react";
import { useDraggable } from "@dnd-kit/core";

export function Draggable(props) {
  const { attributes, listeners, setNodeRef, transform } = useDraggable({
    id: props.id,
  });
  const style = transform
    ? {
        transform: `translate3d(${transform.x}px, ${transform.y}px, 0)`,
      }
    : undefined;

  return (
    <button ref={setNodeRef} style={style} {...listeners} {...attributes}>
      {props.children}
    </button>
  );
}
```

{% endtab %}
{% endtabs %}

希望这份快速入门指南能让你了解 `@dnd-kit` 的简单和强大。还有很多东西需要学习，希望你通过阅读对应的 API 文档，了解所有传递给 `<DndContext>`、`useDroppable` 和 `useDraggable` 的属性。
