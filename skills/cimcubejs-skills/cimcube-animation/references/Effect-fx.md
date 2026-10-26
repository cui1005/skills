# Animation.Effect — 光效与后处理（3 类 · 可运行示例）

> 前提：`map` 已创建；`const E = Glodon.CIMCube.Animation`；命名空间 `Glodon.CIMCube.Animation.Effect.*`。公共初始化与组合模式见 `references/patterns.md`。
> 本族 3 类：`CameraShooting` / `AmbientOcclusion` / `Metaptosis`。
> **控制方法**：均为 **show/hide 族**（`CameraShooting` 另 `playPathAnimation`、`AmbientOcclusion` 另 `updateOption`；`Metaptosis` 仅 `show()`/`hide()`/`dispose()`）。**勿按基类 `Animation`（仅 `dispose`/`play`/`stopAutoDispose`）猜方法**。
> 同域其他族：`Effect-roaming.md`（相机与角色漫游 5 类）· `Effect-entity.md`（锚点·体块·介质 6 类）。

## 1. CameraShooting — 摄像机扫描光束（探照灯扫掠）

一道光束从光源 `emitPosition` 扫向目标区 `lightPosition`，可沿时间轴摆动。注意：这是**光效**（探照/扫描），不是"相机拍摄"。

```js
const cs = new E.Effect.CameraShooting(map, {
  emitPosition: { lon: 116.40, lat: 39.90, height: 100 },  // 光源|信号源
  lightPosition: { lon: 116.41, lat: 39.90, height: 0 },   // 目标点
  lightBeamAlpha: 0.8,    // 光束透明度
  lightAlpha: 0.5,        // 地面光透明度
  borderAlpha: 1.0,       // 描边透明度，默认 1.0
  width: 50,              // 地面光宽度，默认 50 米
  color: Cesium.Color.CYAN, // 光颜色
});
cs.show();
cs.playPathAnimation([
  // 光束摆动轨迹点：{timeStamp, lightPosition, width}
  { timeStamp: 0, lightPosition: { lon: 116.40, lat: 39.90, height: 0 }, width: 30 },
  { timeStamp: 5, lightPosition: { lon: 116.42, lat: 39.90, height: 0 }, width: 60 },
], true);                 // loopAnimation 是否循环，默认 true
cs.hide(); cs.dispose();
```

## 2. AmbientOcclusion — 环境光遮蔽（AO）

全局后处理 AO（物体相接处的环境阴影），作用于整帧画面。

```js
const ao = new E.Effect.AmbientOcclusion(map, {
  intensity: 1.0,     // 遮蔽强度（默认 1.0，建议 0~1）
  range: 1.0,         // 遮蔽范围（默认 1.0，建议 0~1）
  blurStepSize: 1.0,  // 模糊处理步长（默认 1.0，建议 0~2）
  bias: 1.0,          // 遮蔽模糊偏移（默认 1.0，建议 0~1）
  debugShowAmbientOcclusion: false,  // 只显示 AO 图
  debugShowNormal: false,            // 只显示法线图
});
ao.updateOption({ intensity: 1.5 });
ao.show(); ao.hide(); ao.dispose();
```

## 3. Metaptosis — 迁徙特效

中心"菱形锥/锥台模型" + 底部波纹扩散 + 从目标点飞向（或由中心发散）的**迁徙线**，用于人流迁徙 / 信号汇聚展示。APIDOC 语义即"迁徙特效"（误称"视角变形/细胞凋亡"均不准确）。

```js
const meta = new E.Effect.Metaptosis(map, {
  // 中心模型（几何参数同 PrismPoint）
  originPosition: { lon: 116.40, lat: 39.90, height: 0 },  // 中心模型位置
  numberOfPlies: 4, slices: 4, height: 10,   // 层/面/大小（默认 4/4/10）
  color: Cesium.Color.BLUE,                  // 默认蓝
  silhouetteColor: Cesium.Color.WHITE,       // 轮廓默认白
  baseAlpha: 0.5, increaseAlpha: 0.2, topRadius: 1,
  // 中心下方图片/扩散（光圈波纹）
  centerColor: Cesium.Color.WHITE, centerRadius: 1,  // 中心点圆心
  centerImageUrl: undefined,                 // 中心点图片效果
  imageUrl: "./assets/ripple.png",           // 扩散波纹图片
  radius: 50, duration: 2,                   // 波纹扩散半径/动画时长
  moireColor: Cesium.Color.WHITE,            // 扩散颜色（与图片颜色求与）
  postProcessStage: false,                   // 是否使用后期
  // 目标点扩散
  targetPointRadius: 20,
  targetImageUrl: "./assets/target.png",
  // 迁徙线（APIDOC 中目标点列表 targetPosition 与线数据 targetPositionList 并存，按需选用）
  targetPosition: [{ lon: 116.30, lat: 39.80, height: 100 }],
  targetPositionList: [{ lon: 116.30, lat: 39.80, height: 100 }],
  lineUrl: "./assets/line.png",   // 动态线的贴图 url
  lineColor: Cesium.Color.CYAN, lineWidth: 2,
  bezierCurve: 0.3,             // 贝塞尔拉伸力度
  stretchDirection: 0.5,        // 拉伸方向 0~1（默认 0.5；越大越接近终点）
  lineType: 3,                  // 0 实线 | 1 发光线 | 2 虚线 | 3 动态线
  lineDuration: 3,              // 动态线场合动画时长
  glowPower: 0.2,               // 发光线场合发光亮度（默认 0.2）
});
meta.show(); meta.hide(); meta.dispose();   // 仅 3 个控制方法
```

## 性能建议

- `AmbientOcclusion` 是**全局后处理**（作用于整帧画面，非局部特效），与 `map.scene.realisticRendering` 的 Bloom / EyeAdaptation 叠加时注意整体开销。
- `Metaptosis` 同时含模型 + 波纹 + 迁徙线，`targetPositionList` 点数建议控制在百级。

**最佳实践**：SPA 路由切换或演示结束 → 调 `dispose()` 而非 `hide()`（否则 GPU 资源泄漏）。
