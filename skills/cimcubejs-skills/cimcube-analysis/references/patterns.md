# Analysis 域公共初始化与常见组合模式

> 公共初始化三域通用；组合模式按业务分组（与 SKILL.md 节号 ①②③ 对应）。

## 公共初始化（所有 references 示例的前提）

```js
const map = new Glodon.CIMCube({
  id: "cimcubeContainer", gisResourceUrl: "./dist/",
  center: { lon: 114.3, lat: 30.5, height: 3000, pitch: -Math.PI / 3 }, // 示例中心，按项目改
});
// 先加载参与分析的 3D 数据（建筑 S3M/3DTiles、地形/DEM、管线、模型），详见 cimcube-layer
// const tiles = await map.layerManager.add(...);

const A = Glodon.CIMCube.Analysis;                  // 分析命名空间
const Cesium = Glodon.CIMCube.Cesium;               // SDK 内嵌 Cesium 内核
const FINISH = Glodon.CIMCube.Global.Defines.MapActionType.FINISH; // CutFill 完成事件
// 视域/通视/开敞度等类要求 Cartesian3 坐标（ECEF 米），用该辅助函数从经纬度构造：
const llh = (lon, lat, height = 0) => Cesium.Cartesian3.fromDegrees(lon, lat, height);
```

## ① 地形与土方组合模式

### 模式 A：多标高填挖方案对比

```js
async function twoSchemesCompare(boundary) {
  const r1 = await runCutFill(boundary, 45);
  const r2 = await runCutFill(boundary, 48);
  compareTable({ "方案一（45m）": r1, "方案二（48m）": r2 });
}
function runCutFill(boundary, level) {
  const cf = new A.CutFill(map, { boundary, level, showVisualization: false });
  return new Promise((resolve) => {
    cf.on(FINISH, () => resolve({
      fill: cf.getFillVolume(), cut: cf.getCutVolume(),
      net: cf.getFillVolume() - cf.getCutVolume(),   // 填-挖净量
    }));
  });
}
```

### 模式 B：一键用地适宜性评价（坡度 + 径流 + 高程热力）

```js
const bbox = [114.28, 30.48, 114.32, 30.52];
const slope = new A.SlopeAnalysis(map, { region: bbox });   // 自动计算
const runoff = new A.Runoff(map, { region: bbox, unitGrid: 5 });
runoff.doAnalysis();
const heat = new A.ElevationHeatMap(map, { region: bbox });
// 三者同范围叠加：坡度→选平缓区，径流→避行洪通道，热力→看地形高差
```

### 模式 C：范围/折线来源

分析范围与折线均可让用户绘制：多边形范围给 `CutFill.boundary` / `TerrainTINAnalysis.boundary`，折线转成 `Cartographic[]` 给 `SectionAnalysis.doAnalysis`——取点见 cimcube-controls 的 `TakeCoordinates`。

## ② 可视与规划组合模式

### 模式 A：基站/监控点视域覆盖评估（视域 + 通视抽查）

```js
const vs = new A.ViewShedAnalysis(map, { viewPosition: llh(lon, lat, h), visualRange: 1000, horizontalViewAngle: 120 });
vs.doAnalysis();
// 关键目标点逐个通视校验，统计可视率
const checks = [
  { name: "路口A", p: llh(lx1, ly1, 1.5) },
  { name: "路口B", p: llh(lx2, ly2, 1.5) },
];
checks.forEach((c, i) => {
  const v = new A.VisibilityAnalysis(map, { startPos: llh(lon, lat, h), endPos: c.p });
  v.doAnalysis();
  window.setTimeout(() => { if (i === checks.length - 1) { /* 截屏汇总 */ } }, 100);
});
// 用完统一 dispose
```

### 模式 B：规划方案「退线 → 限高 → 视廊」三步合规核查

```js
new A.BackLine(map, { positions: redlinePts, width: 10, height: 8 }).doAnalysis();
new A.LimitHeight(map, { positions: flatPolygon, limitHeight: 45 }).doAnalysis();
new A.ViewCorridorAnalysis(map, { startInfo, endInfo }).doAnalysis();
```

### 模式 C：现状模型拍平后叠加规划方案

```js
new A.ModelFlat(map, { layers: [oldTiles], positions: redevArea, height: 0 }).doAnalysis();
// 然后在拍平区域上叠加新方案 S3M / 白模图层
```

## ③ 日照与环境组合模式

### 模式 A：从鼠标绘制范围直接驱动 FloodAnalysis

```js
// 参考 cimcube-controls：用 TakeCoordinates 逐个取范围顶点坐标，业务自组 PolygonGraphic
const coords = await takePolygon(map);            // 伪代码：TakeCoordinates mode:"polygon"
const area = new Glodon.CIMCube.Graphic.PolygonGraphic({
  positions: coords, style: { fill: "rgba(26,115,232,0.2)" },
});
map.layerManager.getLayer("graphics").add(area);  // PolygonGraphic 需挂图层显示

const flood = new A.FloodAnalysis(map);
flood.startByTerrain(area, { minHeight: 0, maxHeight: 35 });
```

### 模式 B：体块 + 地形两阶段淹没汇报

先体块（看建筑浸水高度）后地形（看道路/地块泛洪范围），便于一张图对比两种风险：

```js
flood.startByGraphic(area, { minHeight: 0, maxHeight: 60 });
// ... 体块演示结束后
flood.stopByGraphic();
flood.startByTerrain(area, { minHeight: 0, maxHeight: 45 });
flood.stopByTerrain();
```
