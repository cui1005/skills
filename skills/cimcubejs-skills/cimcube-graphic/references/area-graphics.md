# 面 — PolygonGraphic · RectangleGraphic · EllipseGraphic

命名空间 `Glodon.CIMCube.Graphic`（下文 `G`）。通用选项见 SKILL.md §二。

| 图元 | 必填 | 关键默认 | 私有方法（超类方法不计） |
|---|---|---|---|
| `PolygonGraphic` | `positions:Array` | `fill:true`、`materialType:'color'`、`material:'white'`、`outline:false`、`outlineColor:Cesium.Color.BLACK`、`outlineWidth:1`、`closeTop:true`、`closeBottom:true`、**`clampToGround:true`**、`heightReference:NONE`、`extrudedHeightReference:NONE`、`zIndex:0`、`disableDepthTest:false` | `setHighlight({fillColor, outline:{color,width}})`、`setStyle(PolygonStyleOptions)`、`toGeoJSON()` |
| `RectangleGraphic` | `positions:Array` = **对角 2 点**、`height:Number` | `rotation:0`（单位度，从北向起算）、`heightReference`、`extrudedHeight` 系、fill/material/outline 系、`visibleRange`、`zIndex`；只读 `drawType` | —（**无 `setStyle`**；改样式走构造 opts） |
| `EllipseGraphic` | `position` 中心、`semiMajorAxis`、`semiMinorAxis`（米）、`height` | `rotation`（从北方向逆时针，单位度）、fill/material/outline 系、`numberOfVerticalLines`、`visibleRange`、`zIndex`；等高 = 圆 | —（**无 `setStyle`**；改样式走构造 opts） |

```js
const G = Glodon.CIMCube.Graphic;
// 面（默认贴地；拉伸用 extrudedHeight）
const poly = new G.PolygonGraphic({
  id: "pg01", name: "用地",
  positions: [[116.39,39.90,0],[116.41,39.90,0],[116.41,39.92,0],[116.39,39.92,0]],
  material: "#3a9eff", materialType: "color", outline: true, outlineColor: "#0a5ac2", outlineWidth: 1,
  clampToGround: true,                       // 默认即 true；贴地可用 classificationType: "terrain"/"tiles3d"/"both"
  // extrudedHeight: 30, heightReference: G.HeightReference.NONE,  // 拉伸为体
});
layer.addGraphic(poly);
poly.setHighlight({ fillColor: "rgba(255,212,0,0.85)", outline: { color: "#ff8c00", width: 3 } });
// 矩形：positions = 对角 2 点
const rect = new G.RectangleGraphic({
  id: "rc01", name: "地块",
  positions: [[116.39,39.90],[116.41,39.92]], height: 0, rotation: 15,   // rotation 单位度
  material: "rgba(255,212,0,0.5)", outline: true, outlineColor: "#ff8c00",
});
layer.addGraphic(rect);
// 椭圆 / 圆：semiMajorAxis = semiMinorAxis 即圆
const ellipse = new G.EllipseGraphic({
  id: "el01", name: "影响范围",
  position: [116.40,39.91,0], semiMajorAxis: 800, semiMinorAxis: 500, height: 0,
  rotation: 30, material: "rgba(255,80,80,0.35)", outline: true, outlineColor: "#ff0000",
});
layer.addGraphic(ellipse);
```
