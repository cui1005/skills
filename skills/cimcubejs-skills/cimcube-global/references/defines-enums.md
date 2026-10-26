# Global.Defines — 枚举常量全表（runtime 43 键 / 8 组）

> 本文件对应 SKILL.md 路由表的「43 个枚举键，按 8 组展开：图层/影像/地形、图元/几何、输入事件、绘制/编辑/测量/捕捉、动画/状态/效果、光照/环境、模型地形裁剪、视图/布局/其它」行。

> **使用原则**：APIDOC 里标注「可取值见 Defines」的参数，用 `DEF.XXX.YYY` 引用，**不要写死字符串/数字**。
> APIDOC 页面列出 **42** 个；runtime 实际 **43** 个（多 `Anchor`、`GmlVersion`；APIDOC 里的 `MouseEventType` **运行时不存在**，见 pitfalls.md「一、Defines 枚举避坑」第 3 条）。

## 1.1 图层 / 影像 / 地形（cimcube-layer 主用）

| 枚举 | 取值概览 | 说明 |
|---|---|---|
| `LayerType` | `GROUP/IMAGERY/TERRAIN/TILES3D/GEOJSON/KML/GML/GLTF/DIV/GRAPHIC/CLUSTER/INSTANCE/I3S/S3M/VECTORTILE/GSMM/TRANSFORMABLE/USD`（20，值为小写字符串，如 `TILES3D:"3dtiles"`） | 图层类型 |
| `TerrainLayerType` | `DEFAULT/ARCGIS_TERRAIN/CESIUM_TERRAIN/GEOSERVER_TERRAIN/SUPERMAP_TERRAIN` | 地形源类型 |
| `ClassificationType` | `TERRAIN/TILES3D/BOTH` | 贴分类对象 |
| `CollisionType` | `box/sphere/cylinder/cone/heightfield/convexHull/mesh` | 图层碰撞体 |
| `MoveType` | `YX/ZX/ZY` | 图层移动方式 |
| `ImageryLayerType` | `WMS/WMTS/TMS/XYZ/TDT_VEC/TDT_IMG/TDT_TER/TDT_CVA/TDT_CIA/TDT_CTA/BAIDU_IMG/BAIDU_VEC/GAODE_IMG/GAODE_VEC/GAODE_CIA/ARCGIS_IMAGE/ARCGIS_IMG/ARCGIS_GRAY/GOOGLE_IMG/TENCENT_IMG/GOOGLE_IMAGE/GOOGLE_ROAD/BAIDU/GAODE_IMAGE/GAODE_ROAD/TDT_IMAGE/TDT_ROAD/TENCENT_IMAGE/SUPERMAP_IMAGE/OFFLINE_IMG/OFFLINE_MIX/CUSTOM/DEBUG/IMAGE`（值为小写字符串，如 `TDT_IMG:"tdt_img"`） | 影像图层内置源（天地图/高德/百度/ArcGIS/Google/腾讯…） |
| `ImageryServiceUrl` | `google_image/arcgis_image/arcgis_gray/gaode_image_cloud/gaode_image/gaode_road/tianditu_image/tencent_image` | 常用底图服务 key |

> 影像源完整用法（含 SDK 内置 provider/默认 tk、天地图/高德等内置底图无需填 `url`）见 **cimcube-layer/references/imagery-terrain.md**（内置底图用法与示例）。

## 1.2 图元 / 几何（cimcube-graphic 主用）

| 枚举 | 取值概览 | 说明 |
|---|---|---|
| `GeometryType` | `NONE/POINT/POLYLINE/POLYGON/CURVE/RECTANGLE/ELLIPSE/CIRCLE/PLANE/CORRIDOR/POLYLINE_VOLUME/CYLINDER/ELLIPSOID/WALL/BOX` | 几何类型 |
| `GraphicType` | `POINT/BILLBOARD/LABEL/MODEL/POLYLINE/CURVE/POLYGON/RECTANGLE/ELLIPSE/CIRCLE/PLANE/CORRIDOR/POLYLINE_VOLUME/CYLINDER/ELLIPSOID/WALL/BOX`（值为小写字符串） | 图元类型（图元类后缀与枚举名大多一致） |
| `LineType` | `SOLID/NORMAL/GLOW/DASHES/TRACK/ANT/ARROW/DASH` | 动态线样式 |
| `PolylineType` | `SOLID/GLOW/DASHES/TRACK/ANT/ARROW` | 线类型 |
| `PointType` | `VECTOR/ICON/DYNAMIC/HEATMAP/MODEL` | 点类型 |
| `TextPlacement` | `POINT/LINE` | 文本位置 |

## 1.3 输入事件

