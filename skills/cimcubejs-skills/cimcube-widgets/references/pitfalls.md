# Widgets 坑点清单（误用对照 · 性能红线）

> 归属 `cimcube-widgets`。本文件是**查阅型**清单：写代码前扫一遍、写完对照检查。
> 公共初始化与组合模式见 `references/patterns.md`；CSS 引入与样式覆盖见 `references/styles.md`。

## 一、误用对照（Deprecated / 不存在 / 易错）

| 写法 | 真相 |
|---|---|
| `new W.Compass()` | ❌ 未导出；用 `new W.CompassWidget().addToMap(map)`，内部实例不对外暴露，显隐走根 DOM |
| `new W.Geocoder(...)` | ❌ 未导出；用 `new W.GeocoderWidget(geoCoderMode).addToMap(map)` |
| `new W.LayerControl(...)` | ❌ 未导出（内部类）；图层管理面板不是业务控件，唯一开关是初始化参数 `params.layerControl: true` |
| 业务代码里 `new W.LayerControlWidget({ layerManager }).addToMap(map)` | ❌ **重复挂载**——`new Glodon.CIMCube()` 初始化时已自动挂过一个，页面里会出现 **2 个 `.cimcube-widget-container`（两份完整面板）**；正确做法只写 `params.layerControl: true` |
| 以为"看不到面板 = 没挂载" | ❌ 面板默认已挂载，`params.layerControl` 默认 `false` → `_container.style.display = 'none'` |
| 开启面板后立即期望列出已有图层 | ❌ 地形/影像/图元分组初始为空，需点分组头「刷新」重建列表（`_update()` 是空实现） |
| `new W.DraggableDialog(...)` | ❌ 未导出；要对话框能力走 `CameraRoam` / `GroundRoam` |
| `new W.BaseWidget()` | ❌ 未导出；仅内部基类，供理解 `enabled`/`show`/`hide` |
| `new W.MapLegend()` / `new W.MapSplit()` | ❌ 无后缀类是内部实现，未导出；分别经 `MapLegendWidget`（`.addTo(map)`）/ `MapSplitWidget` 使用 |
| `new W.MapLegendWidget({ sources })` + `.addToMap(map)` | ⚠️ `MapLegendWidget` 挂载方法是 **`.addTo(map)`**，不是 `addToMap` |
| `new W.GeocoderWidget()` 不传参 | ⚠️ **必须传** `geoCoderMode`；不传内部走 OFFSET（高德）。`DEF.GeocoderMode.NOOFFSET=0`（天地图）/ `OFFSET=1`（高德） |
| `new W.StatusBar({ degreePrecision: true })` | ⚠️ precision 是 **Number**（默认 6 / 2），文档表误标 Boolean |
| `MapSplitWidget` 传 `layersVisible` | ⚠️ 运行时键是 `viewPortLayersVisible`（`layersVisible` 非运行时键） |
| 重复 `addToMap` / `addTo` 或误把挂载返回值当句柄 | ⚠️ **单例与句柄**：多个控件为**单例**。**重复挂载会抛 `Cannot redefine property`**（SDK 内部 `Object.defineProperty` 写 `map.xxx`），务必防重（挂前判 DOM 已存在或记录已挂载标志），勿重复调用。`addToMap(map)` / `addTo(map)` **一律返回 `undefined`**——**挂前先持实例** `const w = new X(...); w.addToMap(map);` 再用 `w` 作句柄；`MapLegendWidget` 另有访问器 `map.mapLegend`（挂载后非 undefined），但 `map.compass / statusBar / geocoder / distanceLegend` 访问器**均不存在**（句柄用实例 `w` 或根 DOM）。 |
| 用 CSS 改 `DragModel` Gizmo 颜色 | ⚠️ Gizmo 是 3D Primitive，非 DOM；改 options 各轴颜色项 |
| `MapSplitWidget.getMapSplit()` 与 `new U.MapSplit(...)` 同时用 | ⚠️ 同一引擎，**互斥**（同一时刻只允许一个分屏工具在 `begin`） |

## 二、性能红线

| 控件 | 开销 | 建议 |
|---|---|---|
| CompassWidget / DistanceLegend / StatusBar / GeocoderWidget | 纯 DOM | 常驻没问题 |
| StatusBar（取数） | 极低（Service 内部 250ms 节流） | 长时间不关心时 `enabled=false` 卸掉省 CPU |
| GeocoderWidget（输入联想） | 取决于后端 | 业务侧做输入防抖；或改用 `Utils.POISearchService` 自绘搜索框（见 cimcube-utils） |
| CameraRoam / GroundRoam | 中（录制实体 + 回放 PathAnimation） | 录制完关闭对话框（destroy）；切页时清理录制点 entity |
| 图层管理面板（`params.layerControl: true`） | 中（每帧同步图层树，图层多时 diff 重排） | 图层 >100 或频繁增删时注意 `show:false` 隐藏也会继续同步——配合业务节流 |
| MapLegendWidget | 低 | 数据源只读；事件监听器记得 remove（`dataSourceAdded.removeEventListener`） |
| MapSplitWidget | 中高（每帧对左右视口分别渲染） | 对比完成即 `getMapSplit().stop()`；对比层尽量是同一来源不同样式，避免双倍请求 |
| DragModel | 中（Gizmo 3D 图元 + 拾取） | 编辑结束 `deactivate()`/`clear()`；`continueEdit:false` 可避免误编辑下个模型 |

> 浮层 Widget 全部是 DOM + 少量 Cesium entity 叠加，不进渲染瓶颈；面板/卷帘/编辑 Gizmo 才有实质渲染或逐帧开销。
