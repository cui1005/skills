# CaeFeCloud — 有限元云图（可运行示例）

> 前提：见 `patterns.md`（`map`、`A`）。
> 加载有限元（CAE/FEA）分析结果 JSON 并在模型相应位置生成应力/变形热点云图，热点带呼吸（Twinkle）动画，可一键飞近。适合结构分析成果在三维场景中的表达。

构造参数：

| 字段 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `origin` | `Array` | — | 原点经纬度 `[lon, lat, height]`，例 `[106.689936, 29.559434, 500]` |
| `rotation` | `Array` | `[0,0,0]` | 相对 X/Y/Z 旋转（度） |
| `show` | `Boolean` | `true` | 是否显示 |
| `hotspotHeight` | `Number` | `0.1` | 热点相对模型表面抬高（米） |
| `twinkle` | `Object` | — | 动画参数（参考 `Glodon.CIMCube.Animation.PointEffect.Twinkle`，如 `openDepthTest / onlyExsufflation / breathDuration / size / minSize / viewingDistance` 等） |

方法：`loadFromUrl(url)` 异步加载 FEA JSON（返回 Promise）；`setData(json)` 直接注入 JSON 数据；`flyTo(opts?)` 飞到数据范围（`opts.duration` 默认 `2` 秒，可传 `offset` 等）；`dispose()` 销毁。

```js
const cae = new A.CaeFeCloud(map, {
  origin: [106.689936, 29.559434, 500.0],
  rotation: [0, 0, 0],
  show: true,
  hotspotHeight: 0.1,
  twinkle: {
    openDepthTest: true,
    onlyExsufflation: true,
    breathDuration: 1.0,
    size: 8,
    minSize: 2,
    viewingDistance: 500,
  },
});
await cae.loadFromUrl("./data/fea-bridge-sample.json"); // FEA JSON → 自动生成热点云图
// 或直接注入数据： cae.setData(feaJson);
cae.flyTo({ duration: 2 });
// cae.dispose();
```

> FEA JSON 需符合 SDK 约定的云图数据格式（按 `data` 中点位/数值生成热点）。若加载后无云图，优先核对 JSON 结构与 `origin` 是否对准模型坐标系。
