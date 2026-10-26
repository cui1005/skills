# Animation.LineEffect — 线条特效（5 类 · 可运行示例）

> 前提：`map` 已创建；`const E = Glodon.CIMCube.Animation`；命名空间 `Glodon.CIMCube.Animation.LineEffect.*`。
> 构造签名均为 `new Xxx(map, opt)`；生命周期多为 `show()/hide()/dispose()` + `updateOption()`。**例外**：`RopeLines` 无 `show()/hide()` **方法**（`show` 是布尔属性）；`TrailLines` 无 updateOption（仅 show/hide/dispose）。

## 1. FlyLines — 飞线（竖直）

**注意**：是**竖直飞线**（startHeight → endHeight），不是弧线飞线。多用于大屏物流链路、人口迁徙。

```js
const fl = new E.LineEffect.FlyLines(map, {
  positionList: [
    { lon: 116.4, lat: 39.9 },
    { lon: 121.47, lat: 31.23 },
  ],
  lineColor: Cesium.Color.RED,   // 默认 Red
  lineWidth: 2,                   // 默认 1px
  startHeight: 0,                 // 起始高度（米），默认 0
  endHeight: 200,                 // 终了高度（米），默认 200
  speed: 10,                      // 线速度 m/s，默认 10
  percent: 0.1,                   // 特效范围 0~1，默认 0.1
  randomStart: 0.1,               // 起始随机性，默认 0.1
  randomSpeed: 0.1,               // 速度随机性，默认 0.1
});
fl.show();
fl.updateOption({ speed: 20, lineColor: Cesium.Color.BLUE });
fl.hide(); fl.dispose();
```

## 2. DynamicLines — 动态线（6 种 lineType）

最通用的线特效，支持实线/发光线/虚线/动态线/虚线动态线/动态箭头线。

```js
const dl = new E.LineEffect.DynamicLines(map, {
  pathList: [[                      // 二维数组
    { lon: 116.40, lat: 39.90, height: 0 },
    { lon: 116.41, lat: 39.91, height: 0 },
  ]],
  lineType: 3,               // 0实线|1发光线|2虚线|3动态线|4虚线动态线|5动态箭头线
  lineColor: Cesium.Color.GREEN,
  lineWidth: 4,
  duration: 2,                // 动态线动画时长
  clampToGround: false,       // 是否贴地
  disableDepthTest: false,    // 是否关闭深度检测
  zIndex: 1,
  useBezierCurve: false,      // 贝塞尔曲线
  bezierCurve: 0.5,           // 拉伸力度
  stretchDirection: 0.5,
  glowPower: 0.2,             // 发光线场合，默认 0.2
  taperPower: 1.0,            // 发光线场合，逐渐变细
  gapColor: Cesium.Color.TRANSPARENT,  // 虚线(2)/虚线动态线(4)/动态箭头线(5)的 gap 颜色，默认透明
  dashLength: 8,   // 实线长度：虚线默认 8px、虚线动态线 8px、动态箭头线 8px
  gapLength: 8,    // 间隔长度，默认 8px（虚线动态线/动态箭头线用）
  dashWidthPattern: 1.0,  // dash 与 gap 宽度比率，默认 1.0
  image: "./assets/arrow.png", // 动态线(lineType3)/动态箭头线(lineType5)的图片 url
  blendRatio: 0,      // 0-1 颜色融合；缺省 0 表示图片颜色不参与融合
});
dl.show();
dl.setPathData(newPaths);    // 更新路径
dl.updateOption({ lineColor: Cesium.Color.RED });
dl.hide(); dl.dispose();
```

> 各 `lineType` 只使用自己的一组专有字段（虚线组 gapColor/dashLength；虚线动态线另 gapLength/dashWidthPattern；动态箭头线再需 image），示例列举仅为便于对照。

## 3. RoadLine — 道路流光

沿道路中心线做车流光，支持图片贴图 + 衰减 + 扩散。

