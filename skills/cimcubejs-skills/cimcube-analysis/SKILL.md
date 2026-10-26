---
name: cimcube-analysis
description: "CIMCube 空间分析 Analysis：①地形土方 坡度坡向、坡度数值、填挖方、剖面采样、地形三角网 TIN、径流、高程热力；②可视规划 视域、通视、视廊、视球/开敞度、天际线、限高合规、最近点、模型拍平、退线退距、CAE 有限元云图、管线断面；③日照环境 日照光影、立面分析/顶视图出图、洪水淹没、水污染扩散。做坡度坡向、土方填挖、剖面采样、径流、高程热力、视域通视视廊、天际线、限高合规、最近设施、拍平、退距、日照光影等空间分析时激活；图层加载属 cimcube-layer、图元绘制属 cimcube-graphic、取点交互属 cimcube-controls、测量量算属 cimcube-utils。"
---

# CIMCube 空间分析（Analysis 24 类 · 单技能路由）

> 命名空间 `Glodon.CIMCube.Analysis.*`
> **定位**：本 SKILL.md 只做**索引路由**。完整 options 表、可运行示例与坑点全部外置于同目录 `references/<类>.md`。**写某个类的完整代码前，先读对应文件**，不要凭记忆重写示例。

## 跨域约定（适用于全部 24 类）

1. 触发因类而异（下表现行标注）：构造自动执行 / `doAnalysis()` / `startByGraphic()·startByTerrain()` / 静态调用。`FINISH` 事件**仅 `CutFill` 定义**；仅 `LimitHeight` 继承 `EventDispatcher` 可 `on()` 监听 `change`。
2. 分析对象 = map 中**已加载的 3D 数据 / 地形高程**（S3M/3DTiles/地形/DEM，先按 **cimcube-layer** 加载）；分析类构造统一挂 map 主实例（`CalculateSlope` 不接收 map、`NearestPointAnalysis` 为静态调用，例外）。
3. 坐标约定：`region`/`boundary`/`positions` 等范围参数多为**经纬度数组（度）**；视点/起终点/观察点等用 **`Cesium.Cartesian3.fromDegrees(lon, lat, height)`**；角度参数为角度制；颜色接受 `String`/`Cesium.Color`。
4. 改参后需再次 `doAnalysis()`/`update()` 才重算；不再使用一律 `clear()/dispose()` 释放。
5. 公共初始化与组合模式见 `references/patterns.md`。
6. **命名约定**：多数分析类以 `XxxAnalysis` 结尾；`CutFill` / `Sunlight` / `LimitHeight` / `BackLine` / `CaeFeCloud` / `ModelFlat` / `Runoff` / `CalculateSlope` 为例外（无 `Analysis` 后缀）；剖面三兄弟（`SectionAnalysis` / `TerrainSectionAnalysis` / `ModelSectionAnalysis`）中**剖切统一走 `SectionAnalysis`**。

## ① 地形与土方（9 类）

| 类 | 用途 | 触发 / 结果要点 | 清理 | 写码前必读 |
|---|---|---|---|---|
| `SlopeAnalysis` | 坡度 + 坡向色带 | 构造自动分析；`showSlope()`/`showAspect()` 切换 | `hide()`/`dispose()` | `references/SlopeAnalysis.md` |
| `CalculateSlope` | 坐标坡度数值（无 map） | `doAnalysis()` 返回结果 | `dispose()` | `references/CalculateSlope.md` |
| `CutFill` | 填挖方量 | 构造后异步：`FINISH` 事件取 getter；`setLevel`/`setBoundary` 后 `update()` | `clearPrimitive()`/`dispose()` | `references/CutFill.md` |
| `SectionAnalysis` | 沿折线采样高程（通用剖面） | `doAnalysis(折线, 间隔m, cb)`，cb 收 `{length,height}[]`；**三兄弟勿混**（下两行仅采样对象不同，均以本文件为准） | `dispose()` | `references/SectionAnalysis.md` |
| `TerrainSectionAnalysis` | 地形剖面变体 | `sampleTerrainLevel` 采样默认地形，无需 layer | `dispose()` | `references/SectionAnalysis.md` |
| `ModelSectionAnalysis` | 模型剖面变体 | 图层作 `doAnalysis(modelLayer,…)` 第 1 参 | `dispose()` | `references/SectionAnalysis.md` |
| `TerrainTINAnalysis` | 地形三角网（单范围） | 构造即按 `boundary` 显示 | `clear()`/`hide()`/`dispose()` | `references/TerrainTINAnalysis.md` |
| `Runoff` | 径流模拟 | `doAnalysis()` 开始；`setOpts()` 改参 | `clearPrimitive()`/`dispose()` | `references/Runoff.md` |
| `ElevationHeatMap` | 地形高程热力图（**≠ `Utils.HeatMap` 数据点热力**） | 构造即计算（`autoCompute` 默认 true） | `hide()`/`dispose()` | `references/ElevationHeatMap.md` |

