# ElevationHeatMap — 地形高程热力图（可运行示例）

> 前提：`map` 已创建、已加载地形高程数据；`const A = Glodon.CIMCube.Analysis`。
> options：`region`（bbox，**必传，不支持全球范围**）、`colorArray`（替换默认色带，建议不少于 10 个颜色）、`autoCompute`（是否自动计算地形高程图例并随相机位置更新，默认 true）。

```js
const heat = new A.ElevationHeatMap(map, {
  region: [114.28, 30.48, 114.32, 30.52],
  colorArray: [
    "#1565c0", "#1e88e5", "#42a5f5", "#90caf9", "#b3e5fc",
    "#fff59d", "#ffd54f", "#ffb300", "#fb8c00", "#e65100",
  ], // ≥10 色，低→高
  autoCompute: true,
});
heat.show();
heat.setLegend(image, min, max);   // 自定图例图片与数值范围（示意）
heat.hide();
heat.dispose();
```
