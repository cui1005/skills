---
name: cimcube-utils
description: "CIMCube 工具集 Utils：①测量量算：坐标/夹角/方位角/距离/面积/坡度/三角测量/体积；②编辑与场景：吸附、剖切族（模型/场景/盒/地形）、区域裁剪、开挖、取盒、缓冲、等高线；③服务·转换·对比：卷帘分屏、坐标转换、视线里程、热力图、模型表面折线批注、瓦片克隆、凸包破碎，以及状态栏/距离比例尺/图例/指南针/POI/性能监控的数据层。测距测面积、体积、剖切裁剪开挖、等高线、坐标转换、热力图时激活；UI 组件本身属 cimcube-widgets，交互取点属 cimcube-controls、图层加载属 cimcube-layer、空间分析属 cimcube-analysis、图元绘制属 cimcube-graphic。"
---

# CIMCube 工具集（Utils 29 类 · 单技能路由）

> **基线**：命名空间 `Glodon.CIMCube.Utils.*`，枚举在 `Glodon.CIMCube.Global.Defines`。
> **定位**：本 SKILL.md 只做**索引路由**。完整 options 表、可运行示例与坑点全部外置于同目录 `references/<功能族>.md`。**写某个类的完整代码前，先读对应文件**，不要凭记忆重写示例。

## 跨域约定（适用于全部 29 类）

1. **构造签名分三类**：交互/场景型 `new X(map[, opts])`（Measure、裁剪开挖、分屏、HeatMap、批注、各 Service、Cloner）；纯静态类**不要 new**——`Utils.CoordinateTransform` / `Utils.SightLimitCalculate` / `Utils.BufferTool` 直接调静态方法；`Utils.POISearchService` **只收 options（不含 map）**。
2. **控制方法按族记忆**（勿跨族猜方法）：
   - `Measure`：`activate()/deactivate()/completion()/undo()/clearAll()/restore()`（三角测量模式另有 `get/delete/show/hide/setTriangulationType/setStyle`；体积模式 `setLayer/setDatum/setVolume`）。
   - 剖切族 `BoxClip/SceneClip/ModelClip`：`activate()/deactivate()/clip()/clear()/dispose()`；`ModelCut/ModelExcavation`：`activate()/addRegion/doAnalysis()/clear()`。
   - 分屏 `MapSplit/MultiViewPort`：`begin()/stop()`（**无 activate**）；服务类 `XxxService`：`start()/stop()`（**无 play**）；`ModelMarkup`：`activate(DrawEditMode.DRAW/EDIT)`（两态互斥）。
3. **坐标/单位**：点位用 `{lon, lat, height}`（度/度/米，如开挖 `positions` 需逆时针）；`BoxClip.clipEx`/`ModelClip.clip` 的 `posA/posB` 为 `Cartesian3`；`BufferTool` 输入输出为 `Cartesian`(Cartographic)，距离单位**米**。
4. **释放**：SPA 路由切换或演示结束一律 `dispose()`（分屏先 `stop()`、剖切/开挖/取盒先 `clear()/deactivate()`），不要只 `deactivate()/hide()`，否则残留剖切面/材质/相机输入接管。
5. 公共初始化与组合模式见 `references/patterns.md`；文档口径坑 / 废弃类 / 臆造 API / 性能红线见 `references/pitfalls.md`。
6. **命名与边界约定**：Utils 里纯逻辑 / 非 UI 的类一律以 `XxxService` / `XxxTool` / `XxxUI` / `XxxCalculation` 结尾（UI 组件本体归 cimcube-widgets）；**Utils = 通用工具**（无专用业务结果协议），**Analysis = 业务级空间分析**（结果 / 完成回调**因类而异**，无统一 `FINISH`），两者不要混用。

## ① 测量与体积（Measure 族 4 类）

