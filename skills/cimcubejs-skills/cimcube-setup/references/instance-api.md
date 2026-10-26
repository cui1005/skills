# CIMCube 实例成员与方法（含 dispose）

> 本文件对应 SKILL.md 路由表的「用实例：核心对象速查（scene/camera/globe/layerManager）、可读写成员、方法表（事件/定位/拾取/资源）、静态成员、`dispose()`」行。

> 相关文件：构造参数 → `constructor-options.md`；可运行脚手架 → `quick-start.md`；排错 → `pitfalls.md`

## 一、核心对象速查表

创建实例后，以下对象可直接使用（按最常用排序）：

| 属性 | 类型 | 主要用途 |
|---|---|---|
| `map.scene` | `Glodon.CIMCube.Scene` | 场景管理：加光源、真实感渲染（Bloom / 体积云 / 大气 / 雾）、后期效果 |
| `map.camera` | `Glodon.CIMCube.Camera` | 相机控制：`flyTo` / `flyToBoundingSphere` / `getCaptureImg` |
| `map.globe` | `Glodon.CIMCube.Globe` | 地球对象：地形配置、椭球属性 |
| `map.layerManager` | `Glodon.CIMCube.Layer.LayerManager` | 图层管理：影像 / 地形 / 3DTiles 等所有图层增删改查 |
| `map.baseMap` | `Glodon.CIMCube` 的默认底图设置 | 初始化默认底图（见 `constructor-options.md` 的 baseMap 节） |

> ⚠️ 本技能**只暴露 CIMCube 封装层**：请用上表的 `map.scene / map.camera / map.globe / map.layerManager` 等子系统对象完成业务；引擎内核原生对象及其下级接口不在本技能文档范围内，也不要在业务代码里依赖。

**APIDOC 其余实例成员**（可读写，按需使用）：`hardwareAA`（硬件抗锯齿）、`logarithmicDepthBuffer`、`screenSpaceEventHandler`、`shadowQuality`、`shadowRange`、`terrainShadows`、`useDefaultLabel`、`useDefaultTerrain`、`usePostProcessWater`。

## 二、按坐标飞行/定位：用 map.camera（⚠️ 不是 map.flyTo）

> **实测区分**：`map.flyTo(...)` 只接受**实体/数据源**（Entity / DataSource / ImageryLayer），传经纬度对象**不会移动相机**；
> 按经纬度飞行请使用 `map.camera.flyTo` / `map.camera.setView`，目标点字段为 **`lon / lat / height`**，姿态 `orientation` 为**弧度制**。

```js
// ① 平滑飞到北京，3km 高度，45° 俯视（target 用 lon/lat/height；flyToPoint 表示精确飞到该点）
map.camera.flyTo({
  target: { lon: 116.4074, lat: 39.9042, height: 3000 },
  flyToPoint: true,
  orientation: { heading: 0, pitch: -Math.PI / 4, roll: 0 }, // 弧度制
  duration: 3,
});

// ② 无动画直接定位（destination 用 lon/lat/height）
map.camera.setView({
  destination: { lon: 116.4074, lat: 39.9042, height: 3000 },
  orientation: { heading: 0, pitch: -Math.PI / 4, roll: 0 },  // 弧度制
});

// ③ 飞到某个图层（target 直接传图层，自动取包围球，无需 flyToPoint）
const tiles = map.layerManager.getLayer("my-s3m-tiles");  // ⚠️ 无 getLayerById，用 getLayer(id)
map.camera.flyTo({ target: tiles, duration: 2 });

// ④ 实体/数据源飞行才用 map.flyTo
map.flyTo(entityOrDataSource);
```

> ⚠️ 不要对相机 API 传 `{x, y, z}` 经纬度：`{x, y, z}` 会被当作 ECEF 笛卡尔坐标（米），相机将飞到错误位置（实测传北京经纬度会跑到极地附近）。
> 字段约定速记：**位置统一用 `lon / lat / height`（度/度/米）；姿态 `center.heading/pitch/roll` 与 camera 的 `orientation` 均为弧度制**（`-45°` 写 `-Math.PI/4`）。

## 三、APIDOC 收录的实例方法速览

CIMCube 主类同时混入 EventDispatcher（继承自 Cesium 事件体系），事件/监听与功能性方法：

| 分类 | 方法 | 说明 |
|---|---|---|
| 事件 | `on` / `off` / `addEventListener` / `removeEventListener` / `removeEventListeners` / `hasEventListener` / `dispatchEvent` / `fire` | 监听与派发事件（图层生命周期事件见 **cimcube-layer**） |
| 定位 | `flyTo` | 飞行定位（实测：主要面向实体/数据源，经纬度飞行请用 `map.camera.flyTo`） |
| 拾取 | `getFeaturesAtPixel(windowPosition, {tolerance=3, layers})` | 屏幕坐标拾取要素 |
| 渲染调试 | `setDebugShowPostRender(show)` | 开关逐帧后处理调试渲染 |
| 资源 | `releaseCachedResources()` | 释放缓存资源 |
| 地形 | `enableDefaultTerrain()` / `disableDefaultTerrain()` | ⚠️ APIDOC 已标记 **Deprecated**；请用 `useDefaultTerrain` 属性或 `params.useDefaultTerrain` |
| 释放 | `dispose()` | 释放全部 WebGL 资源（见下） |

**静态方法 / 常量**（挂在类上）：

| 方法 | 说明 |
|---|---|
| `Glodon.CIMCube.createScene(map, options)` | ⚠️ **关键方法**：为已有 CIMCube 实例应用场景管理配置。`options` 结构为 `{ globalSettings, mapLayers, defaultView }`——**这三个字段构造器不处理**，必须通过此方法应用（源码：内部调 `parseSceneSetting(globalSettings)` + `layerManager.addSceneLayers(mapLayers)` + `_flyToDefaultView(defaultView)`）。CIMCube 场景管理分享页（`token=xxx`）导出的 JSON 就是这个结构，直接传给第二个参数即可。 |
| `Glodon.CIMCube.setMultiThreadRequestingEnabled(bool)` | 开关多线程请求 |
| `Glodon.CIMCube.setOcclusionCullingEnabled(bool)` | 开关遮挡剔除 |
| `Glodon.CIMCube.VersionInfo` | 当前 SDK 版本信息 |
| `Glodon.CIMCube.version` | 只读版本号（**与上面的 `VersionInfo` 是两个东西**：`version` 是版本号字符串，`VersionInfo` 是版本信息对象） |

## 四、map.dispose()

销毁实例、释放 WebGL 上下文与 DOM 监听器。单页应用切换页面时**必须调用**，否则会造成显存泄漏。

```js
// React/Vue 组件卸载时调用
window.addEventListener("beforeunload", () => map.dispose());
```

配合 `releaseCachedResources()` 可在 dispose 前先清缓存资源；SPA 路由切换推荐「先停业务（图层/动画）→ `releaseCachedResources()` → `dispose()`」的顺序。
