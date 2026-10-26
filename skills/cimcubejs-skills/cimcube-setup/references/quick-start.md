# 快速上手：从 0 到第一个地球

> 本文件对应 SKILL.md 路由表的「搭第一个页面：完整 HTML 脚手架、两种 SDK 加载方式、3D/2D 初始化模式、首屏自检清单、**引擎根三要素 + 静态服务器要求**（**写任何 SDK 路径前先读**）」行。

> 相关文件：构造参数字段表 → `constructor-options.md`；实例成员/方法 → `instance-api.md`；排错 → `pitfalls.md`

> ⚠️ **引擎根默认 `v3.**.0/dist/`**（SDK 解压根，含 `cimcube/cimcube.js` 与 `Resource/` 的那一层），本文件示例均以此为前缀，**基准是 HTML 所在目录**。
> **发布包不含 SDK**：把 SDK 解压成工作区下的 `v3.**.0/dist/` 即可直接照抄；放在别处时把 `v3.**.0/dist` 前缀整体替换（基准是 HTML 所在目录）。
> 页面另需**静态服务器**打开（`file://` 下 Resource 的 XHR 会被 CORS 拦）。

**引擎根 = 同时含下面这三样的那一层目录**（不多不少，也是「SDK 放对了没有」的唯一判据）：

| 相对引擎根的路径 | 用途 | 缺失后果 |
|---|---|---|
| `cimcube/cimcube.js` | 主 JS（**不是** `CIMCubeSDK.js`） | `Glodon` 未定义，页面空白 |
| `cimcube/Widgets/widgets.css` | Cesium 内核基础样式 | canvas 卡在默认 **300×150** |
| `Resource/` | 静态资源（天空盒 / LUT / 控件样式） | 资源 404、控件无样式 |

## 方式一：同步 script 标签加载（最简单，离线/内网部署必用）

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>CIMCube Quick Start</title>
  <!-- ★ Cesium 内核基础样式：必须引入！缺它 canvas 不会撑满容器（停留默认 300x150） -->
  <link rel="stylesheet" href="v3.55.0/dist/cimcube/Widgets/widgets.css">
  <!-- CIMCube 内置 Widgets 样式（指南针/图层控制等，按需引入） -->
  <link rel="stylesheet" href="v3.55.0/dist/Resource/css/Widgets.css">
</head>
<body>
  <!-- ★ 容器：高度务必给出（推荐 100vh）；高度为 0（父级 flex 未给高 / 绝对定位没给高）会导致 canvas 高 0、白屏 -->
  <div id="cimcubeContainer" style="width:100%;height:100vh"></div>

  <!-- SDK 主 JS（SDK 包内路径为 cimcube/cimcube.js，不是 CIMCubeSDK.js） -->
  <script src="v3.55.0/dist/cimcube/cimcube.js"></script>

  <script>
    // 1. 创建 CIMCube 实例 — 这一步之后 map.scene / map.camera / map.globe 即可用
    const map = new Glodon.CIMCube({
      id: "cimcubeContainer",                         // ★ DOM 容器 ID（必填）
      gisResourceUrl: "v3.55.0/dist/",                        // ★ Resource 的【父目录】（必填，SDK 内部会拼 "Resource/..."）
      center: {                                        // 可选：初始相机位置
        lon: 116.397428,                               //   经度（度）★ 字段名是 lon，不是 x
        lat: 39.90923,                                 //   纬度（度）★ 字段名是 lat，不是 y
        height: 3000,                                  //   高度（米）★ 字段名是 height，不是 z
        // ★ heading/pitch/roll 为【弧度制】：构造器原样透传给 Cesium setView，未做度→弧度转换
        heading: 0,                                    //   航向角（弧度）；0=北、Math.PI/2=东
        pitch: -Math.PI / 4,                           //   俯仰角（弧度）；-π/4=45°俯视、-π/2=正俯视、0=水平
        roll: 0,                                       //   翻滚角（弧度），一般 0
      },
    });
  </script>
