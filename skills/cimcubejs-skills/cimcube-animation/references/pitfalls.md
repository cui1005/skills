# Animation 坑点清单（虚构类名黑名单 · 生命周期 · 释放口径）

> 归属 `cimcube-animation`。**查阅型**清单：写前扫一遍、写完对照。
> 各族的构造参数与可运行示例见同目录其它 `references/*.md`；公共初始化与组合模式见 `references/patterns.md`。

## 一、虚构类名黑名单（SDK 中**不存在**，不要 `new`）

以下名字在 API_DOC 与运行时均**不存在**，写码时勿凭直觉杜撰：

| 虚构写法 | SDK 真实替代 |
|---|---|
| `FirstPerson` / `ThirdPerson` | `Effect.FirstPersonRoaming` / `Effect.ThirdPersonRoaming` |
| `TrajectoryRoaming` | `Effect.TrackRoaming`（模型沿轨迹漫游） |
| `FlyLines`（用于道路/管线场景时误写） | `LineEffect.FlyLines`（通用飞线）；道路流光是 `LineEffect.RoadLine` |
| `RoadLines` / `FlowLines` | `LineEffect.RoadLine` / `LineEffect.DynamicLines` |
| `ModelExplosion` | `ModelEffect.ModelExplode`（动词形式） |
| `ModelCutting` / `ModelRoaming` | SDK 无；模型裁剪用 `Utils.ModelCut`，模型漫游用 `Effect.TrackRoaming` |
| `RotateAnimation` / `MoveAlongPath` | SDK 无；旋转/位移用 `ModelEffect.ModelInteroperation`（要素级） |
| 各种 `XxxParticle` 后缀（如 `FireParticle` / `SmokeParticle`） | `ParticleEffect.Fire` / `ParticleEffect.Smoke` / `ParticleEffect.Water` |
| `RadarSweep` / `ScanningWall` | `Effect.Radar` / `Effect.Wall` |
| `DynamicCircle` / `PulsePlane` / `Flare` | `PlaneEffect.Dynamic` / `PlaneEffect.RingScan` / `Effect.AmbientOcclusion` |
| `FlickerPoint` / `WavePoint` | `PointEffect.Twinkle` / `PointEffect.Dynamic` |
| `RaindropPoint` | `WeatherEffect.Rain` |
| `Wind` / `Cloud` / `Thunder` / `SandStorm` | SDK 无天气类；风/云用 `RR.volumetricClouds`（真实感渲染），雷/沙尘需自研 |

> 记忆法：**7 个命名空间**（`Effect` / `LineEffect` / `ModelEffect` / `ParticleEffect` / `PlaneEffect` / `PointEffect` / `WeatherEffect`）+ 类名即其成员名。
> 真实类名与逐类归属见 `using-cimcube-skills/references/class-routing.md`（类目唯一真相源视图，自动生成）。

## 二、生命周期与释放口径

| 族 | 开启 / 停止 | 释放 | 易错点 |
|---|---|---|---|
| `Effect`（play/stop 族） | `play()` / `stop()` | `dispose()` | `PolylineVolumeFlow` 释放方法是 **`destroy()`**（无 `dispose`） |
| `Effect`（show/hide 族） | `show()` / `hide()` | `dispose()` | `Radar` / `Metaptosis` / `AmbientOcclusion` 只有 show/hide/dispose |
| `LineEffect` / `PlaneEffect` | `show()` / `hide()` | `dispose()` | `RopeLines` / `UVTextureEffect` 的 `show` 是**布尔属性**（无 show()/hide() 方法） |
| `ModelEffect` | `doExplode()` / `play()` / `enable()` | `dispose()` | **均无 show/hide**，按任务驱动 |
| `ParticleEffect` | `play()` / `stop()` | `dispose()` | 粒子未 `stop()` 会持续占用 GPU |
| `WeatherEffect` | `show()` / `hide()` | `dispose()` | **无 play/stop** |
| `PointEffect` | 仅 `Dynamic` 有 `play()` / `stop()` | `dispose()` | `Scatter` / `Twinkle` / `InstanceBillboard` 只有 show/hide；后两者**无 updateOption** |

> SPA 路由切换或演示结束一律 `dispose()`（`PolylineVolumeFlow` 用 `destroy()`），不要只 `stop()` / `hide()`，否则 GPU 资源泄漏。

## 三、构造签名例外

- `Effect.CameraRoaming(map.camera [, opts])` —— **唯一接收 Camera 的类**（入参是 `map.camera`，不是 `map`）；其余一律 `new Xxx(map, opt)`。
- 基类 `Animation` 只提供 `dispose()` / `play()` / `stopAutoDispose(autoDispose, delay)`；`show/hide/stop/updateOption` 全部由子类自行实现，**不要按基类猜方法**。

