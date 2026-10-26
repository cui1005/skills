# CIMCube 类目路由清单（自动生成 · 勿手改）

> 本文件由**类目登记表**（仓库侧构建期真相源）自动生成，**请勿手工编辑**——改类目请改真相源后重跑生成器，不要在此手改。
> 本文件随技能包分发，是 Agent / 使用者唯一需要读取的类目视图（覆盖 15 域 / 176 类 + 例外口径）。
>
> 查「某模块有哪些真实类 / 该类归哪个技能 / 有没有 API_DOC 页」→ 先看本文件 §一/§二，再去对应技能的 `SKILL.md` 路由表取用法。

## 一、域 → 技能 → 类数

> 类数口径 = 类目登记表 `domains[].classes` 长度；同一口径的发布声明值由三方向对账（SDK classes.list.html ↔ 登记表 ↔ 发布清单，见 [D3]）。

| 域（命名空间） | 归属技能 | 技能入口 | 类数 |
|---|---|---|---:|
| Root(CIMCube + CIMCubeSDKLoader) | `cimcube-setup` | `cimcube-setup/SKILL.md` | 1 |
| Camera | `cimcube-camera` | `cimcube-camera/SKILL.md` | 1 |
| Scene.Lights + SkyBox + ReflectBox | `cimcube-scene` | `cimcube-scene/SKILL.md` | 6 |
| Scene.RealisticRendering | `cimcube-realistic-rendering` | `cimcube-realistic-rendering/SKILL.md` | 8 |
| ScenePhysics(Root) | `cimcube-physics` | `cimcube-physics/SKILL.md` | 1 |
| Globe | `cimcube-globe` | `cimcube-globe/SKILL.md` | 1 |
| Layer | `cimcube-layer` | `cimcube-layer/SKILL.md` | 24 |
| Graphic | `cimcube-graphic` | `cimcube-graphic/SKILL.md` | 18 |
| Animation | `cimcube-animation` | `cimcube-animation/SKILL.md` | 42 |
| Analysis | `cimcube-analysis` | `cimcube-analysis/SKILL.md` | 24 |
| Simulation | `cimcube-simulation` | `cimcube-simulation/SKILL.md` | 3 |
| Utils | `cimcube-utils` | `cimcube-utils/SKILL.md` | 28 |
| Widgets | `cimcube-widgets` | `cimcube-widgets/SKILL.md` | 17 |
| Controls | `cimcube-controls` | `cimcube-controls/SKILL.md` | 1 |
| Global.Defines / Global.Math / Global.Logger.Logger | `cimcube-global` | `cimcube-global/SKILL.md` | 1 |
| **合计** | 15 个技能域 | — | **176** |

## 二、逐类归属（FQN → 技能）

> FQN 为**运行时可直接引用的全限定名**（`Glodon.CIMCube.*`）；备注列给出与 API_DOC 的差异、导出状态与易错点。
> 各类的构造参数 / 方法用法见对应技能的 `SKILL.md` 路由表指向的 `references/*.md`。

### cimcube-setup｜Root(CIMCube + CIMCubeSDKLoader)（1 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.CIMCubeSDKLoader` | API_DOC 只记「SDK版本加载器」与无参构造，未记录任何公开成员/方法；运行时原型含 load() 但签名未验证 → 勿臆造参数 |

### cimcube-camera｜Camera（1 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.Camera` | — |

### cimcube-scene｜Scene.Lights + SkyBox + ReflectBox（6 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.Scene` | — |
| `Glodon.CIMCube.Scene.DirectionalLight` | — |
| `Glodon.CIMCube.Scene.GroundSkyBox` | — |
| `Glodon.CIMCube.Scene.PointLight` | — |
| `Glodon.CIMCube.Scene.ReflectBox` | — |
| `Glodon.CIMCube.Scene.SpotLight` | — |

