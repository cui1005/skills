---
name: cimcube-physics
description: "CIMCube 场景物理引擎 ScenePhysics：设定世界原点与重力；刚体（盒/球/柱/锥/凸包/高度场/布）；模型图层碰撞；线角速度与约束（铰链/滑块/点对点）及铰链电机；接触回调与 CCD 防高速穿透；软体（绳/布）。当用户需要做物理模拟、刚体、碰撞检测、重力下落、软体、约束时激活。"
---

# CIMCube 场景物理引擎（ScenePhysics）

> 本技能只覆盖 **`map.scene.physics`（根类 `Glodon.CIMCube.ScenePhysics` 的惰性单例）**。渲染环境见 `cimcube-scene`；真实感光照见 `cimcube-realistic-rendering`；参与碰撞的模型图层加载见 `cimcube-layer`。
>
> **SKILL.md 是纯路由索引**：本页只保留「入口速记 + 路由表」；启停/刚体/操控 API 表、可运行模式、避坑与性能全部下沉到 `references/`。

## 入口速记

| 你要的东西 | 真实路径 |
|---|---|
| 物理单例 | `map.scene.physics`（惰性单例；**用方法，不用属性直改**） |
| 开启 | `await map.scene.physics.enablePhysics(worldOrigin, gravity)` |
| 不存在 | `map.scene.ScenePhysics` / `map.scene.ScenePhysics.enabled` / `.gravity` / `.debugDraw`（对象属性直改） |
| 坐标系 | **原点 ENU 局部坐标（米）**，非经纬度 |
| 刚体显示 | 程序化刚体仅参与物理；要显示需 `bindVisual(handle, renderObject)` |

## 路由表（按需读取，勿整包加载）

| 你要查什么 | 读这个文件 |
|---|---|
| 启停与状态、7 种刚体创建、状态与操控（速度/约束/CCD）API 表 | `references/members.md` |
| Quick Start 与「模型图层参与碰撞」可运行片段 | `references/patterns.md` |
| 避坑记录（易错写法 → 真实 API）+ 性能建议 | `references/pitfalls.md` |

## See Also

- **cimcube-layer** — 参与碰撞的 gltf/S3M 图层加载与 `setCollisionModel` 目标
- **cimcube-scene** — 场景环境与渲染（`scene` 单例本身、大气/天空/阴影/画质）
- **cimcube-realistic-rendering** — 真实感光照（物理与渲染解耦，刚体需 `bindVisual` 才可见）
- **cimcube-graphic** — 用图元/模型作为 `bindVisual` 的渲染对象

## 本域类目归属清单

- `using-cimcube-skills/references/class-routing.md` — 「类 → 归属技能」与类数的**唯一真相源视图**（自动生成物，勿手改；由仓库侧类目登记表生成的视图）。本域条目见其「§二 逐类归属」对应小节；各类的构造参数 / 方法用法见上方路由表指向的 `references/*.md`。
- `using-cimcube-skills/references/domains-index.md` — 命名空间总路由与各域入口文件（手写导航）。
