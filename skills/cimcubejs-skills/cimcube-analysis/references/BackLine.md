# BackLine — 建筑退距线（可运行示例）

> 前提：见 `patterns.md`（`map`、`A`）。
> 给定建筑红线（至少 2 点），按左右退距生成退距范围面（可着色内部物体）与退距线，侧向联动展示。适合日照/消防/规划退距审查。

构造参数：

| 字段 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `positions` | `Array` | — | 红线数据（≥2 点）；支持 对象数组/二维数组/扁平一维数组（见下） |
| `side` | `String` | `Both` | `Left` / `Right` / `Both`；以首点看向第二点的方向定左右 |
| `width` | `Number` | `10` | 总退距宽度（米），左右各占一半 |
| `height` | `Number` | `10` | 退距高度（米） |
| `color` | `String`/`Color` | `#ff0000` | 退距范围边框颜色 |
| `alpha` | `Number` | `0.5` | 边框透明度 |
| `overlayColor` | `String`/`Color` | `#0000ff` | 内部物体着色颜色 |
| `overlayMixAlpha` | `Number` | `0.5` | 内部物体着色透明度 |
| `lineColor` | `String`/`Color` | `#ff0000` | 退距线颜色 |
| `lineWidth` | `Number` | `3` | 退距线像素宽 |
| `left` | `Object` | — | 左侧退距覆盖：`{width(默认 width/2), height(默认 height), color, alpha, overlayColor, overlayMixAlpha}` |
| `right` | `Object` | — | 右侧退距覆盖：结构同 `left` |

方法：`doAnalysis()` 执行分析；`setOpts(options)` 修改参数；`setPositions(positions)`；`setSide(side)`；`clear()` 清理；`dispose()` 销毁。可视开关为字段：`leftShow` / `rightShow` / `lineShow`（布尔）。

```js
// positions 支持多种写法：
//   对象数组 [{lon,lat,height}, {lon,lat,height}, ...]
//   二维数组 [[lon,lat,height], ...]
//   扁平数组 [lon1,lat1,h1, lon2,lat2,h2, ...]（首尾相连成闭合多边形时要求 ≥4 点）
const bl = new A.BackLine(map, {
  positions: [
    { lon: 116.4, lat: 39.9, height: 0 },
    { lon: 116.402, lat: 39.9, height: 0 },
    { lon: 116.402, lat: 39.902, height: 0 },
    { lon: 116.4, lat: 39.902, height: 0 },   // 回到首点 → 闭合，做面域退距
  ],
  width: 20,
  height: 6,
  side: "Both",
  color: "#ff0000",
  alpha: 0.5,
  overlayColor: "#0000ff",
  overlayMixAlpha: 0.5,
  lineColor: "#ff0000",
  lineWidth: 3,
});
bl.doAnalysis();
// 改单侧显示 / 调整参数后重新分析
bl.setSide("Right");
bl.setOpts({ width: 30 });
bl.doAnalysis();
// bl.leftShow = false;   // 隐藏左侧退距面
// bl.clear();
bl.dispose();
```