### cimcube-realistic-rendering｜Scene.RealisticRendering（8 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCUBE.Scene.RealisticRendering.EyeAdaptation` | — |
| `Glodon.CIMCube.Scene.RealisticRendering.Bloom` | — |
| `Glodon.CIMCube.Scene.RealisticRendering.ExponentialHeightFog` | — |
| `Glodon.CIMCube.Scene.RealisticRendering.LightShaft` | — |
| `Glodon.CIMCube.Scene.RealisticRendering.Origin` | — |
| `Glodon.CIMCube.Scene.RealisticRendering.SkyAtmosphere` | — |
| `Glodon.CIMCube.Scene.RealisticRendering.SkyLight` | — |
| `Glodon.CIMCube.Scene.RealisticRendering.VolumetricClouds` | — |

### cimcube-physics｜ScenePhysics(Root)（1 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.ScenePhysics` | — |

### cimcube-globe｜Globe（1 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.Globe` | — |

### cimcube-layer｜Layer（24 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.Layer.BaseLayer` | — |
| `Glodon.CIMCube.Layer.ClusterLayer` | — |
| `Glodon.CIMCube.Layer.DivLayer` | — |
| `Glodon.CIMCube.Layer.FeatureCollection` | — |
| `Glodon.CIMCube.Layer.FeatureStyleParser` | — |
| `Glodon.CIMCube.Layer.GSMMLayer` | — |
| `Glodon.CIMCube.Layer.GeoJsonFeatureCollection` | — |
| `Glodon.CIMCube.Layer.GeoJsonLayer` | — |
| `Glodon.CIMCube.Layer.GltfLayer` | — |
| `Glodon.CIMCube.Layer.GmlLayer` | — |
| `Glodon.CIMCube.Layer.GraphicLayer` | — |
| `Glodon.CIMCube.Layer.GroupLayer` | — |
| `Glodon.CIMCube.Layer.I3STilesLayer` | — |
| `Glodon.CIMCube.Layer.ImageryLayer` | — |
| `Glodon.CIMCube.Layer.InstanceLayer` | — |
| `Glodon.CIMCube.Layer.InstanceLayerModel` | — |
| `Glodon.CIMCube.Layer.KmlLayer` | — |
| `Glodon.CIMCube.Layer.LayerManager` | — |
| `Glodon.CIMCube.Layer.S3MTilesLayer` | — |
| `Glodon.CIMCube.Layer.TerrainLayer` | — |
| `Glodon.CIMCube.Layer.Tiles3DLayer` | — |
| `Glodon.CIMCube.Layer.TransformableGroupLayer` | — |
| `Glodon.CIMCube.Layer.USDLayer` | — |
| `Glodon.CIMCube.Layer.VectorTileLayer` | — |

### cimcube-graphic｜Graphic（18 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.Graphic.BaseGraphic` | 有 API_DOC 页但未挂命名空间（基类） |
| `Glodon.CIMCube.Graphic.BillboardGraphic` | — |
| `Glodon.CIMCube.Graphic.BoxGraphic` | — |
| `Glodon.CIMCube.Graphic.CorridorGraphic` | — |
| `Glodon.CIMCube.Graphic.CylinderGraphic` | — |
| `Glodon.CIMCube.Graphic.EllipseGraphic` | — |
| `Glodon.CIMCube.Graphic.EllipsoidGraphic` | — |
| `Glodon.CIMCube.Graphic.GeoPoint` | — |
| `Glodon.CIMCube.Graphic.LabelGraphic` | — |
| `Glodon.CIMCube.Graphic.ModelGraphic` | 有 APIDOC 页但未挂命名空间（仅内部实现） |
| `Glodon.CIMCube.Graphic.PlaneGraphic` | — |
| `Glodon.CIMCube.Graphic.PointGraphic` | — |
| `Glodon.CIMCube.Graphic.PolygonGraphic` | — |
| `Glodon.CIMCube.Graphic.PolylineGraphic` | — |
| `Glodon.CIMCube.Graphic.PolylineVolumeGraphic` | — |
| `Glodon.CIMCube.Graphic.RectangleGraphic` | — |
| `Glodon.CIMCube.Graphic.TwoDimensionPoint` | — |
| `Glodon.CIMCube.Graphic.WallGraphic` | — |

