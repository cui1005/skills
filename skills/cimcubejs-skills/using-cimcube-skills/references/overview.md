# CIMCube 总览与结构约定

> 本文件对应 SKILL.md 路由表的「CIMCube 是什么、支持哪些格式/分析类/特效体系、结构约定（写某类前先读什么）」行。

> 技能激活与路由回到 `SKILL.md` 速查表与路由表。

## 快速认识 CIMCube

CIMCube 是广联达推出的用于在 Web 浏览器中创建 **3D 地球**与 **2D 地图** 的 JavaScript 渲染引擎，无需插件支持，在 CesiumJS 内核之上进行了深度二次开发，特别增强了：

- 国产三维瓦片格式支持：**S3M**（超图）、**I3S**（Esri）、**GSM**、**USD**
- **24 种空间分析类**：SlopeAnalysis / CutFill / ViewShedAnalysis / VisibilityAnalysis / SkylineAnalysis / Sunlight / FloodAnalysis 等（详见 `references/class-routing.md` 的「§二 逐类归属 → cimcube-analysis」，已与 `classes.list.html` 完全对齐）
- **7 大特效体系，41+ 类**：漫游动画、飞线、粒子、天气、平面扫描、点动态、模型特效
- 国内数据生态：GeoJSON / KML / GML / GLTF / 矢量瓦片 / 实例化图层 / GeoJsonFeatureCollection / FeatureStyleParser
- 所有导出 API 均挂在 **`Glodon`** 全局对象下（完整层级树见 `references/namespace-map.md`）

## 结构约定（写某类代码前必读）

1. Analysis / Animation / Utils / Widgets / Controls / Simulation / Global / Camera / Scene / Realistic-rendering 等域的 `references/` 均为**平级按类（或按族）**组织示例。
2. 顺序固定：**先读该技能 SKILL.md 路由表 → 再打开命中的那一个 references 文件**，勿整包加载。
3. 回答「某模块有哪些真实类 / 该用哪个类」：读 `references/class-routing.md`（类目**唯一真相源视图**，自动生成：域 → 技能 → 类数 + 逐类 FQN 归属 + 例外口径），再按其指引去对应技能的 `SKILL.md` 路由表取可运行示例。
4. 「用户任务 → 该用哪个技能」见 `references/task-matrix.md`；各技能关键 API、类数与坑位见 `references/skill-capabilities.md`。
