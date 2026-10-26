---
name: cimcube-graphic
description: "CIMCube 图元/矢量标注 Graphic：图元覆盖点/线/面/体（Polygon、Polyline、椭圆/圆、平面板、Box、Cylinder、Ellipsoid、Corridor、Rectangle、Wall）与标注类（Billboard 广告牌、Label 标注）；坐标类 GeoPoint。经图元图层增删查与事件。用户提点/线/面等图元或增删改查/拾取时激活；绘制编辑归 cimcube-layer。"
---

# CIMCube Graphic 图元/矢量标注（14 种实体图元）

> **基线**：命名空间 `Glodon.CIMCube.Graphic`（下文 `G` = `Glodon.CIMCube.Graphic`）。
> **校准声明**：精读 18 个 API_DOC 类页 + GraphicLayer 文档，并逐段核验 `cimcube.js` 导出/方法实现。
> **口径**：图元改样式**只有一个入口 `setStyle(xxxStyleOptions)`**（原 `setOptions` 即将废弃，已从本技能全部移除）；其中仅 `Point/Label/Billboard/Polygon/Polyline/Wall` 6 类提供该接口，余者须在构造 opts 里一次性给全——覆盖清单见 `references/styles.md`。标注 `Model` 未导出。
> ⚠️ 常见臆造写法——**全部不存在**：`position:{x,y,z}`、构造时把样式全部塞进 `style:{...}`、枚举数值（HorizontalOrigin/VerticalOrigin/CornerType 方向/角型数值写反）、Polygon/Polyline 的 `clampToGround` 默认值写反、Corridor `width` 当线宽、`GraphicLayer.add/update/removeById/getGraphicsByCondition` 等方法名。本页所有接口与默认值均来自 API_DOC 原文。
> **定位**：本 SKILL.md 只做**索引路由 + 跨域铁律 + 通用构造选项 + 枚举速查**。各图元族完整构造选项/方法/示例全部外置于 `references/<图元族>.md`。**写某类图元代码前，先读对应文件**，不要凭记忆重写示例。

## 一、四条铁律（最容易错）

1. **未导出 `BaseGraphic`/`ModelGraphic`**：`Glodon.CIMCube.Graphic` 没有这两个键，`new G.ModelGraphic(...)` 会报错。要往场景挂 GLTF 模型，走 `cimcube-layer` 的模型图层。
2. **`position`/`positions` 是数值数组**：点类 `[lon,lat,height]`（高度省略=0）；线面类为数组的数组。**不要传 `{x,y,z}` 对象**。
3. **贴地默认值别记反**：`PolygonGraphic.clampToGround` **默认 `true`**；`PolylineGraphic` 构造**没有** `clampToGround` 键（线条贴地/被遮挡样式走 `setStyle(PolylineStyleOptions)` 的 `clampToGround` / `occludedLine`）。
4. **`CorridorGraphic.width` = 缓冲半径（米）**，不是总宽、不是像素。输入 15 → 左右各 15m 共 30m 宽。

> 另：枚举方向/角型极易记反，见下方 §三（`VerticalOrigin.TOP = -1`、`HorizontalOrigin.CENTER = 0`、`CornerType.ROUNDED = 0`）。

## 二、通用构造选项（所有图元适用）

- `id` / `name` 可选；`position` 或 `positions` 必填（见 §一-2）。
- 带 `style`（如 `PointStyleOptions`）的类，其 `style` **优先级高于其他样式选项**；schema 见 <https://cimcube.glodon.com/openservice/user-guide/405>。
- `fill`（面类）、`material`（颜色字符串/Cesium 颜色）、`outline`/`outlineColor`/`outlineWidth`（勾边；**`outlineWidth` 单位像素，Windows 上多数不生效**）。
- 视距：`visibleRange:{near,far}`（相机距离可见范围）、`distanceDisplayCondition:[near,far]`（视距显示）、`scaleByDistance`/`translucencyByDistance`/`pixelOffsetScaleByDistance:[near,nearValue,far,farValue]`（视距缩放/透明/偏移）。
- **构造后改样式只能用 `setStyle(xxxStyleOptions)`**（`setOptions` 已废弃，勿写）；临时高亮用 `setHighlight`，它不改基础样式。覆盖清单见 `references/styles.md`。

## 三、常量枚举速查（数值即含义，勿记反）

