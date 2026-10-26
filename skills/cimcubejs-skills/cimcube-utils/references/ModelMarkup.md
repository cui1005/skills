# Utils.ModelMarkup — 模型表面折线批注

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

模型面绘折线（每个拾取点带顶点标记），折线分段对应**段标签**；同一实例有**绘制（DRAW）与编辑（EDIT）两个互斥阶段**。`options` = ModelMarkupOptions：`type`（当前仅 `"polyline"`，省略=该值，其他类型抛 `Unsupported ModelMarkup type`）+ 样式。

```js
const mm = new U.ModelMarkup(map, {
  type: "polyline",
  lineStyle:  { color: "#F99D0B", width: 2, type: "solid" },   // type: solid|dash|dashes；可加 gapColor/dashLength/gapLength
  vertexStyle:{ size: 8, color: "#F99D0B", outlineColor: "#FFFFFF", outlineWidth: 2 },
  labelStyle: { text: "模型批注", fontSize: 14, fillColor: "#FFFFFF",
                backgroundColor: "rgba(32,38,47,0.85)", backgroundPadding: [7, 5] },
});
// 事件 = MapActionType（注册 / 解绑：mm.on/off，off 可只传 type 或带回调）
mm.on(DEF.MapActionType.ADD,     (carto) => console.log("拾取到点", carto));   // Cesium.Cartographic
mm.on(DEF.MapActionType.FINISH,  (item) => console.log("完成", item.id, item.graphic)); // MarkupItem（右键结束，≥2 点）
mm.on(DEF.MapActionType.CHANGED, (evt) => console.log(evt.changeType, evt.segmentIndex, evt.item)); // changeType='geometry'|'label'
mm.on(DEF.MapActionType.DELETE,  ({ id }) => console.log("删除", id));
mm.activate(DEF.DrawEditMode.DRAW);   // 进入绘制（先关编辑）。参数省略默认 DRAW
// …左键在模型表面拾点，右键结束一条…
mm.activate(DEF.DrawEditMode.EDIT);   // 互斥切换：退出绘制进入编辑
mm.selectItem(id);                    // 选中某条并进入节点编辑
mm.getSelectedItem();                 // 当前选中批注（MarkupItem|undefined）
mm.clearSelection();                  // 取消选中并退出节点编辑
mm.deactivate();                      // 回到 NONE
mm.get(id); mm.delete(id); mm.clearAll();
mm.getMarkupLayer();                  // GraphicLayer（折线+顶点）
mm.getLabelLayer();                   // DivLayer（段标签）
mm.mode; mm.isDrawing; mm.isEditing;  // 只读阶段状态
mm.dispose();                         // 释放图层/事件/交互，不可再用
```

> MarkupItem 结构（运行时）：`{ id, type, graphic, vertexGraphics[], segments[{index, labelDiv, text}] }`。编辑阶段的改动（拖节点=geometry、改标签文本=label）都会派发 `CHANGED`。若只需在模型表面做一次性取点，更轻量的是 `map.controls` + 吸附（`Snap.md`）。

## 常见模式：模型折线批注生命周期（绘制→查询→删除）

```js
const mm = new U.ModelMarkup(map, { type: "polyline" });
let lastId;
mm.on(DEF.MapActionType.FINISH, (item) => { lastId = item.id; mm.activate(DEF.DrawEditMode.EDIT); mm.selectItem(lastId); });
mm.activate(DEF.DrawEditMode.DRAW);
// 需要清除时： mm.deactivate(); mm.dispose();
```

## 性能

| 工具 | 开销 | 建议 |
|---|---|---|
| ModelMarkup | 低~中 | 一条 = 多点折线；批注多时监听 DELETE 及时同步业务 |
