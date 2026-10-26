# 两种 SDK 加载方式

> 本文件对应 SKILL.md 路由表的「静态 `<script>` 引入、`CIMCubeSDKLoader` CDN 按版本加载、资源路径与容器 DOM」行。

> ⚠️ **引擎根默认 `v3.**.0/dist/`**（SDK 解压根，含 `cimcube/cimcube.js` 与 `Resource/` 的那一层），本文件示例均以此为前缀，**基准是 HTML 所在目录**。
> **发布包不含 SDK**：把 SDK 解压成工作区下的 `v3.**.0/dist/` 即可直接照抄；放在别处时把 `v3.**.0/dist` 前缀整体替换（路径基准是 HTML 所在目录，该目录须同时含 `cimcube/cimcube.js` 与 `Resource/`）。
> 页面还须走静态服务器（`file://` 下 Resource 的 XHR 被 CORS 拦）。

## 方式 A：静态 script 标签（推荐，简单项目）

```html
<!DOCTYPE html>
<html>
<head>
  <!-- 1a. Cesium 内核基础样式：必须引入！缺它 canvas 不会撑满容器（停留默认 300x150） -->
  <link rel="stylesheet" href="v3.55.0/dist/cimcube/Widgets/widgets.css">
  <!-- 1b. CIMCube Widgets CSS（如果使用 UI 组件） -->
  <link rel="stylesheet" href="v3.55.0/dist/Resource/css/Widgets.css">

  <!-- 2. 引入 SDK 主 JS 文件（SDK 包内路径为 cimcube/cimcube.js） -->
  <script src="v3.55.0/dist/cimcube/cimcube.js"></script>
</head>
<body>
  <div id="cimcubeContainer" style="width:100%;height:100vh"></div>

  <script>
    // SDK 加载完成后，Glodon 全局可用
    const map = new Glodon.CIMCube({
      id: "cimcubeContainer",
      gisResourceUrl: "v3.55.0/dist/",   // ★ Resource 的父目录（SDK 内部会拼 "Resource/..."）
      // lon/lat/height：经度纬度（度）+ 高度（米）；heading/pitch/roll 为弧度（-π/4 = 45°俯视）
      center: { lon: 116.397428, lat: 39.90923, height: 3000, pitch: -Math.PI / 4 },
    });
  </script>
</body>
</html>
```

## 方式 B：CIMCubeSDKLoader 按版本号从 CDN 异步加载（不本地部署 SDK 时）

```js
// CIMCubeSDKLoader 是【CDN 版本加载器】：new CIMCubeSDKLoader(version).load()
// 会注入 https://cimcube.glodon.com/websdk/v<version>/cimcube/cimcube.js（失败重试最多 10 次）
// 不接受 libraryPath/resourcePath 参数；本地离线部署请直接用方式 A 的 <script> 标签
const loader = new Glodon.CIMCube.CIMCubeSDKLoader();          // 不传版本号则加载 latest；需锁定版本时传入与 SDK 解压根目录一致的版本号（如 "3.57.0"）

loader.load().then(() => {
  // CDN SDK 注入完成，Glodon 全局可用
  const map = new Glodon.CIMCube({
    id: "cimcubeContainer",
    gisResourceUrl: "v3.55.0/dist/",  // 静态资源仍指向本地 Resource 的父目录
  });
});
```
