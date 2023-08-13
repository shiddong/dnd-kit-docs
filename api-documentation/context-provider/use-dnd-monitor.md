# useDndMonitor

在 `DndContext` provider 所包裹的组件下使用 `useDndMonitor` hook，可以监听发生在 `DndContext` 中的各种拖放事件。

```jsx
import { DndContext, useDndMonitor } from "@dnd-kit/core";

function App() {
  return (
    <DndContext>
      <Component />
    </DndContext>
  );
}

function Component() {
  // 监听发生在父级 `DndContext` provider 下的拖放事件
  useDndMonitor({
    onDragStart(event) {},
    onDragMove(event) {},
    onDragOver(event) {},
    onDragEnd(event) {},
    onDragCancel(event) {},
  });
}
```
