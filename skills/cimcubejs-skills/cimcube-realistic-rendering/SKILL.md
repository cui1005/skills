---
name: cimcube-realistic-rendering
description: "CIMCube 真实感渲染（UE4 管线）与场景美化默认方案。内置两套实测风格模板「蓝天白云 / 暖阳」+ 画质档位：用户只要提到美化场景、场景更好看/更逼真/更真实、提升画质/效果、开启真实感、电影级画质，一律默认套用该模板的「蓝天白云」风格；用户提到暖色调/暖阳/夕阳/黄昏时改用「暖阳」风格（不加风格切换按钮）。覆盖总开关与 8个子模块（ 辉光 / 天空光 / 大气散射 / 高度雾 / 体积云 / 自动曝光/ 体积光 / 原点，含 EyeAdaptation）、光源（平行光 / 点光源 / 聚光灯，构造后必须注册，每类上限 16）。当用户要美化场景、开启真实感、蓝天白云/暖阳/晴天/黄昏等风格切换、色调预设、画质分档、打光、辉光、体积云、大气散射、高度雾、体积光束、自动曝光时激活。"
---

# CIMCube 真实感渲染（场景美化默认方案）

> 本技能只覆盖 **`map.scene.enableRealisticRendering` 真实感管线 + 8 个子模块 + 三类光源 + 风格模板**。场景环境（普通大气 / 天空盒 / 反射盒 / 后期水 / 画质 / AO）见 `cimcube-scene`；物理见 `cimcube-physics`；模型 PBR 材质见 `cimcube-layer`。
>
> **SKILL.md 是纯路由索引**：写码前按「路由表」只读需要的那一个 `references/` 文件，避免整包灌入上下文。

## ★ 默认策略：美化场景 / 开启真实感 = 套风格模板

用户说下面任意一类话，**不要从零调参**，直接读 `references/style-presets.md` 套模板：

- "美化场景""让场景更好看 / 更逼真 / 更真实""提升画质 / 效果""优化渲染效果"
- "开启真实感""真实感渲染""电影级画质"
- "蓝天白云""暖阳""暖色调""晴天""黄昏""换个色调 / 风格"

套用规则：

1. **只用一种风格，不加切换按钮**：默认 `applyTonePreset(map, "blueSky")`（蓝天白云）。用户提到"暖色调""暖阳""暖色""夕阳""黄昏""傍晚"时改用 `"warmSun"`（暖阳）。页面只放用到的那一组预设；用户明确要求"可切换风格"时才同时放两组并加按钮。
2. **只填场景输入**：`RR.origin` 用场景中心（有模型时取主模型位置或相机初始视角的经纬度），相机视角沿用页面原有设置。其余参数照抄模板，不要自行改值。
3. **画质默认 `medium`**：IDE 内嵌浏览器、多个 3D Tiles 图层时必须 `medium`；用户明确要"最高画质"且在独立浏览器运行时才用 `high`。
4. **顺带配图层**：已有图层按模板「配套的图层设置」补阴影模式、植被 `faceToCamera`、水面 PBR 参数；图层自身的定位参数（origin/offset/rotation）保持不动。
5. **清理冲突配置**：页面原有的 `skyBox` / `firmamentSkyBox` / `dynamicVolumetricSky` / `colorMode` / 原生 `scene.volumetricClouds` 等普通场景配置先移除。
6. **模板之外才手调**：用户点名要改某个子模块（如"云再厚一点""雾浓一些"）时，在模板基础上只改对应字段；字段含义查 `style-presets.md` 的字段说明表与 `submodules.md`。

## 入口速记

| 你要的东西 | 真实路径 |
|---|---|
| 场景美化 / 风格模板 | `references/style-presets.md`：`setupRealisticBase` → `applyQualityTier` → `applyTonePreset` |
| 真实感管线总开关 | `map.scene.enableRealisticRendering = true`（切 UE4 tonemapper；`false` 恢复普通 tonemapper） |
| 子模块实例 | `map.scene.realisticRendering`（**小写 r** 的 getter 实例，8 个子模块） |
| 子模块开关 | `map.scene.realisticRendering.bloom.enabled = true` 等（**不要 new**） |
| 可 new 的光源类 | `new Glodon.CIMCube.Scene.DirectionalLight / PointLight / SpotLight(opts)`，构造后必须 `map.scene.addLightSource(light)` |
| 初始化预置 | `new Glodon.CIMCube({id, gisResourceUrl, ...})`；如果有完整的场景数据，直接调用 `Glodon.CIMCube.createScene(map, { defaultView,mapLayers, globalSettings:{ realisticRendering: {...} } })` |

## 路由表（写码前必读）

