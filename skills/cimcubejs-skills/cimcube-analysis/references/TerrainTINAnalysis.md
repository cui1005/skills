# TerrainTINAnalysis — 地形三角网（可运行示例）

> 前提：`map` 已创建、已加载地形高程数据；`const A = Glodon.CIMCube.Analysis`。
> options：`enabled`、`boundary`（显示范围，**只支持一个范围**）、`tinLineColor` / `tinFillColor`（线与面颜色）、`showImage`。

```js
const tin = new A.TerrainTINAnalysis(map, {
  enabled: true,
  boundary: [/* 单范围 */],
  tinLineColor: "#9e9e9e",
  tinFillColor: "#90caf9",
});
tin.show();
tin.hide();
tin.clear();       // 清空三角网
tin.dispose();
```
