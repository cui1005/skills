# 点与标注 — PointGraphic · BillboardGraphic · LabelGraphic

命名空间 `Glodon.CIMCube.Graphic`（下文 `G`）。通用选项见 SKILL.md §二。

| 图元 | 必填 | 关键默认 | 私有方法（超类方法不计） |
|---|---|---|---|
| `PointGraphic` | `position:Array` | `pixelSize:1`、`color:'WHITE'`、`outlineColor:'BLACK'`、`outlineWidth:0`、`heightReference:NONE`、`disableDepthTest:false`；成员 `clampToGround:false` | `setLabel(labelOptions)→LabelGraphic`、`setStyle(PointStyleOptions)` |
| `BillboardGraphic` | `position:Array`、`image:String` | `scale:1`、`color:'WHITE'`、`pixelOffset:[0,0]`（或 TwoDimensionPoint）、`heightReference:NONE`、视距四元组同点 | `setLabel(labelOptions)→LabelGraphic`、`setStyle(BillboardStyleOptions)` |
| `LabelGraphic` | `position:Array`；`text`（默认 `''`） | `font:'30px sans-serif'`、`scale:1`、`fillColor:'white'`、`outlineColor:'black'`、`outlineWidth:1`、`style:LabelStyle.FILL`、`showBackground:false`、`backgroundColor:'rgba(42,42,42,0.8)'`、`backgroundPadding:[7,5]`、`pixelOffset:[0,0]`、`horizontalOrigin:CENTER`、`verticalOrigin:CENTER`、`heightReference:NONE` | `setStyle(LabelStyleOptions)` |

```js
const G = Glodon.CIMCube.Graphic;
// 点（贴地 + 视距显示）
const pt = new G.PointGraphic({
  id: "pt01", name: "监测点",
  position: [116.391, 39.905, 12],       // [经度, 纬度, 高度(米)]
  pixelSize: 12, color: "#ffd400", outlineColor: "#000000", outlineWidth: 2,
  heightReference: G.HeightReference.CLAMP_TO_GROUND,
  distanceDisplayCondition: [0, 5000],
});
layer.addGraphic(pt);
// 广告牌（图片）
const bb = new G.BillboardGraphic({
  id: "bb01", name: "图标", position: [116.391, 39.905, 12],
  image: "./icons/camera.png", scale: 1.2, pixelOffset: [0, -18],
  verticalOrigin: G.VerticalOrigin.BOTTOM,
});
layer.addGraphic(bb);
// 文本标注
const label = new G.LabelGraphic({
  id: "lb01", name: "名称注记",
  position: [116.391, 39.905, 12], text: "A-001",
  font: "16px sans-serif", fillColor: "white", outlineColor: "black", outlineWidth: 1,
  style: G.LabelStyle.FILL_AND_OUTLINE, showBackground: true, backgroundColor: "rgba(0,0,0,0.6)",
  horizontalOrigin: G.HorizontalOrigin.LEFT, verticalOrigin: G.VerticalOrigin.BOTTOM,
});
layer.addGraphic(label);
// Point/Billboard 挂标签：返回可再加入图层的 LabelGraphic
const lg = pt.setLabel({ text: "监测点", font: "14px sans-serif", fillColor: "#ffffff" });
layer.addGraphic(lg);
```
