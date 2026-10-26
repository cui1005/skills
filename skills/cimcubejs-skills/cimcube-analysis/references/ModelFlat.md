# ModelFlat — 模型拍平（可运行示例）

> 前提：见 `patterns.md`（`map`、`A`）；`tilesLayer` 为已加载的建筑 S3M/倾斜摄影图层。
> 在指定多边形内把 3D 模型（建筑/倾斜摄影/点云等）压到指定高度，可多区域叠加、按区域增删。常用于「现状→拍平→重新规划」流程。

构造参数：

| 字段 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `layers` | `Array` | — | 要压平的图层数组 |
| `positions` | `Array` | — | 压平范围多边形点 `[lon,lat,...]` 或 `[{lon,lat},...]` |
| `height` | `Number` | `0` | 压平高度（米），`[0,10000]` |
| `includeTerrain` | `Boolean` | `true` | 是否连地形一起压平 |

区域方法：`addRegion(options)` 添加压平范围；`removeRegion(id)` / `getRegion(id)`；`setHeight(h, id?)` / `setLayers(layers, id?)` / `setPositions(positions, id?)` / `setIncludeTerrain(bool, id?)`。整体：`doAnalysis()` 执行；`show` 字段切换显隐；`clear()` 清理（兼容旧接口，将废弃）；`dispose()` 销毁。

```js
const flat = new A.ModelFlat(map, {
  layers: [tilesLayer],                  // 建筑 S3M/倾斜摄影图层
  positions: [116.4, 39.9, 116.41, 39.9, 116.41, 39.905, 116.4, 39.905],
  height: 0,
  includeTerrain: true,
});
flat.doAnalysis();          // 圈内建筑压平
// 再加一块压平区（可显式指定区域 id，便于 removeRegion）
flat.addRegion({
  id: "region-2",
  positions: [116.412, 39.906, 116.418, 39.906, 116.418, 39.912, 116.412, 39.912],
  height: 10,
});
flat.doAnalysis();
flat.show = false;          // 隐藏压平效果
flat.show = true;           // 恢复（内部会重建）
flat.removeRegion("region-2"); // 移除该压平区
// flat.clear();
flat.dispose();
```
