# 命名空间总览（`Glodon` 全局对象）

> 本文件对应 SKILL.md 路由表的「`Glodon.CIMCube.*` 命名空间层级树、各子系统归属与类数」行。所有导出 API 均挂在 **`Glodon`** 下。

```
Glodon
└── CIMCube                           # 主类 + 核心子系统
    ├── Camera                        # 相机定位/飞行/截图/视角/自转（map.camera）
    ├── Scene                         # 场景管理（→ cimcube-scene：环境/阴影/后处理/天空盒/反射盒）
    │   ├── RealisticRendering        #   真实感渲染 8 类：Bloom/HeightFog/SkyAtmosphere/VolumetricClouds/LightShaft/SkyLight/Origin + EyeAdaptation 大写 CIM**CUBE** 变体（→ cimcube-realistic-rendering）
    │   ├── DirectionalLight / PointLight / SpotLight / GroundSkyBox / ReflectBox  #   5 类（光源 3 类 → cimcube-realistic-rendering；天空盒/反射盒 → cimcube-scene）
    ├── ScenePhysics                  # ⚠️ 根命名空间类（不是 Scene.ScenePhysics），1 类（→ cimcube-physics）
    ├── Globe                         # 地球对象 1 类
    ├── Graphic                       # 矢量图元/标注（APIDOC 18 类页；运行时导出 14 图元 + 5 枚举 + GeoPoint/TwoDimensionPoint，BaseGraphic/ModelGraphic 页未导出 → cimcube-graphic）
    ├── Layer                         # 图层体系（LayerManager + 22 种 Layer + FeatureCollection + GeoJsonFeatureCollection + FeatureStyleParser + BaseLayer = 共 24 类）
    ├── Analysis                      # 空间分析（24 类，命名惯例 XxxAnalysis，CutFill/Sunlight 等 8 个例外）
    ├── Animation                     # 动画特效（7 子模块，共 41 类）
    │   ├── Effect(14) / LineEffect(5) / ModelEffect(3)
    │   ├── ParticleEffect(3) / PlaneEffect(8)
    │   └── PointEffect(4) / WeatherEffect(3) + Animation 基类
    ├── Simulation                    # 仿真（导出 4 类：3 文档类 + GeoRoamingGenerator 无文档帧数据生成器）
    ├── Utils                         # 工具集（共 28 类，已与 classes.list.html 对齐；DrawEdit/RoadLine/ElevationProfile 等 5 项为上层封装模式，非 SDK 原生类）
    ├── Widgets                       # UI 组件（共 16 类，惯例 API 类 + 同名 Widget 后缀 UI 容器成对）
    ├── Controls                      # 交互控件（SDK 仅 TakeCoordinates 1 类；BaseControl 为业务抽象，建议按 start/stop/pause/reset 生命周期自行封装）
    └── Global                        # 全局：仅 Logger.Logger 1 个真实类；Defines / Math 是命名空间容器非 Class
```

> 逐类归属与「某模块有哪些真实类」：读 `references/class-routing.md`（类目唯一真相源视图，自动生成；含域 → 技能 → 类数、逐类 FQN 归属与例外口径）。
