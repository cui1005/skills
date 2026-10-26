# 真实感管线总开关与 8 个子模块

> 本文件对应 SKILL.md 路由表的「总开关、8 个子模块（bloom/skyLight/skyAtmosphere/exponentialHeightFog/volumetricClouds/eyeAdaptation/lightShaft/origin）成员表、雨后黄昏真实感示例」行。

```js
map.scene.enableRealisticRendering = true;   // 切到 UE4 tonemapper 真实感管线（图层材质会走真实感光照）
const RR = map.scene.realisticRendering;     // RealisticRendering 实例，8 个子模块均为 getter 属性
```

> 关闭：`map.scene.enableRealisticRendering = false`（恢复普通 tonemapper）。
> 子模块类 APIDOC 归属 `Glodon.CIMCube.Scene.RealisticRendering.xxx`，但运行时**只能通过 `map.scene.realisticRendering.xxx` 取实例**，`Glodon.CIMCube.Scene` 上不可 new 这些类。

| 子模块 | 主要成员（均为直接读写） |
|---|---|
| `RR.origin` | `lon / lat / height`（真实感场景原点，默认 `[108.9236,34.5408,0]`） |
| `RR.skyLight` | `enabled / color / intensity`（默认 `#FFFFFF`、1；**无 topColor/bottomColor**） |
| `RR.skyAtmosphere` | `enabled / atmosphereHeight / groundAlbedo / rayleighScattering(色) / rayleighScatteringScale / rayleighExponentialDistribution / mieScatteringScale / mieAbsorptionScale / mieExponentialDistribution / skyLuminanceFactor / aerialPerspective* / tentIntensity / tentFilterRadius / tentFilterRadiusScale / tentFilterScalar` |
| `RR.exponentialHeightFog` | `enabled / height / heightFalloff / density / startDistance / cutoffDistance / color / maxOpacity / secondDensity / secondHeightFalloff / inscatteringColor / inscatteringExponent / inscatteringStartDistance / inscatteringDensity` |
| `RR.volumetricClouds` | `enabled / bottomRadiusKm / heightKm / cloudCoverage / cloudSpeed / cloudDirection / cloudColor / cloudShadowIntensity / cloudShadowSoftness / cloudQuality(0\|1\|2)` |
| `RR.bloom` | `enabled / intensity`（**无 threshold/radius**） |
| `RR.eyeAdaptation` | `enabled / minEV100 / maxEV100 / histogramLogMin / histogramLogMax / lowPercent / highPercent / exposureCompensation / speedUp / speedDown / eyeAdaptationMode`（自动曝光/瞳孔适应） |
| `RR.lightShaft` | `enabled / bloomThreShold / bloomScale / bloomMaxBrightness / bloomTint / lightShaftBlurNumSamples / lightShaftFirstPassDistance`（阳光体积光束，由 DirectionalLight 提供） |

> 天光不是可 new 的光源：`RR.skyLight` 无构造入口，只能经 `map.scene.realisticRendering.skyLight` 使用。
> ⚠️ 文档口径：`RR.eyeAdaptation` 对应的 **APIDOC 类锚点路径带大写 typo** `Glodon.CIMCUBE.Scene.RealisticRendering.EyeAdaptation`（`CIMCUBE` 全大写）；运行时入口不受影响，仍是 `map.scene.realisticRendering.eyeAdaptation`。

## 真实感示例：雨后黄昏城市

```js
map.scene.enableRealisticRendering = true;
const RR = map.scene.realisticRendering;
RR.origin.lon = 116.397; RR.origin.lat = 39.909; RR.origin.height = 30;

RR.skyLight.color = '#dff3ff'; RR.skyLight.intensity = 1.2;
RR.skyAtmosphere.enabled = true;
RR.skyAtmosphere.rayleighScattering = '#7db8ff';
RR.exponentialHeightFog.enabled = true;
RR.exponentialHeightFog.height = 20; RR.exponentialHeightFog.density = 0.01;

const sun = new Glodon.CIMCube.Scene.DirectionalLight({
  position: { lon: 116.4, lat: 39.92, height: 300 },
  targetPosition: { lon: 116.397, lat: 39.909, height: 0 },
  color: '#ffd9a0', intensity: 2.2,
});
map.scene.addLightSource(sun);   // 运行中想换向：改 position/targetPosition 即可，无需移除

RR.lightShaft.enabled = true; RR.lightShaft.bloomScale = 0.7;
RR.bloom.enabled = true; RR.bloom.intensity = 1.0;
RR.eyeAdaptation.enabled = true;
```
