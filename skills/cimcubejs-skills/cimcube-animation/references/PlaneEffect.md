# Animation.PlaneEffect — 平面特效（8 类 · "科技感"来源 · 可运行示例）

> 前提：`map` 已创建；`const E = Glodon.CIMCube.Animation`；命名空间 `Glodon.CIMCube.Animation.PlaneEffect.*`。
> 构造签名均为 `new Xxx(map, opt)`；多为 `show()/hide()/dispose()` + `updateOption()`。**例外**：`Moire` 无 updateOption（仅 show/hide/dispose，运行时可改 `zIndex` 成员）。

## 1. Radar — 雷达扫描（圆形）

```js
const radar = new E.PlaneEffect.Radar(map, {
  originPosition: { lon: 116.4, lat: 39.9, height: 0.2 },
  color: Cesium.Color.fromCssColor("#1a73e8"),
  duration: 4,            // 动画时长（秒），默认 1
  radius: 800,            // 半径（米），默认 10
  progressive: 1,         // 颜色衰减力度，默认 1
  roundRim: true,         // 圆形描边，默认 true
  anticlockwise: true,    // 逆时针，默认 true
  projection: false,      // 投影方式
});
radar.show();
radar.updateOption({ duration: 6, color: Cesium.Color.GREEN });
radar.hide(); radar.dispose();
```

> 与 `Animation.Effect.Radar`（光束扫描）不同命名空间，同属雷达族。

## 2. RingScan — 波纹圆面（多圈）

```js
const rs = new E.PlaneEffect.RingScan(map, {
  originPosition: { lon: 116.4, lat: 39.9, height: 0.2 },
  color: Cesium.Color.YELLOW,
  duration: 1,            // 默认 1
  radius: 500,            // 默认 10
  progressive: 1,
  count: 3,               // 波纹个数，默认 1
  projection: false,
});
rs.show();
```

## 3. DiffuseWall — 扩散墙（从中心向外）

```js
const dw = new E.PlaneEffect.DiffuseWall(map, {
  center: [116.40, 39.90, 0],   // 必填，中心位置
  height: 100,                   // 墙高，默认 100
  color: Cesium.Color.YELLOW,
  radius: 100,                   // 扩散墙半径，默认 100
  edge: 32,                      // 扩散墙边数，默认 32（圆形）
  time: 3,                       // 扩散效果时间，默认 3s
});
dw.show();
```

## 4. Dynamic — 动态面（贴图流动）

```js
const dyn = new E.PlaneEffect.Dynamic(map, {
  pathList: [{ lon: 116.40, lat: 39.90, height: 0 }],
  imageUrl: "./assets/water.png",
  color: Cesium.Color.WHITE,   // 设白色相当于不传
  duration: 4,
  direction: 0,
});
dyn.show();
dyn.updateOption({ duration: 2 });
dyn.hide(); dyn.dispose();
```

## 5. Moire — 扩散旋转特效

波纹扩散 + 旋转 + 呼吸 + 透明度渐变的复合平面特效。APIDOC 语义为「扩散旋转特效」——别处误称"莫尔条纹"是误读。**仅 show/hide/dispose 三个方法（无 updateOption）**，动态参数须构造时确定；运行中只能改 `zIndex`（get/set 成员）。

```js
const moire = new E.PlaneEffect.Moire(map, {
  originPosition: { lon: 116.4, lat: 39.9, height: 0.2 }, // 模型位置（圆心）
  radius: 500,                      // 波纹扩散半径（米）
  imageUrl: "./assets/ripple.png",  // 图片效果
  projection: false,                // 是否使用投影方式
  disableDepthTest: false,          // 是否关闭深度检测，默认 false
  duration: 3,                      // 动画时长（一轮）
  startTime: 0,                     // 动画开始位置（按时长比例），默认 0
  repeat: true,                     // 是否循环
  breathe: false,                   // 是否呼吸方式
  rotate: true,                     // 是否旋转
  rotationClockwise: true,          // 是否顺时针旋转，默认 true
  scale: true,                      // 是否扩散（半径渐大）
  scaleLargen: true,                // 是否渐变大，默认 true（scale 从 0→1）
  alpha: false,                     // 是否透明度渐变
  alphaLimpid: true,                // 是否渐变清晰，默认 true（alpha 从 0→1）
  zIndex: 0,                        // 渲染先后顺序，默认 0
});
moire.show(); moire.hide(); moire.dispose();
moire.zIndex = 5;   // 运行时仅可改 zIndex 成员
```

