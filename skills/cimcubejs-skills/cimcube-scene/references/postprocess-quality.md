# 后期水 / 色彩 / 画质调试 · 环境光遮蔽 AO

> 本文件对应 SKILL.md 路由表的「后期水 / 画质调试（HDR/MSAA/目标帧率）/ AO」行。

## 1.4 后期水 / 色彩

| 成员 | 类型 | 说明 |
|---|---|---|
| `postProcessWater` / `usePostProcessWater` | Object / Boolean | 后期水；运行时开关 `map.scene.postProcessWater`，初始预置走 `createScene` 的 `globalSettings.waterEffect`（色/频率/透明度） |
| `brightness / contrast / hue / saturation / gamma` | Number | 画面色彩调整；运行时在 `map.scene.*`，初始预置走 `createScene` 的 `globalSettings.colorDisplay` |
| `colorMode` | String（默认 `HSB`） | 色彩调整模式 |
| `volumetricClouds` | `Glodon.CIMCube.Cesium.VolumetricClouds`（只读） | 原生体积云（低配方案）；真实感云用 `RR.volumetricClouds`（见 `cimcube-realistic-rendering`），二选一 |
```js
map.scene.postProcessWater = true;
```

## 1.5 画质 / 调试

`msaaSamples`（MSAA 采样，注意不能与后处理混用）、`hardwareAA`、`logarithmicDepthBuffer`、`enableHdr()/disableHdr()`、`enableMoon()/disableMoon()`、`enableGlobeTerrainWireframe`、`useDefaultTerrain / useDefaultLabel`、`enableDefaultTerrain()/disableDefaultTerrain()`、`debugShowFramesPerSecond / debugShowDrawCalls / debugShowPostRender`、`targetFrameRate`、`minSightDistance`（默认 1）/`maxSightDistance`（默认 5000 万，超此不渲染）、`enableAnimation`、`autoUpdateGlobeTransparency`、`extent / width / height / canvas`、`rendererManager`、`memoryController`、`baseMap`、`terrainProvider`、`light`（`Cesium.Light`）。

## 1.7 环境光遮蔽 AO

| 成员 | 说明 |
|---|---|
| `ambientOcclusion` | `AmbientOcclusion`（只读）。常用 `map.scene.ambientOcclusion.enable / .intensity` |
| `aoOptimizeWhenMoving` | 相机运动时 AO 降档优化 |

```js
map.scene.ambientOcclusion.enable = true;
map.scene.ambientOcclusion.intensity = 1.0;
```
