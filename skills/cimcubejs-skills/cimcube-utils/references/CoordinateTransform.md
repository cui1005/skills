# Utils 静态计算 — CoordinateTransform / SightLimitCalculate（纯静态）

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

## 一、CoordinateTransform — EPSG 坐标转换（纯静态）

**没有** `new CoordinateTransform(map)` 后的屏幕↔经纬度↔笛卡尔、度分秒等 API——杜撰（无此 API）。真实类仅一个静态方法（内部基于 proj4，已预注册常用投影定义）：

```js
// 经纬度 → CGCS2000 高斯投影 / Web 墨卡托 / 其他 EPSG 代号
const xy = U.CoordinateTransform.epsgTranslate([116.4, 39.9], "EPSG:4326", "EPSG:4490"); // CGCS2000 地理
const m  = U.CoordinateTransform.epsgTranslate([116.4, 39.9], "EPSG:4326", "EPSG:3857"); // Web 墨卡托
// returns Array.<Number>（如 [lon, lat] 或 [x, y]）
```

> 屏幕像素↔经纬度用 `map.controls` 取点或 Cesium `map.scene.pickPosition` 自己算；度分秒格式化用 `Global.Math`（见 cimcube-global）。EPSG 对子相同可缓存。

## 二、SightLimitCalculate — 视线限制静态计算

**纯静态**（名字易误导为遮挡分析，实际是**沿线里程**工具）。`calculate(startPoint, endPoint, pointOnLine)`：三个 `Graphic.GeoPoint`（有 `.lon/.lat/.height`），求线上某点距首尾的直线距离与**水平**（同高度）距离；共线校验失败抛 `Cesium.RuntimeError`（“传入点不在线上”）。

```js
const r = U.SightLimitCalculate.calculate(geoStart, geoEnd, geoPointOnLine);
// r = { lengthFromStartPoint, lengthFromEndPoint,       // 直线距离（米）
//       horLengthFromStartPoint, horLengthFromEndPoint } // 水平距离（米）
```

> 典型用途：沿管线/断面的桩号定位、视线限制判定。需要“视点→模型被遮挡比例”的**数值统计并不存在**（杜撰）；可视类分析走 `Analysis.ViewShedAnalysis` 等（cimcube-analysis），并自行读结果。

## 性能

| 工具 | 开销 | 建议 |
|---|---|---|
| CoordinateTransform / SightLimitCalculate | 极低（纯函数） | 可高频调用；EPSG 对子相同可缓存 |