### cimcube-animation｜Animation（42 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.Animation.Animation` | — |
| `Glodon.CIMCube.Animation.Effect.AmbientOcclusion` | — |
| `Glodon.CIMCube.Animation.Effect.BillboardEntity` | — |
| `Glodon.CIMCube.Animation.Effect.BillboardRoaming` | — |
| `Glodon.CIMCube.Animation.Effect.CameraRoaming` | — |
| `Glodon.CIMCube.Animation.Effect.CameraShooting` | — |
| `Glodon.CIMCube.Animation.Effect.FirstPersonRoaming` | — |
| `Glodon.CIMCube.Animation.Effect.Metaptosis` | — |
| `Glodon.CIMCube.Animation.Effect.PolylineVolumeFlow` | — |
| `Glodon.CIMCube.Animation.Effect.PrismPoint` | — |
| `Glodon.CIMCube.Animation.Effect.Radar` | — |
| `Glodon.CIMCube.Animation.Effect.ThirdPersonRoaming` | — |
| `Glodon.CIMCube.Animation.Effect.TrackRoaming` | — |
| `Glodon.CIMCube.Animation.Effect.VideoEntity` | — |
| `Glodon.CIMCube.Animation.Effect.Wall` | — |
| `Glodon.CIMCube.Animation.LineEffect.DynamicLines` | — |
| `Glodon.CIMCube.Animation.LineEffect.FlyLines` | — |
| `Glodon.CIMCube.Animation.LineEffect.RoadLine` | — |
| `Glodon.CIMCube.Animation.LineEffect.RopeLines` | — |
| `Glodon.CIMCube.Animation.LineEffect.TrailLines` | — |
| `Glodon.CIMCube.Animation.ModelEffect.ModelComplexColor` | 3.57.0 新增：模型单体要素按高程分段着色（step 硬切分 / gradient 段间插值） |
| `Glodon.CIMCube.Animation.ModelEffect.ModelExplode` | — |
| `Glodon.CIMCube.Animation.ModelEffect.ModelInteroperation` | — |
| `Glodon.CIMCube.Animation.ModelEffect.UVTextureEffect` | — |
| `Glodon.CIMCube.Animation.ParticleEffect.Fire` | — |
| `Glodon.CIMCube.Animation.ParticleEffect.Smoke` | — |
| `Glodon.CIMCube.Animation.ParticleEffect.Water` | — |
| `Glodon.CIMCube.Animation.PlaneEffect.DiffuseWall` | — |
| `Glodon.CIMCube.Animation.PlaneEffect.Dynamic` | — |
| `Glodon.CIMCube.Animation.PlaneEffect.Moire` | — |
| `Glodon.CIMCube.Animation.PlaneEffect.Radar` | — |
| `Glodon.CIMCube.Animation.PlaneEffect.RingScan` | — |
| `Glodon.CIMCube.Animation.PlaneEffect.Sequence` | — |
| `Glodon.CIMCube.Animation.PlaneEffect.Statics` | — |
| `Glodon.CIMCube.Animation.PlaneEffect.StaticsRingScan` | — |
| `Glodon.CIMCube.Animation.PointEffect.Dynamic` | — |
| `Glodon.CIMCube.Animation.PointEffect.InstanceBillboard` | — |
| `Glodon.CIMCube.Animation.PointEffect.Scatter` | — |
| `Glodon.CIMCube.Animation.PointEffect.Twinkle` | — |
| `Glodon.CIMCube.Animation.WeatherEffect.Fog` | — |
| `Glodon.CIMCube.Animation.WeatherEffect.Rain` | — |
| `Glodon.CIMCube.Animation.WeatherEffect.Snow` | — |

