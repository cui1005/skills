---
name: cimcube-widgets
description: "CIMCube UI 组件 Widget：①浮层导航与状态 — 罗盘/指南针/指北针（拖拽旋转俯仰/双击回正北）、地名搜索框、底部状态栏（经纬度/高程/方位角/俯仰角/FPS）、距离比例尺、路径漫游录制对话框；②面板与交互 — 图例面板（点击联动要素显隐）、双屏卷帘对比、模型三维拖拽编辑（平移/旋转/缩放）。用户要加以上浮层/面板类 UI 或自绘状态类 UI 时激活（Widget 后缀类与同名实例成对）；图层控制面板只由初始化参数 layerControl 开启（见 cimcube-setup），不是本技能组件；数据来源属 cimcube-utils，图层数据属 cimcube-layer、空间分析属 cimcube-analysis、相机路径动画属 cimcube-animation。"
---

# CIMCube UI 组件（Widgets 11 导出类 + 6 内部类 · 单技能路由）

> **基线**：命名空间 `Glodon.CIMCube.Widgets.*`，枚举取 `Glodon.CIMCube.Global.Defines`。
> **定位**：本 SKILL.md 只做**索引路由**。完整 options 表、DOM 锚点、可运行示例与坑点全部外置于同目录 `references/<控件>.md`。**写某个控件的完整代码前，先读对应文件**，不要凭记忆重写示例。

```js
const W   = Glodon.CIMCube.Widgets;
const DEF = Glodon.CIMCube.Global.Defines;
const map = new Glodon.CIMCube({
  id: "cimcubeContainer",
  gisResourceUrl: "./dist/",
  // ★ 「图层管理」面板只是一个初始化参数：写入即显示（默认 false = 已挂载但隐藏）
  //   面板由 CIMCube 自动挂载，业务不要 new LayerControlWidget，也不要去取句柄
  params: { layerControl: true },
});
```

## 跨域约定（适用于全部 Widgets）

1. **导出现实（已定论）**：`Glodon.CIMCube.Widgets` 全局**只导出 11 个类**——下表 ①② 组的 **9 个是可挂载组件**，另 `LayerControlWidget` 只由初始化参数开启、**不是业务组件**（见 ④ 节）。另有 5 个类（`BaseWidget / Compass / Geocoder / LayerControl / DraggableDialog`）有 API_DOC 页、包内也有实现，但**未挂到命名空间**——`new W.xxx` 得 `undefined`。详见 ③ 组与 `references/BaseWidget.md`。
2. **挂载签名分三类**（勿按基类猜）：★ **只有 `LayerControlWidget` 由 `params.layerControl` 自动挂载**；其余 10 类（含 4 个浮层 Compass/Geocoder/StatusBar/DistanceLegend）SDK **都不会自动挂**，必须业务显式调用挂载方法。Widget 类 `new X().addToMap(map)`（**`MapLegendWidget` 例外为 `.addTo(map)`**；`LayerControlWidget` **不挂载**，见 ④ 节）；`DragModel` 无挂载——`new DragModel(map[, opts])` 后 `.activate()`；`CameraRoam/GroundRoam` 为位置参数构造后 `.showDialog()`。
3. **单例与句柄**：多个控件为单例，挂载/句柄的误用与正确写法（重复挂载抛 `Cannot redefine property`、`addToMap/addTo` 返回 `undefined`、`map.xxx` 访问器不存在、句柄挂前持实例、防重）见 **references/pitfalls.md**「单例与句柄」条；逐控件写法见对应 `references/<控件>.md`。
4. **生命周期**：`extends BaseWidget` 的类（`StatusBar / DistanceLegend` + 内部 `Compass`）有 `enabled`（挂/卸 DOM，真正停用）与 `show()/hide()`（仅切 `visibility`，不省服务开销）；`CompassWidget / GeocoderWidget` 是 mixin 包装，停用靠内部实例或直接移除根元素。
5. **样式**：只引 `dist/Resource/css/Widgets.css` 一个就够（它是 `Resource/css/` 下 13 个分册的聚合超集，分册无需再引）；覆盖一律放自有样式表并置于 SDK CSS 之后，勿改 SDK 文件（完整引入清单、CSS 文件与 DOM 锚点对照见 `references/styles.md`）。
6. 公共初始化与组合模式见 `references/patterns.md`；误用对照与性能红线见 `references/pitfalls.md`。

## ① 浮层导航与状态（overlay 6 类）

