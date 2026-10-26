# Utils 坑点清单（文档口径 · 废弃类 · 臆造 API · 性能红线）

> 归属 `cimcube-utils`。**查阅型**清单：写前扫一遍、写完对照。公共初始化与组合模式见 `references/patterns.md`。

## 一、文档口径坑（先记这两条）

1. **`BoxDraggerUI` ≠ 运行时可 `new` 的名字**：运行时 `Glodon.CIMCube.Utils` 只有 `BoxDragger`（注释名 vs 导出名不一致），一律 `new U.BoxDragger`。
2. **常量以 `Defines` 枚举对象为准**（`D.ModelClipMode`/`D.ModelCutMode`/`D.SnapType`/`D.MeasureMode`/`D.MapActionType`/`D.GeoCoderMode`/`D.DrawEditMode` 等），不要写死字符串。

## 二、Deprecated — 不存在的类（勿用）

以下名称在 API_DOC 与运行时均已核对**不存在**：

| 不存在的写法 | SDK 真实替代方案 |
|---|---|
| `ElevationProfile` 沿线取高程画纵断面（2D `{d,h}`） | `Analysis.TerrainSectionAnalysis`（cimcube-analysis）：折线切割 → FINISH 的 `[{d,h}]` 点集，自行画图 |
| `RoutePlan` 路径规划 / 路网 A* | SDK 无通用路径规划类；自行接路网+最短路算法或第三方导航 |
| `Analysis.Distance / Area / Volume` | 交互用 `Utils.Measure`；程序化用 `MeasureUtils.MeasureFormula` / `Utils.VolumeCalculation`；土方量用 `Analysis.CutFill` |
| `MapLegend`（图例 JSON 数据） / `SkylineQuery` / `RoutePlan` | SDK 均无；图例图标资源用 `Utils.MapLegendService` 生成后自行渲染 |
| `UnitConvert` / `ScreenToWorld` / `Measure.calculate/measure`（臆造实例方法） | 单位换算用 `Utils.MeasureUtils.MeasurementUtils`；屏幕↔世界用 `map.controls`/`map.scene.pickPosition`；测量用 `Functions.calculateXxx` |
| `TerrainClip` 独立 APIDOC 页 | 无独立页（非 APIDOC 公开项）；优先用 `SceneClip({ includeTerrain:true, excavate })` |
| `RoadLine`（道路红线**绘制**工具） | SDK 无此绘制工具；`Animation.LineEffect.RoadLine` 只是「道路流光」**特效**。画红线用 `Graphic.Polyline`（+ `Controls.TakeCoordinates` 取点、`Utils.Snap` 吸附） |

### 臆造 API → 真实 API（编辑族）

| 臆造写法 | 真实 |
|---|---|
| `new U.Snap(map, { targetLayers, mode:'vertex+edge+grid', pixelTolerance, gridSize })` | `new U.Snap(map, { targetType, typeList, tolerance, screenCaptureFrameRate, ...颜色 })`；白名单用 `addWithoutSnapObject/removeWithoutSnapObject` |
| `snap.pick(x, y)` 手动命中 / `snap.on('SNAPPED')` | 不存在。吸附自动接管 `map.controls` 左键取点；事件是 `'mouseSnapClickup'`（在 Snap 实例上），结果在 `map.controls.snapSelectedResults` |
| `new U.BoxClip(map, { targetLayers, box:{min,max}, invert })` + `boxClip.apply()` | `clip(target)` LAYER 单图层 / `clipEx(posA,posB,height,offset?,targets?)` OVERALL 全局；无 invert（裁剪方向由模式决定，见 ModelCutMode） |
| `new U.ModelClip(map, { targetLayers, box })` | `new U.ModelClip(map, { clipMode, layer })`，`activate()` 绘线或 `clip(posA,posB)` |
| `new U.BoxDraggerUI(map)` | 导出名为 `U.BoxDragger`；`start(cb, offset)` 三击（2 角 + 高度）回调，而非"拖手柄调已存在的盒" |
| `new U.ModelCut(map, { target, plane:{point, normal} })` / "沿平面切成两半" | 区域裁剪 `{ cutMode, region([lon,lat,height]多边形), layers }`，`activate/addRegion/removeRegion`；无 plane 语义 |
| `new U.BufferTool(map, { target, distance })`、`buf.intersects()` | 静态 `U.BufferTool.generateXxxBuffer`，输入 Cartographic 数组；没有实例方法/图层目标/相交查询 |
| `new U.ContourLine(map, { target: demLayer, interval, levels })` → 输出折线要素 | `{ spacing, lineWidth, lineColor, region, gridSpacingX/Y }`；回调数组 `{left,top,height}` 供自绘 |
| `new U.ModelExcavation(map, { polygon })` | `positions`（逆时针 `{lon,lat,height}`）+ `depth` + 贴图 URL，`doAnalysis()` 触发 |
| `Utils.ClipBase` 上的模式增删 / `modelClipMode` | 剖切模式常量在 `D.ModelClipMode`；ClipBase 只给 on/off/dispose + `EVT_POS_CONFIRMED`('poscomfirm') / `EVT_READY_TO_CLIP`('rtclip') |
| `DrawEdit` / `Section` / 盒 `invert` 思路 | SDK 无这些；交互取点画图用 cimcube-controls 的 TakeCoordinates；剖切后图形化导出可基于剪平面自实现 |