```js
const rl = new E.LineEffect.RoadLine(map, {
  pathList: [
    { lon: 116.40, lat: 39.90, height: 0 },
    { lon: 116.41, lat: 39.91, height: 0 },
  ],
  arrowColor: Cesium.Color.GREEN,  // 线（车流方向）颜色 ← 线颜色是 arrowColor
  color: Cesium.Color.GRAY,        // 背景颜色
  reverseDirection: false,         // 方向翻转 true/false（布尔值，不是颜色！）
  width: 6,                        // 线宽
  duration: 10,                    // 跑完全程时间（秒），默认 10
  speed: undefined,                // m/s，优先级高于 duration，默认 undefined
  imageUrl: "./assets/car-flow.png",
  lineLength: 100,                 // 线段长度
  dampingStart: -1,    // 从第 n 个线段开始透明度衰减，默认 -1 不衰减
  dampingCount: -1,    // 衰减速度：第 n 个线段长度内透明度 1→0，默认 -1 不衰减
  startPercent: 0,                 // 动画开始位置（入场动画）0~1
  dashLengthPattern: 0.8,          // 图片长度占线段比例，默认 0.8
  dashWidthPattern: 1.0,           // 图片宽度占线段比例，默认 1.0
  spread: false,       // 开启扩散（需 dashWidthPattern < 1 才生效），默认 false
  spreadDuration: 2,               // 扩散特效时间，默认 2s
  spreadColor: "rgba(255,0,0,0.8)",
  clampToGround: false,            // 是否贴地，默认 false（道路演示常置 true）
  disableDepthTest: false,         // 默认 false
});
rl.show();
rl.updateOption({ speed: 20 });
rl.hide(); rl.dispose();
```

> RoadLine 可读写成员：`percent` 动画完成百分比；`pause` 暂停/继续；`arriveAtDesignatedPoint` 设定目标百分比（到达即停止）；`startPercent` 入场起点。

## 4. RopeLines — 圆柱线

物理感更强的圆柱线（带粗糙度/金属度/贴图），适合索道、吊缆、跨江线缆。

```js
const rope = new E.LineEffect.RopeLines(map, {
  pathList: [[{ lon: 116.40, lat: 39.90, height: 50 }, { lon: 116.41, lat: 39.91, height: 50 }]],
  color: new Cesium.Color(0.1, 0.1, 0.1, 1),  // 默认即此
  radius: 0.02,                // 圆柱半径，默认 0.02
  slices: 16,                  // 精细程度，默认 16
  shadowMode: Cesium.ShadowMode.ENABLED,
  show: true,                  // 初始是否显示
  texture: "./assets/cable.jpg",  // 漫反射贴图 URL；为空时无光照平面着色
  textureRepeat: { x: 4, y: 1 }, // 有贴图时 {x/u=周向, y/v=轴向} 平铺倍数
  roughness: 0.82,             // 粗糙度 0~1，默认约 0.82（有纹理时生效）
  metallic: 0.5,               // 金属度 0~1，默认 0.5（有纹理时生效）
});
rope.show = false;      // ⚠️ 无 show()/hide() 方法，show 是布尔属性（get/set）
rope.updateOption({ roughness: 0.5 });   // 纯色 ↔ 贴图不支持运行时切换
rope.updatePosition(0, [                 // positions 必须成对两顶点，否则直接忽略
  { lon: 116.40, lat: 39.90, height: 50 },
  { lon: 116.42, lat: 39.92, height: 50 },
]);
rope.dispose();
```

## 5. TrailLines — 轨迹线（带箭头）

GPX 轨迹回放、车辆轨迹展示，默认渐进色 + 可选箭头。

```js
const tl = new E.LineEffect.TrailLines(map, {
  pathList: [[{ lon: 116.40, lat: 39.90, height: 0 }, { lon: 116.41, lat: 39.91, height: 0 }]],
  lineType: 0,                  // 0 实线 | 2 虚线
  lineColor: Cesium.Color.YELLOW,
  lineWidth: 2,
  duration: 5,
  clampToGround: false,
  imageUrl: "./assets/trail.png",  // 默认渐进色图片
  startTime: 0,
  repeat: true,                    // 是否循环
  lineLengthLimit: 300,            // 线段长度限制（米），超出自动切割
  nearDistanceDisplay: 0,          // 近点可视范围
  farDistanceDisplay: Infinity,    // 远点可视范围
  gapColor: Cesium.Color.TRANSPARENT,
  dashLength: 64,                  // 虚线长度，默认 64
  arrowImageUrl: "./assets/arrow.png",
  arrowLength: 3,                  // 箭头长度，默认 3（相邻像素屏幕空间距离差）
  arrowWidth: 3,                   // 箭头宽度（线宽的倍数），默认 3
});
tl.show();
tl.hide(); tl.dispose();
```

> TrailLines 仅 show/hide/dispose（无 setPathData/updateOption），变更轨迹需重建实例。

## 性能建议

| 类 | 开销 | 建议 |
|---|---|---|
| FlyLines | 低 | 50~200 条中；>1000 条合并批次或降级静态线 |
| DynamicLines | 低 | lineType=1 发光线稍重；clampToGround 略增开销 |
| RoadLine | 低 | 1~3 条道路同时动 |
| RopeLines | 低 | roughness/metallic 仅在有纹理时生效 |
| TrailLines | 低 | lineLengthLimit 300 自动切割避免长线性能问题 |
