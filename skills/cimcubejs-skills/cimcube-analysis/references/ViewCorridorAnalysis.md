# ViewCorridorAnalysis — 视廊分析（可运行示例）

> 前提：见 `patterns.md`（`map`、`A`）。
> 起点、终点各取一个「横截面宽高」生成三维可视廊体：分析廊体空间是否被建筑侵入并着色高亮。适合景观视廊、规划中轴线视廊控制。

构造参数：

| 字段 | 类型 | 说明 |
|---|---|---|
| `layers` | `Array`/`Object` | 参与分析的图层（省略则取场景可见数据） |
| `startInfo.position` | `Array`/`Object` | 起点 `[lon,lat,height]` 或 `{lon,lat,height}` |
| `startInfo.width` | `Number` | 起点拓宽距离（米），默认 `20`（单侧，双侧对称共约 2 倍） |
| `startInfo.height` | `Number` | 起点抬高距离（米），默认 `30`（单侧，上下对称共约 2 倍） |
| `endInfo` | `Object` | 同上结构，为终点截面参数 |
| `style.fillColor` | `String`/`Color` | 廊体填充色，默认 `#00ff00` |
| `style.alpha` | `Number` | 填充透明度，默认 `0.3` |
| `style.outlineColor` | `String`/`Color` | 廊体外框颜色，默认 `#0000ff` |
| `style.outlineWidth` | `Number` | 外框线宽，默认 `2` |
| `style.highlightColor` | `String`/`Color` | 侵入/遮挡高亮色，默认 `#ff0000` |

方法：`doAnalysis()` 执行分析；`dispose()` 销毁。

```js
const corridor = new A.ViewCorridorAnalysis(map, {
  startInfo: { position: { lon: 116.4, lat: 39.9, height: 3 }, width: 20, height: 30 },
  endInfo:   { position: [116.401, 39.902, 3],                width: 20, height: 30 },
  style: {
    fillColor: "#00ff00", alpha: 0.3,
    outlineColor: "#0000ff", outlineWidth: 2,
    highlightColor: "#ff0000",
  },
});
corridor.doAnalysis();
// corridor.dispose();
```
