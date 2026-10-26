# Utils.HeatMap — 2D/3D 数据热力

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

`new U.HeatMap(map [, options])`。options（均可选）：`type`（"2D"/"3D"，默认 "3D"）、`radius`（半径 30）、`blur`（30）、`grid`（性能颗粒度 1）、`liftDistance`（热力高程，米）、`heightDiff`（3D 凸起幅度 100）、`maxValue`（100）、`precision`（经纬度精度 8）、`gradient`（渐进色，最多 8 档）、`maxOpacity`（1.0）/`minOpacity`（0.0）、`growthAnimation`（生长动画帧 60）、`clampToGround`（false）、`offsetLevel`（-10~10，越大显示精度越高）。运行时还支持 `canvas`（canvas 元素或 id，默认自建）。

```js
const hm = new U.HeatMap(map, {
  type: "3D", liftDistance: 50, radius: 15, blur: 20,
  gradient: { 0.2: "blue", 0.4: "cyan", 0.6: "lime", 0.8: "yellow", 1.0: "red" },
  maxOpacity: 0.9, minOpacity: 0.0, heightDiff: 2000,
});
hm.setData([{ lon: 116.40, lat: 39.90, value: 10 }, { lon: 116.41, lat: 39.91, value: 30 }]); // value ≤ maxValue
// 成员可读写：hm.show / hm.radius / hm.blur / hm.gradient / hm.clampToGround
hm.dispose();
```

> **无 `render()/refresh()/start()/stop()`**；`setData()` 即刷新。数据体量大小有限制（超大点集先格网聚合）。开销：中（点→格点）——点集先聚合；`grid` 控制颗粒度；用完 dispose。区别于 `Analysis.ElevationHeatMap`（对区域地形的**高程着色**热力）。