### 臆造 API → 真实 API（服务族）

| 臆造写法 | 真实 |
|---|---|
| `CoordinateTransform` 实例转换 / DMS 格式化 | 仅静态 `epsgTranslate(coords, fromEPSG, toEPSG)` |
| `SightLimitCalculate` 返回"遮挡百分比" | `calculate(start,end,onLine)` 返回沿线直线+水平距离；点不在线抛 `Cesium.RuntimeError` |
| `HeatMap.render()/refresh()/start()/stop()` | 无；`setData([{lon,lat,value}])` 即刷新 |
| `PerformanceListening.on('UPDATE'/'WARN')` | 无事件；`start()` 后读成员 `fps/ms/drawCalls/numberOfVertices` |
| `CompassService.on('HEADING_CHANGE')` | `addEventListener({widget, method})` + 读 `cameraHeading` |
| `new U.POISearchService(map, token)` | `new U.POISearchService({ geoCoderMode, accessToken })`（**不含 map**） |
| `MapSplit.activate()/position/滑块` | `new X(map, layerManager, opts)` + `begin()/stop()` + `setSplitPosition(px)` |

## 三、性能红线（汇总）

| 工具 | 开销 | 建议 |
|---|---|---|
| Measure | 低~中（挂屏幕事件 + 实时标注） | 一次只激活 1 个；用完 `dispose()`（释放会清标注并退深度限制） |
| VolumeCalculation | 中（随瓦片加载统计） | 图层关掉/换层需 `setLayer`；不显示用 `dispose` |
| MeasureFormula / MeasurementUtils | 极低（纯函数） | 可高频同步调用，重复输入参数可缓存 |
| MapSplit / MultiViewPort | 中~高（区域重绘） | begin 态互斥，一次一个；完成即 `stop()`；影像层优先 |
| HeatMap | 中（点→格点） | 点集先聚合；`grid` 控制颗粒度；用完 dispose |
| SpatialTilesetCloner | 高（瓦片复制） | region 尽量小；`lodFactor` 抬高；动画完无需连线则关 |
| ModelMarkup | 低~中 | 一条 = 多点折线；批注多时监听 DELETE 及时同步业务 |
| PerformanceListening | 极低 | 全局唯一；开发态常开、生产关闭 |
| CoordinateTransform / SightLimitCalculate | 极低（纯函数） | 可高频调用；EPSG 对子相同可缓存 |
| CompassService / StatusBarService / DistanceLegendService | 极低 | `start/stop` 严格配对，结束 `dispose` |
| POISearchService | 低（网络 IO） | token 与服务模式匹配；结果自己控制频率 |
| ConvexObjectBreaker | 中（几何切割） | 迭代次数受控；碎块数量别失控 |

> **三戒**：① 同一时刻只激活 1 个交互工具（量测/剖切/分屏 begin 态互斥）；② 交互结束立即 `dispose`（分屏先 `stop()`，剖切/开挖/取盒先 `clear()/deactivate()`），否则残留剖切面/材质/相机输入接管；③ 静态类别 `new`；Measure 换算先确认输入基准（长度米/面积平方米），POISearchService 的 token 须与服务模式匹配。
