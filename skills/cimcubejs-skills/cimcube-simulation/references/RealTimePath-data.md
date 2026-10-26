# RealTimePath 数据：喂给引擎 · 帧协议 · 演示生成器

> 回主路由：`../SKILL.md`；构造 options 与数据流见 `RealTimePath.md`；方法与事件见 `RealTimePath-events.md`。

## 2.3 数据生成与喂给引擎

```js
// 方式 A：用运行时隐藏的演示生成器造帧（见 §2.6）
const gen = new S.GeoRoamingGenerator({ bounds: [minLon, minLat, maxLon, maxLat], unitGrid: 25 });
gen.generateGridPoints();
gen.generateRandomPathIndex(/* 条数= */ 6);
const raw = gen.generateRoamingVehicles(/* fps= */ 10, /* 每条路径车辆= */ 6, /* 发车间隔ms= */ 2000,
                                        /* 到点停留ms= */ 4000, /* 速度下限 */ 10, /* 速度上限 */ 20);
// raw = { fps: 10, frames: [...] }   ← 就是 RealTimePath.md §2.1 的 raw 协议
// 方式 B：自建后端（或把 raw 交给后端生成），协议见 §2.5
// const raw = await fetchRealtimeFrames(featureId, from, to);  // {fps, frames}
const converted = rtp.conversionFramePathData(raw);   // { fps, startRealTime, simulation }；modelIdCallBack 为空时返回 null
if (converted) {
  rtp.updatePaths({
    featuresPath: converted.simulation,   // 由 conversion 产出（内含 PositionFragment 等内部对象）
    fps: converted.fps,                   // 模拟帧率，默认 10（帧间隔 1000/fps ms）
    startRealTime: converted.startRealTime, // 默认 0（原始首帧绝对时间，引擎按 updatePaths 时刻即时起播）
    replace: true,                        // 默认 false：true=清空重放；false=追加新智能体
    autoComplete: true,                   // 默认 true：非 replace 时旧数据自动跑到终点/销毁
  });
}
```

引擎从 `updatePaths()` 起按 **`Date.now()` 真实时钟**在 `map.scene.preUpdate` 每帧 tick：时间轴 1ms = 现实 1ms，模型沿途自动朝向行进方向并平滑插值。**没有 play/pause/seek/倍速接口**——要暂停/回放只能 `dispose()`/重进或把 `replace:true` 重新喂同批数据；同一 `RealTimePath` 上连续 `updatePaths` 会替换上一份监听。

## 2.5 帧协议细则（自建后端务必按此）

```js
{
  fps: 10,                                   // 数据帧率
  frames: [                                  // 按 realTime 升序
    {
      realTime: 1720000000000,               // 绝对毫秒（转换时以首帧为 0 基线）
      commands: [
        { id: "truck_001", cmd: "C",         // C=创建 | U=更新 | D=销毁
          att: ["aoc_class_name", "geo", "state", "speed", "readyTime"],
          val: ["Dumper", "116.40|39.90|0", "READY", "30", "1720000000000"] },
        { id: "truck_001", cmd: "U",
          att: ["state", "geo"], val: ["GO", "116.41|39.91|0"] },
        // ... 每个后续帧一条 {id, cmd:'U', att:['geo'], val:['lon|lat|height']} 位置推进
        { id: "truck_001", cmd: "D", att: [], val: [] },   // 结束
      ],
    },
    // ...
  ],
}
```

- `C` 帧触发 `modelIdCallBack(att, val)`（att/val 为同一命令内的属性名/值数组，按位置对应）；请在里面用业务字段（上例 `aoc_class_name`）查回 `deviceModelList` 的 modelId。无 modelId 的 feature 会跳过。
- `state` 属性变化时触发 `animationNameCallBack(att, val, modelId, featureID)`，返回要切到的动画名（可与模型各动画轴名对应）。
- `geo` 值用 `|` 分隔 `lon|lat|height`（高度可省略，缺省 0）；属性在 `C` 帧也可先给一个点。
- 其余属性名（`speed` 等）进入该 feature 的 `properties`，值变即派发 `propertiesChanged`。
- 创建后若中途不再喂 `geo`，智能体会停在最后一点并继续播放当前动画（适合做"卸料/装载停靠"等状态机）。

## 2.6 GeoRoamingGenerator 演示帧数据生成器（运行时导出 · 无 API_DOC）

`RealTimePath` 的**演示/压测数据生成器**（矿卡车队式：READY → GO → UNLOADING → RETURN → LOADING → 销毁）。因无文档页，以下签名按当前实现记录：

```js
const gen = new S.GeoRoamingGenerator({ bounds: [minLon, minLat, maxLon, maxLat], unitGrid: 25 });
// bounds: [lon0,lat0,lon1,lat1] 必填（4 个数）；unitGrid 默认 25：网格边长（米，经纬度按约 1°≈ 单位换算）
gen.generateGridPoints();          // 建网格；返回 { lonlat: Float32Array, xy: Float32Array }
gen.generateRandomPathIndex(count=1, minStepCells=20);  // 返回 count 条网格格点索引数组（起点随机）
const raw = gen.generateRoamingVehicles(
  fps = 5,          // 输出帧率
  carsPerPath = 8,  // 每条路径车辆数
  readyGapMs = 2000,// 同路径相邻车辆发车间隔
  stayMs = 4000,    // 到达(UNLOADING)后停留再返回的毫秒
  speedMin = 10,    // 速度下限（数值越大越快，约 m/s）
  speedMax = 20,    // 速度上限
  baseTime = Date.now(),
);                  // → { fps, frames:[{realTime, commands:[{id,cmd,att,val}]}] } 即 RealTimePath.md §2.1 raw 协议
gen.framesData;     // 也存到该字段（generateRoamingVehicles 内部赋值）
```

配套使用见 §2.3 方式 A；其中每条 feature 的额外属性含 `aoc_class_name / speed / readyTime / goStartTime / unloadingStartTime / returnStartTime / loadingStartTime / finishTime`，`aoc_class_name` 供 `modelIdCallBack` 区分车型。
