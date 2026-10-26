# RealTimePath 实时路径引擎（数据流 + 构造）

命名空间 `Glodon.CIMCube.Simulation`（下称 `S`）。

> 回主路由：`../SKILL.md`（导出真相 / 跨域铁律）；姊妹族（绳管控件）见 `RopeRenderers.md`。
>
> **本族按用途拆三份**：本文件（数据流 + 构造 options）｜`RealTimePath-data.md`（喂数据、帧协议、GeoRoamingGenerator）｜`RealTimePath-events.md`（方法与事件）。

## 2.1 总览与数据流

一条"实时路径" = 一个 featureID 的智能体（车辆/无人机/人员）在一段时间轴上的位置 + 动画 + 业务属性。数据不是直接传点数组，而是走两段式协议：

```
后端/演示生成器            raw = { fps, frames: [ { realTime: 毫秒, commands: [ { id, cmd:'C'|'U'|'D', att:[...], val:[...] } ] } ] }
        │  conversionFramePathData(raw)   （需先配好 modelIdCallBack/animationNameCallBack）
        ▼
simulation = { [featureID]: { modelId, position: PositionFragment,
                              animationName: AnimationFragment,
                              properties: { [属性名]: PropertyFragment },
                              createEvent?, destructEvent? } }
        │  updatePaths({ featuresPath: simulation, fps, replace, autoComplete })
        ▼
RealTimePathEntity × 智能体（ModelInstanceCollection 实例化渲染，真实时钟推进）
```

`cmd` 语义：`C`（create：此刻创建智能体，触发 `modelIdCallBack` 定 modelId）、`U`（update：更新 geo/state/业务属性）、`D`（destruct：到时销毁）。命令属性名 `geo`（位置）、`state`（动画状态名）是内建键；`geo` 的 val 值格式 **`"lon|lat|height"`**（`|` 分隔，高度可省略）；其它属性名进入 `properties`。

## 2.2 构造 options

```js
const rtp = new S.RealTimePath(map, {
  deviceModelList: [                    // 设备模型清单（决定能引用哪些 modelId）
    {
      modelId: "Dumper",                // 必给：模型唯一 id（deviceModelList 的键）
      url: "./models/dumper.gltf",      // 必给：模型资源 url
      scale: 1,                         // 可选 默认 1
      rotation: [roll, pitch, heading], // 可选 默认 [0,0,0]，模型自身姿态（单位度，行进方向仍会自动对向）
      animation: { enable: true, name: "", speedup: 2 }, // 可选：enable 默认全动画循环；name 指定动画；speedup 默认 2
      // 以下外观键可选，均为 ModelInstanceCollection/glTF appearance 参数族：
      phongLighting: false, alpha: 1, color: "#FFFFFF",
      luminanceAtZenith: 0.2, hue: 0, saturation: 1, brightness: 1,
      gamma: 1, contrast: 1, metallicRoughnessFactor: [0, 0],  // 建议白膜质感
      doubleSided: false, silhouetteColor: "#FFFFFF", silhouetteSize: 0,
      shadowMode: 0,   // ShadowMode，默认 DISABLED
    },
  ],
  modelIdCallBack(att, val) {           // ⚠️ 必配（conversionFramePathData 需要，缺省返回 null）
    // att=命令属性名数组, val=对应值数组（含 aoc_class_name 等业务字段）
    return "Dumper";                    // 返回 deviceModelList 里的 modelId
  },
  animationNameCallBack(att, val, modelId, featureID) {
    // state 属性每次变化触发：按业务状态返回要播放的动画名（返回 null 不切）
    return "走";                        // 模型内部自动循环；配合 animation.enable=false 时完全由本回调控制
  },
  canSelected: false,                   // 可选默认 false（预留字段，未影响拾取）
  debugShowPathByPolyline: false,       // 可选默认 false：updatePaths 时自动叠加整条漫游 debug 线
  // 性能（可选）：
  initialCapacity: 50,                  // 默认 50：每种 模型+动画 实例集合的初始容量/扩容步长
  batchUpdateEnabled: false,            // API_DOC 默认 true；运行时构造初值 false——大量智能体建议显式 true
  batchSize: 1000,                      // 每帧批次更新上限
  emptyCollectionDestroyIntervalMs: 5000, // 默认 5000：空闲实例集合回收间隔（0 = 禁用）
});
```

> 构造后仍可整体改配置：运行时还公开 `rtp.setOptions(options)`（键同构造 options，**API_DOC 未列**；会重建 feature 表，适合换设备清单/回调后再 updatePaths）。

## 下一步

- 怎么造数据 / 怎么喂 / 帧协议细则 / 演示生成器 → `RealTimePath-data.md`
- 方法与事件（含事件双通道、`showPathByPolylines`、`dispose`）→ `RealTimePath-events.md`
