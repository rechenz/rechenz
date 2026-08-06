# UE5 C++ 闯关手册

> 银狼出品 · 2026-08-06
> 目标：从零到能独当一面的 UE 程序。主线 24 题，全通 = 一个能拿出手的 demo + 90% 日常知识面。

## 开局设定

- 引擎：UE 5.8（你机器上那个）
- 语言：**C++ 为主**。蓝图只准用在美术向的东西上（动画蓝图、材质、Niagara 编辑器侧配置）
- 主线：从零做一个「地牢生存」小游戏，每关往里塞功能。俯视角或第三人称随你
- 节奏：每题 1~3 天，按自己状态来
- 卡住先查文档，再问 AI，最后问我也行——别闷头死磕，但也不许每题都直接抄答案
- 验收标准是硬性的：跑通了、看得到效果，才算过

---

## 第一关：地基 — 对象与生命周期

### 题 1 · 组件拼装 + 生命周期日志
做一个 `AActor`，挂 2~3 个组件（场景组件 + 静态网格 + 音频），打印每个组件的 BeginPlay / EndPlay 调用顺序。

- 知识点：Actor/Component 生命周期、初始化顺序、UPROPERTY 反射
- 验收：日志里能看出销毁顺序和创建顺序相反（先创建的后销毁），组件初始化在 Actor 的 BeginPlay 之前

### 题 2 · 委托与事件
给 Actor 加一个动态多播委托 `OnSomethingHappened`，另一个类绑定它，触发后收到回调。

- 知识点：`DECLARE_DYNAMIC_MULTICAST_DELEGATE`、动态/静态委托区别、`BindDynamic` / `BindUObject`
- 验收：事件触发时绑定方收到回调；解绑后不再收到；蓝图侧也能绑定这个委托

### 题 3 · 移动平台
做一个沿路径来回移动的平台，细节面板可配：移动距离、速度、端点停留时间。

- 知识点：Tick、`FMath` 插值、移动组件配置、细节面板 UPROPERTY 分类
- 验收：角色站在平台上会被带着走，到达端点停够配置的时间再折返

---

## 第二关：操作 — 输入与角色

### 题 4 · Enhanced Input 角色
自定义 `ACharacter` 子类：移动、跑步、跳跃、冲刺，用 Enhanced Input（IA + IM），键鼠手柄都支持。

- 知识点：`UInputAction` / `UInputMappingContext`、`UEnhancedInputComponent`、输入优先级（UI 挡玩法）
- 验收：角色能走能跑能跳能冲刺；打开暂停菜单时，玩法输入被屏蔽但 UI 能操作

### 题 5 · GameMode 三件套
自定义 GameMode / PlayerController / PlayerState：死亡后重生、分数记录、`ShowScore` 控制台命令打印分数。

- 知识点：Gameplay 框架各对象职责、生命周期、控制台命令（`Exec`）
- 验收：死亡→重生循环正常，分数跨重生保留（存在 PlayerState 里，不在 Pawn 里）

### 题 6 · GameInstance 跨关卡
用 GameInstance 存跨关卡数据：血量、分数、已解锁关卡，切关卡不丢。

- 知识点：`UGameInstance` 生命周期、跨关卡数据传递、和 PlayerState 的分工
- 验收：从关卡 A 切到 B，血量分数都还在；GameInstance 在整个游戏进程只存在一份

---

## 第三关：数据 — 让游戏可配置

### 题 7 · DataTable 敌人配置
DataTable 存敌人数据（血量/伤害/速度/掉落物），运行时读表生成敌人。

- 知识点：`UDataTable`、`FTableRowBase`、`FDataTableRowHandle` 软引用
- 验收：改表不改代码，敌人属性就变；表里加一行新敌人，游戏里就能刷出来

### 题 8 · GameplayTags 伤害克制
敌人打标签（火/冰/机械），技能带标签，命中时按标签算伤害加成。

- 知识点：`FGameplayTag`、`FGameplayTagContainer` 查询、标签层级（`Damage.Type.Fire`）
- 验收：对弱火敌人伤害 ×2，对抗火 ×0.5，纯配置驱动，不加 if-else 链

### 题 9 · SaveGame 存档
存档/读档：位置、血量、背包（TArray 结构体）、时间戳。

