# Utils.BoxDragger — 三击取盒（⚠️ APIDOC 页名 BoxDraggerUI）

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

- 运行时创建：`new U.BoxDragger(map)`（**不是** `U.BoxDraggerUI`）。
- `start(finishCallback, offset = 0)`：进入取盒流程，内部复用 `map.controls.takeCoordinates`（等效 `active(3)`）——第 1 击定底面角 A，移动中第 2 击定对角 B 后**上下移动鼠标拉高度**（有实时盒体线框），第 3 击确认高度并回调。
- 回调载荷（源码/实测）：`{ posAInWorld, posBInWorld, height, posAInCartegraphic, posBInCartegraphic, posCInCartegraphic }`——前两个为底面两角世界坐标 Cartesian3，`height` 米，后三个为经纬高 `{lon,lat,height}`。
- `dispose()`：移除盒体线框与清理。
- 典型组合：取盒 → 交给 `BoxClip.clipEx` 做全局盒剖。

```js
const dragger = new U.BoxDragger(map);
dragger.start((r) => {
  console.log('盒高', r.height, '角点LLH', r.posAInCartegraphic, r.posBInCartegraphic);
  boxClip.clipEx(r.posAInWorld, r.posBInWorld, r.height, 0);   // 进入盒剖切
  dragger.dispose();
});
```

## 组合：盒剖 + 取盒（用户框选一个基坑范围查看地下）

```js
const box = new U.BoxClip(map);
const dragger = new U.BoxDragger(map);
dragger.start((r) => {
  box.clipEx(r.posAInWorld, r.posBInWorld, r.height, 0);   // 0 = 贴地表向下看地下
  dragger.dispose();
});
// 需要恢复：box.clear(); box.dispose();
```

> ⚠️ 命名口径坑：APIDOC/类文档页名写 `BoxDraggerUI`，但浏览器 `Glodon.CIMCube.Utils` 只有 `BoxDragger`（class 注释名 ≠ 导出名），`new` 一律用 `U.BoxDragger`。臆造 `new U.BoxDraggerUI(map)` / "拖手柄调已存在的盒" 均不成立——`start(cb, offset)` 是**三击**（2 角 + 高度）回调。
