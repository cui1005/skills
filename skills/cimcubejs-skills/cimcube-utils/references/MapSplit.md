# Utils 卷帘/分屏对比 — MapSplit / MultiViewPort

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

两者构造签名、机制、方法几乎一致（`MapSplit` 卷帘为左右/上下两半拖动分隔线；`MultiViewPort` 分屏比对，按图层分组切换哪组可见）。注意 opts 里图层数组**运行时读取的是 `viewPortLayersVisible`**（个别 API_DOC 版本把 MapSplit 的写成 `layersVisible`，成员方法 `setSplitLayersVisible` 存在——以运行时键为准）。

```js
const ls = [
  { id: layer2018.id, visible: true  },  // 这一侧显示 2018
  { id: layer2025.id, visible: false },  // 另一侧显示 2025
];
const split = new U.MapSplit(map, map.layerManager, {
  mode: "horizontal",                    //Glodon.CIMCube.Global.Defines.ViewPortModeType.HORIZONTAL='horizontal'
  splitPosition: Math.round(innerWidth / 2), // 像素（默认画布宽/2）
  viewPortLayersVisible: ls,
});
split.begin();
split.setSplitPosition(200);             // 像素
console.log(split.getSplitPosition());
split.setSplitLayersVisible([/* 新的 {id,visible} 列表 */]);
split.stop();       // 结束（内部自动复位分屏视口，业务侧不要手动读写）
split.dispose();
```

```js
const mvp = new U.MultiViewPort(map, map.layerManager, {
  mode: "vertical",                      // ViewPortModeType：'horizontal' | 'vertical'
  viewPortLayersVisible: [ { id: groupA.id, visible: true }, { id: groupB.id, visible: false } ],
});
mvp.begin();
// mvp.setViewPortLayersVisible([...]);
mvp.stop(); mvp.dispose();
```

> 底层机制：`MapSplit`/`MultiViewPort` 通过视口裁剪（scissor）分区域渲染，由工具内部自管理；影像层（`LayerType.IMAGERY`）走 `setSplitDirection(LEFT/RIGHT)`，其余图层走可见裁剪。因此**同一时刻只允许一个分屏工具处于 begin**。UI 封装见 Widgets：`MapSplitWidget`（panels）`getMapSplit()` 拿到底层 Utils.MapSplit 实例。

## 常见模式：对比工具做 A/B 切换（正确启停顺序）

```js
let cur = null;
function compareA() {
  cur?.stop?.(); cur?.dispose?.();
  cur = new U.MapSplit(map, map.layerManager, { mode: "horizontal", viewPortLayersVisible: lsA });
  cur.begin();
}
function compareB() {
  cur?.stop?.(); cur?.dispose?.();
  cur = new U.MultiViewPort(map, map.layerManager, { mode: "vertical", viewPortLayersVisible: lsB });
  cur.begin();
}
```

## 性能

| 工具 | 开销 | 建议 |
|---|---|---|
| MapSplit / MultiViewPort | 中~高（区域重绘） | begin 态互斥，一次一个；完成即 `stop()`；影像层优先 |
