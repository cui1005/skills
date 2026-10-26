# 典型任务 → Skill 选择矩阵

> 本文件对应 SKILL.md 路由表的《「用户任务 → 该用哪个技能」的 14 行选择矩阵》行。

> 定位方式：先在下表找最接近的用户任务 → 取「首选 Skill」→ 读该技能 `SKILL.md` 路由表 → 打开对应的 `references/` 文件。
> 各技能的能力与坑位见 `skill-capabilities.md`；激活关键词见 `SKILL.md` 速查表。

| 用户任务 | 首选 Skill | 辅助 Skill |
|---|---|---|
| "帮我搭一个 CIMCube 3D 地球页面" | cimcube-setup | cimcube-camera, cimcube-scene |
| "加载一个 S3M 倾斜摄影图层" | cimcube-layer | cimcube-camera（飞到图层范围） |
| "在地图上画几个带标注的广告牌" | cimcube-graphic | cimcube-layer（GraphicLayer） |
| "做一条从北京飞到上海的飞线动画" | cimcube-animation（cimcube-animation/references/LineEffect.md · FlyLines） | cimcube-camera, cimcube-global |
| "计算这块区域的填挖方量" | cimcube-analysis（CutFill，cimcube-analysis/references/CutFill.md） | cimcube-controls（取点） |
| "给场景加雨天效果 + 体积云" | cimcube-animation（cimcube-animation/references/WeatherEffect.md） + cimcube-realistic-rendering（RR.volumetricClouds） | — |
| "打开真实感渲染 / 调辉光·体积云·大气散射" | cimcube-realistic-rendering | cimcube-scene（阴影/后处理） |
| "做个物体掉落 / 碰撞 / 铰链门" | cimcube-physics | cimcube-layer（碰撞图层） |
| "在页面上加指南针和状态栏" | cimcube-widgets（cimcube-widgets/references/StatusBar.md） | — |
| "测量两点距离 / 多边形面积" | cimcube-utils（cimcube-utils/references/Measure.md · Measure） | — |
| "把屏幕坐标转经纬度" | cimcube-utils（cimcube-utils/references/CoordinateTransform.md） | cimcube-global |
| "做视域分析/基站选址" | cimcube-analysis（ViewShedAnalysis，cimcube-analysis/references/ViewShedAnalysis.md） | cimcube-controls（取点） |
| "画红线/多边形并编辑" | cimcube-controls（TakeCoordinates 取点）+ cimcube-graphic（PolygonGraphic 标绘） | cimcube-utils（cimcube-utils/references/Snap.md · 顶点吸附 Snap） |
| "加图层控制树面板" | cimcube-setup（初始化参数 `params: { layerControl: true }`） | cimcube-layer（面板操作的是 `map.layerManager`） |
| "美化场景 / 视觉提质（天空·大气·云·水·树·阴影·AO·真实感渲染）" | cimcube-scene（天空/大气/云/水/阴影/AO）+ cimcube-realistic-rendering（真实感渲染） | 树木：`cimcube-layer`（树木为图层时）/ `cimcube-animation`（`Glodon.CIMCube.Animation.PointEffect.InstanceBillboard` 实例化看板） |
