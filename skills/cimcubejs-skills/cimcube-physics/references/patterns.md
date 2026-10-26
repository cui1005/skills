# ScenePhysics 可运行模式

> 本文件对应 SKILL.md 路由表的《Quick Start 与「模型图层参与碰撞」可运行片段》行。

> API 表见 `members.md`；坑位见 `pitfalls.md`。

## Quick Start

```js
const physics = map.scene.physics;

// 1) 开启：世界原点(经纬高) + 重力（第 3 参为 softBody 开关）
await physics.enablePhysics([116.4, 39.9, 10], -9.81);
physics.enableSoftBody();                 // 启用软体（绳/布）约束解算

// 2) 程序化刚体（仅物理，不带渲染；要显示需 bindVisual）
const floor  = physics.createBox({ size: [10, 10, 1], position: [0, 0, -0.5], mass: 0 });  // mass=0 静止
const sphere = physics.createSphere({ radius: 0.5, position: [0, 0, 5], mass: 1, friction: 0.5, restitution: 0.2 });

// 3) 抛一下（解算自动随帧推进，无需手动 step）
physics.setLinearVelocity(sphere, [0, 1, 0]);

// 4) 停止
physics.disablePhysics();
```

## 模型图层参与碰撞（对模型更真实，建议优先）

```js
// 使整个 S3M/glTF 图层参与物理；type 取 CollisionType（box/sphere/…/mesh）
physics.setCollisionModel(gltfLayer, { type: 'mesh', mass: 0, useTriangleMesh: true });
physics.setCollisionModelNode(gltfLayer, nodeId, { type: 'box', mass: 1 }); // 单个节点动态化
physics.resetCollisionModelNode(gltfLayer, nodeId);   // 还原该节点为静态
physics.removeRigidBody(gltfLayer, nodeId);           // 移除
```
