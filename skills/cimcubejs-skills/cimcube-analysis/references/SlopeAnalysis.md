# SlopeAnalysis — 坡度 / 坡向（可运行示例）

> 前提：`map` 已创建、已加载地形高程数据；`const A = Glodon.CIMCube.Analysis`。
> 构造 options 仅 `region`：`[minLon, minLat, maxLon, maxLat]`（度）。构造后自动计算，无 `FINISH` 事件，用方法切换显示。

```js
const slope = new A.SlopeAnalysis(map, {
  region: [114.28, 30.48, 114.32, 30.52],   // 分析范围 bbox（度）
});
slope.showSlope();     // 显示坡度分布
slope.showAspect();    // 切换显示坡向分布
slope.hide();          // 隐藏
slope.dispose();
```

> 若只想取“某组坐标的坡度数值”（不依赖地图、无可视化），见 `CalculateSlope.md`。
