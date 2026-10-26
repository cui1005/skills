# CIMCube Skills 能力详情（16 个）

> 本文件对应 SKILL.md 路由表的「16 个技能各自的**关键 API、类数与已知坑位**」行。
> 激活关键词与技能路由见 `SKILL.md` 速查表；逐类可运行示例见各技能自己的 `references/`。

| # | Skill | 主要能力（关键 API / 类数 / 坑位） |
|---|---|---|
| 1 | cimcube-setup | `new Glodon.CIMCube(...)`、opts 构造参数、核心属性 |
| 2 | cimcube-scene | map.scene.atmosphere、map.scene.skyBox.mode、map.scene.shadows、map.scene.postProcessStages、map.scene.ambientOcclusion、ReflectBox |
| 3 | cimcube-realistic-rendering | map.scene.realisticRendering、DirectionalLight/PointLight/SpotLight、map.scene.addLightSource、RR.bloom/eyeAdaptation |
| 4 | cimcube-physics | map.scene.physics、enablePhysics、createBox/createSphere、setCollisionModel、addHinge |
| 5 | cimcube-camera | 相机定位/飞行动画/姿态视野读写（heading/pitch/roll 弧度·只读、fov 度）、getCaptureImg 截图（需 preserveDrawingBuffer）、getCameraInfo(true).cameraCopy 视角一键复现、环绕自转、changed/moveStart/moveEnd 事件 —— flyTo/flyToBoundingSphere 单对象签名、target=图层/{lon,lat,height}/Cartesian3/Rectangle、offset 角度制+range 米 |
| 6 | cimcube-globe | Globe 对象配置 |
| 7 | cimcube-layer | 图层加载/管理/事件，共 24 类（LayerManager + BaseLayer + 22 种 Layer）；入口 `map.layerManager`（⚠️ 无 `map.addLayer`）；公共契约 `show`/`alpha`；事件须用数值枚举 `Defines.LayerEventType.*` |
| 8 | cimcube-graphic | 运行时 Graphic 命名空间 14 图元类 + 5 枚举 + 2 坐标类（BaseGraphic/ModelGraphic 有文档页未导出）：构造 opts 顶层键 + style（*StyleOptions 优先），position(s) 数值数组；经 GraphicLayer.addGraphic/removeGraphic/removeAllGraphics/getGraphicById/getAllGraphics + GRAPHIC_ADDED/ENTITY_CLICK 事件管理，绘制/编辑在 cimcube-layer；`LabelGraphic` 的 `text` 默认 `''`、`setStyle` 覆盖 6 类（原 `setOptions` 已废弃） |
| 9 | cimcube-animation | 动画特效 41 类单技能（Effect 14 + LineEffect 5 + PlaneEffect 8 + ModelEffect 3 + ParticleEffect 3 + WeatherEffect 3 + PointEffect 4 + Animation 基类）：SKILL.md 为纯路由索引，逐命名空间可运行示例平置于 references/（Effect/LineEffect/PlaneEffect/ModelEffect/ParticleEffect/WeatherEffect/PointEffect + patterns.md），写某类前先读 SKILL 路由表定位文件；控制方法分族——ModelEffect 无 show/hide、PolylineVolumeFlow 用 destroy()、WeatherEffect 无 play/stop、PointEffect 仅 Dynamic 有 play/stop、RopeLines/UVTextureEffect 的 show 是布尔属性；BillboardEntity 已废弃 |
| 10 | cimcube-analysis | 空间分析 24 类单技能：SKILL.md 为纯路由索引，逐类可运行示例平置于 references/（22 个类文件 + patterns.md 公共初始化，剖切三兄弟共享 SectionAnalysis.md），写某类前先读 SKILL 路由表定位文件；无统一 start/FINISH、触发因类而异——NearestPointAnalysis 静态免 map、FloodAnalysis 走 startByGraphic/startByTerrain、仅 CutFill 有 FINISH、LimitHeight 派发 change |
| 11 | cimcube-simulation | 命名空间 4 导出类：RealTimePath=帧命令数据(C/U/D+geo/state)→conversionFramePathData→updatePaths 驱动的实例化模型漫游（真实时钟 tick、事件 featureRendered/featureDisposed/featureMoved/propertiesChanged，**无 play/pause/回放接口**）；CatenaryCable / InsulationOverheadNet=纯 RopeLines 绳管渲染（showRope/showNet/updateOption/dispose，无净空/物理计算）；GeoRoamingGenerator=无 API_DOC 的演示帧数据生成器 |
| 12 | cimcube-utils | 工具集 28 类单技能；控制方法分族——静态类（CoordinateTransform/SightLimitCalculate/BufferTool）不 new、POISearchService 只收 options、分屏 begin/stop、服务 start/stop |
| 13 | cimcube-widgets | UI 组件 11 导出类单技能；①浮层导航与状态 6 类 + ②面板与交互 4 类。关键坑——**全局仅导出 11 类**（BaseWidget/Compass/Geocoder/LayerControl/DraggableDialog 有 API_DOC 页但未导出，内部实例**不对外暴露访问器**，多数控件保存 `addToMap()` 返回值作为句柄——但 `CompassWidget`/`GeocoderWidget` 是 mixin 包装、**无返回值**，显隐走根 DOM；仅 `map.mapLegend` 是 CIMCube 自有访问器）、MapLegendWidget 用 `.addTo(map)`、GeocoderWidget 必传 geoCoderMode、StatusBar precision 为 Number、卷帘键 `viewPortLayersVisible`、**图层管理面板不是业务控件：由 CIMCube 自动挂载，开关只有初始化参数 `params.layerControl`（业务再 new 会出现两个叠层面板）** |
| 14 | cimcube-controls | 真实入口：`map.controls`（内建 `takeCoordinates`/`allTakeCoordinates`/snap 配置）；自建 `new Controls.TakeCoordinates(map.controls)`；事件 `MapActionType.ADD/MOVE/FINISH` + 底层 `ScreenSpaceEventType`；导出类 TakeCoordinates + PickEventHandler；无 mode/undo/BaseControl |
| 15 | cimcube-global | 全局常量定义、数学工具 |
| 16 | using-cimcube-skills | 本页：使用指南 + 总览 + Quick Start |
