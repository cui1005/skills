---
name: cimcube-animation
description: "CIMCube 动画特效 Animation：飞线/动态线/道路流光/圆柱线/轨迹线、雷达面与雷达光束、波纹圆环/扩散旋转/扩散墙/动态面/序列贴图面/静态面、模型爆炸分解/单体要素动画/白模贴图、相机漫游/第一人称巡检/第三人称跟随/轨迹漫游/扫描光束/视频投射/管道流动/动态墙、环境光遮蔽、迁徙汇聚、粒子/火焰/烟雾/喷水/下雨/下雪/起雾/散点扩散/实例化看板。相机与角色漫游属本技能；图层加载属 cimcube-layer、图元绘制属 cimcube-graphic、场景大气/天空盒/阴影属 cimcube-scene、数据驱动回放属 cimcube-simulation。"
---

# CIMCube 动画特效（Animation 42 类 · 单技能路由）

> 命名空间 `Glodon.CIMCube.Animation.*`。
> **定位**：本 SKILL.md 只做**索引路由**。完整 options 表、可运行示例与坑点全部外置于同目录 `references/<命名空间>.md`（`Effect` 族 14 类再按漫游 / 实体 / 光效拆为 3 个文件，见 §①）。**写某个类的完整代码前，先读对应文件**，不要凭记忆重写示例。

## 跨域约定（适用于全部 42 类）

1. **构造签名分两类**：仅 `Effect.CameraRoaming(camera [, opts])` 接收 `Glodon.CIMCube.Camera`（即 `map.camera`），其余均为 `new Xxx(map, opt)`。
2. **控制方法无统一约定，按命名空间分族**（勿按基类猜方法）：
   - 基类 `Animation`（`new Animation()`）**仅** `dispose()` / `play()` / `stopAutoDispose(autoDispose = false, delay = 1000)`；`show/hide/stop/updateOption` 由各子类自行实现。
   - `LineEffect`/`PlaneEffect`：多为 `show()/hide()/dispose()` + `updateOption()`（例外见对应 reference）。
   - `ModelEffect` 4 类：**均无 show/hide**，按任务驱动（`doExplode`/`play`/`setLayer`/`enable`）。
   - `ParticleEffect`：`play()/stop()/show()/hide()/dispose()/updateOption()`。
   - `WeatherEffect`：`show()/hide()/dispose()/updateOption()`（**无 play/stop**）。
   - `PointEffect`：仅 `Dynamic` 有 `play()/stop()`；`Scatter`/`Twinkle` 为 show/hide/updateOption；`InstanceBillboard` 仅 show/hide + `setXxx`（**无 updateOption**）。
   - `Effect`：play/stop 族（漫游仿真）与 show/hide 族（显示型特效）并存；`PolylineVolumeFlow` 释放方法为 `destroy()`（**无 dispose**）。
3. 坐标约定：`position`/`pathList`/`positionList` 等点位用 `{lon, lat, height}`（度/度/米）或 `{x:经度, y:纬度, z:米}`（两套并存，按各类示例）；颜色接受 `String`/`Cesium.Color`；角度参数多为角度制（`Effect.CameraRoaming` 的 `pitchOffset` 等）。
4. **释放**：SPA 路由切换或演示结束一律 `dispose()`（`PolylineVolumeFlow` 用 `destroy()`），不要只 `stop()`/`hide()`，否则 GPU 资源泄漏。
5. 公共初始化与组合模式见 `references/patterns.md`；**虚构类名黑名单**（哪些名字 SDK 里根本不存在）见 `references/pitfalls.md`。

## ① 漫游与场景特效实体（Effect 14 类 → 3 个 reference 文件）

> **按族定位后只读对应文件**：`Effect-roaming.md`（漫游 5 类）· `Effect-entity.md`（锚点·体块·介质 6 类）· `Effect-fx.md`（光效·后处理 3 类）。

### ①-1 相机与角色漫游（5 类）→ `references/Effect-roaming.md`