| 类 | 用途 | 挂载 / 控制要点 | 写码前必读 |
|---|---|---|---|
| `CompassWidget` | 罗盘（拖外圈旋转 / 拖内圈俯仰 / 双击回正北 / +/- 缩放） | `new CompassWidget().addToMap(map)`（**无 options**）；mixin 包装无返回值，显隐/移除走根 DOM `.cimcube-compass` | `references/Compass.md` |
| `GeocoderWidget` | 地名搜索框（左上，自带输入 + 结果下拉） | `new GeocoderWidget(geoCoderMode).addToMap(map)`；**必传** `DEF.GeocoderMode.NOOFFSET`(天地图0) \| `OFFSET`(高德1) | `references/Geocoder.md` |
| `StatusBar` | 底部状态栏（纬度N/S·经度E/W·高程·方位角·俯仰角·FPS/ms） | `const sb = new StatusBar([options]); sb.addToMap(map)`（**返回 `undefined`**，句柄即 `sb` 本身或根 DOM `.cimcube-status-bar`；`map.statusBar` 访问器不存在）；extends BaseWidget；单例 | `references/StatusBar.md` |
| `DistanceLegend` | 距离比例尺（随相机自动换算，≥1000 自动 km） | `const dl = new DistanceLegend(); dl.addToMap(map)`（**返回 `undefined`**，句柄即 `dl` 本身或根 DOM `.cimcube-distance-legend`；`map.distanceLegend` 访问器不存在）；`updateContent({barWidth, distance})`；单例 | `references/DistanceLegend.md` |
| `CameraRoam` | 路径漫游录制对话框（路径点表 + 左键拾点 + 回放） | `new CameraRoam(container, dialogId, dialogTitle, map, map.camera).showDialog()` | `references/CameraRoam.md` |
| `GroundRoam` | 贴地路径漫游录制对话框 | 构造/继承同 `CameraRoam`；贴地拾点默认抬 10m | `references/CameraRoam.md` |

## ② 面板与交互（panels 3 类）

| 类 | 用途 | 挂载 / 控制要点 | 写码前必读 |
|---|---|---|---|
| `MapLegendWidget` | 图例面板（点击联动整层/单值要素显隐） | `new MapLegendWidget(options).addTo(map)`（**注意 addTo**）；`options.sources` 必填；内部 → `map.mapLegend` | `references/MapLegend.md` |
| `MapSplitWidget` | 双屏卷帘对比（左右/上下 + 拖动分隔条） | `new MapSplitWidget(layerManager, options).addToMap(map)`；`getMapSplit()` 拿底层实例 | `references/MapSplit.md` |
| `DragModel` | 模型三维拖拽编辑 Gizmo（平移/旋转/缩放） | `new DragModel(map[, options])` → `.activate()`；结束 `deactivate()`/`clear()`/`dispose()` | `references/DragModel.md` |

## ③ 内部类与文档遗留（⚠️ 不可 `new`）

| 类 | 状态 | 真实可达途径 | 写码前必读 |
|---|---|---|---|
| `BaseWidget` | 内部基类，未导出 | 供理解 `StatusBar/DistanceLegend/Compass` 的 `enabled`/`show`/`hide` 语义 | `references/BaseWidget.md` |
| `Compass` | 未导出（extends BaseWidget：`enabled` / `show()` / `hide()`） | 经 `CompassWidget` 挂载，内部实例**无对外访问器** | `references/Compass.md` |
| `Geocoder` | 未导出（内部类） | 仅 `GeocoderWidget` 内部实例化（`.cimcube-geocoderContainer`），**无访问器** | `references/BaseWidget.md` |
| `DraggableDialog` | 未导出（内部基类） | `CameraRoam`/`GroundRoam` 的父类，无访问器 | `references/BaseWidget.md` |
| `MapLegend` / `MapSplit`（无后缀实现类） | 未导出 | `MapLegendWidget` / `MapSplitWidget` 的内部实现；`MapSplit` 引擎用法见 `cimcube-utils/references/MapSplit.md` | `references/MapSplit.md` |

## ④ 图层管理面板（只由初始化参数开启，**不是本技能组件**）

```js
// ✅ 唯一正确用法：初始化参数
new Glodon.CIMCube({ container: "cimcubeContainer", params: { layerControl: true } });
// ❌ 业务不要 new W.LayerControlWidget(...)，也不要取句柄
```

- `LayerControlWidget` 虽是导出的 11 类之一，但 CIMCube 构造时**已自动挂载**（默认 `false` = 已挂载但 `display:none`）；再 new 会出现**两个叠层面板**。
- 改图层显隐/透明度/层级请直接操作 `map.layerManager`（见 **cimcube-layer**）；面板样式与 DOM 锚点见 `references/styles.md`（外层 `.cimcube-widget-container`）。
- 内部类 `LayerControl`（未导出）是它的内部实现，**不可 new、无对外访问器**。
- 参数细节见 `cimcube-setup/references/constructor-options.md`。

## See Also

- **cimcube-utils** — `CompassService` / `StatusBarService` / `DistanceLegendService` / `POISearchService` / `MapLegendService`：本技能多数控件的**数据来源**；层卷帘底层 `Utils.MapSplit` 见 `cimcube-utils/references/Services.md`、`cimcube-utils/references/MapSplit.md`
- **cimcube-animation** — `Animation.Effect.CameraRoaming` 及 `PathAnimation`：CameraRoam/GroundRoam 录制回放的底层相机路径动画
- **cimcube-layer** — 各图层类型与 `layer.style` 渲染键：拼 `MapLegendWidget` 数据源、`DragModel` 编辑目标的源头
- **cimcube-setup** — 创建 CIMCube 实例（`map.scene / map.camera / map.globe / map.layerManager` 等核心子系统）；图层管理面板开关 `params.layerControl` 见 `cimcube-setup/references/constructor-options.md`
- **cimcube-controls** — 自绘"取点"交互时 `map.controls.takeCoordinates`（录制对话框内部用的拾取机制）

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
