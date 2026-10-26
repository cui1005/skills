# Utils 克隆与破碎 — SpatialTilesetCloner / ConvexObjectBreaker

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

## 一、SpatialTilesetCloner — 3DTiles 范围克隆

对**已上地图**的 3DTiles 做闭合范围提取，克隆一份并支持抬升偏移、渐进着色、LoD 降级、端点垂直连线与入场动画。

```js
const cloner = new U.SpatialTilesetCloner(map, {
  layers: obliqueLayer,                        // Tiles3DLayer 或数组（需已 addLayer）
  region: [[106.71, 29.60, 0], [106.77, 29.59, 0], [106.75, 29.55, 0]], // 闭合范围；范围判定仅用经纬度，高度用于连线
  offset: [0, 0, 200],                         // 东/北/上偏移（米）
  simplify: true, simplifyMaxPoints: 30,       // 道格拉斯-普克抽稀
  gradient: { /* 见 Tiles3DLayer.setGradientEffect */ },
  lodFactor: 2,                                // LoD 降级系数（>0）
  showConnectorLines: true,                    // 原始端点 ↔ 抬升端点的垂直连线
  connectorLineStyle: { lineColor: "#FFEB3B", lineWidth: 2, lineType: /* DynamicLines 线型 */ },
  animationDuration: 3,                        // 入场动画秒数（0=无；图层 LOADED 后开始）
});
cloner.activate();          // 执行：复制图层→范围裁剪→着色→降 LOD→连线
console.log(cloner.region, cloner.simplifiedRegion); // 原始范围 / 抽稀后轮廓
cloner.deactivate();        // 停止动画并移除克隆图层与连线（deactivate(true) 同默认）
cloner.dispose();
```

> 动态连接线的线型/颜色等与 `Animation.LineEffect.DynamicLines`（cimcube-animation）一致。克隆得到的图层仍在 layerManager 中，可按普通图层管理。开销：**高（瓦片复制）**——region 尽量小；`lodFactor` 抬高；动画完无需连线则关。

## 二、ConvexObjectBreaker — 凸包破碎（物理）

从 three.js 移植的破碎工具：用撞击点/法线附近平面反复切割凸包，产出碎块几何（`position/quaternion/geometry.attributes.position+normal` 非索引 soup、`userData.mass/velocity/...`）。

```js
const breaker = new U.ConvexObjectBreaker(1.4, 0.0001); // minSizeForBreak(米) / smallDelta
const obj = breaker.prepareBreakableObject(convexObj, mass, velocity, angularVelocity, true);
// ① 直接平面切割：Cesium.Plane 位于源几何局部空间
const plane = Cesium.Plane.fromPointNormal(point, normal);
breaker.cutByPlane(obj, plane);             // → { object1, object2 }（任一侧为空则 null）
// ② 模拟撞击破碎
const debris = breaker.subdivideByImpact(obj, impactPoint, normal, 1, 2); // 径向/随机迭代次数；返回碎块数组，无法细分返回 []
```

> 供碰撞/破坏场景（结合 `Animation.Effect` 播放碎块轨迹）。若只是几何拆分凸多面体做体积/可视化，可考虑 `Graphic` 与 measure 的体积工具，不一定需要本类。

## 性能

| 工具 | 开销 | 建议 |
|---|---|---|
| SpatialTilesetCloner | 高（瓦片复制） | region 尽量小；`lodFactor` 抬高；动画完无需连线则关 |
| ConvexObjectBreaker | 中（几何切割） | 迭代次数受控；碎块数量别失控 |
