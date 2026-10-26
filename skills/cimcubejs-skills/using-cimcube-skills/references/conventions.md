# 语言与代码约定

> 本文件对应 SKILL.md 路由表的「中文交互、`Glodon.CIMCube.*` 全前缀写法、单位约定（度/米/弧度）」行。

- 与用户交互默认使用**简体中文**。
- 所有代码示例优先使用 `Glodon.CIMCube.*` 完整命名空间，避免依赖 import 解构。
- 如果同时涉及 CesiumJS 原生 API 和 CIMCube 封装 API，**优先给出 CIMCube 封装 API 的写法**，并补充两者差异说明。
- 单位约定：`lon/lat` 为度、`height` 为米；相机 `heading/pitch/roll` 为**弧度**（只读）；`fov` 为度。
