# ViewDomeAnalysis — 开敞度（视球）分析（可运行示例）

> 前提：见 `patterns.md`（`map`、`A`、`llh`）。
> 以某点为中心、指定半径生成半球/球面视野：中心向半球面采样，可视与遮挡处分别着色。可用于开敞度评估、环视遮挡判断。

构造参数：

| 字段 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `viewPoint` | `Cesium.Cartesian3` | `(0,0,0)` | 观察点位置 |
| `radius` | `Number` | `100` | 分析半径（米），`(0,1000]` |
| `visibleColor` | `String`/`Color` | `#00ff00` | 可视区颜色 |
| `invisibleColor` | `String`/`Color` | `#ff0000` | 不可视区颜色 |
| `showVisibleArea` | `Boolean` | `true` | 是否显示可视区域 |
| `showInvisibleArea` | `Boolean` | `true` | 是否显示不可视区域 |
| `startAngle` | `Number` | `0` | 起始方位角（度） |
| `endAngle` | `Number` | `360` | 结束方位角（度） |
| `dynamic` | `Boolean` | `false` | 是否动态模式 |

方法：`doAnalysis()` 执行分析；`setViewPoint(cartesian3)` 换观察点；`dispose()` 销毁。

```js
const dome = new A.ViewDomeAnalysis(map, {
  viewPoint:         llh(116.4076, 39.9089, 1.5),
  radius:            300,
  visibleColor:      "rgba(0,255,0,0.45)",
  invisibleColor:    "rgba(255,0,0,0.5)",
  showVisibleArea:   true,
  showInvisibleArea: true,
});
dome.doAnalysis();
// 换点重新开敞度评估
dome.setViewPoint(llh(116.4082, 39.9095, 1.5));
dome.doAnalysis();
// dome.dispose();
```

> 结果以半球着色可视化为主，类页未暴露「开敞率数值」回调；如需要数值，可按扇区统计可视/全部采样（或联 VisibilityAnalysis 多点采样）自算。