| 类 | 用途 | 控制方法要点 | 写码前必读 |
|---|---|---|---|
| `Utils.Measure` | 交互量测（**13 种 MeasureMode 枚举** + MapActionType 事件） | `activate()`（防重入）/`deactivate()`/`completion()`/`setPosition`/`updatePosition`/`undo`/`clearAll`/`restore`；只读 `isActive`/`measureMode` | `references/Measure.md` |
| `Utils.MeasureUtils.MeasureFormula` | 测量公式库（**纯静态**） | `calculateDistance/calculateArea(geometry, measureMode, map?)`；mode 严格校验，贴地类需 `map` | `references/MeasureUtils.md` |
| `Utils.MeasureUtils.MeasurementUtils` | 量测辅助（**纯静态**） | `pickPositionByMousePosition`（修负高程）+ `length/areaUnitConversion`、`getLength/AreaUnitString` | `references/MeasureUtils.md` |
| `Utils.VolumeCalculation` | 3DTiles 瓦片级体积测算 | `do()`/`getVolume(featureId?)`/`setLayer`/`dispose()`；事件字符串 `"change"`（**无 calculate()**） | `references/VolumeCalculation.md` |

## ② 编辑与场景工具（Edit 族 11 类）

| 类 | 用途 | 控制方法要点 | 写码前必读 |
|---|---|---|---|
| `Utils.Snap` | 吸附开关（接管 `map.controls` 左键取点） | `enable()`/`disable()`/`clear()`/`dispose()`；事件 `'mouseSnapClickup'`；属性 setter 即时生效 | `references/Snap.md` |
| `Utils.ClipBase` | 剖切基类（父类） | `on/off/dispose`；静态 `EVT_POS_CONFIRMED='poscomfirm'`/`EVT_READY_TO_CLIP='rtclip'` | `references/Clip.md` |
| `Utils.BoxClip` | 盒体剖切（单层 / 全层） | `clip(target)`（LAYER）/`clipEx(posA,posB,height,offset?,targets?)`（OVERALL）；`pickOn/pickOff`；事件 `BoxClipEvent` | `references/Clip.md` |
| `Utils.SceneClip` | 场景剖切（可含地形+开挖） | `activate()`/`deactivate()`/`clip(posA,posB,mode?)`/`clear()`/`dispose()`；opts `includeTerrain`+`excavate` | `references/Clip.md` |
| `Utils.ModelClip` | 单体模型剖切 | `activate()`（绘剖切线）/`clip(posA,posB,mode?,target?)`/`clear()`/`dispose()` | `references/Clip.md` |
| `Utils.TerrainClip` | 地形开挖基础件（⚠️ 无独立 APIDOC 页，进阶） | `addRegion/removeRegion/doAnalysis()/getResult()/setDepth/clear()` | `references/Clip.md` |
| `Utils.ModelCut` | 区域裁剪显示（多边形 INNER/OUTER） | `activate()`/`deactivate()`/`addRegion`/`removeRegion`/`dispose()` | `references/Cut.md` |
| `Utils.ModelExcavation` | 模型开挖（多边形+深度挖洞） | `doAnalysis()`/`clear()`/`dispose()`；便捷 `setPositions/setDepth/setBottomTexture/setFacadeTexture` | `references/Cut.md` |
| `Utils.BoxDragger` | 三击取盒（⚠️ APIDOC 页名 `BoxDraggerUI`，运行时导出名 `BoxDragger`） | `start(finishCallback, offset?)`/`dispose()`；回调 `posAInWorld/posBInWorld/height` | `references/BoxDragger.md` |
| `Utils.BufferTool` | 纯静态缓冲几何（**无 map**） | 静态 `generatePoint/Polyline/PolygonBuffer(points, distance)`；输入 `Cartographic` 数组、距离米 | `references/BufferTool.md` |
| `Utils.ContourLine` | 地形等高线可视化（材质层 + canvas 回调） | `show()`/`hide()`；`addChangeListener(cb)`（`[{left,top,height}]`）/`removeChangeListener` | `references/ContourLine.md` |

## ③ 服务 · 转换 · 对比（Services 族 14 类）