| 枚举 | 值 |
|---|---|
| `CornerType` 转角 | `ROUNDED 0` 圆角 / `MITERED 1` 尖角 / `BEVELED 2` 平角 |
| `HeightReference` 高度参考 | `NONE 0` 顶点高=椭球高；`CLAMP_TO_GROUND 1` 贴地形+3D Tiles；`RELATIVE_TO_GROUND 2` 相对地形+3D Tiles 的高度；`CLAMP_TO_TERRAIN 3` 仅贴地形；`RELATIVE_TO_TERRAIN 4` 仅相对地形；`CLAMP_TO_3D_TILE 5` 仅贴 3D Tiles；`RELATIVE_TO_3D_TILE 6` 仅相对 3D Tiles |
| `HorizontalOrigin` | `CENTER 0` 中 / `LEFT 1` 左 / `RIGHT 2` 右 |
| `VerticalOrigin` | `CENTER 0` 中 / `BOTTOM 1` 底 / `BASELINE 2` 基线 / `TOP -1` 顶 |
| `LabelStyle` | `FILL 0` 填充 / `OUTLINE 1` 仅描边 / `FILL_AND_OUTLINE 2` 填充+描边 |

```js
heightReference: G.HeightReference.CLAMP_TO_GROUND,   // 1 标牌贴地
horizontalOrigin: G.HorizontalOrigin.CENTER, verticalOrigin: G.VerticalOrigin.BOTTOM,  // 标签在坐标点下方居中
cornerType: G.CornerType.MITERED,                     // 1 尖角
```

## 四、14 个图元路由

| 图元族 | 图元 | 必填 | 写码前必读 |
|---|---|---|---|
| 点与标注 | `PointGraphic` | `position` | `references/point-label.md` |
| 点与标注 | `BillboardGraphic` | `position`、`image` | `references/point-label.md` |
| 点与标注 | `LabelGraphic` | `position`（`text` 默认 `''`） | `references/point-label.md` |
| 线 | `PolylineGraphic` | `positions` | `references/line-graphics.md` |
| 线 | `PolylineVolumeGraphic` | `positions`、`shape` | `references/line-graphics.md` |
| 线 | `CorridorGraphic` | `positions`、`width`(=缓冲半径) | `references/line-graphics.md` |
| 面 | `PolygonGraphic` | `positions` | `references/area-graphics.md` |
| 面 | `RectangleGraphic` | `positions`(对角 2 点)、`height` | `references/area-graphics.md` |
| 面 | `EllipseGraphic` | `position`、`semiMajorAxis`、`semiMinorAxis`、`height` | `references/area-graphics.md` |
| 体 / 空间 | `WallGraphic` | `positions` | `references/solid-graphics.md` |
| 体 / 空间 | `BoxGraphic` | `position`、`dimensions` | `references/solid-graphics.md` |
| 体 / 空间 | `CylinderGraphic` | `position`、`length`、`topRadius`、`bottomRadius` | `references/solid-graphics.md` |
| 体 / 空间 | `EllipsoidGraphic` | `position`、`radii` | `references/solid-graphics.md` |
| 体 / 空间 | `PlaneGraphic` | `position`、`normal`、`dimensions` | `references/solid-graphics.md` |
| 跨域 | 导出真相 / 图元公共能力 / 坐标类 / 与 GraphicLayer 边界 | — | `references/base-contract.md` |
| 跨域 | 样式体系（构造 opts / setStyle / setHighlight） | — | `references/styles.md` |
| 跨域 | 快速上手 + 5 个组合模式 | — | `references/patterns.md` |
| 跨域 | 真实代码提醒 + 误区对照表 | — | `references/pitfalls.md` |

> **核对结论**：上表 14 个图元的必填字段已与 `references/` 逐份交叉核对**全部一致**，唯一调整是 `LabelGraphic`（`text` 有默认 `''`，非硬必填）。4 个跨域文件路径全部存在且一一对应。

## See Also

- **cimcube-layer** — GraphicLayer 的创建/图层参数/`startDraw` 绘制体系与 pick 事件全集；ModelLayer/GLTF 加载替代 ModelGraphic
- **cimcube-global** — `Defines`（`LayerEventType`、`GraphicType`、`LineType`、`ClassificationType` 常量）与坐标转换
- **cimcube-controls / cimcube-utils** — 地图取点后按本页构造图元（量算见 cimcube-utils/references/Measure.md）
- **using-cimcube-skills/references/class-routing.md**（类目唯一真相源视图，自动生成）+ **using-cimcube-skills/references/domains-index.md**（总路由）

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
