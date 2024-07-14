# GAS文档
我对于 Unreal Engine 5 的 Gameplay Ability System 插件（GAS）的理解，以及一个简单的多人游戏样本项目的介绍。请注意，这并非官方提供的文档，并且此项目及本人均未得到 Epic Games 的授权或关联。因此，我无法确保所提供的信息完全准确无误。

这份文档的目标是解释 GAS 中的主要概念和类，并基于我使用它的经验提供一些额外的见解。在社区用户中，关于 GAS 存在大量的“部落知识”（即非正式传播的经验和技巧），我打算在这里分享我所有的相关知识。

本示例项目与文档已更新至 Unreal Engine 5.3 版本。尽管我们为 Unreal Engine 的旧版本保留了文档的分支，但这些分支不再接受维护与更新，因此可能包含错误或信息已经过时。请务必选用与你当前使用引擎版本相匹配的文档分支进行查阅。

[GASShooter](https://github.com/tranek/GASShooter) 作为配套的示例项目，深入演示了如何在多人 FPS（第一人称射击）或 TPS（第三人称射击）游戏场景下，利用 GAS 实现更为复杂的机制与功能

最为完整和权威的文档资源，永远是插件本身的源代码。

## 目录

## 1.GameplayAbilitySystem 插件简介
引用自[官方文档](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html)：
>**Gameplay技能系统** 是一个高度灵活的框架，可用于构建你可能会在RPG或MOBA游戏中看到的技能和属性类型。你可以构建可供游戏中的角色使用的动作或被动技能，使这些动作导致各种属性累积或损耗的状态效果，实现约束这些动作使用的"冷却"计时器或资源消耗，更改技能等级及每个技能等级的技能效果，激活粒子或音效，等等。简单来说，此系统可帮助你在任何现代RPG或MOBA游戏中设计、实现及高效关联各种游戏中的技能，既包括跳跃等简单技能，也包括你喜欢的角色的复杂技能集。

GameplayAbilitySystem（GAS）插件由 Epic Games 开发，并随 Unreal Engine 一同发布。这一系统已在多款 AAA 商业游戏中经过实战考验，包括《Paragon》和《Fortnite》等知名作品，证明了其稳定性和高效性。

插件提供的即用方案针对单人及多人游戏的全面解决方案：
* 实现基于等级的角色能力和技能，附带可选的成本与冷却时间。([GameplayAbilities](#concepts-ga))
* 操纵属于游戏对象的数值`Attributes`。([Attributes](#concepts-a))
* 对游戏对象施加状态效果。([GameplayEffects](#concepts-ge))
* 向 Actors 应用`游戏标签（GameplayTags）`。([GameplayTags](#concepts-gt))
* 生成视觉或声音效果。([GameplayCues](#concepts-gc))
* 复制上述所有内容。

在多人游戏中，GAS 提供对[客服端预测](#concepts-p)的支持：
* 技能激活。
* 播放动画蒙太奇。
* 对`属性（Attributes）`的更改。
* 应用`游戏标签（GameplayTags）`。
* 生成`GameplayCues`。
* 通过连接到`CharacterMovementComponent`的`RootMotionSource`函数进行移动。

**GAS 必须在 C++**中设置，但设计师可以在蓝图中创建 `GameplayAbilities` 和 `GameplayEffects`。

GAS 的当前问题：
* `GameplayEffect`延迟协调（这导致了高延迟的玩家相较于低延迟的玩家，在使用冷却时间较短的能力时，能够触发的频率较低，进而影响了游戏的公平性和玩家体验）。
* 无法预测`GameplayEffects`的删除。但是，我们可以预测添加带有反向效果的`GameplayEffects`，从而有效地消除它们。这并不总是适当或可行的，并且仍然是一个问题。
* 缺少样板模板、多人游戏示例和文档。希望这会有所帮助！

**[⬆ 目录](#table-of-contents)**

## 2.示例项目
随附的文档中包含了一个多人第三人称射击游戏的示例项目，该项目针对的是初次使用GameplayAbilitySystem插件但对Unreal Engine并不陌生的人群。用户应当已经了解C++、蓝图（Blueprints）、用户界面管理器（UMG）、复制机制（Replication），以及Unreal Engine中的其他中级主题。这个项目提供了如何设置一个基本的、适用于多人游戏的第三人称射击项目示例，在该项目中，`AbilitySystemComponent`（简称`ASC`）被放置在`PlayerState`类上以处理玩家或AI控制的英雄角色的能力系统，而`ASC`则被放置在`Character`类上以管理AI控制的小兵角色的能力系统。

>译者注：这涉及到GAS的一个概念，比如我创建一个了`ALunaCharacterBase`派生`LunaCharacter`类和`LunaEnemyBase`类，关于`OwnerActor`和`AvatarActor`。对于玩家来说，`OwnerActor`是`LunaPlayerState`类，`AvatarActor`是`LunaCharacter`类。`AbilitySystemComponent`和`AttributeSet`初始化在`PlayerState`类。对于AI来说，非玩家控制的AI，其`OwnerActor`和`AvatarActor`都是`LunaEnemyBase`，`AbilitySystemComponent`和`AttributeSet`初始化在`LunaEnemyBase`。

|  类型  | Owner Actor | Avatar Actor |
|:------:|:-----------:|:------------:|
|   AI   |    Pawn     |     Pawn     |
| Player | PlayerState |     Pawn     |

----------------------------------------

目的是使这个项目保持简单，同时展示Gameplay Ability System（GAS）的基础，并通过注释详尽的代码演示一些常见的能力请求。由于项目主要面向初学者，因此它没有涉及诸如[预测发射物](#concepts-p-spawn)之类的高级主题。

演示的概念：
* `PlayerState` 与 `Character` 上的`ASC`。
* 复制`Attributes`（简称AS）。
* 复制动画蒙太奇。
* `GameplayTags`。
* 在`GameplayAbilities`内部和外部应用和删除`GameplayEffects`。
* 应用受护甲减伤后的伤害来改变角色的生命值。
* `GameplayEffectExecutionCalculations`(游戏玩法效果执行计算)。
* 眩晕效果。
* 死亡和重生。
* 从服务器上的能力生成 Actors（发射物）。
* 通过瞄准射击和冲刺动作，预判性地改变本地玩家的速度。
* 持续消耗体力进行冲刺。
* 使用法力来施展能力。
* 被动技能。
* 堆叠`GameplayEffects`。
* 在蓝图中创建的`GameplayAbilities`。
* 在 C++ 创建的`GameplayAbilities`。
* 实例化每个`Actor`的`GameplayAbilities`。
* 非实例化的`GameplayAbilities`（跳跃）。
* 静态`GameplayCues`（如射击时弹丸撞击产生的粒子特效）
* Actor相关的`GameplayCues`（冲刺和眩晕粒子效果）

英雄类具有以下能力：

| 能力     | 输入绑定    | 预测  | C++/蓝图 | 描述                                            |
| ------ | ------- | --- | ------ | --------------------------------------------- |
| 跳跃     | 空格键     | 是   | C++    | 使英雄跳跃。                                        |
| 开火     | 鼠标左键    | 否   | C++    | 从英雄的枪中发射弹丸。动画是预测的，但弹丸不是。                      |
| 瞄具瞄准   | 鼠标右键    | 是   | 蓝图     | 按住按钮时，英雄会走得更慢，相机会放大，以便用枪进行更精确的射击。             |
| 冲刺     | 左 Shift | 是   | 蓝图     | 按住按钮时，英雄会跑得更快，消耗体力。                           |
| 前向冲刺   | Q       | 是   | 蓝图     | 英雄消耗耐力向前冲刺。                                   |
| 被动护甲叠加 | 被动(无按键) | 否   | 蓝图     | 每 4 秒，英雄获得一层盔甲，最多 4 层。受到伤害会移除一层盔甲。            |
| 流星     | R       | 否   | 蓝图     | 玩家瞄准一个位置，向敌人投下一颗流星，造成伤害并击晕他们。目标是预测的，而生成流星则不是。 |

`GameplayAbilities`是用 C++ 还是 Blueprint 创建的并不重要。这里使用了两者的混合，例如如何在每种语言中执行它们。

小兵不附带任何预定义的`GameplayAbilities`。红色小兵有更多的生命恢复，而蓝色小兵有更高的起始生命值。

对于`GameplayAbility`的命名，我使用了后缀`_BP`来表`_BP`的逻辑是在蓝图中创建的。缺少后缀意味着逻辑是用 C++ 创建的。

>译者注：我的项目，比如Luna里，在C++中我使用项目民作为每个类的前缀，比如LunaCharacterBase，LunaPlayerController,LunaAbilitySystemComponent等。在蓝图中，我使用缩写前缀，比如大多数蓝图类使用BP_，还有UI蓝图使用WBP_,接口用BPI，GA蓝图用GA_，GE蓝图用GE_等。我还有一个仓库是关于命名规范的，感兴趣的可以看看。

| 前缀  | 资源类型            |
| --- | --------------- |
| GA_ | GameplayAbility |
| GC_ | GameplayCue     |
| GE_ | GameplayEffect  |

**[⬆ Back to Top](#table-of-contents)**

## 3.使用 GAS 设置项目
使用 GAS 设置项目的基本步骤：
1. 在编辑器中启用 GameplayAbilitySystem 插件。
2. 编辑`YourProjectName.Build.cs`以将`"GameplayAbilities", "GameplayTags", "GameplayTasks"`添加到`PrivateDependencyModuleNames`中。注：如果后续有报错的情况下可以把`"GameplayAbilities"`移动到`PublicDependencyModuleNames`。
3. 刷新重新生成 Visual Studio 项目文件。
4. 从 4.24 开始，现在必须调用 `UAbilitySystemGlobals::Get().InitGlobalData()` 使用 [`TargetData`](#concepts-targeting-data)。示例项目在`UAssetManager::StartInitialLoading()`中执行此操作。有关更多信息，请参见 [`InitGlobalData()`](#concepts-asg-initglobaldata)。

这就是启用 GAS 所需要做的全部事情。从这里，将[`ASC`](#concepts-asc) 和 [`AttributeSet`](#concepts-as) 添加到您的`Character`或`PlayerState`中并开始制作[`GameplayAbilities`](#concepts-ga) 和 [`GameplayEffects`](#concepts-ge)！