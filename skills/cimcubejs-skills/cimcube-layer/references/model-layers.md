# 模型类图层 — GltfLayer · GSMMLayer · I3STilesLayer · S3MTilesLayer · USDLayer

入口 `map.layerManager`。

> 变换三件套 `origin`/`offset`/`rotation`/`scale` 通用，详见 `references/tiles3d.md`。

## GltfLayer（单体模型，`type:'gltf'`）

```js
const gltf = new Glodon.CIMCube.Layer.GltfLayer({
  id: "car", name: "小车", url: "./data/car.glb",   // 或 gltfBuffer: ArrayBuffer
  origin: [116.39, 39.90, 0], rotation: [0, 0, 0], offset: [0, 0, 0], scale: 1,
  shadowMode: "disabled", canSelected: true,
  phongLighting: false, useDefaultMetallicRoughness: true, usedAnimations: true,
});
```

- 方法：`getBoundingSphere()` `getBoundingVolume()` `getBoundingVolumeHeight()` `invalidateBoundingVolume()` `getOriginWorldCoord()` `flyTo()` `move(angle,distance,mode?)`（**顺时针**）`updateLayerTransformation()` `setStyle({modelColor:{mode,color,mixAmount}, modelSilhouette:{color,size}})` `addAnimation(opt)` `addAllAnimations(playOpt)` `getAnimationByName(name)` `getAnimationByIndex(i)` `getAllAnimation()` `removeAnimation(t)` `removeAllAnimation()` `setClippingPlanes(value)` `setMultiClippingPlanes(value)` `addMultiClippingPlane(id,collections,region)` `removeMultiClippingPlane(id)` `addDragHandle()` + 坐标转换 `enuToWorldPosition/localToWorldPosition/worldPositionToLayerENU/worldPositionToLocal`
- 自有事件：`LOADED` `TRANSFORM_CHANGED`

## GSMMLayer（广联达 GSMM，`type:'gsmm'`）

需 `url`（.gsmm 文件）+ `assetsUrl`（assets 目录）。

- 专有：`getChildByID(id)` `updateModelMatrix()` `updateWeight(id, repeatCount)`（配重）`enableDynamicPointEffect(enable)` `setDynamicPointEffect(opts)` `showOrientedBoundingBox`
- 特有构造键：`color`（吊绳颜色，默认 `#191919`）`slices`（吊绳精细度，默认 16）
- 自有事件：`LOADED` `TRANSFORM_CHANGED`

## I3STilesLayer / S3MTilesLayer（瓦片倾斜，`type:'i3s'` / `'s3m'`）

同为瓦片倾斜，**选项一致**：`lightColor` `origin` `shadowMode` `offset` `rotation`；方法仅 `dispose()` `flyTo()` `getBoundingSphere()`；自有事件 `LOADED` `TRANSFORM_CHANGED`。

## USDLayer（`type:'usd'`）

```js
const usd = new Glodon.CIMCube.Layer.USDLayer({
  id: "usd", name: "USD 模型",
  url: "./data/model.usdz", origin: [116.39, 39.90, 0], scale: 1,
});
usd.on(Defines.LayerEventType.LOADED, () => usd.flyTo());  // flyTo 需 Model ready
```

- 方法：`getModel()` `getBoundingSphere()` `flyTo()`（内部等 `Model.readyEvent`）`addAnimation/addAllAnimations/getAllAnimation/getAnimationByName/getAnimationByIndex/removeAnimation/removeAllAnimation` `getUsdCustomAttributesByPath(usdPath)` `getAllUsdCustomAttributesByPath()` `dispose()`
- 自有事件：`LOADED`
