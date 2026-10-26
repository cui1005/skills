# Utils.Snap — 吸附（把"左键取点"变成"落吸附点"）

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

## 相关枚举

```js
const D = Glodon.CIMCube.Global.Defines;
D.SnapTargetType;     // {MODEL_3D:'model3D', VECTOR_2D:'vector2D', BOTH:'both'}
D.SnapType;           // {ENDPOINT:'endPoint', MIDPOINT:'midPoint', LINE:'line', FACE:'face', INTERSECTION:'intersection'}
```

## 原理与构造 options

原理：`new Utils.Snap(map, opts)` 是**开关式**吸附服务。`snap.enable()` 后它会把自己注册为 `map.controls.Snap`、置位控制器的 `_enableMouseSnap`；此后鼠标靠近吸附特征时高亮显示吸附标记，**左键点击会以吸附点（而非鼠标原始位置）落点**——正在 active 的 TakeCoordinates 收到的 `ADD`/底层 `LEFT_CLICK` 坐标即吸附后坐标。
构造 options（全部可选，默认与 `map.controls` 配置一致）：

| key | 说明 |
|---|---|
| `targetType` | `D.SnapTargetType`，默认跟随 `map.controls.snapTargetType`（默认 both） |
| `typeList` | `D.SnapType` 数组，默认跟随 `map.controls.snapTypeList`（默认 ['face','endPoint']） |
| `tolerance` | 像素容差（默认 7） |
| `screenCaptureFrameRate` | 捕捉帧间隔（ms，文档默认 50） |
| `faceColor/faceFillColor/lineColor/pointColor/midPointColor/roundPointColor/intersectionColor` | 各类吸附标记颜色 |
| `faceWidth/endPointRadius/midPointRadius/roundPointRadius/intersectionRadius` | 吸附标记尺寸（像素） |

## 方法与属性

- `enable()`：开启吸附（内部 `controls._enableMouseSnap = true`）。
- `disable()`：关闭（内部置 false 并 `clear()` 清吸附高亮）。
- `clear()`：移除吸附高亮标记、清空 `controls.snapSelectedResults`。
- `dispose()`：销毁。
- `addWithoutSnapObject(obj)` / `removeWithoutSnapObject(obj)`：加入/移出"不参与吸附"白名单（如拾取的临时把手）。
- 属性 setter/getter：`snapTargetType` / `snapTypeList` / `tolerance` / `screenCaptureFrameRate` / `showResults` / `faceWidth` / 各颜色与半径（改后即时生效，不必重建）。
- **事件 `'mouseSnapClickup'`**（源码 fire，payload `e.arguments` = `map.controls.snapSelectedResults`，含 `snapWorldPosition` 等）：吸附高亮状态下左键命中吸附点时触发，随后自动清高亮。

```js
const snap = new Glodon.CIMCube.Utils.Snap(map, {
  targetType: D.SnapTargetType.BOTH,
  typeList: [D.SnapType.FACE, D.SnapType.ENDPOINT],
  tolerance: 7,
});
snap.on('mouseSnapClickup', (e) => {
  console.log('吸附落点', e.arguments.snapWorldPosition);   // 世界坐标 Cartesian3
});
snap.enable();                       // 场景吸附生效
// 业务：同一时刻只需监听 TakeCoordinates 的 add/finish 即可拿到吸附后的坐标
// 用完：snap.disable();
```

> 与「取点控件」的完整事件体系（`MapActionType.ADD/MOVE/FINISH`、底层 `ScreenSpaceEventType`）见 cimcube-controls。

## 避坑

臆造 API → 真实 API 完整对照见 `pitfalls.md`（编辑族表）。要点：不存在 `snap.pick(x, y)` 与 `'SNAPPED'` 事件——吸附自动接管 `map.controls` 左键取点，事件是 `'mouseSnapClickup'`，结果在 `map.controls.snapSelectedResults`；`targetLayers` / `pixelTolerance` / `gridSize` 不是 Snap 的构造项，白名单用 `addWithoutSnapObject/removeWithoutSnapObject`。
