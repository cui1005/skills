# Animation.ModelEffect — 模型动态（4 类 · 可运行示例）

> 前提：`map` 已创建、`Tiles3DLayer`/GLTF 图层已加载；`const E = Glodon.CIMCube.Animation`；命名空间 `Glodon.CIMCube.Animation.ModelEffect.*`。
> **4 类均无 show/hide**，按任务驱动：`ModelExplode` 用 `doExplode()/cancelExplode()/setOpts()`，`ModelInteroperation` 用 `play()/restore()/removeFeatures(ids)/setOpts()`，`UVTextureEffect` 用 `setLayer(layer)/setOpts()/dispose()`（`show` 是布尔属性），`ModelComplexColor` 用 `enable()/disable()/removeFeatures(ids)/setOpts()`。
> `ModelExplode` / `ModelInteroperation` 实现上继承 `EventDispatcherSDK`（APIDOC 未列出具体事件名）。

## 1. ModelExplode — 模型炸开（BIM 楼层分解）

```js
const exp = new E.ModelEffect.ModelExplode(map, {
  tiles3DLayer: s3mLayer,         // 必填，Tiles3DLayer 图层，需要保证图层ready
  duration: 2,                   // 炸开动画时间（秒），默认 2
  offset: { x: 0, y: 5, height: 1 },  // 楼层炸开偏移
  levelInfo: [                    // 指定楼层炸开信息
    { levelName: "F1", x: 0, y: 5, height: 1 },
    { levelName: "F2", x: 0, y: 10, height: 2 },
  ],
  mode: 1,                       // 1 投影坐标 | 2 地理坐标，默认 1
});
exp.setOpts({ duration: 4 });
exp.doExplode();      // 执行炸开
exp.cancelExplode();  // 撤回
exp.dispose();
```

## 2. ModelInteroperation — 模型单体要素动画

对 `Tiles3DLayer` 中指定 featureId 的要素做**整体偏移/旋转**（东北天 ENU 坐标系）。核心参数 `ids/offset/rotation/rotationCenter` 位于 `interoperationList` 的**每一项内**（不在 options 顶层）。

```js
const mi = new E.ModelEffect.ModelInteroperation(map, {
  tiles3DLayer: s3mLayer,        // 必填，Tiles3DLayer 图层，需要保证图层ready
  duration: 2,                   // 动画时间，默认 2s
  field: "id",                   // featureId 对应字段名，默认 "id"
  interoperationList: [          // 效果列表
    {
      ids: ["feature-1", "feature-2"],  // featureId 集合
      offset: { x: 0, y: 0, height: 0 },  // 东北天偏移：x=东, y=北, height=天；默认 0
      rotation: { x: 0, y: 0, z: 0 },     // 绕 X/Y/Z 轴旋转角度（度），默认 0
      rotationCenter: { x: 0, y: 0, z: 0 }, // 旋转中心点（东北天），默认 0
    },
  ],
});
mi.play();                                    // 执行动画
mi.removeFeatures(["feature-1"]);             // 移除指定 featureId 的偏移/旋转（需传 ids）
mi.restore();                                 // 复原（撤回全部效果）
mi.setOpts({ duration: 1, interoperationList: [/* 同上结构 */] });  // 更新参数
mi.dispose();
```

## 3. UVTextureEffect — 白模贴图

给白模（3D Tiles/GLTF）侧面(facade)与顶面(top)贴材质。`texture` 与 `color` 同传时按 `mixLevel` 混合：0 = 颜色不融合（只用贴图），1 = 只用颜色。

```js
const uvt = new E.ModelEffect.UVTextureEffect(map, {
  // tileset 可选：Cesium3DTileset 对象；info 可选：材质配置
  info: {
    facade: { texture: "./assets/buildings.jpg", color: "red", mixLevel: 0 }, // 侧面
    top:    { texture: "./assets/roof.jpg",       color: "red", mixLevel: 0 }, // 顶面
  },
});
uvt.setLayer(gltfLayer);   // 设置要贴图的图层 —— setLayer(layer) 单参
uvt.setOpts({              // 运行中更换贴图；支持 {layer, info}，info 不支持数组
  layer: gltfLayer,
  info: {
    facade: { texture: "./assets/buildings-2.jpg", color: "red", mixLevel: 0 },
  },
});
uvt.show = true;           // ⚠️ show 是布尔属性（无 show()/hide() 方法）
uvt.dispose();
```

## 4. ModelComplexColor — 模型单体要素按高程分段着色

按要素的高程把模型分成若干段并逐段着色（BIM 构件分级着色 / 淹没水位分级）。`colorList` 里每段给 `ids`（featureId 集合）与 `colorStops`（高程比例色带，`ratio` 为该段上界，0~1）。

```js
const mcc = new E.ModelEffect.ModelComplexColor(map, {
  tiles3DLayer: s3mLayer,      // 必填，Tiles3DLayer 图层，需要保证图层 ready
  field: "id",                 // featureId 对应字段名，默认 "id"
  mode: "step",                // step 硬切分 | gradient 段间插值，默认 step
  replace: true,                // true 替换漫反射，false 与原材质相乘，默认 true
  colorList: [                  // 着色列表
    { ids: ["feature-1", "feature-2"], colorStops: [{ ratio: 0.5, color: "#2f6fdd" }] },
    { ids: ["feature-3"],             colorStops: [{ ratio: 1.0, color: "#e04b3a" }] },
  ],
});
mcc.enable();                       // 写入纹理并开启着色
mcc.removeFeatures(["feature-1"]); // 移除指定 featureId 的分段着色
mcc.setOpts({ mode: "gradient" });  // 更新参数（切段间插值）
mcc.disable();                      // 关闭着色，恢复原材质
mcc.dispose();
```

## 性能建议

| 类 | 开销 | 建议 |
|---|---|---|
| ModelExplode | 中 | 炸开完毕后保留状态，不要反复 doExplode/cancelExplode |
| ModelInteroperation | 中 | ids 数量 <500；duration 适中避免卡顿 |
| UVTextureEffect | 中 | 大模型贴图在加载完成后再 setLayer |
| ModelComplexColor | 中 | 色带分段数与 ids 数量都会放大纹理写入量，演示结束及时 `disable()`/`dispose()` |