| 类 | 用途 | 控制方法要点 | 写码前必读 |
|---|---|---|---|
| `Utils.MapSplit` | 卷帘分屏对比（左右/上下） | `new X(map, layerManager, opts)` + `begin()/stop()/dispose()`/`setSplitPosition`/`getSplitPosition`/`setSplitLayersVisible` | `references/MapSplit.md` |
| `Utils.MultiViewPort` | 多视口分屏比对 | 同 `MapSplit`（视口裁剪，单实例互斥） | `references/MapSplit.md` |
| `Utils.CoordinateTransform` | EPSG 坐标转换（**纯静态**） | `epsgTranslate(coords, fromEPSG, toEPSG)`（**无实例转换/DMS**） | `references/CoordinateTransform.md` |
| `Utils.SightLimitCalculate` | 视线里程静态计算（**纯静态**） | `calculate(start, end, pointOnLine)`（`Graphic.GeoPoint` 入参；共线失败抛错） | `references/CoordinateTransform.md` |
| `Utils.HeatMap` | 2D/3D 数据热力 | `setData([{lon,lat,value}])` 即渲染（**无 render/refresh**）；成员 `show/radius/blur/gradient/clampToGround` | `references/HeatMap.md` |
| `Utils.ModelMarkup` | 模型表面折线批注（绘制/编辑两态） | `activate(DrawEditMode.DRAW↔EDIT)`/`selectItem`/`getSelectedItem`/`deactivate`/`clearAll`/`dispose`；事件 `MapActionType` | `references/ModelMarkup.md` |
| `Utils.CompassService` | 罗盘数据服务（`cameraHeading`） | `start()/stop()`；`addEventListener({widget,method})`（**非 on("HEADING_CHANGE")**）；`faceNorth/rotateHeading` | `references/Services.md` |
| `Utils.StatusBarService` | 状态栏数据服务 | `start()/stop()`；回调参数按序：经纬高 → 方位俯仰 → fps → ms | `references/Services.md` |
| `Utils.DistanceLegendService` | 像素↔距离换算服务 | `start()/stop()`（**无回调**，UI 侧读换算结果） | `references/Services.md` |
| `Utils.PerformanceListening` | 性能监听（**全局唯一**） | `start()/stop()`；成员 `fps/ms/drawCalls/numberOfVertices`（**无 UPDATE/WARN 事件**） | `references/Services.md` |
| `Utils.POISearchService` | 地名/POI 查询（**只收 options，不含 map**） | `search/geocode/geocoding/reverseGeocoding`；`geoCoderMode` 天地图 NOOFFSET / 高德 OFFSET + token | `references/Services.md` |
| `Utils.MapLegendService` | 图例图标资源服务（无参） | `getUrlResource(symbol)`/`getSvgResource(style)` → `Promise<资源>`（**非图例 JSON**） | `references/Services.md` |
| `Utils.SpatialTilesetCloner` | 3DTiles 范围克隆抬升 | `activate()`（复制→裁剪→渐变→降 LoD→连线）/`deactivate()`/`dispose()`；成员 `region`/`simplifiedRegion` | `references/SpatialTilesetCloner.md` |
| `Utils.ConvexObjectBreaker` | 凸包破碎（物理） | `new X(minSizeForBreak?, smallDelta?)`；`prepareBreakableObject`/`cutByPlane`/`subdivideByImpact` | `references/SpatialTilesetCloner.md` |

## See Also

- **cimcube-controls** — `TakeCoordinates`/`map.controls`：所有取点交互的底层；`Utils.Snap` 接管的就是它的左键
- **cimcube-global** — `Defines` 的 Utils 相关枚举全表（MeasureMode/MeasureShowResult/DatumMode/LengthUnit/AreaUnit/TriangulationType/MapActionType/ModelClipMode/ModelCutMode/SnapType/SnapTargetType/ViewPortModeType/GeoCoderMode/DrawEditMode）
- **cimcube-layer** — 裁剪/克隆目标 = 图层（`Layer`/`Tiles3DLayer` 等 `setClippingPlanes` 能力）；`layerManager` 是分屏/克隆的入参
- **cimcube-graphic** — 缓冲环/开挖范围/等高线点/量测结果上屏（`Graphic.GeoPoint` 是 `SightLimitCalculate` 入参）
- **cimcube-analysis** — `CutFill`（填挖量）区别于 `VolumeCalculation`（几何体积）；`ElevationHeatMap`（地形高程热力）区别于 `HeatMap`（数据热力）；`TerrainSectionAnalysis` 替代不存在的 `ElevationProfile`
- **cimcube-widgets** — `Compass`/`StatusBar`/`DistanceLegend`/`Geocoder`/`MapLegendWidget`/`MapSplitWidget` 是这些 Service 的现成 UI 消费方（UI 示例见 cimcube-widgets/references/StatusBar.md、cimcube-widgets/references/DistanceLegend.md、cimcube-widgets/references/Geocoder.md、cimcube-widgets/references/MapLegend.md、cimcube-widgets/references/MapSplit.md）

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