## 6. Sequence — 序列贴图面

把多张序列图（雪碧图）逐帧贴在面上播放，常用于动态流动/水纹/波动贴图。

```js
const seq = new E.PlaneEffect.Sequence(map, {
  pathList: [   // 顶点序列；长度 2 时按矩形处理
    { lon: 116.40, lat: 39.90, height: 0 },
    { lon: 116.40, lat: 39.91, height: 0 },
    { lon: 116.41, lat: 39.91, height: 0 },
    { lon: 116.41, lat: 39.90, height: 0 },
  ],
  imageUrlList: ["./assets/seq-1.png", "./assets/seq-2.png"], // 图片路径数组
  color: Cesium.Color.WHITE,  // 与图片颜色融合，默认 WHITE
  duration: 1,     // 整个序列图的持续时间（秒），默认 1
  count: 4,        // 每行画面个数；要求序列图为正方形（行列一致），默认 1
  projection: false,        // 默认 false
  disableDepthTest: false,  // 默认 false
});
seq.setPathList(newPaths);
seq.show();
seq.updateOption({ imageUrlList: [...], color: Cesium.Color.RED, duration: 2 });
seq.dispose();
```

> 每帧时长 = duration / 图数 / count；各序列图须正方形且 count 一致。

## 7. Statics — 静态面

贴静态纹理的面（可选入场渐出动画与旋转角），"动态"交给贴图内容或光照。

```js
const stat = new E.PlaneEffect.Statics(map, {
  pathList: [   // 顶点序列；长度 2 时按矩形处理
    { lon: 116.40, lat: 39.90, height: 0 },
    { lon: 116.40, lat: 39.91, height: 0 },
    { lon: 116.41, lat: 39.90, height: 0 },
  ],
  imageUrl: "./assets/panel.png",  // 图片路径
  color: Cesium.Color.WHITE,       // 与图片颜色融合
  duration: 0,        // 入场渐出动画时长（秒），默认 0 = 无入场动画
  rotation: 0,        // 旋转角度（度）
  projection: false,  // 是否使用投影方式
});
stat.setPathList(newPaths);
stat.show();
stat.updateOption({ imageUrl: "./assets/panel2.png", color: Cesium.Color.GREEN, rotation: 90, duration: 1 });
stat.dispose();
```

## 8. StaticsRingScan — 圆环特效

静态圆环（内/外圆半径 + 填充/描边颜色），**无扫描动画**。APIDOC 语义为「圆环特效」，勿按类名直译成"静态波纹圆面"。

```js
const srs = new E.PlaneEffect.StaticsRingScan(map, {
  originPosition: { lon: 116.4, lat: 39.9, height: 0.2 }, // 圆心
  innerRadius: 8,                      // 内圆半径（米），默认 8
  outerRadius: 10,                     // 外圆半径（米），默认 10
  fillColor: Cesium.Color.CYAN,        // 填充颜色
  outlineColor: Cesium.Color.fromCssColor("#00FF9D"), // 描边颜色，默认 #00FF9D
});
srs.show();
srs.updateOption({ outerRadius: 50, fillColor: Cesium.Color.GREEN });
srs.dispose();
```

## 性能建议

| 类 | 开销 | 建议 |
|---|---|---|
| Radar / RingScan | 低~中 | 不建议 >20 个同时开；z 抬 0.1~0.5m 防 z-fighting |
| DiffuseWall | 低~中 | edge 边数 32 已足够；>64 不明显 |