### cimcube-analysis｜Analysis（24 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.Analysis.BackLine` | — |
| `Glodon.CIMCube.Analysis.CaeFeCloud` | — |
| `Glodon.CIMCube.Analysis.CalculateSlope` | — |
| `Glodon.CIMCube.Analysis.CutFill` | — |
| `Glodon.CIMCube.Analysis.ElevationHeatMap` | — |
| `Glodon.CIMCube.Analysis.FacadeAnalysis` | — |
| `Glodon.CIMCube.Analysis.FloodAnalysis` | — |
| `Glodon.CIMCube.Analysis.LimitHeight` | — |
| `Glodon.CIMCube.Analysis.ModelFlat` | — |
| `Glodon.CIMCube.Analysis.ModelSectionAnalysis` | — |
| `Glodon.CIMCube.Analysis.NearestPointAnalysis` | — |
| `Glodon.CIMCube.Analysis.PipeLineCrossSection` | — |
| `Glodon.CIMCube.Analysis.Runoff` | — |
| `Glodon.CIMCube.Analysis.SectionAnalysis` | — |
| `Glodon.CIMCube.Analysis.SkylineAnalysis` | — |
| `Glodon.CIMCube.Analysis.SlopeAnalysis` | — |
| `Glodon.CIMCube.Analysis.Sunlight` | — |
| `Glodon.CIMCube.Analysis.TerrainSectionAnalysis` | — |
| `Glodon.CIMCube.Analysis.TerrainTINAnalysis` | — |
| `Glodon.CIMCube.Analysis.ViewCorridorAnalysis` | — |
| `Glodon.CIMCube.Analysis.ViewDomeAnalysis` | — |
| `Glodon.CIMCube.Analysis.ViewShedAnalysis` | — |
| `Glodon.CIMCube.Analysis.VisibilityAnalysis` | — |
| `Glodon.CIMCube.Analysis.WaterPollutionAnalysis` | — |

### cimcube-simulation｜Simulation（3 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.Simulation.CatenaryCable` | — |
| `Glodon.CIMCube.Simulation.InsulationOverheadNet` | — |
| `Glodon.CIMCube.Simulation.RealTimePath` | — |

### cimcube-utils｜Utils（28 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.Utils.BoxClip` | — |
| `Glodon.CIMCube.Utils.BoxDragger` | API_DOC 页名为 `Glodon.CIMCube.Utils.BoxDraggerUI` |
| `Glodon.CIMCube.Utils.BufferTool` | — |
| `Glodon.CIMCube.Utils.ClipBase` | — |
| `Glodon.CIMCube.Utils.CompassService` | — |
| `Glodon.CIMCube.Utils.ContourLine` | — |
| `Glodon.CIMCube.Utils.ConvexObjectBreaker` | — |
| `Glodon.CIMCube.Utils.CoordinateTransform` | — |
| `Glodon.CIMCube.Utils.DistanceLegendService` | — |
| `Glodon.CIMCube.Utils.HeatMap` | — |
| `Glodon.CIMCube.Utils.MapLegendService` | — |
| `Glodon.CIMCube.Utils.MapSplit` | — |
| `Glodon.CIMCube.Utils.Measure` | — |
| `Glodon.CIMCube.Utils.MeasureUtils.MeasureFormula` | — |
| `Glodon.CIMCube.Utils.MeasureUtils.MeasurementUtils` | — |
| `Glodon.CIMCube.Utils.ModelClip` | — |
| `Glodon.CIMCube.Utils.ModelCut` | — |
| `Glodon.CIMCube.Utils.ModelExcavation` | — |
| `Glodon.CIMCube.Utils.ModelMarkup` | — |
| `Glodon.CIMCube.Utils.MultiViewPort` | — |
| `Glodon.CIMCube.Utils.POISearchService` | — |
| `Glodon.CIMCube.Utils.PerformanceListening` | — |
| `Glodon.CIMCube.Utils.SceneClip` | — |
| `Glodon.CIMCube.Utils.SightLimitCalculate` | — |
| `Glodon.CIMCube.Utils.Snap` | — |
| `Glodon.CIMCube.Utils.SpatialTilesetCloner` | — |
| `Glodon.CIMCube.Utils.StatusBarService` | — |
| `Glodon.CIMCube.Utils.VolumeCalculation` | — |