## ② 可视与规划（11 类）

| 类 | 用途 | 触发 / 结果要点 | 清理 | 写码前必读 |
|---|---|---|---|---|
| `ViewShedAnalysis` | 视域分析 | `doAnalysis()` 场景着色可视/不可视；`drawSketch()` 叠加视锥网格 | `clear()`/`dispose()` | `references/ViewShedAnalysis.md` |
| `VisibilityAnalysis` | 通视分析（两点） | `doAnalysis()` 直接绘制 可视段→遮挡段 折线；`setStartPos/setEndPos` 换点（Cartesian3） | `reset()`/`dispose()` | `references/VisibilityAnalysis.md` |
| `ViewCorridorAnalysis` | 视廊分析 | `doAnalysis()` 绘制两点间可视廊体 | `dispose()` | `references/ViewCorridorAnalysis.md` |
| `ViewDomeAnalysis` | 开敞度（视球） | `doAnalysis()` 半球面着色；`setViewPoint()` 换点 | `dispose()` | `references/ViewDomeAnalysis.md` |
| `SkylineAnalysis` | 天际线（⚠️ 视口级） | `doAnalysis()` 在**当前视角画面**标记；`getSkylineArray()` 取逐列高度 | `clear()`/`dispose()` | `references/SkylineAnalysis.md` |
| `LimitHeight` | 限高核查 | `doAnalysis()` 开启，随瓦片加载增量更新并派发 `change`；`getAnalysisResult()` | `clear()`/`dispose()` | `references/LimitHeight.md` |
| `BackLine` | 建筑退距线 | `doAnalysis()` 绘退距范围面+退距线；`setOpts()`/`setSide()`/`setPositions()` | `clear()`/`dispose()` | `references/BackLine.md` |
| `ModelFlat` | 模型拍平 | `addRegion()` 加拍平区 → `doAnalysis()`；可多区 | `removeRegion()`/`clear()`/`dispose()` | `references/ModelFlat.md` |
| `CaeFeCloud` | 有限元云图 | `loadFromUrl()`/`setData()` 加载 FEA JSON → 热点展示；`flyTo()` | `dispose()` | `references/CaeFeCloud.md` |
| `PipeLineCrossSection` | 管线横断面 | `doAnalysis(cb)` 回调返回断面图（`dataBase64`）与管道对象 | `clear()`/`dispose()` | `references/PipeLineCrossSection.md` |
| `NearestPointAnalysis` | 最近点分析 | **静态** `doAnalysis(target, points, ignoreHeight, count)` → 距离升序数组 | 无（纯函数，免 map） | `references/NearestPointAnalysis.md` |

## ③ 日照与环境（4 类）

| 类 | 用途 | 触发 / 结果要点 | 清理 | 写码前必读 |
|---|---|---|---|---|
| `Sunlight` | 日照光影随时间模拟 | `doAnalysis()`；时间轴 `pause()/stop()/setMultiplier()/setCurrentTime()` | `stop()`/`dispose()` | `references/Sunlight.md` |
| `FacadeAnalysis` | 建筑立面图 / 顶视图 | `doAnalysis(cb)` **异步**回图；`getMergedImage(cb)` 导出；`setMode()` facade/top | `clear()`/`dispose()` | `references/FacadeAnalysis.md` |
| `FloodAnalysis` | 洪水淹没（体块/地形） | **无 `doAnalysis()`**：范围须为挂图层的 `PolygonGraphic`，`startByGraphic()`/`startByTerrain()` 触发 | `stopByGraphic()`/`stopByTerrain()`/`clear()`/`dispose()` | `references/FloodAnalysis.md` |
| `WaterPollutionAnalysis` | 水污染扩散 | `doAnalysis()` | `dispose()` | `references/WaterPollutionAnalysis.md` |

## See Also

- **cimcube-layer** — 分析对象来源：先加载 3D 数据 / 地形 / DEM 再分析（`SectionAnalysis.layer` 等）
- **cimcube-controls** — `TakeCoordinates`：交互画分析范围 / 剖面折线 / 淹没 polygon
- **cimcube-graphic** — `PolygonGraphic` / `GraphicLayer`：构造 FloodAnalysis 等分析范围图元、结果持久化
- **cimcube-setup** — CIMCube 初始化与坐标/姿态约定
- **cimcube-camera** — flyTo / 截图存证（SkylineAnalysis 与当前视角相关）
- **cimcube-utils** — 纯量算（测距/面积，cimcube-utils/references/Measure.md）与日照等场景的交互测量，非 Analysis 域

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
