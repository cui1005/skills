# 性能建议与常见问题排查

> 本文件对应 SKILL.md 路由表的「调优与排错」行。

> 相关文件：构造参数 → `constructor-options.md`；实例 API → `instance-api.md`；脚手架 → `quick-start.md`

## 一、资源路径与裸球黑屏（两条高频约定）

> ⚠️ 资源路径约定：`gisResourceUrl` 指向 **Resource 的父目录**（该目录下应能看到 `Resource/SkyBox/`、`Resource/offline/`、`Resource/luts_data/` 等）。若控制台出现 `Resource/Resource/SkyBox/...` 类 404，说明误填成了 Resource 目录本身。

> ⚠️ 裸初始化（不加任何图层）时地面是**黑色**的：默认不挂影像图层且地球基色为黑，需通过 LayerManager 自行添加底图，属正常现象。

## 二、性能建议

1. **容器尺寸匹配设备像素比**：HiDPI 屏幕默认按 `devicePixelRatio` 渲染，FPS 低时可手动降低抗锯齿/像素比（见实例成员 `hardwareAA`）。
2. **按需禁用默认地形**：纯 2D / 管理后台等静止画面场景，用 `params.useDefaultTerrain=false` 等开关精简默认能力（APIDOC 目前未披露 requestRenderMode 类底层开关）。
3. **及时 dispose**：SPA 路由切换必须调 `dispose()`，避免 WebGL 上下文累积。
4. **gisResourceUrl 走 CDN**：Resource 目录（天空盒、LUT）在生产环境放 CDN，首屏更快。
5. **不要在循环中重复创建实例**：初始化是重操作，单页单实例最佳。

## 三、常见问题排查表

| 现象 | 最可能原因 | 解决 |
|---|---|---|
| 黑屏/白屏、无任何报错 | 容器高度为 0 | 给 `#cimcubeContainer` 设置 `height:100vh` 或明确像素高 |
| canvas 只有 **300×150**、画面缩在左上角 | 缺少 Cesium 内核样式表 | 在 `<head>` 引入 `<link rel="stylesheet" href="./dist/cimcube/Widgets/widgets.css">`（官方示例必引） |
| 控制台报 404 且 URL 中出现 **`Resource/Resource/SkyBox/...`** | `gisResourceUrl` 填成了 Resource 目录本身 | 改填 Resource 的**父目录**（如 `"./dist/"`），结尾带 `/`；正确路径下应能访问到 `Resource/SkyBox/` |
| 相机停在北京 2 万公里高空（116.41667, 39.91667, 2e7m），`center` 没生效 | `center` 字段名写成了 `x/y/z` 或 `lng/lat` | 使用 **`lon=经度, lat=纬度, height=高度(米)`**；字段名错误时静默回落默认值 |
| `center.pitch: -45` 实际视角却是 **-58.3°** 左右（heading 传 90 也会偏到 ~116.6°） | 构造器不做度→弧度转换，`center.heading/pitch/roll` 按**弧度**解释 | 姿态传弧度：`pitch: -Math.PI/4`（45°）、`pitch: -Math.PI/2`（正俯视）；不写 pitch 时默认就是 -45° |
| 地面/地球全黑但星空正常 | 裸初始化默认无影像图层（`imageryLayers=0`），地球基色为黑 | 属正常现象，通过 LayerManager 加载 ImageryLayer/离线瓦片底图即可 |
| 页面切换后显存持续上涨 / 报 WebGL context 过多 | 未调用 `map.dispose()` | SPA 路由切换或组件卸载时调用 `dispose()`（见 `instance-api.md`） |
| `map.flyTo({lon, lat, height})` 相机不动 | `map.flyTo` 只收实体/数据源（Entity / DataSource / ImageryLayer） | 经纬度飞行改用 `map.camera.flyTo` / `map.camera.setView`（见 `instance-api.md`） |
| **图层数为 0**（`LM.getAllLayers().length === 0`）、真实感渲染没开、视角不飞到预设位置 | `globalSettings` / `mapLayers` / `defaultView` 三个字段**构造器不处理**，直接塞进 `new CIMCube(opts)` 会被静默忽略 | 构造器只负责基础创建，场景管理配置走 `Glodon.CIMCube.createScene(map, { globalSettings, mapLayers, defaultView })`；场景管理分享页导出的 JSON 整体传给第二个参数即可 |

## 四、构造参数实测陷阱（详细证据）

以下为第三节排查表中「容器尺寸」（见第 25–26 行）与「center 弧度」（见第 29 行）相关现象行的完整实测数据，供定位时对照。字段层面的速记见 `constructor-options.md`。

### 容器尺寸陷阱

> 常见说法「容器必须有明确宽高，否则黑屏/空白」**并不准确**。按运行时实测分开看：

| 情形 | 实测结果 |
|---|---|
| 容器**完全不写**宽高（裸 `<div id="cimcubeContainer"></div>`，块级） | ✅ **仍然出图**：宽度由块级自动撑满父级，高度被内容撑开（实测容器 1905×958、canvas 1905×953＝视口尺寸）。但尺寸由视口/内容决定、**不可控**，不推荐依赖 |
| 容器**给明确宽高**（如 400×300） | ✅ canvas 精确等于容器（实测 400×300）——**推荐**，行为可预期 |
| 容器**高度为 0**（`height:0`、父级 flex 没给高、绝对定位没给高、外层 `overflow:hidden` 压成 0） | ❌ **白屏/黑屏**：实测容器 1920×0 → canvas 1920×0，什么都不渲染 |
| 缺 `<head>` 的 `cimcube/Widgets/widgets.css` | ❌ canvas **卡在默认 300×150**（实测容器 1920×1080 时 canvas 仍为 300×150） |

```html
<!-- ✅ 推荐：显式给尺寸，行为可预期 -->
<div id="cimcubeContainer" style="width:100vw;height:100vh;position:absolute;top:0;left:0"></div>

<!-- ⚠️ 能出图，但尺寸退化为"视口大小"，不要依赖 -->
<div id="cimcubeContainer"></div>

<!-- ❌ 真正会导致白屏/黑屏：容器高度塌陷为 0 -->
<div id="cimcubeContainer" style="height:0"></div>
<div style="display:flex;overflow:hidden"><div id="cimcubeContainer" style="flex:0 0 0"></div></div>
```

**排查口诀**：白屏/黑屏先看**容器高度是不是 0**（`getComputedStyle(el).height` 与 `el.clientHeight`）；canvas 缩在左上角且只有 300×150 则是**缺 Cesium 内核样式表**。

### center 弧度陷阱

> ⚠️ 构造器把 `center.heading/pitch/roll` **原样透传**给 Cesium `camera.setView({orientation})`，**没有做角度→弧度转换**——必须传弧度。
> 不传 pitch 时默认视角确实是 -45°（SDK 内部默认值用了 `toRadians(-45)`）；但一旦自己写 `pitch: -45`，-45 会被当成 **-45 弧度**，归一化后实际 pitch ≈ **-58.31°**（heading 同理：`heading: 90` 实际 ≈ 116.6°）。
> 正确写法：`pitch: -Math.PI / 4`（45°俯视）、`pitch: -Math.PI / 2`（正俯视），或用 `Glodon.CIMCube.Global.Math` 的角度转换工具。
