<!-- @Author glodon -->
<!-- @CreatTime 2026-10-13 -->

# 真实感风格模板：蓝天白云 / 暖阳

> 本文件对应 SKILL.md 路由表的「风格模板 / 画质档位」行。
> 来源：`globe.html`（广东场景）与 `sanzhongxin-scene.html`（上海三中心场景）实测通过的参数。
> **用户提到"美化场景""开启真实感"等需求时，默认整套套用本模板**（默认蓝天白云、`medium` 画质，不加切换按钮），只填写 ④ 场景输入；用户提到暖色调时改用暖阳，见「风格选择规则」。

模板分四层，按顺序装配：

| 层 | 内容 | 是否随风格变化 |
|---|---|---|
| ① 公共底座 | 前置开关、光照、阴影、各子模块的固定参数 | 否 |
| ② 风格预设 | 光照时间/颜色、天光、散射、雾色、云色、辉光、饱和度、对比度（14 个字段） | **是** |
| ③ 画质档位 | 阴影画质、体积云画质、AO | 否，按设备性能选 |
| ④ 场景输入 | `RR.origin`、相机初始视角 | 每个场景各自填写 |

## ① 公共底座

```js
function setupRealisticBase(map, origin) {
  var scene = map.scene;

  // 前置硬性要求：必须在 enableRealisticRendering 之前
  scene.logarithmicDepthBuffer = true;
  map.globe.depthTestAgainstTerrain = true;
  scene.enableHdr();
  scene.atmosphere = false;            // 关普通大气，由 RR.skyAtmosphere 接管
  map.globe.enableLighting = true;

  map.camera.fov = 65;
  scene.maxSightDistance = 30000000;
  scene.lightIntensity = 1;
  scene.sunLightIntensity = 1.6;
  scene.gamma = 1.05;

  scene.shadows = true;
  scene.softShadows = true;
  scene.shadowRange = 5000;
  scene.shadowIntensity = 0.55;

  scene.enableRealisticRendering = true;
  var RR = scene.realisticRendering;

  // ④ 场景输入：原点必须是本场景中心，照搬别的场景坐标会导致大气/雾错位
  RR.origin.lon = origin.lon;
  RR.origin.lat = origin.lat;
  RR.origin.height = origin.height || 0;

  RR.skyLight.enabled = true;

  RR.skyAtmosphere.enabled = true;
  RR.skyAtmosphere.skyLuminanceFactor = "#FFFFFF";
  RR.skyAtmosphere.groundAlbedo = "#676767";
  RR.skyAtmosphere.rayleighScatteringScale = 0.0331;
  RR.skyAtmosphere.mieAbsorptionScale = 0.000444;

  RR.exponentialHeightFog.enabled = true;
  RR.exponentialHeightFog.height = 50;
  RR.exponentialHeightFog.heightFalloff = 0.05;
  RR.exponentialHeightFog.density = 0.0004;
  RR.exponentialHeightFog.startDistance = 3000;
  RR.exponentialHeightFog.cutoffDistance = 30000;
  RR.exponentialHeightFog.maxOpacity = 0.6;

  RR.volumetricClouds.enabled = true;
  RR.volumetricClouds.bottomRadiusKm = 0.8;
  RR.volumetricClouds.heightKm = 0.65;
  RR.volumetricClouds.cloudCoverage = 0.85;
  RR.volumetricClouds.cloudSpeed = 0.35;     // >0 云层才会飘动
  RR.volumetricClouds.cloudDirection = 45;
  RR.volumetricClouds.cloudShadowIntensity = 0.25;
  RR.volumetricClouds.cloudShadowSoftness = 0.9;

  RR.bloom.enabled = true;

  RR.eyeAdaptation.enabled = true;
  RR.eyeAdaptation.exposureCompensation = 0.8;
  RR.eyeAdaptation.speedUp = 2;
  RR.eyeAdaptation.speedDown = 1;

  return RR;
}
```

## ② 风格预设

