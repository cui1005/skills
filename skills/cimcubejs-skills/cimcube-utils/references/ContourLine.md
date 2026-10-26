# Utils.ContourLine — 地形等高线可视化（材质层 + canvas 覆盖）

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

| 项 | 内容 |
|---|---|
| 构造 | `new U.ContourLine(map, { spacing?: 100, lineWidth?: 2, lineColor?: '#FF0000', region?: [-180,-90,180,90], gridSpacingX?: 80, gridSpacingY?: 30 })` |
| `region` | 剖面/搜索范围：`[minLon, minLat, maxLon, maxLat]` |
| `spacing` | 等高距（米）；`lineWidth` 线宽 px；`lineColor`；`gridSpacingX/Y` 采样网格（px） |
| 生命周期 | `show()` 显示 / `hide()` 清除材质并停掉监听 |
| 数据回调 | `const remove = contour.addChangeListener(cb)`：**cb(list)** 收等高线点数组，list 元素 `{left, top, height}`——`left/top` 是相对画布左上角的像素偏移，`height` 是对应等高线高程（米）；返回的 `remove` 可直接调用移除；`removeChangeListener(cb)` 按回调引用移除 |

等高线是**基于地形材质（由引擎内部管理，不对外暴露）绘制 + 离屏渲染后把像素坐标回调出来**：适合在业务 Canvas 上叠加标线/标高点、绘制用户自定义等高线样式。注意需要地形生效、范围别太大（受 `gridSpacingX/Y` 采样密度影响性能）。

```js
const contour = new U.ContourLine(map, { spacing: 10, lineColor: '#ff9900', region: [116.2, 39.8, 116.5, 40.0] });
contour.addChangeListener((list) => {
  // list: [{left:100, top:100, height:20}, ...]；把点画到覆盖在 canvas 上的等高线图层
  drawOnOverlay(list);
});
// contour.hide(); contour.dispose();
```

> ⚠️ 臆造（不可用）：`new U.ContourLine(map, { target: demLayer, interval, levels })` → 输出折线要素 不存在。真实入参为 `{ spacing, lineWidth, lineColor, region, gridSpacingX/Y }`；回调数组 `{left,top,height}` 供自绘。
