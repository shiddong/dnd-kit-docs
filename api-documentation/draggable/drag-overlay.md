# 拖动浮层

`<DragOverlay>` 组件能够为 draggable 元素渲染一个拖动浮层，并且该浮层是脱离普通文档流、相对于当前网页的可视区域进行定位的。

![](../../.gitbook/assets/dragoverlay.png)

## 什么时候应该使用一个拖动浮层呢？

在某些的使用场景下，相较于直接转变 [`useDraggable`](usedraggable.md) 对应的 draggable 源节点的位置，你可能更应该使用一个拖动浮层，比如：

- 需要**预览**当前的 draggable 源节点即将放置的位置，可以在拖动时更新 draggable 源节点的位置，并且不会影响拖动浮层。
- 需要将元素从一个容器拖到另一个容器中，此时强烈推荐使用 `<DragOverlay>` 组件，draggable 元素在拖动时可以从源容器中卸载并在另一个容器中执行挂载，并且也不会影响拖动浮层。
- draggable 元素位于一个**可滚动的容器中，**也建议使用 `<DragOverlay>` 组件，否则你需要自行将 draggable 元素设置为 `position: fixed`，才能使得该元素不会在滚动容器中溢出，并且滚动时在容器中的滚动位置也能够正常。
- draggable 元素位于一个**虚拟列表**中，此时绝对应该需要一个拖动浮层，因为随着虚拟列表容器的滚动，在拖动时 draggable 源节点会被卸载。
- 无需自行处理就能在**放置时有一个丝滑的动画**。

## 使用方式

在 `<DragOverlay>` 组件的 children 支持渲染任何合法的 JSX。

`<DragOverlay>` 组件需要**保持始终处于挂载状态**，才会执行放置动画。如果条件式地渲染 `<DragOverlay>` 组件，它的放置动画是无法正常工作的。

