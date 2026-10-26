# 天空盒与反射盒（GroundSkyBox / ReflectBox）

> 本文件对应 SKILL.md 路由表的「天空盒 GroundSkyBox（SkyBoxMode 17 预置 / 自定义 6 面）/ 反射盒 ReflectBox」行。

两个类共享同一套 options（`ReflectBox extends GroundSkyBox`），区别只是用途：`map.scene.skyBox` 贴背景、`map.scene.reflectBox` 为 PBR 提供环境反射。

> ⚠️ **真实感与天空盒不要同时使用**：开启 `map.scene.enableRealisticRendering = true` 后，天空由真实感管线（`RR.skyAtmosphere` 大气散射 / `RR.skyLight` 天空光）接管，`map.scene.skyBox` / `map.scene.firmamentSkyBox` 会被覆盖而**不可见**。二者二选一，不要同时配置，否则天空盒会“消失”。

```js
const gsb = new Glodon.CIMCube.Scene.GroundSkyBox({
  mode: Glodon.CIMCube.Global.Defines.SkyBoxMode.SUNSET, // 内置预设
  show: true,
  useHDRCubeMap: false,          // 需要 HDR 天空时也可依赖 mode 自动判定
  sources: {                     // 与 mode 互斥：自定义 6 面贴图后 mode 不再起作用
    positiveX: '.../px.jpg', positiveY: '.../py.jpg', positiveZ: '.../pz.jpg',
    negativeX: '.../nx.jpg', negativeY: '.../ny.jpg', negativeZ: '.../nz.jpg',
  },
});
map.scene.skyBox = gsb;              // 整体替换；或在已存在实例上 map.scene.skyBox.mode = …
```

**mode 预置表（17 项，即 `Defines.SkyBoxMode`，值小写字符串；内部按 `${GIS_RESOURCE_URL}Resource/SkyBox/{mode}/` 加载 6 面）**

| 组 | 成员 | 效果 |
|---|---|---|
| 常规 JPG（低开销） | `FIRECLOUD / DAWN / DUSK / MIDNIGHT / SUMMER / BLUESKY / SUNSET / LATERNIGHT / SUNNYSUN / CLOUDY / FACTORY / BUILDING / CITY` | 火云/晨曦/黄昏/子夜/晴空/蓝天/晚霞/深夜/艳阳/多云/厂房/楼群/都市 |
| HDR（更好反射，贵） | `HDR_CLOUDY / HDR_FACTORY / HDR_URBAN` | 多云/厂房/都市 HDR |
| 默认 | `DEFAULT` | 默认蓝天 |

成员：`mode`（读小写）、`show`、`sources`、`useHDRCubeMap`、`state`（加载状态机）、`error`；方法：`isReady() / isLoading() / isFailed() / destroy()`。加载失败时 `error` 含原因。

> 反射盒：`map.scene.reflectBox = new ReflectBox({ mode: … })`；切场景后可 `map.scene.reflectBox.mode = …`（内部复用实例不重建）。模型要出反射效果需 PBR 材质（roughness 低），见 `cimcube-layer`。