| 枚举 | 取值概览 | 说明 |
|---|---|---|
| `ScreenSpaceEventType` | `LEFT_DOWN:0, LEFT_UP:1, LEFT_CLICK:2, LEFT_DOUBLE_CLICK:3, RIGHT_DOWN:5…, MIDDLE_CLICK:12, MOUSE_MOVE:15, WHEEL:16, PINCH_START:17…, RIGHT_DOUBLE_CLICK:20, MIDDLE_DOUBLE_CLICK:21, MOUSE_HOVER:22`（**number**） | 屏幕事件类型 |
| `LayerEventType` | 鼠标事件（LEFT_DOWN…WHEEL/PINCH/MOUSE_LEAVE/ENTITY_CLICK/MOUSE_SNAP/ENTITY_MOUSE_ENTER/ENTITY_MOUSE_LEAVE）+ 图层事件（GRAPHIC_DELETED/GRAPHIC_ADDED/VISIBLE/LOADED/TRANSFORM_CHANGED/TILELOADED/PRECLUSTER/POSTCLUSTER/HIGHLIGHT/DESELECTION） | 图层注册事件（含鼠标转发） |
| `KeyboardEventModifier` | `SHIFT/CTRL/ALT` | 键盘组合 |

## 1.4 绘制 / 编辑 / 测量 / 捕捉

| 枚举 | 取值概览 | 说明 |
|---|---|---|
| `DrawEditMode` | `NONE/DRAW/EDIT` | 绘制/编辑工具阶段 |
| `MapActionType` | `ADD/MOVE/DELETE/FINISH/CHANGED` | 地图数据操作类型 |
| `MeasureMode` | `None/Coordinate/Angle/Azimuth/Space_Distance/Ground_Distance/Triangulation/Space_Area/Ground_Area/Project_Area/Slope/Model_Distance/Volume/Project_Distance` | 测量模式（cimcube-utils 用） |
| `MeasureShowResult` | `ALWAYS/NEVER/TRIGGER_HOVER/TRIGGER_CLICK` | 测量结果显示时机 |
| `DatumMode` | `CustomPlane/LowestPointPlane/HighestPointPlane/MiddlePointPlane` | 测量基准面 |
| `AreaUnit` | `S_MM/S_CM/S_M/S_KM/S_HA/S_MU` | 面积单位 |
| `LengthUnit` | `MM/CM/M/KM` | 长度单位 |
| `SnapType` | `ENDPOINT/MIDPOINT/LINE/FACE/INTERSECTION` | 捕捉类型 |
| `SnapTargetType` | `MODEL_3D/VECTOR_2D/BOTH` | 捕捉目标 |
| `TriangulationType` | `DEFAULT/HORIZONTAL/VERTICAL` | 三角测量方式 |

## 1.5 动画 / 状态 / 效果（cimcube-animation 用）

| 枚举 | 取值概览 | 说明 |
|---|---|---|
| `AnimationStatus` | `READY:1, LOADING:2, ANIMATION:3, END:4`（**number**） | 特效动画状态 |
| `FlyStatus` | `NOTHING/FLY/FLYING/COMPLETE` | 飞行状态 |
| `FeatureEffectMode` | `HIGHLIGHT/FLASH/ANIMATION/BREATH` | 要素特效 |
| `FeatureRenderMode` | `ENTITY/PRIMITIVE/BUFFER` | 要素渲染方式 |
| `DynamicHaloMode` | `UP/DOWN/UPTODOWN/DOWNTOUP` | 动态光晕方向 |
| `InterpolationAlgorithm` | `Default/LinearApproximation/LagrangePolynomialApproximation/HermitePolynomialApproximation` | 插值算法 |

## 1.6 光照 / 环境（cimcube-scene / cimcube-realistic-rendering 用）

| 枚举 | 取值概览 | 说明 |
|---|---|---|
| `ShadowMode` | `DISABLED/ENABLED/CAST_ONLY/RECEIVE_ONLY` | 阴影模式 |
| `SkyBoxMode` | `FIRECLOUD/DAWN/DUSK/MIDNIGHT/SUMMER/BLUESKY/SUNSET/LATERNIGHT/SUNNYSUN/CLOUDY/FACTORY/BUILDING/CITY/HDR_CLOUDY/HDR_FACTORY/HDR_URBAN/DEFAULT` | 内置天空盒预设 |

## 1.7 模型 / 地形 裁剪（cimcube-utils 用）

| 枚举 | 取值概览 | 说明 |
|---|---|---|
| `ModelClipMode` | `VERTICAL/HORIZONTAL` | 模型剖切模式 |
| `ModelCutMode` | `INNER/OUTER` | 模型裁切模式 |
| `TerrainClipMode` | `STENCIL/PRECISE/PERFORMANCE` | 地形开挖模式 |

## 1.8 视图 / 布局 / 其它

| 枚举 | 取值概览 | 说明 |
|---|---|---|
| `StandardView` | `Top/Bottom/Front/Back/Right/Left/TopFrontRight/…/BackLeft`（24 组合） | 标准视角 |
| `ViewPortModeType` | `VERTICAL/HORIZONTAL` | 分屏方向 |
| `Anchor` | `leftTop/leftCenter/leftBottom/centerTop/center/centerBottom/rightTop/rightCenter/rightBottom`（9，值为小写 camelCase 字符串） | 控件/气泡锚点（runtime 有，APIDOC 未收录） |
| `GeocoderMode` | `NOOFFSET/OFFSET` | 地名定位模式 |
| `BodyType` | `static/dynamic/kinematic` | 物理刚体类型 |
| `GmlVersion` | —（runtime 有，APIDOC 未收录） | GML 版本 |