- 知识点：`USaveGame`、序列化、槽位管理、`UGameplayStatics` 存档 API
- 验收：退出重进游戏，位置和背包都在；存档文件能在 Save 目录看到，损坏时能兜底

---

## 第四关：世界交互 — 碰撞与物理

### 题 10 · 射击系统
LineTrace 射击：准星、命中判定、命中特效、伤害回调、弹孔。

- 知识点：碰撞通道（Object/Trace Channel）、`FCollisionQueryParams`、Trace 与 Overlap 区别
- 验收：打中敌人掉血，打中墙出弹孔 + 粒子 + 声音，有弹道 traer；穿透/阻挡可配置

### 题 11 · Overlap 拾取
可拾取物（血包/金币），Overlap 触发拾取，带缩放淡出反馈。

- 知识点：Overlap 事件、碰撞预设（Preset）、`GenerateOverlapEvents`、碰撞响应矩阵
- 验收：走过自动拾取，有反馈动画，金币计数同步到 UI

### 题 12 · 物理交互
可推动的箱子 + 手雷投掷物。

- 知识点：物理材质、冲量施加、`UProjectileMovementComponent`、碰撞响应（Ignore/Block/Overlap）
- 验收：箱子推得动但推不飞；手雷会弹跳、延时爆炸、造成范围伤害，抛物线受重力影响

---

## 第五关：AI — 让敌人活起来

### 题 13 · BehaviorTree 巡逻追敌
AIController + 黑板：巡逻路径 → 发现玩家 → 追击 → 近战攻击 → 丢失目标回巡逻。

- 知识点：`UBehaviorTree`、`UBlackboardComponent`、选择/顺序/任务节点、装饰器（Condition）、自定义 BTTask
- 验收：视野内追击，出视野回巡逻；攻击有前摇和伤害窗口；Blackboard 值能在调试面板实时看

### 题 14 · 感知系统
用 `UAIPerceptionComponent` 做视野 + 听觉：开枪声会把远处敌人引过来查探。

- 知识点：AIPerception、感官配置（视线/听觉）、刺激响应、和 BT 的数据桥接
- 验收：背对玩家时看不到；听到枪声会走到声源位置查探，找不到才回巡逻

---

## 第六关：表现 — 动画、特效、声音

### 题 15 · AnimInstance 驱动
动画蓝图 + C++ AnimInstance：按速度切 Idle/Walk/Run，跳跃状态，受击蒙太奇。

- 知识点：`UAnimInstance`、动画蓝图变量、`UAnimMontage`、`Montage_Play`
- 验收：移动动画和速度匹配；跳跃有起跳/落地；受击播蒙太奇但不打断移动逻辑

### 题 16 · 动画通知
攻击动画用 AnimNotify 开伤害判定窗口，AnimNotifyState 控制持续时间。

- 知识点：`AnimNotify` / `AnimNotifyState`、C++ 通知类、判定窗口设计
- 验收：挥刀动画中段才出伤害，不是起手瞬间；挥空也有反馈；通知能带参数（伤害倍率）

### 题 17 · 特效音效集成
命中粒子、死亡爆炸、脚步声（动画通知触发）、背景音乐（音频组件 + 衰减）。

- 知识点：Niagara（C++ 侧 SpawnSystemAtLocation）、`UAudioComponent`、Attenuation、MetaSound（选一）
- 验收：每个关键动作都有视听反馈；音频有距离衰减；音效/音乐分开控制音量

---

## 第七关：界面 — UMG

### 题 18 · HUD 全家桶
血条（动态绑定）、准星、伤害飘字、击杀计数、暂停菜单（继续/退出/音量）。

- 知识点：`UUserWidget`、控件绑定（BindWidget）、UMG 事件、C++ 与控件蓝图混合
- 验收：血量变化 UI 实时同步；飘字有随机偏移；暂停菜单能屏蔽玩法输入并控制音量

### 题 19 · 接口交互
按 E 与场景物体交互：门（开关）、NPC（对话）、宝箱（打开）。

- 知识点：`UINTERFACE` 接口、面向接口编程、交互提示 UI（Trace 检测）
- 验收：准星对准交互物显示提示，离开消失；按 E 执行各自逻辑；新增交互物不用改玩家代码

---

## 第八关：架构 — 项目级能力