### cimcube-widgets｜Widgets（17 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.Widgets.BaseWidget` | 有 API_DOC 页但未挂命名空间（内部基类） |
| `Glodon.CIMCube.Widgets.CameraRoam` | — |
| `Glodon.CIMCube.Widgets.Compass` | 有 API_DOC 页但未挂命名空间（内部实现），对外用 CompassWidget |
| `Glodon.CIMCube.Widgets.CompassWidget` | — |
| `Glodon.CIMCube.Widgets.DistanceLegend` | — |
| `Glodon.CIMCube.Widgets.DragModel` | — |
| `Glodon.CIMCube.Widgets.DraggableDialog` | 有 API_DOC 页但未挂命名空间（内部基类，CameraRoam/GroundRoam 父类） |
| `Glodon.CIMCube.Widgets.Geocoder` | 有 API_DOC 页但未挂命名空间（内部实现），对外用 GeocoderWidget |
| `Glodon.CIMCube.Widgets.GeocoderWidget` | — |
| `Glodon.CIMCube.Widgets.GroundRoam` | — |
| `Glodon.CIMCube.Widgets.LayerControl` | 内部类：图层管理面板本体，只由初始化参数 params.layerControl 开启，业务不 new |
| `Glodon.CIMCube.Widgets.LayerControlWidget` | 内部类的导出壳，同样只由 params.layerControl 开启，业务不 new |
| `Glodon.CIMCube.Widgets.MapLegendWidget` | — |
| `Glodon.CIMCube.Widgets.MapOptionsWidget` | 内部类：地图选项面板，业务不 new |
| `Glodon.CIMCube.Widgets.MapSplitWidget` | — |
| `Glodon.CIMCube.Widgets.StatusBar` | — |
| `Glodon.CIMCube.Widgets.statusBarMixin` | 内部 mixin：API_DOC 有类页、运行时未导出；直接用 Widgets.StatusBar |

### cimcube-controls｜Controls（1 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.Controls.TakeCoordinates` | — |

### cimcube-global｜Global.Defines / Global.Math / Global.Logger.Logger（1 类）

| 类（FQN） | 备注 |
|---|---|
| `Glodon.CIMCube.Global.Logger.Logger` | — |

## 三、例外口径（不计入上面的类数）

### 3.1 命名空间入口

- `Glodon.CIMCube` — SDK 主类（命名空间入口），API_DOC 类页存在但 classes.list 下拉未单列，不计入 SDK 类计数

### 3.2 运行时扩展（API_DOC 无类页）

- `Glodon.CIMCube.Simulation.GeoRoamingGenerator` — 运行时导出，API_DOC 无类页（classes.list 未收）；默认参数与 READY→GO→UNLOADING→RETURN→LOADING 状态机见 cimcube-simulation/references/RealTimePath-data.md

### 3.3 API_DOC 页名 ≠ 运行时导出名

- `Glodon.CIMCube.Utils.BoxDragger` → API_DOC 页 `Glodon.CIMCube.Utils.BoxDraggerUI`（业务代码用前者）

## 四、非 SDK 类（上层封装模式 · SDK 不存在，需业务自行封装）

1. Utils.DrawEdit (封装：Controls.TakeCoordinates + Utils.Measure + Graphic.*增删改查)
2. Utils.RoutePlan (封装：Controls.TakeCoordinates + Global.Math 路径 + Graphic.Polyline 渲染)
3. Utils.SkylineQuery (封装：Analysis.SkylineAnalysis 关闭渲染仅取结果折线)
4. Utils.ElevationProfile (封装：Analysis.TerrainSectionAnalysis + 外部 Chart 渲染)
5. RoadLine 绘制工具 (Animation.LineEffect.RoadLine 仅作为特效，绘制原语需 Graphic.Polyline 组合)

> 上述名称在 SDK 中**不存在**，不要 `new`；对应能力用 cimcube-controls / cimcube-utils / cimcube-graphic 组合实现。
