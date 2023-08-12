# useDndMonitor

可以在 `DndContext` provider 所包裹的组件下使用 `useDndMonitor` hook，监视发生在 `DndContext` 中不同的拖放事件。

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
  // 监视发生在父级 `DndContext` provider 下的 拖放事件
  useDndMonitor({
    onDragStart(event) {},
    onDragMove(event) {},
    onDragOver(event) {},
    onDragEnd(event) {},
    onDragCancel(event) {},
  });
}
```
