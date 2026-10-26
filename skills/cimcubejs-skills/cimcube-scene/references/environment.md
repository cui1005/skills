# 大气 / 天空 / 环境光 · 雾 · 阴影（`map.scene`）

> 本文件对应 SKILL.md 路由表的「大气、天空、雾（map.scene.fog）、环境光」与「阴影」两行。写这几类代码前先读本文。

## 1.1 大气 / 天空 / 环境光

| 成员 | 类型 | 说明 |
|---|---|---|
| `atmosphere` | Boolean（默认 false） | 大气氛围开关。置 true 后联动：大气高度/`globe.enableLighting`/地面大气/雾/太阳与天空盒显示 |
| `firmamentSkyBox` | `Cesium.SkyBox` | 苍穹天空盒（背景球，最外层） |
| `skyBox` | `GroundSkyBox` | 近地天空盒（替换/读取用 `map.scene.skyBox.mode = …` 或整体 `new GroundSkyBox`） |
| `reflectBox` | `ReflectBox` | 环境反射盒（影响 PBR 高光反射，复用 `map.scene.reflectBox` 对象或整体替换） |
| `nephogram` | Boolean | 云图层开关（高轨云图，性能优于真实感体积云） |
| `fixedLightTime` / `fixedLightDirection` | String / Vector3 | 固定太阳时刻（如 `'11:00:00'`）/固定光照方向 |
| `lightColor` | String | 环境光颜色（默认 `#FFFFFF`） |
| `lightIntensity` | Number | 环境光强度 |
| `sunLightIntensity` | Number | 太阳光强度 |
| `backgroundColor` | `Cesium.Color` | 场景背景色 |
| `dynamicVolumetricSky` | String | 动态体积天空模式，值取自 SkyBoxMode（如 `'SUMMER'`），运行时属性 |

### 1.1.1 程序化光源注册（入口在 map.scene）

光源类（`DirectionalLight` / `PointLight` / `SpotLight`）归属 `cimcube-realistic-rendering`，但**注册入口在 `map.scene`**——只 `new` 不 `addLightSource` 不会生效：

| 方法 | 说明 |
|---|---|
| `map.scene.addLightSource(light)` | 把创建好的光源加入场景（光源才生效） |
| `map.scene.removeLightSource(light)` | 移除已注册光源 |

```js
const sun = new Glodon.CIMCube.Scene.DirectionalLight({ /* … 见 cimcube-realistic-rendering */ });
map.scene.addLightSource(sun);   // 必须注册，否则光源不可见
```

## 1.2 雾（原生 Cesium Fog，APIDOC 类型 `Cesium.Fog`）

`map.scene.fog` 常用成员：`enabled / density / minimumBrightness / screenSpaceErrorFactor`（对象为 Cesium 原生 fog）。真实感雾用 `cimcube-realistic-rendering` 的 `RR.exponentialHeightFog`；天气粒子起雾用 `cimcube-animation` 的 `WeatherEffect.Fog`。

## 1.3 阴影

| 成员 | 类型 | 说明 |
|---|---|---|
| `shadows` | Boolean | 阴影总开关（**光源投影的前提**） |
| `softShadows` | Boolean | 软阴影（PCF，贵） |
| `shadowQuality` | String | 阴影质量（默认 `"medium"`，取字符串质量档） |
| `shadowColor` | Color/String | 阴影颜色 |
| `shadowIntensity` | Number | 阴影浓度 |
| `shadowRange` | Number | 阴影可见范围（米） |
| `shadowUpdateFrequency` | Number | 阴影更新频率 |
| `shadowOptimizeWhenMoving` / `shadowOptimizeWhenIdling` | Boolean | 相机运动/静止时阴影优化 |
| `castShadowAlphaThreshold` | Number | 透明像素作废阈值 |
| `terrainShadows` | `Defines.ShadowMode` | 地形阴影模式：`DISABLED/ENABLED/CAST_ONLY/RECEIVE_ONLY` |
| `shadowMap` | `Cesium.ShadowMap`（只读） | 底层阴影贴图 |
| `updateShadowMap()` | 方法 | 手动强制刷新阴影 |

```js
map.scene.shadows = true; map.scene.softShadows = true;
map.scene.shadowColor = '#111827'; map.scene.shadowIntensity = 0.6;
map.scene.terrainShadows =Glodon.CIMCube.Global.Defines.ShadowMode.ENABLED;
```
