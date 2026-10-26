# 构造参数 opts 详解（`new Glodon.CIMCube(opts)`）

> 本文件对应 SKILL.md 路由表的「写构造参数：全字段表、内置底图类型。

> ⚠️ **关键区分**（源码已确认）：`new Glodon.CIMCube(opts)` 构造器**只处理**以下 5 个字段——`id` / `gisResourceUrl` / `center` / `baseMap` / `params`。
>
> ❌ **以下 3 个字段构造器会静默忽略**，必须通过静态方法 `Glodon.CIMCube.createScene(map, options)` 应用：
> - `globalSettings` — 真实感渲染、场景环境、天空盒、AO 等预置
> - `mapLayers` — 影像底图 + 3D Tiles + 矢量瓦片等图层数组
> - `defaultView` — 初始视角（场景管理分享页的视角字段）
>
> 如果你的配置来自 CIMCube 场景管理分享页（`token=xxx` 的 JSON），请把整个 JSON 传给 `createScene(map, json)`，**不要**塞进构造器。

`new Glodon.CIMCube(opts)` — `opts` 为对象，以下为最常用字段（构造器实际处理的）：

| 字段 | 类型 | 必填 | 默认 | 说明 |
|---|---|---|---|---|
| `id` | `String` | ✅ | — | DOM 元素 ID（或直接传 HTMLElement）。容器**高度不能为 0**（否则 canvas 高 0、白屏）；实测不写宽高也能出图但尺寸不可控，建议显式 `width:100%;height:100%`（完整对照与排查口诀见 pitfalls.md「容器尺寸陷阱」）。 |
| `gisResourceUrl` | `String` | ✅ | — | **Resource 目录的父目录**相对路径——SDK 内部会自行拼接 `Resource/SkyBox/...`、`Resource/luts_data/...` 去加载天空盒、LUT、瓦片等静态资源。**结尾必须带 `/`**。例：Resource 在 `./dist/Resource/` 时填 `"./dist/"`（填成 `"./dist/Resource/"` 会请求出 `Resource/Resource/...` 导致 404）。 |
| `center` | `Object` | — | 全球视角 | 初始相机位置 + 姿态。位置用度/米，姿态用**弧度**（见下）。 |
| `center.lon` | `Number` | — | `116.41667` | 经度（十进制度，WGS84）★ 字段名是 `lon`，不是 `x` |
| `center.lat` | `Number` | — | `39.91667` | 纬度（十进制度，WGS84）★ 字段名是 `lat`，不是 `y` |
| `center.height` | `Number` | — | `2.0e7`（2 万公里全球视角） | 高度（米，相对于椭球面）★ 字段名是 `height`，不是 `z`。字段名写错时静默回落到默认值 |
| `center.heading` | `Number` | — | `0` | 航向角（**弧度 rad**）：0=北 / `Math.PI/2`=东 / `Math.PI`=南 / `3*Math.PI/2`=西 |
| `center.pitch` | `Number` | — | `-Math.PI/4`（即 -45°） | 俯仰角（**弧度 rad**），负向下：`-Math.PI/2` 正俯视、`-Math.PI/4` 斜视、`0` 水平 |
| `center.roll` | `Number` | — | `0` | 翻滚角（**弧度 rad**），一般保持 0 |
| `baseMap` | `Object` | — | 无底图 | 默认底图配置（见下节「baseMap」）。 |
| `params` | `Object` | — | `{}` | 场景配置参数（见下节「params」）。 |

> ⚠️ `center.heading/pitch/roll` 必须传**弧度**（构造器原样透传给 Cesium `camera.setView`，**不做度→弧度转换**）。完整实测（如 `pitch:-45` 实际 ≈ -58.31°）与正确写法见 pitfalls.md「center 弧度陷阱」。

## baseMap 字段（构造即挂默认底图，APIDOC 支持）

```js
const map = new Glodon.CIMCube({
  id: "cimcubeContainer",
  gisResourceUrl: "./dist/",
  baseMap: {
    show: true,
    name: "天地图影像",
    type: Glodon.CIMCube.Global.Defines.LayerType.IMAGERY, // ★ 仅支持 imagery 类型
    imageryType: Glodon.CIMCube.Global.Defines.ImageryLayerType.TDT_IMG, // 等价字符串 "tdt_img"
    minimumLevel: 1,
    maximumLevel: 18,
    // 需访问令牌的底图可覆盖默认 token：
    // accessToken: { field: "tk", value: "你的KEY" }
  },
});
```

`imageryType` 内置底图类型（`Global.Defines.ImageryLayerType`）：

| 厂商 | 取值 | 说明 |
|---|---|---|
| 天地图 | `TDT_VEC` / `TDT_IMG` / `TDT_TER` | 矢量 / 影像 / 地形 |
| 天地图注记 | `TDT_CVA` / `TDT_CIA` / `TDT_CTA` | 矢量注记 / 影像注记 / 地形注记 |
| 高德 | `GAODE_IMG` / `GAODE_VEC` / `GAODE_CIA` | 影像 / 矢量 / 注记 |
| 百度 | `BAIDU_IMG` / `BAIDU_VEC` | 影像 / 矢量 |

> 等价于实例化后用 `layerManager` 添加 `ImageryLayer`（配方见 `cimcube-layer/references/imagery-terrain.md`）。不传 `baseMap` 时默认不带影像图层。

## params 字段（场景/控件开关，APIDOC）

| 字段 | 说明 |
|---|---|
| `sceneMode` | 场景模式（3D / 2D / 2.5D） |
| `mapProjection` | 地图投影（2.5D / 2D 场合；默认地理投影） |
| `terrainUrl` | 地形服务 URL |
| `requestVertexNormals` | 地形服务是否请求顶点法线 |
| `shadows` | 开启光源投影与阴影接收 |
| `shouldAnimate` | 是否启用 Cesium 动画时钟 |
| `useDefaultTerrain` | 使用默认地形 |
| `useDefaultLabel` | 开启默认内置注记图层 |
| `layerControl` | **图层管理面板显示开关**（默认 `false`）——面板由 CIMCube 构造时**自动挂载**（源码：`new LayerControlWidget({layerManager}).addToMap(this)`），`false` 时只是被置 `display:none`；置 `true` 才显示。业务**不要再** `new LayerControlWidget(...)`，否则会出现两个叠层面板 |
| `fullscreenButton` / `homeButton` / `navigationHelpButton` / `sceneModePicker` | Cesium 对应工具按钮开关 |

## 容器尺寸配置陷阱

> ⚠️ 容器高度塌陷为 0 会白屏/黑屏、缺 `Widgets/widgets.css` 会让 canvas 卡在默认 300×150。完整实测对照表（含「不写宽高也能出图但不可控」「明确宽高推荐」等情形）与排查口诀见 pitfalls.md「容器尺寸陷阱」。
