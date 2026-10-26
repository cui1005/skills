# Globe 常见模式（只用真实成员）

> 本文件对应 SKILL.md 路由表的「可运行模式：地表挡与不挡 / 夜景光照 / 季节底色」行。

> 成员语义见 `members.md`；分工与排查见 `pitfalls.md`。

## 模式 A：地表挡与不挡

```js
// 施工/贴地正确性优先 → 开启深度测试
function groundMode(on) {
  map.globe.depthTestAgainstTerrain = on;
}
// 需要透视看地下 → 半透明 + 地下颜色
function peekUnderground(on) {
  const G = map.globe;
  G.translucency.enabled = on; // 注意：translucency 是对象，经 .enabled 开关
  G.undergroundColor = Glodon.CIMCube.Cesium.Color.fromCssColorString(
    on ? "rgba(120,180,255,0.35)" : "rgba(120,180,255,0.02)"
  );
}
```

> 注意：这只是 Globe 能做的"球壳"层面的调整。相机能否下钻、是否允许进入地下等属 Camera 能力（见 **cimcube-camera**）；真正的"地下模型"本身还要加载对应图层（见 **cimcube-layer**）。

## 模式 B：夜景 / 光照主题（仅 Globe 层）

```js
function setGlobeLook(night) {
  const G = map.globe;
  if (night) {
    G.enableLighting = false; // 关闭太阳光照，走夜间氛围（大气/灯光由 Scene 提供）
  } else {
    G.enableLighting = true;
  }
}
```

> 大气、天空盒、云、Bloom 等夜间氛围请走 `map.scene`（见 **cimcube-scene**）/ `map.scene.realisticRendering.*`（见 **cimcube-realistic-rendering**），不要试图在 Globe 上设置。

## 模式 C：用真实成员做季节底色

```js
function setSeason(base) {
  // base 传 CSS 颜色字符串，如 "#3f8a3e"（夏）、"#e8eef7"（冬）
  map.globe.baseColor = Glodon.CIMCube.Cesium.Color.fromCssColorString(base);
}
```
