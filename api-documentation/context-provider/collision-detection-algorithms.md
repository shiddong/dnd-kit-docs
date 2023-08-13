# Collision detection algorithms

如果你熟悉如何制作 2D 游戏，你可能已经了解过碰撞检测算法的概念。

一个比较简单的形式是两个轴对齐的矩形之间的碰撞检测 —— 矩形没有发生旋转。这种形式的碰撞检测通常被称为 [Axis-Aligned Bounding Box](https://developer.mozilla.org/en-US/docs/Games/Techniques/2D_collision_detection#Axis-Aligned_Bounding_Box) (AABB).

在内置的碰撞检测算法中，假定有一个矩形边框。

> 一个元素的边框是完全包围了该元素及其子元素的最小可能矩形（与该元素的用户坐标系的轴对齐）。\
> – 来源: [MDN](https://developer.mozilla.org/en-US/docs/Glossary/bounding_box)

也就是说，不管 draggable 与 droppable 节点是一个圆形还是三角形，它们的边框仍然是一个矩形。

![](../../.gitbook/assets/axis-aligned-rectangle.png)

如果你想使用矩形以外的形状进行碰撞检测，可以创建一个自己的[自定义碰撞检测算法](collision-detection-algorithms.md#zi-ding-yi-peng-zhuang-jian-ce-suan-fa)。

## 矩形相交

默认情况下，[`DndContext`](./) 使用的就是**矩形相交**的碰撞检测算法。

该算法的工作原理是当两个矩形的任意 4 条边无间隔，即存在相交，说明此时发生了碰撞。存在任何的间隔都意味着没有发生碰撞。

所以，为了使得一个 draggable 元素是 **over** 在一个 droppable 区域之上的，需要这两者的矩形边框是相交的。

![](../../.gitbook/assets/rect-intersection-1-.png)

## 最近中心点

虽然矩形相交算法适用于大多数的拖放场景，但由于它要求 draggable 元素与 droppable 区域的矩形边框直接接触并相交，相对来说，这是一种比较严格的方式。

而对于某些场景，比如[排序](../../presets/sortable/)列表来说，则更加推荐使用一个相对更宽容的碰撞检测算法。

顾名思义，最近中心点算法是找到某个中心点离当前 draggable 元素的矩形边框中心点最近的 droppable 容器：

![](../../.gitbook/assets/closest-center-2-.png)

## 最近邻角

与最近中心点算法相似，最近邻角算法也不要求 draggable 元素与 droppable 区域的矩形边框直接相交。

而是测量当前的 draggable 元素的所有 4 个角与每个 droppable 区域的 4 个角的距离，从而找到最近的那个 droppable 区域。

![](../../.gitbook/assets/closest-corners.png)

如上图所示，距离测算方式是分别对 draggable 元素与 droppable 区域矩形边框的对应角进行计算：左上角对左上角，右上角对右上角，左下角对左下角，右下角对右下角。

### **何时该使用最近邻角算法而非最近中心点算法?**

在大多数情况下，**最近中心点算法**的效果很好，并且通常是排序列表场景下的默认推荐算法，因为它相较于**矩形相交算法**具有更为宽容的体验。

通常，最近中心点与最近邻角算法得到的结果一致。但在某些情况下有所例外，例如在创建 droppable 容器相互堆叠的界面时，比如看板，最近中心点算法得到的结果有时会是整个看板列的下层 droppable 区域，而不是当前列中的某个 droppable 区域。

![最近中心点是 'A'，但通过人眼看到的可能以为是 'A2'](../../.gitbook/assets/closest-center-kanban.png)

在这些情况下，首选使用**最近邻角**算法，其结果更符合人眼所看到的结果。

![正如通过人眼所看到的，最近邻角是 'A2'](../../.gitbook/assets/closest-corners-kanban.png)

## Pointer within

顾名思义，pointer within 碰撞检测算法仅仅在指针进入到其他 droppable 容器的边界矩形时才会记录碰撞。

这种碰撞检测算法非常适合于高精度的拖放页面。

{% hint style="info" %}

顾名思义，这种碰撞检测算法**仅仅适用于基于指针的 sensors**。所以，如果你想使用 `pointerWithin` 碰撞检测算法的话，建议使用[组合的碰撞检测算法](collision-detection-algorithms.md#zu-he-xian-you-de-peng-zhuang-jian-ce-suan-fa)，这样可以为键盘 sensor 回退到其他形式的碰撞检测算法。
{% endhint %}

## 自定义碰撞检测算法

在更高级别的使用场景中，可能我们提供的这些碰撞检测算法并不适用，那么你需要自定义一个自己的碰撞检测算法。

你可以从头开始编写一个新的碰撞检测算法，也可以对现有的两种或多种算法进行组合使用。

### 组合现有的碰撞检测算法

有时候，你不需要从头开始编写一个自定义的碰撞检测算法，完全可以组合现有的碰撞算法来增强它们。

一个比较常见的例子是使用 `pointerWithin` 碰撞检测算法。顾名思义，这种算法依赖于指针坐标，因此在使用键盘 sensor 时不起作用。它也是一种非常高精度的碰撞检测算法，有助于在没有检测到碰撞时回退到更宽容的算法形式。

```javascript
import { pointerWithin, rectIntersection } from "@dnd-kit/core";

function customCollisionDetectionAlgorithm(args) {
  // 首先，看看根据指针是否能检测到任何碰撞
  const pointerCollisions = pointerWithin(args);

  // 碰撞检测算法会返回一个碰撞记录列表
  if (pointerCollisions.length > 0) {
    return pointerCollisions;
  }

  // 若根据指针没有检测到碰撞，则使用矩形相交算法
  return rectIntersection(args);
}
```

算法组合的另一个例子是，对某些 droppable 容器与其他的容器使用不一样的碰撞检测算法。

例如，当你开发一个排序列表，同时也支持将列表中某些元素移入到垃圾桶中，那么你可以组合使用`最近中心点`和`矩阵相交`碰撞检测算法。

![对除垃圾桶以外的所有 droppable 容器使用最近邻角算法](../../.gitbook/assets/custom-collision-detection.png)

![对于垃圾桶的 droppable 容器使用矩阵相交算法](../../.gitbook/assets/custom-collision-detection-intersection.png)

从代码实现的角度看，上述示例中的自定义相交算法如下所示：

```javascript
import { closestCorners, rectIntersection } from "@dnd-kit/core";

function customCollisionDetectionAlgorithm({ droppableContainers, ...args }) {
  // 首先，看看垃圾桶的 droppable 容器矩形边框是否相交
  const rectIntersectionCollisions = rectIntersection({
    ...args,
    droppableContainers: droppableContainers.filter(({ id }) => id === "trash"),
  });

  // 碰撞检测算法会返回一个碰撞记录列表
  if (rectIntersectionCollisions.length > 0) {
    // 当发生相交，则提前返回
    return rectIntersectionCollisions;
  }

  // 若没有发生相交，则使用其他碰撞检测形式
  return closestCorners({
    ...args,
    droppableContainers: droppableContainers.filter(({ id }) => id !== "trash"),
  });
}
```

### 创建自定义的碰撞检测算法

对于更高级的使用场景或检测非矩形、非轴对齐形状之间的碰撞，你需要创建一个自定义的碰撞检测算法。

下面是一个[检测圆形之间碰撞](https://developer.mozilla.org/en-US/docs/Games/Techniques/2D_collision_detection#Circle_Collision)的例子。

```javascript
/**
 * Sort collisions in descending order (from greatest to smallest value)
 */
export function sortCollisionsDesc(
  {data: {value: a}},
  {data: {value: b}}
) {
  return b - a;
}

function getCircleIntersection(entry, target) {
  // Abstracted the logic to calculate the radius for simplicity
  var circle1 = {radius: 20, x: entry.offsetLeft, y: entry.offsetTop};
  var circle2 = {radius: 12, x: target.offsetLeft, y: target.offsetTop};

  var dx = circle1.x - circle2.x;
  var dy = circle1.y - circle2.y;
  var distance = Math.sqrt(dx * dx + dy * dy);

  if (distance < circle1.radius + circle2.radius) {
    return distance;
  }

  return 0;
}

/**
 * Returns the circle that has the greatest intersection area
 */
function circleIntersection({
  collisionRect,
  droppableRects,
  droppableContainers,
}) => {
  const collisions = [];

  for (const droppableContainer of droppableContainers) {
    const {id} = droppableContainer;
    const rect = droppableRects.get(id);

    if (rect) {
      const intersectionRatio = getCircleIntersection(rect, collisionRect);

      if (intersectionRatio > 0) {
        collisions.push({
          id,
          data: {droppableContainer, value: intersectionRatio},
        });
      }
    }
  }

  return collisions.sort(sortCollisionsDesc);
};
```

想要了解更多信息，请参考内置的碰撞检测算法的实现。
