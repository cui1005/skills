# Utils 服务类 — Compass / StatusBar / DistanceLegend / PerformanceListening / POISearch / MapLegendService

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。这些 Service 只出数据/状态，**UI 在 Widgets 侧**。

```js
const U   = Glodon.CIMCube.Utils;
const DEF = Glodon.CIMCube.Global.Defines;   // DrawEditMode / MapActionType / GeoCoderMode ...
```

## 一、CompassService — 罗盘（指北针）数据服务

`new U.CompassService(map)`。核心成员 `cameraHeading`（相机方位角，弧度）。事件通知用 `addEventListener({widget, method})`（**不是** `on("HEADING_CHANGE")`）。

```js
const cs = new U.CompassService(map);
const compassWidget = { update: () => {} };        // 自定义任意 widget 对象
cs.addEventListener({
  widget: compassWidget,
  method: () => {                                   // start() 后相机每帧变化被通知；可读成员做 UI
    document.getElementById("arrow").style.transform =
      `rotate(${(-cs.cameraHeading * 180 / Math.PI).toFixed(1)}deg)`;
  },
});
cs.start();                                         // 开启 heading 监听（postRender）
// cs.stop();
cs.faceNorth();                                     // 相机朝向正北
cs.rotateHeading(0.5);                              // 旋转 heading（弧度，正=顺时针）
cs.zoomIn(); cs.zoomOut();                          // 相机拉近/拉远 1 倍
cs.dispose();
```

> 旋转/俯仰状态机辅助：`calculateRotateAngle(rect, mousedownPoint, mousemovePoint)`、`startRotateHeading()/startRotatePitch(angle)/endRotatePitch()` 供拖拽旋转式 UI 用。现成 UI 在 `Widgets.Compass`/`CompassWidget`（cimcube-widgets）。

## 二、StatusBarService — 状态栏数据服务

`new U.StatusBarService(map)`（也接受 Cesium.Scene）。先 `addEventListener` 再 `start()`，preRender 节流 250ms 触发 `method`，参数**按序固定**：经度(°) → 纬度(°) → 椭球高(m) → 方位角(°) → 俯仰角(°) → 帧率 → 帧耗时。

```js
const ss = new U.StatusBarService(map);
ss.addEventListener({
  widget: statusBarDiv,
  method: (lng, lat, height, heading, pitch, fps, ms) => {
    statusBarDiv.innerHTML = `${lng.toFixed(6)}, ${lat.toFixed(6)} · ${height.toFixed(1)}m · h${heading.toFixed(1)}° p${pitch.toFixed(1)}° · ${fps}fps`;
  },
});
ss.start();
console.log(ss.fps, ss.ms);   // 未开启或引擎未提供时为 'NA'
ss.stop(); ss.dispose();
```

## 三、DistanceLegendService — 像素↔距离换算服务

`new U.DistanceLegendService(map)`；`start()`/`stop()` 启停后台换算服务，数据供**距离图例 UI**（`Widgets.DistanceLegend`，cimcube-widgets · references/DistanceLegend.md）使用。技能侧无回调事件；UI 侧自行读取换算结果。

## 四、PerformanceListening — 性能监听

`new U.PerformanceListening(map)`（无 options）。**全局只能开启一个**（挂 `rendererManager.performanceListening` 单点）。`start()` 后读取成员：

```js
const pl = new U.PerformanceListening(map);
pl.start();
// 成员（Number，引擎未提供为 'NA'）：pl.fps / pl.ms / pl.drawCalls / pl.numberOfVertices
pl.stop(); pl.dispose();
```

> 没有 UPDATE/WARN 回调事件（杜撰）；监控面板自己做定时器读取成员即可。

## 五、POISearchService — 地名 / POI 查询

**构造参数是 options（不含 map）**，两种服务后端由 `GeoCoderMode` 决定，accessToken 必须匹配：

```js
const poi = new U.POISearchService({
  geoCoderMode: DEF.GeoCoderMode.NOOFFSET,  // 0=天地图（无偏移，传天地图 token）
  // geoCoderMode: DEF.GeoCoderMode.OFFSET, // 1=高德（有偏移，传高德 token）
  accessToken: "你的 token",
  // server / geocoderServer / reverseGeocoderServer：可选，自建服务地址
});
poi.search("北京西站").then((raw) => console.log(raw));   // 原始查询结果（天地图 search2 / 高德 newpoisearch）
poi.geocode("北京西站");                                  // 地名定位（同底层）
poi.geocoding({ /* 地址→坐标，国内 */ });                 // 结构化地址 → 经纬度
poi.reverseGeocoding({ /* 经纬度 → 地址 */ });
poi.queryParameters = { /* 覆盖查询参数，字段见天地图/高德文档 */ };
```

> 无结果 list 包装/flyTo（UI 的 flyTo 在 `Widgets.Geocoder`/`GeocoderWidget`）。

## 六、MapLegendService — 图例图标资源服务

`new MapLegendService()`（无参工具，非服务常驻）。作用是**按符号/样式生成图例图标资源**，供 `Widgets.MapLegendWidget`（cimcube-widgets · references/MapLegend.md）或自绘图例面板使用；不是"由分析结果生成色阶 JSON 数组"。

```js
const legendRes = new U.MapLegendService();
const icon = await legendRes.getUrlResource(symbol);   // 图标符号 → Promise<String|Element> 图标资源
const svg  = await legendRes.getSvgResource(style);    // 矢量样式 → Promise<String|Element> svg 图标资源
// 拿到资源后填入图例 DOM 即可（或直接交给 MapLegendWidget）
```

## 常见模式：自定义状态条（Service 数据 → 自己的 DOM）

```js
const ss = new U.StatusBarService(map);
ss.addEventListener({ widget: bar, method: (lng, lat, h, hd, p, fps, ms) => render(bar, lng, lat, h, hd, p) });
ss.start();
```

## 性能

| 工具 | 开销 | 建议 |
|---|---|---|
| PerformanceListening | 极低 | 全局唯一；开发态常开、生产关闭 |
| CompassService / StatusBarService / DistanceLegendService | 极低 | `start/stop` 严格配对，结束 `dispose` |
| POISearchService | 低（网络 IO） | token 与服务模式匹配；结果自己控制频率 |

> 🌟 **三戒**：① `start/stop` 严格配对，结束要 `dispose`；② `addEventListener({widget, method})` 不是 `on("XXX")`；③ POISearchService 的 token 必须与服务模式（天地图/高德）匹配，否则搜不到结果。
