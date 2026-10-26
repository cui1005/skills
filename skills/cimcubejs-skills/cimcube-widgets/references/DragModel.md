# DragModel 模型三维编辑

## 1. 最小用法

```js
const dragModel = new W.DragModel(map, {
  showTips: true,                 // 默认 true：激活后屏幕提示
  tips: "单击鼠标左键选择需要操作的模型",  // 自定义提示文案（默认值即此句）
  continueEdit: true,             // 默认 false：true 时编辑完一个自动接手下一个点选
  UIScaling: 1.0,                 // 默认 1.0：Gizmo 缩放
  noUI: false,                    // 默认 false：true 隐藏整套编辑 Gizmo（纯逻辑平移/旋转）
  enableScaleX: true, enableScaleY: true, enableScaleZ: true,  // 缩放轴默认全部 false
});
dragModel.activate();
// 取消/收尾：
// dragModel.deactivate();   // 停用（保留目标与场景状态，可再次 activate）
// dragModel.clear();        // 清空（恢复场景，不销毁实例）
// dragModel.dispose();      // 彻底释放（不再使用）
```

## 2. 目标怎么来

被编辑对象可以是：实现了 `getDragableTarget()` 的拾取实体（返回带可拖拽标记的目标），或原生 `Model` / `Cesium3DTileset` / 实例图层成员（内部有 `_tryGetTarget` 自动识别）。两种进入编辑的方式：

1. `activate()` 后直接在场景**左键单击**模型（开启 `showTips` 时会出现"单击左键取点"类引导文字），内部选中即显示 Gizmo；
2. 自己的拾取逻辑中拿到目标后调用 `dragModel.drawMoveHandle(target)` 进入编辑。

运行时还暴露 `setTargetLayer(layer)`（API_DOC 未列）把后续操作限定在某图层成员（对实例图层 / 3DTiles 比较有用）。

## 3. 三维 Gizmo 与选项（全部键见 DragModel.html，此处为默认值速记）

| 分组 | 默认行为 |
|---|---|
| 平移 | `enableTranslationX/Y/Z` **true**，平面 `XY/YZ/XZ` **true**；各轴/面颜色可配（默认红/绿/蓝等） |
| 旋转 | `enableRotationX/Y/Z` **true**（三个旋转环） |
| 缩放 | `enableScaleX/Y/Z` **false**；整体范围 `targetScaleRange` 默认 `[0.1, Number.MAX_SAFE_INTEGER]` |
| 方向锁定 | `enableLockDirectionTranslationXY/YZ/XZ` 等默认 **false** + 各锁颜色与 `lockDirectionAngle` 角度阈值（同向锁定） |
| 右键移动 | `enableRightButtonMove` 默认 false（开启时支持右键平移目标，**仅 3DTiles**） |
| 延续编辑 | `continueEdit` false；`UIScaling` 1.0；`noUI` false（不建 Gizmo） |
| 自定义 UI | options 传 `dragModelUI` / `dragModelTipUI`（替换默认编辑控件与提示） |

编程驱动（目标已确定）：`dragModel.rotateTarget([rx, ry, rz])`（旋转角增量）与 `dragModel.translateTarget([dx, dy, dz])`（位移增量，世界/模型轴按当前 Gizmo 态）；`dragModel.UI` getter 可取当前编辑 UI。

## 4. 样式与渲染说明

Gizmo 是 **Cesium 三维 Primitive 组合**（平移轴箭头 / 旋转环 / 缩放盒 + 平面手柄），**不是 DOM**，因此没有对应 CSS；全屏提示文字用 HTML overlay（对应 `dragModelTipUI`）。**不要**用 CSS 去改 Gizmo 颜色——改 options 各轴颜色项。
