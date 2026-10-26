# LimitHeight — 限高核查（可运行示例）

> 前提：见 `patterns.md`（`map`、`A`）；且已把范围内建筑 3D 数据加载进场景、相机能覆盖该范围。
> 对多边形范围内的 3D 建筑/模型做限高检测：超出指定高度的部分按颜色罩染（可叠加包围盒），随瓦片加载持续刷新，结果可通过事件与 `getAnalysisResult()` 读取。适合规划限高合规核查。

构造参数：

| 字段 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `positions` | `Array` | — | 控高多边形范围，扁平经纬度：`[lon1, lat1, lon2, lat2, ...]`（无高程） |
| `limitHeight` | `Number` | `20` | 限高基准（米） |
| `color` | `String`/`Color` | — | 超出部分的罩染颜色 |
| `mixAlpha` | `Number` | `0.5` | 罩染透明度 |
| `showBox` | `Boolean` | `false` | 是否显示限高包围盒 |

方法：`doAnalysis()` 触发分析；`setPositions(pos)` / `setLimitHeight(h)` / `setColor(c)` / `setMixAlpha(a)`；`update()` 图层数据变化后刷新；`clear()` 清除效果；`dispose()` 销毁。**继承 `EventDispatcher`**，支持 `on/off` 事件语法。

```js
const lh = new A.LimitHeight(map, {
  positions: [116.38, 39.90, 116.42, 39.90, 116.42, 39.92, 116.38, 39.92], // 范围多边形（≥3 点）
  limitHeight: 60,
  color: "#ff0000",
  mixAlpha: 0.5,
  showBox: false,
});
lh.doAnalysis();   // 开启限高分析（后续范围内瓦片加载/刷新会增量更新）
// 当前实现中分析状态变化派发 type="change"（负载含 limitLayerFeatureMaps），可用 on() 监听
lh.on("change", (e) => {
  console.log("限高核查刷新", e.type);
});

const list = lh.getAnalysisResult();   // 超限要素数组（结构以类页/实际运行为准）
lh.setLimitHeight(80);                 // 改限高后再核查
lh.doAnalysis();
// lh.clear();
lh.dispose();
```

> 参与核查的图层即场景中位于该多边形内的 3D 瓦片数据；请先加载好建筑数据并让相机覆盖该范围，瓦片就绪后结果才会完整。
