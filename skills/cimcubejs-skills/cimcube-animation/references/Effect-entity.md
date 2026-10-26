# Animation.Effect — 锚点·体块·介质特效（6 类 · 可运行示例）

> 前提：`map` 已创建；`const E = Glodon.CIMCube.Animation`；命名空间 `Glodon.CIMCube.Animation.Effect.*`。公共初始化与组合模式见 `references/patterns.md`。
> 本族 6 类：`BillboardEntity` / `PrismPoint` / `VideoEntity` / `PolylineVolumeFlow` / `Wall` / `Radar`。
> **控制方法与释放方式各不同，勿按基类 `Animation`（仅 `dispose`/`play`/`stopAutoDispose`）猜方法**：
> - **show/hide 族**：`BillboardEntity`、`VideoEntity`（另 `play`/`stop`）、`Wall`（另 `setPathData`/`updateOption`）、`Radar`（仅 show/hide/dispose）
> - **事件派发型**：`BillboardEntity`、`PrismPoint` 继承 `EventDispatcherSDK`，提供 `on`/`off`/`fire` 与 `addEventListener` 全家（**无 `once`**）；`BillboardEntity` 官方标注「即将废弃」，慎用
> - **`PolylineVolumeFlow` 特例**：释放方法为 `destroy()`（**无 `dispose`**）；另有 `show`/`hide`/`updateOption`/`setPathData`/`setRadius`/`setCornerType`/`setStyle`
> 同域其他族：`Effect-roaming.md`（相机与角色漫游 5 类）· `Effect-fx.md`（光效与后处理 3 类）。

## 1. BillboardEntity — Cesium 锚点实体（⚠️ 官方标注「即将废弃」）

用 Cesium 原生 Entity 方式创建锚点（位置/图片/文字标签），可响应点击事件。继承 `EventDispatcherSDK`。**APIDOC 明确标注"即将废弃"，新项目慎用**；可评估替代：PrismPoint（立体锚点）或 cimcube-graphic 的 `BillboardGraphic`。

```js
const be = new E.Effect.BillboardEntity(map, {
  originPosition: { lon: 116.40, lat: 39.90, height: 0 },  // 锚点位置
  billboard: { imageUrl: "./assets/marker.png", width: 48, height: 48, scale: 1 },
  label: { text: "烟感器", font: "12pt Microsoft YaHei", fillColor: "#ffffff" },
  disableDepthTestDistance: 1000,  // 相机离图形距离 > 该值(米) 则开启深度检测
});
be.on(Glodon.CIMCube.Global.Defines.ScreenSpaceEventType.LEFT_CLICK, (e) => {
  console.log(e);   // 唯一内置事件：LEFT_CLICK 鼠标左键点击
});
be.addEventListener("CUSTOM", handler);   // 自定义事件亦可
be.dispatchEvent({ type: "CUSTOM" });
be.hasEventListener("CUSTOM");
be.removeEventListener("CUSTOM", handler);
be.removeEventListeners();               // 不传 type 时删除全部
be.show(); be.hide(); be.dispose();
```

`label` 可用项：text/font/fillColor/outlineWidth/outlineColor/pixelOffsetX/pixelOffsetY/showBackground/backgroundColor/backgroundPaddingX/backgroundPaddingY。

## 2. PrismPoint — 立体锚点（菱形锥模型，事件派发 + 模型动画）

生成一个菱形锥立体模型作为锚点（APIDOC 语义即"立体锚点·菱形锥"），支持旋转动画与鼠标事件。继承 `EventDispatcherSDK`。

```js
const pp = new E.Effect.PrismPoint(map, {
  originPosition: { lon: 116.40, lat: 39.90, height: 5 },  // 模型位置
  numberOfPlies: 4, slices: 4,   // 层数 / 面数，默认 4 / 4
  height: 10,                    // 模型大小，默认 10
  color: Cesium.Color.BLUE,                // 模型颜色，默认蓝色
  silhouetteColor: Cesium.Color.WHITE,     // 轮廓颜色，默认白色
  baseAlpha: 0.5,                // 基础透明度
  increaseAlpha: 0.2,            // 递增透明度
  topRadius: 1,                  // 上半径（米）；下半径为 0 → 锥形
  duration: 2,                   // 动画时长（转角 90 度所需）
  shadow: false,                 // 是否加阴影
  shadowScale: 10,               // 阴影比例（默认模型为阴影的 10 倍大小）
});
pp.createModel();     // 创建模型
pp.doAnimation();     // 开启动画
pp.on(Glodon.CIMCube.Global.Defines.ScreenSpaceEventType.LEFT_CLICK, (e) => { /* 点击 */ });
pp.show(); pp.hide(); pp.dispose();
```

## 3. VideoEntity — 视频投射到多边形表面

