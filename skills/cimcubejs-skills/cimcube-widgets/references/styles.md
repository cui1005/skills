# Widgets 样式加载与覆盖

> 归属 `cimcube-widgets`。本文件是**查阅型**清单：写挂载代码前按需查 CSS 文件与锚点类名。
> 公共初始化与组合模式见 `references/patterns.md`；误用对照与性能见 `references/pitfalls.md`。

## 1. 引入清单

> ⚠️ 引擎根默认 `v3.**.0/dist/`（**发布包不含 SDK**；SDK 解压位置不同时把此前缀整体替换，路径基准是 HTML 所在目录）。

```html
<link rel="stylesheet" href="v3.55.0/dist/cimcube/Widgets/widgets.css">   <!-- Cesium 内核基础样式（必引，缺它 canvas 卡在 300×150） -->
<link rel="stylesheet" href="v3.55.0/dist/Resource/css/Widgets.css">      <!-- ★ CIMCube 全部控件样式，只需这一个 -->
<script src="v3.**.0/dist/cimcube/cimcube.js"></script>
```

> ⚠️ **不要再单独引** `Compass.css` / `StatusBar.css` / `LayerControl/LayerControl.css` / `Geocoder.css` / `DistanceLegend.css` / `MapLegend.css` / `MapSplit.css` / `CameraRoam.css` / `Div.css` / `DrawEdit.css` / `Draggable-resizable-dialog.css` / `LayerControl/MixinMain.css`：
> `Widgets.css`（≈18 KB）就是这些文件的**聚合超集**——SDK v3.55.0 实测：把 `dist/Resource/css/` 下 13 个分册逐条规则归一化后与 `Widgets.css` 比对，**0 条缺失**；浏览器里只引 `Widgets.css` 与全引时的计算样式**逐项一致**。重复引入不改变渲染，只多几个 HTTP 请求。
> 分册仍保留在 SDK 里，用于**对照/覆写参考**（想确认某控件某项默认值时可直接翻对应小文件）。

## 2. CSS 与 DOM 锚点对照

样式实际都在 `dist/Resource/css/Widgets.css`（下表「CSS 文件」列是 SDK 里的同名分册，便于定位规则来源）：

| 组件 | CSS 文件 | 主要锚点 |
|---|---|---|
| 罗盘 | `Compass.css` | `.cimcube-compass`、`.cimcube-out-ring`、`.cimcube-gyro`、`.cimcube-zoom-controller` |
| 地名搜索 | `Geocoder.css` | `.cimcube-geocoderContainer` |
| 状态栏 | `StatusBar.css` | `.cimcube-status-bar`、`.cimcube-location` |
| 距离比例尺 | `DistanceLegend.css` | `.cimcube-distance-legend`、`.cimcube-label`、`.cimcube-scale-bar` |
| 图层管理（初始化参数开启） | `LayerControl/LayerControl.css`（+ `Widgets.css` 的定位宿主规则） | **`.cimcube-widget-container`（定位宿主，默认 `top:10px/left:10px/width:200px`）**、`.cimcube-widget-layerControl`、`.cimcube-widget-treeNode`、`.cimcube-widget-position` |
| 图例 | `MapLegend.css` | `.cimcube-maplegend-container/-head/-body`、`.cimcube-accordion`、`.cimcube-legend` |
| 卷帘 | `MapSplit.css` | `.cimcube-mapsplit-container`、`.cimcube-mapsplit-slider(-horizontal/-vertical)`、`.cimcube-mapsplit-switch` |
| 对话框（父类用） | `Draggable-resizable-dialog.css` | `.cimcube-drag-resize-dialog`、`.cimcube-titlebar` |

## 3. 覆盖定制

**别改 SDK 文件**，放你自己的样式表且置于 SDK CSS 之后：

```css
.cimcube-status-bar { background: rgba(13, 71, 161, .9) !important; }   /* StatusBar.css 类 */
.cimcube-compass { right: 180px; }                                       /* Compass.css：换位 */
.cimcube-geocoderContainer { top: 60px; }                                /* Geocoder.css：换位 */
.cimcube-widget-container { left: auto; right: 20px; top: 120px; }       /* LayerControl 换位（定位宿主是外层容器，不是 .cimcube-widget-layerControl） */
.cimcube-maplegend-container { bottom: 50px; }                           /* 图例上移避开其它面板 */
.cimcube-mapsplit-slider-horizontal { background: #e6a23c; }             /* 分隔条高亮 */
```
