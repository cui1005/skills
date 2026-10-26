# Utils.BufferTool — 纯静态缓冲几何（无 map）

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

`new U.BufferTool()` 不需要参数，三个**静态方法**直接算缓冲边界（内部 turf buffer，经纬度球面近似）：

| 方法 | 输入 | 返回 | 注意 |
|---|---|---|---|
| `U.BufferTool.generatePointBuffer(point, distance)` | `point`：单个 Cesium.Cartographic | 该点缓冲（近似圆）边界 `Array<Cartographic>` | |
| `U.BufferTool.generatePolylineBuffer(points, distance)` | `points`：Cartographic 数组（≥2，否则抛"点数小于2，构不成线串"） | 线缓冲边界 `Array<Cartographic>` | |
| `U.BufferTool.generatePolygonBuffer(points, distance, preserveHole = true)` | `points`：多边形顶点 Cartographic 数组（≥3，**必须闭合：首点=末点**，否则 turf 抛 First and last Position are not equivalent） | 外环 `Array<Cartographic>`；`preserveHole` 生效且有内缩边界时返回 `[外环, 内环]` | preserveHole 用负缓冲抠内环 |

- `distance` 单位是**米**（内部 `distance/1000` 转千米）。
- 输入对象的读取字段是 Cesium.Cartographic 的 `longitude/latitude`（弧度），结果 `Cartographic` 高度为 0。拿到结果后用 `Glodon.CIMCube.Global.Math.Cartographic.toDegrees` / Graphic.Polygon/Polyline 上屏。

```js
const { Cartographic } = Glodon.CIMCube.Global.Math;   // 或 Cesium 对应类
const pts = [[116.35,39.9],[116.36,39.9]].map(([l, a]) => Cartographic.fromDegrees(l, a));
const ring = Glodon.CIMCube.Utils.BufferTool.generatePolylineBuffer(pts, 30);   // 30m 道路缓冲
// ring.forEach(c => console.log(Cartographic.toDegrees(c.longitude), Cartographic.toDegrees(c.latitude)));
```

> ⚠️ 臆造（不可用）：`new U.BufferTool(map, { target, distance })`、`buf.intersects()` 不存在。真实为静态 `U.BufferTool.generateXxxBuffer`，输入 Cartographic 数组；**没有实例方法/图层目标/相交查询**。
