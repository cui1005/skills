# TakeCoordinates 完整 API（构造 / 属性 / 方法 / 坐标与 Key）

> 本文件对应 SKILL.md 路由表的「查方法/属性/坐标与 pointKey 格式」行。

> 相关：`references/quick-start.md`（可运行骨架）、`references/events.md`（事件回调数据）

## 3.1 构造与实例属性

| 项 | 说明 |
|---|---|
| `new TakeCoordinates(controls)` | controls 必须是 map 的拾取控制器（即 `map.controls`）。构造即向它注册（push 进 `allTakeCoordinates`），之后能收到全部拾取事件 |
| `.controls`（只读） | 构造时传入的控制器（`map.controls`） |
| `.isActive` | 布尔。`active()`/`deactive()` 维护，默认 false；仅 true 时左键点击才自动累积点 |
| `.intercep` | 布尔。双击场景防连击的内部标记，默认 false（一般不手动改） |
| `.enableMouseMove` | 布尔（继承自 MouseMoveEventDispatcher）。默认 false；置 true 或调 `activeMove` 后控制器才持续派发鼠标移动事件 |
| 继承 | `MouseMoveEventDispatcher` -> `EventDispatcher`：可用 `on/off/fire/hasEventListener/dispatchEvent/removeEventListeners` |

## 3.2 方法与真实行为

| 方法 | 行为 |
|---|---|
| `active(count = 500, rightClickEnd = false)` | 进入取点：每次左键（命中场景）自动累积一个点并 fire `ADD`；当累积数等于 `count` 自动 `deactive()` 并 fire `FINISH`；`rightClickEnd=true` 时右键触发 `stopMove/deactive + FINISH`。默认 count 500 意味着不主动结束就一直在收点 |
| `deactive()` | 停止累积（isActive=false），已有点保留 |
| `activeMove(rightClickMovingEnd = false)` | 进入移动悬停模式：开启鼠标移动事件流；鼠标移动时 fire `MOVE`（不改动已有锚点）；`rightClickMovingEnd=true` 时右键自动 `stopMove()` |
| `beginMoving(index, rightClickMovingEnd = false)` | 指定第 index 个锚点进入"拖拽更新"：之后移动事件会同步改写内部该点坐标，并 fire `MOVE`（带 `movingIndex`） |
| `beginMovingByPointKey(pointKey, rightClickMovingEnd = false)` | 按 ADD 事件返回的 `pointKey` 定位后 `beginMoving` |
| `stopMove()` | 结束移动/悬停（enableMouseMove=false，movingIndex 复位） |
| `clearPoint()` | 清空内部锚点数组（不退出 active） |
| `remove(index)` | 删除第 index 个锚点并 fire `REMOVE`（事件：`{type:'remove', position, pointKey, coordinates}`） |
| `removeByPointKey(pointKey)` | 按 key 删除锚点 |
| `clear()` | 一键复位：stopMove + deactive + clearPoint + 解绑全部监听后重建内部监听，实例可继续用 |
| `dispose()` | 从控制器注册表移除（`allTakeCoordinates` 删除自己）+ 复位 + 移除全部监听。**自建实例用完必须调**，否则它永远在收场景事件 |

## 3.3 坐标与 Key 格式

- 事件坐标统一为 **经纬高对象 `{lon, lat, height}`**（lon/lat 度、height 米），由控制器把场景拾取到的世界坐标换算而来。
- 每个 ADD 点带 `pointKey`：36 位 UID 字符串，供 `removeByPointKey/beginMovingByPointKey` 引用。
- `coordinates` 数组元素即 `{lon,lat,height}` 对象，事件回调中为**当前全量快照**。
- 像素坐标另有 `mousePosition: {x, y}`（屏幕坐标），与 `position`（经纬高）不要混用。