```js
const ve = new E.Effect.VideoEntity(map, {
  positionList: [
    { lon: 116.40, lat: 39.90, height: 0 },
    { lon: 116.41, lat: 39.90, height: 0 },
    { lon: 116.41, lat: 39.91, height: 0 },
    { lon: 116.40, lat: 39.91, height: 0 },
  ],
  videoSrc: "./assets/demo.mp4",
  autoplayVideo: "autoplay",
  loop: "loop",
  videoType: "video/mp4",
});
ve.addVideoEntity();  // 添加视频实体
ve.play(); ve.stop();
ve.show(); ve.hide(); ve.dispose();
```

## 4. PolylineVolumeFlow — 管道流动/轨迹特效

默认值（未传时）：`cornerType` 圆角、`style.loop=true`、`style.type="trail"`、`style.blendingRatio=0`、`style.repeat=[1,1]`、`style.duration=2000`（文档默认原值，示例中按量级传秒）。

```js
const pvf = new E.Effect.PolylineVolumeFlow(map, {
  path: [[116.40, 39.90, 0], [116.41, 39.91, 0]],  // 至少两个 [lon,lat,height]
  radius: 1,                       // 管道半径
  cornerType: Cesium.CornerType.ROUNDED,  // 转角类型，默认圆角
  style: {
    color: "#1a73e8",               // css 颜色 / Cesium.Color / 数组
    duration: 4,                    // 循环一次时间
    image: "./assets/flow.png",     // 贴图（路径或 base64）
    loop: true,
    type: "trail",                  // "trail" 轨迹 | "flow" 流动，默认 "trail"
    blendingRatio: 0.5,             // 颜色融合比例
    repeat: [1, 4],                 // [x/u, y/v] 平铺倍数
  },
});
pvf.setPathData(newPath);
pvf.setRadius(2);
pvf.setStyle({ color: "#ea4335" });
pvf.setCornerType(Cesium.CornerType.MITERED);
pvf.updateOption({ /* ... */ });
pvf.show(); pvf.hide(); pvf.destroy();   // ⚠️ 释放用 destroy()，无 dispose
```

## 5. Wall — 动态墙

```js
const wall = new E.Effect.Wall(map, {
  pathList: [
    { lon: 116.40, lat: 39.90, height: 0 },
    { lon: 116.41, lat: 39.90, height: 0 },
  ],
  height: 30,                // 墙高
  color: "#1a73e8",
  imageUrl: "./assets/wall.png",
  blendRatio: 1,             // 0-1，颜色融合程度，默认 1
  duration: 1,               // 动画时长 s
  direction: "y",            // x|y|-x|-y
  stripe: 1,                 // 波浪次数
  decayFactor: 0,            // 颜色衰减
  speedFactor: 1,            // 1 匀速；>1 先慢后快；<1 先快后慢
  successive: true,          // 循环完成后是否暂停一段时间
  staticsWall: false,        // 是否静态墙
  flyline: false,            // 是否开启飞线
  flylineWidth: 10,           // 飞线宽 m
  flylineLength: 100,         // 飞线长 m
  flylineDirection: "y",
  flylineSpeed: 1,
  flylineColor: "#fbbc05",
});
wall.setPathData(newPath);
wall.updateOption({ color: "#ea4335" });
wall.show(); wall.hide(); wall.dispose();
```

## 6. Radar — 雷达光束扫描

中心点上方的锥形光束 + 圆环底盘效果。

```js
const radar = new E.Effect.Radar(map, {
  centerPosition: { lon: 116.40, lat: 39.90, height: 0 },  // 中心点（光源|信号源）
  slices: 12,             // 光照面数，默认 12（需 360 能整除）
  lightBeamAlpha: 0.6,    // 光束透明度
  lightCirqueAlpha: 0.4,  // 圆环透明度
  cirqueRadius: 50,       // 圆环半径，默认 50 米
  floorLevel: 0,          // 地表高度
  color: Cesium.Color.CYAN,  // 光颜色
  image: undefined,       // 可传图片 url：按图片内容显示光束（替代光颜色；底盘仍用 color）
});
radar.show(); radar.hide(); radar.dispose();   // 仅 3 个控制方法
```

平面圆盘式雷达扫掠见 `PlaneEffect.Radar`（不同命名空间，同属雷达族）。

## 性能建议

| 类 | 开销 | 建议 |
|---|---|---|
| VideoEntity | 中（视频纹理） | 一个场景 1-2 个，关闭后 dispose |
| PolylineVolumeFlow | 中 | path 段数 <500 |
| Wall | 低~中 | 单面墙；多面墙合并 pathList |

**最佳实践**：SPA 路由切换或演示结束 → 调 `dispose()`（`PolylineVolumeFlow` 用 `destroy()`）而非 `hide()`（否则 GPU 资源泄漏）。
