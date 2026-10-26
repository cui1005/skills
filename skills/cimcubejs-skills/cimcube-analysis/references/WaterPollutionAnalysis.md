# WaterPollutionAnalysis — 水污染扩散（可运行示例）

> 前提：`map` 已创建；`const A = Glodon.CIMCube.Analysis`。
> **options（类页）：** `region`（Array，范围）、`time`（Array，时间）、`positions`（Array，位置）、`data`（Array，数据）、`speed`（Number，速率）。

```js
const wp = new A.WaterPollutionAnalysis(map, {
  region:    [/* 分析范围 */],
  time:      [/* 时间序列（起止 / 采样） */],
  positions: [/* 污染源位置 */],
  data:      [/* 污染浓度等数据 */],
  speed:     1,              // 扩散速率
});
wp.doAnalysis();             // 触发扩散分析
wp.dispose();                // 释放资源
```
