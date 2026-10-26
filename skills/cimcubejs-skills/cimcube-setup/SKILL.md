---
name: cimcube-setup
description: "CIMCube 实例初始化与场景搭建：CIMCubeSDKLoader 加载器、构造参数、核心属性、资源路径配置。当用户需要创建 CIMCube 实例、初始化三维地球或二维地图、配置容器、指定资源路径、设置初始相机位置，或释放资源时激活。"
---

# CIMCube 实例初始化与场景搭建

> **基线**：命名空间前缀 `Glodon.CIMCube.*`
> **定位**：本 SKILL.md 只做**索引路由 + 跨域铁律**。完整可运行页面、构造参数字段表、实例成员/方法、性能与排错全部外置于同目录 `references/*.md`。**写初始化代码前先读对应文件**，不要凭记忆重写示例。

CIMCube 是广联达三维地球/二维地图引擎的**主入口类**（`new CIMCube(opts)`），负责创建渲染上下文、加载资源，并暴露 scene / camera / globe / layerManager 等核心子系统对象。

## 一、七条铁律（最容易错）

1. **引擎根默认 `v3.**.0/dist/`**：示例里的 `cimcube/cimcube.js`、`Widgets/widgets.css`、`gisResourceUrl` 都以此为前缀（基准是 **HTML 所在目录**）。**引擎包不进发布包**——SDK 解压位置不同时，把此前缀整体替换成你的实际相对路径；**禁止**边写边猜。
2. **`gisResourceUrl` 填 Resource 的父目录且结尾带 `/`**：Resource 在 `v3.**.0/dist/Resource/` 时填 `"v3.**.0/dist/"`；填 `"v3.**.0/dist/Resource/"` 会请求 `Resource/Resource/...` 导致 404。
3. **`center` 位置字段是 `lon / lat / height`**（度/度/米），不是 `x/y/z`、不是 `lng/lat`；写错时**静默回落默认值**（北京 2 万公里高空）。
4. **`center.heading/pitch/roll` 是弧度**：构造器原样透传 Cesium，不做角度转换——`pitch: -45` 实际 ≈ **-58.31°**；45° 俯视写 `pitch: -Math.PI / 4`。
5. **容器高度不能塌陷为 0**（`height:0`、父级 flex 未给高、绝对定位没给高 → canvas 高 0 → 白屏/黑屏），且 `<head>` **必须**引 `v3.**.0/dist/cimcube/Widgets/widgets.css`（缺它 canvas 卡在默认 **300×150**）。两条均已实测。宽度块级会自动撑满；容器完全不写宽高时实测**也能出图**（canvas 退化为视口尺寸），但尺寸不可控，**照例写 `width:100%;height:100%` 最稳**。另外页面**必须走静态服务器**打开（`file://` 下 Resource 的 XHR 会被 CORS 拦）。
6. **经纬度飞行用 `map.camera.flyTo`**，不是 `map.flyTo`（后者只收实体/数据源，传经纬度不会移动相机）。
7. ⚠️ **`globalSettings` / `mapLayers` / `defaultView` 这三个字段构造器不处理**！它们由**静态方法** `Glodon.CIMCube.createScene(map, options)` 统一应用（源码：构造器内部 `_getMap` 创建实例后，`createScene` 调 `parseSceneSetting(globalSettings)` + `layerManager.addSceneLayers(mapLayers)` + `_flyToDefaultView(defaultView)`）。直接塞进 `new CIMCube({ globalSettings, mapLayers, defaultView })` 会**静默被忽略**——图层数为 0、真实感渲染不开、视角不飞到预设位置。

```js
// ★ 正确用法：构造器只负责基础创建，场景配置走 createScene
// 第一步：构造基础实例（id / gisResourceUrl / center / baseMap / params 走这里）
const map = new Glodon.CIMCube({
  id: "cimcubeContainer",                  // DOM 容器 ID（必填）
  gisResourceUrl: "v3.55.0/dist/",          // Resource 的父目录（必填，结尾带 /）
  center: { lon: 116.397428, lat: 39.90923, height: 3000, pitch: -Math.PI / 4 }, // 姿态弧度
  params: { layerControl: true },           // 可选：右上角「图层管理」面板
});

// 第二步：用 createScene 静态方法应用场景管理配置（globalSettings/mapLayers/defaultView 只在这里生效）
// 场景管理分享页导出的 JSON 就直接传给这一项
Glodon.CIMCube.createScene(map, {
  globalSettings: { /* 真实感渲染、场景环境等预置 */ },
  mapLayers:      [ /* 影像底图 + 3D Tiles 等图层数组 */ ],
  defaultView:    { /* 初始视角（lon/lat/height + heading/pitch/roll） */ },
});

map.scene; map.camera; map.globe; map.layerManager;  // 创建后即可用

```

> 裸初始化（不传 `baseMap`、不加图层）时地面为**黑色**属正常：默认无影像图层，需 `map.layerManager` 自行挂底图（见 **cimcube-layer**）。
> 如果场景配置来自 CIMCube 场景管理分享页（`token=xxx` 的 JSON），整个 JSON 对象直接作为 `createScene` 的第二个参数即可——分享页 JSON 的 `defaultView` / `mapLayers` / `globalSettings` 字段名与 `createScene` 的 `options` 完全对齐。

## 二、路由表（按需读取）

| 你要做什么 | 读这个文件 |
|---|---|
| 搭第一个页面：完整 HTML 脚手架、两种 SDK 加载方式、3D/2D 初始化模式、首屏自检清单、**引擎根三要素 + 静态服务器要求**（**写任何 SDK 路径前先读**） | `references/quick-start.md` |
| 写构造参数：全字段表、内置底图类型 | `references/constructor-options.md` |
| 用实例：核心对象速查（scene/camera/globe/layerManager）、可读写成员、方法表（事件/定位/拾取/资源）、静态成员、`dispose()` | `references/instance-api.md` |
| 调优与排错 | `references/pitfalls.md` |

## See Also

- **cimcube-camera** — `flyTo` 参数详解、`flyToBoundingSphere`、视图截图（`map.camera` 的完整能力）
- **cimcube-layer** — `map.layerManager`：裸球加底图/地形/3DTiles 的下一步（19 种图层加载指南）
- **cimcube-scene** — 场景环境：大气 / 天空盒 / 反射盒 / 雾 / 阴影 / 后期水 / AO
- **cimcube-realistic-rendering** — 真实感渲染（Bloom / 体积云 / 大气散射 / 高度雾 / 体积光 / 自动曝光）与光源
- **cimcube-physics** — 场景物理引擎 ScenePhysics
- **using-cimcube-skills** — 总览目录、命名空间树、SDK 加载方式（`using-cimcube-skills/references/sdk-loading.md` 与 `quick-start.md` 互补）
- **using-cimcube-skills/references/class-routing.md**（类目唯一真相源视图，自动生成）+ **using-cimcube-skills/references/domains-index.md**（总路由） — 根命名空间类目归属

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
