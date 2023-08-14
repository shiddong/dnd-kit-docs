# 拖动浮层

可以用 `<DragOverlay>` 组件为 draggable 元素渲染一个浮层，并且该浮层是脱离普通文档流、相对于当前网页的可视区域进行定位的。

![](../../.gitbook/assets/dragoverlay.png)

## 什么时候应该使用一个拖动浮层呢？

在某些的使用场景下，相较于直接转变 [`useDraggable`](usedraggable.md) 对应的 draggable 源节点的位置，你更应该使用一个拖动浮层：

- 如果需要**预览**当前的 draggable 源节点即将放置时的区域，可以在拖动时更新 draggable 源节点的位置，而不会影响拖动浮层。
- 如果需要将元素从一个容器拖到另一个容器中，强烈推荐使用 `<DragOverlay>` 组件，draggable 元素在拖动时可以从源容器中卸载并在另一个容器中执行挂载，且不会影响拖动浮层。
- 如果 draggable 元素位于一个**可滚动的容器中，**也建议使用 `<DragOverlay>` 组件，否则你需要自行将 draggable 元素设置为 `position: fixed`，才能使得该元素不会在滚动容器中溢出，并且滚动时在容器中的滚动位置也是正常的。
- 如果 draggable 元素位于一个**虚拟列表**中，此时绝对应该需要一个拖动浮层，因为在拖动时随着虚拟列表容器的滚动，draggable 源节点会被卸载。
- 如果你希望不需要多做什么事情就可以获得一个**丝滑的放置动画**。

## 使用方式

You may render any valid JSX within the children of the `<DragOverlay>`.&#x20;

你可以在 `<DragOverlay>` 组件的子元素中渲染任何合法的 JSX。

`<DragOverlay>` 组件需要**时刻处于挂载状态**，才会执行放置动画。如果条件式地渲染 `<DragOverlay>` 组件，它的放置动画是无法正常工作的。

经验之谈，`<DragOverlay>` 组件应当尽量渲染在 draggable 组件之外，并遵循[展示型组件的模式](drag-overlay.md#presentational-components)以保持良好的关注点分离。

反而是应该需要条件式地渲染传递给 `<DragOverlay>` 组件的子元素。

{% tabs %}
{% tab title="App.jsx" %}

```jsx
import React, { useState } from "react";
import { DndContext, DragOverlay } from "@dnd-kit/core";

import { Draggable } from "./Draggable";

/* The implementation details of <Item> and <ScrollableList> are not
 * relevant for this example and are therefore omitted. */

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

虽然不强求，但我们仍然建议你将 draggable 元素设计为[展示型组件](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)，从而与 `@dnd-kit` 解耦。

使用该模式，同一组件就具有两种使用版本，一种是拖动浮层中渲染的展示型组件，另一种是内部渲染了展示型组件的 draggable 元素。

#### 包装节点

As you may have noticed from the example above, we can create small abstract components that render a wrapper node and make any children rendered within draggable:

从上面的例子中，你应该可以看出，可以通过创建小型的抽象组件，由一个包装节点与任何子节点一起渲染为 draggable 元素：

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

/* The implementation details of <Item> is not
 * relevant for this example and therefore omitted. */

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

This way, you can create two versions of your component, one that is presentational, and one that is draggable and renders the presentational component **without the need for additional wrapper elements**:

这样，同一组件就具有了两种使用版本，一个是纯展示型态，另一个是渲染了展示型态组件的 draggable 元素，并且**不需要添加额外的包装节点**。

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

默认情况下，拖动浮层不是渲染在 portal 中。而是渲染在当前的渲染容器中。

如果你想将 `<DragOverlay>` 渲染在其他容器中，可以从 `react-dom` 中引入 [`createPortal`](https://reactjs.org/docs/portals.html) 工具方法。

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

You may render any valid JSX within the children of the `<DragOverlay>`. However, **make sure that the components rendered within the drag overlay do not use the `useDraggable` hook**.

Prefer conditionally rendering the `children` of `<DragOverlay>` rather than conditionally rendering `<DragOverlay>`, otherwise drop animations will not work.

### Class name and inline styles

If you'd like to customize the[ wrapper element](drag-overlay.md#wrapper-element) that the `DragOverlay`'s children are rendered into, use the `className` and `style` props:

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

### Drop animation

Use the `dropAnimation` prop to configure the drop animation.

```typescript
interface DropAnimation {
  duration: number;
  easing: string;
}
```

The `duration` option should be a number, in `milliseconds`. The default value is `250` milliseconds. The `easing` option should be a string that represents a valid [CSS easing function](https://developer.mozilla.org/en-US/docs/Web/CSS/easing-function). The default easing is `ease`.

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

To disable drop animations, set the `dropAnimation` prop to `null`.

```jsx
<DragOverlay dropAnimation={null}>{/* ... */}</DragOverlay>
```

{% hint style="warning" %}
The `<DragOverlay>` component should **remain mounted at all times** so that it can perform the drop animation. If you conditionally render the `<DragOverlay>` component, drop animations will not work.
{% endhint %}

### Modifiers

Modifiers let you dynamically modify the movement coordinates that are detected by sensors. They can be used for a wide range of use-cases, which you can learn more about by reading the [Modifiers](../modifiers.md) documentation.

For example, you can use modifiers to restrict the movement of the `<DragOverlay>` to the bounds of the window:

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

### Transition

By default, the `<DragOverlay>` component does not have any transitions, unless activated by the [`Keyboard` sensor](../sensors/keyboard.md). Use the `transition` prop to create a function that returns the transition based on the [activator event](../sensors/#activators). The default implementation is:

```javascript
function defaultTransition(activatorEvent) {
  const isKeyboardActivator = activatorEvent instanceof KeyboardEvent;

  return isKeyboardActivator ? "transform 250ms ease" : undefined;
}
```

### Wrapper element

By default, the `<DragOverlay>` component renders your elements within a `div` element. If your draggable elements are list items, you'll want to update the `<DragOverlay>` component to render a `ul` wrapper instead, since wrapping a `li` item without a parent `ul` is invalid HTML:

```jsx
<DragOverlay wrapperElement="ul">{/* ... */}</DragOverlay>
```

### `z-index`

The `zIndex` prop sets the [z-order](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index) of the drag overlay. The default value is `999` for compatibility reasons, but we highly recommend you use a lower value.&#x20;
