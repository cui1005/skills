---
name: cimcube-simulation
description: "CIMCube 仿真域 Simulation：①实时路径/智能体漫游引擎 — 把带时间戳的帧命令数据渲染为实例化模型，按真实时钟实时推进（无播放/暂停/回放接口）；②两点间绳管垂缆渲染（中点点下垂，标准悬链线外观、无净空解析）；③绝缘跨越防护网渲染（承载索间布绝缘撑杆+封网绳，仅渲染）；④漫游演示帧数据生成器。当用户要实时路径/车队漫游、两点垂缆/索道、电力跨越防护绝缘网、仿真帧数据驱动模型时激活。"
---

# CIMCube Simulation 仿真域（4 导出类 · 单技能路由）

> **基线**：命名空间 `Glodon.CIMCube.Simulation`（下称 `S`）。
> **校准声明**（3 个 API_DOC 类页 `RealTimePath.html` / `CatenaryCable.html` / `InsulationOverheadNet.html` + `cimcube.js` 逐段核验）：`Simulation` 导出对象为 **4 类** = API_DOC 3 类 + `GeoRoamingGenerator`（**无 API_DOC 页**，演示数据生成器）。
> **定位**：本 SKILL.md 只做**索引路由 + 跨域铁律 + 导出现实**。完整 options 表、可运行示例与坑点全部外置于同目录 `references/`（按**族**分组：`RealTimePath.md` 数据流+构造 / `RealTimePath-data.md` 数据·帧协议·演示生成器 / `RealTimePath-events.md` 方法与事件 / `RopeRenderers.md`（绳管 2 类）/ `patterns.md`（组合模式·性能·防误导））。**写某一类的完整代码前，先读对应文件**，不要凭记忆重写示例。

```js
const S = Glodon.CIMCube.Simulation;
// Simulation = { RealTimePath, CatenaryCable, InsulationOverheadNet, GeoRoamingGenerator }
const map = new Glodon.CIMCube({ id: "cimcubeContainer", gisResourceUrl: "./dist/" });
```

## 跨域铁律（4 条）

1. **两类性质完全不同**：`RealTimePath` 是**动画引擎**（帧命令数据 → 实例化模型实时漫游，真实时钟推进，**无 play/pause/seek**）；`CatenaryCable` / `InsulationOverheadNet` 是**纯渲染控件**（只把几何画成绳管，内部用 `Animation.LineEffect.RopeLines`，**没有**净空/垂弧计算、碰撞或物理接口）。
2. **`modelIdCallBack` 必配**：`conversionFramePathData()` 依赖它把 `C` 帧的业务属性映射到 `deviceModelList` 的 modelId；缺省时返回 `null`，转换直接失败。
3. **`geo` 值格式是 `"lon|lat|height"` 字符串**（`|` 分隔，高度可省略），不是数组、不是 `{x,y,z}`。
4. **`dispose()` 必调**：`RealTimePath` 会挂 `map.scene.preUpdate` 监听 + 实例 Primitive；绳管控件同样持有 Primitive。SPA 切页必须释放。

## ① 主引擎：RealTimePath（数据驱动）

| 类 | 用途 | 关键契约 | 写码前必读 |
|---|---|---|---|
| `RealTimePath` | 帧命令数据驱动的实例化模型实时漫游引擎（extends EventDispatcher） | 构造必给 `deviceModelList` + `modelIdCallBack`；数据两段式 `conversionFramePathData(raw)` → `updatePaths({featuresPath, fps, replace, autoComplete})`；无播放控制接口 | `references/RealTimePath.md`（构造·数据流）→ `references/RealTimePath-data.md`（喂数据·帧协议）→ `references/RealTimePath-events.md`（方法与事件） |
| `GeoRoamingGenerator` | 在 bounds 网格内生成 RealTimePath 演示帧数据（无 API_DOC 页） | `new S.GeoRoamingGenerator({bounds:[lon0,lat0,lon1,lat1], unitGrid})` → `generateGridPoints()` / `generateRandomPathIndex()` / `generateRoamingVehicles()` | `references/RealTimePath-data.md` §2.6 |

## ② 纯渲染控件：绳管（两者共用 `references/RopeRenderers.md` §1 通用约定）

| 类 | 用途 | 关键契约 | 写码前必读 |
|---|---|---|---|
| `CatenaryCable` | 两点间垂缆绳管渲染（P0→P1 + sag） | `P0`/`P1` 必给（`{lon,lat,height}` / `[lon,lat,height]` / Cartesian3）；`sag` 为数值；显隐用 **Boolean 成员 `showRope`**（非方法）；`updateOption()` 改点/样式即刷 | `references/RopeRenderers.md` §1+§2 |
| `InsulationOverheadNet` | 承载索/绝缘撑杆/封网绳组渲染 | 四角 `startP0`/`startP1`/`endP0`/`endP1` 必给；三组材质 `stressCable`/`strut`/`netCable`；显隐用 **Boolean 成员 `showNet`**；`updateOption()` 改分组样式 | `references/RopeRenderers.md` §1+§3 |

## ③ 跨域文件

| 内容 | 写码前必读 |
|---|---|
| 组合模式（矿卡车队 / 业务后端实时数据 / 绳管仅渲染接入）+ 性能建议 + 文档与运行时不一致项 | `references/patterns.md` |

## 内部对象（⚠️ 不可 `new`）

| 名称 | 状态 | 真实可达途径 |
|---|---|---|
| `RealTimePathEntity` | 内部 | 单 feature 的模型实例容器，`rtp.getFeatureModel(id)` / `rtp.getAllFeaturesModel()` 返回 |
| `PositionFragment` / `AnimationFragment` / `PropertyFragment` | 内部 | 时间序列片段对象，由 `rtp.conversionFramePathData()` 生成、`updatePaths()` 消费 |

## See Also

- **cimcube-global** — `Defines`（`ShadowMode` 等数值枚举）、`Math` 坐标工具；**cimcube-setup** — 实例初始化
- **cimcube-layer** — 图层/实例模型（`deviceModelList` 的 modelId 常对应业务图层属性，供 `modelIdCallBack` 挑选）；RopeLines 绳管引擎属动画线束域（`Glodon.CIMCube.Animation.LineEffect`）
- **using-cimcube-skills/references/class-routing.md**（类目唯一真相源视图，自动生成）+ **using-cimcube-skills/references/domains-index.md**（总路由） — 本域类目路由与归属

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
