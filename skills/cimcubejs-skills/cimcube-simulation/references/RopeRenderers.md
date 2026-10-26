# 绳管渲染控件：CatenaryCable 垂缆 / InsulationOverheadNet 绝缘跨越防护网

命名空间 `Glodon.CIMCube.Simulation`（下称 `S`）。

> 回主路由：`../SKILL.md`（导出真相 / 跨域铁律）；姊妹族（数据驱动引擎）见 `RealTimePath.md`。

## 1 通用约定（两者共有）

- **纯渲染控件**：只把几何画成绳管，内部用 `Animation.LineEffect.RopeLines` 分段圆柱渲染（`cimcube.js` 内与动画线束共用）。**没有**净空/垂弧计算、碰撞、受力或"网格编辑"接口。
- **`sag` 一律是数值**（中点相对弦的下垂米数，缺省 0 = 绷直），不是 `{type:"cos",...}` 之类的对象。
- **端点格式**：`{lon,lat,height}` / `[lon,lat,height]` / Cesium `Cartesian3` 均可。
- **显隐是 Boolean 成员**（`showRope` / `showNet`），不是方法；构造 options 里另有 `show`（默认 true）。
- **样式**：要纯色时记得把 `texture` 置空（`""`/`null`）以跳过内置 SVG 缆绳纹理。
- **生命周期**：改点/改样式用 `updateOption(部分键)` 即刷；销毁用 `dispose()`（必调，SPA 切页释放 Primitive）。

## 2 CatenaryCable 垂缆

两点（P0→P1）之间的一段**下垂绳管**，几何是"高度上减去 `sag*4·u·(1-u)`"的标准抛物线垂弧（u 为沿线归一化参数）。

```js
const cable = new S.CatenaryCable(map, {
  P0: { lon: 116.40, lat: 39.90, height: 20 },   // 起点（也可给 [lon,lat,height] 或 Cartesian3）
  P1: { lon: 116.41, lat: 39.92, height: 24 },   // 终点
  radius: 0.03,              // 绳管半径（建议显式给出）
  color: "#3a7afe",          // 颜色
  // texture: ""|null,       // 传 null/"" = 纯色；不传 = 内置缆绳 SVG 纹理（默认）
  textureRepeat: { x: 1, y: 1 },  // x=周向U / y=轴向V
  roughness: 0.82,           // 默认 0.82
  metallic: 0.5,             // 默认 0.5
  sag: 2,                    // 弧垂（中点相对弦的下垂米数，数值型；缺省 0 = 绷直）
  segments: 50,              // 默认 50：采样段数
  slices: 16,                // 默认 16：圆周细分
  shadowMode: 0,             // ShadowMode，默认 DISABLED
  show: true,                // 默认 true
});
cable.showRope = false;  cable.showRope = true;   // Boolean 成员：显隐整条绳
cable.updateOption({ P0, P1, sag: 5, segments: 80, radius: 0.05, color: "#f00",
                     texture, textureRepeat, roughness, metallic, shadowMode }); // 改点/弧垂/样式即刷
cable.dispose();                                     // 销毁绳管（组件本身可丢弃）
```

> `textureRepeat.x` 是绕周（U）、`y` 是沿线（V）。连续线缆可拆成多段 `CatenaryCable`，或用 `updateOption` 改端点实现拖动。

## 3 InsulationOverheadNet 绝缘跨越防护网

电力施工跨越场景的网具控件：两根**承载索**（`startP0→endP0`、`startP1→endP1`）按悬链下坠，沿跨距方向在 `u ∈ [netCableEdgeOffsetStart, netCableEdgeOffsetEnd]` 之间布设若干**绝缘撑杆**与**封网绳**，全为绳管渲染。四角点与分组材质见下：

```js
const net = new S.InsulationOverheadNet(map, {
  // 四角（承载索端点；可 {lon,lat,height} / [lon,lat,height] / Cartesian3）
  startP0: { lon: 116.40, lat: 39.90, height: 30 },
  startP1: { lon: 116.42, lat: 39.90, height: 30 },   // 与 startP0 同侧
  endP0:   { lon: 116.40, lat: 39.92, height: 34 },   // startP0 的对端承载索终点
  endP1:   { lon: 116.42, lat: 39.92, height: 34 },
  netCableEdgeOffsetStart: 0.4,   // 默认 0.4：封网沿跨距(u)起始位置
  netCableEdgeOffsetEnd:   0.6,   // 默认 0.6：封网终止位置
  strutCount: 4,                  // 默认 4（运行时 clamp ≥2）：绝缘撑杆数量（首尾杆恰在 offset 两端）
  netSpanCount: 9,                // 默认 9：两撑杆之间沿横向(v)的内部网绳条数（不含两侧承载索）
  netCrossCount: 6,               // 默认 6：相邻撑杆之间沿跨距(u)的横跨网绳条数
  stressCable: {                  // 承载索（两根，悬链下坠）
    radius: 0.028, color: "#c8dce8", textureRepeat: { x: 10, y: 25 },
    roughness: 0.82, metallic: 0.1, sag: 0, segments: 40, slices: 16,
  },
  strut: {                        // 绝缘撑杆（黄杆，当前渲染为无弧直杆）
    radius: 0.056, color: "#f5d800", roughness: 0.82, metallic: 0.45, slices: 16,
  },
  netCable: {                     // 绝缘封网绳（网眼在 stress/netCrossCount 之间交错成网）
    radius: 0.01, color: "#c8dce8", roughness: 0.85, metallic: 0.35, sag: 0,
    segments: 10, slices: 8,
  },
  shadowMode: 0,                  // 默认 DISABLED
  show: true,                     // 默认 true
});
net.showNet = false;  net.showNet = true;
net.updateOption({ startP0, endP1, netCableEdgeOffsetStart: 0.2, netCableEdgeOffsetEnd: 0.8,
                   strutCount: 5, netSpanCount: 11, netCrossCount: 8,
                   stressCable: { radius: 0.03, sag: 1 }, netCable: { color: "#0af" },
                   strut: { radius: 0.06 }, shadowMode });
net.dispose();
```

> 几何简释：`startP0`/`startP1` 与 `endP0`/`endP1` 确定跨距方向与两侧位置；撑杆按 u 均匀分布、端点落在两根悬链承载索折线上；相邻两杆之间生成 `netSpanCount` 条横向(v)网绳 + `netCrossCount` 条纵向(u)网绳，全部为有弧（可给各分组 `sag`）绳管，视觉上织成跨越网。**没有**"mesh/网格编辑/受力"等几何对象接口。
> 样式均内置 SVG 纹理（承载索/撑杆/网绳各有默认），颜色给纯色时记得可把 `texture` 置空使用。
