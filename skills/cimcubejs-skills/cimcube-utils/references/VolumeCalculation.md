# Utils.VolumeCalculation — 3DTiles 体积测算

> 归属 `cimcube-utils`。公共初始化见 `patterns.md`。

`new U.VolumeCalculation(map [, { layer, field }])`：对 **Tiles3DLayer** 进行瓦片级体积统计，`extends EventDispatcherSDK`。**没有 calculate()/Promise 返回值**，正确用法：

```js
const vol = new U.VolumeCalculation(map, {
  layer: tiles3dLayer,      // 指定图层（必选能力；setLayer 只接受 Tiles3DLayer）
  field: "id",              // 指定字段（如 featureId 所在属性）
});
vol.on("change", (e) => console.log("体积更新", e)); // 事件字符串为 "change"（小写）
vol.do();                       // 开始计算（内部监听图层可见性 / 瓦片加载后统计）
// 计算完成后读取：
vol.getVolume();               // -> Number 总体积（change 数据未到达前可能 undefined）
vol.getVolume("feature-1");    // -> 指定 featureId 的体积（缺省为 0）
vol.setLayer(otherTiles3dLayer);
vol.dispose();
```

- 开销：中（随瓦片加载统计）。图层关掉/换层需 `setLayer`；不显示用 `dispose`。
- 与 `Analysis.CutFill`（填挖方，cimcube-analysis `references/CutFill.md`）的区别：CutFill 关联设计面/地形算填挖量，`VolumeCalculation` 只按 3D Tiles 的几何做体积测算。
