# useDndContext

For advanced use-cases, for example, if you're building your own presets on top of `@dnd-kit/core`, you may want to have access to the internal context of `<DndContext>` that the `useDraggable` and `useDroppable` have access to.

对于更高级的使用场景，比如，你正在 `@dnd-kit/core` 之上构建自己的预置能力，你可能需要访问 `<DndContext>` 中 `useDraggable` 与 `useDroppable` 能够访问的内部上下文。

```jsx
import { useDndContext } from "@dnd-kit/core";

function CustomPreset() {
  const dndContext = useDndContext();
}
```

如果你认为你正在开发的预置能力对其他人也有帮助，请在 `dnd-kit` 的仓库中新开一个 PR 进行讨论。