```js
var tonePresets = {
  blueSky: {
    name: "蓝天白云",
    fixedLightTime: "11:00:00",
    lightColor: "#FFFFFF",
    skyLightColor: "#F2F8FF",
    skyLightIntensity: 1.5,
    rayleighScattering: "#2A5CFF",
    mieScatteringScale: 0.0025,
    fogColor: "#CFE3F5",
    fogInscatteringColor: "#E6F2FC",
    fogInscatteringDensity: 0.1,
    cloudColor: "#FFFFFF",
    bloomIntensity: 0.06,
    saturation: 1.12,
    contrast: 1.05
  },
  warmSun: {
    name: "暖阳",
    fixedLightTime: "17:45:00",
    lightColor: "#FFD9A8",
    skyLightColor: "#FFD6A5",
    skyLightIntensity: 1.4,
    rayleighScattering: "#3A62E8",
    mieScatteringScale: 0.004,
    fogColor: "#F3D9BC",
    fogInscatteringColor: "#FFC98A",
    fogInscatteringDensity: 0.2,
    cloudColor: "#FFE0BC",
    bloomIntensity: 0.1,
    saturation: 1.18,
    contrast: 1.08
  }
};

function applyTonePreset(map, key) {
  var preset = tonePresets[key];
  var scene = map.scene;
  var RR = scene.realisticRendering;

  scene.fixedLightTime = preset.fixedLightTime;
  scene.lightColor = preset.lightColor;
  scene.saturation = preset.saturation;
  scene.contrast = preset.contrast;
  RR.skyLight.color = preset.skyLightColor;
  RR.skyLight.intensity = preset.skyLightIntensity;
  RR.skyAtmosphere.rayleighScattering = preset.rayleighScattering;
  RR.skyAtmosphere.mieScatteringScale = preset.mieScatteringScale;
  RR.exponentialHeightFog.color = preset.fogColor;
  RR.exponentialHeightFog.inscatteringColor = preset.fogInscatteringColor;
  RR.exponentialHeightFog.inscatteringDensity = preset.fogInscatteringDensity;
  RR.volumetricClouds.cloudColor = preset.cloudColor;
  RR.bloom.intensity = preset.bloomIntensity;
}
```

### 预设字段说明

| 字段 | 写入位置 | 蓝天白云 | 暖阳 | 调整方向 |
|---|---|---|---|---|
| `fixedLightTime` | `scene.fixedLightTime` | `11:00:00` | `17:45:00` | 决定太阳高度角；越接近日落越暖、越暗，水面越易发黑 |
| `lightColor` | `scene.lightColor` | `#FFFFFF` | `#FFD9A8` | 直射光色调 |
| `skyLightColor` | `RR.skyLight.color` | `#F2F8FF` | `#FFD6A5` | 环境光色调，影响阴影面颜色 |
| `skyLightIntensity` | `RR.skyLight.intensity` | 1.5 | 1.4 | 画面整体偏暗时优先调高 |
| `rayleighScattering` | `RR.skyAtmosphere.rayleighScattering` | `#2A5CFF` | `#3A62E8` | 天空主色，越饱和天越蓝 |
| `mieScatteringScale` | `RR.skyAtmosphere.mieScatteringScale` | 0.0025 | 0.004 | 太阳周围光晕/雾霾感 |
| `fogColor` | `RR.exponentialHeightFog.color` | `#CFE3F5` | `#F3D9BC` | 远景雾色 |
| `fogInscatteringColor` | `RR.exponentialHeightFog.inscatteringColor` | `#E6F2FC` | `#FFC98A` | 朝向太阳方向的雾色 |
| `fogInscatteringDensity` | `RR.exponentialHeightFog.inscatteringDensity` | 0.1 | 0.2 | 逆光雾的浓度 |
| `cloudColor` | `RR.volumetricClouds.cloudColor` | `#FFFFFF` | `#FFE0BC` | 云的底色 |
| `bloomIntensity` | `RR.bloom.intensity` | 0.06 | 0.1 | 高光泛光，>0.2 容易发糊 |
| `saturation` | `scene.saturation` | 1.12 | 1.18 | 整体饱和度，>1.25 植被易过绿 |
| `contrast` | `scene.contrast` | 1.05 | 1.08 | 整体对比度 |

