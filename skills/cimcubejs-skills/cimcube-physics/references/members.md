# ScenePhysics 成员与 API 表

> 本文件对应 SKILL.md 路由表的「启停与状态、7 种刚体创建、状态与操控（速度/约束/CCD）API 表」行。

> 入口与最小示例见 `SKILL.md`；可运行片段见 `patterns.md`；坑位见 `pitfalls.md`。

## 一、启停与状态

| 用途 | API |
|---|---|
| 是否已开启 | `physics.enabled`（只读 Boolean） |
| 开启 | `enablePhysics(worldOrigin:[lon,lat,h], gravity:Number, softBody?:Boolean)`（异步） |
| 启用软体 | `enableSoftBody()` |
| 停止 | `disablePhysics()` |
| 重力 | `setGravity(y)`（或向量）/ `getGravity()` |
| 世界原点 | `getWorldOrigin()` |
| 坐标换算 | `physics.coordinates`（ENU 与经纬度等转换） |

## 二、刚体创建

**通用 options**：`mass`（0=静态，默认 0）、`friction`（默认 0.5）、`restitution`、`position:[x,y,z]`、`rotation`（欧拉或四元数）、`scale`、`collisionGroup`、`collisionMask`。各自特有：

| 方法 | 特有参数 | 说明 |
|---|---|---|
| `createBox({size})` | `size:[w,h,d]` | 方体 |
| `createSphere({radius})` | `radius` | 球 |
| `createCylinder({radius,height})` | `radius/height` | 圆柱 |
| `createCone({radius,height})` | `radius/height` | 圆锥 |
| `createConvexHull({points})` | 顶点集 | 凸包 |
| `createHeightfield({...})` | 高度场 | 地形刚体 |
| `createCloth({...})` | 布料 | 需先 `enableSoftBody()` |

## 三、状态与操控

| 用途 | API |
|---|---|
| 线/角速度 | `setLinearVelocity(handle,[x,y,z]) / getLinearVelocity`、`setAngularVelocity / getAngularVelocity` |
| 刚体绑定渲染 | `bindVisual(handle, renderObject)` |
| 铰链 / 滑块 / 球点约束 | `addHinge(opts)`、`addSliderConstraint(opts)`、`addPoint2PointJoint(opts)` |
| 铰链电机 | `setHingeMotor(handle, {targetVelocity,...})` |
| 接触回调 | `collectContactImpulses(cb)`（返回冲量）、`addAfterStepListener(cb)` |
| 调试渲染 | `renderDebugCollisionBox({color,...}) / removeDebugCollisionBox()` |
| 连续碰撞 CCD | `setCCDEnabled(bool)`、`setCCDThreshold(v)`（防高速穿透） |

> 调参注意：程序化刚体若不 `bindVisual` 仅参与物理计算；软体（绳/布）只建议轻量使用；大量刚体碰撞优先用图层 mesh 碰撞而非每个三角面生成盒子。调试完用 `removeDebugCollisionBox` 清除。