经验表明，`<DragOverlay>` 组件应当尽量渲染在 draggable 组件之外，并遵循[展示型组件模式](drag-overlay.md#presentational-components)以保持良好的关注点分离。

反而是应该需要条件式地渲染传递给 `<DragOverlay>` 组件的 children。

{% tabs %}
{% tab title="App.jsx" %}

```jsx
import React, { useState } from "react";
import { DndContext, DragOverlay } from "@dnd-kit/core";

import { Draggable } from "./Draggable";

/* <Item> 和 <ScrollableList> 组件的实现细节与以下示例无关，因此略过 */

function App() {
  const [items] = useState(["1", "2", "3", "4", "5"]);
  const [activeId, setActiveId] = useState(null);

  return (
    <DndContext onDragStart={handleDragStart} onDragEnd={handleDragEnd}>
      <ScrollableList>
        {items.map((id) => (
          <Draggable key={id} id={id}>
            <Item value={`Item ${id}`} />
          </Draggable>
        ))}
      </ScrollableList>

      <DragOverlay>
        {activeId ? <Item value={`Item ${activeId}`} /> : null}
      </DragOverlay>
    </DndContext>
  );

  function handleDragStart(event) {
    setActiveId(event.active.id);
  }

  function handleDragEnd() {
    setActiveId(null);
  }
}
```

{% endtab %}

{% tab title="Draggable.jsx" %}

```jsx
import React from "react";
import { useDraggable } from "@dnd-kit/core";

function Draggable(props) {
  const { attributes, listeners, setNodeRef } = useDraggable({
    id: props.id,
  });

  return (
    <li ref={setNodeRef} {...listeners} {...attributes}>
      {props.children}
    </li>
  );
}
```

{% endtab %}
{% endtabs %}

## 模式

### 展示型组件

这是一种可选的模式，但仍然建议将 draggable 元素设计为[展示型组件](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)，从而与 `@dnd-kit` 解耦。

使用该模式，同一组件就具有两种使用版本，一种是拖动浮层中渲染的展示型组件，另一种是内部渲染了展示型组件的 draggable 元素。

#### 包装节点

从上面的例子中可以看出，可以将一个包装节点与任意子节点一起渲染为 draggable 元素，从而创建一个小型的抽象组件：

{% tabs %}
{% tab title="Draggable.jsx" %}

```jsx
import React from "react";
import { useDraggable } from "@dnd-kit/core";

function Draggable(props) {
  const Element = props.element || "div";
  const { attributes, listeners, setNodeRef } = useDraggable({
    id: props.id,
  });

  return (
    <Element ref={setNodeRef} {...listeners} {...attributes}>
      {props.children}
    </Element>
  );
}
```

{% endtab %}
{% endtabs %}

使用该模式，可以在 `<Draggable>` 与 `<DragOverlay>` 组件中渲染展示型组件。

{% tabs %}
{% tab title="App.jsx" %}

```jsx
import React, { useState } from "react";
import { DndContext, DragOverlay } from "@dnd-kit/core";

import { Draggable } from "./Draggable";

/* <Item> 组件的实现细节与以下示例无关，因此略过 */

function App() {
  const [isDragging, setIsDragging] = useState(false);

  return (
    <DndContext onDragStart={handleDragStart} onDragEnd={handleDragEnd}>
      <Draggable id="my-draggable-element">
        <Item />
      </Draggable>

      <DragOverlay>{isDragging ? <Item /> : null}</DragOverlay>
    </DndContext>
  );

  function handleDragStart() {
    setIsDragging(true);
  }

  function handleDragEnd() {
    setIsDragging(false);
  }
}
```

{% endtab %}
{% endtabs %}

#### Ref 转发

使用 [ref 转发模式](https://reactjs.org/docs/forwarding-refs.html)可以将展示型组件与 `useDraggable` 进行关联：

```jsx
import React, { forwardRef } from "react";

const Item = forwardRef(({ children, ...props }, ref) => {
  return (
    <li {...props} ref={ref}>
      {children}
    </li>
  );
});
```

这样，同一组件就具有了两种使用版本，一种是纯展示型态，另一种是渲染了展示型态组件的 draggable 元素，并且**不需要添加额外的包装节点**。

```jsx
import React from 'react';
import {useDraggable} from '@dnd-kit/core';

function DraggableItem(props) {
  const {attributes, listeners, setNodeRef} = useDraggable({
    id: props.id,
  });

  return (
    <Item ref={setNodeRef} {...attributes} {...listeners}>
      {value}
    </Item>
  )
});
```

### Portals

默认情况下，拖动浮层不是渲染在 portal 中，而是渲染在当前 draggable 元素的渲染容器中。

如果要将 `<DragOverlay>` 渲染在其他容器中，可以从 `react-dom` 中引入 [`createPortal`](https://reactjs.org/docs/portals.html) 方法进行实现。

```jsx
import React, { useState } from "react";
import { createPortal } from "react-dom";
import { DndContext, DragOverlay } from "@dnd-kit/core";

function App() {
  return (
    <DndContext>
      {createPortal(<DragOverlay>{/* ... */}</DragOverlay>, document.body)}
    </DndContext>
  );
}
```

## Props

```typescript
{
  adjustScale?: boolean;
  children?: React.ReactNode;
  className?: string;
  dropAnimation?: DropAnimation | null;
  style?: React.CSSProperties;
  transition?: string | TransitionGetter;
  modifiers?: Modifiers;
  wrapperElement?: keyof JSX.IntrinsicElements;
  zIndex?: number;
}
```

### Children

在 `<DragOverlay>` 组件的 children 支持渲染任何合法的 JSX。但是需要**注意在其中不能再使用 `useDraggable`**。

只能条件式地渲染 `<DragOverlay>` 组件的 children，而不能条件式地渲染 `<DragOverlay>` 组件本身，否则它的放置动画是无法正常工作的。

### className 与内联样式 style

若要对 `DragOverlay` 组件 children 的[包装节点](drag-overlay.md#bao-zhuang-jie-dian)的渲染样式进行自定义，可以使用 `className` 与 `style` 这两个参数：

```jsx
<DragOverlay
  className="my-drag-overlay"
  style={{
    width: 500,
  }}
>
  {/* ... */}
</DragOverlay>
```

### 放置动画

使用 `dropAnimation` 参数可以配置放置时的动画效果。

```typescript
interface DropAnimation {
  duration: number;
  easing: string;
}
```

`duration` 选项是一个 number 类型，单位为 `milliseconds`， 其默认值为 `250` 毫秒（milliseconds）。`easing` 选项是一个 string 类型，表示一个合法的 [CSS easing 函数](https://developer.mozilla.org/zh-CN/docs/Web/CSS/easing-function)，其默认值为 `ease`.

```jsx
<DragOverlay
  dropAnimation={{
    duration: 500,
    easing: "cubic-bezier(0.18, 0.67, 0.6, 1.22)",
  }}
>
  {/* ... */}
</DragOverlay>
```

将 `dropAnimation` 设置为 `null`，可以禁用放置动画。

```jsx
<DragOverlay dropAnimation={null}>{/* ... */}</DragOverlay>
```

{% hint style="warning" %}
`<DragOverlay>` 组件需要**保持始终处于挂载状态**，才会执行放置动画。如果条件式地渲染 `<DragOverlay>` 组件，它的放置动画是无法正常工作的。
{% endhint %}

### Modifiers

利用 Modifiers 可以动态地修改 sensors 检测到的移动坐标，它们的使用场景非常广泛，阅读 [Modifiers](../modifiers.md) 可以了解更多。

比如，利用 modifiers 可以将 `<DragOverlay>` 的移动限制在窗口边界内：

```jsx
import { DndContext, DragOverlay } from "@dnd-kit";
import { restrictToWindowEdges } from "@dnd-kit/modifiers";

function App() {
  return (
    <DndContext>
      {/* ... */}
      <DragOverlay modifiers={[restrictToWindowEdges]}>{/* ... */}</DragOverlay>
    </DndContext>
  );
}
```

### 过渡效果

默认情况下，`<DragOverlay>` 没有任何过渡效果，除非 [`Keyboard` sensor](../sensors/keyboard.md) 被激活。使用 `transition` 参数可以创建一个函数，该函数根据[activator 事件](../sensors/#activators)返回过渡效果。默认的实现方式是：

```javascript
function defaultTransition(activatorEvent) {
  const isKeyboardActivator = activatorEvent instanceof KeyboardEvent;

  return isKeyboardActivator ? "transform 250ms ease" : undefined;
}
```

### 包装节点

默认情况下，`<DragOverlay>` 组件会在子元素上包装一层 `div` 节点。若 draggable 节点是列表元素，因为在非 `ul` 节点下使用 `li` 是不合法的 HTML，可能需要使用 `ul` 替代默认的 `div` 节点：

```jsx
<DragOverlay wrapperElement="ul">{/* ... */}</DragOverlay>
```

### `z-index`

`zIndex` 参数用于设置拖动浮层的 [z-order](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index)。
考虑到兼容性，其默认值设置为 `999`，不过建议可以设置一个较低的值。
