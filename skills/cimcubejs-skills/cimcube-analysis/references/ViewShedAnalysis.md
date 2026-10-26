# ViewShedAnalysis — 视域分析（可运行示例）

> 前提：见 `patterns.md`（`map`、`A`、`Cesium`、`llh`）。
> 以视点为中心的扇形/圆锥可视分析：对可视/遮挡物着色，可叠加视锥采样网格。适合基站选址、监控布点等「某点看四周」场景。

构造参数：

| 字段 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `viewPosition` | `Cesium.Cartesian3` | `(0,0,0)` | 视点位置 |
| `targetPosition` | `Cesium.Cartesian3` | `(1,1,1)` | 朝向参考点（与 direction/pitch 配合决定视域朝向） |
| `horizontalViewAngle` | `Number` | `90` | 水平张角（度），`(0,180)`；`>170` 时精度下降 |
| `verticalViewAngle` | `Number` | `90` | 垂直张角（度），`(0,180)` |
| `visualRange` | `Number` | `1000` | 可视距离（米），`(0,100000)` |
| `direction` | `Number` | `0` | 视域朝向角（度） |
| `pitch` | `Number` | `0` | 视域俯仰角（度） |
| `viewHeightOffset` | `Number` | `0` | 视点高度修正（米） |
| `visibleAreaColor` | `String`/`Color` | `#00ff00` | 可视区域颜色 |
| `invisibleAreaColor` | `String`/`Color` | `#ff0000` | 不可视区域颜色 |

方法：`doAnalysis()` 触发分析；`drawSketch()` 绘制辅助视锥/采样网格；`clear()` 清除结果；`dispose()` 销毁。

```js
const vs = new A.ViewShedAnalysis(map, {
  viewPosition:   llh(116.4076, 39.9089, 80),   // 视点（杆顶/瞭望点）
  targetPosition: llh(116.408, 39.909, 50),     // 朝向参考点
  horizontalViewAngle: 90,
  verticalViewAngle:   60,
  visualRange:         1500,
  viewHeightOffset:    2,
  visibleAreaColor:    "rgba(52,168,83,0.45)",
  invisibleAreaColor:  "rgba(234,67,53,0.45)",
});
vs.doAnalysis();     // ① 触发分析，色带落在场景
vs.drawSketch();     // ② 可选：叠加视锥/网格便于观察
// 需要换视点/参数时：旧实例 dispose 后按新参数重建再 doAnalysis（该类页未提供 setter）
// vs.clear();        // 清除结果
vs.dispose();        // 最终销毁
```

> 结果以场景可视化为准，该类页**未暴露**事件/数值读取接口。需要「百分比可视率」等业务数值时，可对该视域内的特征点分别做 VisibilityAnalysis 通视判断统计。
