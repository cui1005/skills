# 体 / 空间几何 — WallGraphic · BoxGraphic · CylinderGraphic · EllipsoidGraphic · PlaneGraphic

命名空间 `Glodon.CIMCube.Graphic`（下文 `G`）。通用选项见 SKILL.md §二。

| 图元 | 必填 | 关键默认 | 私有方法（超类方法不计） |
|---|---|---|---|
| `WallGraphic` | `positions:Array`（顶点高度定义墙体顶部；或配合 minimum/maximumHeights） | `fill:true`、`material:'#FFFFFF'`；`minimumHeights:Array`（墙底椭球高）/`maximumHeights:Array`（墙顶椭球高），数组与顶点数一致、可任意起伏 | `setStyle({fill,color,outline,outlineColor,outlineWidth,extrudedHeight})`；只读 `drawType` |
| `BoxGraphic` | `position` 中心、`dimensions:{length,width,height}`（米） | `orientation:{heading,pitch,roll}`（度，默认 0）、`heightReference`、fill/material/outline 系、`visibleRange`；**无 `setStyle`** | —（仅继承方法；改样式走构造 opts） |
| `CylinderGraphic` | `position` 中心、`length`、`topRadius`、`bottomRadius`（米） | fill/material/outline 系、`numberOfVerticalLines`、`slices`（圆周边数，**≥3**）、`heightReference`、`visibleRange` | —（**无 `setStyle`**；改样式走构造 opts） |
| `EllipsoidGraphic` | `position` 中心、`radii:{east,north,up}`（米） | 扫描参数 `minimumClock`/`maximumClock`（垂直扫描起止角，度，从上方向顺时针）、`minimumCone`/`maximumCone`（水平扫描起止角）、fill/material/outline 系、`heightReference`、`visibleRange` | —（**无 `setStyle`**；改样式走构造 opts） |
| `PlaneGraphic` | `position` 中心、`normal:{east,north,up}`、`dimensions:{length,width}` | fill/material/outline 系、`visibleRange`；outlineWidth Windows 不生效；**无 `setStyle`** | —（仅继承方法；改样式走构造 opts） |

```js
const G = Glodon.CIMCube.Graphic;
// 墙（沿折线立墙，可起伏）
const wall = new G.WallGraphic({
  id: "wl01", name: "围挡",
  positions: [[116.39,39.90,0],[116.41,39.90,0],[116.41,39.92,0]],
  minimumHeights: [0, 0, 0], maximumHeights: [8, 12, 8],
  material: "#8ab4ff", fill: true,
});
layer.addGraphic(wall);
// 盒子（中心 + 尺寸 + 姿态角度制）
const box = new G.BoxGraphic({
  id: "bx01", name: "楼块",
  position: [116.40,39.91,15], dimensions: { length: 30, width: 20, height: 30 },
  orientation: { heading: 30, pitch: 0, roll: 0 },
  material: "rgba(58,158,255,0.7)", outline: true, outlineColor: "#0a5ac2",
});
layer.addGraphic(box);
// 圆柱 / 圆台（topRadius = bottomRadius 即圆柱）
const cyl = new G.CylinderGraphic({
  id: "cy01", name: "烟囱",
  position: [116.40,39.91,25], length: 50, topRadius: 2, bottomRadius: 5,
  slices: 32, material: "#d0d0d0",
});
layer.addGraphic(cyl);
// 椭球 / 球（radii 三轴相等即球）
const ball = new G.EllipsoidGraphic({
  id: "el01", name: "球体",
  position: [116.40,39.91,20], radii: { east: 20, north: 20, up: 20 },
  material: "rgba(255,212,0,0.6)",
});
layer.addGraphic(ball);
// 平面板（中心 + 法向 + 尺寸）
const plane = new G.PlaneGraphic({
  id: "pl01", name: "剖面",
  position: [116.40,39.91,15], normal: { east: 0, north: 1, up: 0 },
  dimensions: { length: 40, width: 30 }, material: "rgba(255,80,80,0.4)",
});
layer.addGraphic(plane);
```
