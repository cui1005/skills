# NearestPointAnalysis — 最近点分析（静态调用，不需要 map）

> 在给定点集合中求距离 target 最近的若干点（按距离升序返回）。**纯静态工具类，不需要 map**，也不产生任何场景元素。

```js
const A = Glodon.CIMCube.Analysis;
const pointCollection = [
  { location: { lon: 116.40, lat: 39.90, height: 0 }, properties: { name: "消防站A" } },
  { location: { lon: 116.41, lat: 39.89, height: 0 }, properties: { name: "消防站B" } },
  { location: { lon: 116.404, lat: 39.895, height: 0 }, properties: { name: "应急点C" } },
];

const result = A.NearestPointAnalysis.doAnalysis(
  { lon: 116.405, lat: 39.895, height: 0 },  // target
  pointCollection,                           // pointCollection
  true,                                      // ignoreHeight=true：按平面距离计算
  3                                          // count：取最近 3 个
);

result.forEach((item) => {
  // item: { distance, point: { location, properties } }
  console.log(item.point.properties.name, item.distance.toFixed(2) + "m");
});
```

> 适合就近设施查找、路径起终点吸附等纯计算场景；若需「可视」就近（要求两点通视），再对候选结果配合 VisibilityAnalysis 逐个通视校验即可。
