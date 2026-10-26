# FacadeAnalysis — 建筑立面图 / 顶视图（可运行示例）

> 前提：`map` 已创建；`const A = Glodon.CIMCube.Analysis`。
> 生成建筑物**立面图（含展开体块）**或**顶视图**。结果通过回调异步返回。

构造 options（类页）：`mode`（`"facade"` 立面图 / `"top"` 顶视图）、`line`（参考线坐标）、`height` / `length`（展开范围）、`outputMaxSize`（输出图最大尺寸）、`showVolume` / `volumeColor` / `volumeAlpha`（体块显示与配色）。

**主要方法：** `doAnalysis(callback)`、`getMergedImage(callback)`、`getMode()`、`setMode()`、`setLine()`、`setHeight()`、`setLength()`、`setShowVolume()`、`setVolumeColor()`、`setVolumeAlpha()`、`setOutputMaxSize()`、`clear()`、`dispose()`。

```js
const fac = new A.FacadeAnalysis(map, {
  mode: "facade",            // "facade" 立面图 | "top" 顶视图
  line: [/* 立面展开参考线，坐标形制见类页 */],
  height: 60,                // 展开高度范围（示意）
  length: 200,               // 展开长度范围（示意）
  showVolume: true,          // 是否显示分析体块
  volumeColor: "#1a73e8",
  volumeAlpha: 0.6,
  outputMaxSize: 4096,       // 输出图最大尺寸（示意）
});

fac.doAnalysis((result) => {
  console.log("立面图结果", result);   // 结果结构以 API_DOC 为准
});

// 改视图模式后重新分析（mode/line/height/length 等均有对应 setter）
fac.setMode("top");
fac.doAnalysis((result) => {});

// 导出合并图（多段立面拼合等场景）
fac.getMergedImage((img) => console.log("合并图", img));

fac.clear();                 // 清空结果
fac.dispose();
```
