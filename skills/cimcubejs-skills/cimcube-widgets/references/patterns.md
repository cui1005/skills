# Widgets 公共模式（初始化 · 组合 · 卸载）

> 本文件是**写前必读**的公共套路。
> 逐控件细节见同目录 `<控件>.md`；CSS 引入与样式覆盖见 `references/styles.md`；误用对照与性能见 `references/pitfalls.md`。

## 1. 公共初始化

> CSS/JS 引入见 `references/styles.md`；`W`/`DEF`/`map` 创建样板见 `../SKILL.md` 开头 js 块。

一键挂载常用浮层（Quick Start）：

```js
// 图层管理面板不在这里挂：初始化时写 `params: { layerControl: true }` 即可
// （面板由 CIMCube 自动挂载，再 new 一次会出现两个叠层面板）
new W.CompassWidget().addToMap(map);                                          // 无返回值：显隐走 .cimcube-compass
const statusBar = new W.StatusBar({ coordinate: true, azimuth: true, pitch: true, fps: true }); statusBar.addToMap(map); // 返回 undefined：句柄即 statusBar 本身
new W.GeocoderWidget(DEF.GeocoderMode.NOOFFSET).addToMap(map);                // 无返回值：走 .cimcube-geocoderContainer
const legend = new W.DistanceLegend(); legend.addToMap(map);                 // 返回 undefined：句柄即 legend 本身
```

## 2. 常见模式

### 模式 A：标准浮层骨架 + 卸载

```js
function mountOverlay(map, container) {
  new W.CompassWidget().addToMap(map);                  // mixin 包装：无返回值，走 DOM
  const statusBar = new W.StatusBar(); statusBar.addToMap(map);    // 返回 undefined：句柄即 statusBar 本身
  new W.GeocoderWidget(DEF.GeocoderMode.NOOFFSET).addToMap(map); // 无返回值，走 DOM
  const distance = new W.DistanceLegend(); distance.addToMap(map); // 返回 undefined：句柄即 distance 本身
  const roots = () => Array.from(container.querySelectorAll(
    ".cimcube-compass, .cimcube-status-bar, .cimcube-geocoderContainer, .cimcube-distance-legend, .cimcube-widget-container"
  ));
  return {
    statusBar, distance,
    hideAll: () => roots().forEach((el) => { el.style.visibility = "hidden"; }),
    showAll: () => roots().forEach((el) => { el.style.visibility = ""; }),
    teardown() {                                              // SPA 路由离开：停取数 + 摘 DOM
      statusBar.enabled = false;               // 停用即停内部服务
      distance.enabled = false;
      roots().forEach((el) => el.remove());    // 罗盘/搜索框一并摘除
    },
  };
}
```

> 只有 `StatusBar / DistanceLegend`（BaseWidget 子类）有 `enabled`（控制 DOM 挂/卸）；`CompassWidget / GeocoderWidget` 是 mixin 包装（`addToMap` 无返回值，内部实例**不对外暴露**），一律**直接移除根元素**（`.cimcube-compass` / `.cimcube-geocoderContainer`），勿重复 `addToMap`。

### 模式 B：搜索框布局 / 换源（只使用导出的 GeocoderWidget）

```js
new W.GeocoderWidget(DEF.GeocoderMode.OFFSET).addToMap(map);
// 不接受自定义 container，固定生成 .cimcube-geocoderContainer（左上）：需要句柄就操作这个根元素
// 改位置/观感 → 覆盖 Geocoder.css；换源/完全自绘 → Utils.POISearchService（见 cimcube-utils references/Services.md）。
```

### 模式 C：百分比沉浸模式（整体显隐浮层 DOM）

```js
function toggleImmersive(immersive, container) {
  container.querySelectorAll(".cimcube-compass, .cimcube-status-bar, .cimcube-geocoderContainer, .cimcube-distance-legend")
    .forEach((el) => { el.style.display = immersive ? "none" : ""; });
}
```

### 模式 D：一套面板 + 业务图层联动

```js
function setupPanels(map, { compareLayerId }) {
  // ⚠️ 图层管理面板这里【不用挂】：CIMCube 构造时已自动挂载，显隐由构造参数 params.layerControl 控制
  //    （再 new 一次 LayerControlWidget 会出现两个叠层面板）
  // ★ addTo / addToMap 一律返回 undefined，须先持实例再挂载，否则 panels.legend/split 为 undefined
  const legend = new W.MapLegendWidget({
    title: "业务图例",
    sources: [makeLegendSource(map.layerManager.getLayer(compareLayerId))],
  });
  legend.addTo(map);
  const split = new W.MapSplitWidget(map.layerManager, {
    mode: DEF.ViewPortModeType.VERTICAL,
    viewPortLayersVisible: [{ id: compareLayerId, visible: true }],
  });
  split.addToMap(map);
  const editor = new W.DragModel(map, { enableScaleX: true, enableScaleY: true, enableScaleZ: true });
  editor.activate();
  const panels = { legend, split, editor };

  // 依据某图层的样式对象拼图例数据源（按运行时键）
  function makeLegendSource(layer) {
    const g = (layer.style && layer.style.geometryType) || "POLYGON";
    return { layerID: layer.id, layerName: layer.name || layer.id, style: { geometryType: g, fillColor: layer.style?.fillColor, color: layer.style?.color, outline: layer.style?.outline } };
  }
  return panels;
}
```

### 模式 E：图例点击联动层显隐（图例即开关）

`MapLegendWidget` 已内置：均匀源点击切整层，分类源（`style.items` + `style.field`）点击单值行只显隐命中该值的要素；业务只需组织好数据源（`legend.dataSourceAdded` 可监听新增项），无需自接 picking。

### 模式 F：批量卸下 panels

```js
function teardownPanels(map, panels, container) {
  panels.split.getMapSplit()?.dispose();     // 停卷帘
  panels.editor.deactivate(); panels.editor.dispose();
  panels.legend.dispose();                    // 清 DOM + map.mapLegend
  // 图层管理面板（自动挂载）/ MapSplitWidget 等面板 DOM：
  container.querySelectorAll(
    ".cimcube-widget-layerControl, .cimcube-widget-container, .cimcube-mapsplit-external-container, .cimcube-maplegend-container"
  ).forEach((el) => el.remove());
}
```
