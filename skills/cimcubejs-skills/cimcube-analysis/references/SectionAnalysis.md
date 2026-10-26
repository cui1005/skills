# 剖面：SectionAnalysis（统一入口）/ TerrainSectionAnalysis / ModelSectionAnalysis

> 三者同属“沿折线按间隔采样高程”，**签名一致、差异仅在采样对象**。日常剖切**统一使用 `SectionAnalysis`**：把目标 3DTiles 图层传给 `options.layer`，地形与模型图层都适用，避免记混另外两个类：

| 类 | 采样对象 | 差异 |
|---|---|---|
| `SectionAnalysis(map, { layer })` | 传入的 `Tiles3DLayer`（或数组），**只对传入图层采样** | 需提供 layer，最通用 → 默认选用 |
| `TerrainSectionAnalysis(map, { sampleTerrainLevel })` | 场景默认地形 | 无需 layer，多一个 `sampleTerrainLevel` 参数 |
| `ModelSectionAnalysis(map)` | `doAnalysis(modelLayer, ...)` 现场传模型图层 | 无 options，参数在第 1 位 |

> 前提：`map` 已创建；`terrainTiles3DLayer` 为已加载的 `Tiles3DLayer`；`const A = Glodon.CIMCube.Analysis`。

```js
// ── 统一用法：对 3DTiles 地形/模型图层采样 ──────────────────────────
const sec = new A.SectionAnalysis(map, {
  layer: terrainTiles3DLayer,       // Tiles3DLayer | Tiles3DLayer[]
});
sec.doAnalysis(
  [Cesium.Cartographic.fromDegrees(114.29, 30.49),
   Cesium.Cartographic.fromDegrees(114.30, 30.50),
   Cesium.Cartographic.fromDegrees(114.31, 30.50)],  // 分析折线（经纬度）
  10,                                                // 采样间隔，单位米
  (points) => {
    // points: Object[]，Object.length = 水平距离(m)，Object.height = 对应高程(m)
    draw2DProfileChart(document.getElementById("profile-canvas"), points);
  },
);
sec.dispose();
```
