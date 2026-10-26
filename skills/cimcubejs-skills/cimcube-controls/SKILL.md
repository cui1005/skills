---
name: cimcube-controls
description: "CIMCube 场景交互控件 Controls —— 取点交互体系：内建 TakeCoordinates 单例，或自建取点处理器；事件通道（ADD/MOVE/FINISH，携带坐标/点标识/拾取对象）与底层鼠标移动事件（MOUSE_MOVE）；吸附命中接管左键取点。控件不渲染光标/提示/临时图形，画点画线与撤销重做需业务层自行实现。适合取点输入/吸附取点/自定义点选工具/锚点拖拽编辑时激活。"
---

# CIMCube Controls 交互取点（TakeCoordinates / map.controls）

> 本技能回答「在三维场景里怎么用鼠标取坐标点 / 拖动锚点 / 吸附到对象特征点」这一族问题。
> **本文件是纯路由索引**：先按 §二 路由表读对应 `references/` 文件，再写码；不要凭记忆写方法名与事件名。

## 一、心智模型（30 秒版）

```
map.controls（构造时自动创建，注册全部鼠标事件并每帧拾取）
├── takeCoordinates      // 内建 TakeCoordinates 单例
├── allTakeCoordinates   // 全部实例（含你 new 的）
└── snapTypeList / snapTargetType / snapSelectedResults / cancelSelectedPickObject()
Glodon.CIMCube.Controls（运行时只导出 2 个类）
├── TakeCoordinates      // 取点/锚点管理：active 后累积左键点并派发事件
└── PickEventHandler     // 轻量事件壳（无累积逻辑，很少直接用）
```

四条硬约束：

1. **控件不画任何东西**——不加标记、不画线、不显示提示；视觉一律用 cimcube-graphic 自绘。
2. **只有 `active()` 状态才自动累积点**；未 active 时只能靠底层 `ScreenSpaceEventType` 通道自己收。
3. **自建实例用完必须 `dispose()`**，否则它永远留在 `allTakeCoordinates` 里继续收场景事件。
4. **自绘反馈的三步走对接口**：事件坐标是 `{lon,lat,height}` **对象**，Graphic 图元要 `[lon,lat,height]` **数组**（须转换）；建图层用 `map.layerManager.addLayer({ type:Glodon.CIMCube.Global.Defines.LayerType.GRAPHIC, ... })`，加/删图元用 `addGraphic()` / `removeGraphic()`。

> 另两条常见误解：`Controls.BaseControl` / `Controls.ControlManager` 在 SDK 中**不存在**（所谓「5 种取点模式 / 事件链 / 撤销重做」全为臆造）；`PickEventHandler` **无官方用法示例**，常规取点直接用 `TakeCoordinates`。

## 二、路由表（按需加载，不要一次全读）

| 你要做的事 | 先读 |
|---|---|
| 第一次上手 / 要可运行骨架 | `references/quick-start.md`（对象关系全图 + Quick Start + Promise 模板 + 画点标记） |
| 查方法/属性/坐标与 pointKey 格式 | `references/take-coordinates-api.md`（构造属性、11 个方法真实行为、坐标与 Key 格式） |
| 写事件相关代码 | `references/events.md`（通道 A 派生事件 + 通道 B 底层事件 + 行为注意） |
| 吸附到模型角点/面/端点 | `references/snap.md`（`map.controls` 吸附相关暴露项 + 吸附接管取点的链路；Snap 类构造项/方法/事件 → `cimcube-utils/references/Snap.md`） |
| 画折线/多边形、悬浮预览、锚点拖拽、工具按钮 | `references/patterns.md`（4 组业务封装 + 组合排错提示） |
| 确认某类的归属 / 是否真实存在 | `using-cimcube-skills/references/class-routing.md`（类目唯一真相源视图，自动生成；由仓库侧类目登记表生成的视图） |

最小骨架（粘到 `new Glodon.CIMCube(...)` 之后即可跑，完整版见 `references/quick-start.md`）：

```js
const D = Glodon.CIMCube.Global.Defines;
const tc = map.controls.takeCoordinates;   // 内建单例；隔离用 new Glodon.CIMCube.Controls.TakeCoordinates(map.controls)
tc.on(D.MapActionType.ADD,    (e) => console.log('新增点', e.position, e.pointKey));
tc.on(D.MapActionType.FINISH, (e) => console.log('完成', e.coordinates));
tc.active(3, true);                        // 取满 3 点自动结束；true = 右键可提前结束
```

事件一句话记忆：**通道 A = 派生业务事件**（`MapActionType.ADD/MOVE/FINISH`，仅 active 时自动生成，带 `pointKey`）；**通道 B = 底层拾取事件**（`ScreenSpaceEventType.LEFT_CLICK/MOUSE_MOVE`，无论是否 active 都收，带 `pickedObject`）。全表见 `references/events.md`。

## 三、与相邻模块的边界

- 测量/量算：`Utils.Measure` 等自带标记与结果渲染，不想自己画 UI 时优先用（见 `cimcube-utils/references/Measure.md`）。
- 吸附：`Utils.Snap` 负责吸附可视化与命中（见 `cimcube-utils/references/Snap.md`）；TakeCoordinates 只负责收点。
- 底层拾取：`map.scene.pick` 系 / Layer 的 `LayerEventType`（见 cimcube-camera / cimcube-layer）。
- 自绘反馈：坐标到手后用 cimcube-graphic 的 Point/Polyline/Polygon 画过程图形。
- Analysis / 剖切等带点参数的业务：同样用本技能把点坐标喂给分析器。

## See Also

- **cimcube-utils** — `Utils.Measure` 现成交互测量（`cimcube-utils/references/Measure.md`）
- **cimcube-utils** — `Utils.Snap` 吸附与编辑类工具（`cimcube-utils/references/Snap.md`）
- **cimcube-graphic** — 用取到的点坐标绘制点/线/面标记
- **cimcube-global** — `Defines.MapActionType / ScreenSpaceEventType / SnapType / SnapTargetType` 常量取值
