# Animation.WeatherEffect — 天气系统（3 类 · 可运行示例）

> 前提：`map` 已创建；`const E = Glodon.CIMCube.Animation`；命名空间 `Glodon.CIMCube.Animation.WeatherEffect.*`。
> 所有天气类共享方法：`show()` / `hide()` / `dispose()` / `updateOption(opt)`（**无 play/stop**）。构造函数形如 `new Xxx(map, opt)`（仅 `Snow` 的 `opt` 可省略）。

## 1. Rain — 雨

```js
const rain = new E.WeatherEffect.Rain(map, {
  darkness: 0.5,    // 阴天暗度 0.1~0.9
  density: "normal", // 雨大小：none|small|normal|great
});
rain.show();
rain.updateOption({ density: "great" });  // 加大
rain.hide();
rain.dispose();
```

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `darkness` | Number | 0.5 | 阴天/天空暗度 0.1~0.9 |
| `density` | String | - | 雨大小：`none`（雨停）/ `small`（小雨）/ `normal`（中雨）/ `great`（大雨） |

## 2. Snow — 雪

```js
const snow = new E.WeatherEffect.Snow(map, {
  darkness: 0.5,
  density: "small",  // 雪大小：none|small|normal|great
  thickness: 0.3,    // 雪的厚度 0~1
});
snow.show();
```

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `darkness` | Number | 0.5 | 阴天/天空暗度 0.1~0.9 |
| `density` | String | small | 雪密度：`none`（雪停）/ `small`（小雪）/ `normal`（中雪）/ `great`（大雪）；未传时默认小雪 |
| `thickness` | Number | - | 雪的厚度 0~1 |

## 3. Fog — 雾

与 `map.scene.realisticRendering.exponentialHeightFog` 区别：scene 的是**渲染级指数高度雾**；WeatherEffect.Fog 是**天气级特效雾**（仅 `darkness`/`lightAttenuation`/`visualDistance` 三参数，**无 `windDirection`**），两套可并存不冲突。

```js
const fog = new E.WeatherEffect.Fog(map, {
  darkness: 0.5,            // 0.0~1.0
  lightAttenuation: 3.5,    // 光线衰减指数 (0,+∞)，越小雾化越慢
  visualDistance: 5000,     // 最远可视范围（米）
});
fog.show();
```

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `darkness` | Number | 0.5 | 阴天/天空暗度 0.0~1.0 |
| `lightAttenuation` | Number | 3.5 | 光线衰减指数 (0,+∞) |
| `visualDistance` | Number | 5000 | 最远可视范围（米） |

## 性能建议

| 类 | 开销 | 建议 |
|---|---|---|
| Rain/Snow | 高~很高（每帧百万粒子） | 移动端关闭；duration/density 适中 |
| Fog | 中 | 建议勿与 Scene 高度雾叠加（会过浓）；确需并存时给 Fog 低 darkness |

**最佳实践**：天气类全场景开销大，density 不要长期保持 `great`，演示完即 `hide()`；SPA 路由切换或演示结束 → 调 `dispose()`。
