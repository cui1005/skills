---
name: cimcube-global
description: "CIMCube 全局基础 Global — 两个命名空间容器：Defines（运行时枚举常量）与 Math（静态方法：经纬度与世界坐标互转、坐标数组互转、图层局部坐标与世界坐标、矩阵配准、范围与包围盒判定、弧度角度、颜色转换、贝塞尔插值）。写 CIMCube 代码需要用枚举常量替代魔法字面量、或做经纬度与坐标/矩阵换算时使用。"
---

# CIMCube Global 基础 — 枚举常量 / 数学计算

> 命名空间挂载（SDK 源码 + 浏览器运行时双重核对）：
> - `Glodon.CIMCube.Global.Defines` — **枚举常量统一出口**（runtime 43 个键）
> - `Glodon.CIMCube.Global.Math` — 坐标系/局部坐标转换与几何判定工具（runtime 25 个方法）
>
> **定位**：本 SKILL.md 只做**索引路由**。完整清单（枚举 / 方法）与避坑记录外置于同目录 `references/*.md`。**写码前先按下方路由表读对应文件**，不要凭记忆重写。

---

## Quick Start（真实 API，可直接跑）

```js
const DEF    = Glodon.CIMCube.Global.Defines;      // 枚举常量
const M      = Glodon.CIMCube.Global.Math;         // 数学/坐标工具

// ① 枚举：以常量代替魔法字面量（值为小写字符串或数字）
const layer = map.layerManager.getLayer("myGraphicLayer");
layer.add(new Glodon.CIMCube.Graphic.PointGraphic({
  position: { x: 116.4, y: 39.9, z: 20 },
  style:    { color: "#ea4335", pixelSize: 10 },
}));

// ② Math：经纬度 ↔ 世界笛卡尔（ECEF 米）
const world = M.llhToCartesian(116.397428, 39.90923, 0); // → {x, y, z}
const llh   = M.cartesianToLlh(world);                    // → {lon, lat, height}
console.log(llh.lon, llh.lat);
```

---

## 模块路由索引

| 子模块 | 覆盖内容 | 何时用 | 写码前必读 |
|---|---|---|---|
| `Global.Defines` | 43 个枚举键，按 8 组展开：图层/影像/地形、图元/几何、输入事件、绘制/编辑/测量/捕捉、动画/状态/效果、光照/环境、模型地形裁剪、视图/布局/其它 | 凡 APIDOC 标注「可取值见 Defines」的参数，一律 `DEF.XXX.YYY` 引用而非写死字符串/数字 | `references/defines-enums.md` |
| `Global.Math` | 25 个静态方法：单位与颜色 / 经纬度↔笛卡尔 / 图层局部坐标与矩阵（含 SVD 配准）/ 点与元素范围判定 / AABB·OBB / 贝塞尔插值 | 坐标转换、图纸配准、范围判定；**通用距离/面积请改用 cimcube-utils 的 `Utils.Measure`** | `references/math-methods.md` |
| `Global.Logger` | 容器对象（runtime 实测）：`logger` 预建实例（12 个日志方法 + `setLevel`）+ `LogLevels`（7 级枚举）；用法与禁用项见避坑记录 | 需要全局日志时 | `references/pitfalls.md`「三、Logger 避坑」 |
| 避坑记录 | 枚举不存在项与拼写差异、Math 方法黑名单（APIDOC 有但 runtime 未导出）、`modelToWorldCoordinates` 签名偏差、Logger 容器结构 | 写码前先扫一遍，避免踩 APIDOC 与 runtime 不一致的坑 | `references/pitfalls.md` |

> ⚠️ 类数口径：`Global.Defines` / `Global.Math` 都是**命名空间容器**（非可实例化 Class），SDK 类计数中不计入；`Global.Logger` runtime 为容器（内含 Logger 类实例）。

---

## 性能建议

| 用法 | 说明 |
|---|---|
| 坐标转换尽量批量 | 用 `cartesianArrayToCoordinatesArray` / `coordinatesArrayToCartesianArray` 代替循环单点调，避免逐点对象创建 |
| 点选/范围测试用现成判定 | 优先 `calculatePointInCircleRange` 等，别自己写经纬度算法 |
| 大场景不要每帧转坐标 | 转换结果缓存；需要贴地高/采样用 cimcube-globe 的 `sampleTerrain*` |

## See Also

- **cimcube-layer** — `Defines.ImageryLayerType`（`tdt_img` 等）与 `Defines.LayerType` 的消费方，含内置 provider/默认 tk；内置底图（天地图/高德…）直接用枚举免填 url 的用法见 **cimcube-layer/references/imagery-terrain.md**
- **cimcube-camera** — 相机 `destination` 用 `{lon,lat,height}`，姿态弧度制；与 `Math.cartesianToLlh/llhToCartesian` 配合取位置
- **cimcube-graphic / cimcube-utils** — `GraphicType / MeasureMode / ModelClipMode` 等枚举的消费方；通用距离/面积计算用 `Utils.Measure`（`cimcube-utils/references/Measure.md`、`cimcube-utils/references/Clip.md`）
- **cimcube-globe / cimcube-scene / cimcube-realistic-rendering** — `TerrainLayerType / ShadowMode / SkyBoxMode` 的消费方
- **cimcube-analysis / cimcube-animation** — 分析/动画类没有集中类型枚举，直接 new 具体类

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
