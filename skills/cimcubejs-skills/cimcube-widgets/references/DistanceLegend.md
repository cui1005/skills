# DistanceLegend 距离图例（比例尺）

`new DistanceLegend()` **无参构造** + `addToMap(map)`；挂载后内部起 `Utils.DistanceLegendService`，监听其 `change` 事件自动刷新。手动刷新的公开入口是 `updateContent(info)`：

```js
const legend = new W.DistanceLegend(); legend.addToMap(map);  // ★ 返回 undefined，句柄用 legend 本身
legend.updateContent({ barWidth: 120, distance: 500 }); // 条宽(px) / 代表的实际距离(米)
// distance ≥ 1000 时自动格式化为 "X km"（如 1 km），否则 "X m"
```

- **单例**：重复 `addToMap` 会抛 `Cannot redefine property`、`map.distanceLegend` 访问器不存在、`addToMap` 返回 `undefined`（机制与防重见 **references/pitfalls.md**「单例与句柄」条）；句柄用上面的 `legend` 或根 DOM `.cimcube-distance-legend`。
- DOM：`.cimcube-distance-legend` > `.cimcube-label`（距离文本）+ `.cimcube-scale-bar`（比例条，宽度/居中由 `barWidth` 算，基准 125px 内居中）。
- 初始 wrapper `visibility:hidden`，首次拿到数据才显示。
- 继承 `BaseWidget` → 有 `enabled`（真正停用即停内部服务）与 `show()/hide()`（仅 visibility）。

## 样式

`Resource/css/DistanceLegend.css`。换位示例：`.cimcube-distance-legend { right: 20px; }`。
