# useDroppable

![](../../.gitbook/assets/droppable-1-.png)

## 参数

```typescript
interface UseDroppableArguments {
  id: string | number;
  disabled?: boolean;
  data?: Record<string, any>;
}
```

### 标识符

参数 `id` 是一个 `string` 或 `number` 类型的唯一标识符。在某个 [`DndContext`](../context-provider/) provider 下不应存在其他的 **droppable** 元素使用同一个标识符。

当你同时使用 `useDroppable` 和 `useDraggable` 创建一个组件时，它们是可以使用同一个标识符的，因为 droppable 与 draggable 元素不是存储在同一个键值对之中。

### 禁用

由于[不能在条件语句中调用 hook](https://reactjs.org/docs/hooks-rules.html)，所以当你需要临时禁用某个 `droppable` 区域时，可以将 `disabled` 参数设置为 `true`。

### 数据

在一些复杂的使用场景中，你可能需要在事件处理函数、modifiers 或自定义的 sensors 中访问关于 droppable 元素的一些数据，此时可以使用 `data` 参数。

例如，当你正在开发一个可排序的预置能力，你可以使用 `data` 属性来存储排序列表中 droppable 元素的索引，便于在自定义的 sensor 中访问它。

```jsx
const { setNodeRef } = useDroppable({
  id: props.id,
  data: {
    index: props.index,
  },
});
```

另一个更复杂的例子是，`data` 参数有助于在 draggable 节点与 droppable 区域之间建立关联，例如，通过它可以指定某个 droppable 区域可以接收哪些类型的 draggable 节点。

```jsx
import { DndContext, useDraggable, useDroppable } from "@dnd-kit/core";

function Droppable() {
  const { setNodeRef } = useDroppable({
    id: "droppable",
    data: {
      accepts: ["type1", "type2"],
    },
  });

  /* ... */
}

function Draggable() {
  const { attributes, listeners, setNodeRef, transform } = useDraggable({
    id: "draggable",
    data: {
      type: "type1",
    },
  });

  /* ... */
}

function App() {
  return <DndContext onDragEnd={handleDragEnd}>/* ... */</DndContext>;

  function handleDragEnd(event) {
    const { active, over } = event;

    if (over && over.data.current.accepts.includes(active.data.current.type)) {
      // 处理逻辑
    }
  }
}
```

## 属性

```typescript
{
  rect: React.MutableRefObject<LayoutRect | null>;
  isOver: boolean;
  node: React.RefObject<HTMLElement>;
  over: {id: UniqueIdentifier} | null;
  setNodeRef(element: HTMLElement | null): void;
}
```

### Node

#### `setNodeRef`

为了使 `useDroppable` 能够正常工作，需要将 `setNodeRef` 属性附加在想要成为 droppable 区域的 HTML 元素上。

```jsx
function Droppable(props) {
  const { setNodeRef } = useDroppable({
    id: props.id,
  });

  return <div ref={setNodeRef}>{/* ... */}</div>;
}
```

#### `node`

当前节点的[引用](https://reactjs.org/docs/refs-and-the-dom.html)，其接收了传递的 `setNodeRef` 参数

#### `rect`

对于某些复杂的场景，可能需要当前 droppable 区域的边界矩形测量数据。

### Over

#### `isOver`

当某个 `draggable` 元素被拖动到 droppable 区域时，可以通过 `useDroppable` 返回的布尔值类型的 `isOver` 属性改变其显示的外观或内容。

#### `over`

如果你想要改变 droppable 元素的外观，以响应 draggable 元素被拖动到不同的 droppable 容器之上，可以检查是否定义了 `over` 值。根据使用场景，也可以通过 `id` 属性对 其他 droppable 组件的渲染输出进行修改
