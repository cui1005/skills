---
name: using-cimcube-skills
description: "CIMCube Skills 使用指南。当用户询问如何使用这套 CIMCube 技能、CIMCube 模块快速查询、SDK 加载方式、命名空间前缀，或需要帮助、获取 CIMCube Skills 的总览目录时激活。"
---

# CIMCube Agent Skills 使用指南

> **SKILL.md 是纯路由索引**：本页**只**保留「16 技能激活词速查表 + 路由表」。
> 总览介绍、命名空间树、SDK 加载、语言约定、能力详情、任务矩阵全部下沉到 `references/`。

## 前置准备

- **浏览器预览（可选但推荐）**：三维地球需跑在浏览器才能看到真实渲染。建议注册 **Chrome DevTools MCP**

## 全局约定

所有 API 用完整前缀 Glodon.CIMCube.*，不要 import 解构。

## 路由表（按需读取，勿整包加载）

| 你要查什么 | 读这个文件 |
|---|---|
| CIMCube 是什么、支持哪些格式/分析类/特效体系、结构约定（写某类前先读什么） | `references/overview.md` |
| `Glodon.CIMCube.*` 命名空间层级树、各子系统归属与类数 | `references/namespace-map.md` |
| **「某模块有哪些真实类 / 该类归哪个技能 / 有没有 API_DOC 页」** | `references/class-routing.md`（**类目唯一真相源视图**，自动生成物，勿手改） |
| 静态 `<script>` 引入、`CIMCubeSDKLoader` CDN 按版本加载、资源路径与容器 DOM | `references/sdk-loading.md` |
| 中文交互、`Glodon.CIMCube.*` 全前缀写法、单位约定（度/米/弧度） | `references/conventions.md` |
| 命名空间 → 技能 → 域入口文件 的总路由 | `references/domains-index.md` |
| 16 个技能各自的**关键 API、类数与已知坑位** | `references/skill-capabilities.md` |
| 「用户任务 → 该用哪个技能」的 14 行选择矩阵 | `references/task-matrix.md` |

## Domain Skills 速查表（16 个）

当你在对话中提到下表的**激活关键词**，Agent 会自动引用对应的 Skill 文档给出准确示例；各技能的能力与坑位见 `references/skill-capabilities.md`。