| 你要写 / 查什么 | 读这个文件 |
|---|---|
| **美化场景 / 开启真实感（默认）**：公共底座 + 蓝天白云/暖阳 14 字段预设、high/medium/low 画质档位、配套图层设置 | `references/style-presets.md` |
| 总开关、8 个子模块（bloom/skyLight/skyAtmosphere/exponentialHeightFog/volumetricClouds/eyeAdaptation/lightShaft/origin）成员表、雨后黄昏真实感示例 | `references/submodules.md` |
| 三类光源的 options / 成员 / 注册与移除 | `references/lights.md` |
| 易错写法对照、性能取舍 | `references/pitfalls.md` |

## Quick Start（模板装配）

```js
// 1. 公共底座 + 场景原点（函数体见 references/style-presets.md）
setupRealisticBase(map, { lon: 121.319855, lat: 31.214706 });
// 2. 画质档位：受限环境默认 medium
applyQualityTier(map, "medium");
// 3. 风格：默认蓝天白云；用户提到暖色调时改为 "warmSun"
applyTonePreset(map, "blueSky");
```

额外打光（可选，真实感不依赖光源；要投影需 `map.scene.shadows = true`）：

```js
const sun = new Glodon.CIMCube.Scene.DirectionalLight({
  position: { lon: 116.4, lat: 39.9, height: 200 },  // 光源经纬高（米）
  color: '#fff3c4',
  intensity: 2.0,                                    // 真实感模式下为照度(lux)
});
map.scene.addLightSource(sun);
// map.scene.removeLightSource(sun);
```

也可采用初始化时一次性预置（等价于上面的装配）：

> ⚠️ **构造器不处理** `globalSettings` / `mapLayers` / `defaultView` 这三个字段！必须通过 `Glodon.CIMCube.createScene(map, options)` 应用（详见 **cimcube-setup** 七条铁律第 7 条）。

```js
// 第一步：构造基础实例
const map = new Glodon.CIMCube({
  id: "cimcubeContainer",
  gisResourceUrl: "v3.55.0/dist/",
});

// 第二步：用 createScene 应用真实感渲染预置
Glodon.CIMCube.createScene(map, {
  globalSettings: {
    realisticRendering: {
      enable: true,
      origin: [116.4, 39.9, 0],
      skylight: { enable: true, intensity: 1, color: '#FFFFFF' },
      skyAtmosphere: { enable: true, rayleighScattering: '#2c68ff' },
      exponentialHeightFog: { enable: true, density: 0.005 },
      volumetricClouds: { enable: true, cloudCoverage: 1 },
      bloom: { enable: true, intensity: 0.675 },
    },
  },
  mapLayers: [],
  defaultView: { lon: 116.4, lat: 39.9, height: 2000 },
});
```

## 高频契约（别踩）

- ★ **开 `enableRealisticRendering` 前必须先开 `map.scene.logarithmicDepthBuffer = true` 与 `map.globe.depthTestAgainstTerrain = true`**（管线硬性依赖，缺一深度/遮挡即错；后者在 `map.globe` 上，不在 `scene` 上）。模板的 `setupRealisticBase` 已包含，并配套 `enableHdr()`、`fov = 65`、`maxSightDistance = 30000000`。
- `map.scene.realisticRendering` 是**小写 r** 的 getter 实例；8 个子模块**不可 new**，只能 `RR.xxx.enabled = true`。
- `RR.origin` 必须是本场景中心，从别的页面复制模板时最容易漏改。
- 三类光源 `new` 之后**必须** `map.scene.addLightSource(light)` 才参与光照，每类上限 16。
- 光源的物理单位（lux / cd）**仅在**真实感模式下生效。
- 真实感模式发黑先查 `RR.eyeAdaptation` 与 `RR.skyLight.intensity`；光源要投影需 `map.scene.shadows = true`；只有水面发黑时调水面图层亮度，不改全局。
- ★ **真实感与天空盒（skyBox / firmamentSkyBox）互斥**：开 `enableRealisticRendering` 后天空由 `RR.skyAtmosphere` / `RR.skyLight` 接管，天空盒会被覆盖而看不见，二者不要同时配置。

## See Also

- **cimcube-scene** — 场景环境：普通大气 / 天空盒（GroundSkyBox 17 预置）/ 反射盒 / **阴影（`map.scene.shadows`，光源投影需开）** / 后期水 / 画质 / AO；`globalSettings` 中渲染项与 `realisticRendering` 并列
- **cimcube-layer** — 模型图层 PBR 材质（roughness 低才出反射/真实感高光，配合 `map.scene.reflectBox`）
- **cimcube-camera** — 真实感视角机位与飞行
- **cimcube-physics** — 场景物理引擎（ScenePhysics 刚体/碰撞/约束）
- **cimcube-global** — `Defines.SkyBoxMode` 等取值

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
