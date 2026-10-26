# Runoff — 径流分析（可运行示例）

> 前提：`map` 已创建、已加载地形高程数据；`const A = Glodon.CIMCube.Analysis`。
> region 框选范围内，基于地形做降雨汇流（流向线/箭头动态展示）。

主要 options：`region`（bbox，同 SlopeAnalysis 形制）、`levelTerrain`、`unitGrid`（格网大小）、`autoDepress`（自动填洼）、`maxFlowDistance`；展示类见 SKILL.md 速查表的“流线族 / 箭头族”（如 `showFlowLines`、`flowLineColor`、`flowLineWidth`、`showFlowArrows`、`flowArrowWidth` 等）。

```js
const runoff = new A.Runoff(map, {
  region: [114.28, 30.48, 114.32, 30.52],
  unitGrid: 5,
  autoDepress: true,
  showFlowLines: true,
  showFlowArrows: true,
  flowLineWidth: 4,
});
runoff.doAnalysis();       // 开始径流模拟
runoff.setOpts({ showFlowArrows: false });   // 改参数（重跑见类页）
runoff.clearPrimitive();   // 清除流线/箭头
runoff.dispose();
```
