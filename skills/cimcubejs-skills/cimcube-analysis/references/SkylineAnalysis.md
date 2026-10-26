# SkylineAnalysis — 天际线分析（可运行示例）

> 前提：见 `patterns.md`（`map`、`A`）。
> 对**当前视角画面**中的建筑/模型轮廓做天际线标记（`dynamic`=动态后期效果，`static`=静态处理），并返回逐列高度趋势数组供二次渲染。适合城市界面天际线观察、视口级校验。

构造参数：

| 字段 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `width` | `Number` | — | 天际线像素宽度 |
| `color` | `String`/`Color` | — | 天际线颜色 |
| `mode` | `String` | `dynamic` | `dynamic`（动态） / `static`（静态） |

方法：`doAnalysis()` 开始分析；`getSkylineArray()` 返回一维数组（逐列高度趋势，值域约 `0~1`）；`setSkylineWidth(w)` / `setSkylineColor(c)` / `setSkylineMode(m)`；`clear()` 清除；`dispose()` 销毁。

```js
const sky = new A.SkylineAnalysis(map, { width: 2, color: "#00ff00", mode: "dynamic" });
sky.doAnalysis();

const arr = sky.getSkylineArray();          // 逐列高度趋势，可用于自绘 2D 天际线
sky.setSkylineColor("#ffd54f");
sky.setSkylineMode("static");
sky.setSkylineWidth(3);
// sky.clear();
sky.dispose();
```

> ⚠️ 该类是**视口画面级**识别（非从指定观察点输出几何折线/开阔度），也没有 observer/radius 参数；凡需「指定观察点 + 半径 → 折线/可见比例」的能力请用 ViewShedAnalysis + VisibilityAnalysis 组合实现。
