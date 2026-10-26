# Animation.PointEffect — 点动态（4 类 · 可运行示例）

> 前提：`map` 已创建；`const E = Glodon.CIMCube.Animation`；命名空间 `Glodon.CIMCube.Animation.PointEffect.*`。
> 方法差异：仅 `Dynamic` 有 `play()/stop()`；`Scatter` / `Twinkle` 只有 `show()/hide()/dispose()/updateOption()`；`InstanceBillboard` 只有 `show()/hide()/dispose()` + 6 个 `setXxx`（无 `updateOption`）。**勿对 Scatter/Twinkle/InstanceBillboard 调 `play()`**。

## 1. Dynamic — 动态点轨迹

沿路径运动的点轨迹（不同于 LineEffect.TrailLines：Dynamic 是点序列）。

```js
const dp = new E.PointEffect.Dynamic(map, {
  pathList: [              // 点轨迹 {lon,lat,height}
    { lon: 116.40, lat: 39.90, height: 0 },
    { lon: 116.41, lat: 39.91, height: 0 },
  ],
  pointCount: 50,          // 时间内点的数量
  pointDistance: 5,        // 点与点距离（米），设置后 pointCount 失效
  size: 10, minSize: 3,
  image: "./assets/point.png",
  duration: 5,              // 点动画时长
  breathDuration: 1,        // 呼吸动画时长
  viewingDistance: 200,
  useBezierCurve: false,
  bezierCurve: 0.5,         // 贝塞尔拉伸力度
  stretchDirection: 0.5,    // 拉伸方向 0~1
});
dp.play(); dp.stop();
dp.setPathData(newPath);   // 更新路径
dp.updateOption({ size: 15 });
dp.show(); dp.hide(); dp.dispose();
```

## 2. Twinkle — 闪烁/呼吸点

事故告警、POI 高亮。

```js
const tw = new E.PointEffect.Twinkle(map, {
  positionList: [{ x: 116.41, y: 39.90, z: 2 }],   // 注意是 {x,y,z}
  size: 20, minSize: 5,
  image: "./assets/warn.png",
  onlyExsufflation: false,   // true 时只有扩散效果
  breathDuration: 1.2,       // 呼吸动画时长
  viewingDistance: 200,
  openDepthTest: true,
});
tw.show();
tw.updateOption({ size: 25 });
tw.hide(); tw.dispose();
```

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `positionList` | Array | - | 点的数据 `{x,y,z}` |
| `size` | Number | - | 点大小（最大值） |
| `minSize` | Number | - | 点大小最小值 |
| `image` | Url | - | 点图片 url |
| `onlyExsufflation` | Boolean | - | 只有扩散效果 |
| `breathDuration` | Number | - | 呼吸动画时长 |
| `viewingDistance` | Number | 200 | 最佳观测距离 |
| `openDepthTest` | Boolean | true | 深度检测（不支持实时变动） |

## 3. Scatter — 散点扩散

水波纹扩散效果（多圈同心圆）。

```js
const sc = new E.PointEffect.Scatter(map, {
  positions: [{ x: 116.41, y: 39.90, z: 0 }],   // {x,y,z}
  color: Cesium.Color.YELLOW,
  size: 20,                  // 圆直径，默认 20
  spreadRatio: 4,            // 扩散范围倍数，>1，默认 4
  duration: 2,
  interval: 1,               // 动画等待时长
  totalWaves: 3,             // 扩散浪数
  visibleWaves: 2,           // 可见层数
  waveAlpha: 1,
  synchronous: true,         // 是否同步
  viewDistance: 200,
  depthTest: true,
});
sc.show();
sc.updateOption({ color: Cesium.Color.RED });
sc.hide(); sc.dispose();
```

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `positions` | Array | - | 点数据 `{x, y, z}` |
| `color` | Color | 黄 | 点颜色 |
| `size` | Number | 20 | 圆直径 |
| `spreadRatio` | Number | 4 | 扩散为点大小的倍数，须 >1 |
| `duration` | Number | - | 点动画时长 |
| `interval` | Number | - | 动画等待时长 |
| `totalWaves` | Number | 3 | 扩散浪数 |
| `visibleWaves` | Number | 2 | 可见扩散层数 |
| `waveAlpha` | Number | 1 | 扩散层透明度 |
| `synchronous` | Boolean | true | 动画是否同步 |
| `viewDistance` | Number | 200 | 最佳观测距离（点大小随距离变）；设 0 则固定点大小 |
| `depthTest` | Boolean | true | 深度检测；不支持实时变动 |

## 4. InstanceBillboard — 实例化看板（大量 POI）

> **1000+ 个 POI** 用 InstanceBillboard 而非 GraphicLayer.BillboardGraphic（性能差距 10x+）。

```js
const ib = new E.PointEffect.InstanceBillboard(map, {
  positionList: [
    { lon: 116.40, lat: 39.90, height: 10 },
    { lon: 116.41, lat: 39.91, height: 10 },
    // ... 10000 个点
  ],
  image: "./assets/poi.png",
  bottomImage: "./assets/poi-base.png",
  imageScale: 1,
  bottomImageScale: 1,
  bottomImageOffset: 0,
});
ib.show();
// 动态更新
ib.setImage("https://example.com/new-poi.png");
ib.setImageScale(1.5);
ib.setBottomImage("./assets/poi-base-2.png");
ib.setBottomImageScale(1.2);
ib.setBottomImageOffset(10);
ib.setPositionList(newPositions);
ib.hide(); ib.dispose();
```

## 性能建议

| 类 | 开销 | 建议 |
|---|---|---|
| Twinkle | 低（像素点动画） | 1000 个内安全 |
| Scatter | 低 | 同步模式 synchronous=true 更省 |
| Dynamic | 低~中 | pointCount <200；pathList 段数 <500 |
| InstanceBillboard | 低（GPU 实例化） | 10000 个 POI 用这个，不用 GraphicLayer |
