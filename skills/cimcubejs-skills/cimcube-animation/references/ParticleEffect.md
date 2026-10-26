# Animation.ParticleEffect — 粒子系统（3 类 · 可运行示例）

> 前提：`map` 已创建；`const E = Glodon.CIMCube.Animation`；命名空间 `Glodon.CIMCube.Animation.ParticleEffect.*`。
> 所有粒子类共享方法：`play()` / `stop()` / `show()` / `hide()` / `dispose()` / `updateOption(opt)`；并从基类继承 `stopAutoDispose(autoDispose = false, delay = 1000)`（`stop()` 后可自动延迟回收 GPU 资源，适合烟花等一次性场景）。

## 1. Fire — 火焰

```js
const fire = new E.ParticleEffect.Fire(map, {
  position: { lon: 116.41, lat: 39.905, height: 0 },
  fireRadius: 60,         // 火焰范围（米），勿小于 50
  scale: 1,
  windDirection: 180,     // 0~359 顺时针
  windStrength: 0.3,      // 0~1
});
fire.play();
fire.updateOption({ fireRadius: 80, windStrength: 0.5 });
fire.stop();
fire.dispose();
```

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `position` | Object | - | `{lon, lat, height}` |
| `fireRadius` | Number | - | 火焰范围（米），勿小于 50 |
| `scale` | Number | 1 | 放大缩小 |
| `windDirection` | Number | - | 风向 0~359 顺时针；0 南风，90 西风，180 北风，270 东风 |
| `windStrength` | Number | - | 强度 0~1 |

## 2. Smoke — 浓烟

常与 Fire 叠加（烟在火上 5~10 米）。

```js
const smoke = new E.ParticleEffect.Smoke(map, {
  position: { lon: 116.41, lat: 39.905, height: 6 },
  smokeRadius: 20,        // 浓烟范围（米），勿小于 10
  scale: 0.5,
  windDirection: 180,
  windStrength: 0.3,
  density: 0.5,           // 0~10
  thickness: 3.0,          // 0~10
});
smoke.play();
```

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `position` | Object | - | `{lon, lat, height}` |
| `smokeRadius` | Number | - | 浓烟范围（米），勿小于 10 |
| `scale` | Number | 0.5 | 放大缩小 |
| `windDirection` | Number | - | 风向 0~360 |
| `windStrength` | Number | - | 强度 0~1（强度 1 相当于 5 级风） |
| `density` | Number | 0.5 | 密度 0~10 |
| `thickness` | Number | 3.0 | 浓度 0~10 |

## 3. Water — 喷水

```js
const fountain = new E.ParticleEffect.Water(map, {
  position: { lon: 116.4, lat: 39.9, height: 0 },
  waterRadius: 1,         // 半径，默认 1
  strength: 1.5,          // 强度 0.6~3
  scale: 1,
  color: Cesium.Color.CYAN,
  direction: { heading: 0, roll: 0, pitch: 90 },  // 喷射方向（度）
});
fountain.play();
```

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `position` | Object | - | `{lon, lat, height}` |
| `waterRadius` | Number | 1 | 半径 |
| `strength` | Number | - | 强度 0.6~3 |
| `scale` | Number | 1 | 放大缩小 |
| `color` | Cesium.Color \| String | - | 颜色（接受 `Cesium.Color` 或颜色串） |
| `direction` | Object | - | `{heading, roll, pitch}`（度） |

> ⚠️ `updateOption` 仅支持 `strength` / `scale` / `position` / `direction` 四属性（不含 `waterRadius` / `color`）；APIDOC 对 `strength` 范围标注前后矛盾（构造处 0.6~3、updateOption 处 0~1），按构造处 0.6~3 传参。

## 性能建议

| 类 | 开销 | 建议 |
|---|---|---|
| Fire/Smoke/Water | 中（每粒子 1 vertex） | 同场景 5~10 个；移动端减半 |

**最佳实践**：粒子给 bounds 避免飞离屏幕无限远（泄漏）；演示结束调 `dispose()` 而非 `stop()`。