| 类 | 用途 | 控制方法要点 |
|---|---|---|
| `Effect.CameraRoaming` | 相机沿路径自动漫游导览（入参 `map.camera`） | `setPath`/`play`/`pause`/`goOn`/`setDuration`/`setLoopMode`/`stop` |
| `Effect.FirstPersonRoaming` | 第一人称地面巡检（WASD） | `play()`/`stop()`；状态成员 `viewInfo`/`moveStatus` |
| `Effect.ThirdPersonRoaming` | 第三人称跟随模型 | `play()`/`stop()` |
| `Effect.TrackRoaming` | 模型沿轨迹漫游（多模型+骨骼动画） | `play()` + `flyToModel`/`setRoamingData`/`replay`/`initRoamingStatus` |
| `Effect.BillboardRoaming` | Billboard 沿路径运动 | `show()`/`hide()` + `setPathData`/`updateOption` |

### ①-2 锚点·体块·介质特效（6 类）→ `references/Effect-entity.md`

| 类 | 用途 | 控制方法要点 |
|---|---|---|
| `Effect.BillboardEntity` | Cesium 锚点（⚠️ 官方标注即将废弃） | `show()`/`hide()` + 事件派发 `on/off/fire` |
| `Effect.PrismPoint` | 立体锚点·菱形锥模型 | `createModel()`/`doAnimation()` + 事件派发 |
| `Effect.VideoEntity` | 视频投射到多边形表面 | `addVideoEntity()` + `play`/`stop`/`show`/`hide` |
| `Effect.PolylineVolumeFlow` | 管道流动/轨迹特效 | `show`/`hide` + `setPathData`/`setStyle`；**释放用 `destroy()`** |
| `Effect.Wall` | 动态墙/飞线墙 | `show()`/`hide()` + `setPathData`/`updateOption` |
| `Effect.Radar` | 雷达光束扫描 | 仅 `show()`/`hide()`/`dispose()` |

### ①-3 光效与后处理（3 类）→ `references/Effect-fx.md`

| 类 | 用途 | 控制方法要点 |
|---|---|---|
| `Effect.CameraShooting` | 摄像机扫描光束（光源→目标摆动） | `show()` + `playPathAnimation` |
| `Effect.AmbientOcclusion` | 环境光遮蔽（AO，全局后处理） | `show()`/`hide()` + `updateOption` |
| `Effect.Metaptosis` | 迁徙特效（中心模型+迁徙线汇聚） | 仅 `show()`/`hide()`/`dispose()` |

## ② 视觉图形（LineEffect 5 + PlaneEffect 8 + ModelEffect 3 = 16 类）

| 类 | 用途 | 控制方法要点 | 写码前必读 |
|---|---|---|---|
| `LineEffect.FlyLines` | 飞线（竖直） | `show()`/`hide()`/`updateOption()` | `references/LineEffect.md` |
| `LineEffect.DynamicLines` | 动态线（6 种 lineType） | `show()`/`hide()` + `setPathData`/`updateOption` | `references/LineEffect.md` |
| `LineEffect.RoadLine` | 道路流光 | `show()`/`hide()` + `updateOption`；成员 `percent`/`pause` | `references/LineEffect.md` |
| `LineEffect.RopeLines` | 圆柱线 | ⚠️ `show` 是**布尔属性**（无 show()/hide()）；`updatePosition` 须成对顶点 | `references/LineEffect.md` |
| `LineEffect.TrailLines` | 轨迹线（带箭头） | 仅 `show()`/`hide()`/`dispose()`（无 updateOption） | `references/LineEffect.md` |
| `PlaneEffect.Radar` | 雷达扫描（圆形） | `show()`/`hide()` + `updateOption` | `references/PlaneEffect.md` |
| `PlaneEffect.RingScan` | 波纹圆面（多圈） | `show()`/`hide()` + `updateOption` | `references/PlaneEffect.md` |
| `PlaneEffect.DiffuseWall` | 扩散墙（中心向外） | `show()`/`hide()` | `references/PlaneEffect.md` |
| `PlaneEffect.Dynamic` | 动态面（贴图流动） | `show()`/`hide()` + `updateOption` | `references/PlaneEffect.md` |
| `PlaneEffect.Moire` | 扩散旋转特效 | 仅 `show()`/`hide()`/`dispose()`；运行时可改 `zIndex` 成员 | `references/PlaneEffect.md` |
| `PlaneEffect.Sequence` | 序列贴图面 | `show()` + `setPathList`/`updateOption` | `references/PlaneEffect.md` |
| `PlaneEffect.Statics` | 静态面 | `show()` + `setPathList`/`updateOption` | `references/PlaneEffect.md` |
| `PlaneEffect.StaticsRingScan` | 圆环特效（无扫描动画） | `show()` + `updateOption` | `references/PlaneEffect.md` |
| `ModelEffect.ModelExplode` | 模型炸开（BIM 楼层分解） | ⚠️ 无 show/hide：`doExplode()`/`cancelExplode()`/`setOpts()` | `references/ModelEffect.md` |
| `ModelEffect.ModelInteroperation` | 模型单体要素动画 | ⚠️ 无 show/hide：`play()`/`restore()`/`removeFeatures(ids)`/`setOpts()` | `references/ModelEffect.md` |
| `ModelEffect.UVTextureEffect` | 白模贴图 | ⚠️ `show` 是布尔属性；`setLayer(layer)`/`setOpts()` | `references/ModelEffect.md` |
| `ModelEffect.ModelComplexColor` | 模型单体要素**按高程分段着色**（3.57.0 新增） | ⚠️ 无 show/hide：`enable()`/`disable()`/`setOpts()`/`removeFeatures(ids)`/`dispose()` | `references/ModelEffect.md` |

