# CameraRoam / GroundRoam 路径漫游录制对话框

两者构造一致，`extends Widgets.DraggableDialog`（对话框能力继承自内部基类 `DraggableDialog`，**未导出**），对外只多一个 `dialog`（DOMElement）成员与继承来的 `showDialog()`：

```js
const roamDlg = new W.CameraRoam(
  document.body,        // container：放置对话框的 DOM
  "cameraRoamDlg",      // dialogId
  "路径漫游录制",        // dialogTitle
  map,                  // Glodon.CIMCube
  map.camera            // Glodon.CIMCube.Camera
);
roamDlg.showDialog();
// roamDlg.dialog  → 对话框 DOM（可自行做显隐/定位）
```

## 内置 UI（knockout ViewModel，交互已核实）

- **路径点表格**：每行"操作（删除 / 定位 / 替换 / 插入）+ 经度(°) / 纬度(°) / 椭球高(m) / 方位角(°) / 俯仰角(°) / 翻滚角(°) / 时长(s)"的可编辑输入框；时长默认 2s/点，总时长自动求和。
- **添加路径点**：在场景里**左键点击拾取**——鼠标移动有"单击左键取点"提示 label，落点生成蓝色 pin（`PinBuilder` 图片）；`GroundRoam` 拾点默认在相机高度基础上加 `heightDiff 10m`（相机位置可高于地形）。
- **回放**：由 `Animation.Effect` 底层的 `PathAnimation(camera, { timeAheadInSecond: 2 })` 执行，支持播放比例（`.cimcube-playRatio` 输入，按钮区 `.cimcube-buttonset`）；播放结束自动复位为未播放状态。

> 这些数据都在内部 ViewModel（非对外 API）：在 class 上只有 `dialog / showDialog` 可编程操作；要深度控制录制点请基于 camera 路径动画自己封装（`cimcube-animation`）。`GroundRoam` 与 `CameraRoam` 的差异是"贴地"（面向地面约束/贴地路径）；录制取点与回放形态一致。

## 样式

`CameraRoam.css`（对话框 `.cimcube-drag-resize-dialog` 800×500，内容表格 `.cimcube-coordCol / .cimcube-col / .cimcube-action`、`.cimcube-buttonset .cimcube-playRatio`）；通用对话框皮肤在 `Draggable-resizable-dialog.css`（`.cimcube-drag-resize-dialog`、`.cimcube-titlebar`）。