新增风格（阴天、黄昏、清晨等）时，复制一组上述 14 个字段改值即可，切换函数不用动。

## ③ 画质档位

```js
var qualityTiers = {
  high:   { shadowQuality: "high", cloudQuality: 2, ao: true,  clouds: true,  bloom: true },
  medium: { shadowQuality: null,   cloudQuality: 1, ao: false, clouds: true,  bloom: true },
  low:    { shadowQuality: null,   cloudQuality: 0, ao: false, clouds: false, bloom: false }
};

function applyQualityTier(map, key) {
  var tier = qualityTiers[key];
  var scene = map.scene;
  var RR = scene.realisticRendering;

  if (tier.shadowQuality) scene.shadowQuality = tier.shadowQuality;
  RR.volumetricClouds.enabled = tier.clouds;
  RR.volumetricClouds.cloudQuality = tier.cloudQuality;
  RR.bloom.enabled = tier.bloom;
  if (scene.ambientOcclusion) {
    scene.ambientOcclusion.enable = tier.ao;
    scene.ambientOcclusion.intensity = 0.6;
  }
}
```

| 档位 | 适用 | 说明 |
|---|---|---|
| `high` | 独立显卡、独立浏览器窗口、模型量少 | `globe.html` 原始配置 |
| `medium` | 集成显卡、IDE 内嵌浏览器、多个 3D Tiles 图层 | 默认推荐；三中心场景 5 个图层时用此档稳定 |
| `low` | 低端设备、超大范围城市 | 关体积云与辉光，保留大气与雾 |

`high` 档在内嵌浏览器 + 多图层场景下曾出现 WebGL 上下文丢失、页面每 15–25 秒自动重载；出现此现象先降到 `medium`。

## 风格选择规则

| 用户说法 | 风格 |
|---|---|
| 未提色调（美化场景、开启真实感、更逼真、提升画质等） | `blueSky` 蓝天白云（默认） |
| 暖色调、暖色、暖阳、夕阳、黄昏、傍晚、温暖 | `warmSun` 暖阳 |

- 只装配选中的那一组预设，**不加风格切换按钮**。
- 用户明确要求"可切换风格 / 加切换按钮"时，才同时放两组预设并自行加按钮。

## 完整装配示例

```js
var map = new Glodon.CIMCube({
  id: "cimcubeContainer",
  gisResourceUrl: "https://cimcube-test.glodon.com/websdk/latest/",
  baseMap: {
    show: true,
    type: Glodon.CIMCube.Global.Defines.LayerType.IMAGERY,
    imageryType: Glodon.CIMCube.Global.Defines.ImageryLayerType.TDT_IMG,
    minimumLevel: 1,
    maximumLevel: 18
  }
  // 模板的天空由 RR.skyAtmosphere + RR.volumetricClouds 提供，不配 globalSettings.skybox / dynamicVolumetricSky
});

setupRealisticBase(map, { lon: 121.319855, lat: 31.214706 });
applyQualityTier(map, "medium");
applyTonePreset(map, "blueSky");   // 用户提到暖色调时改为 "warmSun"
```


## 配套的图层设置

风格模板只管场景与真实感子模块；图层按类型配合以下设置效果最好（图层 API 见 `cimcube-layer`）：

| 图层类型 | 设置 |
|---|---|
| 建筑 | `shadowMode: Defines.ShadowMode.ENABLED` |
| 地面 / 道路 | `shadowMode: Defines.ShadowMode.RECEIVE_ONLY` |
| 树木 / 植被 | `faceToCamera: true` |
| 水面 | `shadowMode: DISABLED`，`metallicRoughnessFactor: [0.66, 0.08]`，加载后 `brightness ≈ 1.0`、`saturation 0.8`、`hue 0.18`、`luminanceAtZenith 0.2` |
| 通用 | `maximumScreenSpaceError: 16` |

水面参数的取值依据见 `pitfalls.md` 第 12 条。
