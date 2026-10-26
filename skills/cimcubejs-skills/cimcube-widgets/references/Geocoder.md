# GeocoderWidget 地名搜索框

> 归属 `cimcube-widgets`。本文件只写**导出的 `GeocoderWidget`**；DOM/样式锚点见 `references/styles.md`，公共初始化与组合模式见 `references/patterns.md`，误用对照见 `references/pitfalls.md`。

## 1. 挂载（必须传 geoCoderMode）

```js
const W = Glodon.CIMCube.Widgets, DEF = Glodon.CIMCube.Defines;
// geoCoderMode：Defines.GeocoderMode.NOOFFSET = 0（天地图）| OFFSET = 1（高德）
new W.GeocoderWidget(DEF.GeocoderMode.NOOFFSET).addToMap(map);
// 不传 / undefined 时内部走 OFFSET（高德）
```

⚠️ **必须传 `geoCoderMode`**：不传不会报错，但坐标系按高德（OFFSET）解算，配天地图底图会偏移。

## 2. 句柄与生命周期

`addToMap(map)` 会创建 `<div class="cimcube-geocoderContainer">` 追加到地图浮层容器，并在内部完成联想搜索与 `flyTo` 定位。

- `addToMap` **无返回值**（mixin 包装，内部实例不对外暴露）→ 没有 `enabled` / `show()` / `hide()` 可用。
- **显隐 / 移位 / 移除一律操作根 DOM** `.cimcube-geocoderContainer`：

```js
const el = document.querySelector(".cimcube-geocoderContainer");
el.style.display = "none";   // 隐藏
el.remove();                 // 彻底移除（勿重复 addToMap，详见 references/pitfalls.md「单例与句柄」条）
```

- **不接受自定义 `container`**：搜索框固定生成在左上，想改位置只能用 CSS 覆盖（见 §4）。

## 3. 换源 / 完全自绘搜索框

需要自定义容器、换 POI 数据源或自己控制查询时机时，不要用内部实现，改用 `Utils.POISearchService` 自行实现 UI（搜索/联想 API 以 **cimcube-utils** 的 `references/Services.md` 为准）。

## 4. 样式

`Geocoder.css`（已聚合进 `Widgets.css`，无需单独引）：`.cimcube-geocoderContainer`（左上搜索框），结果下拉在容器内渲染。覆盖请放自有样式表且置于 SDK CSS 之后：

```css
.cimcube-geocoderContainer { top: 60px; }   /* 换位 */
```

## 5. 坑点

| 误用 | 正确做法 |
|---|---|
| `new W.GeocoderWidget()` 不传参 | ⚠️ **必传** `geoCoderMode`；`NOOFFSET`(0)= 天地图，`OFFSET`(1)= 高德 |
| `new W.Geocoder({ container, ... })` | ❌ 内部类未挂命名空间（`new W.Geocoder` 为 undefined）；用 `GeocoderWidget` 或 `Utils.POISearchService` 自绘 |
| `const g = new W.GeocoderWidget(mode).addToMap(map); g.hide()` | ❌ `addToMap` 无返回值；直接操作 `.cimcube-geocoderContainer` |
| 传自定义 `container` | ❌ 不支持；换位置用 CSS，换 UI 用 `Utils.POISearchService` |