| # | Skill 名称 | 激活关键词示例 |
|---|---|---|
| 1 | **cimcube-setup** | 初始化 CIMCube、创建实例、CIMCubeSDKLoader、资源路径、构造参数 |
| 2 | **cimcube-scene** | 场景环境：大气/天空盒（GroundSkyBox 17 预置）/反射盒、雾、阴影、后处理与色彩、画质调试、环境光遮蔽 AO、拾取坐标、globalSettings |
| 3 | **cimcube-realistic-rendering** | 真实感渲染：`enableRealisticRendering` 总开关 + 8 子模块（bloom/skyLight/skyAtmosphere/exponentialHeightFog/volumetricClouds/eyeAdaptation/lightShaft/origin）、光源平行光/点光/聚光、UE4 管线、辉光、天空光、大气散射、高度雾、体积云、自动曝光、体积光、点光源、聚光灯 |
| 4 | **cimcube-physics** | 场景物理引擎 ScenePhysics：刚体创建、模型图层碰撞、速度/约束/接触回调、CCD、碰撞检测、重力、铰链、软体 |
| 5 | **cimcube-camera** | 相机、相机控制、视角、飞行、flyTo、飞到、定位、setView、flyToBoundingSphere、包围球、lookAt、截图、快照、getCaptureImg、getCameraInfo、自转、startRotate、相机事件、pickEllipsoid、fov、heading |
| 6 | **cimcube-globe** | Globe、地球、椭球配置 |
| 7 | **cimcube-layer** | 图层、LayerManager、Tiles3D、S3M、I3S、地形、影像、GeoJSON、KML、GLTF、USD、FeatureStyleParser、GeoJsonFeatureCollection、GML、MVT、实例化图层、点聚合 |
| 8 | **cimcube-graphic** | 图元、Graphic、矢量标注、点、线、面、体、广告牌 Billboard、Label、Polygon、Polyline、椭圆 EllipseGraphic、平面板 PlaneGraphic、GeoPoint、TwoDimensionPoint、CornerType、HeightReference |
| 9 | **cimcube-animation** | 漫游、导览、第一人称、第三人称、轨迹漫游、billboard 漫游、摄像机扫描光束、视频投射、管道流动、立体锚点、动态墙、雷达、环境光遮蔽、迁徙特效、飞线、动态线、道路流光、圆柱线、轨迹线、波纹圆面、扩散墙、动态面、扩散旋转特效、序列贴图面、静态面、圆环特效、模型炸开、单体要素动画、白模贴图、粒子、火焰、烟雾、喷泉、下雨、下雪、起雾、动态点轨迹、闪烁点、散点扩散、实例化看板、CameraRoaming、FirstPersonRoaming、ThirdPersonRoaming、TrackRoaming、BillboardRoaming、CameraShooting、VideoEntity、PolylineVolumeFlow、PrismPoint、Wall、Radar、AmbientOcclusion、Metaptosis、FlyLines、DynamicLines、RoadLine、RopeLines、TrailLines、PlaneEffect.Radar、RingScan、DiffuseWall、Moire、Sequence、Statics、StaticsRingScan、ModelExplode、ModelInteroperation、UVTextureEffect、Fire、Smoke、Water、Rain、Snow、Fog、Dynamic、Twinkle、Scatter、InstanceBillboard、模型爆炸 |
| 10 | **cimcube-analysis** | 坡度、坡向、坡度数值、填挖方、土方量、地形剖面、剖面高程采样、剖切、地形三角网、径流、高程热力、视域、通视、视廊、视球、开敞度、天际线、限高核查、最近点、模型拍平、退线、CAE 有限元云图、管线横断面、日照、日照模拟、建筑立面图、顶视图、洪水淹没、洪水、水污染扩散、污染扩散、SlopeAnalysis、CalculateSlope、CutFill、SectionAnalysis、TerrainSectionAnalysis、ModelSectionAnalysis、TerrainTINAnalysis、Runoff、ElevationHeatMap、ViewShedAnalysis、VisibilityAnalysis、ViewCorridorAnalysis、ViewDomeAnalysis、SkylineAnalysis、LimitHeight、NearestPointAnalysis、ModelFlat、BackLine、CaeFeCloud、PipeLineCrossSection、Sunlight、FacadeAnalysis、FloodAnalysis、WaterPollutionAnalysis、空间分析、管线断面、立面分析 |
| 11 | **cimcube-simulation** | 仿真、实时路径、车辆漫游、垂缆、索道、绝缘网、跨越防护网、悬链线 |
| 12 | **cimcube-utils** | 测量、测距、测面积、测夹角、方位角、坡度、贴地距离/面积、投影面积、三角测量、模型体积、单位换算、吸附 Snap、剖切 BoxClip/SceneClip/ModelClip/ClipBase、区域裁剪 ModelCut、开挖 ModelExcavation/TerrainClip、缓冲 BufferTool、等高线 ContourLine、取盒 BoxDragger、EPSG 坐标转换、卷帘分屏 MapSplit/MultiViewPort、视线里程 SightLimitCalculate、数据热力 HeatMap、模型表面折线批注 ModelMarkup、图层克隆 SpatialTilesetCloner、凸包破碎 ConvexObjectBreaker、状态栏/比例尺/图例/POI/性能的 Service 数据层（UI 组件本身见 cimcube-widgets）；Measure、VolumeCalculation、Snap、ClipBase、MapSplit、CoordinateTransform、HeatMap、ModelMarkup、CompassService、POISearchService…、热力图、瓦片克隆 |
| 13 | **cimcube-widgets** | 指南针、指北针、回正北、地名搜索框、状态栏、距离图例比例尺、路径漫游录制、贴地漫游、图层控制树（只由初始化参数 `params: { layerControl: true }` 开启，见 cimcube-setup）、图例面板、卷帘组件、模型三维编辑、CompassWidget、GeocoderWidget、StatusBar、DistanceLegend、CameraRoam、GroundRoam、MapLegendWidget、MapSplitWidget、DragModel |
| 14 | **cimcube-controls** | 场景取点/锚点（map.controls + TakeCoordinates + 吸附联动）、交互控件、ADD、FINISH、MOUSE_MOVE |
| 15 | **cimcube-global** | 常量、枚举、Defines、Math、全局 |
| 16 | **using-cimcube-skills** | 帮助、怎么用、技能目录、命名空间、如何使用、CIMCube 技能、SDK 加载 |

## See Also

- **cimcube-setup** — 实例初始化、构造参数、容器 DOM（加载方式见 `references/sdk-loading.md`）
- **cimcube-layer / cimcube-camera / cimcube-graphic** — 「加载数据 → 定位飞行 → 标绘」最短链路
- **`references/class-routing.md`（本技能，自动生成）** — 「某模块有哪些真实类 / 该类归哪个技能 / 有没有 API_DOC 页」的**唯一真相源视图**：域 → 技能 → 类数 + 逐类 FQN 归属 + 例外口径；**类数口径以它为准**，勿另立数字
- **`references/domains-index.md`（本技能，手写）** — 命名空间总路由与各域入口文件导航