## ③ 粒子 · 天气 · 点动态（ParticleEffect 3 + WeatherEffect 3 + PointEffect 4 = 10 类）

| 类 | 用途 | 控制方法要点 | 写码前必读 |
|---|---|---|---|
| `ParticleEffect.Fire` | 火焰粒子 | `play()`/`stop()`/`show()`/`hide()`/`updateOption()` | `references/ParticleEffect.md` |
| `ParticleEffect.Smoke` | 浓烟粒子 | 同上（常与 Fire 叠加） | `references/ParticleEffect.md` |
| `ParticleEffect.Water` | 喷水粒子 | 同上；`updateOption` 仅 strength/scale/position/direction | `references/ParticleEffect.md` |
| `WeatherEffect.Rain` | 雨 | `show()`/`hide()`/`updateOption()`（无 play/stop） | `references/WeatherEffect.md` |
| `WeatherEffect.Snow` | 雪 | 同上（`opt` 可省略） | `references/WeatherEffect.md` |
| `WeatherEffect.Fog` | 雾（天气级特效雾） | 同上（无 windDirection） | `references/WeatherEffect.md` |
| `PointEffect.Dynamic` | 动态点轨迹 | `play()`/`stop()` + `setPathData`/`updateOption` | `references/PointEffect.md` |
| `PointEffect.Twinkle` | 闪烁/呼吸点 | `show()`/`hide()` + `updateOption`（**勿 play**） | `references/PointEffect.md` |
| `PointEffect.Scatter` | 散点扩散 | `show()`/`hide()` + `updateOption`（**勿 play**） | `references/PointEffect.md` |
| `PointEffect.InstanceBillboard` | 实例化看板（大量 POI） | 仅 `show()`/`hide()` + 6 个 `setXxx`（**勿 play**） | `references/PointEffect.md` |

## See Also

- **cimcube-camera** — `map.camera` 是 `Effect.CameraRoaming` 的入参；`flyTo`/`getCaptureImg` 与漫游/特效展示配合
- **cimcube-realistic-rendering** — Bloom 辉光增强飞线/雷达；`RR.exponentialHeightFog` 高度雾与 `WeatherEffect.Fog` 区别
- **cimcube-scene** — 场景环境（阴影/天空盒/后期水）与天气粒子的氛围叠加
- **cimcube-graphic** — `BillboardGraphic`（少量 POI）是 `BillboardRoaming` 入参，与 `InstanceBillboard`（大量 POI）对比
- **cimcube-layer** — `Tiles3DLayer` 是 `ModelExplode`/`ModelInteroperation` 的必填入参
- **cimcube-simulation** — `RealTimePath` 数据驱动时间轴回放（区别于视觉展示的漫游）
- **cimcube-global** — `Global.Defines.InterpolationAlgorithm` 等枚举

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
