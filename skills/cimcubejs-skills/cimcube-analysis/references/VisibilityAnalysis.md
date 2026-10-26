# VisibilityAnalysis — 通视分析（两点通视，可运行示例）

> 前提：见 `patterns.md`（`map`、`A`、`llh`）。
> 对「起点→终点」做单次遮挡判断：无遮挡整体绿色；有遮挡绿线画到首个遮挡物、红线连到终点，并可显示两端点位。适合两楼间通视评估、塔吊与大楼视线遮挡等。

构造参数：

| 字段 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `startPos` | `Cesium.Cartesian3` | — | 起点（观察窗/相机位），**必须传** |
| `endPos` | `Cesium.Cartesian3` | — | 终点（目标楼顶点），**必须传** |
| `showPosition` | `Boolean` | `true` | 是否绘制起终点标记点 |
| `visibleColor` | `String`/`Color` | `#00ff00` | 可视段颜色 |
| `invisibleColor` | `String`/`Color` | `#ff0000` | 遮挡段颜色 |

方法：`setStartPos(cartesian3)` / `setEndPos(cartesian3)` 动态改点（**必须传 `Cartesian3`**）；`doAnalysis()` 执行并绘制；`reset()` 清空本次判定实体与标记；`dispose()` 销毁。

```js
const vis = new A.VisibilityAnalysis(map, {
  startPos: llh(116.4076, 39.9089, 60),
  endPos:   llh(116.3974, 39.9087, 5),
  visibleColor:   "#00ff00",
  invisibleColor: "#ff0000",
  showPosition:   true,
});
vis.doAnalysis();   // 直接绘制判定结果（绿=可视/绿+红=被遮挡）
// 换目标楼再判断：
vis.setEndPos(llh(116.3980, 39.9087, 6));
vis.doAnalysis();
// vis.reset();   // 清除本组实体后重开一组
vis.dispose();
```

> 该判定是逐条射线式，比较快，适合批量跑（循环内 set 端点 + doAnalysis + 截取场景后再 reset）。