### 题 20 · Subsystem 事件总线
做一个 `UWorldSubsystem` 全局事件总线：击杀、拾取、关卡进度，UI 和玩法都从它取数据。

- 知识点：Subsystem 生命周期（Engine/GameInstance/World/Player）、解耦
- 验收：任何系统能发事件，UI 只订阅不直接依赖玩法类；Subsystem 在关卡切换时行为正确

### 题 21 · 对象池
子弹/粒子/尸体做对象池，复用而不是反复 Spawn/Destroy。

- 知识点：对象池模式、Actor 复用（SetActorHiddenInGame + 状态重置）、性能意识
- 验收：连续射击 5 分钟，Spawn/Destroy 调用次数为 0（打印统计验证）；池满时有扩容策略

### 题 22 · 异步加载
用 `FStreamableManager` 异步加载武器模型和音效，加载完再换装，不卡主线程。

- 知识点：异步资产加载、`FStreamableHandle`、软引用 `TSoftObjectPtr` / `TSoftClassPtr`
- 验收：换武器不卡顿；加载中有 UI 反馈；取消加载能正确释放句柄

### 题 23 · 编辑器工具（进阶）
用 Editor Utility Widget 或 C++ 编辑器模块做一个批量工具：一键生成 100 个带碰撞的随机地牢房间。

- 知识点：编辑器扩展、EUW、事务支持（可撤销）、模块结构（Runtime/Editor 分离）
- 验收：点一个按钮生成 100 个房间，Ctrl+Z 能整体撤销；工具不污染运行时代码

### 题 24 · 插件化（进阶）
把对象池 + 事件总线抽成一个独立插件（带模块），复制到空项目里也能编译运行。

- 知识点：插件结构、Module 依赖、`.uplugin`、公开 API 设计
- 验收：新项目启用插件后能直接调用；API 干净，不依赖你的游戏类型

---

## 隐藏关卡（区域赛后再开）

- **网络多人**：属性复制 + RPC（Server/Client/Multicast）做联机打怪
- **GAS**：AttributeSet + GameplayEffect + AbilityTask 做完整技能系统
- **Niagara 高级**：GPU 粒子、数据接口
- **渲染**：自定义 Shader、材质参数集合、后期处理材质
- **线程**：FRunnable / AsyncTask / ParallelFor 处理重计算

---

## 通关标准

主线 24 题全通 = 一个能拿出手的地牢生存 demo + 覆盖 UE 程序 90% 的日常知识面。

到时候这就是你投老米实习的敲门砖——比简历上写「熟悉 UE5」有用一百倍。毕竟，重要的不是数值，是操作。

---

## 📦 素材说明（UE 5.8 实测）

> ⚠️ **5.8 已移除 Starter Content 内容包**（FeaturePacks 里没有 StarterContent.upack），编辑器里搜不到是正常的。

### 素材来源（引擎自带，零成本）

| 素材 | 位置 | 用途 |
|------|------|------|
| BasicShapes（方块/球体/圆锥/圆柱） | `Engine\Content\BasicShapes` → 项目 `Content\BasicShapes` | 所有占位：敌人、子弹、手雷、房间 |
| Quinn/Manny 角色 + 全套动画 | `Templates\TemplateResources\High\Characters\Content\Mannequins` → 项目 `Content\Characters\Mannequins` | 玩家/敌人骨骼网格、Walk/Jog/Jump/Attack/HitReact/Death/Pistol/Rifle 动画、ABP_Unarmed 动画蓝图 |
| NS_Damage 伤害特效 | `Templates\TP_ThirdPerson\Content\Variant_Combat\VFX` → 项目 `Content\VFX` | 命中/受击粒子 |

### 已拷入 `F:\project\IWannaBecomeUEMaster\Content\`

- `BasicShapes/` — 7 个基础几何体 + 材质
- `Characters/Mannequins/` — 128 文件 / 125MB，路径 `/Game/Characters/Mannequins/...`（与模板引用一致）
- `VFX/NS_Damage.uasset`

### 注意

- 手动拷 uasset 后需在 Content Browser 右键 Refresh 或重启编辑器
- 拷贝时保持目录结构，资产内部引用路径才能对上
- BGM 是唯一需要外部素材的（题 17），用 CC0 免费音频或先不放
