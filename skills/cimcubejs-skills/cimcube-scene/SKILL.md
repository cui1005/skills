---
name: cimcube-scene
description: "CIMCube 场景渲染与光环境：提供大气、天空与天空盒、雾、阴影、后期水与色彩等后处理、画质调试、环境光遮蔽AO、反射盒及 globalSettings 渲染预置等场景环境能力。当用户需要调场景环境、大气、天空盒、雾、阴影、后处理、画质调试、环境光遮蔽、反射盒、globalSettings 时激活。"
---

# CIMCube 场景渲染与光环境

> 本技能覆盖 **`map.scene` 的环境 / 阴影 / 后期水 / 天空盒 / 反射盒**。
> **SKILL.md 是纯路由索引**：写码前按下面「路由表」只读你真正需要的那一个 `references/` 文件，避免整包灌入上下文。

## 路由表（写码前必读）

| 你要写 / 查什么 | 真实入口 | 读这个文件 |
|---|---|---|
| 大气、天空、雾（`map.scene.fog`）、环境光 | `map.scene.atmosphere`、`map.scene.skyBox`、`map.scene.firmamentSkyBox`、`map.scene.fog` | `references/environment.md` |
| 阴影 | `map.scene.shadows / softShadows / shadowQuality / terrainShadows / updateShadowMap()` | `references/environment.md` |
| 后期水 / 画质调试（HDR/MSAA/目标帧率）/ AO | `map.scene.postProcessStages`、`enableHdr()`、`msaaSamples`、`map.scene.ambientOcclusion.enable / .intensity` | `references/postprocess-quality.md` |
| 天空盒 GroundSkyBox（SkyBoxMode 17 预置 / 自定义 6 面）/ 反射盒 ReflectBox | `new Glodon.CIMCube.Scene.GroundSkyBox / ReflectBox(opts)` | `references/skybox-reflectbox.md` |
| 易错写法对照、性能取舍 | — | `references/pitfalls.md` |

## Quick Start

```js
const scene = map.scene;
const Defines = Glodon.CIMCube.Global.Defines;

// ===== 氛围（非真实感模式基础盘） =====
map.scene.atmosphere = true;                                  // 大气+雾+日地光照联动
map.scene.skyBox.mode =Glodon.CIMCube.Global.Defines.SkyBoxMode.SUNSET;            // 近地天空盒换“晚霞”预设
map.scene.shadows = true;                                     // 开阴影（详见阴影小节）

// ===== 反射盒（建筑/车窗上有环境反射感） =====
map.scene.reflectBox = new Glodon.CIMCube.Scene.ReflectBox({ mode:Glodon.CIMCube.Global.Defines.SkyBoxMode.CITY });

// ===== 真实感渲染 + 光源 → 见 cimcube-realistic-rendering =====
// ===== 场景物理 → 见 cimcube-physics =====
```

初始化时可选择用 `globalSettings` 一次性预置（渲染项与图层/视图并列）：

> ⚠️ **构造器不处理** `globalSettings` / `mapLayers` / `defaultView` 这三个字段！必须通过 `Glodon.CIMCube.createScene(map, options)` 应用（详见 **cimcube-setup** 七条铁律第 7 条）。

```js
// 第一步：构造基础实例（id / gisResourceUrl / center / params 走这里）
const map = new Glodon.CIMCube({
  id: "cimcubeContainer",
  gisResourceUrl: "v3.55.0/dist/",
});

// 第二步：用 createScene 应用 globalSettings / mapLayers / defaultView
Glodon.CIMCube.createScene(map, {
  globalSettings: {
    skybox: Glodon.CIMCube.Global.Defines.SkyBoxMode.SUNSET,   // 近地天空盒
    environmentReflection: Glodon.CIMCube.Global.Defines.SkyBoxMode.SUNSET, // 或 'SKY' = 关反射盒
    atmosphere: true,
    lightColor: '#FFFFFF',
    lightingTime: 660,                       // 转成时刻 '11:00'（分钟数）
    light: { enable: true, intensity: 2.0 },
    ambientOcclusion: { enable: true, intensity: 1.0 },
    bloom: { enable: false, brightness: -0.3, contrast: 128, delta: 1, sigma: 5, stepSize: 3.78 },
    shadowEffect: { shadows: true, softShadows: false, terrainShadows: { castShadow: false, receiveShadow: true }, color: '#000000', intensity: 0.5, visualRange: 10000 },
    // 真实感管线子项 realisticRendering 见 cimcube-realistic-rendering
  },
  mapLayers: [],
  defaultView: { lon: 116.4, lat: 39.9, height: 2000 },
});
```

## 高频契约（别踩）

- `map.scene.shadows` 是**光源投影的前提**；只开光源不开阴影不出影。
- `msaaSamples`（MSAA）**不能与后处理混用**；`hardwareAA` 是另一条路径。
- 近地天空盒改预设用 `map.scene.skyBox.mode = …`（17 项，见 `references/skybox-reflectbox.md`）；`sources` 与 `mode` 互斥。
- 原生体积云 `map.scene.volumetricClouds` 与真实感云 `RR.volumetricClouds` **二选一**；低配用 `map.scene.nephogram` 云图。
- ★ **天空盒（skyBox / firmamentSkyBox）与真实感渲染互斥**：开 `map.scene.enableRealisticRendering` 后天空盒会被真实感天空接管而看不见，二者不要同时配置；要自定义天空就关真实感用 `map.scene.skyBox`。
- 初始渲染预置在 `globalSettings`，**不是** `sceneSetting`；且 `globalSettings` 必须通过 `Glodon.CIMCube.createScene(map, { globalSettings })` 应用，**构造器不处理**。

## See Also

- **cimcube-realistic-rendering** — 真实感管线（`enableRealisticRendering` + `map.scene.realisticRendering` 8 子模块：bloom/skyLight/skyAtmosphere/exponentialHeightFog/volumetricClouds/eyeAdaptation/lightShaft/origin）与光源 DirectionalLight/PointLight/SpotLight
- **cimcube-physics** — 场景物理引擎 ScenePhysics（`map.scene.physics` 刚体/碰撞/约束/软体/CCD）
- **cimcube-setup** — map 初始化（`globalSettings` 里渲染项与图层/视图等并列）
- **cimcube-camera** — `pickPosition`/`windowCoordinatesToCartesian`、相机飞行与 `scene` 其它交互成员
- **cimcube-global** — `Defines.ShadowMode`/`Defines.SkyBoxMode` 取值；坐标转换辅助
- **cimcube-layer / cimcube-graphic** — 模型图层 PBR 材质（配合 `reflectBox`）、影像透明 `setImagerLayerColorToAlphaThreshold`
- **cimcube-utils / cimcube-globe** — 模型剖切、地形开挖与材质（`map.scene.rendererManager`）
- **cimcube-animation** — 天气粒子（`WeatherEffect`）等氛围叠加可与此环境设置组合

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
