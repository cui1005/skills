# TakeCoordinates 两套事件通道（务必分清）

> 本文件对应 SKILL.md 路由表的「写事件相关代码」行。

> 一句话：**通道 A = 派生业务事件（仅 active 时自动生成）**；**通道 B = 底层拾取事件（无论是否 active 都收）**。

## 通道 A：派生事件（推荐）

注册 `tc.on(type, cb)`，type 优先用 `Defines.MapActionType.ADD/FINISH/MOVE`（其值即内部字面量 `'add'/'finish'/'move'`）。只有 active 状态下由 TakeCoordinates 自动生成：

| 事件 | type 值 | 触发 | 回调数据 |
|---|---|---|---|
| `ADD` | `'add'`（`MapActionType.ADD`） | 左键命中场景且 `isActive` | `{type:'add', position, mousePosition, pointKey, coordinates}` |
| `FINISH` | `'finish'`（`MapActionType.FINISH`） | 点数到 count 或右键结束 | `{type:'finish', coordinates}` |
| `MOVE` | `'move'`（`MapActionType.MOVE`） | 鼠标移动（需 activeMove/beginMoving 或 enableMouseMove） | `{type:'move', position, mousePosition, movingIndex?}`（拖拽锚点时带 movingIndex） |
| `REMOVE` | `'remove'`（字面量，MapActionType 枚举无此键） | `remove/removeByPointKey` | `{type:'remove', position, pointKey, coordinates}` |
| `DELETE`/`CHANGED` | `'delete'`/`'changed'` | 枚举存在；SDK 内部默认不主动 fire，供业务扩展（可自己 `tc.fire({type: D.MapActionType.DELETE, ...})`） |

> ADD 回调里的 `mousePosition` 是像素坐标 `{x, y}`，`position` 是经纬高。

## 通道 B：底层拾取事件（`Defines.ScreenSpaceEventType`）

注册 `tc.on(Defines.ScreenSpaceEventType.LEFT_CLICK / RIGHT_CLICK / MOUSE_MOVE / ... , cb)`。由控制器每帧拾取后向全部注册实例派发（**无论是否 active 都会收**），数据形态：

```
{ type: 数值(如 LEFT_CLICK=2),            // 即 ScreenSpaceEventType 数值
  position?: {lon,lat,height},            // 由 worldPosition 换算，仅命中场景时有
  worldPosition: Cartesian3,              // 命中点世界坐标（吸附模式下为吸附点）
  mousePosition: {x,y},                   // 屏幕像素
  pickedObject,                           // 命中的 Entity/Primitive/图层对象等
}
```

用途：需要在“未 active”时也要感知点击（如做右键菜单、空白检测）、或拿到 `pickedObject` 做对象判断。

## 行为注意事项

- active 期间左键会**同时**产生 `ADD` 和底层 `LEFT_CLICK` 两路；若要“能自动累积点”就用 ADD，若要自己完全控制就保持 `deactive` 只监听 LEFT_CLICK。
- MOUSE_MOVE 默认**不派发**（性能门控）：只有当某注册实例 `enableMouseMove=true`（`activeMove`/`beginMoving` 会置位）控制器才持续派发移动事件。
- 右键结束只影响派生通道：`rightClickEnd=true` 时右键会先 `stopMove/deactive + FINISH`；你在 `RIGHT_CLICK` 上另挂的回调仍会执行。
- 纯悬浮预览请用底层 `ScreenSpaceEventType.MOUSE_MOVE`（只需 `enableMouseMove`，不依赖 active）；派生 `MOVE` 要求 `isActive && enableMouseMove` 并会顺带累积左键点。
