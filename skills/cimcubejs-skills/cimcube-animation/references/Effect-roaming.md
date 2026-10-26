# Animation.Effect — 相机与角色漫游（5 类 · 可运行示例）

> 前提：`map` 已创建；`const E = Glodon.CIMCube.Animation`；命名空间 `Glodon.CIMCube.Animation.Effect.*`。公共初始化与组合模式见 `references/patterns.md`。
> 本族 5 类：`CameraRoaming` / `FirstPersonRoaming` / `ThirdPersonRoaming` / `TrackRoaming` / `BillboardRoaming`。
> **控制方法分两类，勿按基类 `Animation`（仅 `dispose`/`play`/`stopAutoDispose`）猜方法**：
> - **play/stop 族（漫游仿真，无 show/hide）**：`CameraRoaming`（另 `pause`/`goOn`）、`FirstPersonRoaming`、`ThirdPersonRoaming`、`TrackRoaming`（另 `replay`/`initRoamingStatus` + 多模型管理）
> - **show/hide 族**：`BillboardRoaming`（另 `setPathData`/`updateOption`）
> 同域其他族：`Effect-entity.md`（锚点·体块·介质 6 类）· `Effect-fx.md`（光效与后处理 3 类）。

## 1. CameraRoaming — 相机漫游（自动导览）

**注意**：第一个参数是 `Glodon.CIMCube.Camera`（即 `map.camera`），不是 `map`。

```js
const roam = new E.Effect.CameraRoaming(map.camera, {
  timeAheadInSecond: 3,    // 前瞻 3 秒，相机运动更平滑
  interpolationAlgorithm: Glodon.CIMCube.Global.Defines.InterpolationAlgorithm.Default,
  interpolationDegree: 2,  // 1-5，非 Default 算法有效
  pitchOffset: -15,        // 俯仰角偏移（度），仅非 Default 算法有效
});  // 构造 opts 仅以上 4 项；duration/loop 通过方法设置
roam.setPath([
  { lon: 116.40, lat: 39.90, height: 800 },
  { lon: 116.41, lat: 39.91, height: 500 },
]);
roam.setDuration(60);      // 改时长
roam.setLoopMode(true);    // 改循环
roam.play();
roam.pause();  // 暂停
roam.goOn();   // 继续（注意：不是 resume）
roam.stop();
roam.dispose();
```

| 方法 | 说明 |
|---|---|
| `setPath(path)` | 设置路径数组 `[{lon,lat,height}]` |
| `play()` | 开始播放（**注意：不是 start**） |
| `pause()` / `goOn()` | 暂停 / 继续（**注意：不是 resume**） |
| `stop()` | 停止 |
| `setDuration(duration)` | 设置持续时间 |
| `setLoopMode(loop)` | 设置循环模式 |
| `dispose()` | 销毁释放资源 |

## 2. FirstPersonRoaming — 第一人称（WASD 地面漫游）

```js
const fp = new E.Effect.FirstPersonRoaming(map, {
  moveStep: 4,         // 前后左右平移步长 m/s
  upDownStep: 1,       // 上下移动步长 m/s
  headingStep: 60,     // 水平视角步长 度/s
  pitchStep: 45,       // 垂直视角步长 度/s
  position: { lon: 116.407, lat: 39.904, height: 1.7 },
  heading: 0, pitch: 0, roll: 0,   // 度
  maxPitch: 60, minPitch: -60,
});
fp.play();   // 开始
fp.stop();   // 停止
fp.dispose();
```

| 参数 | 默认 | 说明 |
|---|---|---|
| `moveStep` | 1 | 前后左右平移步长（米/s） |
| `upDownStep` | 1 | 上下移动步长（米/s） |
| `headingStep` | 3 | 水平视角步长（度/s） |
| `pitchStep` | 3 | 垂直视角步长（度/s） |
| `position` | 相机当前位置 | `{lon,lat,height}` |
| `heading` / `pitch` / `roll` | 相机当前姿态 | 角度（度） |
| `maxPitch` / `minPitch` | 60 / -60 | 俯仰角限制（度） |

> 无 show/hide。播放中读相机状态用只读成员 `viewInfo`（{位置:经纬高, 姿态:航向/俯仰/滚动}）；外部键盘驱动可写方向状态成员 `moveStatus` / `upDownStatus` / `moveLeftRightStatus` / `headingStatus` / `pitchStatus`（取值 -1/0/1）。

## 3. ThirdPersonRoaming — 第三人称（跟随模型）

```js
const tp = new E.Effect.ThirdPersonRoaming(map, {
  position: { lon: 116.407, lat: 39.904, height: 0 },
  modelUrl: "./models/worker.glb",
  scale: 1, frameRate: 1, alwaysAnimation: false,
  eyeHeight: 2,        // 眼睛聚焦高度（米）
  eyeRange: 5,         // 眼睛与模型距离（米）
  eyePitch: 0, eyeMinPitch: -80, eyeMaxPitch: 80, eyeHeading: 0,
  modelHeading: 0, modelRoll: 0, modelPitch: 0,
  moveStep: 10,        // 前后平移步长 m/s
  upDownStep: 1,       // 上下移动步长 m/s
});
tp.play();
tp.stop();
tp.dispose();
```

