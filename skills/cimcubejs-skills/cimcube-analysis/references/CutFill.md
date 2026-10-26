# CutFill — 填挖方分析（土方核心，可运行示例）

> 前提：`map` 已创建、已加载地形高程数据；`const A = Glodon.CIMCube.Analysis`；事件常量 `FINISH = Glodon.CIMCube.Global.Defines.MapActionType.FINISH`。
> **结果异步**：面积/体积 getter 必须在 `FINISH` 回调中取。

主要 options（类页）：`boundary`（分析区域，多点坐标数组）、`level`（基准面标高 m）、`levelTerrain`（地形数据级别，默认 14）、`showVisualization`、`onlySuperficialArea`（只计算贴地面积/高程）、`showOutline`（网格线，默认 true）、配色（`cutColor`/`fillColor`/`flatColor`、`*OutlineColor`、`upperWallColor`/`lowerWallColor`、`showFillFace`）。

```js
const cutFill = new A.CutFill(map, {
  boundary: [/* 多点坐标数组 */],
  level: 48,                       // 基准面标高 48m
  showVisualization: true,
  showOutline: true,
});

// 完成回调（官方事件常量写法）：
cutFill.on(FINISH, (e) => {
  console.log("填挖方分析完成", e);
  const fill = cutFill.getFillVolume();     // 填方
  const cut  = cutFill.getCutVolume();      // 挖方
  const area = cutFill.getTotalArea();      // 区域面积
  console.log(`填 ${fill} m³ · 挖 ${cut} m³`);
});

// 改参数重算：setBoundary / setLevel / setOpts 后 update()（返回 Promise）
cutFill.setLevel(50);
cutFill.update().then(() => console.log("已按新标高重算"));

cutFill.clearPrimitive();   // 清除可视化图元
cutFill.dispose();
```

> 设计面差异：`level` 为**单一基准标高**；如需分区/外部设计面（如按 TIN/DEM 做设计面），按该类 API_DOC 支持的 options 扩展。
