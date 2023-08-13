# Draggable

![](../../.gitbook/assets/draggable-large.svg)

使用 `useDraggable` 可以将 DOM 节点转换成 draggable 元素，从而可以被拖起、移动、放置到 [droppable](../droppable/) 容器中。

## 使用方式

`useDraggable` 对应用程序的代码结构也没有特别的要求。

### 节点的引用

但至少需要将 `useDraggable` 中返回的 `setNodeRef` 函数传递给 DOM 元素，这样它才能访问底层的 DOM 节点，并通过保持对它的跟踪，从而检测它与其他 [droppable](../droppable/) 元素之间是否发生碰撞或相交。

```jsx
import { useDraggable } from "@dnd-kit/core";
import { CSS } from "@dnd-kit/utilities";

function Draggable() {
  const { attributes, listeners, setNodeRef, transform } = useDraggable({
    id: "unique-id",
  });
  const style = {
    transform: CSS.Translate.toString(transform),
  };

  return (
    <button ref={setNodeRef} style={style} {...listeners} {...attributes}>
      /* 任何你想要渲染的内容 */
    </button>
  );
}
```

{% hint style="info" %}
尽量在你的代码中使用最有[语义化](https://developer.mozilla.org/en-US/docs/Glossary/Semantics)的 DOM 元素。\
查看[无障碍指南](../../guides/accessibility.md)，了解如何为屏幕阅读器提供更好的使用体验。
{% endhint %}

### 标识符

参数 `id` 是一个字符串类型的唯一标识符，即在某个 [`DndContext`](../context-provider/) provider 下不应存在其他 **draggable** 元素使用同一个标识符。

### 事件监听器

使用 `useDraggable` 需要将 `listeners` 附加到相应的 DOM 节点，该节点是能够触发拖动的 activator。

虽然我们也可以手动地将这些 listeners 附加到 `setNodeRef` 的同一节点上，但要求开发者分开附加这些 listners，也是有一些明显的好处的。

#### 灵活性

很多拖放库都需要单独暴露出“拖动手柄”的概念，但在 dnd kit 中，通过 `useDraggable` 创建一个拖动手柄非常简单，只需要手动附加 listeners 到一个不同于 draggable 源节点的 DOM 元素上。

```jsx
import { useDraggable } from "@dnd-kit/core";

function Draggable() {
  const { attributes, listeners, setNodeRef } = useDraggable({
    id: "unique-id",
  });

  return (
    <div ref={setNodeRef}>
      /* Some other content that does not activate dragging */
      <button {...listeners} {...attributes}>
        拖动手柄
      </button>
    </div>
  );
}
```

{% hint style="info" %}
当把 listeners 附加到与 draggable 节点不同的节点上，同时还需要将 attributes 附加到同一节点上，以便该节点仍然具有[无障碍性](../../guides/accessibility.md)。
{% endhint %}

如果有需要的话，甚至可以设置多个拖动手柄：

```jsx
import { useDraggable } from "@dnd-kit/core";

function Draggable() {
  const { attributes, listeners, setNodeRef } = useDraggable({
    id: "unique-id",
  });

  return (
    <div ref={setNodeRef}>
      <button {...listeners} {...attributes}>
        拖动手柄 1
      </button>
      /* Some other content that does not activate dragging */
      <button {...listeners} {...attributes}>
        拖动手柄 2
      </button>
    </div>
  );
}
```

#### 性能

采用这样的策略也意味着我们能够使用 [React 的合成事件](https://reactjs.org/docs/events.html)，这比手动为每个节点附加事件监听器的性能要高。

为什么呢？相较于为每个 draggable 节点附加单独的事件监听器，React 在整个 `document` 上为每种类型的事件只附加了一个事件监听器。一旦 draggable 节点发生点击事件，React 在 document 上的事件监听器就会向原始的处理函数派发一个合成事件。

### 转换

In order to actually see your draggable items move on screen, you'll need to move the item using CSS. You can use inline styles, CSS variables, or even CSS-in-JS libraries to pass the `transform` property as CSS to your draggable element.

为了真正地看到 draggable 元素在屏幕上的移动效果，需要用到 CSS 来移动它。可以使用内联样式、CSS 变量，甚至是 CSS-in-JS 库，将 `transform` 属性值作为 CSS 传递给 draggable 元素。

{% hint style="success" %}
处于性能方面的考虑，强烈建议使用 CSS 的 **`transform`** 属性处理 draggable 元素在屏幕上的移动，其他的位置属性如 **`top`**、 **`left`** 或 **`margin`** 都会引起代价昂贵的重绘。了解更多关于 [CSS transforms](https://developer.mozilla.org/en-US/docs/Web/CSS/transform) 的信息。
{% endhint %}

当一个元素开始拖动，`transform` 属性会被填充为需要在屏幕上移动该元素的 `translate` 坐标。`transform` 对象遵循这样的形式：`{x: number, y: number, scaleX: number, scaleY: number}`

`x` 与 `y` 坐标表示 draggable 元素开始拖动后由原点出发的增量。

`scaleX` 与 `scaleY` 属性表示 draggable 元素与它当前所在的 droppable 容器之间的比例差异。对于创建 draggable 元素需要适应于当前所在 droppable 容器大小的界面非常有用。

`CSS` helper 完全是可选的；它是一个用于生成 [CSS transform ](https://developer.mozilla.org/en-US/docs/Web/CSS/transform) 字符串的辅助方法，相当于手动创建这样的字符串：

```javascript
CSS.Translate.toString(transform) ===
  `translate3d(${translate.x}, ${translate.y}, 0)`;
```

### 属性

The `useDraggable` hook \*\*\*\* provides a set of sensible default attributes for draggable items. We recommend you attach these to the HTML element you are attaching the draggable listeners to.

We encourage you to manually attach the attributes that you think make sense in the context of your application rather than using them all without considering whether it makes sense to do so.

For example, if the HTML element you are attaching the `useDraggable` `listeners` to is already a semantic `button`, although it's harmless to do so, there's no need to add the `role="button"` attribute, since that is already the default role.&#x20;

| Attribute              | Default value             | Description                                                                                                                                                                                                                                                                                         |
| ---------------------- | ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `role`                 | `"button"`                | <p>If possible, we recommend you use a semantic <code>&#x3C;button></code> element for the DOM element you plan on attaching draggable listeners to. </p><p></p><p>In case that's not possible, make sure you include the <code>role="button"</code>attribute, which is the default value.</p>      |
| `tabIndex`             | `"0"`                     | In order for your draggable elements to receive keyboard focus, they **need** to have the `tabindex` attribute set to `0` if they are not natively interactive elements (such as the HTML `button` element). For this reason, the `useDraggable` hook sets the `tabindex="0"` attribute by default. |
| `aria-roledescription` | `"draggable"`             | While `draggable` is a sensible default, we recommend you customize this value to something that is tailored to the use case you are building.                                                                                                                                                      |
| `aria-describedby`     | `"DndContext-[uniqueId]"` | Each draggable item is provided a unique `aria-describedby` ID that points to the [screen reader instructions](../context-provider/#screen-reader-instructions) to be read out when a draggable item receives focus.                                                                                |

To learn more about the best practices for making draggable interfaces accessible, read the full accessibility guide:

{% content-ref url="../../guides/accessibility.md" %}
[accessibility.md](../../guides/accessibility.md)
{% endcontent-ref %}

### Recommendations

#### `touch-action`

We highly recommend you specify the `touch-action` CSS property for all of your draggable elements.

> The **`touch-action`** CSS property sets how an element's region can be manipulated by a touchscreen user (for example, by zooming features built into the browser).\
> \
> Source: [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/touch-action)

In general, we recommend you set the `touch-action` property to `none` for draggable elements in order to prevent scrolling on mobile devices.&#x20;

{% hint style="info" %}
For [Pointer Events,](../sensors/pointer.md) there is no way to prevent the default behaviour of the browser on touch devices when interacting with a draggable element from the pointer event listeners. Using `touch-action: none;` is the only way to reliably prevent scrolling for pointer events.

Further, using `touch-action: none;` is currently the only reliable way to prevent scrolling in iOS Safari for both Touch and Pointer events.&#x20;
{% endhint %}

If your draggable item is part of a scrollable list, we recommend you use a drag handle and set `touch-action` to `none` only for the drag handle, so that the contents of the list can still be scrolled, but that initiating a drag from the drag handle does not scroll the page.

Once a `pointerdown` or `touchstart` event has been initiated, any changes to the `touch-action` value will be ignored. Programmatically changing the `touch-action` value for an element from `auto` to `none` after a pointer or touch event has been initiated will not result in the user agent aborting or suppressing any default behavior for that event for as long as that pointer is active (for more details, refer to the [Pointer Events Level 2 Spec](https://www.w3.org/TR/pointerevents2/#determining-supported-touch-behavior)).

## Drag Overlay

The `<DragOverlay>` component provides a way to render a draggable overlay that is removed from the normal document flow and is positioned relative to the viewport.

![](<../../.gitbook/assets/dragoverlay (1).png>)

To learn more about how to use drag overlays, read the in-depth guide:

{% content-ref url="drag-overlay.md" %}
[drag-overlay.md](drag-overlay.md)
{% endcontent-ref %}
