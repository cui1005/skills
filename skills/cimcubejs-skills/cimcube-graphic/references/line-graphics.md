# 线 — PolylineGraphic · PolylineVolumeGraphic · CorridorGraphic

命名空间 `Glodon.CIMCube.Graphic`（下文 `G`）。通用选项见 SKILL.md §二。

| 图元 | 必填 | 关键默认 | 私有方法（超类方法不计） |
|---|---|---|---|
| `PolylineGraphic` | `positions:Array` | `width:1`、`material:'WHITE'`、`depthFailMaterial:undefined`（被遮挡颜色）、`disableDepthTest:false`、`zIndex:0`；**构造无 `clampToGround` 键** | `setHighlight({color,width})`、`setStyle(PolylineStyleOptions)`（`type:'solid'\|'dash'\|'glow'\|'arrow'`、`color`、`width`、`clampToGround`、`occludedLine:{type:Defines.LineType.SOLID, color:'#FFFF00', opacity:1, gapColor:'#000000', gapOpacity:0, clampToGround}`）、`toGeoJSON()` |
| `PolylineVolumeGraphic` | `positions:Array`、`shape:Array.<TwoDimensionPoint>`（沿线横截面，如圆/多边形） | `cornerType`（转角）、`fill`、`material`、`visibleRange`；**无 `setStyle`** | —（仅继承方法；改样式走构造 opts） |
| `CorridorGraphic` | `positions:Array`、**`width:Number` = 缓冲半径（单位米，非总宽/像素）** | `fill`/`material`/`outline`/`outlineColor`/`outlineWidth`、`height`/`heightReference`、`extrudedHeight`/`extrudedHeightReference`、`cornerType`、`visibleRange`、`zIndex`（贴地才生效） | —（**无 `setStyle`**；改样式走构造 opts） |

```js
const G = Glodon.CIMCube.Graphic;
// 折线（含被遮挡线样式）
const line = new G.PolylineGraphic({
  id: "ln01", name: "管线",
  positions: [[116.39,39.90,0],[116.40,39.91,0],[116.41,39.905,0]],
  width: 4, material: "#00c6ff",
});
layer.addGraphic(line);
line.setStyle({ type: "glow", color: "#00c6ff", width: 6 });            // solid/dash/glow/arrow
line.setStyle({ type: "dash", color: "#00c6ff", width: 4,
                occludedLine: { type: "dashes", color: "#ffff00", opacity: 1,
                                gapColor: "#000000", gapOpacity: 0 } });  // 遮挡线
// 沿线横截面体（shape 为 TwoDimensionPoint 数组）
const tube = new G.PolylineVolumeGraphic({
  id: "tube01", name: "管廊",
  positions: [[116.39,39.90,0],[116.41,39.91,0]],
  shape: [new G.TwoDimensionPoint(-1, 1), new G.TwoDimensionPoint(1, 1),
          new G.TwoDimensionPoint(1, -1), new G.TwoDimensionPoint(-1, -1)],
  material: "#3a9eff",
});
layer.addGraphic(tube);
// 走廊：width = 缓冲半径（米），输入 15 → 左右各 15m 共 30m 宽
const corridor = new G.CorridorGraphic({
  id: "cr01", name: "缓冲带",
  positions: [[116.39,39.90,0],[116.41,39.91,0]],
  width: 15, material: "rgba(58,158,255,0.4)",
});
layer.addGraphic(corridor);
```