## 4. TrackRoaming — 模型沿轨迹漫游（多模型 + 骨骼动画）

最复杂的漫游类：支持多模型、骨骼动画、相机跟随、轨迹显示、雷达跟随、贝塞尔曲线。

```js
const track = new E.Effect.TrackRoaming(map, {
  // APIDOC 标准结构：pathList 每个元素 = 一个模型的完整配置
  pathList: [
    {
      url: "./models/worker.glb",     // 模型 url
      size: 1,                        // 模型大小，默认 1
      speed: undefined,               // 漫游速度 m/s；不设则用 data 每点的 duration
      offset: [0, 0, 0],              // 模型偏移 [x,y,z]
      usedAnimation: { enable: true, name: "Run" }, // 骨骼动画（默认启用全部动画）
      modelShow: true,
      modelRotation: [0, 0, 0],       // [roll, pitch, heading]
      data: [                         // 该模型的轨迹点
        { lon: 116.40, lat: 39.90, height: 0, duration: 5 },
        { lon: 116.41, lat: 39.91, height: 0, duration: 5 },
      ],
    },
    // ... 更多模型。模型级字段另有 speedup/phongLighting/alpha/
    //     luminanceAtZenith/hue/saturation/brightness/gamma/contrast/
    //     metallicRoughnessFactor/doubleSided/shadowMode/silhouetteColor/silhouetteSize
  ],
  // 全局与相机跟随
  showPath: false,              // 显示轨迹，默认 false
  eyeHeight: 1.8,               // 跟随后视线高度，默认 1.8 米
  cameraRange: 0.1,             // 相机跟随距离，默认 0.1
  flyToFollowModelDuration: 0,  // 飞行到跟随模型的时间
  hprLocking: true,             // hpr 姿态锁定，默认 true
  bezierCurve: false,           // 是否用贝塞尔曲线
  turnCornerSpeed: 0.2,         // 转角速度 0.2 度/帧
});
track.play();
// 多模型管理
track.flyToModel(index, duration);              // 飞到第 index 个模型
track.setModelDisplay(index, display);          // 显示/隐藏某个模型
track.setRadarFollow(index);                    // 为某模型增加扩散雷达（单参）
track.removeRadarFollow(index);                 // 删除某模型的雷达特效
track.activeAnimation(index, animationOpt);     // 激活某模型骨骼动画（动画参数对象）
track.getAllAnimation(index);                   // 获取某模型所有动画信息
track.removeAnimation(index, animationName);    // 移除某模型指定动画（两参）
track.removeAllAnimation(index);                // 移除某模型所有动画
track.setRoamingData(newPathList);              // 重置漫游数据（结构同 pathList）
track.replay();          // 重放
track.initRoamingStatus();  // 漫游回到 Ready 状态
track.dispose();
```

## 5. BillboardRoaming — Billboard 沿路径运动

```js
// 先用 GraphicLayer 承载 billboard（入参就是 BillboardGraphic 图元）
const G = Glodon.CIMCube.Graphic;
const bbLayer = map.layerManager.addLayer({
  type: Glodon.CIMCube.Global.Defines.LayerType.GRAPHIC,   // 'graphic'
  id: "bb-layer", name: "billboard",
});
const bb = new G.BillboardGraphic({
  // ★ addGraphic 收图元对象（不是 { type, position } 配置）；position 是 [lon,lat,height] 数组
  position: [116.40, 39.90, 0],
  image: "./assets/marker.png",
});
bbLayer.addGraphic(bb);
const br = new E.Effect.BillboardRoaming(map, {
  billboard: bb,
  pathList: [
    { lon: 116.40, lat: 39.90, height: 10 },
    { lon: 116.41, lat: 39.91, height: 10 },
  ],
  duration: 10,            // 动画时长（秒），默认 10
  speed: undefined,        // 速度 m/s，优先级高于 duration
  reverseDirection: false, // 方向翻转（true 沿路径反向移动）
  startPercent: 0,         // 起始位置 0~1
});
br.show();
br.updateOption({ /* 更新参数 */ });
br.setPathData(newPathList);
br.hide();
br.dispose();
```

## 常见组合模式

见 `references/patterns.md` §① 漫游与场景特效实体组合模式（模式 A 领导参观一键导览 / 模式 B 领导实地巡查）。

## 性能建议

| 类 | 开销 | 建议 |
|---|---|---|
| CameraRoaming | 低（只驱动相机） | path >1000 点建议抽样 |
| FirstPersonRoaming | 低 | 移动端降低 moveStep |
| ThirdPersonRoaming | 低~中 | 模型骨骼动画 alwaysAnimation=false 节省开销 |
| TrackRoaming（多模型） | 中~高 | 模型数 <20；speed 适中避免相机抖动 |
| BillboardRoaming | 低 | 单 billboard 路径运动 |

**最佳实践**：SPA 路由切换或演示结束 → 调 `dispose()` 而非 `stop()`（否则 GPU 资源泄漏）。
