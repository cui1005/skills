# 组合模式、性能建议与防误导对照

> 回主路由：`../SKILL.md`。逐类细节见 `RealTimePath.md`（含 GeoRoamingGenerator）/ `RopeRenderers.md`（垂缆 + 绝缘网）。

## 公共初始化

```js
const S = Glodon.CIMCube.Simulation;
// Simulation = { RealTimePath, CatenaryCable, InsulationOverheadNet, GeoRoamingGenerator }
const map = new Glodon.CIMCube({ id: "cimcubeContainer", gisResourceUrl: "./dist/" });
```

## 模式 A：矿卡车队（隐藏生成器快速出效果）

```js
function demoRoaming(map) {
  const rtp = new S.RealTimePath(map, {
    deviceModelList: [{ modelId: "Dumper", url: "./models/dumper.gltf", scale: 1 }],
    modelIdCallBack: (att, val) => "Dumper",
    animationNameCallBack: (att, val, modelId, fid) => {
      const s = String(val[att.indexOf("state")] || "");   // READY/GO/UNLOADING/...
      return s === "GO" ? "走" : null;                      // 按需映射动画名
    },
  });
  const g = new S.GeoRoamingGenerator({ bounds: [116.40, 39.90, 116.41, 39.91], unitGrid: 20 });
  g.generateGridPoints(); g.generateRandomPathIndex(3);
  const raw = g.generateRoamingVehicles(10, 5, 1500, 3000, 8, 16);
  const c = rtp.conversionFramePathData(raw);
  c && rtp.updatePaths({ featuresPath: c.simulation, fps: c.fps, replace: true });
  return rtp;   // 监听 rtp.on("featureRendered", ...) 收模型；不再用 rtp.dispose()
}
```

## 模式 B：业务后端实时数据

```js
// 1) 构造时只配 deviceModelList + 两个回调（modelIdCallBack 从业务属性挑模型）
// 2) 周期性拉取原始帧（协议见 RealTimePath.md §2.5）→ conversionFramePathData → updatePaths({..., replace:false})
//    replace:false + autoComplete:true = 旧智能体自动跑完、新数据追加（适合持续对接 IoT 实时位置）
// 3) 订阅 rtp.on("propertiesChanged") 做预警/统计，订阅 featureDisposed 做结束清理
```

## 模式 C：垂缆/网具"仅渲染"接入

```js
function addCables(map) {
  const list = [
    new S.CatenaryCable(map, { P0: {lon:116.40,lat:39.90,height:20}, P1: {lon:116.41,lat:39.92,height:24}, radius: 0.03, color: "#3a7afe", sag: 3 }),
  ];
  const net = new S.InsulationOverheadNet(map, { /* 四角与分组见 RopeRenderers.md §3 */ });
  return { list, net,
    teardown() { list.forEach((c) => c.dispose()); net.dispose(); } };
}
```

## 性能建议

| 项 | 说明 |
|---|---|
| 实例化渲染 | RealTimePath 用 ModelInstanceCollection 合批（同 modelId+动画 一个 Primitive 集合，容量按 initialCapacity 增长），几百智能体可接受 |
| 大批量 | 智能体数量大时把 `batchUpdateEnabled:true` + `batchSize:1000`；空闲集合回收靠 `emptyCollectionDestroyIntervalMs`（默认 5000ms） |
| debug 线 | `debugShowPathByPolyline/showPathByPolylines` 只在调试开，正式跑关掉 |
| RopeLines 绳管 | Catenary/Net 是逐段圆柱绳管：slices/segments 越大越圆滑也越贵；大批长网适当降低 |
| 生命周期 | `dispose()` 必调（清 preUpdate 监听 + 实例 Primitive），SPA 离开页面时把 RealTimePath/绳管都 dispose |

## 文档与运行时不一致（以运行时为准）

- `batchUpdateEnabled`：API_DOC 默认 `true`，运行时构造初值 `false` → `RealTimePath.md` §2.2。
- `showPathByPolylines` 的 `duration`：文档默认 5 秒，运行时默认 1 秒 → `RealTimePath.md` §2.4。
