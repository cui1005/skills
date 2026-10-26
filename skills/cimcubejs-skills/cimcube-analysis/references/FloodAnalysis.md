# FloodAnalysis — 洪水淹没（体块 / 地形，可运行示例）

> 前提：`map` 已创建；`const A = Glodon.CIMCube.Analysis`；已通过控件/图元绘得范围 `PolygonGraphic`。
> **构造只接收 map**，分析范围通过**传入一个 `PolygonGraphic`** 来限定（绘制范围可用 `cimcube-controls` 的 `TakeCoordinates` 取点，或用 `cimcube-graphic` 的 `PolygonGraphic` 放入图层）。

**options（startByGraphic / startByTerrain 共用）：** `minHeight`（最小高程）、`maxHeight`（最大高程）、`height`（可选，自定义固定淹没水位）、`speed`（淹没速率，默认 10 m/s）。

```js
const flood = new A.FloodAnalysis(map);

// 分析范围：一个 PolygonGraphic 实例（示例假设已通过控件/图元绘制得到）
const area = getFloodRangeGraphic();

// ① 体块淹没：区域内的建筑体块随水位上涨
flood.startByGraphic(area, {
  minHeight: 0,              // 最小高程
  maxHeight: 60,             // 最大高程
  speed: 10,                 // 淹没速率 10m/s（默认）
});
flood.stopByGraphic();       // 停止体块淹没

// ② 地形淹没：水位沿地形表面上涨（模拟平原/河道泛洪）
flood.startByTerrain(area, {
  minHeight: 0,
  maxHeight: 45,
  speed: 10,
});
flood.stopByTerrain();

// 固定水位：指定 height 则从 minHeight 淹到该自定义水位
flood.startByGraphic(area, { minHeight: 0, height: 12 });

flood.clear();               // 清空结果
flood.dispose();             // 释放资源
```
