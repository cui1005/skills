# CIMCube 类目总索引（域 → 技能 → 入口文件）

> **手写导航**：本文件只负责「命名空间 → 技能 → 入口文件」的寻址。
> 逐类归属、类数与「有没有 API_DOC 页」在 `references/class-routing.md`（**自动生成物，勿手改**；由仓库侧类目登记表生成，随技能包分发）。
> 查「某模块有哪些真实类 / 该类归哪个技能」→ 直接读 `references/class-routing.md`。

## 一、命名空间总路由

| 命名空间 `Glodon.CIMCube.*` | 对应 Skill（主归属） | 入口文件 | 说明 |
|---|---|---|---|
| （根）`Glodon.CIMCube` / `CIMCubeSDKLoader` | **cimcube-setup** | `cimcube-setup/SKILL.md` | 主类 + 构造参数 + 资源路径 + dispose |
| `.Camera` | **cimcube-camera** | `cimcube-camera/SKILL.md` | 相机定位 / 飞行 / 姿态读写 / 截图 / 自转 |
| `.Scene`（含 3 类光源、GroundSkyBox、ReflectBox） | **cimcube-scene** | `cimcube-scene/SKILL.md` | 场景环境：天空盒 / 大气 / 云图 / 雾 / 阴影 / 后处理 |
| `.Scene.RealisticRendering`（8 子模块） | **cimcube-realistic-rendering** | `cimcube-realistic-rendering/SKILL.md` | 真实感总开关 + 8 子模块 + 3 类光源 |
| `.ScenePhysics`（根命名空间） | **cimcube-physics** | `cimcube-physics/SKILL.md` | 刚体 / 碰撞 / 约束 / 软体 |
| `.Globe` | **cimcube-globe** | `cimcube-globe/SKILL.md` | 地球椭球与地形叠加 |
| `.Layer.*` | **cimcube-layer** | `cimcube-layer/SKILL.md` | 图层体系：入口 `map.layerManager` + 各类图层 |
| `.Graphic.*` | **cimcube-graphic** | `cimcube-graphic/SKILL.md` | 18 种图元与标注 |
| `.Animation.*`（7 子命名空间） | **cimcube-animation** | `cimcube-animation/SKILL.md` | 41+ 类特效：漫游 / 飞线 / 粒子 / 天气 / 平面 / 点动态 / 模型 |
| `.Analysis.*` | **cimcube-analysis** | `cimcube-analysis/SKILL.md` | 空间分析（分 3 组：地形土方 / 可视规划 / 日照环境） |
| `.Simulation.*` | **cimcube-simulation** | `cimcube-simulation/SKILL.md` | 实时路径漫游 / 垂缆 / 绝缘网 |
| `.Utils.*` | **cimcube-utils** | `cimcube-utils/SKILL.md` | 工具集：测量 / 编辑 / 服务 / 转换 |
| `.Widgets.*` | **cimcube-widgets** | `cimcube-widgets/SKILL.md` | UI 组件（内部类与导出壳见 class-routing.md 注记） |
| `.Controls.*` | **cimcube-controls** | `cimcube-controls/SKILL.md` | 取点交互：`TakeCoordinates` / `PickEventHandler` |
| `.Global.Defines` / `.Global.Math` | **cimcube-global** | `cimcube-global/SKILL.md` | 命名空间容器（非可实例化 Class） |

## 二、结构约定（写某类代码前必读）

1. **类清单只有一个真相源（仓库侧）**：`class-routing.md` 是随包分发的**唯一类目视图**（域 → 技能 → 逐类 FQN + 类数 + 例外口径），由仓库侧类目登记表自动生成，**不要手改**；改类目请改仓库侧登记表后重跑生成器。
2. **读取顺序**：本索引定位命名空间 → 对应技能的 `SKILL.md` 路由表 → **只打开命中的那一个** `references/*.md`（勿整包加载）。
3. **域与技能不是 1:1**：`.Scene` 族拆成场景环境（cimcube-scene）/ 真实感渲染（cimcube-realistic-rendering）/ 物理（cimcube-physics）三个技能。
4. **命名空间前缀统一 `Glodon.CIMCube.*`**，不要 `import` 解构。
5. **跨技能例外**：同一个类只归 1 个主技能；例外（内部类、运行时扩展类、API_DOC 页名 ≠ 运行时导出名）统一登记在 `class-routing.md` 的「§三 例外口径」。

## 附：类数口径速查（同一数字可能被读成不同含义）

| 口径 | 含义 | 权威出处 |
|---|---|---|
| **计入 SDK 类计数** | `classes.list.html` Classes 下拉里的条目（命名空间入口、运行时扩展类除外） | `class-routing.md` 的「§一 类数」列（与 SDK classes.list.html 实测、发布清单声明值三向对账，见 [D3]） |
| **导出类** | 运行时真实挂在命名空间下、可直接 `new` 的类 | 各技能 `SKILL.md` 的「跨域约定」 |
| **命名空间条目** | 文档导航里的子命名空间名（`.Effect`、`.MeasureUtils` 等） | 不是类，不计入类数 |

> 凡引用类数请注明口径；**类数只在 `class-routing.md` 处定义一次**，正文里的数字是引用而非定义。
