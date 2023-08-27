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

为了真正地看到 draggable 元素在屏幕上的移动效果，需要用到 CSS 来移动它。你可以使用内联样式、CSS 变量，甚至是 CSS-in-JS 库，将 `transform` 属性值作为 CSS 传递给 draggable 元素。

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

`useDraggable` 为 draggable 元素提供了合理的默认属性集。推荐你为附加了 draggable listeners 的 HTML 元素也同时附加上这些属性。

最好是能够手动的选择具有语义的属性，而不是在无脑地直接全部使用。

比如，如果附加了 `useDraggable` 导出的 `listeners` 的 HTML 元素本身就是一个语义化的 `button`，由于它已经具有默认的 role 属性，就没有必要再为其设置属性 `role="button"`，尽管这也没什么影响。

| 属性                   | 默认值                    | 描述                                                                                                                                                                                |
| ---------------------- | ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `role`                 | `"button"`                | <p>如果可以，最好是使用语义化的 <code>&#x3C;button></code> 来表示附加 listeners 的 DOM 元素。 </p><p></p><p>如果不可以，需要包含属性 <code>role="button"</code>，这也是默认值。</p> |
| `tabIndex`             | `"0"`                     | 如果 draggable 元素不是本地交互元素（如 HTML 的 `button`），则为了使它们能接收到键盘焦点，**需要**将 `tabIndex` 属性设置为 `0`。因此，`useDraggable` 默认设置了 `tabIndex="0"`。    |
| `aria-roledescription` | `"draggable"`             | 虽然 `draggable` 是一个合理地默认值，但还是建议你将它设置为与当前上下文相匹配的某个值。                                                                                             |
| `aria-describedby`     | `"DndContext-[uniqueId]"` | 每个 draggable 元素都有一个唯一的 `aria-describedby` ID，该 ID 指向[屏幕阅读器指令](../context-provider/#screen-reader-instructions)，以便在 draggable 元素获得焦点时进行读出。     |

想要了解更多关于使 draggable 界面无障碍性的最佳实践，请阅读完整的无障碍指南：

{% content-ref url="../../guides/accessibility.md" %}
[accessibility.md](../../guides/accessibility.md)
{% endcontent-ref %}

### 推荐

#### `touch-action`

强烈推荐为所有的 draggable 元素指定 CSS 属性 `touch-action`。

> CSS 属性 **`touch-action`** 设置了触摸屏用户如何在某个元素区域中进行操作（比如，通过浏览器内置的缩放功能进行操作）。\
> \
> 来源: [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/touch-action)

通常，为了防止 draggable 元素在移动设备上进行滚动，建议将它的 `touch-action` 属性设置为 `none`。

{% hint style="info" %}

对于[指针事件](../sensors/pointer.md)，当与来自指针事件监听器的 draggable 元素交互时，将无法阻止浏览器在触摸设备上的默认行为。使用 `touch-action: none;` 是在触发指针事件时，可以防止滚动的唯一可靠方法。

此外，使用 `touch-action: none;` 是目前对于在 iOS Safari 中的触摸与指针事件，可以防止滚动的唯一可靠方法。
{% endhint %}

如果 draggable 元素是一个滚动列表中的一部分，建议使用一个拖动手柄，同时仅为拖动手柄设置属性 `touch-action` 为 `none`，这样，列表中的内容仍然可以滚动，但根据拖动手柄触发拖动后不会在页面中滚动。

一旦触发了 `pointerdown` 或 `touchstart` 事件，对 `touch-action` 值的任何修改都会被忽略。在触发指针或触摸事件后，只要指针处于 active 状态，通过编程化的方式将 `touch-action` 的值从 `auto` 修改为 `none`，不会导致用户代理终止或压制该事件的任何默认行为。（更多的细节信息，可以参考[Pointer Events Level 2 Spec](https://www.w3.org/TR/pointerevents2/#determining-supported-touch-behavior)）

## Drag Overlay

`<DragOverlay>` 组件支持为 draggable 元素渲染一个脱离正常文档流饼相对于视口定位的浮层。

![](<../../.gitbook/assets/dragoverlay (1).png>)

想要了解更多关于如何使用拖动浮层的信息，请阅读以下指南：

{% content-ref url="drag-overlay.md" %}
[drag-overlay.md](drag-overlay.md)
{% endcontent-ref %}
