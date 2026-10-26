# ScenePhysics 避坑记录与性能建议

> 本文件对应 SKILL.md 路由表的「避坑记录（易错写法 → 真实 API）+ 性能建议」行。

> 由 `SKILL.md` §五 + 性能建议 外置而来。API 表见 `members.md`，模式见 `patterns.md`。

## 避坑记录（易错写法 → 真实 API）

1. **`map.scene.ScenePhysics.enabled/gravity/debugDraw`（对象属性直改）不存在** → `map.scene.physics` 实例 + 方法（`enablePhysics()`、`setGravity()`、`renderDebugCollisionBox()`）。
2. **`map.scene.ScenePhysics` 顶层写法不存在** → 经 `map.scene.physics` 取惰性单例。
3. 坐标为**原点 ENU 局部坐标（米）**，不是经纬度；经纬度只在 `enablePhysics(worldOrigin)` 时给一次。
4. 程序化刚体（`createBox/createSphere/...`）**只参与物理、不带渲染**；要看见必须 `bindVisual(handle, renderObject)`，或直接用图层 `setCollisionModel`。
5. 解算**自动随帧推进**，无需手动 `step()`。
6. 软体（`createCloth` / `enableSoftBody`）性能敏感，仅轻量使用。
7. 高速物体穿透 → 开 CCD（`setCCDEnabled(true)` + `setCCDThreshold(v)`）。

## 性能建议

| 场景 | 推荐 |
|---|---|
| 通用 | 只对交互对象开刚体；场景静止时 `disablePhysics()` 释放 |
| 高速碰撞 | 才开 CCD（有额外开销） |
| 软体 | 数量少（绳/布），避免大面积布料 |
| 大量刚体 | 优先用图层 mesh 碰撞，而非每个三角面生成盒子 |
