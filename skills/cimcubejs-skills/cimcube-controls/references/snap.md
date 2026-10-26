# map.controls 暴露项 + 吸附（Snap 联动）

> 本文件对应 SKILL.md 路由表的「吸附到模型角点/面/端点」行。

> 相关：`cimcube-utils/references/Snap.md`（Snap 类自身的全部构造项、方法、属性与事件）

## map.controls 暴露项（源码级，APIDOC 命名空间页对应它）

| 项 | 说明 |
|---|---|
| `map.controls.takeCoordinates` | 内建 TakeCoordinates（CIMCube 构造时自动 new 并注册） |
| `map.controls.allTakeCoordinates` | 已注册 TakeCoordinates 数组（含内建与你 new 的） |
| `map.controls.enabledClickDelay` | 双击消抖：200ms 内第二击视为双击不产生单击取点。默认 false |
| `map.controls.snapTargetType` | `Defines.SnapTargetType`：`MODEL_3D='model3D' / VECTOR_2D='vector2D' / BOTH='both'`，默认 BOTH |
| `map.controls.snapTypeList` | `Defines.SnapType` 数组：`ENDPOINT/MIDPOINT/LINE/FACE/INTERSECTION`，默认 `['face','endPoint']`（面+端点） |
| `map.controls.snapSelectedResults` | 最近一次吸附命中的结果（含吸附世界点/对象），null 表示未命中 |
| `map.controls.cancelSelectedPickObject()` | 取消当前被高亮选中的拾取对象（清除其选中高亮状态） |
| `map.controls.Snap` | 激活中的吸附服务引用（内部由 `Utils.Snap.enable()` 注入，**勿手动赋值**） |

> 说明：APIDOC 把控制器这些成员显示在 `Glodon.CIMCube.Controls` 命名空间下，但**运行时命名空间只导出 2 个类**，上述入口统一走 `map.controls`，别被文档页误导去访问 `Controls.takeCoordinates`。

## 吸附如何接管取点

吸附体系不在 Controls 类上开启，而是由 **`Glodon.CIMCube.Utils.Snap`** 接管：

1. `new Utils.Snap(map, opts).enable()` — 内部注册为 `map.controls.Snap`、置位控制器的 `_enableMouseSnap`。
2. 鼠标靠近吸附特征（对象面/端点/中点/交点，范围受 `snapTargetType`/`snapTypeList`/`tolerance` 控制）时，Snap 高亮吸附标记，结果写入 `map.controls.snapSelectedResults`。
3. 此时左键点击，控制器**不再按鼠标实际位置取点**，而是以 `snapSelectedResults.snapWorldPosition` 作为点击点 — 正在 active 的 TakeCoordinates 也会在 `ADD` 里收到吸附后的坐标。
4. 吸附点击命中时向 Snap 实例 fire `'mouseSnapClickup'`，`arguments` 即完整吸附结果；随后自动清除高亮。

构造项、方法、属性与事件详见 `cimcube-utils/references/Snap.md`；组合用法（吸附开关 + 精准画线）见 `cimcube-utils/references/patterns.md` 模式 E。