</body>
</html>
```

## 方式二：CIMCubeSDKLoader（CDN 版本加载器，需联网 / 联调验证）

> ⚠️ **APIDOC 提示**：`Glodon.CIMCube.CIMCubeSDKLoader` 在 APIDOC 中为"SDK 版本加载器"，构造**无参**且文档**未记录公开成员/方法**；运行时导出该构造器且其原型含 `load()`，但加载 URL、重试策略等细节文档未披露、无法本地核实。
> 建议：**离线/内网部署一律用方式一的 `<script>` 标签**；确需 CDN 按版本加载时，参考 CIMCube 官方在线示例的调用方式，勿照搬下面未经验证的写法。

```js
// 运行时存在的用法（仅供参考，未经 APIDOC 与本地验证）：
const loader = new Glodon.CIMCube.CIMCubeSDKLoader();
loader.load().then(() => {
  // SDK 注入完成后 Glodon.CIMCube 可用
  const map = new Glodon.CIMCube({
    id: "cimcubeContainer",
    gisResourceUrl: "v3.55.0/dist/",   // 静态资源仍指向本地 Resource 的父目录
  });
});
```

## 常见初始化模式

### 模式 A：最小化 3D 地球（裸球，需自行加底图）

```js
const map = new Glodon.CIMCube({
  id: "cimcubeContainer",
  gisResourceUrl: "v3.55.0/dist/",
  center: { lon: 116.4, lat: 39.9, height: 5000, pitch: -Math.PI / 4 }, // ★ pitch 弧度：-π/4 = 45°俯视
  params: { layerControl: true },   // 可选：开启「图层管理」面板（面板由 CIMCube 自动挂载，勿手动 new）
});
// 裸初始化无影像图层（地面黑色属正常），之后按业务需要通过 LayerManager 加影像/地形/3D 瓦片层
```

### 模式 B：2D 地图模式（仅平面视图，节省资源）

```js
const map = new Glodon.CIMCube({
  id: "cimcubeContainer",
  gisResourceUrl: "v3.55.0/dist/",
  center: { lon: 116.4, lat: 39.9, height: 50000, pitch: -Math.PI / 2 }, // pitch=-π/2 正俯视（弧度）
  params: { sceneMode: "2D" }, // ★ APIDOC 字段：params.sceneMode / params.mapProjection（2D/2.5D 投影）
});
// 配合 2D 底图图层：ImageryLayer with XYZ/OSM provider
```

### 模式 C：还原 CIMCube 场景管理分享页（含图层 + 真实感渲染预置）

> ⚠️ **不要**把 `globalSettings` / `mapLayers` / `defaultView` 直接塞进构造器——构造器会忽略它们！必须通过 `Glodon.CIMCube.createScene(map, options)` 应用（见「七条铁律」第 7 条）。

```js
// 假设 sceneJson 是从 CIMCube 场景管理分享页拿到的完整 JSON（token=xxx 的响应）
// 它的结构就是 { defaultView, mapLayers, globalSettings, ... }
const sceneJson = /* 场景管理分享页导出的 JSON */;

// 第一步：构造基础实例
const map = new Glodon.CIMCube({
  id: "cimcubeContainer",
  gisResourceUrl: "v3.55.0/dist/",
  params: { layerControl: true },
});

// 第二步：用 createScene 应用分享页配置（三个字段在这里才生效）
Glodon.CIMCube.createScene(map, sceneJson);
// createScene 内部等价于：
//   parseSceneSetting(sceneJson.globalSettings, map)     → 应用真实感渲染/场景环境
//   layerManager.addSceneLayers(sceneJson.mapLayers)     → 加载底图 + 3D Tiles 等
//   _flyToDefaultView(sceneJson.defaultView, map)        → 飞到预设视角
```

## 首屏自检清单（出问题先按序过一遍）

0. **引擎根前缀正确**（默认 `v3.**.0/dist/`；SDK 放在别处时已把全篇前缀整体替换——前缀写错 = 一定 404），且页面经**静态服务器**打开而非 `file://`。
1. `<head>` 里引了 `v3.**.0/dist/cimcube/Widgets/widgets.css`（缺它 canvas 卡在 300×150）。
2. 容器 `#cimcubeContainer` 的**高度不为 0**（推荐 `height:100vh` 或像素高）。实测：不写宽高也能出图，但高度为 0 一定白屏（见 `references/constructor-options.md` 的「容器尺寸配置陷阱」）。
3. `gisResourceUrl` 填的是 **Resource 的父目录**且结尾带 `/`（`"v3.**.0/dist/"` 而非 `"v3.**.0/dist/Resource/"`）。
4. `center` 字段名是 `lon / lat / height`，姿态用弧度。
5. 地面全黑属正常——裸初始化无影像图层，需 `map.layerManager` 挂底图（见 `cimcube-layer`）。
