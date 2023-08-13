# Droppable

![](../../.gitbook/assets/droppable-large.svg)

使用 `useDroppable` 可以将一个 DOM 节点设置为 droppable 区域，从而使 [draggable](../draggable/) 元素可以放入其中。

## 使用方式

`useDroppable` 对应用程序的代码结构没有特别的要求。

但至少需要将 `useDroppable` 中返回的 `setNodeRef` 函数传递给 DOM 元素，这样它才能对底层的 DOM 节点进行注册，并通过保持对它的跟踪，从而检测它与其他的 draggable 元素是否发生碰撞或相交。

{% hint style="info" %}
如果你是刚接触 `ref` 的概念，推荐你先看看 React 官方文档中的相关文章 [Refs 与 DOM](https://reactjs.org/docs/refs-and-the-dom.html#adding-a-ref-to-a-dom-element)
{% endhint %}

```jsx
import { useDroppable } from "@dnd-kit/core";

function Droppable() {
  const { setNodeRef } = useDroppable({
    id: "unique-id",
  });

  return <div ref={setNodeRef}>/* 任何你想要渲染的内容 */</div>;
}
```

你可以设置任意数量的 droppable 容器，只要确保它们都有唯一的 `id` 进行区分。不过，每个 droppable 容器都应该是唯一的节点，所以不能将多个 ref 赋给同一个 droppable 容器。

若要设置多个 droppable 容器，只要多次使用 `useDroppable` 即可。

```jsx
function MultipleDroppables() {
  const { setNodeRef: setFirstDroppableRef } = useDroppable({
    id: "droppable-1",
  });
  const { setNodeRef: setSecondDroppableRef } = useDroppable({
    id: "droppable-2",
  });

  return (
    <section>
      <div ref={setFirstDroppableRef}>/* 任何你想要渲染的内容 */</div>
      <div ref={setSecondDroppableRef}>/* 任何你想要渲染的内容 */</div>
    </section>
  );
}
```

如果需要动态地渲染一个 droppable 容器的列表，推荐创建一个可复用的 Droppable 组件，然后根据需要多次渲染该组件。

```jsx
function Droppable(props) {
  const { setNodeRef } = useDroppable({
    id: props.id,
  });

  return <div ref={setNodeRef}>{props.children}</div>;
}

function MultipleDroppables() {
  const droppables = ["1", "2", "3", "4"];

  return (
    <section>
      {droppables.map((id) => (
        <Droppable id={id} key={id}>
          Droppable 容器的 id: ${id}
        </Droppable>
      ))}
    </section>
  );
}
```

更多关于 `useDroppable` 的使用细节，请参考 API 文档中的以下章节：

{% page-ref page="usedroppable.md" %}
