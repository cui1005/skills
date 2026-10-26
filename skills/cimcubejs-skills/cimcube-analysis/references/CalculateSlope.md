# CalculateSlope — 坡度数值计算（不挂地图，可运行示例）

> 前提：`const A = Glodon.CIMCube.Analysis`。
> 纯数值类：根据坐标数据直接算坡度，**构造不接收 map**、无可视化。结果结构与完整 data 形制以 `dist/API_DOC/` 类页为准。

```js
const calc = new A.CalculateSlope({
  data: [/* 坐标数据（Array），形制见类页；也可在实例上赋值 calc.data */],
});
const result = calc.doAnalysis();   // 计算结果（返回结构以类页为准）
calc.dispose();
```
