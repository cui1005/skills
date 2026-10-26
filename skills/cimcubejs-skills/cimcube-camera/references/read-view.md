# 读取当前视角 · 截图

入口 `map.camera`。

## `getCameraInfo(includeCameraCopy=false)`

返回相机当前状态（同步、无副作用）：

```js
const info = map.camera.getCameraInfo();
// info = {
//   lon: 116.4074, lat: 39.9042, height: 3000,   // 相机位置（度/度/米）
//   x: …, y: …, z: …,                            // positionWC 的 ECEF 分量（米）
//   heading: 0, pitch: -0.785…, roll: 0,         // 姿态（弧度）
// }
```

- `x/y/z` 是 **ECEF** 坐标，仅作展示/存储，别当经纬度用。
- 传 `true`：额外生成 `cameraCopy` 字符串——**可直接复制执行的恢复视角代码**（内部是 `JSON.stringify` 出的 `map.camera.flyTo({target:{height,lat,lon}, orientation:{heading,pitch,roll}})`）。适合做"书签/分享当前视角"功能：

```js
const { cameraCopy } = map.camera.getCameraInfo(true);
localStorage.setItem("view", cameraCopy);        // 存
eval(localStorage.getItem("view"));              // 恢复（或自己 JSON.parse 后调 flyTo）
```

## `getCurrentViewInfo()`

"当前视角拾取"信息（比 getCameraInfo 多一个屏幕中心落点）：

```js
const v = map.camera.getCurrentViewInfo();
// v = {
//   orientation: {heading, pitch, roll},         // 弧度
//   location:    {lon, lat, height},             // 相机经纬高（度/度/米）
//   positionWC:  {x, y, z},                      // ECEF
//   center?:     {lon, lat, height},             // 屏幕中心射线与地球椭球交点（仅命中时存在；
//                                                //   height 复用相机高度；不含地形/模型拾取）
// }
```

## `print()`

`console.log` 两行输出：`lon, lat, height, heading, pitch, roll` 与 `x, y, z`（等价 `getCameraInfo(true)` 的前两个日志），调试用。

## `getCaptureImg()` — 截图

```
const snapImg = map.camera.getCaptureImg();   // 同步！返回对象；不是 Promise
// snapImg = { image: "data:image/png;base64,…", width, height }
```

**前置条件（运行时强校验）**：画布必须开启 `preserveDrawingBuffer`，否则方法打印
`preserveDrawingBuffer需要指定为true.` 并返回 `undefined`。
开启方式：构造 CIMCube 时经 `params`（原样透传给渲染内核）注入 WebGL 上下文选项：

```js
const map = new Glodon.CIMCube({
  id: "cimcubeContainer",
  gisResourceUrl: "./dist/",
  params: {
    contextOptions: { webgl: { preserveDrawingBuffer: true } }, // ★ 截图前提
  },
});
```

> ⚠️ `preserveDrawingBuffer` 只能在 WebGL 上下文创建时设定，**运行中无法开启**；忘了开就要重建实例。
> 该开关常驻会损失一部分渲染性能，截图需求不频繁时量力开启。

用法：

```js
const snapImg = map.camera.getCaptureImg();
if (snapImg) {
  const a = document.createElement("a");
  a.href = snapImg.image;                 // dataURL
  a.download = `view-${Date.now()}.png`;
  a.click();                           // 或直接 <img src={snapImg.image}>
  console.log(snapImg.width, snapImg.height);
}
```

> 与注意：`getCaptureImg()` **无参数**、**同步返回** `{image,width,height}`（`width/height` 为画布像素）；不返回 Promise、不支持自定义尺寸/格式。
