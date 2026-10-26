# 样式体系（三个入口，别混）

命名空间 `Glodon.CIMCube.Graphic`（下文 `G`）。

> ⚠️ **`setOptions` 即将废弃**：不要再写 `graphic.setOptions({...})`。构造后修改样式统一用 **`setStyle(xxxStyleOptions)`**。

| 入口 | 说明 |
|---|---|
| 构造 `opts` | 顶层直接写 `pixelSize/color/width/material/...`；可选 `style` 传 `PointStyleOptions` 等 `xxxStyleOptions`，其优先级最高 |
| `setStyle(styleOptions)` | **构造后改样式的唯一推荐写法**。传对应图元的 `xxxStyleOptions`（同 user-guide 405） |
| `setHighlight(options)` | Polygon `{fillColor, outline:{color,width}}`、Polyline `{color,width}`——悬停/选中高亮，不改变基础样式 |

## setStyle 覆盖清单（API_DOC + 运行时双向核实）

| 覆盖情况 | 图元 |
|---|---|
| ✅ **有 `setStyle`** | `PointGraphic`、`LabelGraphic`、`BillboardGraphic`、`PolygonGraphic`、`PolylineGraphic`、`WallGraphic` |
| ⛔ **无 `setStyle`** | `RectangleGraphic`、`EllipseGraphic`、`CorridorGraphic`、`CylinderGraphic`、`EllipsoidGraphic`、`PolylineVolumeGraphic`、`BoxGraphic`、`PlaneGraphic` —— 改样式请在**构造 opts 里一次性给全**（或销毁后按新 opts 重建图元） |

> 运行时依据：有 `setStyle` 的图元内部走 `XxxAttr.config2Entity(e, this.cesiumEntity.*)`；其余图元当前只保留即将废弃的 `setOptions`（`XxxAttr.style2Entity`）。**不要**对上表 ⛔ 行写 `setStyle` 或 `setOptions`。

```js
const G = Glodon.CIMCube.Graphic;
// 点：改大小 / 勾边 / 视距
pt.setStyle({ pixelSize: 18, outline: true, outlineColor: "#ff0000", outlineWidth: 2,
              distanceDisplayCondition: [0, 20000],
              scaleByDistance: [0, 1, 20000, 0.5] });
// 线：线型（solid/dash/glow/arrow）
line.setStyle({ type: "glow", color: "#00c6ff", width: 6 });
// 线：被遮挡部分单独着色
line.setStyle({ type: "dash", color: "#00c6ff", width: 4,
                occludedLine: { type: "dashes", color: "#ffff00", opacity: 1,
                                gapColor: "#000000", gapOpacity: 0 } });
// 高亮（不改变基础样式）
poly.setHighlight({ fillColor: "rgba(255,212,0,0.85)", outline: { color: "#ff8c00", width: 3 } });
// setStyle 传 xxxStyleOptions（与 user-guide 405 的 FeatureStyle 同构）
poly.setStyle(PolygonStyleOptions);
wall.setStyle({ fill: true, color: "#8ab4ff", outline: true, outlineColor: "#0a5ac2",
                outlineWidth: 1, extrudedHeight: 10 });
```

> ⚠️ `setStyle` 必须传对应的 `PointStyleOptions`/`LabelStyleOptions`/`BillboardStyleOptions`/`PolygonStyleOptions`/`PolylineStyleOptions`/`WallStyleOptions`，**不是普通 Object**；schema 见 <https://cimcube.glodon.com/openservice/user-guide/405>。
