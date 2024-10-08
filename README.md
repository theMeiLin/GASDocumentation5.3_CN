# GAS文档
我对于 Unreal Engine 5 的 Gameplay Ability System 插件（GAS）的理解，以及一个简单的多人游戏样本项目的介绍。请注意，这并非官方提供的文档，并且此项目及本人均未得到 Epic Games 的授权或关联。因此，我无法确保所提供的信息完全准确无误。

这份文档的目标是解释 GAS 中的主要概念和类，并基于我使用它的经验提供一些额外的见解。在社区用户中，关于 GAS 存在大量的“部落知识”（即非正式传播的经验和技巧），我打算在这里分享我所有的相关知识。

本示例项目与文档已更新至 Unreal Engine 5.3 版本。尽管我们为 Unreal Engine 的旧版本保留了文档的分支，但这些分支不再接受维护与更新，因此可能包含错误或信息已经过时。请务必选用与你当前使用引擎版本相匹配的文档分支进行查阅。

[GASShooter](https://github.com/tranek/GASShooter) 作为配套的示例项目，深入演示了如何在多人 FPS（第一人称射击）或 TPS（第三人称射击）游戏场景下，利用 GAS 实现更为复杂的机制与功能

最为完整和权威的文档资源，永远是插件本身的源代码。

## 目录

<!-- TOC -->
* [GAS文档](#gas文档)
  * [目录](#目录)
  * [1.GameplayAbilitySystem 插件简介](#1gameplayabilitysystem-插件简介)
  * [2.示例项目](#2示例项目)
  * [3.使用 GAS 设置项目](#3使用-gas-设置项目)
  * [4.GAS 概念](#4gas-概念)
    * [章节](#章节)
    * [4.1 Ability System Component](#41-ability-system-component)
    * [4.1.1 复制模式](#411-复制模式)
    * [4.1.2 设置和初始化](#412-设置和初始化)
    * [4.2 Gameplay Tags](#42-gameplay-tags)
    * [4.2.1 响应游戏标签的变化](#421-响应游戏标签的变化)
    * [4.3 Attributes](#43-attributes)
      * [4.3.1 Attribute 定义](#431-attribute-定义)
      * [4.3.2 `BaseValue` vs `CurrentValue`（基础值 vs 当前值）](#432-basevalue-vs-currentvalue基础值-vs-当前值)
      * [4.3.3 Meta Attributes](#433-meta-attributes)
      * [4.3.4 响应属性更改](#434-响应属性更改)
      * [4.3.5 Derived Attributes（派生属性）](#435-derived-attributes派生属性)
    * [4.4 Attribute Set](#44-attribute-set)
      * [4.4.1 Attribute Set 定义](#441-attribute-set-定义)
      * [4.4.2 Attribute Set Design（设计）](#442-attribute-set-design设计)
        * [4.4.2.1 Subcomponents with Individual Attributes（带有个别属性的子组件）](#4421-subcomponents-with-individual-attributes带有个别属性的子组件)
        * [4.4.2.2 在运行时添加和删除 AttributeSet](#4422-在运行时添加和删除-attributeset)
        * [4.4.2.3 Item Attributes (Weapon Ammo)【物品属性（武器弹药)】](#4423-item-attributes-weapon-ammo物品属性武器弹药)
          * [4.4.2.3.1 在物品上使用普通浮点数](#44231-在物品上使用普通浮点数)
          * [4.4.2.3.2 在物品上使用 `AttributeSet`](#44232-在物品上使用-attributeset)
          * [4.4.2.3.3 物品上的 `ASC`](#44233-物品上的-asc)
      * [4.4.3 Defining Attributes](#443-defining-attributes)
      * [4.4.4 Initializing Attributes](#444-initializing-attributes)
      * [4.4.5 PreAttributeChange()](#445-preattributechange)
      * [4.4.6 PostGameplayEffectExecute()](#446-postgameplayeffectexecute)
      * [4.4.7 OnAttributeAggregatorCreated()](#447-onattributeaggregatorcreated)
    * [4.5 Gameplay Effects](#45-gameplay-effects)
      * [4.5.1 Gameplay Effect Definition](#451-gameplay-effect-definition)
      * [4.5.2 应用 Gameplay Effects](#452-应用-gameplay-effects)
      * [4.5.3 移除 Gameplay Effects](#453-移除-gameplay-effects)
      * [4.5.4 Gameplay Effect Modifiers（游戏玩法效果修改器）](#454-gameplay-effect-modifiers游戏玩法效果修改器)
        * [4.5.4.1 Multiply and Divide Modifiers（乘法和除法修饰符）](#4541-multiply-and-divide-modifiers乘法和除法修饰符)
        * [4.5.4.2 Gameplay Tags on Modifiers](#4542-gameplay-tags-on-modifiers)
      * [4.5.5 Stacking Gameplay Effects](#455-stacking-gameplay-effects)
      * [4.5.6 Granted Abilities（授予能力）](#456-granted-abilities授予能力)
      * [4.5.7 Gameplay Effect Tags](#457-gameplay-effect-tags)
      * [4.5.8 Immunity（免疫）](#458-immunity免疫)
      * [4.5.9 Gameplay Effect Spec](#459-gameplay-effect-spec)
        * [4.5.9.1 SetByCallers](#4591-setbycallers)
      * [4.5.10 Gameplay Effect Context](#4510-gameplay-effect-context)
      * [4.5.11 Modifier Magnitude Calculation](#4511-modifier-magnitude-calculation)
      * [4.5.12 Gameplay Effect Execution Calculation](#4512-gameplay-effect-execution-calculation)
        * [4.5.12.1 Sending Data to Execution Calculations](#45121-sending-data-to-execution-calculations)
          * [4.5.12.1.1 SetByCaller](#451211-setbycaller)
          * [4.5.12.1.2 Backing Data Attribute Calculation Modifier](#451212-backing-data-attribute-calculation-modifier)
          * [4.5.12.1.3 Backing Data Temporary Variable Calculation Modifier](#451213-backing-data-temporary-variable-calculation-modifier)
          * [4.5.12.1.4 Gameplay Effect Context](#451214-gameplay-effect-context)
      * [4.5.13 Custom Application Requirement](#4513-custom-application-requirement)
      * [4.5.14 Cost Gameplay Effect](#4514-cost-gameplay-effect)
      * [4.5.15 Cooldown Gameplay Effect](#4515-cooldown-gameplay-effect)
        * [4.5.15.1 Get the Cooldown Gameplay Effect's Remaining Time](#45151-get-the-cooldown-gameplay-effects-remaining-time)
        * [4.5.15.2 监听冷却开始和结束](#45152-监听冷却开始和结束)
        * [4.5.15.3 Predicting Cooldowns](#45153-predicting-cooldowns)
      * [4.5.16 Changing Active Gameplay Effect Duration](#4516-changing-active-gameplay-effect-duration)
      * [4.5.17 在运行时创建动态 Gameplay Effects](#4517-在运行时创建动态-gameplay-effects)
        * [例子](#例子)
      * [4.5.18 Gameplay Effect Containers](#4518-gameplay-effect-containers)
    * [4.6 Gameplay Abilities](#46-gameplay-abilities)
      * [4.6.1 Gameplay Ability Definition](#461-gameplay-ability-definition)
        * [4.6.1.1 Replication Policy](#4611-replication-policy)
        * [4.6.1.2 Server Respects Remote Ability Cancellation](#4612-server-respects-remote-ability-cancellation)
        * [4.6.1.3 Replicate Input Directly](#4613-replicate-input-directly)
      * [4.6.2 Binding Input to the ASC](#462-binding-input-to-the-asc)
        * [4.6.2.1 Binding to Input without Activating Abilities](#4621-binding-to-input-without-activating-abilities)
      * [4.6.3 Granting Abilities](#463-granting-abilities)
      * [4.6.4 Activating Abilities](#464-activating-abilities)
        * [4.6.4.1 Passive Abilities](#4641-passive-abilities)
        * [4.6.4.2 Activation Failed Tags](#4642-activation-failed-tags)
      * [4.6.5 Canceling Abilities](#465-canceling-abilities)
      * [4.6.6 Getting Active Abilities](#466-getting-active-abilities)
      * [4.6.7 Instancing Policy](#467-instancing-policy)
      * [4.6.8 Net Execution Policy](#468-net-execution-policy)
      * [4.6.9 Ability Tags](#469-ability-tags)
      * [4.6.10 Gameplay Ability Spec](#4610-gameplay-ability-spec)
      * [4.6.11 Passing Data to Abilities](#4611-passing-data-to-abilities)
      * [4.6.12 Ability Cost and Cooldown](#4612-ability-cost-and-cooldown)
      * [4.6.13 Leveling Up Abilities](#4613-leveling-up-abilities)
      * [4.6.14 Ability Sets](#4614-ability-sets)
      * [4.6.15 Ability Batching](#4615-ability-batching)
      * [4.6.16 Net Security Policy](#4616-net-security-policy)
    * [4.7 Ability Tasks](#47-ability-tasks)
    * [4.7.1 Ability Task Definition](#471-ability-task-definition)
    * [4.7.2 Custom Ability Tasks](#472-custom-ability-tasks)
    * [4.7.3 Using Ability Tasks](#473-using-ability-tasks)
    * [4.7.4 Root Motion Source Ability Tasks](#474-root-motion-source-ability-tasks)
    * [4.8 Gameplay Cues](#48-gameplay-cues)
      * [4.8.1 Gameplay Cue Definition](#481-gameplay-cue-definition)
      * [4.8.2 Triggering Gameplay Cues](#482-triggering-gameplay-cues)
      * [4.8.3 Local Gameplay Cues](#483-local-gameplay-cues)
      * [4.8.4 Gameplay Cue Parameters](#484-gameplay-cue-parameters)
      * [4.8.5 Gameplay Cue Manager](#485-gameplay-cue-manager)
      * [4.8.6 Prevent Gameplay Cues from Firing](#486-prevent-gameplay-cues-from-firing)
      * [4.8.7 Gameplay Cue Batching](#487-gameplay-cue-batching)
        * [4.8.7.1 Manual RPC](#4871-manual-rpc)
        * [4.8.7.2 Multiple GCs on one GE](#4872-multiple-gcs-on-one-ge)
      * [4.8.8 Gameplay Cue Events](#488-gameplay-cue-events)
      * [4.8.9 Gameplay Cue Reliability](#489-gameplay-cue-reliability)
    * [4.9 Ability System Globals](#49-ability-system-globals)
      * [4.9.1 InitGlobalData()](#491-initglobaldata)
    * [4.10 Prediction](#410-prediction)
      * [4.10.1 Prediction Key](#4101-prediction-key)
      * [4.10.2 Creating New Prediction Windows in Abilities](#4102-creating-new-prediction-windows-in-abilities)
      * [4.10.3 Predictively Spawning Actors](#4103-predictively-spawning-actors)
      * [4.10.4 Future of Prediction in GAS](#4104-future-of-prediction-in-gas)
      * [4.10.5 Network Prediction Plugin](#4105-network-prediction-plugin)
    * [4.11 Targeting](#411-targeting)
      * [4.11.1 Target Data](#4111-target-data)
    * [`FGameplayAbilityTargetData`](#fgameplayabilitytargetdata)
      * [4.11.2 Target Actors](#4112-target-actors)
      * [4.11.3 Target Data Filters](#4113-target-data-filters)
      * [4.11.4 Gameplay Ability World Reticles](#4114-gameplay-ability-world-reticles)
      * [4.11.5 Gameplay Effect Containers Targeting](#4115-gameplay-effect-containers-targeting)
  * [5. Commonly Implemented Abilities and Effects](#5-commonly-implemented-abilities-and-effects)
    * [5.1 Stun](#51-stun)
    * [5.2 Sprint](#52-sprint)
    * [5.3 Aim Down Sights](#53-aim-down-sights)
    * [5.4 Lifesteal](#54-lifesteal)
    * [5.5 Generating a Random Number on Client and Server](#55-generating-a-random-number-on-client-and-server)
    * [5.6 Critical Hits](#56-critical-hits)
    * [5.7 Non-Stacking Gameplay Effects but Only the Greatest Magnitude Actually Affects the Target](#57-non-stacking-gameplay-effects-but-only-the-greatest-magnitude-actually-affects-the-target)
    * [5.8 Generate Target Data While Game is Paused](#58-generate-target-data-while-game-is-paused)
    * [5.9 One Button Interaction System](#59-one-button-interaction-system)
  * [6. Debugging GAS](#6-debugging-gas)
    * [6.1 showdebug abilitysystem](#61-showdebug-abilitysystem)
    * [6.2 Gameplay Debugger](#62-gameplay-debugger)
    * [6.3 GAS Logging](#63-gas-logging)
  * [7. Optimizations](#7-optimizations)
    * [7.1 Ability Batching](#71-ability-batching)
    * [7.2 Gameplay Cue Batching](#72-gameplay-cue-batching)
    * [7.3 AbilitySystemComponent Replication Mode](#73-abilitysystemcomponent-replication-mode)
    * [7.4 Attribute Proxy Replication](#74-attribute-proxy-replication)
    * [7.5 ASC Lazy Loading](#75-asc-lazy-loading)
  * [8. Quality of Life Suggestions](#8-quality-of-life-suggestions)
    * [8.1 Gameplay Effect Containers](#81-gameplay-effect-containers)
    * [8.2 Blueprint AsyncTasks to Bind to ASC Delegates](#82-blueprint-asynctasks-to-bind-to-asc-delegates)
  * [9. Troubleshooting](#9-troubleshooting)
    * [9.1 `LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`](#91-logabilitysystem-warning-cant-activate-localonly-or-localpredicted-ability-s-when-not-local)
    * [9.2 `ScriptStructCache` errors](#92-scriptstructcache-errors)
    * [9.3 Animation Montages are not replicating to clients](#93-animation-montages-are-not-replicating-to-clients)
    * [9.4 Duplicating Blueprint Actors is setting AttributeSets to nullptr](#94-duplicating-blueprint-actors-is-setting-attributesets-to-nullptr)
    * [9.5 unresolved external symbol UEPushModelPrivate::MarkPropertyDirty(int,int)](#95-unresolved-external-symbol-uepushmodelprivatemarkpropertydirtyintint)
    * [9.6 Enum names are now represented by path name](#96-enum-names-are-now-represented-by-path-name)
  * [10. Common GAS Acronyms](#10-common-gas-acronyms)
  * [11. Other Resources](#11-other-resources)
    * [11.1 Q&A With Epic Game's Dave Ratti](#111-qa-with-epic-games-dave-ratti)
      * [11.1.2 Community Questions 2](#1112-community-questions-2)
<!-- TOC -->

<a name="intro"></a>
## 1.GameplayAbilitySystem 插件简介
引用自[官方文档](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html)：
>**Gameplay技能系统** 是一个高度灵活的框架，可用于构建你可能会在RPG或MOBA游戏中看到的技能和属性类型。你可以构建可供游戏中的角色使用的动作或被动技能，使这些动作导致各种属性累积或损耗的状态效果，实现约束这些动作使用的"冷却"计时器或资源消耗，更改技能等级及每个技能等级的技能效果，激活粒子或音效，等等。简单来说，此系统可帮助你在任何现代RPG或MOBA游戏中设计、实现及高效关联各种游戏中的技能，既包括跳跃等简单技能，也包括你喜欢的角色的复杂技能集。

GameplayAbilitySystem（GAS）插件由 Epic Games 开发，并随 Unreal Engine 一同发布。这一系统已在多款 AAA 商业游戏中经过实战考验，包括《Paragon》和《Fortnite》等知名作品，证明了其稳定性和高效性。

插件提供的即用方案针对单人及多人游戏的全面解决方案：
* 实现基于等级的角色能力和技能，附带可选的成本与冷却时间。([GameplayAbilities](#concepts-ga))
* 操纵属于游戏对象的数值`Attributes`。([Attributes](#concepts-a))
* 对游戏对象施加状态效果。([`GameplayEffects`](#concepts-ge))
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

**GAS 必须在 C++** 中设置，但设计师可以在蓝图中创建 `GameplayAbilities` 和 ``GameplayEffects``。

GAS 的当前问题：
* `GameplayEffect`延迟协调（这导致了高延迟的玩家相较于低延迟的玩家，在使用冷却时间较短的能力时，能够触发的频率较低，进而影响了游戏的公平性和玩家体验）。
* 无法预测``GameplayEffects``的删除。但是，我们可以预测添加带有反向效果的``GameplayEffects``，从而有效地消除它们。这并不总是适当或可行的，并且仍然是一个问题。
* 缺少样板模板、多人游戏示例和文档。希望这会有所帮助！

**[⬆ 回到顶部](#目录)**

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
* 在`GameplayAbilities`内部和外部应用和删除``GameplayEffects``。
* 应用受护甲减伤后的伤害来改变角色的生命值。
* `GameplayEffectExecutionCalculations`(游戏玩法效果执行计算)。
* 眩晕效果。
* 死亡和重生。
* 从服务器上的能力生成 Actors（发射物）。
* 通过瞄准射击和冲刺动作，预判性地改变本地玩家的速度。
* 持续消耗体力进行冲刺。
* 使用法力来施展能力。
* 被动技能。
* 堆叠``GameplayEffects``。
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

**[⬆ 回到顶部](#目录)**

## 3.使用 GAS 设置项目
使用 GAS 设置项目的基本步骤：
1. 在编辑器中启用 GameplayAbilitySystem 插件。
2. 编辑`YourProjectName.Build.cs`以将`"GameplayAbilities", "GameplayTags", "GameplayTasks"`添加到`PrivateDependencyModuleNames`中。注：如果后续有报错的情况下可以把`"GameplayAbilities"`移动到`PublicDependencyModuleNames`。
3. 刷新重新生成 Visual Studio 项目文件。
4. 从 4.24 开始，现在必须调用 `UAbilitySystemGlobals::Get().InitGlobalData()` 使用 [`TargetData`](#concepts-targeting-data)。示例项目在`UAssetManager::StartInitialLoading()`中执行此操作。有关更多信息，请参见 [`InitGlobalData()`](#concepts-asg-initglobaldata)。

这就是启用 GAS 所需要做的全部事情。从这里，将[`ASC`](#concepts-asc) 和 [`AttributeSet`](#concepts-as) 添加到您的`Character`或`PlayerState`中并开始制作[`GameplayAbilities`](#concepts-ga) 和 [``GameplayEffects``](#concepts-ge)！

**[⬆ 回到顶部](#目录)**

## 4.GAS 概念

### 章节

> 4.1 [Ability System Component](#concepts-asc)    
> 4.2 [Gameplay Tags](#concepts-gt)    
> 4.3 [Attributes](#concepts-a)    
> 4.4 [Attribute Set](#concepts-as)    
> 4.5 [Gameplay Effects](#concepts-ge)    
> 4.6 [Gameplay Abilities](#concepts-ga)    
> 4.7 [Ability Tasks](#concepts-at)    
> 4.8 [Gameplay Cues](#concepts-gc)    
> 4.9 [Ability System Globals](#concepts-asg)    
> 4.10 [Prediction](#concepts-p)

### 4.1 Ability System Component
`AbilitySystemComponent`（`ASC`）是 GAS 的核心。它是一个`UActorComponent` ([`UAbilitySystemComponent`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemComponent/index.html))，用于处理与系统的所有交互。任何希望使用[`GameplayAbilities`](#concepts-ga)（游戏玩法能力）、拥有[`Attributes`](#concepts-a)（属性）或接收[``GameplayEffects``](#concepts-ge)的`Actor`都必须附加一个`ASC`（AbilitySystemComponent）。这些对象全部存在于`ASC`内部，并由其管理和复制（除了`Attributes`，它们是由各自的[`AttributeSet`](#concepts-as)进行复制的）。开发者被期望但不是强制要求去继承并扩展这个类。

附加了`ASC`的`Actor`称为`ASC`的`OwnerActor`。`ASC`的物理表现形式`Actor`被称为`AvatarActor`。`OwnerActor`和`AvatarActor`可以是同一个`Actor`，就像在一款多人在线战术竞技游戏（MOBA）中简单的AI小兵那样。
>译者注：这句话说明了在某些游戏设计中，特别是对于简单的AI控制单位（如MOBA游戏中的小兵），OwnerActor（通常负责持有和激活ASC）和AvatarActor（ASC的物理表现形式）可以是同一实体。这种设计简化了对象结构，因为不需要额外的Actor来分别承担能力和物理表现的角色。

它们也可以是不同的`Actors`，例如在MOBA游戏中玩家控制的英雄，其中`OwnerActor`是`PlayerState`和`AvatarActor`是英雄的`Character`类。大多数`Actors`都会拥有`ASC`。如果你的`Actor`将会重生，并且需要在每次重生之间保持`Attributes`（属性）或 ``GameplayEffects``（游戏玩法效果）的持久性（就像MOBA游戏中的英雄那样），那么`ASC`（AbilitySystemComponent）的理想位置是在 `PlayerState`上。

**注意:** 如果`ASC`位于`PlayerState`上，则需要提高`PlayerState`的`NetUpdateFrequency`。它在`PlayerState`上的默认值非常低，这可能导致像`Attributes`（属性）和`GameplayTags`（游戏玩法标签）这样的变化在客户端发生延迟或感知到的滞后。一定要启用[`Adaptive Network Update Frequency`](https://docs.unrealengine.com/en-US/Gameplay/Networking/Actors/Properties/index.html#adaptivenetworkupdatefrequency)，《堡垒之夜》就使用了这一特性。
>在你的PlayerState构造函数中调用NetUpdateFrequency。
>```c++
>ALunaPlayerState::ALunaPlayerState()
>{
>	NetupdateFrequency = 100.f;
>	
>	// your code
>	// ······
>}
>```

无论是`OwnerActor`还是`AvatarActor`（如果它们是不同的`Actors`），都应该实现`IAbilitySystemInterface`接口。这个接口有一个必须被覆写的函数，`UAbilitySystemComponent* GetAbilitySystemComponent() const`，它返回指向其自身`ASC`（AbilitySystemComponent）的指针。`ASCs`组件们通过查找这个接口函数，在系统内部彼此间进行交互。

`ASC`（AbilitySystemComponent）在其`FActive`GameplayEffects`Container Active`GameplayEffects``成员变量中维护当前激活的``GameplayEffects``（游戏玩法效果）

`ASC`（AbilitySystemComponent）在其`FGameplayAbilitySpecContainer ActivatableAbilities`成员变量中存储被授予的`Gameplay Abilities`（游戏玩法能力）。每当计划遍历ActivatableAbilities.Items时，务必在循环上方添加`ABILITYLIST_SCOPE_LOCK();`来锁定列表，防止在迭代过程中因移除能力而导致的列表更改。每个处于作用域内的`ABILITYLIST_SCOPE_LOCK();`都会递增`AbilityScopeLockCount`计数器，当它超出作用域时则递减计数器。不要尝试在`ABILITYLIST_SCOPE_LOCK();`的作用域内移除能力（清除能力的函数内部会检查`AbilityScopeLockCount`，以防止在列表被锁定时移除能力）。

### 4.1.1 复制模式
`ASC`（AbilitySystemComponent）定义了三种不同的复制模式，用于复制``GameplayEffects``（游戏玩法效果）、`GameplayTags`（游戏玩法标签）和`GameplayCues`（游戏玩法线索）——分别是`Full`（全量）、`Mixed`（混合）和`Minimal`（最小）。而`Attributes`（属性）则是通过它们所属的`AttributeSet`进行复制的。

| 复制模式      | 何时使用               | 描述                                                                       |
| --------- | ------------------ | ------------------------------------------------------------------------ |
| `Full`    | 单人游戏               | 每个`GameplayEffect`都会复制到每个客户端。                                            |
| `Mixed`   | 多人游戏，玩家控制的`Actors` | ``GameplayEffects``仅复制给拥有该效果的客户端。而`GameplayTags`和`GameplayCues`则被复制给所有客户端  |
| `Minimal` | 多人游戏，AI控制的`Actors` | ``GameplayEffects``永远不会被复制给任何客户端。只有`GameplayTags`和`GameplayCues`被复制给所有客户端。 |
>译者者：按我理解就是单机游戏，PVP，PVE(可能包含PVP)。

**注意:** `Mixed` 复制模式期望 `OwnerActor` 的 `Owner` 是`Controller`。`PlayerState` 的`Owner` 默认是 `Controller`， 但 `Character` 的不是。如果在`OwnerActor` 不是 `PlayerState` 的情况下使用 `Mixed` 复制模式， 那么你需要通过调用 `SetOwner()` 方法并传入一个有效的 `Controller` 来设置 `OwnerActor` 的所有者。

从 4.24 版本开始，`PossessedBy()`方法现在会将 `Pawn` 的所有者设置为新的 `Controller`。

**[⬆ 回到顶部](#目录)**

### 4.1.2 设置和初始化
`ASC`（Actor Subclass Component）通常在 `OwnerActor` 的构造函数中创建，并明确标记为可复制。**这必须在 C++ 中完成**。

```c++  
AGDPlayerState::AGDPlayerState()  
{  
    // 创建技能系统组件，并设置其显式复制
    AbilitySystemComponent = CreateDefaultSubobject<UGDAbilitySystemComponent>(TEXT("AbilitySystemComponent"));
    AbilitySystemComponent->SetIsReplicated(true);
    //...
}  
```

`ASC`（Actor Subclass Component）需要在服务器和客户端上使用其 `OwnerActor` 和 `AvatarActor` 进行初始化。你应该在`Pawn` 的 `Controller` 已经设置（即在占有之后）进行初始化。对于单人游戏，你只需关注服务器端的初始化路径。

对于玩家控制的角色，其中 `ASC` 位于 `Pawn` 上，我通常会在服务器端的`Pawn` 的 `PossessedBy()`函数中进行初始化，并在客户端的 `PlayerController` 的`AcknowledgePossession()` 函数中进行初始化。

```c++  
void APACharacterBase::PossessedBy(AController * NewController)  
{
    Super::PossessedBy(NewController);  
    if (AbilitySystemComponent)    
    {       
        AbilitySystemComponent->InitAbilityActorInfo(this, this);
    }
    
    // ASC MixedMode 复制要求 ASC 所有者的所有者是控制器。
    SetOwner(NewController);
}  
```

```c++  
void APAPlayerControllerBase::AcknowledgePossession(APawn* P)  
{  
    Super::AcknowledgePossession(P);  
    APACharacterBase* CharacterBase = Cast<APACharacterBase>(P);    
    if (CharacterBase)    
    {
        CharacterBase->GetAbilitySystemComponent()->InitAbilityActorInfo(CharacterBase, CharacterBase);
    } 
	 
    //...
}  
```

对于玩家控制的角色，其中`ASC` 位于 `PlayerState` 上，我通常会在服务器端的 `Pawn` 的 `PossessedBy()`函数中初始化 `ASC`，并在客户端的 `Pawn` 的 `OnRep_PlayerState()` 函数中进行初始化。这样可以确保 `PlayerState` 在客户端存在。

```c++  
// 仅限服务器  
void AGDHeroCharacter::PossessedBy(AController * NewController)  
{  
    Super::PossessedBy(NewController);  
    AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();    
    if (PS)    
    {
        // 在服务器上设置 ASC。客户端在 OnRep_PlayerState（） 中执行此操作
        AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());  
        // AI 不会有 PlayerControllers，因此我们可以在此处再次初始化，以确保万无一失。 
        // 对于具有 PlayerController 的英雄，启动两次没有坏处。      
        PS->GetAbilitySystemComponent()->InitAbilityActorInfo(PS, this);    
    }  
          
    //...  
}  
```

```c++  
// 仅限客户端 
void AGDHeroCharacter::OnRep_PlayerState()  
{  
    Super::OnRep_PlayerState();  
    AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();    
    if (PS)    
    {
        // 为客户端设置 ASC。服务器在 PossessedBy 中执行此操作。       
        AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());
	      
        // 初始化客户端的 ASC Actor 信息。当服务器拥有新的 Actor 时，它将初始化其 ASC。 
        AbilitySystemComponent->InitAbilityActorInfo(PS, this);    
    }  
    
    // ...
}  
```

>译者注：我一般都使用第二种方法初始化ASC，尽量使用虚幻引擎提供的框架开发。

如果你收到错误信息 `LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`这意味着你在客户端没有正确初始化你的 `ASC`。

**[⬆ 回到顶部](#目录)**

### 4.2 Gameplay Tags
[`FGameplayTags`](https://docs.unrealengine.com/en-US/API/Runtime/GameplayTags/FGameplayTag/index.html) 是以 `Parent.Child.Grandchild...` 形式的层次化命名标签，它们注册在 `GameplayTagManager` 中。这些标签对于分类和描述对象的状态极其有用。例如，如果一个角色处于眩晕状态，我们可以在眩晕持续期间给它添加一个 `State.Debuff.Stun` 的`GameplayTag`。

你会发现，以前使用布尔值或枚举来处理的事情，现在可以用`GameplayTags`来替代，并且你会基于对象是否具有特定的 `GameplayTags` 来执行布尔逻辑判断。
>在游戏开发中，GameplayTags 可以作为更灵活的替代方案，取代原本使用布尔变量或枚举的地方。
>开发者会根据对象是否携带特定的 GameplayTags 来进行条件判断和逻辑处理。

当我们给对象添加标签时，通常会将其添加到对象的 `ASC`（如果存在的话），以便 `ASC`（Gameplay Ability System）能够与这些标签交互。`UAbilitySystemComponent` 实现了`IGameplayTagAssetInterface` 接口，提供了访问其拥有的 `GameplayTags` 的方法。

多个 `GameplayTags` 可以存储在 `FGameplayTagContainer` 中。相比于使用 `TArray<FGameplayTag>`，选择 `GameplayTagContainer` 更优，因为 `GameplayTagContainers` 内部实现了一些效率优化。虽然标签本质上是 `FNames` 类型，但如果项目设置中启用了`Fast Replication`，它们可以被高效地打包在 `GameplayTagContainers` 中进行复制。`Fast Replication` 要求服务器和客户端拥有相同的`GameplayTags` 列表，但这通常不会成为问题，因此建议启用这个选项。此外，`GameplayTagContainers` 还支持返回 `TArray<FGameplayTag>` 以供迭代使用。

存储在 `FGameplayTagCountContainer` 中的 `GameplayTags` 包含一个 `TagMap`，用于记录该 GameplayTag 的实例数量。即使 `FGameplayTagCountContainer` 中仍然存在某个 `GameplayTag`，但如果其 `TagMapCount`为零，这也可能发生。在调试过程中，如果发现 ASC 似乎仍持有某个 `GameplayTag`，你可能会遇到这种情况。任何`HasTag()`或 `HasMatchingTag()` 或类似的功能都会检查 `TagMapCount`，如果 `GameplayTag` 不存在或其 `TagMapCount`为零，则会返回 false。

`GameplayTags` 必须预先在 `DefaultGameplayTags.ini` 文件中定义。虚幻编辑器在项目设置中提供了一个界面，让开发者无需直接编辑 `DefaultGameplayTags.ini`即可管理 `GameplayTags`。`GameplayTags` 编辑器支持创建、重命名、搜索引用以及删除 `GameplayTags`。

![GameplayTag Editor in Project Settings](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/gameplaytageditor.png))

搜索 `GameplayTag` 引用会调出编辑器中熟悉的 `Reference Viewer` 图表，显示所有引用该 `GameplayTag` 的资源。然而，这并不会显示出任何引用了 `GameplayTag` 的 C++ 类。

重命名 `GameplayTags` 会产生重定向，使得仍然引用原始 `GameplayTag` 的资源能够转向新的 `GameplayTag`。如果可能的话，我更倾向于创建一个新的 `GameplayTag`，手动更新所有引用指向新的 `GameplayTag`，然后删除旧的 `GameplayTag`，以此避免产生重定向。

除了 `Fast Replication`，`GameplayTag` 编辑器还提供了一个选项，用于填充经常被复制的 `GameplayTags`，以进一步优化它们的性能。

如果从 `GameplayEffect` 添加，`GameplayTags` 将会被复制。`ASC` 允许你添加 `LooseGameplayTags`，这类标签不会被复制，需要手动管理。示例项目中使用了 `LooseGameplayTag` 对 `State.Dead` 进行标记，以便客户端在其生命值降至零时能立即作出反应。复活操作会手动将 `TagMapCount` 重置为零。只有在处理 `LooseGameplayTags` 时，才应手动调整`TagMapCount`。比起直接修改 `TagMapCount`，更推荐使用 `UAbilitySystemComponent::AddLooseGameplayTag()` 和 `UAbilitySystemComponent::RemoveLooseGameplayTag()` 函数。

在 C++ 中获取对`GameplayTag`的引用：
```c++  
FGameplayTag::RequestGameplayTag(FName("Your.GameplayTag.Name"))  
```

对于高级的 `GameplayTag` 操作，如获取父级或子级 `GameplayTags`，可以查看 `GameplayTagManager` 提供的函数。要访问 `GameplayTagManager`，需要包含 `GameplayTagManager.h` 头文件，并通过 `UGameplayTagManager::Get().FunctionName` 的方式调用其功能。实际上，`GameplayTagManager` 将 `GameplayTags` 存储为关系节点（父级、子级等），这比常规字符串操作和比较提供了更快的处理速度。

`GameplayTags` 和 `GameplayTagContainers` 可以使用可选的 `UPROPERTY` 特性 `Meta = (Categories = "GameplayCue")`，这一特性会在蓝图中过滤显示的标签，仅展示具有`GameplayCue` 父级标签的 `GameplayTags`。当你确定 `GameplayTag` 或 `GameplayTagContainer` 变量仅用于 `GameplayCues` 时，这一点非常有用。

另一种选择是使用名为 `FGameplayCueTag` 的独立结构体，它封装了一个 `FGameplayTag`，并且在蓝图中自动过滤 `GameplayTags`，仅显示那些具有 `GameplayCue` 作为父级标签的标签。

如果你想在函数中过滤一个 `GameplayTag` 参数，可以使用 `UFUNCTION` 特性 `Meta = (GameplayTagFilter = "GameplayCue")`。然而，函数中的 `GameplayTagContainer` 参数无法被过滤。如果你希望修改引擎以支持这一功能，可以参考 `Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagGraphPin.cpp` 中 `SGameplayTagGraphPin::ParseDefaultValueData()`函数的实现。这里调用了 `FilterString = UGameplayTagsManager::Get().GetCategoriesMetaFromField(PinStructType);` 并将 `FilterString` 传递给 `SGameplayTagWidget` 在 `SGameplayTagGraphPin::GetListContent()` 中使用。而 `GameplayTagContainer` 相关的这些函数，在 `Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagContainerGraphPin.cpp` 文件中，并没有检查元字段属性，也就无法传递过滤条件。

示例项目广泛使用`GameplayTags`。

**[⬆ 回到顶部](#目录)**

### 4.2.1 响应游戏标签的变化
`ASC`（能力系统组件）提供了当`GameplayTags`被添加或移除时的委托。它接收一个`EGameplayTagEventType`参数，可以指定仅在`GameplayTag`被添加/移除时触发，或者在`GameplayTag`的`TagMapCount`有任何改变时触发。

```c++  
AbilitySystemComponent->RegisterGameplayTagEvent(FGameplayTag::RequestGameplayTag(FName("State.Debuff.Stun")), EGameplayTagEventType::NewOrRemoved).AddUObject(this, &AGDPlayerState::StunTagChanged);  
```

回调函数有一个参数用于接收`GameplayTag`和一个新的`TagCount`值。

```c++  
virtual void StunTagChanged(const FGameplayTag CallbackTag, int32 NewCount);  
```

**[⬆ 回到顶部](#目录)**

### 4.3 Attributes

#### 4.3.1 Attribute 定义
`Attributes`是由结构体[`FGameplayAttributeData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayAttributeData/index.html)定义的浮点数值，它可以代表从游戏角色的生命值到等级，再到药水充能次数等各种游戏相关的数值。如果某个数值是与游戏玩法紧密相关，并且属于某个`Actor`（游戏中的活动实体，如角色、物品等），那么应该考虑使用`Attribute`来表示它。`Attributes`一般应当只由[``GameplayEffects``](#concepts-ge)（游戏玩法效果）进行修改，这样ASC（能力系统组件）才能对这些变化进行[预测](#concepts-p)，确保游戏状态的一致性和正确性。

`Attributes`由[`AttributeSet`](#concepts-as)定义并存储其中。`AttributeSet`负责复制那些标记为需要复制的`Attributes`。关于如何定义`Attributes`，请参阅有关[`AttributeSets`](#concepts-as)的部分。

**提示:** 如果你不希望某个`Attribute`出现在编辑器的`Attributes`列表中，你可以使用`Meta = (HideInDetailsView)``属性修饰符`。

**[⬆ 回到顶部](#目录)**

#### 4.3.2 `BaseValue` vs `CurrentValue`（基础值 vs 当前值）
一个`Attribute`由两个值组成——`BaseValue`（基础值）和`CurrentValue`（当前值）。`BaseValue`是该属性的永久值，而`CurrentValue`则是`BaseValue`加上来自`GameplayEffects`（游戏玩法效果）的临时修改。例如，你的角色可能有一个移动速度属性，其`BaseValue`为600单位/秒。由于目前没有`GameplayEffects`修改移动速度，因此`CurrentValue`也是600单位/秒。如果她获得了一个临时的50单位/秒的移动速度增益，`BaseValue`保持不变，仍为600单位/秒，而`CurrentValue`现在变为600+50，总计650单位/秒。当移动速度增益效果到期后，`CurrentValue`会回退到`BaseValue`的600单位/秒。

GAS（Gameplay Ability System，游戏玩法能力系统）的初学者常常会将`BaseValue`与某一`Attribute`的最大值混淆，并试图将其当作最大值来处理。这是一种错误的做法。对于那些可能变化或在能力（abilities）或用户界面（UI）中引用的`Attributes`的最大值，应当作为独立的`Attributes`来对待。对于硬编码的最大和最小值，可以通过定义带有`FAttributeMetaData`的`DataTable`来设置这些边界值，但是Epic在其结构体上方的注释中称这种方法仍在“开发中”。更多细节可以在`AttributeSet.h`文件中找到。为了避免混淆，我建议那些在能力或UI中可引用的最大值应作为独立的`Attributes`来创建，而仅用于限制`Attributes`范围的硬编码最大和最小值，则应在`AttributeSet`中定义为固定的浮点数。`Attributes`的限制（clamping）在[PreAttributeChange()](#concepts-as-preattributechange)方法中讨论，该方法处理的是`CurrentValue`的变化；而在[PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute)方法中则处理来自`GameplayEffects`的`BaseValue`变化。
>译者注：简单来说，`BaseValue`不应该被误认为是`Attribute`的最大值。如果`Attribute`的最大值是动态的或需要在游戏逻辑中引用，它应该作为一个独立的属性存在。而对于那些仅用于限制`Attribute`范围的固定最大和最小值，可以作为硬编码的浮点数在`AttributeSet`中定义。此外，通过[PreAttributeChange()](#concepts-as-preattributechange)和[PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute)方法，可以分别在`Attribute`的`BaseValue`和`CurrentValue`发生改变时进行限制处理。

对`BaseValue`的永久性改变来源于`Instant`（即时）的`GameplayEffects`，而`Duration`（持续时间）和`Infinite`（无限）的`GameplayEffects`则改变`CurrentValue`。周期性的`GameplayEffects`被视为类似于瞬发的`GameplayEffects`，并且它们也改变`BaseValue`。

**[⬆ 回到顶部](#目录)**

#### 4.3.3 Meta Attributes

一些`Attributes`被当作临时值的占位符，这些值旨在与其它`Attributes`交互，这类属性被称为`Meta Attributes`。例如，我们通常将伤害定义为一种`Meta Attributes`。这样一来，GameplayEffect并不会直接改变我们的生命值`Attributes`，而是使用一个名为伤害的`Meta Attributes`作为占位符。这样做的好处是，伤害值可以在[`GameplayEffectExecutionCalculation`](#concepts-ge-ec)（游戏玩法效果执行计算）中通过增益和减益效果进行修改，并且可以在`AttributeSet`中进一步操作，例如先从当前护盾`Attributes`中扣除伤害值，然后再从生命值`Attributes`中扣除剩余的伤害。伤害`Meta Attributes`在不同的`GameplayEffects`之间不具备持久性，每次都会被新的`GameplayEffect`覆盖。通常情况下，`Meta Attributes`不会在网络间进行复制。

`Meta Attributes`为诸如伤害和治疗这类概念提供了良好的逻辑分离，区分了“我们造成了多少伤害？”和“我们如何处理这些伤害？”这两个问题。这种逻辑上的分离意味着我们的`Gameplay Effects`（游戏玩法效果）和`Execution Calculations`（执行计算）不需要了解目标是如何处理伤害的。以伤害为例，`Gameplay Effect`决定了造成多少伤害，然后`AttributeSet`（属性集）决定如何处理这些伤害。并非所有角色都拥有相同的`Attributes`，特别是当你使用继承自基类的`AttributeSet`时。基类的`AttributeSet`可能只包含生命值`Attributes`，而继承的`AttributeSet`子类可能还会添加护盾`Attributes`。具有护盾`Attributes`的子类`AttributeSet`会以不同于基类AttributeSet的方式来分配所受的伤害。

尽管使用`Meta Attributes`是一种很好的设计模式，但它并非强制性的。如果你的所有伤害实例都只使用同一个`Execution Calculation`（执行计算），并且所有角色共享同一个`AttributeSet`（属性集）类，那么你可能可以直接在`Execution Calculation`内部处理伤害分配至生命值、护盾等，并直接修改这些`Attributes`。这样做，你牺牲的只是灵活性，但这对你来说可能完全是可以接受的。

**[⬆ 回到顶部](#目录)**

#### 4.3.4 响应属性更改
为了监听`Attribute`的变化以便更新用户界面（UI）或其他游戏玩法，可以使用`UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)`函数。这个函数返回一个委托（delegate），你可以绑定这个委托，每当`Attribute`发生变化时，它会被自动调用。委托会提供一个`FOnAttributeChangeData`参数，其中包含了`NewValue`（新值）、`OldValue`（旧值）以及`FGameplayEffectModCallbackData`的信息。

**注意:** `FGameplayEffectModCallbackData`将仅在服务器上设置。

```c++  
AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(AttributeSetBase->GetHealthAttribute()).AddUObject(this, &AGDPlayerState::HealthChanged);  
```

```c++  
virtual void HealthChanged(const FOnAttributeChangeData& Data);  
```


示例项目中，通过绑定`GDPlayerState`上的`Attribute`值变化委托来更新HUD（抬头显示）并响应玩家死亡事件，即当生命值降至零时。

示例项目中包含了一个自定义的蓝图节点，它将上述功能封装成了一个`ASyncTask`。这个`ASyncTask`在`UI_HUD` UMG Widget中被用来更新生命值、魔法值和耐力值。一旦创建，这个`AsyncTask`将一直持续运行，直到手动调用`EndTask()`方法来结束任务，而这通常在UMG Widget的`Destruct`事件中完成。相关实现可以参考`AsyncTaskAttributeChanged.h/cpp`文件。

![Listen for Attribute Change BP Node](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/attributechange.png)

**[⬆ 回到顶部](#目录)**

#### 4.3.5 Derived Attributes（派生属性）
要创建一个其值部分或全部来源于一个或多个其他 `Attributes` 的 `Derived Attribute`（派生属性），可以使用带有一个或多个基于 `Attribute` 的或 [`MMC`](#concepts-ge-mmc)（Modified Modifier Magnitude Calculator，修正修饰符幅度计算器）[`Modifiers`](#concepts-ge-mods)  （修饰符） 的 `Infinite`（无限持续）`GameplayEffect`。当依赖的 `Attribute` 更新时，`Derived Attribute` 会自动随之更新。

对于 `Derived Attribute`（衍生属性）上的所有 `Modifiers`（修饰符），其最终计算公式与 
`Modifier Aggregators`（修饰符聚合器）所使用  的公式相同。如果需要按照特定顺序执行计算，那么应将所有计算逻辑置于一个 `MMC`（Modified Modifier Magnitude Calculator，修改后的修饰符幅度计算器）内部完成。

```C++
((CurrentValue + Additive) * Multiplicitive) / Division  
```

**注意:** 如果在PIE（Play In Editor，编辑器内播放）模式下使用多个客户端进行测试，你需要在编辑器偏好设置中禁用 `Run Under One Process` 选项。否则，当除第一个客户端以外的其他客户端上的独立 `Attributes` 更新时，`Derived Attributes`（衍生属性）将无法得到更新。

在这个例子中，我们应用了一个 `Infinite`（无限持续）的 `GameplayEffect`，用来根据公式 `TestAttrA = (TestAttrA + TestAttrB) * (2 * TestAttrC)` ，从 `Attributes TestAttrB` 和 `TestAttrC` 派生出 `TestAttrA` 的值。每当 `TestAttrB` 或 `TestAttrC` 中的任何一个属性更新其值时，  `TestAttrA` 都会自动重新计算其自身的值。

![Derived Attribute Example](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/derivedattribute.png)

**[⬆ 回到顶部](#目录)**

### 4.4 Attribute Set
#### 4.4.1 Attribute Set 定义
`AttributeSet` 用于定义、保存和管理对 `Attributes` 的更改。 开发人员应当从 [`UAttributeSet`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAttributeSet/index.html) 继承。 在 `OwnerActor` 的构造函数中创建一个 `AttributeSet` 会自动将其注册到对应的 `ASC` 中。**这项操作必须用 C++ 来完成。**

**[⬆ 回到顶部](#目录)**

#### 4.4.2 Attribute Set Design（设计）
一个 `ASC`（属性集组件）可以拥有一个或多个 `AttributeSet`。由于 `AttributeSet` 的内存开销非常小，因此使用多少个 `AttributeSet` 完全取决于开发者的组织决策。

可以接受的做法是为游戏中的每个 `Actor` 共享一个大型的单一 `AttributeSet`，并且仅在需要时使用其中的属性，而忽略那些未使用的属性。

或者，你可以选择拥有多个 `AttributeSet`，这些 `AttributeSet` 分别代表了根据需要选择性地添加到 `Actor` 上的不同属性组。例如，你可以有一个针对生命值相关的 `AttributeSet`，另一个针对法力值相关的 `AttributeSet`，等等。在一个 MOBA 游戏中，英雄可能需要法力值，而小兵则可能不需要。因此，英雄会获得法力值 `AttributeSet`，而小兵则不会。

此外，可以通过继承 `AttributeSet` 作为另一种选择性地决定 `Actor` 拥有哪些属性的方法。属性内部被称为 `AttributeSetClassName.AttributeName`。当你继承一个 `AttributeSet` 时，所有来自父类的属性仍然会以父类的名字作为前缀。

虽然你可以拥有多个 `AttributeSet`，但是你不应该在一个 `ASC` 上拥有同一个类的多个 `AttributeSet` 实例。如果你有来自同一类的多个 `AttributeSet`，系统将无法确定使用哪一个 `AttributeSet`，它只会随机选择其中一个。

##### 4.4.2.1 Subcomponents with Individual Attributes（带有个别属性的子组件）
在拥有多个可受伤害部件的 `Pawn` 的场景下，比如单独可受伤害的护甲部件，如果已知 `Pawn` 可能拥有的最大可受伤害部件数量，建议在单个 `AttributeSet` 中创建相应数量的生命值 `Attributes` —— 如 `DamageableCompHealth0`、`DamageableCompHealth1` 等——来表示这些可受伤害部件的逻辑“槽位”。在你的可受伤害部件类实例中，分配一个槽位编号 `Attribute`，这个编号可以被 `GameplayAbilities` 或者 [`Executions`](#concepts-ge-ec) （执行）读取，以便知道应该对哪个 `Attribute` 应用伤害。对于拥有少于最大数量或完全没有可受伤害部件的 `Pawns`，这种方式也是可行的。即使 `AttributeSet` 中存在某个 `Attribute`，也不意味着一定要使用它。未使用的 `Attributes` 占用的内存非常少。

如果你的子组件每个都需要大量的 `Attributes`，或者潜在的子组件数量没有上限，子组件可以脱离并被其他玩家使用（例如武器），或者出于任何其他原因这种方法不适用于你，我建议放弃使用 `Attributes`，而是直接在组件上存储普通的浮点数。有关详细信息，请参阅 [Item Attributes](#concepts-as-design-itemattributes) （物品属性）。

##### 4.4.2.2 在运行时添加和删除 AttributeSet
`AttributeSet` 可以在运行时添加到或从 `ASC` 中移除；然而，移除 `AttributeSet` 可能是危险的。例如，如果客户端上的 `AttributeSet` 在服务器之前被移除，并且有一个属性值变化被复制到客户端，那么该属性将找不到其对应的 `AttributeSet`，从而导致游戏崩溃。

关于武器添加到库存：

```c++  
AbilitySystemComponent->GetSpawnedAttributes_Mutable().AddUnique(WeaponAttributeSetPointer);  
AbilitySystemComponent->ForceReplication();  
```

关于从库存中移除武器：
```c++  
AbilitySystemComponent->GetSpawnedAttributes_Mutable().Remove(WeaponAttributeSetPointer);  
AbilitySystemComponent->ForceReplication();  
```

##### 4.4.2.3 Item Attributes (Weapon Ammo)【物品属性（武器弹药)】
有几种方法可以实现带有 `Attributes` 的可装备物品（如武器弹药、护甲耐久度等）。所有这些方法都将值直接存储在物品上。这是对于在其生命周期内可以被多个玩家装备的物品来说必要的做法。

实现可装备物品属性的方法：
1. 在物品上使用普通浮点数（**推荐**）
	+ 直接在物品上存储普通的浮点数值，这样可以避免与 `AttributeSet` 和 `ASC` 相关的复杂性。
2. 在物品上使用单独的 `AttributeSet`
	+ 为每个物品创建一个单独的 `AttributeSet`，并将其附加到物品上。这种方法增加了灵活性，但同时也引入了更多的管理负担。****
3. 在物品上使用单独的 `ASC`
	+ 为每个物品配备一个单独的 `ASC`，并在其中管理物品的所有属性。这种方法提供了最大的灵活性，但也可能导致更复杂的架构和更高的资源消耗。

###### 4.4.2.3.1 在物品上使用普通浮点数
使用普通浮点数代替 `Attributes`，而不是使用 `Attributes`，可以在物品类实例上存储普通的浮点数值。例如，《堡垒之夜》（Fortnite）和 [GASShooter](https://github.com/tranek/GASShooter) 就是这样处理枪支弹药的。对于枪支，可以直接将最大弹匣容量、当前弹匣内的弹药数量、备用弹药等作为复制的浮点数 (`COND_OwnerOnly`) 存储在枪支实例上。如果武器共享备用弹药，则可以将备用弹药移动到角色上作为一个 `Attribute`，存储在共享的弹药 `AttributeSet` 中（重新装填能力可以使用 `Cost GE` 从备用弹药中提取到枪支的浮点数弹匣弹药中）。因为你没有使用 `Attributes` 来处理当前弹匣中的弹药，所以你需要在 `UGameplayAbility` 中重写一些函数来检查和应用枪支上的浮点数值的成本。在授予能力时，将枪支设置为 [`GameplayAbilitySpec`](https://github.com/tranek/GASDocumentation#concepts-ga-spec) 中的 `SourceObject`，这意味着在能力内部你可以访问到授予该能力的枪支。

为了防止枪支在自动射击期间回传弹药量并覆盖本地弹药量，可以在 `PreReplication()` 函数中当玩家具有 `IsFiring` `GameplayTag` 时禁用弹药量的复制。这样做实际上是在执行自己的本地预测。

```c++  
void AGSWeapon::PreReplication(IRepChangedPropertyTracker& ChangedPropertyTracker)  
{  
    Super::PreReplication(ChangedPropertyTracker);  
    DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, 
                                  PrimaryClipAmmo, 
                                  (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag))
    );    
    DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, SecondaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));
}  
```

优点：
1. 避免使用 `AttributeSets`的限制（见下文）

限制：
1. 无法使用现有的 `GameplayEffect` 工作流程
	+ 不能使用现有的 `GameplayEffect` 流程（如使用弹药时的 Cost GEs 等）。
2. 需要重写 `UGameplayAbility` 的关键函数
	+ 需要进行额外工作来重写 `UGameplayAbility` 中的关键函数，以检查和应用枪支浮点数值的弹药成本。

###### 4.4.2.3.2 在物品上使用 `AttributeSet`
使用一个单独的 `AttributeSet` 在物品上，并在[将物品添加到玩家的库存时将其 添加到玩家的 `ASC` 中](#concepts-as-design-addremoveruntime)，这种方法是可以工作的，但它有一些重要的限制。我在早期版本的 [GASShooter](https://github.com/tranek/GASShooter) 中为武器弹药实现了这种方法。武器将其`Attributes`（如最大弹匣容量、当前弹匣中的弹药数量、备用弹药等）存储在一个位于武器类上的 `AttributeSet` 中。如果武器共享备用弹药，你可以将备用弹药移到角色上的共享弹药 `AttributeSet` 中。当武器在服务器上被添加到玩家的库存时，武器会将其 `AttributeSet` 添加到玩家的 `ASC::SpawnedAttributes` 中。服务器随后会将这些信息复制到客户端。如果武器从库存中移除，它会从 `ASC::SpawnedAttributes` 中移除其 `AttributeSet`。

当你将 `AttributeSet` 存储在 `OwnerActor` 之外的对象上（比如武器）时，最初会在 `AttributeSet` 中遇到一些编译错误。解决方法是在 `BeginPlay()` 中构造 `AttributeSet` 而不是在构造函数中，并在武器上实现 `IAbilitySystemInterface`（在将武器添加到玩家库存时设置指向 `ASC` 的指针）。

```c++  
void AGSWeapon::BeginPlay()  
{  
    if (!AttributeSet)    
    {       
        AttributeSet = NewObject<UGSWeaponAttributeSet>(this);    
    }    
    //...
}  
```

你可以通过查看这个 [旧版GASShooter](https://github.com/tranek/GASShooter/tree/df5949d0dd992bd3d76d4a728f370f2e2c827735) 来了解具体实现

优点：
1. 可以使用现有的 `GameplayAbility` 和 `GameplayEffect` 工作流程
	+ 可以利用现有的 `GameplayEffect` 流程（如使用弹药时的 `Cost GEs` 等）。
2. 对于非常小的物品集合设置简单

限制：
1. 为每种武器类型都需要创建一个新的 `AttributeSet` 类：
	+ `ASCs` 只能功能上有每个类的一个 `AttributeSet` 实例，因为对某个 `Attribute` 的更改会查找 `ASCs` 的 `SpawnedAttributes` 数组中该 `AttributeSet` 类的第一个实例。同一 `AttributeSet` 类的其他实例会被忽略。
2. 玩家库存中只能拥有每种类型的武器各一件：
	+ 由于上述原因，即每个 `AttributeSet` 类只能有一个实例，因此玩家库存中只能拥有每种类型的武器各一件。
3. 移除 `AttributeSet` 是危险的：
	+ 在 GASShooter 中，如果玩家因火箭而自杀，玩家会立即从他的库存中移除火箭发射器（包括从 `ASC` 中移除其 `AttributeSet`）。当服务器复制火箭发射器的弹药 `Attribute` 发生变化时，客户端的 `ASC` 上已经不存在该 `AttributeSet`，这会导致游戏崩溃。

###### 4.4.2.3.3 物品上的 `ASC`
在每个物品上放置整个 `AbilitySystemComponent` 是一种极端的做法。我个人没有这样做过，也没有见过实际项目中采用这种方式。要使其正常工作，需要大量的工程设计和实现。

>译者注：这种方法的主要考虑因素包括：
>		1.资源消耗：每个 ASC 都会占用一定的内存和计算资源。
>		2. 复杂性：每个物品都有自己的 ASC 会增加系统的整体复杂性。
>		3. 维护成本：随着物品数量的增加，维护每个物品的 ASC 及其相关逻辑将会变得更加困难。
尽管这种方法提供了高度的灵活性和定制性，但在大多数情况下，使用更简单的方案（如直接在物品上存储浮点数值或使用单独的 AttributeSet）可能是更好的选择。

>在具有相同所有者但不同化身的多个对象上放置多个 AbilitySystemComponent是否可行？

>在具有相同所有者但不同化身的对象（例如，在棋子和武器/物品/投射物上，所有者设置为 PlayerState）上放置多个 AbilitySystemComponent 是否可行？

可行性分析：
1. 实现挑战：
	+ 实现在所有者演员上的 IGameplayTagAssetInterface 和 IAbilitySystemInterface 可能会面临一些挑战。
		+ IGameplayTagAssetInterface：聚合来自所有 ASC 的标签可能可行，但需要注意 -HasAllMatchingGameplayTags 可能需要跨 ASC 聚合来满足。
		+ IAbilitySystemInterface：确定哪个 ASC 是权威的以及如何处理 GameplayEffect 的应用更为复杂。
2. 潜在解决方案：
	+ 为不同的对象（如Pawn和武器）分别实现 IGameplayTagAssetInterface 和 IAbilitySystemInterface，并定义一套规则来决定哪些 GameplayEffect 应用于哪个 ASC。
	+ 可能需要自定义逻辑来协调多个 ASC 之间的交互，确保一致性和正确性。
3. 独立 ASC 的优势：
	+ 单独在棋子和武器上放置 ASC 可以更好地区分描述武器的标签与描述拥有棋子的标签。
	+ 授予武器的标签也可以“适用于”所有者，而不适用于其他任何东西，这样可以在一定程度上保持属性和 GameplayEffects 的独立性，同时让所有者聚合拥有的标签。
4. 复杂性考量：
	+ 拥有多个具有相同所有者的 ASC 可能会引入额外的复杂性，特别是在处理 GameplayEffect 的应用和同步方面。

>结论：虽然理论上可行，但在实践中需要仔细设计和实现来解决多 ASC 的协调问题。这可能会带来额外的开发成本和技术挑战。

>我看到的第一个问题是实现在所有者演员上的 IGameplayTagAssetInterface 和 IAbilitySystemInterface。

+ IGameplayTagAssetInterface：聚合来自所有 ASC 的标签可能是可行的（但需要注意 -HasAllMatchingGameplayTags 可能仅通过跨 ASC 聚合来满足。仅仅将调用转发给每个 ASC 并将结果合并在一起是不够的）。
+ IAbilitySystemInterface：这个问题更为棘手：哪个 ASC 是权威的？如果有人想要应用一个 GameplayEffect —— 应该将其应用到哪个 ASC 上？也许你可以解决这些问题，但这个问题中最难的部分是所有者将拥有多个 ASC。
解决方案探讨
1. 聚合标签：
	+ 对于 IGameplayTagAssetInterface，可以通过遍历所有 ASC 来聚合标签。需要注意的是 -HasAllMatchingGameplayTags 的实现可能需要跨 ASC 聚合来确保正确性。这意味着需要自定义逻辑来处理标签的查询，确保所有相关的 ASC 都被考虑到。
2. 确定权威 ASC：
	+ 对于 IAbilitySystemInterface，需要定义一个策略来确定哪个 ASC 是权威的。一种可能的方法是为特定类型的 GameplayEffect 定义一个优先级列表，根据这个列表来决定哪个 ASC 应该接收 GameplayEffect。
	+ 另一种方法是根据 GameplayEffect 的来源和目的来决定应该应用到哪个 ASC。例如，如果 GameplayEffect 来源于武器，则应用到武器的 ASC；如果是全局效果，则应用到棋子的 ASC。
3. 协调多个 ASC：
	+ 需要自定义逻辑来协调多个 ASC 之间的交互，确保一致性和正确性。这可能涉及到定义一套规则来决定哪些 GameplayEffect 应用于哪个 ASC，以及如何处理这些 ASC 之间的数据同步。

>结论：虽然理论上可行，但在实践中需要仔细设计和实现来解决多 ASC 的协调问题。这可能会带来额外的开发成本和技术挑战。

Dave Ratti 从 Epic 的回答关于 [community questions #6](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)

---

我们列出了八个问题：
1. 我们如何根据需要在 GameplayAbilities 之外或与 GameplayAbilities 无关的情况下创建范围预测窗口？例如，当一个发射后不管的射弹击中敌人时，如何在本地预测 GameplayEffect 造成的伤害？

PredictionKey 系统实际上并不是为此而设计的。从根本上讲，该系统的工作原理是客户端启动预测操作，使用密钥将其告知服务器，然后客户端和服务器都运行相同的操作并将预测副作用与给定的预测密钥相关联。例如，“我正在预测性地激活一项能力”或“我已经生成了目标数据，并将在 WaitTargetData 任务之后预测性地运行能力图的一部分”。

使用这种模式，PredictionKey 从服务器“反弹”并通过 UAbilitySystemComponent::ReplicatedPredictionKeyMap（复制属性）返回到客户端。一旦密钥从服务器复制回来，客户端就可以撤消所有本地预测副作用（GameplayCues、GameplayEffects）：复制的版本*将存在*，如果不存在，则为错误预测。准确知道​​何时撤消预测副作用在这里至关重要：如果太早，您会看到间隙，如果太晚，您将得到“双重”。 （请注意，这是指状态预测，例如基于持续时间的游戏效果的循环 GameplayCue。“突发”GameplayCues 和即时游戏效果永远不会“撤消”或回滚。如果与它们关联的预测密钥，它们只会在客户端上被跳过）。

为了进一步说明这一点：至关重要的是，预测操作是服务器不会自行执行的操作，而只有在客户端告诉它们时才会执行。因此，除非“某事”是服务器在客户端通知后才会执行的操作，否则通用的“按需创建密钥并告知服务器以便我可以运行某事”是行不通的。

回到最初的问题：类似于发射后就不管的射弹。Paragon 和 Fornite 都有使用 GameplayCues 的射弹演员类。但是，我们不使用预测密钥系统来执行这些操作。相反，我们有一个关于非复制 GameplayCues 的概念。GameplayCues 只是在本地触发并被服务器完全跳过。本质上，所有这些都是直接调用 UGameplayCueManager::HandleGameplayCue。它们不通过 UAbilitySystemComponent 路由，因此不会进行预测密钥检查/早期返回。

非复制 GameplayCues 的缺点是，它们不会被复制。因此，这取决于射弹类/蓝图，以确保调用这些函数的代码路径在每个人身上都运行。我们有启动提示（在 BeginPlay 中调用）、爆炸、击中墙壁/角色等。

这些类型的事件已经在客户端生成，因此调用非复制的游戏提示并不是什么大问题。复杂的蓝图可能很棘手，作者需要确保他们了解在哪里运行什么。

2. 当使用 WaitNetSync AbilityTask 和 OnlyServerWait 在本地预测的 GameplayAbility 中创建范围预测窗口时，由于服务器正在等待带有预测密钥的 RPC，玩家是否可以通过延迟向服务器发送数据包来控制 GameplayAbility 时间，从而进行作弊？这在 Paragon 或 Fortnite 中是否曾出现过问题？如果是，Epic 做了什么来解决这个问题？

是的，这是一个合理的担忧。在服务器上运行的任何等待客户端“信号”的能力蓝图都可能容易受到延迟切换类型漏洞的攻击。

Paragon 有一个类似于 UAbilityTask_WaitTargetData 的自定义定位任务。在这个任务中，我们有超时或“最大延迟”，我们会在客户端等待即时定位模式。如果定位模式正在等待用户确认（按下按钮），那么它将被忽略，因为用户可以慢慢来。但是对于即时确认目标的技能，我们只会等待一段时间，然后 A) 在服务器端生成目标数据 B) 取消该技能。

我们从未有过 WaitNetSync 这样的机制，我们很少使用。

不过，我认为 Fortnite 不会使用任何类似的东西。Fortnite 中的武器技能是特殊情况，批量处理到单个 Fortnite 特定的 RPC：一个 RPC 用于激活技能、提供目标数据和结束技能。因此，在 Battle Royale 中，武器技能本质上不会受到此影响。

我的看法是，这可能是可以在整个系统范围内解决的问题，但我认为我们不会很快做出改变。针对您提到的情况，对 WaitNetSync 进行局部修复以包含最大延迟可能是一项合理的任务，但同样，我们不太可能在不久的将来在我们这边做到这一点。

3. Paragon 和 Fortnite 使用了哪种 EGameplayEffectReplicationMode，Epic 对何时使用它们有什么建议？

这两款游戏基本上都对玩家控制的角色使用混合模式，对 AI 控制的角色（AI 小兵、丛林小兵、AI Husk 等）使用最小模式。这是我建议大多数人在多人游戏中使用该系统的方式。越早设置这些，越好。

Fortnite 在优化方面更进一步。它实际上根本不为模拟代理复制 UAbilitySystemComponent。在拥有 Fortnite 玩家状态类的 ::ReplicateSubobjects() 中跳过组件和属性子对象。我们确实将能力系统组件中的最低限度复制数据推送到 pawn 本身的结构（基本上是属性值的子集和我们在位掩码中复制的标签白名单子集）。我们称之为“代理”。在接收端，我们获取在 pawn 上复制的代理数据，并将其推回到玩家状态的能力系统组件中。因此，FNBR 中确实为每个玩家提供了一个 ASC，但它并不直接复制：而是通过 pawn 上的最小代理结构复制数据，然后路由回接收端的 ASC。这是有优势的，因为它 A) 是一组更简单的数据 B) 利用了 pawn 相关性。

我不确定自那时以来已经完成的其他服务器端优化（复制图等）是否仍然有必要，而且它不是最易于维护的模式。

4. 由于我们无法根据 GameplayPrediction.h 预测 GameplayEffects 的移除，是否有任何策略可以减轻延迟对移除 GameplayEffects 的影响？例如，当移除移动速度慢的元素时，我们目前必须等待服务器复制 GameplayEffect 移除，从而导致玩家角色位置的突然变化。

这是一个棘手的问题，我没有好的答案。我们通常通过容差和平滑来绕过这些问题。我完全同意能力系统和与角色移动系统的精确同步并不好，我们确实想修复它。

我有一个允许预测移除 GE 的架子，但在继续前进之前永远无法解决所有边缘情况。但这并不能解决所有问题，因为角色移动仍然有一个内部保存的移动缓冲区，它对能力系统和可能的移动速度修改器等一无所知。即使无法预测 GE 的移除，仍然有可能进入校正反馈循环。

如果你认为你的情况确实很危急，你可以预测性地添加一个 GE，这会抑制你的移动速度 GE。我自己从来没有这样做过，但之前曾对此进行过理论研究。它可能能够帮助解决某一类问题。

5. 我们知道，AbilitySystemComponent 存在于 Paragon 和 Fortnite 中的 PlayerState 上，以及 Action RPG Sample 中的角色上。Epic 对于 AbilitySystemComponent 应该存在于何处的内部规则、指南或建议是什么——它的所有者应该是谁？

一般来说，我认为任何不需要重生的东西都应该有相同的所有者和 Avatar 角色。任何东西，如 AI 敌人、建筑物、世界道具等。

任何重生的东西都应该有不同的所有者和 Avatar，这样 Ability 系统组件就不需要在重生后保存/重新创建/恢复。

PlayerState 是合乎逻辑的选择，它会复制到所有客户端（而 PlayerController 则不是）。

缺点是 PlayerState 总是相关的，所以你可能会在 100 个玩家的游戏中遇到问题。（请参阅问题 #3 中 FN 所做的事情的注释）。

6. 是否有可能让多个 AbilitySystemComponents 拥有相同的所有者但拥有不同的化身（例如，在 pawn 和武器/物品/射弹上，所有者设置为 PlayerState）？

我看到的第一个问题是在拥有者 Actor 上实现 IGameplayTagAssetInterface 和 IAbilitySystemInterface。前者可能是可行的：只需聚合所有 ASC 中的标签（但要注意 - HasAlMatchingGameplayTags 只能通过跨 ASC 聚合来满足。仅仅将该调用转发给每个 ASC 并将结果一起进行 OR 是不够的）。但后者更加棘手：哪个 ASC 是权威的？如果有人想应用 GE - 哪一个应该接收它？也许你可以解决这些问题，但问题的这一方面将是最困难的：所有者将在其下拥有多个 ASC。

不过，将棋子和武器上的 ASC 分开是有意义的。例如，区分描述武器的标签和描述拥有棋子的标签。也许赋予武器的标签也“适用于”所有者，而不适用于其他任何东西，这确实有意义（例如，属性和 GE 是独立的，但所有者将像我上面描述的那样汇总拥有的标签）。我相信这可以奏效。但拥有同一个所有者的多个 ASC 可能会变得很危险。

7. 有没有办法阻止服务器覆盖拥有客户端上本地预测技能的冷却时间？在高延迟的情况下，当本地冷却时间到期但服务器上仍处于冷却状态时，这将允许拥有客户端“尝试”再次激活该技能。当拥有客户端的激活请求通过网络到达服务器时，服务器可能已冷却完毕，或者服务器可能能够将激活请求排队等待剩余的毫秒数。否则，延迟较高的客户端在重新激活技能之前会比延迟较低的客户端有更长的延迟。这在冷却时间非常短的技能（例如冷却时间可能不到一秒的基本攻击）中最为明显。如果没有办法阻止服务器覆盖本地预测技能的冷却时间，那么 Epic 减轻高延迟对重新激活技能的影响的策略是什么？换个例子来说，Epic 如何设计 Paragon 的基本攻击和其他能力，以便高延迟玩家能够以与低延迟玩家相同的速度通过本地预测进行攻击或激活？

简而言之，没有办法阻止这种情况，Paragon 肯定存在问题。
更高延迟的连接在基本攻击时 ROF 会更低。

我试图通过添加“GE 协调”来解决这个问题，在计算 GE 持续时间时会考虑延迟。本质上允许服务器占用部分总 GE 时间，以便 GE 客户端的有效时间与任何延迟量 100% 一致（尽管波动仍然可能导致问题）。然而，我从来没有让它处于可以发货的状态，项目进展很快，我们从来没有完全解决这个问题。

Fortnite 自己记录武器的射击率：它不使用 GE 来冷却武器。如果这对您的游戏来说是一个严重问题，我会推荐它。

8. Epic 的 GameplayAbilitySystem 插件路线图是什么？
Epic 计划在 2019 年及以后添加哪些功能？
我们觉得目前系统总体上相当稳定，我们没有人负责开发主要的新功能。偶尔会针对 Fortnite 或
UDN/pull 请求进行错误修复和小改进，但目前就是这样。
从长远来看，我认为我们最终会推出“V2”或进行一些重大更改。我们从编写这个系统中学到了很多东西，觉得我们做对了很多，也做错了很多。我很想有机会纠正
这些错误并改进上面指出的一些致命缺陷。
如果 V2 真的要推出，提供升级路径将是至关重要的。我们永远不会制作 V2 并永远将 Fortnite 留在 V1 上：将有一些路径或程序会尽可能自动迁移，尽管几乎肯定仍需要手动重新制作。

优先修复将是：
● 与角色移动系统的更好互操作性。统一客户端预测。
● GE 移除预测（问题 #4）
● GE 延迟协调（问题 #8）
● 通用网络优化，例如批处理 RPC 和代理结构。大部分是我们为 Fortnite 所做的工作，但找到了将其分解为更通用形式的方法，至少这样游戏就可以更轻松地编写自己的游戏特定优化。

我会考虑进行更一般的重构类型的更改：
● 我想从根本上摆脱让 GE 直接引用
电子表格值的做法，相反，它们将能够发出参数，并且这些
参数可以由与电子表格
值绑定的某个更高级别的对象填充。当前模型的问题在于，由于 GE 与曲线表行紧密耦合，因此无法共享。我认为可以编写一个通用的参数化系统，并将其作为 V2 系统的基础。
● 减少 UGameplayAbility 上的“策略”数量。我会删除 ReplicationPolicy
InstancingPolicy。在我看来，Replication 几乎从未真正需要过，而且会造成
混乱。应该通过将
FGameplayAbilitySpec 设为可以子类化的 UObject 来替换 InstancingPolicy。这应该是具有事件且可蓝图化的
“非实例化能力对象”。
UGameplayAbility 应该是“每次执行时实例化”的对象。如果您需要实际实例化，它可以是可选的：相反，“非实例化”能力将通过新的 UGameplayAbilitySpec 对象实现。
● 系统应提供更多“中级”构造，例如“过滤的 GE 应用程序容器”（数据驱动将哪些 GE 应用于具有更高级别游戏逻辑的哪些参与者）、“重叠体积支持”（根据碰撞原始重叠事件应用“过滤的 GE 应用程序容器”）等。这些是每个项目最终以自己的方式实现的构建块。正确处理它们并非易事，因此我认为我们应该更好地提供一些基本实现。
● 总体而言，减少启动和运行项目所需的样板。可能是一个单独的模块“Ex 库”或任何可以提供开箱即用的被动能力或基本命中扫描武器之类的东西。此模块是可选的，但可以让你快速启动并运行。
● 我想将 GameplayCues 移至与能力系统不耦合的单独模块。我认为这里可以做出很多改进。

这只是我的个人意见，不代表任何人的承诺。我认为最现实的行动方案是，随着新引擎技术计划的实施，能力系统将需要更新，那时就是做这种事情的时候。这些计划可能与脚本、网络或物理/角色运动有关。不过，这一切都是长远考虑，所以我无法对时间表做出承诺或估计。

---

优点：
1. 可以利用现有的 `GameplayAbility` 和 `GameplayEffect` 工作流程，例如使用 `Cost GEs` 来处理弹药消耗等。
2. 可以在每个武器的 `AbilitySystemComponent (ASC)` 上重用 `AttributeSet` 类。

限制：
1. 未知的工程成本
2. 可行性问题

**[⬆ 回到顶部](#目录)**

#### 4.4.3 Defining Attributes
**`Attributes` 必须在 `AttributeSet` 的头文件中定义**。建议在每个 `AttributeSet` 头文件的顶部添加以下宏块。这将自动为您的 `Attributes` 生成 getter 和 setter 函数。
```c++  
// Uses macros from AttributeSet.h  
#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \  
    GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \    
    GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \    
    GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \    
    GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)  
```

一个复制的健康属性可以像下面这样定义：

```c++  
UPROPERTY(BlueprintReadOnly, Category = "Health", ReplicatedUsing = OnRep_Health)  
FGameplayAttributeData Health;  
ATTRIBUTE_ACCESSORS(UGDAttributeSetBase, Health)  
```

还要在头文件中定义 `OnRep` 函数：

```c++  
UFUNCTION()  
virtual void OnRep_Health(const FGameplayAttributeData& OldHealth);  
```

`AttributeSet` 的 .cpp 文件应该使用预测系统使用的 `GAMEPLAYATTRIBUTE_REPNOTIFY` 宏填充 `OnRep` 函数：

```c++  
void UGDAttributeSetBase::OnRep_Health(const FGameplayAttributeData& OldHealth)  
{  
    GAMEPLAYATTRIBUTE_REPNOTIFY(UGDAttributeSetBase, Health, OldHealth);
}  
```

最后，需要将 `Attribute` 添加到 `GetLifetimeReplicatedProps`：

```c++  
void UGDAttributeSetBase::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const  
{  
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);  
    DOREPLIFETIME_CONDITION_NOTIFY(UGDAttributeSetBase, Health, COND_None, REPNOTIFY_Always);
}  
```


`REPNOTIFY_Always` 指示 `OnRep` 函数在本地值已经等于从服务器下传的值时触发（由于预测）。默认情况下，如果本地值与从服务器下传的值相同，则不会触发 `OnRep` 函数。

如果 `Attribute` 不像 `Meta Attribute` 那样进行复制，那么可以跳过 `OnRep` 和 `GetLifetimeReplicatedProps` 这两个步骤。

**[⬆ 回到顶部](#目录)**

#### 4.4.4 Initializing Attributes
有多种方法可以初始化 `Attributes`（设置它们的 `BaseValue` 以及因此而设置的 `CurrentValue` 为某个初始值）。Epic 推荐使用即时 `GameplayEffect`。这也是样本项目中采用的方法。

参见样本项目中的 `GE_HeroAttributes` 蓝图，了解如何创建一个即时 `GameplayEffect` 来初始化 `Attributes`。这个 `GameplayEffect` 的应用发生在 C++ 中。

如果你在定义 `Attributes` 时使用了 `ATTRIBUTE_ACCESSORS` 宏，那么对于每个 `Attribute`，都会自动生成一个初始化函数在 `AttributeSet` 上，你可以在 C++ 中随时调用这些函数。

```c++  
// `InitHealth(float InitialValue)` 是为使用 `ATTRIBUTE_ACCESSORS` 宏定义的 `Attribute` `'Health'` 自动生成的初始化函数。  
AttributeSet->InitHealth(100.0f);  
```

参见 `AttributeSet.h` 以了解更多初始化 `Attributes` 的方法。

**注意：** 在 4.24 之前，`FAttributeSetInitterDiscreteLevels` 无法与 `FGameplayAttributeData` 一起工作。它是在 `Attributes` 作为原始浮点数时创建的，并且会抱怨 `FGameplayAttributeData` 不是“纯旧数据”（`POD`）。这个问题在 4.24 版本中已修复 [链接](https://issues.unrealengine.com/issue/UE-76557)。

**[⬆ 回到顶部](#目录)**

#### 4.4.5 PreAttributeChange()
`PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue)` 是 `AttributeSet` 中的主要函数之一，用于在更改发生前响应 `Attribute` 的 `CurrentValue` 变化。这是通过引用参数 `NewValue` 来限制传入的 `CurrentValue` 变化的理想位置。

例如，为了限制移动速度修正值，样本项目是这样做的：

```c++  
if (Attribute == GetMoveSpeedAttribute())  
{  
    // 减速不能少于 150 个单位，也不能加速超过 1000 个单位    
    NewValue = FMath::Clamp<float>(NewValue, 150, 1000);
}  
```

`GetMoveSpeedAttribute()` 函数是由我们添加到 `AttributeSet.h` 中的宏块创建的（[定义属性](#concepts-as-attributes)）。

这会在任何 `Attribute` 发生变化时触发，无论是使用 `Attribute` 设置器（由 `AttributeSet.h` 中的宏块定义（[定义属性](#concepts-as-attributes)））还是使用 [`GameplayEffects`](#concepts-ge)。

**注意：** 在这里进行的任何限制都不会永久改变 `ASC` 上的修正值。它只会改变查询修正值时返回的值。这意味着任何重新计算所有修正值得到的 `CurrentValue` 的操作，如使用 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 和 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc)，都需要再次实现限制。

**注意：** Epic 对 `PreAttributeChange()` 的注释说明不建议在此处处理游戏玩法事件，而是主要用于限制。推荐处理 `Attribute` 变化时的游戏玩法事件的位置是 `UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)`（[响应属性变化](#concepts-a-changes)）。

**[⬆ 回到顶部](#目录)**

#### 4.4.6 PostGameplayEffectExecute()
`PostGameplayEffectExecute(const FGameplayEffectModCallbackData & Data)` 只在瞬时 `GameplayEffect` 引起 `Attribute` 的 `BaseValue` 发生变化后触发。当 `Attribute` 从 `GameplayEffect` 发生变化时，这是一个有效的进一步操作 `Attribute` 的位置。

例如，在样本项目中，我们会在这里从健康 `Attribute` 中减去最终伤害 `Meta Attribute`。如果有护盾 `Attribute`，我们会在从健康中减去剩余伤害之前先从中减去伤害。样本项目还利用此位置来应用被击反应动画、显示浮动伤害数字、以及给击杀者分配经验值和金币奖励。按照设计，伤害 `Meta Attribute` 总是通过瞬时 `GameplayEffect` 传递，而不会通过 `Attribute` 设置器。

其他仅通过瞬时 `GameplayEffects` 改变其 `BaseValue` 的 `Attributes`，如法力值和耐力值，也可以在这里限制到它们的最大值对应的 `Attributes`。

**注意：** 当 `PostGameplayEffectExecute()` 被调用时，对 `Attribute` 的更改已经发生，但尚未向客户端复制，因此在这里限制值不会导致两次网络更新。客户端只会在限制之后收到更新。

**[⬆ 回到顶部](#目录)**

#### 4.4.7 OnAttributeAggregatorCreated()
`OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator)` 在为此集合中的 `Attribute` 创建 `Aggregator` 时触发。它允许自定义设置 [`FAggregatorEvaluateMetaData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FAggregatorEvaluateMetaData/index.html)。`AggregatorEvaluateMetaData` 由 `Aggregator` 使用来根据应用于它的所有 [`Modifiers`](#concepts-ge-mods) 评估 `Attribute` 的 `CurrentValue`。默认情况下，`AggregatorEvaluateMetaData` 仅由 `Aggregator` 使用来确定哪些 `Modifiers` 符合条件，例如 `MostNegativeMod_AllPositiveMods` 允许所有正向 `Modifiers` 但限制负向 `Modifiers` 仅为最负的那个。Paragon 就使用了这种方法，只允许最负的移动速度减速效果应用于玩家，无论他们身上有多少减速效果，同时应用所有正向的移动速度增益。不符合条件的 `Modifiers` 仍然存在于 `ASC` 上，只是没有被聚合到最终的 `CurrentValue` 中。一旦条件发生变化，它们有可能符合条件，例如如果最负的 `Modifier` 到期，下一个最负的 `Modifier`（如果存在的话）就会符合条件。

要在只允许最负 `Modifier` 和所有正向 `Modifiers` 的例子中使用 `AggregatorEvaluateMetaData`：

```c++  
virtual void OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const override;  
```  
  
```c++  
void UGSAttributeSetBase::OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const  
{  
    Super::OnAttributeAggregatorCreated(Attribute, NewAggregator);  
    if (!NewAggregator)    
    {       
        return;    
    }  
    
    if (Attribute == GetMoveSpeedAttribute())    
    {       
        NewAggregator->EvaluationMetaData = &FAggregatorEvaluateMetaDataLibrary::MostNegativeMod_AllPositiveMods;    
    }
}  
```

你应该将自定义的 `AggregatorEvaluateMetaData` 作为静态变量添加到 `FAggregatorEvaluateMetaDataLibrary` 中。

**[⬆ 回到顶部](#目录)**

### 4.5 Gameplay Effects

#### 4.5.1 Gameplay Effect Definition
[`GameplayEffects`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffect/index.html) (`GE`) 是能力通过自身和其他对象改变 [`Attributes`](#concepts-a) 和 [`GameplayTags`](#concepts-gt) 的载体。它们可以引起立即的 `Attribute` 变化，如伤害或治疗，或者施加长期的状态增益/减益，如移动速度提升或眩晕。`UGameplayEffect` 类旨在成为一个 **仅数据** 类，用来定义单一的游戏效果。不应向 `GameplayEffects` 添加额外的逻辑。通常设计师会创建许多 `UGameplayEffect` 的蓝图子类。

`GameplayEffects` 通过 [`Modifiers`](#concepts-ge-mods) 和 [`Executions` (`GameplayEffectExecutionCalculation`)](#concepts-ge-ec) 来改变 `Attributes`。

`GameplayEffects` 有三种持续时间类型：`Instant`（瞬时）、`Duration`（持续时间）和 `Infinite`（无限）。

此外，`GameplayEffects` 还可以添加/执行 [`GameplayCues`](#concepts-gc)。一个 `Instant`（瞬时）`GameplayEffect` 将会调用 `GameplayCue` 的 `GameplayTags` 的 `Execute` 方法，而 `Duration`（持续时间）或 `Infinite`（无限）`GameplayEffect` 将会调用 `GameplayCue` 的 `GameplayTags` 的 `Add` 和 `Remove` 方法。

| 持续时间类型     | GameplayCue 事件 | 何时使用                                                                                                                                   |
| ---------- | -------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `Instant`  | Execute        | 用于对 `Attribute` 的 `BaseValue` 进行立即且永久的更改。`GameplayTags` 不会被应用，即使是一帧的时间也不会。                                                             |
| `Duration` | 添加 & 移除        | 用于对 `Attribute` 的 `CurrentValue` 进行临时更改，并应用将在 `GameplayEffect` 到期或手动移除时移除的 `GameplayTags`。持续时间在 `UGameplayEffect` 类/蓝图中指定。             |
| `Infinite` | 添加 & 移除        | 用于对 `Attribute` 的 `CurrentValue` 进行临时更改，并应用将在 `GameplayEffect` 被移除时移除的 `GameplayTags`。这些 `GameplayEffects` 本身永远不会到期，必须由能力或 `ASC` 手动移除。 |

`Duration` 和 `Infinite` `GameplayEffects` 有一个选项可以应用 `Periodic Effects`，这些效果会每隔 `X` 秒（由其 `Period` 定义）应用其 `Modifiers` 和 `Executions`。`Periodic Effects` 在更改 `Attribute` 的 `BaseValue` 和 `Executing` `GameplayCues` 方面被视为 `Instant` `GameplayEffects`。这对于随时间造成伤害（DOT）类型的效果很有用。**注意：** `Periodic Effects` 不能被 [预测](#concepts-p)。

`Duration` 和 `Infinite` `GameplayEffects` 可以在应用后暂时关闭和开启，如果它们的 `Ongoing Tag Requirements`（[Gameplay Effect Tags](#concepts-ge-tags)）不满足/满足的话。关闭 `GameplayEffect` 会移除其 `Modifiers` 和已应用的 `GameplayTags` 的效果，但不会移除 `GameplayEffect` 本身。重新开启 `GameplayEffect` 会重新应用其 `Modifiers` 和 `GameplayTags`。

如果你需要手动重新计算 `Duration` 或 `Infinite` `GameplayEffect` 的 `Modifiers`（例如你有一个 `MMC` 使用的数据不是来自 `Attributes`），你可以调用 `UAbilitySystemComponent::ActiveGameplayEffects.SetActiveGameplayEffectLevel(FActiveGameplayEffectHandle ActiveHandle, int32 NewLevel)`，其中 `NewLevel` 与它当前的级别相同，可以通过 `UAbilitySystemComponent::ActiveGameplayEffects.GetActiveGameplayEffect(ActiveHandle).Spec.GetLevel()` 获取。基于支持 `Attributes` 的 `Modifiers` 会在这些支持的 `Attributes` 更新时自动更新。`SetActiveGameplayEffectLevel()` 中用于更新 `Modifiers` 的关键功能是：

```C++  
MarkItemDirty(Effect);  
Effect.Spec.CalculateModifierMagnitudes();  
// 私有函数，否则我们会调用这三个函数，而不需要将级别设置为原来的级别  
UpdateAllAggregatorModMagnitudes(Effect);  
```

`GameplayEffects` 通常不会实例化。当一个能力或 `ASC` 想要应用一个 `GameplayEffect` 时，它会从 `GameplayEffect` 的 `ClassDefaultObject` 创建一个 [`GameplayEffectSpec`](#concepts-ge-spec)。成功应用的 `GameplayEffectSpecs` 然后会被添加到一个新的结构体 `FActiveGameplayEffect` 中，这是 `ASC` 在一个特殊的容器结构体 `ActiveGameplayEffects` 中跟踪的内容。

**[⬆ 回到顶部](#目录)**

#### 4.5.2 应用 Gameplay Effects
`GameplayEffects` 可以通过多种方式应用，包括来自 [`GameplayAbilities`](#concepts-ga) 和 `ASC` 上的函数，通常采用 `ApplyGameplayEffectTo` 的形式。不同的函数本质上是便利函数，最终会调用 `UAbilitySystemComponent::ApplyGameplayEffectSpecToSelf()` 到 `Target` 上。

为了在 `GameplayAbility` 之外应用 `GameplayEffects`，例如从一个投射物，你需要获取 `Target` 的 `ASC` 并使用其中一个函数来 `ApplyGameplayEffectToSelf`。

你可以通过绑定到 `ASC` 的委托来监听任何 `Duration` 或 `Infinite` `GameplayEffects` 应用到 `ASC` 上的时刻：

```c++  
AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf.AddUObject(this, &APACharacterBase::OnActiveGameplayEffectAddedCallback);  
```  

回调函数:  

```c++  
virtual void OnActiveGameplayEffectAddedCallback(UAbilitySystemComponent* Target, const FGameplayEffectSpec& SpecApplied, FActiveGameplayEffectHandle ActiveHandle);  
```

服务器总是会调用这个函数，不论复制模式如何。自主代理（autonomous proxy）仅在 `Full` 和 `Mixed` 复制模式下为复制的 `GameplayEffects` 调用此函数。模拟代理（simulated proxy）仅在 `Full` [复制模式](#concepts-asc-rm)下调用此函数。

**[⬆ 回到顶部](#目录)**

#### 4.5.3 移除 Gameplay Effects
`GameplayEffects` 可以通过多种方式移除，包括来自 [`GameplayAbilities`](#concepts-ga) 和 `ASC` 上的函数，通常采用 `RemoveActiveGameplayEffect` 的形式。不同的函数本质上是便利函数，最终会调用 `FActiveGameplayEffectsContainer::RemoveActiveEffects()` 到 `Target` 上。

为了在 `GameplayAbility` 之外移除 `GameplayEffects`，你需要获取 `Target` 的 `ASC` 并使用其中一个函数来 `RemoveActiveGameplayEffect`。

你可以通过绑定到 `ASC` 的委托来监听任何 `Duration` 或 `Infinite` `GameplayEffects` 从 `ASC` 上被移除的时刻：

```c++  
AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate().AddUObject(this, &APACharacterBase::OnRemoveGameplayEffectCallback);  
```  

回调函数:

```c++  
virtual void OnRemoveGameplayEffectCallback(const FActiveGameplayEffect& EffectRemoved);  
```

服务器总是会调用这个函数，不论复制模式如何。自主代理（autonomous proxy）仅在 `Full` 和 `Mixed` 复制模式下为复制的 `GameplayEffects` 调用此函数。模拟代理（simulated proxy）仅在 `Full` [复制模式](#concepts-asc-rm)下调用此函数。

**[⬆ 回到顶部](#目录)**

#### 4.5.4 Gameplay Effect Modifiers（游戏玩法效果修改器）
`Modifiers` 改变一个 `Attribute`，并且是唯一可以 [预测性](#concepts-p) 地改变一个 `Attribute` 的方式。一个 `GameplayEffect` 可以拥有零个或多个 `Modifiers`。每个 `Modifier` 负责通过指定的操作只改变一个 `Attribute`。

| 操作    | 描述                                                                                                         |
| ------- | ------------------------------------------------------------------------------------------------------------------- |
| `Add`   | 将结果加到 `Modifier` 指定的 `Attribute` 上。使用负值进行减法运算。                                                |
| `Multiply` | 将结果乘到 `Modifier` 指定的 `Attribute` 上。                                                                      |
| `Divide` | 用结果除以 `Modifier` 指定的 `Attribute`。                                                                         |
| `Override` | 用结果覆盖 `Modifier` 指定的 `Attribute`。                                                                        |

`Attribute` 的 `CurrentValue` 是其所有 `Modifiers` 加上 `BaseValue` 的聚合结果。`Modifiers` 如何聚合的公式定义在 `GameplayEffectAggregator.cpp` 文件中的 `FAggregatorModChannel::EvaluateWithBase` 函数里：

```c++  
((InlineBaseValue + Additive) * Multiplicitive) / Division  
```

任何 `Override` 类型的 `Modifiers` 都会用最后应用的 `Modifier` 来覆盖最终值。

**注意：** 对于基于百分比的变化，请确保使用 `Multiply` 操作，以便在加法之后发生。

**注意：** [预测](#concepts-p) 在处理百分比变化时可能会出现问题。

有四种类型的 `Modifiers`：Scalable Float（可缩放浮点数）、Attribute Based（基于属性）、Custom Calculation Class（自定义计算类）和 Set By Caller（由调用者设置）。它们都会生成某个浮点数值，然后根据 `Modifier` 的操作来改变指定的 `Attribute`。

| `Modifier` 类型            | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Scalable Float`           | `FScalableFloats` 是一种结构，它可以指向一个 Data Table，其中变量作为行，等级作为列。Scalable Floats 会自动读取在当前等级（或在 [`GameplayEffectSpec`](#concepts-ge-spec) 上被覆盖的不同等级）下的指定表格行的值。这个值可以通过系数进一步调整。如果没有指定 Data Table/Row，则将其视为 1，因此系数可以用来在所有等级上硬编码单个值。![ScalableFloat](https://github.com/tranek/GASDocumentation/raw/master/Images/scalablefloats.png)                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `Attribute Based`          | `Attribute Based` 类型的 `Modifiers` 采用 `Source`（创建了 `GameplayEffectSpec` 的实体）或 `Target`（接收了 `GameplayEffectSpec` 的实体）上的支持 `Attribute` 的 `CurrentValue` 或 `BaseValue`，并通过系数以及系数前后的加法进一步修改它。`Snapshotting` 意味着支持的 `Attribute` 在创建 `GameplayEffectSpec` 时被捕获，而没有快照则意味着 `Attribute` 在应用 `GameplayEffectSpec` 时被捕获。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `Custom Calculation Class` | `Custom Calculation Class` 为复杂的 `Modifiers` 提供最大的灵活性。这种 `Modifier` 接受一个 [`ModifierMagnitudeCalculation`](#concepts-ge-mmc) 类，并可以通过系数以及系数前后的加法进一步操纵生成的浮点值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `Set By Caller`            | `SetByCaller` 类型的 `Modifiers` 是在运行时由能力或其他创建 `GameplayEffectSpec` 的实体在 `GameplayEffectSpec` 上设置的值。例如，如果你想根据玩家按住按钮为能力充电的时间长短来设置伤害，就可以使用 `SetByCaller`。`SetByCallers` 实质上是 `TMap<FGameplayTag, float>`，存在于 `GameplayEffectSpec` 上。`Modifier` 只是告诉 `Aggregator` 查找与提供的 `GameplayTag` 关联的 `SetByCaller` 值。`Modifiers` 使用的 `SetByCallers` 只能使用 `GameplayTag` 版本的概念。这里禁用了 `FName` 版本。如果 `Modifier` 设置为 `SetByCaller`，但在 `GameplayEffectSpec` 上不存在具有正确 `GameplayTag` 的 `SetByCaller`，游戏将抛出运行时错误并返回 0 的值。这在 `Divide` 操作的情况下可能会导致问题。有关如何使用 `SetByCallers` 的更多信息，请参见 [`SetByCallers`](#concepts-ge-spec-setbycaller)。 |

**[⬆ 回到顶部](#目录)**

##### 4.5.4.1 Multiply and Divide Modifiers（乘法和除法修饰符）
默认情况下，在将所有 `Multiply` 和 `Divide` 类型的 `Modifiers` 乘入或除入 `Attribute` 的 `BaseValue` 之前，会先将它们加在一起。

```c++  
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) const  
{  
    ...    
    float Additive = SumMods(Mods[EGameplayModOp::Additive], 
                             GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Additive), 
                             Parameter
    );
        
    float Multiplicitive = SumMods(Mods[EGameplayModOp::Multiplicitive], 
                                   GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Multiplicitive), 
                                   Parameters
    );    
    float Division = SumMods(Mods[EGameplayModOp::Division], 
                             GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Division), 
                             Parameters
    ); 
       
    ...   
     
    return ((InlineBaseValue + Additive) * Multiplicitive) / Division; 
       
    ...
}  
```  
  
```c++  
float FAggregatorModChannel::SumMods(const TArray<FAggregatorMod>& InMods, float Bias, const FAggregatorEvaluateParameters& Parameters)  
{  
    float Sum = Bias;  
    for (const FAggregatorMod& Mod : InMods)    
    {       
        if (Mod.Qualifies())       
        {          
            Sum += (Mod.EvaluatedMagnitude - Bias);       
        }    
    }  
    
    return Sum;
}  
```

这个公式会导致一些意外的结果。首先，这个公式在将所有 `Modifiers` 乘入或除入 `BaseValue` 之前，会先将它们加在一起。大多数人会期望这些 `Modifiers` 相乘或相除。例如，如果有两个 `Multiply` 类型的 `Modifiers` 其值为 `1.5`，大多数人会期望 `BaseValue` 被乘以 `1.5 x 1.5 = 2.25`。相反，这个公式将 `1.5` 加在一起，将 `BaseValue` 乘以 `2`（`50% 增加 + 另一个 50% 增加 = 100% 增加`）。这是来自 `GameplayPrediction.h` 的例子：对 `500` 的基础速度施加 `10%` 的加速效果将是 `550`。再施加另一个 `10%` 的加速效果，速度将是 `600`。

其次，这个公式有一些未记录的规定，关于哪些值可以使用，因为它是针对 Paragon 设计的。

`Multiply` 和 `Divide` 类型的乘法加法公式的规则：
* `(最多有一个值 < 1) 并且 (任意数量的值 [1, 2))`
* `或者 (一个值 >= 2)`

公式中的 `Bias` 基本上是从范围 `[1, 2)` 内的数字中减去整数部分。第一个 `Modifier` 的 `Bias` 从循环开始前设置为 `Bias` 的起始 `Sum` 值中减去，这就是为什么单独的任何值都可以工作，以及为什么一个小于 `1` 的值可以与范围 `[1, 2)` 内的数字一起工作。

一些 `Multiply` 的示例：
Multiplier: `0.5`  
`1 + (0.5 - 1) = 0.5`，正确

Multiplier: `0.5, 0.5`  
`1 + (0.5 - 1) + (0.5 - 1) = 0`，不正确，期望值为 `1`？多个小于 `1` 的值对于加法乘数来说没有意义。Paragon 设计时仅使用了 [最大负值作为 `Multiply` 类型 `Modifiers` 的值](#cae-nonstackingge)，因此最多只会有一个小于 `1` 的值乘入 `BaseValue`。

Multiplier: `1.1, 0.5`  
`1 + (0.5 - 1) + (1.1 - 1) = 0.6`，正确

Multiplier: `5, 5`  
`1 + (5 - 1) + (5 - 1) = 9`，不正确，期望值为 `10`。始终是 `Modifiers` 的总和减去 `Modifiers` 的数量加上 `1`。

许多游戏希望他们的 `Multiply` 和 `Divide` 类型的 `Modifiers` 在应用于 `BaseValue` 之前能够相乘或相除。
为了实现这一点，你需要 **修改引擎代码** 中的 `FAggregatorModChannel::EvaluateWithBase()` 函数。

```c++  
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) const  
{  
    ...    
    float Multiplicitive = MultiplyMods(Mods[EGameplayModOp::Multiplicitive], Parameters);    
    float Division = MultiplyMods(Mods[EGameplayModOp::Division], Parameters); 
       
    ...  
    
    return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
}  
```  
  
```c++  
float FAggregatorModChannel::MultiplyMods(const TArray<FAggregatorMod>& InMods, 
                                          const FAggregatorEvaluateParameters& Parameters)  
{  
    float Multiplier = 1.0f;  
    for (const FAggregatorMod& Mod : InMods)    
    {       
        if (Mod.Qualifies())       
        {          
            Multiplier *= Mod.EvaluatedMagnitude;       
        }    
    }  
    
    return Multiplier;
}  
```

**[⬆ 回到顶部](#目录)**

##### 4.5.4.2 Gameplay Tags on Modifiers
`SourceTags` 和 `TargetTags` 可以为每个 [Modifier](#concepts-ge-mods) 设置。它们的工作方式与 `GameplayEffect` 的 [`Application Tag 要求`](#concepts-ge-tags) 相同。因此，这些标签只在效果应用时考虑。例如，在具有周期性的无限效果的情况下，这些标签只在效果首次应用时考虑，而不是在每次周期执行时考虑。

`Attribute Based` 类型的 `Modifiers` 也可以设置 `SourceTagFilter` 和 `TargetTagFilter`。在确定 `Attribute Based` 类型 `Modifier` 的属性大小时，这些过滤器用于排除某些对该属性的 `Modifiers`。源或目标没有过滤器中所有标签的 `Modifiers` 将被排除。

这意味着具体而言：源 ASC 和目标 ASC 的标签被 `GameplayEffects` 捕获。当创建 `GameplayEffectSpec` 时捕获源 ASC 的标签，而在效果执行时捕获目标 ASC 的标签。在确定无限或持续效果的 `Modifier` 是否“符合条件”（即其 `Aggregator` 符合条件）并且设置了这些过滤器时，将捕获的标签与过滤器进行比较。

**[⬆ 回到顶部](#目录)**

#### 4.5.5 Stacking Gameplay Effects
默认情况下，`GameplayEffects` 会在应用时创建新的 `GameplayEffectSpec` 实例，并不会关心之前已存在的实例。`GameplayEffects` 可以设置为堆叠模式，在这种模式下，不是添加一个新的 `GameplayEffectSpec` 实例，而是改变当前存在的 `GameplayEffectSpec` 的堆叠计数。堆叠仅适用于 `Duration` 和 `Infinite` 类型的 `GameplayEffects`。

有两种类型的堆叠：按源堆叠 (Aggregate by Source) 和按目标堆叠 (Aggregate by Target)。

| Stacking Type       | Description                                                                                                                          |  
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |  
| Aggregate by Source | There is a separate instance of stacks per Source `ASC` on the Target. Each Source can apply X amount of stacks.                     |  
| Aggregate by Target | There is only one instance of stacks on the Target regardless of Source. Each Source can apply a stack up to the shared stack limit. |

堆叠还具有过期、持续时间刷新和周期重置的策略。在 `GameplayEffect` 蓝图中，这些策略有帮助性的悬停提示。

示例项目包括一个自定义蓝图节点，用于监听 `GameplayEffect` 的堆叠变化。HUD UMG 小部件使用它来更新玩家拥有的被动护甲堆叠的数量。这个 `AsyncTask` 会一直存在，直到手动调用 `EndTask()`，我们在 UMG 小部件的 `Destruct` 事件中执行这一操作。参见 `AsyncTaskEffectStackChanged.h/cpp`。

![监听 GameplayEffect 堆栈更改 BP 节点](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/gestackchange.png)

**[⬆ 回到顶部](#目录)**

#### 4.5.6 Granted Abilities（授予能力）
`GameplayEffects` 可以为 `ASCs` 授予新的 [`GameplayAbilities`](#concepts-ga)。只有 `Duration` 和 `Infinite` 类型的 `GameplayEffects` 可以授予能力。

一个常见的用途是在你想迫使另一个玩家做某事时，比如通过击退或拉拽使他们移动。你可以对他们施加一个 `GameplayEffect`，该效果授予他们一个自动激活的能力（参见 [Passive Abilities](#concepts-ga-activating-passive) 了解如何在授予能力时自动激活它），该能力会对他们执行所需的动作。

设计师可以选择 `GameplayEffect` 授予哪些能力、授予时的等级、绑定的 [输入](#concepts-ga-input) 以及授予能力的移除策略。

| 移除策略             | 描述                                                                                                                                                                     |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 立即取消能力 | 当授予该能力的 `GameplayEffect` 从目标移除时，授予的能力会被立即取消并移除。                                                   |
| 结束后移除能力 | 允许授予的能力完成后再从目标移除。                                                                                                   |
| 什么都不做                 | 授予的能力不受从目标移除授予 `GameplayEffect` 的影响。目标永久拥有该能力，直到后来手动移除。 |

**[⬆ 回到顶部](#目录)**

#### 4.5.7 Gameplay Effect Tags
`GameplayEffects` 携带多个 [`GameplayTagContainers`](#concepts-gt)。设计师将为每个类别编辑 `Added` 和 `Removed` 的 `GameplayTagContainers`，并在编译时显示在 `Combined` `GameplayTagContainer` 中。`Added` 标签是此 `GameplayEffect` 添加的新标签，而其父类之前没有这些标签。`Removed` 标签是父类有但此子类没有的标签。

| 类别                          | 描述                                                                                                                                                                                                                                                                                                                                                                        |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GameplayEffect` 资产标签        | `GameplayEffect` 所具有的标签。它们本身不执行任何功能，仅用于描述 `GameplayEffect`。                                                                                                                                                                                                                                        |
| 授予的标签                      | 存在于 `GameplayEffect` 上但也给予被此 `GameplayEffect` 应用到的 `ASC` 的标签。当 `GameplayEffect` 被移除时，这些标签也会从 `ASC` 上移除。这仅适用于 `Duration` 和 `Infinite` 类型的 `GameplayEffects`。                                                                                                                             |
| 持续标签要求                    | 一旦应用，这些标签决定了 `GameplayEffect` 是处于开启还是关闭状态。一个 `GameplayEffect` 即使处于关闭状态仍然可以被应用。如果一个 `GameplayEffect` 因未能满足持续标签要求而关闭，但随后满足了这些要求，则 `GameplayEffect` 会再次开启并重新应用其修改器。这仅适用于 `Duration` 和 `Infinite` 类型的 `GameplayEffects`。 |
| 应用标签要求                    | 决定 `GameplayEffect` 是否可以应用于目标的标签。如果这些要求未得到满足，则 `GameplayEffect` 不会被应用。                                                                                                                                                                                                                      |
| 移除带有标签的 `GameplayEffects` | 当此 `GameplayEffect` 成功应用时，目标上的任何在其 `Asset Tags` 或 `Granted Tags` 中具有这些标签的 `GameplayEffects` 将被移除。                                                                                                                                                                                            |

**[⬆ 回到顶部](#目录)**

#### 4.5.8 Immunity（免疫）
`GameplayEffects` 可以授予免疫能力，有效地阻止其他 `GameplayEffects` 的应用，基于 [`GameplayTags`](#concepts-gt)。虽然可以通过其他方式如 `Application Tag Requirements` 来实现免疫，但使用此系统提供了当 `GameplayEffects` 因免疫而被阻止时的委托 `UAbilitySystemComponent::OnImmunityBlockGameplayEffectDelegate`。

`GrantedApplicationImmunityTags` 检查源 `ASC`（包括源能力的 `AbilityTags` 中的标签，如果有的话）是否具有任何指定的标签。这是一种根据角色或来源的标签来提供对所有 `GameplayEffects` 的免疫的方法。

`Granted Application Immunity Query` 检查传入的 `GameplayEffectSpec` 是否匹配任何查询以阻止或允许其应用。

这些查询在 `GameplayEffect` 蓝图中有帮助性的悬停提示。

**[⬆ 回到顶部](#目录)**

#### 4.5.9 Gameplay Effect Spec
`GameplayEffectSpec`（简称 `GESpec`）可以看作是 `GameplayEffects` 的实例化。它们持有指向所代表的 `GameplayEffect` 类的引用、创建时的等级以及创建者的信息。与 `GameplayEffects` 不同，`GameplayEffectSpecs` 可以在运行时自由创建和修改，在应用前。当应用一个 `GameplayEffect` 时，会从 `GameplayEffect` 创建一个 `GameplayEffectSpec`，并将其实际应用于目标。

`GameplayEffectSpecs` 使用 `UAbilitySystemComponent::MakeOutgoingSpec()` 从 `GameplayEffects` 创建，这是一个 `BlueprintCallable` 方法。`GameplayEffectSpecs` 不必立即应用。通常会将一个 `GameplayEffectSpec` 传递给从能力创建的投射物，以便投射物稍后可以将其应用于它击中的目标。当 `GameplayEffectSpecs` 成功应用时，它们会返回一个新的结构体 `FActiveGameplayEffect`。

值得注意的 `GameplayEffectSpec` 内容包括：
* 创建此 `GameplayEffect` 的 `GameplayEffect` 类。
* 此 `GameplayEffectSpec` 的等级。通常与创建 `GameplayEffectSpec` 的能力的等级相同，但也可以不同。
* 此 `GameplayEffectSpec` 的持续时间。默认为 `GameplayEffect` 的持续时间，但可以不同。
* 对于周期性效果，此 `GameplayEffectSpec` 的周期。默认为 `GameplayEffect` 的周期，但可以不同。
* 此 `GameplayEffectSpec` 的当前堆叠数量。堆叠限制位于 `GameplayEffect` 上。
* [`GameplayEffectContextHandle`](#concepts-ge-context) 告诉我们谁创建了此 `GameplayEffectSpec`。
* 在 `GameplayEffectSpec` 创建时由于快照捕获的 `Attributes`。
* 除了 `GameplayEffect` 授予的 `GameplayTags` 外，`GameplayEffectSpec` 还授予目标的 `DynamicGrantedTags`。
* 除了 `GameplayEffect` 拥有的 `AssetTags` 外，`GameplayEffectSpec` 还拥有的 `DynamicAssetTags`。
* `SetByCaller` `TMaps`。

**[⬆ 回到顶部](#目录)**

##### 4.5.9.1 SetByCallers
`SetByCallers` 允许 `GameplayEffectSpec` 携带与 `GameplayTag` 或 `FName` 关联的浮点值。这些值存储在各自的 `TMaps` 中：`TMap<FGameplayTag, float>` 和 `TMap<FName, float>`。这些值可以用作 `GameplayEffect` 上的 `Modifiers` 或作为通用手段来传递浮点值。通常会在能力内部生成数值数据，并通过 `SetByCallers` 传递给 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 或 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc)。

| `SetByCaller` 用途 | 注意事项                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Modifiers`       | 必须在 `GameplayEffect` 类中提前定义。只能使用 `GameplayTag` 版本。如果在 `GameplayEffect` 类中定义了一个 `SetByCaller`，但 `GameplayEffectSpec` 没有对应的标签和浮点值对，那么在应用 `GameplayEffectSpec` 时游戏会出现运行时错误并返回 0。这对于 `Divide` 操作来说是一个潜在的问题。参见 [`Modifiers`](#concepts-ge-mods)。 |
| 其他地方           | 不需要在任何地方提前定义。读取 `GameplayEffectSpec` 上不存在的 `SetByCaller` 可以返回开发者定义的默认值，并可选地给出警告。                                                                                                                                                                                                                                                                                                       |

为了在 Blueprint 中分配 `SetByCaller` 值，使用你需要的版本（`GameplayTag` 或 `FName`）的 Blueprint 节点：

![分配 SetByCaller](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/setbycaller.png)

为了在 Blueprint 中读取 `SetByCaller` 值，你需要在你的 Blueprint 库中创建自定义节点。

为了在 C++ 中分配 `SetByCaller` 值，使用你需要的版本（`GameplayTag` 或 `FName`）的函数：

```c++  
void FGameplayEffectSpec::SetSetByCallerMagnitude(FName DataName, float Magnitude);  
```  
  
```c++  
void FGameplayEffectSpec::SetSetByCallerMagnitude(FGameplayTag DataTag, float Magnitude);  
```

要在 C++ 中读取 `SetByCaller` 值，使用你需要的版本（`GameplayTag` 或 `FName`）的函数：

```c++  
float GetSetByCallerMagnitude(FName DataName, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;  
```  
  
```c++  
float GetSetByCallerMagnitude(FGameplayTag DataTag, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;  
```

我建议使用 `GameplayTag` 版本而非 `FName` 版本。这可以在 Blueprint 中防止拼写错误。

**[⬆ 回到顶部](#目录)**

#### 4.5.10 Gameplay Effect Context
`GameplayEffectContext` 结构体包含了关于 `GameplayEffectSpec` 的发起者和 [`TargetData`](#concepts-targeting-data) 的信息。这也是一个很好的结构体，可以对其进行子类化以在诸如 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc) / [`GameplayEffectExecutionCalculations`](#concepts-ge-ec)、[`AttributeSets`](#concepts-as) 和 [`GameplayCues`](#concepts-gc) 之间传递任意数据。

为了子类化 `GameplayEffectContext`：

1. 子类化 `FGameplayEffectContext`
2. 重写 `FGameplayEffectContext::GetScriptStruct()`
3. 重写 `FGameplayEffectContext::Duplicate()`
4. 如果你的新数据需要复制，则重写 `FGameplayEffectContext::NetSerialize()`
5. 为你的子类实现 `TStructOpsTypeTraits`，就像父结构体 `FGameplayEffectContext` 那样
6. 在你的 [`AbilitySystemGlobals`](#concepts-asg) 类中重写 `AllocGameplayEffectContext()` 以返回你的子类的新对象

[GASShooter](https://github.com/tranek/GASShooter) 使用了子类化的 `GameplayEffectContext` 来添加 `TargetData`，这些数据可以在 `GameplayCues` 中访问，特别是对于霰弹枪，因为它可以击中多个敌人。

**[⬆ 回到顶部](#目录)**

#### 4.5.11 Modifier Magnitude Calculation
`ModifierMagnitudeCalculations` (`ModMagCalc` 或 `MMC`) 是强大的类，用作 `GameplayEffects` 中的 `Modifiers`。它们的功能类似于 `GameplayEffectExecutionCalculations`，但功能较弱，最重要的是它们可以被 [预测](#concepts-p)。它们的唯一目的是从 `CalculateBaseMagnitude_Implementation()` 返回一个浮点值。你可以在 Blueprint 和 C++ 中子类化并重写这个函数。

`MMCs` 可以用于任何持续时间的 `GameplayEffects` —— `Instant`、`Duration`、`Infinite` 或 `Periodic`。

`MMCs` 的强大之处在于它们能够捕获 `GameplayEffect` 的 `Source` 或 `Target` 上任意数量的 `Attributes` 的值，并且可以完全访问 `GameplayEffectSpec` 以读取 `GameplayTags` 和 `SetByCallers`。`Attributes` 可以被快照或不被快照。快照 `Attributes` 在创建 `GameplayEffectSpec` 时被捕获，而非快照 `Attributes` 在应用 `GameplayEffectSpec` 时被捕获，并且当 `Attribute` 改变时会自动更新，适用于 `Infinite` 和 `Duration` 类型的 `GameplayEffects`。捕获 `Attributes` 会重新计算它们的 `CurrentValue`，根据 `ASC` 上现有的 mods 进行。这种重新计算 **不会** 运行 `AbilitySet` 中的 [`PreAttributeChange()`](#concepts-as-preattributechange)，因此任何限制必须再次在这里进行。

| 快照 | Source 或 Target | 在 `GameplayEffectSpec` 上捕获 | 当 `Attribute` 改变时对于 `Infinite` 或 `Duration` 类型的 `GameplayEffect` 自动更新 |
| ---- | --------------- | ------------------------------ | --------------------------------------------------------------- |
| Yes  | Source          | 创建                           | No                                                              |
| Yes  | Target          | 应用                           | No                                                              |
| No   | Source          | 应用                           | Yes                                                             |
| No   | Target          | 应用                           | Yes                                                             |

来自 `MMC` 的结果浮点值可以在 `GameplayEffect` 的 `Modifier` 中通过系数以及预系数和后系数加法进一步修改。

一个示例 `MMC` 捕获了 `Target` 的 mana `Attribute`，并从毒效果中减少它，减少的数量取决于 `Target` 拥有多少 mana 以及 `Target` 可能具有的一个标签：

```c++  
UPAMMC_PoisonMana::UPAMMC_PoisonMana()  
{  
  
    //ManaDef defined in header FGameplayEffectAttributeCaptureDefinition ManaDef; 
    ManaDef.AttributeToCapture = UPAAttributeSetBase::GetManaAttribute();
    ManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
    ManaDef.bSnapshot = false;  
    //MaxManaDef defined in header FGameplayEffectAttributeCaptureDefinition MaxManaDef;    
    MaxManaDef.AttributeToCapture = UPAAttributeSetBase::GetMaxManaAttribute();
    MaxManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
    MaxManaDef.bSnapshot = false;  
    RelevantAttributesToCapture.Add(ManaDef);
    RelevantAttributesToCapture.Add(MaxManaDef);}  
  
float UPAMMC_PoisonMana::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const  
{  
    // 从源和目标收集标签，因为这会影响应该使用哪些增益    
    const FGameplayTagContainer* SourceTags = Spec.CapturedSourceTags.GetAggregatedTags();
    const FGameplayTagContainer* TargetTags = Spec.CapturedTargetTags.GetAggregatedTags();
    FAggregatorEvaluateParameters EvaluationParameters;
    EvaluationParameters.SourceTags = SourceTags;
    EvaluationParameters.TargetTags = TargetTags;
    float Mana = 0.f;    
    GetCapturedAttributeMagnitude(ManaDef, Spec, EvaluationParameters, Mana);
    Mana = FMath::Max<float>(Mana, 0.0f);
    float MaxMana = 0.f;    
    GetCapturedAttributeMagnitude(MaxManaDef, Spec, EvaluationParameters, MaxMana);    
    MaxMana = FMath::Max<float>(MaxMana, 1.0f); // 避免除以零 
    float Reduction = -20.0f;    
    if (Mana / MaxMana > 0.5f)
    {
        // 如果目标的法力值超过一半，则效果加倍       
        Reduction *= 2;
    }
    if (TargetTags->HasTagExact(FGameplayTag::RequestGameplayTag(FName("Status.WeakToPoisonMana"))))  
    {       
        // 如果目标对毒药法力的攻击较弱，效果加倍
        Reduction *= 2;
    }  
          
    return Reduction;  
}  
```

如果你没有在 `MMC` 的构造函数中将 `FGameplayEffectAttributeCaptureDefinition` 添加到 `RelevantAttributesToCapture`，并且尝试捕获 `Attributes`，你会在捕获过程中遇到一个关于缺失 Spec 的错误。如果你不需要捕获 `Attributes`，则不必向 `RelevantAttributesToCapture` 添加任何内容。

**[⬆ 回到顶部](#目录)**

#### 4.5.12 Gameplay Effect Execution Calculation
`GameplayEffectExecutionCalculations` (`ExecutionCalculation`、`Execution`（你经常会在插件的源代码中看到这个词）、或 `ExecCalc`）是 `GameplayEffects` 对 `ASC` 进行更改最强大的方式。与 `ModifierMagnitudeCalculations` 类似，这些也可以捕获 `Attributes` 并可选地对它们进行快照。与 `MMCs` 不同的是，这些可以改变不止一个 `Attribute`，并且基本上可以执行程序员想要做的任何事情。这种强大和灵活性的缺点是它们不能被 [预测](#concepts-p)，并且必须用 C++ 实现。

`ExecutionCalculations` 只能与 `Instant` 和 `Periodic` 类型的 `GameplayEffects` 一起使用。任何带有 'Execute' 字样的通常指的是这两种类型的 `GameplayEffects`。

快照捕获会在创建 `GameplayEffectSpec` 时捕获 `Attribute`，而非快照捕获会在应用 `GameplayEffectSpec` 时捕获 `Attribute`。捕获 `Attributes` 会根据 `ASC` 上现有的 mods 重新计算它们的 `CurrentValue`。这种重新计算 **不会** 运行 `AbilitySet` 中的 [`PreAttributeChange()`](#concepts-as-preattributechange)，因此任何限制必须再次在这里进行。

| 快照 | Source 或 Target | 在 `GameplayEffectSpec` 上捕获 |
| ---- | --------------- | ------------------------------ |
| Yes  | Source          | 创建                           |
| Yes  | Target          | 应用                           |
| No   | Source          | 应用                           |
| No   | Target          | 应用                           |

为了设置 `Attribute` 捕获，我们遵循 Epic 的 ActionRPG 示例项目中设定的模式，定义一个结构体来保存如何捕获 `Attributes`，并在结构体的构造函数中创建一个副本。每个 `ExecCalc` 都会有一个这样的结构体。**注意：**每个结构体都需要有一个唯一的名称，因为它们共享同一个命名空间。如果结构体使用相同的名称会导致捕获 `Attributes` 时出现不正确的行为（主要是捕获错误的 `Attributes` 的值）。

对于 `Local Predicted`、`Server Only` 和 `Server Initiated` 类型的 `GameplayAbilities`，`ExecCalc` 只在服务器上调用。

基于复杂的公式计算受到的伤害，该公式从 `Source` 和 `Target` 上的多个属性读取数据，这是 `ExecCalc` 最常见的例子。包含的示例项目有一个简单的 `ExecCalc` 来计算伤害，它从 `GameplayEffectSpec` 的 `[SetByCaller]`(#concepts-ge-spec-setbycaller) 读取伤害值，然后根据从 `Target` 捕获的护甲 `Attribute` 来减轻该值。参见 `GDDamageExecCalculation.cpp/.h`。

**[⬆ 回到顶部](#目录)**

##### 4.5.12.1 Sending Data to Execution Calculations
除了捕获 `Attributes` 之外，还有几种方法可以向 `ExecutionCalculation` 发送数据。

###### 4.5.12.1.1 SetByCaller
任何设置在 `GameplayEffectSpec` 上的 [`SetByCallers`](#concepts-ge-spec-setbycaller) 都可以直接在 `ExecutionCalculation` 中读取。

```c++  
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();  
float Damage = FMath::Max<float>(Spec.GetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName("Data.Damage")), false, -1.0f), 0.0f);  
```

###### 4.5.12.1.2 Backing Data Attribute Calculation Modifier
如果你想为 `GameplayEffect` 硬编码一些值，你可以使用 `CalculationModifier` 来传递这些值，其中使用其中一个捕获的 `Attribute` 作为支持数据。

在这个截图示例中，我们正在将 50 加到捕获的 Damage `Attribute` 上。你也可以将其设置为 `Override`，仅接收硬编码的值。

![支持数据属性计算修饰符](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/calculationmodifierbackingdataattribute.png)

`ExecutionCalculation` 在捕获 `Attribute` 时会读取这个值。

```c++  
float Damage = 0.0f;  
// 捕获在伤害 GE 上设置的可选伤害值作为 ExecutionCalculation 下的 CalculationModifier  
ExecutionParams.AttemptCalculateCapturedAttributeMagnitude(DamageStatics().DamageDef, EvaluationParameters, Damage);  
```

###### 4.5.12.1.3 Backing Data Temporary Variable Calculation Modifier
如果你想为 `GameplayEffect` 硬编码一些值，你可以使用 `CalculationModifier`，其中使用一个 `Temporary Variable`（在 C++ 中称为 `Transient Aggregator`）。`Temporary Variable` 与一个 `GameplayTag` 相关联。

在这个截图示例中，我们正在使用 `Data.Damage` `GameplayTag` 将 50 加到一个 `Temporary Variable` 上。

![支持数据临时变量计算修饰符](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/calculationmodifierbackingdatatempvariable.png)

在你的 `ExecutionCalculation` 构造函数中添加支持的 `Temporary Variables`：

```c++  
ValidTransientAggregatorIdentifiers.AddTag(FGameplayTag::RequestGameplayTag("Data.Damage"));  
```

`ExecutionCalculation` 使用类似于 `Attribute` 捕获函数的特殊捕获函数来读取这个值。

```c++  
float Damage = 0.0f;  
ExecutionParams.AttemptCalculateTransientAggregatorMagnitude(FGameplayTag::RequestGameplayTag("Data.Damage"), EvaluationParameters, Damage);  
```

###### 4.5.12.1.4 Gameplay Effect Context
你可以通过 `GameplayEffectSpec` 上的自定义 [`GameplayEffectContext`](#concepts-ge-context) 向 `ExecutionCalculation` 发送数据。

在 `ExecutionCalculation` 中，你可以从 `FGameplayEffectCustomExecutionParameters` 访问 `EffectContext`。

```c++  
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();  
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(Spec.GetContext().Get());  
```

如果你需要修改 `GameplayEffectSpec` 或 `EffectContext`：

```c++  
FGameplayEffectSpec* MutableSpec = ExecutionParams.GetOwningSpecForPreExecuteMod();  
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(MutableSpec->GetContext().Get());  
```

在 `ExecutionCalculation` 中修改 `GameplayEffectSpec` 时需谨慎。参阅 `GetOwningSpecForPreExecuteMod()` 的注释。

```c++  
/** 非常量访问。对此要小心，尤其是在属性捕获后修改等级库时. */  
FGameplayEffectSpec* GetOwningSpecForPreExecuteMod() const;  
```

**[⬆ 回到顶部](#目录)**

#### 4.5.13 Custom Application Requirement
`CustomApplicationRequirement` (`CAR`) 类为设计师提供了高级控制权，以确定是否可以应用 `GameplayEffect`，这比简单的 `GameplayTag` 检查更为灵活。这些可以通过覆盖 Blueprint 中的 `CanApplyGameplayEffect()` 或 C++ 中的 `CanApplyGameplayEffect_Implementation()` 来实现。

使用 `CARs` 的示例：
* `Target` 需要有一定数量的某个 `Attribute`
* `Target` 需要有一定数量的某个 `GameplayEffect` 的叠加次数

`CARs` 还可以做更复杂的事情，例如检查 `Target` 上是否已经存在该 `GameplayEffect` 的实例，并更改现有实例的 [持续时间](#concepts-ge-duration) 而不是应用一个新的实例（返回 false 表示 `CanApplyGameplayEffect()`）。

**[⬆ 回到顶部](#目录)**

#### 4.5.14 Cost Gameplay Effect
`GameplayAbilities` 可以有一个可选的 `GameplayEffect`，专门设计用来作为该能力的成本。成本是指 `ASC` 为了激活 `GameplayAbility` 所需拥有的 `Attribute` 数量。如果 `GA` 无法承担 `Cost GE`，那么它们将无法激活。这个 `Cost GE` 应该是一个 `Instant` 类型的 `GameplayEffect`，具有一个或多个从 `Attributes` 中减去数值的 `Modifiers`。默认情况下，`Cost GEs` 是为了能够被预测而设计的，因此建议保留这种能力，也就是说不要使用 `ExecutionCalculations`。对于复杂的成本计算，`MMCs` 完全适用且被鼓励使用。

刚开始时，你可能会为每个具有成本的 `GA` 设计一个独特的 `Cost GE`。一种更高级的技术是在多个 `GAs` 中重用一个 `Cost GE`，只需修改从 `Cost GE` 创建的 `GameplayEffectSpec`，加入特定于 `GA` 的数据（成本值定义在 `GA` 上）。**这只适用于 `Instanced` 类型的能力。**

重用 `Cost GE` 的两种技术：

1. **使用 `MMC`。** 这是最简单的方法。创建一个 `MMC` 来从 `GameplayAbility` 实例读取成本值，这个实例可以从 `GameplayEffectSpec` 获取。

```c++  
float UPGMMC_HeroAbilityCost::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const  
{  
    const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());  
    if (!Ability)    
    {       
        return 0.0f;    
    } 
     
    return Ability->Cost.GetValueAtLevel(Ability->GetAbilityLevel());
}  
```

在这个示例中，成本值是我在 `GameplayAbility` 子类上添加的一个 `FScalableFloat`。

```c++  
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cost")  
FScalableFloat Cost;  
```

![Cost GE With MMC](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/costmmc.png)

2. **覆盖 `UGameplayAbility::GetCostGameplayEffect()`。** 覆盖此函数并在运行时 [创建一个 `GameplayEffect`](#concepts-ge-dynamic) 来读取 `GameplayAbility` 上的成本值。

**[⬆ 回到顶部](#目录)**

#### 4.5.15 Cooldown Gameplay Effect
`GameplayAbilities` 可以有一个可选的 `GameplayEffect`，专门设计用来作为该能力的冷却时间。冷却时间决定了能力激活后多久可以再次激活。如果 `GA` 仍在冷却中，则它不能被激活。这个 `Cooldown GE` 应该是一个 `Duration` 类型的 `GameplayEffect`，没有 `Modifiers`，并且为每个 `GameplayAbility` 或每个能力槽位（如果你的游戏有可互换的能力分配给共享冷却时间的槽位）在 `GameplayEffect` 的 `GrantedTags` 中有一个唯一的 `GameplayTag`（称为 `Cooldown Tag`）。实际上 `GA` 检查的是 `Cooldown Tag` 的存在而不是 `Cooldown GE` 的存在。默认情况下，`Cooldown GEs` 是为了能够被预测而设计的，因此建议保留这种能力，也就是说不要使用 `ExecutionCalculations`。对于复杂的冷却时间计算，`MMCs` 完全适用且被鼓励使用。

刚开始时，你可能会为每个具有冷却时间的 `GA` 设计一个独特的 `Cooldown GE`。一种更高级的技术是在多个 `GAs` 中重用一个 `Cooldown GE`，只需修改从 `Cooldown GE` 创建的 `GameplayEffectSpec`，加入特定于 `GA` 的数据（冷却时间和 `Cooldown Tag` 定义在 `GA` 上）。**这只适用于 `Instanced` 类型的能力。**

重用 `Cooldown GE` 的两种技术：

1. **使用一个 [`SetByCaller`](#concepts-ge-spec-setbycaller)。** 这是最简单的方法。将共享的 `Cooldown GE` 的持续时间设置为 `SetByCaller` 并关联一个 `GameplayTag`。在你的 `GameplayAbility` 子类上，定义一个 float / `FScalableFloat` 来表示持续时间，一个 `FGameplayTagContainer` 用于存储唯一的 `Cooldown Tag`，以及一个临时的 `FGameplayTagContainer`，我们将用它作为合并我们的 `Cooldown Tag` 和 `Cooldown GE` 的标签后的返回指针。

```c++  
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")  
FScalableFloat CooldownDuration;  
  
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")  
FGameplayTagContainer CooldownTags;  
  
// 我们将在 GetCooldownTags（） 中返回指针的临时容器
// 这将是我们的 CooldownTags 和 Cooldown GE 的冷却标签的结合。  
UPROPERTY(Transient)  
FGameplayTagContainer TempCooldownTags;  
```

然后覆盖 `UGameplayAbility::GetCooldownTags()` 来返回我们的 `Cooldown Tags` 与现有的 `Cooldown GE` 标签的并集。

```c++  
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const  
{  
    FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
    MutableTags->Reset();// MutableTags 写入 CDO 上的 TempCooldownTags，因此请在技能冷却标签更改（移动到不同的插槽）时清除它
    const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
    if (ParentTags)
    {
        MutableTags->AppendTags(*ParentTags);
    }
    MutableTags->AppendTags(CooldownTags);
    
    return MutableTags;
}  
```

最后，覆盖 `UGameplayAbility::ApplyCooldown()` 来注入我们的 `Cooldown Tags` 并向冷却 `GameplayEffectSpec` 添加 `SetByCaller`。

```c++  
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle,
                                       const FGameplayAbilityActorInfo * ActorInfo,
                                       const FGameplayAbilityActivationInfo ActivationInfo) const  
{
    UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
    if (CooldownGE)
    {
        FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
        SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
        SpecHandle.Data.Get()->SetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName(OurSetByCallerTag)),
                                                       CooldownDuration.GetValueAtLevel(GetAbilityLevel())
        );
        ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
    }
}  
```

在这张图片中，冷却时间（cooldown）的持续时长 Modifier 被设置为 SetByCaller，并且关联了一个 Data Tag 为 Data.Cooldown。

![Cooldown GE with SetByCaller](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/cooldownsbc.png)

2. **使用一个 [`MMC`](#concepts-ge-mmc).** 这种设置与上述相同，只是不在 `Cooldown GE` 和 `ApplyCooldown` 中使用 `SetByCaller` 来设置冷却时间的持续时长。相反，将冷却时间的持续时长设置为一个 `Custom Calculation Class`，并指向我们将要创建的新 `MMC`。

```c++  
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")  
FScalableFloat CooldownDuration;  
  
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")  
FGameplayTagContainer CooldownTags;  
  
// 我们将在 GetCooldownTags（） 中将指针返回到的临时容器。
// 这将是我们的 CooldownTags 和 Cooldown GE 的冷却标签的结合。  
UPROPERTY(Transient)  
FGameplayTagContainer TempCooldownTags;  
```

然后，重写 `UGameplayAbility::GetCooldownTags()` 方法来返回我们自定义的 `Cooldown Tags` 与任何现有 `Cooldown GE` 的标签的并集。

```c++  
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const  
{
    FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
    MutableTags->Reset();// MutableTags 写入 CDO 上的 TempCooldownTags，因此请在技能冷却标签更改（移动到不同的插槽）时清除它
    const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
    if (ParentTags)
    {
        MutableTags->AppendTags(*ParentTags);
    }
    MutableTags->AppendTags(CooldownTags);  
      
    return MutableTags;
}  
```

最后，重写 `UGameplayAbility::ApplyCooldown()` 方法来将我们的 `Cooldown Tags` 注入到冷却时间的 `GameplayEffectSpec` 中。

```c++  
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle,
                                       const FGameplayAbilityActorInfo * ActorInfo,
                                       const FGameplayAbilityActivationInfo ActivationInfo) const  
{  
    UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
    if (CooldownGE)
	{
        FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
        SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
        ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
    }
}  
```

```c++  
float UPGMMC_HeroAbilityCooldown::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const  
{  
    const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());  
    if (!Ability)    
    {
        return 0.0f;    
    }  
    
    return Ability->CooldownDuration.GetValueAtLevel(Ability->GetAbilityLevel());
}  
```

![Cooldown GE with MMC](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/cooldownmmc.png)

##### 4.5.15.1 Get the Cooldown Gameplay Effect's Remaining Time
```c++  
bool APGPlayerState::GetCooldownRemainingForTag(FGameplayTagContainer CooldownTags,
                                                float & TimeRemaining,
                                                float & CooldownDuration)  
{  
    if (AbilitySystemComponent && CooldownTags.Num() > 0)
    {       
        TimeRemaining = 0.f;       
        CooldownDuration = 0.f;
        FGameplayEffectQuery const Query = FGameplayEffectQuery::MakeQuery_MatchAnyOwningTags(CooldownTags);
        TArray<TPair<float, float>> DurationAndTimeRemaining = AbilitySystemComponent->GetActiveEffectsTimeRemainingAndDuration(Query);       
        if (DurationAndTimeRemaining.Num() > 0)       
        {          
            int32 BestIdx = 0;          
            float LongestTime = DurationAndTimeRemaining[0].Key;         
            for (int32 Idx = 1; Idx < DurationAndTimeRemaining.Num(); ++Idx)          
            {
                if (DurationAndTimeRemaining[Idx].Key > LongestTime)             
                {               
                    LongestTime = DurationAndTimeRemaining[Idx].Key;                
                    BestIdx = Idx;             
                }          
            }  
            TimeRemaining = DurationAndTimeRemaining[BestIdx].Key;          
            CooldownDuration = DurationAndTimeRemaining[BestIdx].Value;  
	        
            return true;       
	    }    
	}  
	
    return false;
}  
```

**注意：** 在客户端查询冷却时间剩余的时间要求客户端能够接收复制的 `GameplayEffects`。这取决于它们的 `ASC` 的 [复制模式](#concepts-asc-rm)。

##### 4.5.15.2 监听冷却开始和结束
要监听冷却开始，您可以选择响应 `Cooldown GE` 应用时的情况，通过绑定到 `AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf`，或者响应 `Cooldown Tag` 添加时的情况，通过绑定到 `AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)`。我建议监听 `Cooldown GE` 添加的情况，因为这样您还可以访问到应用它的 `GameplayEffectSpec`。从这里您可以判断 `Cooldown GE` 是否是本地预测的，还是服务器校正的。

要监听冷却结束，您可以选择响应 `Cooldown GE` 移除时的情况，通过绑定到 `AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate()`，或者响应 `Cooldown Tag` 移除时的情况，通过绑定到 `AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)`。我建议监听 `Cooldown Tag` 移除的情况，因为当服务器校正的 `Cooldown GE` 到达时，它会移除我们的本地预测的 `Cooldown GE`，导致 `OnAnyGameplayEffectRemovedDelegate()` 触发，即使我们仍然处于冷却状态。在移除预测的 `Cooldown GE` 和应用服务器校正的 `Cooldown GE` 期间，`Cooldown Tag` 不会发生变化。

**注意：** 在客户端监听 `GameplayEffect` 添加或移除的情况要求客户端能够接收复制的 `GameplayEffects`。这取决于它们的 `ASC` 的 [复制模式](#concepts-asc-rm)。

示例项目包括一个自定义的 Blueprint 节点，用于监听冷却开始和结束。HUD UMG 小部件使用它来更新 Meteor 冷却时间剩余的时间。这个 `AsyncTask` 会一直存在，直到手动调用 `EndTask()`，我们在 UMG 小部件的 `Destruct` 事件中执行此操作。参见 [`AsyncTaskCooldownChanged.h/cpp`](Source/GASDocumentation/Private/Characters/Abilities/AsyncTaskCooldownChanged.cpp)。

![Listen for Cooldown Change BP Node](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/cooldownchange.png)

##### 4.5.15.3 Predicting Cooldowns
目前冷却时间实际上无法真正被预测。我们可以开始 UI 冷却计时器，当本地预测的 `Cooldown GE` 应用时，但是 `GameplayAbility` 实际上的冷却时间是与服务器上剩余的冷却时间绑定的。根据玩家的延迟情况，本地预测的冷却时间可能会过期，但 `GameplayAbility` 仍然在服务器上处于冷却状态，这会阻止 `GameplayAbility` 立即重新激活，直到服务器的冷却时间到期。

示例项目通过在本地预测的冷却开始时将 Meteor 能力的 UI 图标灰化，并在服务器校正的 `Cooldown GE` 到达后启动冷却计时器来处理这种情况。

这一游戏机制的一个后果是，高延迟的玩家在短冷却时间的能力上有较低的射击频率，相比低延迟的玩家处于劣势。《堡垒之夜》通过其武器具有定制的账目管理（不使用冷却 `GameplayEffects`）来避免这种情况。

允许真正的预测冷却时间（玩家可以在本地冷却时间到期时激活 `GameplayAbility`，即使服务器仍在冷却状态）是 Epic 希望在未来版本的 GAS 中实现的功能之一。

**[⬆ 回到顶部](#目录)**

#### 4.5.16 Changing Active Gameplay Effect Duration
要更改 `Cooldown GE` 或任何 `Duration` 类型的 `GameplayEffect` 的剩余时间，我们需要更改 `GameplayEffectSpec` 的 `Duration`，更新其 `StartServerWorldTime`，更新其 `CachedStartServerWorldTime`，更新其 `StartWorldTime`，并重新运行 `CheckDuration()` 来检查持续时间。在服务器上执行这些操作并将 `FActiveGameplayEffect` 标记为脏数据将会将更改复制到客户端。

**注意：** 这确实涉及到了 `const_cast` 的使用，可能不是 Epic 预期的更改持续时间的方式，但目前为止似乎工作得很好。

```c++  
bool UPAAbilitySystemComponent::SetGameplayEffectDurationHandle(FActiveGameplayEffectHandle Handle, float NewDuration)  
{  
    if (!Handle.IsValid())    
    {       
        return false;    
    }  
    const FActiveGameplayEffect* ActiveGameplayEffect = GetActiveGameplayEffect(Handle);    
    if (!ActiveGameplayEffect)    
    {       
        return false;    
    }  
    FActiveGameplayEffect* AGE = const_cast<FActiveGameplayEffect*>(ActiveGameplayEffect);
    if (NewDuration > 0)    
    {       
        AGE->Spec.Duration = NewDuration;    
    }    
    else    
    {       
        AGE->Spec.Duration = 0.01f;    
    }  
    AGE->StartServerWorldTime = ActiveGameplayEffects.GetServerWorldTime();
    AGE->CachedStartServerWorldTime = AGE->StartServerWorldTime;
    AGE->StartWorldTime = ActiveGameplayEffects.GetWorldTime();
    ActiveGameplayEffects.MarkItemDirty(*AGE);    
    ActiveGameplayEffects.CheckDuration(Handle);  
    AGE->EventSet.OnTimeChanged.Broadcast(AGE->Handle, AGE->StartWorldTime, AGE->GetDuration());    
    OnGameplayEffectDurationChange(*AGE);  
    
    return true;
}  
```

**[⬆ 回到顶部](#目录)**

#### 4.5.17 在运行时创建动态 Gameplay Effects
在运行时创建动态 `GameplayEffects` 是一个高级主题。您通常不需要经常这样做。

只有 `Instant` 类型的 `GameplayEffects` 可以从头开始在 C++ 中创建。`Duration` 和 `Infinite` 类型的 `GameplayEffects` 不能在运行时动态创建，因为当它们复制时会查找不存在的 `GameplayEffect` 类定义。为了实现这种功能，您应该像在编辑器中通常所做的那样创建一个原型 `GameplayEffect` 类。然后根据需要在运行时自定义 `GameplayEffectSpec` 实例。

在运行时创建的 `Instant` 类型的 `GameplayEffects` 也可以从 [本地预测](#concepts-p) 的 `GameplayAbility` 内部调用。然而，目前还不清楚动态创建是否会带来副作用。

##### 例子

```c++  
// 创建一个动态的即时游戏效果来提供赏金  
UGameplayEffect* GEBounty = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Bounty")));  
GEBounty->DurationPolicy = EGameplayEffectDurationType::Instant;  
  
int32 Idx = GEBounty->Modifiers.Num();  
GEBounty->Modifiers.SetNum(Idx + 2);  
  
FGameplayModifierInfo& InfoXP = GEBounty->Modifiers[Idx];  
InfoXP.ModifierMagnitude = FScalableFloat(GetXPBounty());  
InfoXP.ModifierOp = EGameplayModOp::Additive;  
InfoXP.Attribute = UGDAttributeSetBase::GetXPAttribute();  
  
FGameplayModifierInfo& InfoGold = GEBounty->Modifiers[Idx + 1];  
InfoGold.ModifierMagnitude = FScalableFloat(GetGoldBounty());  
InfoGold.ModifierOp = EGameplayModOp::Additive;  
InfoGold.Attribute = UGDAttributeSetBase::GetGoldAttribute();  
  
Source->ApplyGameplayEffectToSelf(GEBounty, 1.0f, Source->MakeEffectContext());  
```

第二个示例展示了在一个本地预测的 `GameplayAbility` 内创建的运行时 `GameplayEffect`。使用需谨慎（请参阅代码中的注释）！

```c++  
UGameplayAbilityRuntimeGE::UGameplayAbilityRuntimeGE()  
{  
    NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::LocalPredicted;
}  
  
void UGameplayAbilityRuntimeGE::ActivateAbility(const FGameplayAbilitySpecHandle Handle,
                                                const FGameplayAbilityActorInfo* ActorInfo,
                                                const FGameplayAbilityActivationInfo ActivationInfo,
                                                const FGameplayEventData* TriggerEventData)  
{  
    if (HasAuthorityOrPredictionKey(ActorInfo, &ActivationInfo))    
    {       
        if (!CommitAbility(Handle, ActorInfo, ActivationInfo))       
        {          
            EndAbility(Handle, ActorInfo, ActivationInfo, true, true);       
        }  
		
       // Create the GE at runtime.       
        UGameplayEffect* GameplayEffect = 
        NewObject<UGameplayEffect>(GetTransientPackage(),TEXT("RuntimeInstantGE"));       
        GameplayEffect->DurationPolicy = EGameplayEffectDurationType::Instant;// Only instant works with runtime GE.  
       
       // Add a simple scalable float modifier, which overrides MyAttribute with 42.       
       // In real world applications, consume information passed via TriggerEventData. 
       const int32 Idx = GameplayEffect->Modifiers.Num();       
       GameplayEffect->Modifiers.SetNum(Idx + 1);       
       FGameplayModifierInfo& ModifierInfo = GameplayEffect->Modifiers[Idx];
       ModifierInfo.Attribute.SetUProperty(UMyAttributeSet::GetMyModifiedAttribute());
       ModifierInfo.ModifierMagnitude = FScalableFloat(42.f);
       ModifierInfo.ModifierOp = EGameplayModOp::Override;  
       
       // Apply the GE.  
       
       // Create the GESpec here to avoid the behavior of ASC to create GESpecs from the GE class default object.       
       // Since we have a dynamic GE here, this would create a GESpec with the base GameplayEffect class, so we       
       // would lose our modifiers. Attention: It is unknown, if this "hack" done here can have drawbacks!       
       // The spec prevents the GE object being collected by the GarbageCollector, since the GE is a UPROPERTY on the spec.       
       FGameplayEffectSpec* GESpec = new FGameplayEffectSpec(GameplayEffect, {}, 0.f); // "new", since lifetime is managed by a shared ptr within the handle
       ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, FGameplayEffectSpecHandle(GESpec));    
    }    
    EndAbility(Handle, ActorInfo, ActivationInfo, false, false);
}  
```

**[⬆ 回到顶部](#目录)**

#### 4.5.18 Gameplay Effect Containers
Epic 的 [Action RPG 示例项目](https://www.unrealengine.com/marketplace/en-US/product/action-rpg) 实现了一个名为 `FGameplayEffectContainer` 的结构体。这些结构体并未包含在原生 GAS 中，但对于包含 `GameplayEffects` 和 [`TargetData`](#concepts-targeting-data) 非常有用。它自动化了一些努力，例如从 `GameplayEffects` 创建 `GameplayEffectSpecs` 并在其 `GameplayEffectContext` 中设置默认值。在 `GameplayAbility` 中创建 `GameplayEffectContainer` 并将其传递给生成的弹丸非常简单且直观。我没有在附带的示例项目中实现 `GameplayEffectContainers`，以便展示在没有它们的情况下如何在原生 GAS 中工作，但我强烈建议您了解它们，并考虑将它们添加到您的项目中。

要访问 `GameplayEffectContainers` 内的 `GESpecs` 来执行诸如添加 `SetByCallers` 等操作，需要分解 `FGameplayEffectContainer` 并通过其在 `GESpecs` 数组中的索引来访问 `GESpec` 引用。这要求您提前知道想要访问的 `GESpec` 的索引。

![SetByCaller with a GameplayEffectContainer](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/gecontainersetbycaller.png)

`GameplayEffectContainers` 还包含一种可选的有效 [目标选取](#concepts-targeting-containers) 方法。

**[⬆ 回到顶部](#目录)**

### 4.6 Gameplay Abilities

#### 4.6.1 Gameplay Ability Definition
[`GameplayAbilities`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/UGameplayAbility/index.html) (`GA`) 是游戏中 `Actor` 可以执行的任何动作或技能。一次可以有多个 `GameplayAbility` 处于活动状态，例如冲刺和射击。这些可以在蓝图或 C++ 中制作。

`GameplayAbilities` 的示例：
* 跳跃
* 冲刺
* 射击枪械
* 每隔 X 秒被动阻挡攻击
* 使用药水
* 打开门
* 收集资源
* 构建建筑

不应使用 `GameplayAbilities` 实现的内容：
* 基本移动输入
* 一些与 UI 的交互 — 不应使用 `GameplayAbility` 从商店购买物品。

这些并不是规则，只是我的建议。您的设计和实现可能会有所不同。

`GameplayAbilities` 默认具有层级功能，用于修改属性变化的数量或改变 `GameplayAbility` 的功能。

`GameplayAbilities` 的运行取决于 [`Net Execution Policy`](#concepts-ga-net)，可以在拥有者客户端和/或服务器上运行，但不在模拟代理上运行。`Net Execution Policy` 确定了 `GameplayAbility` 是否会被本地 [预测](#concepts-p)。它们包括了对 [可选成本和冷却 `GameplayEffects`](#concepts-ga-commit) 的默认行为。`GameplayAbilities` 使用 [`AbilityTasks`](#concepts-at) 来处理随时间发生的动作，如等待事件、等待属性变化、等待玩家选择目标或使用 `Root Motion Source` 移动 `Character`。**模拟客户端不会运行 `GameplayAbilities`**。相反，当服务器运行能力时，任何需要在模拟代理上视觉播放的内容（如动画序列）都将通过 `AbilityTasks` 或 [`GameplayCues`](#concepts-gc)（用于声音和粒子等外观效果）进行复制或远程过程调用。

所有 `GameplayAbilities` 都会覆盖其 `ActivateAbility()` 函数，其中包含您的游戏逻辑。可以在 `EndAbility()` 中添加额外的逻辑，该逻辑会在 `GameplayAbility` 完成或被取消时运行。

简单 `GameplayAbility` 的流程图：

![Simple GameplayAbility Flowchart](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/abilityflowchartsimple.png)

更复杂的 `GameplayAbility` 的流程图：

![Complex GameplayAbility Flowchart](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/abilityflowchartcomplex.png)

复杂的能力可以通过使用多个相互作用（激活、取消等）的 `GameplayAbilities` 来实现。

##### 4.6.1.1 Replication Policy
不要使用这个选项。它的名称容易误导人，而且您并不需要它。[`GameplayAbilitySpecs`](#concepts-ga-spec) 默认从服务器复制到拥有者客户端。如上所述，**`GameplayAbilities` 不在模拟代理上运行**。它们使用 `AbilityTasks` 和 `GameplayCues` 来复制或 RPC 视觉变化到模拟代理。Epic 的 Dave Ratti 表达了他希望 [在未来移除这个选项](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89) 的愿望。

##### 4.6.1.2 Server Respects Remote Ability Cancellation
这个选项往往带来的麻烦多过好处。这意味着如果客户端的 `GameplayAbility` 因取消或自然完成而结束，它将强制服务器版本无论是否完成都要结束。后者的问题更为重要，特别是对于高延迟玩家使用的本地预测 `GameplayAbilities`。通常您会希望禁用这个选项。

##### 4.6.1.3 Replicate Input Directly
设置这个选项将始终将输入按下和释放事件复制到服务器。Epic 建议不要使用这个选项，而是依赖现有的与输入相关的 [`AbilityTasks`](#concepts-at) 中内置的 `Generic Replicated Events`，前提是您已经将 [输入绑定到您的 `ASC`](#concepts-ga-input)。

Epic的评论：

```c++  
/** 直接输入状态复制。如果在能力上将 `bReplicateInputDirectly` 设置为 `true`，则会调用这些方法，通常这不是一个好的选择。（相反，建议使用通用复制事件 `Generic Replicated Events`。） */  
UAbilitySystemComponent::ServerSetInputPressed()  
```

**[⬆ 回到顶部](#目录)**

#### 4.6.2 Binding Input to the ASC
`ASC` 允许您直接将输入动作绑定到它，并在授予时将这些输入分配给 `GameplayAbilities`。分配给 `GameplayAbilities` 的输入动作会在按下时自动激活这些 `GameplayAbilities`，前提是满足 `GameplayTag` 的要求。分配的输入动作是使用响应输入的内置 `AbilityTasks` 所必需的。

除了用于激活 `GameplayAbilities` 的输入动作外，`ASC` 还接受通用的 `Confirm` 和 `Cancel` 输入。这些特殊输入被 `AbilityTasks` 用于确认诸如 [`Target Actors`](#concepts-targeting-actors) 或取消它们。

要将输入绑定到 `ASC`，您必须首先创建一个枚举，将输入动作名称转换为字节。枚举名称必须与项目设置中用于输入动作的名称完全匹配。`DisplayName` 不重要。

来自示例项目：

```c++  
UENUM(BlueprintType)  
enum class EGDAbilityInputID : uint8  
{  
    // 0 None    
    None         UMETA(DisplayName = "None"),    
    // 1 Confirm    
    Confirm      UMETA(DisplayName = "Confirm"),    
    // 2 Cancel    
    Cancel       UMETA(DisplayName = "Cancel"),    
    // 3 LMB    
    Ability1     UMETA(DisplayName = "Ability1"),    
    // 4 RMB    
    Ability2     UMETA(DisplayName = "Ability2"),    
    // 5 Q    
    Ability3     UMETA(DisplayName = "Ability3"),    
    // 6 E    
    Ability4     UMETA(DisplayName = "Ability4"),    
    // 7 R    
    Ability5     UMETA(DisplayName = "Ability5"),    
    // 8 Sprint    
    Sprint       UMETA(DisplayName = "Sprint"),    
    // 9 Jump    
    Jump         UMETA(DisplayName = "Jump")
};  
```

如果您的 `ASC` 位于 `Character` 上，则在 `SetupPlayerInputComponent()` 中包含绑定到 `ASC` 的函数：

```c++  
// Bind to AbilitySystemComponent  
FTopLevelAssetPath AbilityEnumAssetPath = FTopLevelAssetPath(FName("/Script/GASDocumentation"), FName("EGDAbilityInputID"));  
AbilitySystemComponent->BindAbilityActivationToInputComponent(PlayerInputComponent,
                                                              FGameplayAbilityInputBinds(FString("ConfirmTarget"),  
                                                              FString("CancelTarget"), 
                                                              AbilityEnumAssetPath, 
                                                              static_cast<int32>(EGDAbilityInputID::Confirm), 
                                                              static_cast<int32>(EGDAbilityInputID::Cancel))
);  
```

如果您的 `ASC` 位于 `PlayerState` 上，在 `SetupPlayerInputComponent()` 中可能存在竞态条件，即 `PlayerState` 尚未复制到客户端。因此，我建议尝试在 `SetupPlayerInputComponent()` 和 `OnRep_PlayerState()` 中绑定输入。仅使用 `OnRep_PlayerState()` 是不够的，因为有可能在 `PlayerController` 告知客户端调用 `ClientRestart()` 创建 `InputComponent` 之前，`Actor` 的 `InputComponent` 可能为空，此时 `PlayerState` 已经复制。示例项目演示了在这两个位置尝试绑定输入的方法，并使用布尔值控制这一过程，确保实际只绑定输入一次。

**注意：** 在示例项目中，枚举中的 `Confirm` 和 `Cancel` 并不匹配项目设置中的输入动作名称 (`ConfirmTarget` 和 `CancelTarget`)，但我们通过 `BindAbilityActivationToInputComponent()` 提供了它们之间的映射。由于我们提供了映射，所以它们不必匹配，但也可以匹配。枚举中的其他所有输入都必须与项目设置中的输入动作名称相匹配。

对于只会由一个输入激活的 `GameplayAbilities`（它们总是在相同的“槽位”中，就像 MOBA 游戏一样），我倾向于在我的 `UGameplayAbility` 子类中添加一个变量来定义它们的输入。然后，我可以在授予能力时从 `ClassDefaultObject` 中读取这个变量。

##### 4.6.2.1 Binding to Input without Activating Abilities
如果您不想让 `GameplayAbilities` 在按下输入时自动激活，但仍想将它们绑定到输入以便与 `AbilityTasks` 一起使用，您可以在 `UGameplayAbility` 子类中添加一个新的布尔变量 `bActivateOnInput`，默认值为 `true`，并重写 `UAbilitySystemComponent::AbilityLocalInputPressed()`。

```c++  
void UGSAbilitySystemComponent::AbilityLocalInputPressed(int32 InputID)
{
    // 如果此InputID被泛型确认/取消覆盖且泛型确认/取消回调已绑定，则消耗输入
    if (IsGenericConfirmInputBound(InputID))
    {
        LocalInputConfirm();
        return;
    }

    if (IsGenericCancelInputBound(InputID))
    {
        LocalInputCancel();
        return;
    }

    // ---------------------------------------------------------
    
    ABILITYLIST_SCOPE_LOCK();
    for (FGameplayAbilitySpec& Spec : ActivatableAbilities.Items)
    {
        if (Spec.InputID == InputID)
        {
            if (Spec.Ability)
            {
                Spec.InputPressed = true;
                if (Spec.IsActive())
                {
                    if (Spec.Ability->bReplicateInputDirectly && IsOwnerActorAuthoritative() == false)
                    {
                        ServerSetInputPressed(Spec.Handle);
                    }

                    AbilitySpecInputPressed(Spec);

                    // 触发InputPressed事件。此处不进行复制。如果有监听者，他们可能会将InputPressed事件                     复制到服务器。
                    InvokeReplicatedEvent(EAbilityGenericReplicatedEvent::InputPressed, Spec.Handle, Spec.ActivationInfo.GetActivationPredictionKey());
                }
                else
                {
                    UGSGameplayAbility* GA = Cast<UGSGameplayAbility>(Spec.Ability);
                    if (GA && GA->bActivateOnInput)
                    {
                        // 能力当前未激活，尝试激活它
                        TryActivateAbility(Spec.Handle);
                    }
                }
            }
        }
    }
}
```

**[⬆ 回到顶部](#目录)**

#### 4.6.3 Granting Abilities
将 `GameplayAbility` 授予 `ASC` 会将其添加到 `ASC` 的 `ActivatableAbilities` 列表中，如果满足相应的 [`GameplayTag` 要求](#concepts-ga-tags)，则可以随时激活该 `GameplayAbility`。

我们在服务器上授予 `GameplayAbilities`，这会自动将相应的 [`GameplayAbilitySpec`](#concepts-ga-spec) 复制到拥有该能力的客户端。其他客户端或模拟代理不会收到 `GameplayAbilitySpec`。

示例项目在 `Character` 类上存储了一个 `TArray<TSubclassOf<UGDGameplayAbility>>`，并在游戏开始时从中读取并授予这些能力：

```c++
void AGDCharacterBase::AddCharacterAbilities()
{
    // 只有在服务器上授予能力
    if (Role != ROLE_Authority || !AbilitySystemComponent.IsValid() || AbilitySystemComponent->bCharacterAbilitiesGiven)
    {
        return;
    }

    for (TSubclassOf<UGDGameplayAbility>& StartupAbility : CharacterAbilities)
    {
        AbilitySystemComponent->GiveAbility(FGameplayAbilitySpec(StartupAbility, 
                                            GetAbilityLevel(StartupAbility.GetDefaultObject()->AbilityID),
                                            static_cast<int32>(StartupAbility.GetDefaultObject()->AbilityInputID), 
                                            this)
        );
    }

    AbilitySystemComponent->bCharacterAbilitiesGiven = true;
}
```

在授予这些 `GameplayAbilities` 时，我们会创建 `GameplayAbilitySpec`，其中包含了 `UGameplayAbility` 类、能力等级、绑定的输入以及 `SourceObject` 或授予此 `GameplayAbility` 给此 `ASC` 的对象。

**[⬆ 回到顶部](#目录)**

#### 4.6.4 Activating Abilities
如果为 `GameplayAbility` 分配了输入动作，则当按下该输入且满足其 `GameplayTag` 要求时，它将被自动激活。但这可能不是激活 `GameplayAbility` 的理想方式。`ASC` 提供了四种其他方法来激活 `GameplayAbilities`：通过 `GameplayTag`、通过 `GameplayAbility` 类、通过 `GameplayAbilitySpec` 的句柄和通过事件。通过事件激活 `GameplayAbility` 允许您[与事件一起传递数据负载](#concepts-ga-data)。

```c++  
UFUNCTION(BlueprintCallable, Category = "Abilities")  
bool TryActivateAbilitiesByTag(const FGameplayTagContainer& GameplayTagContainer, bool bAllowRemoteActivation = true);  
  
UFUNCTION(BlueprintCallable, Category = "Abilities")  
bool TryActivateAbilityByClass(TSubclassOf<UGameplayAbility> InAbilityToActivate, bool bAllowRemoteActivation = true);  
  
bool TryActivateAbility(FGameplayAbilitySpecHandle AbilityToActivate, bool bAllowRemoteActivation = true);  
  
bool TriggerAbilityFromGameplayEvent(FGameplayAbilitySpecHandle AbilityToTrigger, FGameplayAbilityActorInfo* ActorInfo, FGameplayTag Tag, const FGameplayEventData* Payload, UAbilitySystemComponent& Component);  
  
FGameplayAbilitySpecHandle GiveAbilityAndActivateOnce(const FGameplayAbilitySpec& AbilitySpec, const FGameplayEventData* GameplayEventData);  
```

要通过事件激活 `GameplayAbility`，必须在 `GameplayAbility` 中设置其 `Triggers`。分配一个 `GameplayTag` 并选择一个 `GameplayEvent` 选项。要发送事件，请使用函数 `UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(AActor* Actor, FGameplayTag EventTag, FGameplayEventData Payload)`。通过事件激活 `GameplayAbility` 允许您传递带有数据的负载。

`GameplayAbility` 的 `Triggers` 还允许您在添加或移除 `GameplayTag` 时激活 `GameplayAbility`。

**注意：** 在 Blueprint 中从事件激活 `GameplayAbility` 时，必须使用 `ActivateAbilityFromEvent` 节点。

**注意：** 当 `GameplayAbility` 应该终止时不要忘记调用 `EndAbility()`，除非您有一个始终运行如被动能力的 `GameplayAbility`。

**本地预测** `GameplayAbilities` 的激活序列：
1. **拥有客户端** 调用 `TryActivateAbility()`
2. 调用 `InternalTryActivateAbility()`
3. 调用 `CanActivateAbility()` 并返回 `GameplayTag` 要求是否满足、`ASC` 是否能承担费用、`GameplayAbility` 是否不在冷却中以及没有其他实例正在激活
4. 调用 `CallServerTryActivateAbility()` 并传递它生成的 `Prediction Key`
5. 调用 `CallActivateAbility()`
6. 调用 `PreActivate()` Epic 称之为“初始化样板代码”
7. 调用 `ActivateAbility()` 最终激活能力

**服务器** 接收到 `CallServerTryActivateAbility()`
1. 调用 `ServerTryActivateAbility()`
2. 调用 `InternalServerTryActivateAbility()` 
3. 调用 `InternalTryActivateAbility()`
4. 调用 `CanActivateAbility()` 并返回 `GameplayTag` 要求是否满足、`ASC` 是否能承担费用、`GameplayAbility` 是否不在冷却中以及没有其他实例正在激活
5. 如果成功则调用 `ClientActivateAbilitySucceed()` 告诉客户端更新其 `ActivationInfo` 以确认服务器已激活，并广播 `OnConfirmDelegate` 委托。这不是输入确认。
6. 调用 `CallActivateAbility()`
7. 调用 `PreActivate()` Epic 称之为“初始化样板代码”
8. 调用 `ActivateAbility()` 最终激活能力

如果任何时候服务器未能激活，则会调用 `ClientActivateAbilityFailed()`，立即终止客户端的 `GameplayAbility` 并撤销任何预测的变化。

##### 4.6.4.1 Passive Abilities
为了实现自动激活并持续运行的被动 `GameplayAbilities`，可以重写 `UGameplayAbility::OnAvatarSet()`，该方法会在 `GameplayAbility` 被授予并且 `AvatarActor` 设置好后自动调用，并在其中调用 `TryActivateAbility()`。

我建议在您的自定义 `UGameplayAbility` 类中添加一个 `bool` 变量，用来指定 `GameplayAbility` 在被授予时是否应该激活。示例项目为其被动护甲叠加能力就是这样做的。

被动 `GameplayAbilities` 通常会有 `Server Only` 的 [`Net Execution Policy`](#concepts-ga-net)。

```c++  
void UGDGameplayAbility::OnAvatarSet(const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilitySpec & Spec)  
{  
    Super::OnAvatarSet(ActorInfo, Spec); 
     
    if (bActivateAbilityOnGranted)    
    {       
        ActorInfo->AbilitySystemComponent->TryActivateAbility(Spec.Handle, false);    
    }
}  
```

Epic 将此函数描述为启动被动能力以及执行类似于 `BeginPlay` 操作的正确位置。

**[⬆ 回到顶部](#目录)**

##### 4.6.4.2 Activation Failed Tags
`GameplayAbilities` 有默认逻辑来告诉您为什么能力激活失败。要启用这一点，您必须设置与默认失败情况对应的 `GameplayTags`。

将这些标签（或您自己的命名约定）添加到您的项目中：
```  
+GameplayTagList=(Tag="Activation.Fail.BlockedByTags",DevComment="")  
+GameplayTagList=(Tag="Activation.Fail.CantAffordCost",DevComment="")  
+GameplayTagList=(Tag="Activation.Fail.IsDead",DevComment="")  
+GameplayTagList=(Tag="Activation.Fail.MissingTags",DevComment="")  
+GameplayTagList=(Tag="Activation.Fail.Networking",DevComment="")  
+GameplayTagList=(Tag="Activation.Fail.OnCooldown",DevComment="")  
```

然后将它们添加到 GASDocumentation\\Config\\DefaultGame.ini 中：

```  
[/Script/GameplayAbilities.AbilitySystemGlobals]  
ActivateFailIsDeadName=Activation.Fail.IsDead  
ActivateFailCooldownName=Activation.Fail.OnCooldown  
ActivateFailCostName=Activation.Fail.CantAffordCost  
ActivateFailTagsBlockedName=Activation.Fail.BlockedByTags  
ActivateFailTagsMissingName=Activation.Fail.MissingTags  
ActivateFailNetworkingName=Activation.Fail.Networking  
```

现在每当能力激活失败时，相应的 `GameplayTag` 将被包含在输出日志消息中或在 `showdebug AbilitySystem` HUD 上可见。

```  
LogAbilitySystem: Display: InternalServerTryActivateAbility. Rejecting ClientActivation of Default__GA_FireGun_C. InternalTryActivateAbility failed: Activation.Fail.BlockedByTags  
LogAbilitySystem: Display: ClientActivateAbilityFailed_Implementation. PredictionKey :109 Ability: Default__GA_FireGun_C  
```

![支持数据临时变量计算修饰符](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/activationfailedtags.png)

**[⬆ 回到顶部](#目录)**

#### 4.6.5 Canceling Abilities
要在内部取消 `GameplayAbility`，您可以调用 `CancelAbility()`。这将会调用 `EndAbility()` 并将其 `WasCancelled` 参数设置为 `true`。

要从外部取消 `GameplayAbility`，`ASC` 提供了一些函数：

```c++
/** 取消指定的能力 CDO。 */
void CancelAbility(UGameplayAbility* Ability);	

/** 取消由传入的 spec handle 指示的能力。如果 handle 在重新激活的能力中未找到，则不执行任何操作。 */
void CancelAbilityHandle(const FGameplayAbilitySpecHandle& AbilityHandle);

/** 取消具有指定标签的所有能力。不会取消 Ignore 实例 */
void CancelAbilities(const FGameplayTagContainer* WithTags=nullptr, const FGameplayTagContainer* WithoutTags=nullptr, UGameplayAbility* Ignore=nullptr);

/** 不管标签如何取消所有能力。不会取消 Ignore 实例 */
void CancelAllAbilities(UGameplayAbility* Ignore=nullptr);

/** 取消所有能力并销毁任何剩余的实例化能力 */
virtual void DestroyActiveState();

```

**注意：** 我发现 `CancelAllAbilities` 在处理 `Non-Instanced` `GameplayAbilities` 时似乎不能正常工作。它似乎遇到 `Non-Instanced` `GameplayAbility` 后就放弃了。`CancelAbilities` 更好地处理 `Non-Instanced` `GameplayAbilities`，这也是示例项目使用的（跳跃是一个非实例化的 `GameplayAbility`）。您的体验可能会有所不同。

**[⬆ 回到顶部](#目录)**

#### 4.6.6 Getting Active Abilities
初学者经常会问：“如何获取活动的能力？”他们可能想要在能力上设置变量或取消它。一次可以有多个 `GameplayAbility` 处于活动状态，因此并没有一个单一的“活动能力”。相反，你需要遍历 `ASC`（Ability System Component）中的 `ActivatableAbilities` 列表（即 `ASC` 拥有的被授予的 `GameplayAbilities`），并找到与你正在寻找的 `Asset` 或 `Granted` 的 `GameplayTag` 相匹配的一个。

`UAbilitySystemComponent::GetActivatableAbilities()` 方法会返回一个 `TArray<FGameplayAbilitySpec>`，你可以遍历这个数组来查找所需的能力。

`ASC` 还包含另一个辅助函数，该函数以 `GameplayTagContainer` 作为参数来协助搜索，而不是手动遍历 `GameplayAbilitySpecs` 列表。`bOnlyAbilitiesThatSatisfyTagRequirements` 参数只会返回满足其 `GameplayTag` 要求且当前可以激活的 `GameplayAbilitySpecs`。例如，你可以有两个基础攻击类型的 `GameplayAbilities`，一个使用武器，另一个徒手进行，正确的攻击方式会根据是否装备了武器来设置 `GameplayTag` 要求从而被激活。更多详细信息，请参阅 Epic 对该函数的注释。

```c++  
UAbilitySystemComponent::GetActivatableGameplayAbilitySpecsByAllMatchingTags(const FGameplayTagContainer& GameplayTagContainer, 
                                                                             TArray < struct FGameplayAbilitySpec* >& MatchingGameplayAbilities, 
                                                                             bool bOnlyAbilitiesThatSatisfyTagRequirements = true)
```

一旦找到所需的 `FGameplayAbilitySpec`，你可以调用其上的 `IsActive()` 方法。

**[⬆ 回到顶部](#目录)**

#### 4.6.7 Instancing Policy
`GameplayAbility` 的 `Instancing Policy` 确定了在激活时是否以及如何实例化 `GameplayAbility`。

| `Instancing Policy`               | 描述                                                      | 使用示例                                                                                                                                                             |
| --------------------------------- | ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 每 Actor 实例化 (Instanced Per Actor) | 每个 `ASC` 只有一个 `GameplayAbility` 的实例，在多次激活之间重复使用。        | 这可能是你最常使用的 `Instancing Policy`。任何能力都可以使用它，并且可以在多次激活之间保持状态。设计者负责在需要的情况下手动重置激活间所需的变量。                                                                              |
| 每次执行实例化 (Instanced Per Execution) | 每次激活 `GameplayAbility` 时，都会创建一个新的 `GameplayAbility` 实例。 | 这些 `GameplayAbilities` 的优点在于每次激活时变量都会被重置。它们的性能比 `每 Actor 实例化` 差，因为每次激活都会产生新的 `GameplayAbilities`。示例项目没有使用这种策略。                                                   |
| 非实例化 (Non-Instanced)              | `GameplayAbility` 在其 `ClassDefaultObject` 上运行，不创建实例。    | 这种策略具有最佳的性能，但也是限制最大的。非实例化的 `GameplayAbilities` 不能存储状态，意味着无法使用动态变量，也无法绑定到 `AbilityTask` 委托。最适合频繁使用的简单能力，如 MOBA 或 RTS 游戏中的小兵基础攻击。示例项目的跳跃 `GameplayAbility` 是非实例化的。 |

**[⬆ 回到顶部](#目录)**

#### 4.6.8 Net Execution Policy
`GameplayAbility` 的 `Net Execution Policy` 确定了谁运行 `GameplayAbility` 以及运行的顺序。

| `Net Execution Policy` | 描述                                                                                                                                                                                                         |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Local Only`           | `GameplayAbility` 仅在拥有客户端上运行。这可能适用于仅在本地进行外观更改的能力。单人游戏应使用 `Server Only`。                                     |
| `Local Predicted`      | `Local Predicted` 类型的 `GameplayAbilities` 首先在拥有客户端上激活，然后在服务器上激活。服务器版本将修正客户端预测中的任何错误。参见 [Prediction](#concepts-p)。 |
| `Server Only`          | `GameplayAbility` 仅在服务器上运行。被动 `GameplayAbilities` 通常为 `Server Only`。单人游戏应使用此策略。                                                                  |
| `Server Initiated`     | `Server Initiated` 类型的 `GameplayAbilities` 首先在服务器上激活，然后在拥有客户端上激活。我个人很少使用这种类型。                                                                     | 

**[⬆ 回到顶部](#目录)**

#### 4.6.9 Ability Tags
`GameplayAbilities` 自带带有内置逻辑的 `GameplayTagContainers`。这些 `GameplayTags` 都不会被复制。

| `GameplayTag Container`     | 描述                                                                                                                                                                                   |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Ability Tags`              | `GameplayAbility` 所拥有的 `GameplayTags`。这些只是用来描述 `GameplayAbility` 的 `GameplayTags`。                                                                              |
| `Cancel Abilities with Tag` | 当这个 `GameplayAbility` 被激活时，其他具有这些 `GameplayTags` 在其 `Ability Tags` 中的 `GameplayAbilities` 将会被取消。                                                   |
| `Block Abilities with Tag`  | 当这个 `GameplayAbility` 处于活动状态时，其他具有这些 `GameplayTags` 在其 `Ability Tags` 中的 `GameplayAbilities` 会被阻止激活。                                          |
| `Activation Owned Tags`     | 这些 `GameplayTags` 会在该 `GameplayAbility` 处于活动状态时赋予给 `GameplayAbility` 的所有者。请注意这些标签不会被复制。                                                    |
| `Activation Required Tags`  | 如果所有者拥有 **所有** 这些 `GameplayTags`，则此 `GameplayAbility` 才能被激活。                                                                                                |
| `Activation Blocked Tags`   | 如果所有者拥有 **任何** 这些 `GameplayTags`，则此 `GameplayAbility` 不能被激活。                                                                                                  |
| `Source Required Tags`      | 如果 `Source` 拥有 **所有** 这些 `GameplayTags`，则此 `GameplayAbility` 才能被激活。`Source` 的 `GameplayTags` 只有当 `GameplayAbility` 被事件触发时才会被设置。 |
| `Source Blocked Tags`       | 如果 `Source` 拥有 **任何** 这些 `GameplayTags`，则此 `GameplayAbility` 不能被激活。`Source` 的 `GameplayTags` 只有当 `GameplayAbility` 被事件触发时才会被设置。   |
| `Target Required Tags`      | 如果 `Target` 拥有 **所有** 这些 `GameplayTags`，则此 `GameplayAbility` 才能被激活。`Target` 的 `GameplayTags` 只有当 `GameplayAbility` 被事件触发时才会被设置。 |
| `Target Blocked Tags`       | 如果 `Target` 拥有 **任何** 这些 `GameplayTags`，则此 `GameplayAbility` 不能被激活。`Target` 的 `GameplayTags` 只有当 `GameplayAbility` 被事件触发时才会被设置。   |

**[⬆ 回到顶部](#目录)**

#### 4.6.10 Gameplay Ability Spec
`GameplayAbilitySpec` 在 `ASC` 上存在，是在 `GameplayAbility` 被授予后定义可激活的 `GameplayAbility` —— 包括 `GameplayAbility` 类、等级、输入绑定和必须与 `GameplayAbility` 类分开的运行时状态。

当 `GameplayAbility` 在服务器上被授予时，服务器会将 `GameplayAbilitySpec` 复制到拥有客户端，以便客户端可以激活它。

激活 `GameplayAbilitySpec` 会根据其 `Instancing Policy` 创建 `GameplayAbility` 的实例（对于 `Non-Instanced` 类型的 `GameplayAbilities` 不会创建实例）。

**[⬆ 回到顶部](#目录)**

#### 4.6.11 Passing Data to Abilities
`GameplayAbilities` 的一般范式是 `Activate->Generate Data->Apply->End`。有时你需要对现有数据采取行动。GAS 提供了几种将外部数据引入到 `GameplayAbilities` 的选项：

| 方法                                          | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ----------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 通过事件激活 `GameplayAbility`                 | 通过包含数据负载的事件来激活 `GameplayAbility`。对于本地预测的 `GameplayAbilities`，事件的数据负载会从客户端复制到服务器。使用两个 `Optional Object` 或者 `TargetData` 变量来传递不符合现有变量的数据。这种方法的缺点在于无法通过输入绑定来激活能力。要通过事件激活 `GameplayAbility`，`GameplayAbility` 必须在其内部设置好 `Triggers`。分配一个 `GameplayTag` 并选择一个 `GameplayEvent` 选项。发送事件时，使用函数 `UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(AActor* Actor, FGameplayTag EventTag, FGameplayEventData Payload)`。|
| 使用 `WaitGameplayEvent` `AbilityTask`         | 使用 `WaitGameplayEvent` `AbilityTask` 来告诉 `GameplayAbility` 在激活之后监听带有数据负载的事件。事件的数据负载及其发送过程与通过事件激活 `GameplayAbilities` 相同。这种方法的缺点在于事件不会被 `AbilityTask` 复制，因此只适用于 `Local Only` 和 `Server Only` 类型的 `GameplayAbilities`。你可以编写自己的 `AbilityTask` 来复制事件的数据负载。                                                                                                                                                                                                                                                                                               |
| 使用 `TargetData`                               | 自定义的 `TargetData` 结构体是一种在客户端和服务器之间传递任意数据的好方法。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| 在 `OwnerActor` 或 `AvatarActor` 上存储数据    | 使用存储在 `OwnerActor`、`AvatarActor` 或任何可以引用的对象上的复制变量。这种方法最灵活，并且可以与通过输入绑定激活的 `GameplayAbilities` 一起使用。但是，它并不能保证数据在使用时会被同步复制。你必须提前确保这一点——也就是说，如果你设置了复制变量并立即激活 `GameplayAbility`，由于潜在的数据包丢失，接收方无法保证这两者的执行顺序。                                                                                                                                                                                                                                   |

**[⬆ 回到顶部](#目录)**

#### 4.6.12 Ability Cost and Cooldown
`GameplayAbilities` 自带可选的成本和冷却时间功能。成本是指 `ASC` 为了激活 `GameplayAbility` 必须拥有的预定义数量的 `Attributes`，通常通过 `Instant` 类型的 `GameplayEffect` 实现（参见 [`Cost GE`](#concepts-ge-cost)）。冷却时间是防止重新激活 `GameplayAbility` 的计时器，直到冷却结束，通常通过 `Duration` 类型的 `GameplayEffect` 实现（参见 [`Cooldown GE`](#concepts-ge-cooldown)）。

在 `GameplayAbility` 调用 `UGameplayAbility::Activate()` 之前，它会调用 `UGameplayAbility::CanActivateAbility()`。这个函数检查拥有 `ASC` 是否能够承担成本（`UGameplayAbility::CheckCost()`）并且确保 `GameplayAbility` 不处于冷却状态（`UGameplayAbility::CheckCooldown()`）。

`GameplayAbility` 调用 `Activate()` 后，可以选择在任何时候提交成本和冷却时间，通过调用 `UGameplayAbility::CommitAbility()`，进而调用 `UGameplayAbility::CommitCost()` 和 `UGameplayAbility::CommitCooldown()`。如果成本和冷却不应该同时提交，设计者可以选择单独调用 `CommitCost()` 或 `CommitCooldown()`。提交成本和冷却会再次调用 `CheckCost()` 和 `CheckCooldown()`，这是 `GameplayAbility` 失败与它们相关的最后机会。拥有 `ASC` 的 `Attributes` 可能在 `GameplayAbility` 被激活后发生变化，在提交时可能无法满足成本要求。如果在提交时 [预测键](#concepts-p-key) 有效，则可以 [本地预测](#concepts-p) 成本和冷却的提交。

有关实现细节，请参阅 [`CostGE`](#concepts-ge-cost) 和 [`CooldownGE`](#concepts-ge-cooldown)。

**[⬆ 回到顶部](#目录)**

#### 4.6.13 Leveling Up Abilities
升级能力有两种常见方法：

| 升级方法                            | 描述                                                                                                                                                                                                      |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 撤销并以新等级重新授予             | 从 `ASC` 中撤销（移除） `GameplayAbility`，然后在服务器上以下一个等级重新授予。如果在升级时 `GameplayAbility` 正处于激活状态，这会导致其终止。                                   |
| 增加 `GameplayAbilitySpec` 的等级   | 在服务器上找到 `GameplayAbilitySpec`，增加其等级，并标记为脏以便复制到拥有客户端。如果在升级时 `GameplayAbility` 正处于激活状态，此方法不会使其终止。 |

这两种方法的主要区别在于是否希望在升级时取消处于激活状态的 `GameplayAbilities`。根据你的 `GameplayAbilities`，你可能会同时使用这两种方法。我建议在 `UGameplayAbility` 的子类中添加一个 `bool` 变量来指定使用哪种方法。

**[⬆ 回到顶部](#目录)**

#### 4.6.14 Ability Sets
`GameplayAbilitySets` 是方便使用的 `UDataAsset` 类，用于保存角色的输入绑定和启动 `GameplayAbilities` 列表，并包含了授予这些 `GameplayAbilities` 的逻辑。子类还可以包含额外的逻辑或属性。例如，《Paragon》中的每个英雄都有一个 `GameplayAbilitySet`，包含了他们所有的 `GameplayAbilities`。

至少从我目前看到的情况来看，我认为这个类是不必要的。示例项目是在 `GDCharacterBase` 及其子类内部处理所有 `GameplayAbilitySets` 的功能。

**[⬆ 回到顶部](#目录)**

#### 4.6.15 Ability Batching
传统的 `Gameplay Ability` 生命周期至少涉及客户端到服务器之间的两到三个远程过程调用（RPC）。

1. `CallServerTryActivateAbility()`
2. `ServerSetReplicatedTargetData()` （可选）
3. `ServerEndAbility()`

如果一个 `GameplayAbility` 在一帧内将所有这些操作作为一个原子组执行，我们可以优化这个工作流程，将所有两到三个 RPC 批量（合并）成一个 RPC。`GAS` 将这种 RPC 优化称为 `Ability Batching`。使用 `Ability Batching` 的典型例子是命中扫描枪（hitscan guns）。命中扫描枪激活后进行线性追踪，将 `TargetData` 发送到服务器，并在一帧内的一个原子组中结束能力。[GASShooter](https://github.com/tranek/GASShooter) 示例项目展示了如何为其命中扫描枪应用这项技术。

半自动枪是最佳案例，将 `CallServerTryActivateAbility()`、`ServerSetReplicatedTargetData()`（子弹命中结果）和 `ServerEndAbility()` 批量成一个 RPC，而不是三个 RPC。

全自动/连发枪将 `CallServerTryActivateAbility()` 和第一发子弹的 `ServerSetReplicatedTargetData()` 批量成一个 RPC，而不是两个 RPC。随后每发子弹都是独立的 `ServerSetReplicatedTargetData()` RPC。最终，当枪停止射击时，`ServerEndAbility()` 作为单独的 RPC 发送。这是最坏情况下的场景，我们只能在第一发子弹上节省一个 RPC 而不是两个。这种情况也可以通过通过 `Gameplay Event` 激活能力的方式来实现，这种方式会将子弹的 `TargetData` 包含在 `EventPayload` 中从客户端发送到服务器。后一种方法的缺点是 `TargetData` 必须在能力之外生成，而批量方法则在能力内部生成 `TargetData`。

默认情况下，`Ability Batching` 在 `ASC` 上是禁用的。要启用 `Ability Batching`，需要重写 `ShouldDoServerAbilityRPCBatch()` 函数返回 `true`：

```c++  
virtual bool ShouldDoServerAbilityRPCBatch() const override { return true; }  
```

现在 `Ability Batching` 已经启用，如果你想对某些能力进行批量处理，在激活这些能力之前，必须先创建一个 `FScopedServerAbilityRPCBatcher` 结构体。这个特殊的结构体会尝试批量处理在其作用域内的任何后续能力。一旦 `FScopedServerAbilityRPCBatcher` 结构体超出作用域，激活的能力将不再尝试批量处理。`FScopedServerAbilityRPCBatcher` 通过在可以批量处理的每个函数中具有特殊代码来工作，这些代码拦截了发送 RPC 的调用，并将消息打包到批量结构体中。当 `FScopedServerAbilityRPCBatcher` 结构体超出作用域时，它会自动将批量结构体通过 RPC 发送到服务器上的 `UAbilitySystemComponent::EndServerAbilityRPCBatch()`。服务器在 `UAbilitySystemComponent::ServerAbilityRPCBatch_Internal(FServerAbilityRPCBatch& BatchInfo)` 中接收批量 RPC。`BatchInfo` 参数将包含标志，指示能力是否应该结束、激活时是否按下了输入以及 `TargetData`（如果包含的话）。这是一个很好的函数，可以在其中设置断点来确认批量处理是否正常工作。或者，可以使用控制台变量 `AbilitySystem.ServerRPCBatching.Log 1` 来启用特殊的能力批量处理日志记录。

这种机制只能在 C++ 中实现，并且只能通过 `FGameplayAbilitySpecHandle` 来激活能力。

```c++
bool UGSAbilitySystemComponent::BatchRPCTryActivateAbility(FGameplayAbilitySpecHandle InAbilityHandle, bool EndAbilityImmediately) 
{ 
    bool AbilityActivated = false; 
    if (InAbilityHandle.IsValid()) 
    { 
        FScopedServerAbilityRPCBatcher GSAbilityRPCBatcher(this, InAbilityHandle); 
        AbilityActivated = TryActivateAbility(InAbilityHandle, true);
        if (EndAbilityImmediately)
        {
            FGameplayAbilitySpec* AbilitySpec = FindAbilitySpecFromHandle(InAbilityHandle);
            if (AbilitySpec)
            {
                UGSGameplayAbility* GSAbility = Cast<UGSGameplayAbility>(AbilitySpec->GetPrimaryInstance());
                GSAbility->ExternalEndAbility();
            }
        }
        
        return AbilityActivated;
    }

    return AbilityActivated;
}
```

GASShooter 重用了相同的批量处理 `GameplayAbility` 用于半自动和全自动枪械，这些枪械永远不会直接调用 `EndAbility()`（而是由一个仅本地的能力来处理，该能力管理玩家输入并根据当前射击模式触发批量处理的能力）。由于所有 RPC 都必须在 `FScopedServerAbilityRPCBatcher` 的作用域内发生，因此我提供了 `EndAbilityImmediately` 参数，使得控制/管理的仅本地能力可以指定此能力是否应该批量处理 `EndAbility()` 调用（半自动枪械），或者不批量处理 `EndAbility()` 调用（全自动枪械），在这种情况下 `EndAbility()` 调用将在稍后通过自己的 RPC 发生。

GASShooter 提供了一个 Blueprint 节点来允许批量处理能力，上述仅本地的能力使用该节点来触发批量处理的能力。

![Activate Batched Ability](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/batchabilityactivate.png)


**[⬆ 回到顶部](#目录)**

#### 4.6.16 Net Security Policy

A `GameplayAbility`'s `NetSecurityPolicy` determines where should an ability execute on the network. It provides protection from clients attempting to execute restricted abilities.  
  
| `NetSecurityPolicy`     | Description                                                                                                                                        |  
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |  
| `ClientOrServer`        | No security requirements. Client or server can trigger execution and termination of this ability freely.                                           |  
| `ServerOnlyExecution`   | A client requesting execution of this ability will be ignored by the server. Clients can still request that the server cancel or end this ability. |  
| `ServerOnlyTermination` | A client requesting cancellation or ending of this ability will be ignored by the server. Clients can still request execution of the ability.      |  
| `ServerOnly`            | Server controls both execution and termination of this ability. A client making any requests will be ignored.                                      |

**[⬆ 回到顶部](#目录)**

### 4.7 Ability Tasks

### 4.7.1 Ability Task Definition
`GameplayAbilities` 仅在一个帧内执行。这本身并不提供太多的灵活性。为了执行随时间发生的动作或响应在某个时刻触发的委托，我们使用称为 `AbilityTasks` 的延迟动作。

GAS 自带许多 `AbilityTasks`：
* 使用 `RootMotionSource` 移动角色的任务
* 播放动画蒙太奇的任务
* 响应 `Attribute` 变化的任务
* 响应 `GameplayEffect` 变化的任务
* 响应玩家输入的任务
* 以及其他更多

`UAbilityTask` 构造函数强制实施了一个硬编码的游戏全局最大值，即同时运行的最大 `AbilityTasks` 数量为 1000 个。在设计可能同时拥有数百个角色的世界中的 `GameplayAbilities`（如即时战略游戏）时，请记住这一点。

**[⬆ 回到顶部](#目录)**

### 4.7.2 Custom Ability Tasks
通常你会创建自定义的 `AbilityTasks`（在 C++ 中）。示例项目自带两个自定义的 `AbilityTasks`：
1. `PlayMontageAndWaitForEvent` 是默认的 `PlayMontageAndWait` 和 `WaitGameplayEvent` `AbilityTasks` 的组合。这允许动画蒙太奇从 `AnimNotifies` 向启动它们的 `GameplayAbility` 发送游戏事件。使用此功能可以在动画蒙太奇的特定时刻触发动作。
2. `WaitReceiveDamage` 监听 `OwnerActor` 是否受到伤害。被动护甲堆叠的 `GameplayAbility` 在英雄受到伤害实例时移除一层护甲。

`AbilityTasks` 包含以下组成部分：
* 创建新实例的静态函数
* 当 `AbilityTask` 完成其目的时广播的委托
* `Activate()` 函数，用于开始主要工作，绑定外部委托等
* `OnDestroy()` 函数，用于清理，包括它绑定的外部委托
* 对于它绑定的任何外部委托的回调函数
* 成员变量和任何内部辅助函数

**注意：** `AbilityTasks` 只能声明一种类型的输出委托。你所有的输出委托都必须是这种类型，无论是否使用参数。为未使用的委托参数传递默认值。

`AbilityTasks` 仅在运行拥有 `GameplayAbility` 的客户端或服务器上运行；但是，可以通过在 `AbilityTask` 构造函数中设置 `bSimulatedTask = true;`，重写 `virtual void InitSimulatedTask(UGameplayTasksComponent& InGameplayTasksComponent);` 并设置任何成员变量以进行复制的方式，让 `AbilityTasks` 在模拟客户端上运行。这仅在罕见情况下有用，例如移动 `AbilityTasks`，在这种情况下你不希望复制每一次移动变化，而是模拟整个移动 `AbilityTask`。所有 `RootMotionSource` `AbilityTasks` 都这样做。参见 `AbilityTask_MoveToLocation.h/.cpp` 作为示例。

如果在 `AbilityTask` 构造函数中设置 `bTickingTask = true;` 并重写 `virtual void TickTask(float DeltaTime);`，则 `AbilityTasks` 可以进行 `Tick`。当你需要在帧之间平滑地插值值时，这是有用的。参见 `AbilityTask_MoveToLocation.h/.cpp` 作为示例。

**[⬆ 回到顶部](#目录)**

### 4.7.3 Using Ability Tasks
要在 C++ 中创建并激活一个 `AbilityTask`（来自 `GDGA_FireGun.cpp`）：

```c++  
UGDAT_PlayMontageAndWaitForEvent* Task = UGDAT_PlayMontageAndWaitForEvent::PlayMontageAndWaitForEvent(this, NAME_None, MontageToPlay, FGameplayTagContainer(), 1.0f, NAME_None, false, 1.0f);  
Task->OnBlendOut.AddDynamic(this, &UGDGA_FireGun::OnCompleted);  
Task->OnCompleted.AddDynamic(this, &UGDGA_FireGun::OnCompleted);  
Task->OnInterrupted.AddDynamic(this, &UGDGA_FireGun::OnCancelled);  
Task->OnCancelled.AddDynamic(this, &UGDGA_FireGun::OnCancelled);  
Task->EventReceived.AddDynamic(this, &UGDGA_FireGun::EventReceived);  
Task->ReadyForActivation();  
```

在 Blueprint 中，我们只需使用为 `AbilityTask` 创建的 Blueprint 节点。我们不需要调用 `ReadyForActivation()`。这会由 `Engine/Source/Editor/GameplayTasksEditor/Private/K2Node_LatentGameplayTaskCall.cpp` 自动调用。`K2Node_LatentGameplayTaskCall` 还会在你的 `AbilityTask` 类中存在这些函数时自动调用 `BeginSpawningActor()` 和 `FinishSpawningActor()`（参见 `AbilityTask_WaitTargetData`）。再次强调，`K2Node_LatentGameplayTaskCall` 只对 Blueprint 执行自动处理。在 C++ 中，我们必须手动调用 `ReadyForActivation()`，`BeginSpawningActor()` 和 `FinishSpawningActor()`。

![Blueprint WaitTargetData AbilityTask](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/abilitytask.png)

要手动取消一个 `AbilityTask`，只需在 Blueprint 中（称为 `Async Task Proxy`）或 C++ 中对该 `AbilityTask` 对象调用 `EndTask()`。

**[⬆ 回到顶部](#目录)**

### 4.7.4 Root Motion Source Ability Tasks
GAS 自带了使用 `Root Motion Sources` 集成到 `CharacterMovementComponent` 中的 `AbilityTasks`，用于随时间移动 `Characters`，实现诸如击退、复杂跳跃、拉拽和冲刺等功能。

**注意：** 对于引擎版本 4.19 和 4.25+，预测 `RootMotionSource` `AbilityTasks` 是可行的。对于引擎版本 4.20-4.24，预测功能存在问题；然而，`AbilityTasks` 仍然能在多人游戏中正常执行其功能，只是会有轻微的网络修正，并且在单人游戏中完美运行。可以从 4.25 版本中挑选出 [预测修复](https://github.com/EpicGames/UnrealEngine/commit/94107438dd9f490e7b743f8e13da46927051bf33#diff-65f6196f9f28f560f95bd578e07e290c) 并应用到自定义的 4.20-4.24 引擎版本中。

**[⬆ 回到顶部](#目录)**

### 4.8 Gameplay Cues

#### 4.8.1 Gameplay Cue Definition
`GameplayCues` (`GC`) 执行与游戏逻辑无关的操作，如音效、粒子效果、摄像头震动等。`GameplayCues` 通常会被复制（除非明确地本地 `Executed`、`Added` 或 `Removed`）并且可以被预测。

我们通过将带有 **强制父名称 `GameplayCue.`** 的相应 `GameplayTag` 以及事件类型 (`Execute`、`Add` 或 `Remove`) 发送给 `ASC` 中的 `GameplayCueManager` 来触发 `GameplayCues`。`GameplayCueNotify` 对象和其他实现了 `IGameplayCueInterface` 的 `Actors` 可以根据 `GameplayCue` 的 `GameplayTag` (`GameplayCueTag`) 订阅这些事件。

**注意：** 再次强调，`GameplayCue` `GameplayTags` 必须以父 `GameplayTag` `GameplayCue` 开头。例如，一个有效的 `GameplayCue` `GameplayTag` 可能是 `GameplayCue.A.B.C`。

有两种类型的 `GameplayCueNotifies`，即 `Static` 和 `Actor`。它们响应不同的事件，不同类型的 `GameplayEffects` 可以触发它们。覆盖对应的事件来实现你的逻辑。

| `GameplayCue` 类型                                                                                                                     | 事件               | `GameplayEffect` 类型     | 描述                                                                                                                                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------ | ---------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`GameplayCueNotify_Static`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayCueNotify_Static/index.html) | `Execute`        | `Instant` 或 `Periodic`  | `Static` `GameplayCueNotifies` 在 `ClassDefaultObject` 上操作（意味着没有实例），非常适合一次性效果，如撞击效果。                                                                                                                                          |
| [`GameplayCueNotify_Actor`](https://docs.unrealengine.com/en-US/BlueprintAPI/GameplayCueNotify/index.html)                           | `Add` 或 `Remove` | `Duration` 或 `Infinite` | `Actor` `GameplayCueNotifies` 在 `Added` 时会生成一个新的实例。因为这些是实例化的，所以它们可以在被 `Removed` 前执行动作。这些适用于循环播放的声音和粒子效果，当支持它们的 `Duration` 或 `Infinite` `GameplayEffect` 被移除或者手动调用移除时，这些效果也会被移除。这些还提供了选项来管理同时允许有多少个实例，以便多次应用相同的效果时只启动一次声音或粒子。 |

`GameplayCueNotifies` 实际上可以响应任何事件，但通常我们会这样使用它们。

**注意：** 使用 `GameplayCueNotify_Actor` 时，请检查 `Auto Destroy on Remove`，否则后续对同一个 `GameplayCueTag` 的 `Add` 调用将不起作用。

当使用除了 `Full` 之外的 `ASC` [复制模式](#concepts-asc-rm) 时，`Add` 和 `Remove` `GC` 事件会在服务器玩家（监听服务器）上触发两次 —— 一次是在应用 `GE` 时，另一次是从 "Minimal" `NetMultiCast` 到客户端。但是，`WhileActive` 事件仍然只会触发一次。所有事件在客户端上都只会触发一次。

示例项目包括了一个用于眩晕和冲刺效果的 `GameplayCueNotify_Actor`，还有一个用于 FireGun 的弹道撞击的 `GameplayCueNotify_Static`。这些 `GCs` 可以通过 [本地触发](#concepts-gc-local) 而不是通过 `GE` 复制来进一步优化。我在示例项目中选择了展示初学者级别的使用方式。

**[⬆ 回到顶部](#目录)**

#### 4.8.2 Triggering Gameplay Cues
在 `GameplayEffect` 成功应用时（未被标签或免疫阻止），填写所有应被触发的 `GameplayCues` 的 `GameplayTags`。

![GameplayCue Triggered from a GameplayEffect](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/gcfromge.png)

`UGameplayAbility` 提供了 Blueprint 节点来 `Execute`、`Add` 或 `Remove` `GameplayCues`。

![GameplayCue Triggered from a GameplayAbility](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/gcfromga.png)

在 C++ 中，你可以直接在 `ASC` 上调用函数（或在你的 `ASC` 子类中将其暴露给 Blueprint）：

```c++  
/** GameplayCues can also come on their own. These take an optional effect context to pass through hit result, etc */  
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());  
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);  
  
/** Add a persistent gameplay cue */  
void AddGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());  
void AddGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);  
  
/** Remove a persistent gameplay cue */  
void RemoveGameplayCue(const FGameplayTag GameplayCueTag);  
    /** Removes any GameplayCue added on its own, i.e. not as part of a GameplayEffect. */  
void RemoveAllGameplayCues();  
```

**[⬆ 回到顶部](#目录)**

#### 4.8.3 Local Gameplay Cues
从 `GameplayAbilities` 和 `ASC` 触发 `GameplayCues` 的公开函数默认是复制的。每个 `GameplayCue` 事件都是多播 RPC。这可能会导致大量的 RPC。GAS 还限制了每个网络更新最多只能有两个相同的 `GameplayCue` RPC。我们可以通过使用本地 `GameplayCues` 来避免这个问题。本地 `GameplayCues` 只在单独的客户端上 `Execute`、`Add` 或 `Remove`。

我们可以使用本地 `GameplayCues` 的场景：
* 投射物撞击
* 近战碰撞撞击
* 从动画蒙太奇触发的 `GameplayCues`

你应该在你的 `ASC` 子类中添加的本地 `GameplayCue` 函数：

```c++  
UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))  
void ExecuteGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);  
  
UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))  
void AddGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);  
  
UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))  
void RemoveGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);  
```  
  
```c++  
void UPAAbilitySystemComponent::ExecuteGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)  
{  
    UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::Executed, GameplayCueParameters);}  
  
void UPAAbilitySystemComponent::AddGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)  
{  
    UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::OnActive, GameplayCueParameters);    UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::WhileActive, GameplayCueParameters);}  
  
void UPAAbilitySystemComponent::RemoveGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)  
{  
    UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::Removed, GameplayCueParameters);}  
```

如果一个 `GameplayCue` 是本地 `Added` 的，那么它应该被本地 `Removed`。如果它是通过复制 `Added` 的，那么它也应该通过复制来 `Removed`。

**[⬆ 回到顶部](#目录)**

#### 4.8.4 Gameplay Cue Parameters
`GameplayCues` 接收一个包含有关 `GameplayCue` 额外信息的 `FGameplayCueParameters` 结构作为参数。如果你从 `GameplayAbility` 或 `ASC` 上的手动函数触发 `GameplayCue`，则必须手动填充传递给 `GameplayCue` 的 `GameplayCueParameters` 结构。如果 `GameplayCue` 是由 `GameplayEffect` 触发的，则以下变量会自动填充到 `GameplayCueParameters` 结构中：

* AggregatedSourceTags
* AggregatedTargetTags
* GameplayEffectLevel
* AbilityLevel
* [EffectContext](#concepts-ge-context)
* Magnitude（如果 `GameplayEffect` 在 `GameplayCue` 标签容器上方的下拉列表中选择了一个用于 Magnitude 的 `Attribute` 并且有一个相应的 `Modifier` 影响该 `Attribute`）

`GameplayCueParameters` 结构中的 `SourceObject` 变量可能是手动触发 `GameplayCue` 时传递任意数据到 `GameplayCue` 的好地方。

**注意：** 参数结构中的一些变量，如 `Instigator`，可能已经存在于 `EffectContext` 中。`EffectContext` 还可以包含一个 `FHitResult` 以确定在世界中触发 `GameplayCue` 的位置。继承 `EffectContext` 可能是一种向 `GameplayCues` 传递更多数据的好方法，特别是那些由 `GameplayEffect` 触发的 `GameplayCues`。

更多信息，请参见 [`UAbilitySystemGlobals`](#concepts-asg) 中填充 `GameplayCueParameters` 结构的 3 个函数。这些函数是虚函数，因此你可以重写它们来自动生成更多的信息。

```c++  
/** Initialize GameplayCue Parameters */  
virtual void InitGameplayCueParameters(FGameplayCueParameters& CueParameters, const FGameplayEffectSpecForRPC &Spec);  
virtual void InitGameplayCueParameters_GESpec(FGameplayCueParameters& CueParameters, const FGameplayEffectSpec &Spec);  
virtual void InitGameplayCueParameters(FGameplayCueParameters& CueParameters, const FGameplayEffectContextHandle& EffectContext);  
```

**[⬆ 回到顶部](#目录)**

#### 4.8.5 Gameplay Cue Manager
默认情况下，`GameplayCueManager` 会在整个游戏目录中扫描 `GameplayCueNotifies` 并在开始游戏时将它们加载到内存中。我们可以通过在 `DefaultGame.ini` 中设置来更改 `GameplayCueManager` 扫描的路径。

```  
[/Script/GameplayAbilities.AbilitySystemGlobals]  
GameplayCueNotifyPaths="/Game/GASDocumentation/Characters"  
```

我们确实希望 `GameplayCueManager` 能够扫描并找到所有的 `GameplayCueNotifies`；但是，我们不希望它在开始游戏时异步加载每一个 `GameplayCueNotifies`。这样会导致无论这些 `GameplayCueNotifies` 及其引用的声音和粒子是否在关卡中使用，都会被加载到内存中。对于像 Paragon 这样的大型游戏来说，这可能会导致数百兆字节不必要的资产占用内存，并在启动时造成卡顿和游戏冻结。

一种替代方案是在启动时仅异步加载游戏过程中触发的 `GameplayCues`。这样做可以减少不必要的内存使用，并避免因异步加载每个 `GameplayCue` 可能导致的游戏硬冻结，代价是首次触发特定 `GameplayCue` 时可能会出现延迟效果。对于 SSD 来说，这种潜在的延迟是不存在的。我没有在 HDD 上进行过测试。如果在 UE 编辑器中使用此选项，在首次加载 `GameplayCues` 时，如果编辑器需要编译粒子系统，可能会出现轻微的卡顿或冻结。在构建版本中这不是问题，因为粒子系统已经预先编译好了。

首先，我们需要继承 `UGameplayCueManager` 并告诉 `AbilitySystemGlobals` 类使用我们的 `UGameplayCueManager` 子类，在 `DefaultGame.ini` 中进行配置。

```  
[/Script/GameplayAbilities.AbilitySystemGlobals]  
GlobalGameplayCueManagerClass="/Script/ParagonAssets.PBGameplayCueManager"  
```

在我们的 `UGameplayCueManager` 子类中，重写 `ShouldAsyncLoadRuntimeObjectLibraries()`。

```c++  
virtual bool ShouldAsyncLoadRuntimeObjectLibraries() const override  
{  
    return false;
}  
```

**[⬆ 回到顶部](#目录)**

#### 4.8.6 Prevent Gameplay Cues from Firing
有时我们不希望触发 `GameplayCues`。例如，如果我们阻挡了一次攻击，我们可能不希望播放与伤害 `GameplayEffect` 关联的撞击效果，或者想要播放自定义的效果。我们可以在 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 中通过调用 `OutExecutionOutput.MarkGameplayCuesHandledManually()` 来实现这一点，然后手动将我们的 `GameplayCue` 事件发送给 `Target` 或 `Source` 的 `ASC`。

如果你永远不希望在特定的 `ASC` 上触发任何 `GameplayCues`，你可以设置 `AbilitySystemComponent->bSuppressGameplayCues = true;`。

**[⬆ 回到顶部](#目录)**

#### 4.8.7 Gameplay Cue Batching
每个触发的 `GameplayCue` 都是一个不可靠的 NetMulticast RPC。在同时触发多个 `GCs` 的情况下，有一些优化方法可以将它们合并为一个 RPC 或者通过发送更少的数据来节省带宽。

##### 4.8.7.1 Manual RPC
假设你有一把霰弹枪，它可以发射八颗弹丸。这意味着有八个追踪和撞击的 `GameplayCues`。[GASShooter](https://github.com/tranek/GASShooter) 采取了一种较为简单的做法，即将所有追踪信息作为 [`TargetData`](#concepts-targeting-data) 存储在 [`EffectContext`](#concepts-ge-ec) 中，从而将它们合并为一个 RPC。虽然这种方法将 RPC 数量从八个减少到了一个，但它仍然在网络中发送了大量的数据（大约 500 字节）。一个更优化的方法是发送一个带有自定义结构的 RPC，在这个结构中高效地编码撞击位置，或者给接收端提供一个随机种子数以便重现或近似撞击位置。客户端随后解包这个自定义结构，并将其转换为 [本地执行的 `GameplayCues`](#concepts-gc-local)。

工作原理如下：
1. 声明一个 `FScopedGameplayCueSendContext`。这会抑制 `UGameplayCueManager::FlushPendingCues()` 直到它超出作用域，意味着所有的 `GameplayCues` 将被排队直到 `FScopedGameplayCueSendContext` 超出作用域。
2. 重写 `UGameplayCueManager::FlushPendingCues()`，根据一些自定义的 `GameplayTag` 将可以批量处理的 `GameplayCues` 合并到你的自定义结构中，并将其作为 RPC 发送给客户端。
3. 客户端接收到自定义结构后，将其解包成本地执行的 `GameplayCues`。

当你的 `GameplayCues` 需要特定的参数，而这些参数不符合 `GameplayCueParameters` 提供的内容，并且你不想将它们添加到 `EffectContext` 中时（比如伤害数值、暴击指示器、护盾破坏指示器、致命一击指示器等），也可以采用这种方法。

https://forums.unrealengine.com/development-discussion/c-gameplay-programming/1711546-fscopedgameplaycuesendcontext-gameplaycuemanager

##### 4.8.7.2 Multiple GCs on one GE
`GameplayEffect` 上的所有 `GameplayCues` 已经在一个 RPC 中发送。默认情况下，`UGameplayCueManager::InvokeGameplayCueAddedAndWhileActive_FromSpec()` 会发送整个 `GameplayEffectSpec`（但转换为 `FGameplayEffectSpecForRPC`）作为不可靠的 NetMulticast，无论 `ASC` 的 `Replication Mode` 如何。这可能会消耗大量的带宽，具体取决于 `GameplayEffectSpec` 中的内容。我们可以通过设置控制台变量 `AbilitySystem.AlwaysConvertGESpecToGCParams 1` 来优化这一点。这会将 `GameplayEffectSpecs` 转换为 `FGameplayCueParameter` 结构，并将其作为 RPC 发送，而不是发送整个 `FGameplayEffectSpecForRPC`。这可能会节省带宽，但也可能丢失一些信息，这取决于 `GESpec` 如何转换为 `GameplayCueParameters` 以及你的 `GCs` 需要知道哪些信息。

**[⬆ 回到顶部](#目录)**

#### 4.8.8 Gameplay Cue Events
`GameplayCues` 对应特定的 `EGameplayCueEvents`：

| `EGameplayCueEvent` | 描述                                                                                                                                                                                                                                                                                                                         |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OnActive`          | 当 `GameplayCue` 被激活（添加）时调用。                                                                                                                                                                                                                                                                                    |
| `WhileActive`       | 当 `GameplayCue` 处于活动状态时调用，即使它实际上并不是刚刚应用的（如正在进行中的加入等）。这不是 `Tick`！当 `GameplayCueNotify_Actor` 被添加或变得相关时，就像 `OnActive` 一样只调用一次。如果你需要 `Tick()`，可以直接使用 `GameplayCueNotify_Actor` 的 `Tick()`。毕竟它是一个 `AActor`。 |
| `Removed`           | 当 `GameplayCue` 被移除时调用。响应此事件的蓝图 `GameplayCue` 函数是 `OnRemove`。                                                                                                                                                                                                             |
| `Executed`          | 当 `GameplayCue` 执行时调用：即时效果或周期性的 `Tick()`。响应此事件的蓝图 `GameplayCue` 函数是 `OnExecute`。                                                                                                                                                                     |

在 `GameplayCue` 开始时发生的事情，但晚加入的玩家错过也无妨的，使用 `OnActive`。对于 `GameplayCue` 中持续的效果，希望晚加入的玩家也能看到的，使用 `WhileActive`。例如，如果你有一个 MOBA 游戏中塔楼结构爆炸的 `GameplayCue`

**[⬆ 回到顶部](#目录)**

#### 4.8.9 Gameplay Cue Reliability
通常来说，`GameplayCues` 应被视为不可靠的，因此不适合用于直接影响游戏玩法的任何内容。

**执行的 `GameplayCues`:** 这些 `GameplayCues` 通过不可靠的多播应用，并始终是不可靠的。

**从 `GameplayEffects` 应用的 `GameplayCues`:**
* 自主代理可靠地接收 `OnActive`、`WhileActive` 和 `OnRemove`  
`FActiveGameplayEffectsContainer::NetDeltaSerialize()` 调用 `UAbilitySystemComponent::HandleDeferredGameplayCues()` 来调用 `OnActive` 和 `WhileActive`。`FActiveGameplayEffectsContainer::RemoveActiveGameplayEffectGrantedTagsAndModifiers()` 调用 `OnRemoved`。
* 模拟代理可靠地接收 `WhileActive` 和 `OnRemove`  
`UAbilitySystemComponent::MinimalReplicationGameplayCues` 的复制调用 `WhileActive` 和 `OnRemove`。`OnActive` 事件由不可靠的多播调用。

**没有 `GameplayEffect` 应用的 `GameplayCues`:**
* 自主代理可靠地接收 `OnRemove`  
`OnActive` 和 `WhileActive` 事件由不可靠的多播调用。
* 模拟代理可靠地接收 `WhileActive` 和 `OnRemove`  
`UAbilitySystemComponent::MinimalReplicationGameplayCues` 的复制调用 `WhileActive` 和 `OnRemove`。`OnActive` 事件由不可靠的多播调用。

如果你需要 `GameplayCue` 中的某些内容是“可靠的”，那么从 `GameplayEffect` 应用它，并使用 `WhileActive` 添加视觉特效，使用 `OnRemove` 移除视觉特效。

**[⬆ 回到顶部](#目录)**

### 4.9 Ability System Globals
[`AbilitySystemGlobals`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemGlobals/index.html) 类持有有关 GAS 的全局信息。大多数变量都可以从 `DefaultGame.ini` 文件中设置。通常你不需要直接与这个类交互，但你应该了解它的存在。如果你需要对 `GameplayCueManager` 或 `GameplayEffectContext` 这样的类进行子类化，你需要通过 `AbilitySystemGlobals` 来实现。

要对 `AbilitySystemGlobals` 进行子类化，需要在 `DefaultGame.ini` 文件中设置类名：

```  
[/Script/GameplayAbilities.AbilitySystemGlobals]  
AbilitySystemGlobalsClassName="/Script/ParagonAssets.PAAbilitySystemGlobals"  
```

#### 4.9.1 InitGlobalData()
从 UE 4.24 版本开始，现在必须调用 `UAbilitySystemGlobals::Get().InitGlobalData()` 才能使用 [`TargetData`](#concepts-targeting-data)，否则你会遇到与 `ScriptStructCache` 相关的错误，并且客户端将从服务器断开连接。这个函数只需要在一个项目中调用一次。Fortnite 在 `UAssetManager::StartInitialLoading()` 中调用它，而 Paragon 在 `UEngine::Init()` 中调用它。我发现将它放在 `UAssetManager::StartInitialLoading()` 中是一个好位置，正如示例项目所示。我认为这是应该复制到你的项目中的样板代码，以避免 `TargetData` 的问题。

如果你在使用 `AbilitySystemGlobals` 的 `GlobalAttributeSetDefaultsTableNames` 时遇到崩溃，你可能需要像 Fortnite 在 `AssetManager` 或 `GameInstance` 中那样稍后调用 `UAbilitySystemGlobals::Get().InitGlobalData()`。

**[⬆ 回到顶部](#目录)**

### 4.10 Prediction
GAS 自带了对客户端预测的支持；然而，并不是所有内容都会被预测。在 GAS 中，客户端预测意味着客户端不必等待服务器的许可来激活 `GameplayAbility` 并应用 `GameplayEffects`。它可以“预测”服务器会给予它这样的权限，并预测它将应用 `GameplayEffects` 的目标。服务器在网络延迟时间之后运行 `GameplayAbility`，并告诉客户端其预测是否正确。如果客户端在其预测中有误，它将“回滚”其由于“预测错误”而做出的更改以匹配服务器的状态。

关于 GAS 预测的权威来源是插件源代码中的 `GameplayPrediction.h`。

Epic 的理念是只预测那些“可以侥幸成功”的事情。例如，Paragon 和 Fortnite 不预测伤害。它们很可能使用了无法预测的 [`ExecutionCalculations`](#concepts-ge-ec) 来处理伤害。这并不是说你不能尝试预测某些事情，比如伤害。当然，如果你能做到并且效果良好，那非常好。

> ... 我们也没有完全采用一个“无缝且自动预测一切”的解决方案。我们仍然认为玩家预测最好保持在最低限度（意思是：只预测你能侥幸成功的最少的东西）。

*来自 Epic 的 Dave Ratti 对新的 [Network Prediction Plugin](#concepts-p-npp) 的评论*

**被预测的内容：**
> * 能力激活
> * 触发事件
> * `GameplayEffect` 应用：
>    * 属性修改（例外：执行不当前预测，仅限属性修饰符）
>    * `GameplayTag` 修改
> * `GameplayCue` 事件（既包括预测内的 `GameplayEffect` 也包括独立的）
> * 动画序列
> * 移动（内置到 UE 的 `UCharacterMovement`）

**未被预测的内容：**
> * `GameplayEffect` 移除
> * `GameplayEffect` 周期性效果（如持续伤害）

*摘自 `GameplayPrediction.h`*

虽然我们可以预测 `GameplayEffect` 的应用，但我们无法预测 `GameplayEffect` 的移除。一种解决此限制的方法是在想要移除 `GameplayEffect` 时预测相反的效果。例如，如果我们预测了降低 40% 的移动速度，可以通过应用增加 40% 的移动速度来预测性地移除它。然后同时移除这两个 `GameplayEffects`。但这并不适用于所有情况，并且对于预测 `GameplayEffect` 移除的支持仍然是需要的。Epic 的 Dave Ratti 表达了希望在未来版本的 GAS 中添加这一功能的愿望。

因为我们无法预测 `GameplayEffect` 的移除，所以我们无法完全预测 `GameplayAbility` 的冷却时间，并且没有相反的 `GameplayEffect` 解决方案。服务器复制的 `Cooldown GE` 将存在于客户端上，任何试图绕过这一点的行为（例如使用 `Minimal` 复制模式）都将被服务器拒绝。这意味着高延迟的客户端需要更长的时间来告知服务器进入冷却状态，并接收服务器的 `Cooldown GE` 移除。这意味着高延迟的玩家将比低延迟的玩家有更低的射击频率，从而处于不利地位。Fortnite 通过使用自定义记账而不是 `Cooldown GEs` 来避免这个问题。

关于预测伤害，我个人不建议这样做，尽管这是大多数人开始使用 GAS 时首先尝试的事情之一。我特别不建议尝试预测死亡。虽然你可以预测伤害，但这样做很棘手。如果你错误地预测了伤害的应用，玩家会看到敌人的生命值突然回升。如果你尝试预测死亡，这种情况尤其尴尬和令人沮丧。假设你错误地预测了一个 `Character` 的死亡，它开始变成布娃娃状态，但在服务器纠正后又停止了布娃娃状态并继续向你射击。

**注意：** 可以无缝预测改变属性的 `Instant` `GameplayEffects`（如 `Cost GEs`），但对于其他角色的 `Instant` 属性变化则会出现短暂的异常或“闪烁”。预测的 `Instant` `GameplayEffects` 实际上被当作 `Infinite` `GameplayEffects` 处理，以便在预测错误时可以回滚。当服务器的 `GameplayEffect` 应用时，可能会存在两个相同的 `GameplayEffect`，导致修饰符被重复应用或暂时不应用。最终它会自我修正，但有时这种闪烁会被玩家注意到。

GAS 的预测实现试图解决的问题包括：
> 1. “我可以这样做吗？”基本的预测协议。
> 2. “撤销”如何在预测失败时撤销副作用。
> 3. “重做”如何避免重新播放我们本地预测但又从服务器复制过来的副作用。
> 4. “完整性”如何确保我们确实预测了所有的副作用。
> 5. “依赖关系”如何管理相互依赖的预测和预测事件链。
> 6. “覆盖”如何预测性地覆盖原本由服务器复制/拥有的状态。

*摘自 `GameplayPrediction.h`*

**[⬆ 回到顶部](#目录)**
#### 4.10.1 Prediction Key
GAS 的预测机制基于 `Prediction Key` 这一概念，这是一个整数标识符，客户端在激活 `GameplayAbility` 时生成。

* 客户端在激活 `GameplayAbility` 时生成预测键。这是 `Activation Prediction Key`。
* 客户端通过 `CallServerTryActivateAbility()` 将此预测键发送给服务器。
* 客户端将其预测键添加到所有在预测键有效期内应用的 `GameplayEffects` 上。
* 客户端的预测键超出作用域。在同一 `GameplayAbility` 中进一步预测的效果需要一个新的 [Scoped Prediction Window](#concepts-p-windows)。


* 服务器从客户端接收预测键。
* 服务器将其预测键添加到所有应用的 `GameplayEffects` 上。
* 服务器将预测键复制回客户端。


* 客户端接收到带有用于应用的预测键的复制 `GameplayEffects`。如果任何复制的 `GameplayEffects` 与客户端使用相同预测键应用的 `GameplayEffects` 匹配，则说明预测正确。在客户端移除其预测的版本之前，目标上会有两个相同的 `GameplayEffect` 的副本。
* 客户端从服务器接收回预测键。这是 `Replicated Prediction Key`。此预测键现在标记为过期。
* 客户端移除 **所有** 使用现在已过期的复制预测键创建的 `GameplayEffects`。服务器复制的 `GameplayEffects` 将持续存在。客户端添加的任何 `GameplayEffects` 如果没有从服务器接收到匹配的复制版本，则被认为是预测错误的。

预测键保证在 `GameplayAbilities` 中从 `Activation` 开始的一个原子指令组 "窗口" 内有效，该窗口从激活预测键开始。你可以认为这个窗口只在一个帧内有效。任何来自延迟动作 `AbilityTasks` 的回调将不再拥有有效的预测键，除非 `AbilityTask` 有一个内置的同步点，该同步点会生成一个新的 [Scoped Prediction Window](#concepts-p-windows)。

**[⬆ 回到顶部](#目录)**

#### 4.10.2 Creating New Prediction Windows in Abilities
为了在 AbilityTasks 的回调中预测更多动作，我们需要创建一个新的 Scoped Prediction Window 并生成一个新的 Scoped Prediction Key。这有时被称为客户端和服务器之间的同步点（Synch Point）。某些 AbilityTasks，如所有与输入相关的任务，具有内置功能来创建新的 Scoped Prediction Window，这意味着这些 AbilityTasks 回调中的原子代码可以使用有效的 Scoped Prediction Key。其他任务，例如 WaitDelay 任务，并没有内置代码来为其回调创建新的 Scoped Prediction Window。如果你需要在没有内置 Scoped Prediction Window 创建功能的任务之后预测动作，比如 WaitDelay，那么你需要手动执行这一操作，使用带有 OnlyServerWait 选项的 WaitNetSync AbilityTask。当客户端遇到带有 OnlyServerWait 的 WaitNetSync 时，它会根据 GameplayAbility 的激活预测键生成一个新的 Scoped Prediction Key，并通过远程过程调用（RPC）将其发送给服务器，并将其添加到任何新应用的 GameplayEffects 中。当服务器遇到带有 OnlyServerWait 的 WaitNetSync 时，它会等待直到接收到客户端发送的新 Scoped Prediction Key 才继续执行。这个 Scoped Prediction Key 跟激活预测键一样发挥作用：被应用于 GameplayEffects 并复制回客户端以标记为过期。Scoped Prediction Key 在其超出作用范围之前都是有效的，即 Scoped Prediction Window 已关闭。因此，只有原子操作，而不是延迟操作，才能使用新的 Scoped Prediction Key。

你可以根据需要创建任意数量的 Scoped Prediction Windows。

如果你想为自己的自定义 AbilityTasks 添加同步点功能，请参考输入相关任务如何实质上将 WaitNetSync AbilityTask 的代码注入其中。

注意： 使用 WaitNetSync 会导致服务器的 GameplayAbility 暂停执行，直到它接收到客户端的消息。这可能会被恶意用户利用，他们可以通过破解游戏并故意延迟发送新的 Scoped Prediction Key 来滥用这一点。虽然 Epic 对 WaitNetSync 的使用非常谨慎，但它建议如果这是一个担忧的话，可以考虑构建一个 AbilityTask 的新版本，该版本具有延迟自动继续执行的功能，即使没有接收到客户端的消息也是如此。

示例项目在 Sprint GameplayAbility 中使用 WaitNetSync 来每次应用耐力消耗时创建一个新的 Scoped Prediction Window，以便能够预测它。理想情况下，在应用成本和冷却时间时，我们希望有一个有效的预测键。

如果你有一个预测的 GameplayEffect 在客户端上重复播放两次，那么你的预测键可能是过期的，并且你正在经历“重做”问题。通常，你可以在应用 GameplayEffect 之前放置一个带有 OnlyServerWait 的 WaitNetSync AbilityTask 来创建一个新的 Scoped Prediction Key 从而解决这个问题。

**[⬆ 回到顶部](#目录)**

#### 4.10.3 Predictively Spawning Actors
在客户端上预测性地生成 `Actors` 是一个较为复杂的主题。GAS（Gameplay Ability System）本身并不直接提供处理这一功能的能力（`SpawnActor` `AbilityTask` 只会在服务器上生成 `Actor`）。关键的概念是在客户端和服务器上都生成一个可复制的 `Actor`。

如果 `Actor` 仅用于装饰或不服务于实际的游戏玩法目的，简单的解决方案是覆盖 `Actor` 的 `IsNetRelevantFor()` 函数，以限制服务器向拥有者客户端进行复制。这样，拥有者客户端将拥有本地生成的版本，而服务器和其他客户端则拥有服务器复制的版本。

```c++  
bool APAReplicatedActorExceptOwner::IsNetRelevantFor(const AActor * RealViewer, const AActor * ViewTarget, const FVector & SrcLocation) const  
{  
    return !IsOwnedBy(ViewTarget);}  
```

如果生成的 `Actor` 影响游戏玩法，例如需要预测伤害的弹射物，则需要更高级的逻辑，这部分内容超出了本文档的范围。可以参考 Epic Games 在 GitHub 上的 UnrealTournament 项目，了解如何预测性地生成弹射物。在这个项目中，他们会为拥有者客户端生成一个虚拟的弹射物，该弹射物与服务器复制的弹射物同步。

**[⬆ 回到顶部](#目录)**

#### 4.10.4 Future of Prediction in GAS
`GameplayPrediction.h` 提及未来可能增加预测 `GameplayEffect` 移除和周期性 `GameplayEffects` 的功能。

Epic 的 Dave Ratti [表达了兴趣](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89) 在修复预测冷却时间时的 `延迟补偿` 问题，该问题导致高延迟玩家相对于低延迟玩家处于不利地位。

Epic 新推出的 [`Network Prediction` 插件](#concepts-p-npp) 预计将与 GAS 完全兼容，就像之前的 `CharacterMovementComponent` 一样。

**[⬆ 回到顶部](#目录)**

#### 4.10.5 Network Prediction Plugin
Epic 最近启动了一项计划，旨在用新的 `Network Prediction` 插件替换 `CharacterMovementComponent`。此插件仍处于非常早期的阶段，但在 Unreal Engine 的 GitHub 上已可供极早期访问。现在还难以判断它将在哪个未来的引擎版本中首次以实验性的 beta 版本亮相。

**[⬆ 回到顶部](#目录)**

### 4.11 Targeting

#### 4.11.1 Target Data
### `FGameplayAbilityTargetData`

`FGameplayAbilityTargetData` 是一种通用结构，用于存储目标数据，这些数据旨在在网络间传递。`TargetData` 通常会持有 `AActor`/`UObject` 引用、`FHitResults` 以及其他通用的位置、方向和起源信息。然而，你可以通过继承它来自定义几乎任何你想放入的数据，作为一种简单的方式来 [在 `GameplayAbilities` 中传递数据](#concepts-ga-data)。基础结构 `FGameplayAbilityTargetData` 不打算直接使用，而是应该被继承。`GAS` 自带了一些位于 `GameplayAbilityTargetTypes.h` 中的 `FGameplayAbilityTargetData` 子类。

`TargetData` 通常由 [`Target Actors`](#concepts-targeting-actors) 或者 **手动创建**，并通过 [`EffectContext`](#concepts-ge-context) 被 [`AbilityTasks`](#concepts-at) 和 [`GameplayEffects`](#concepts-ge) 消费。由于存在于 `EffectContext` 中，[`Executions`](#concepts-ge-ec)、[`MMCs`](#concepts-ge-mmc)、[`GameplayCues`](#concepts-gc) 以及在 [`AttributeSet`](#concepts-as) 后端的函数都可以访问 `TargetData`。

我们通常不会直接传递 `FGameplayAbilityTargetData`，而是使用一个 `FGameplayAbilityTargetDataHandle`，它内部包含了一个指向 `FGameplayAbilityTargetData` 的 `TArray`。这个中间结构为 `TargetData` 提供了多态支持。

继承自 `FGameplayAbilityTargetData` 的示例：

```c++  
USTRUCT(BlueprintType)  
struct MYGAME_API FGameplayAbilityTargetData_CustomData : public FGameplayAbilityTargetData  
{  
    GENERATED_BODY()public:  
  
    FGameplayAbilityTargetData_CustomData() { }
    
    UPROPERTY()    
    FName CoolName = NAME_None;  
    
    UPROPERTY()    
    FPredictionKey MyCoolPredictionKey; 
     
    // This is required for all child structs of FGameplayAbilityTargetData    
    virtual UScriptStruct* GetScriptStruct() const override    
    {        
        return FGameplayAbilityTargetData_CustomData::StaticStruct();    
    }  
    
    // This is required for all child structs of FGameplayAbilityTargetData    
    bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess)    
    {        
        // The engine already defined NetSerialize for FName & FPredictionKey, thanks Epic!        
        CoolName.NetSerialize(Ar, Map, bOutSuccess);        
        MyCoolPredictionKey.NetSerialize(Ar, Map, bOutSuccess);        
        bOutSuccess = true;        
    
        return true;    
    }
}  
  
template<>  
struct TStructOpsTypeTraits<FGameplayAbilityTargetData_CustomData> : public TStructOpsTypeTraitsBase2<FGameplayAbilityTargetData_CustomData>  
{  
    enum    
    {        
        WithNetSerializer = true // This is REQUIRED for FGameplayAbilityTargetDataHandle net serialization to work    
    };
};  
```

要将目标数据添加到句柄：

```c++  
UFUNCTION(BlueprintPure)  
FGameplayAbilityTargetDataHandle MakeTargetDataFromCustomName(const FName CustomName)  
{  
    // Create our target data type,    
    // Handle's automatically cleanup and delete this data when the handle is destructed,   
    // if you don't add this to a handle then be careful because this deals with memory management and memory leaks so its safe to just always add it to a handle at some point in the frame!  
    FGameplayAbilityTargetData_CustomData* MyCustomData = new             
    FGameplayAbilityTargetData_CustomData();    
    // Setup the struct's information to use the inputted name and any other changes we may want to do    
    MyCustomData->CoolName = CustomName;        
    // Make our handle wrapper for Blueprint usage  
    FGameplayAbilityTargetDataHandle Handle;    
    // Add the target data to our handle    
    Handle.Add(MyCustomData);    
    // Output our handle to Blueprint    
    
    return Handle;
}  
```

为了获取值，需要进行类型安全检查，因为从 `FGameplayAbilityTargetDataHandle` 的目标数据中获取值的唯一方式是使用通用的 C/C++ 类型转换，这种方式 **不是类型安全的**，可能会导致对象切片和崩溃。对于类型检查，有多种方法可以实现（实际上可以根据个人喜好选择），两种常见的方法是：

- **Gameplay 标签**：你可以使用子类层次结构，在这种情况下，你知道任何时候特定代码架构的功能发生时，可以对基类父类型进行类型转换并获取其 Gameplay 标签，然后与这些标签进行比较来确定是否对继承的类进行类型转换。
- **Script 结构体和静态结构体**：你可以直接进行类比较（这可能涉及很多 IF 语句或创建一些模板函数），下面是一个这样的例子。基本上，你可以从任何 `FGameplayAbilityTargetData` 获取 Script 结构体（这是它作为 `USTRUCT` 的一个优点，并要求任何继承的类在 `GetScriptStruct` 中指定结构体类型），并检查它是否是你想要的类型。

```c++  
UFUNCTION(BlueprintPure)  
FName GetCoolNameFromTargetData(const FGameplayAbilityTargetDataHandle& Handle, const int Index)  
{     
    // NOTE, there is two versions of this '::Get(int32 Index)' function;   
    // 1) const version that returns 'const FGameplayAbilityTargetData*', good for reading target data values   
    // 2) non-const version that returns 'FGameplayAbilityTargetData*', good for modifying target data values  
    FGameplayAbilityTargetData* Data = Handle.Get(Index); // This will valid check the index for you      
    // Valid check we have something to use, null data means nothing to cast for  
    if(Data == nullptr)    
    {        
        return NAME_None;    
    }   
     
    // This is basically the type checking pass, static_cast does not have type safety, this is why we do this check.    
    // If we don't do this then it will object slice the struct and thus we have no way of making sure its that type.    
    if(Data->GetScriptStruct() == FGameplayAbilityTargetData_CustomData::StaticStruct())    
	{        
        // Here is when you would do the cast because we know its the correct type already
        FGameplayAbilityTargetData_CustomData* CustomData = static_cast<FGameplayAbilityTargetData_CustomData*>(Data);
                
        return CustomData->CoolName;  
    }
        
    return NAME_None;
}  
```

**[⬆ 回到顶部](#目录)**

#### 4.11.2 Target Actors
`GameplayAbilities` 通过 `WaitTargetData` `AbilityTask` 创建 [`TargetActors`](https://docs.unrealengine.com/zh-CN/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityTargetActor/index.html)，用于可视化并捕获来自游戏世界的瞄准信息。`TargetActors` 可以选择性地使用 [`GameplayAbilityWorldReticles`](#concepts-targeting-reticles) 来显示当前的目标。确认后，瞄准信息将作为 [`TargetData`](#concepts-targeting-data) 返回，并可以传递给 `GameplayEffects`。

`TargetActors` 基于 `AActor`，因此它们可以拥有任何类型的可见组件来表示**位置**和**方式**，例如静态网格体或贴花。静态网格体可用于可视化角色将建造的对象的位置。贴花可用于显示地面的效果范围。示例项目使用了带有地面贴花的 [`AGameplayAbilityTargetActor_GroundTrace`](https://docs.unrealengine.com/zh-CN/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityTargetActor_Grou-/index.html)，来代表流星技能的伤害效果范围。它们也可以不显示任何东西。例如，对于瞬间追踪到目标的命中扫描枪（如在[GASShooter](https://github.com/tranek/GASShooter)中使用的），就没有必要显示任何内容。

它们通过基本的追踪或碰撞重叠来捕获瞄准信息，并根据 `TargetActor` 的实现将结果转换为 `FHitResults` 或 `AActor` 数组形式的 `TargetData`。`WaitTargetData` `AbilityTask` 通过其 `TEnumAsByte<EGameplayTargetingConfirmation::Type>` 参数确定目标何时被确认。当**不是**使用 `TEnumAsByte<EGameplayTargetingConfirmation::Type::Instant>` 时，`TargetActor` 通常会在 `Tick()` 中执行追踪/重叠，并根据其实现更新其位置至 `FHitResult`。虽然这在 `Tick()` 中执行追踪/重叠，但通常来说并不糟糕，因为它不会被复制，而且通常一次只有一个（尽管可能有多个）`TargetActor` 在运行。需要注意的是它使用了 `Tick()`，并且一些复杂的 `TargetActors` 可能会在其中做大量工作，例如 GASShooter 中火箭发射器的二级能力。虽然在 `Tick()` 中进行追踪对客户端非常响应迅速，但如果性能损失太大，你可以考虑降低 `TargetActor` 的帧率。在 `TEnumAsByte<EGameplayTargetingConfirmation::Type::Instant>` 的情况下，`TargetActor` 立即生成、产生 `TargetData` 并销毁。`Tick()` 从未被调用。

| `EGameplayTargetingConfirmation::Type` | 目标何时被确认                                                                                                                                                                                                                                                                                                                                                 |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Instant`                              | 瞄准立即发生，无需特殊逻辑或用户输入决定何时“开火”。                                                                                                                                                                                                                                                                                                        |
| `UserConfirmed`                        | 当用户通过绑定到 `Confirm` 输入的能力或调用 `UAbilitySystemComponent::TargetConfirm()` 来确认瞄准时，瞄准发生。`TargetActor` 还会响应绑定的 `Cancel` 输入或调用 `UAbilitySystemComponent::TargetCancel()` 来取消瞄准。                                                                                                                                          |
| `Custom`                               | 游戏玩法瞄准能力负责通过调用 `UGameplayAbility::ConfirmTaskByInstanceName()` 决定瞄准数据何时准备就绪。`TargetActor` 还会响应 `UGameplayAbility::CancelTaskByInstanceName()` 来取消瞄准。                                                                                                                                            |
| `CustomMulti`                          | 游戏玩法瞄准能力负责通过调用 `UGameplayAbility::ConfirmTaskByInstanceName()` 决定瞄准数据何时准备就绪。`TargetActor` 还会响应 `UGameplayAbility::CancelTaskByInstanceName()` 来取消瞄准。不应在数据生成后结束 `AbilityTask`。                                                                                          |

并非每个 `EGameplayTargetingConfirmation::Type` 都被每个 `TargetActor` 支持。例如，`AGameplayAbilityTargetActor_GroundTrace` 不支持 `Instant` 确认。

`WaitTargetData` `AbilityTask` 接受一个 `AGameplayAbilityTargetActor` 类作为参数，并将在每次激活 `AbilityTask` 时生成一个实例，并在 `AbilityTask` 结束时销毁 `TargetActor`。`WaitTargetDataUsingActor` `AbilityTask` 接收一个已经生成的 `TargetActor`，但在 `AbilityTask` 结束时仍会销毁它。这两种 `AbilityTasks` 都不够高效，因为它们要么生成新的 `TargetActor`，要么要求每次使用时都有新生成的 `TargetActor`。它们非常适合原型设计，但在生产环境中，如果你经常需要产生 `TargetData`（例如自动步枪的情况），你可能会考虑对其进行优化。GASShooter 有一个自定义的 `AGameplayAbilityTargetActor` 子类 [源代码](https://github.com/tranek/GASShooter/blob/master/Source/GASShooter/Public/Characters/Abilities/GSGATA_Trace.h) 和一个新的 `WaitTargetDataWithReusableActor` `AbilityTask` [源代码](https://github.com/tranek/GASShooter/blob/master/Source/GASShooter/Public/Characters/Abilities/AbilityTasks/GSAT_WaitTargetDataUsingActor.h)，允许你重复使用 `TargetActor` 而不必销毁它。

默认情况下，`TargetActors` 不会被复制；但是，如果在游戏中有意义展示本地玩家正在瞄准的位置，可以让其他玩家看到，则可以使其复制。它们确实包括默认功能，通过 `WaitTargetData` `AbilityTask` 上的远程过程调用 (RPC) 与服务器通信。如果 `TargetActor` 的 `ShouldProduceTargetDataOnServer` 属性设置为 `false`，则客户端将在确认时通过 `CallServerSetReplicatedTargetData()` RPC 将其 `TargetData` 发送到服务器，在 `UAbilityTask_WaitTargetData::OnTargetDataReadyCallback()` 中实现。如果 `ShouldProduceTargetDataOnServer` 设置为 `true`，客户端将发送通用确认事件 `EAbilityGenericReplicatedEvent::GenericConfirm` RPC 到服务器，在 `UAbilityTask_WaitTargetData::OnTargetDataReadyCallback()` 中实现，服务器将在接收到 RPC 后执行追踪或重叠检查以在服务器上生成数据。如果客户端取消瞄准，它将发送通用取消事件 `EAbilityGenericReplicatedEvent::GenericCancel` RPC 到服务器，在 `UAbilityTask_WaitTargetData::OnTargetDataCancelledCallback()` 中实现。正如你所见，`TargetActor` 和 `WaitTargetData` `AbilityTask` 上有很多委托。`TargetActor` 响应输入以产生和广播 `TargetData` 准备好、确认或取消的委托。`WaitTargetData` 监听 `TargetActor` 的 `TargetData` 准备好、确认和取消的委托，并将这些信息转发回 `GameplayAbility` 和服务器。如果你将 `TargetData` 发送到服务器，你可能希望在服务器上进行验证，确保 `TargetData` 看起来合理，以防止作弊。直接在服务器上生成 `TargetData` 完全避免了这个问题，但可能会导致拥有客户端的预测不准确。

根据你使用的特定 `AGameplayAbilityTargetActor` 子类的不同，`WaitTargetData` `AbilityTask` 节点上将暴露不同的 `ExposeOnSpawn` 参数。一些常见的参数包括：

| 常见 `TargetActor` 参数 | 定义                                                                                                                                                                                                                                                                                                               |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Debug                   | 如果设置为 `true`，则在非发布构建中每当 `TargetActor` 执行追踪时都会绘制调试追踪/重叠信息。请注意，非 `Instant` 类型的 `TargetActors` 会在 `Tick()` 时执行追踪，因此这些调试绘制调用也会在 `Tick()` 时发生。                                                        |
| Filter                  | [可选] 特殊结构用于在追踪/重叠发生时过滤（移除）目标中的 `Actors`。典型用途包括过滤掉玩家的 `Pawn`、要求目标属于特定类别等。有关更高级用途，请参阅 [目标数据过滤](#concepts-target-data-filters)。 |
| Reticle Class           | [可选] `TargetActor` 将生成的 `AGameplayAbilityWorldReticle` 子类。                                                                                                                                                                                                                                                 |
| Reticle Parameters      | [可选] 配置你的 Reticles。参阅 [Reticles](#concepts-targeting-reticles)。                                                                                                                                                                                                                                             |
| Start Location          | 特殊结构，指定追踪应该从何处开始。通常这将是玩家的视角、武器的枪口或 `Pawn` 的位置。                                                                                                                                                                                                      |

使用默认的 `TargetActor` 类，只有当 `Actors` 直接处于追踪/重叠范围内时，它们才是有效的目标。一旦它们离开追踪/重叠范围（比如移动或你转移视线），它们就不再有效。如果你想让 `TargetActor` 记住最后一个或多个有效目标，你需要在自定义 `TargetActor` 类中添加此功能。我将这些称为持久目标，因为它们将持续存在直到 `TargetActor` 收到确认或取消信号，或者 `TargetActor` 在其追踪/重叠中找到新的有效目标，或者目标不再有效（被销毁）。GASShooter 在其火箭发射器的次级能力——追踪火箭的目标选择中使用了持久目标。

**[⬆ 回到顶部](#目录)**

#### 4.11.3 Target Data Filters
使用 `Make GameplayTargetDataFilter` 和 `Make Filter Handle` 节点，你可以过滤掉玩家的 `Pawn` 或仅选择特定类别。如果你需要更高级的过滤功能，你可以继承 `FGameplayTargetDataFilter` 并覆写 `FilterPassesForActor` 函数。

```c++  
USTRUCT(BlueprintType)  
struct GASDOCUMENTATION_API FGDNameTargetDataFilter : public FGameplayTargetDataFilter  
{  
    GENERATED_BODY()  
    /** Returns true if the actor passes the filter and will be targeted */    
    virtual bool FilterPassesForActor(const AActor* ActorToBeFiltered) const override;
};  
```

然而，这并不能直接应用于 `Wait Target Data` 节点，因为它需要一个 `FGameplayTargetDataFilterHandle`。必须创建一个新的自定义 `Make Filter Handle` 来接受这个子类：

```c++  
FGameplayTargetDataFilterHandle UGDTargetDataFilterBlueprintLibrary::MakeGDNameFilterHandle(FGDNameTargetDataFilter Filter, 
                                                                                            AActor* FilterActor)  
{  
    FGameplayTargetDataFilter* NewFilter = new FGDNameTargetDataFilter(Filter);    
    NewFilter->InitializeFilterContext(FilterActor);  
    FGameplayTargetDataFilterHandle FilterHandle;    
    FilterHandle.Filter = TSharedPtr<FGameplayTargetDataFilter>(NewFilter); 
       
    return FilterHandle;
}  
```

**[⬆ 回到顶部](#目录)**

#### 4.11.4 Gameplay Ability World Reticles
[`AGameplayAbilityWorldReticles`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityWorldReticle/index.html) (`Reticles`) 可视化显示你在使用非 `Instant` 确认的 [`TargetActors`](#concepts-targeting-actors) 进行瞄准时所瞄准的对象是谁。`TargetActors` 负责所有 `Reticles` 的生成和销毁生命周期。`Reticles` 是 `AActors`，因此可以使用任何类型的视觉组件来表示。如 [GASShooter](https://github.com/tranek/GASShooter) 中所示的一个常见实现是使用 `WidgetComponent` 在屏幕空间中显示 UMG Widget（始终面向玩家的相机）。`Reticles` 不知道它们位于哪个 `AActor` 上，但你可以在自定义 `TargetActor` 中实现这一功能。`TargetActors` 通常会在每次 `Tick()` 时更新 `Reticle` 的位置至目标的位置。

GASShooter 使用 `Reticles` 显示火箭发射器次级能力追踪火箭锁定的目标。敌人身上的红色指示器就是 `Reticle`。类似的白色图像则是火箭发射器的准星。
![GASShooter 中的 Reticles](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplayabilityworldreticle.png)

`Reticles` 提供了一些 `BlueprintImplementableEvents` 供设计师使用（它们旨在使用蓝图开发）：

```c++  
/** Called whenever bIsTargetValid changes value. */  
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)  
void OnValidTargetChanged(bool bNewValue);  
  
/** Called whenever bIsTargetAnActor changes value. */  
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)  
void OnTargetingAnActor(bool bNewValue);  
  
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)  
void OnParametersInitialized();  
  
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)  
void SetReticleMaterialParamFloat(FName ParamName, float value);  
  
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)  
void SetReticleMaterialParamVector(FName ParamName, FVector value);  
```

`Reticles` 可以选择性地使用由 `TargetActor` 提供的 [`FWorldReticleParameters`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FWorldReticleParameters/index.html) 进行配置。默认结构只提供了一个变量 `FVector AOEScale`。虽然理论上你可以继承这个结构，但是 `TargetActor` 只会接受基础结构。不允许这个结构被继承似乎有些短视，尤其是在默认的 `TargetActors` 中。然而，如果你创建自己的自定义 `TargetActor`，你可以提供自己的自定义 Reticle 参数结构，并在生成它们时手动传递给 `AGameplayAbilityWorldReticles` 的子类。

`Reticles` 默认情况下不会复制，但如果对你的游戏来说显示本地玩家正在瞄准谁对其他玩家有意义的话，你可以使它们复制。

使用默认的 `TargetActors` 时，`Reticles` 只会在当前有效的目标上显示。例如，如果你使用 `AGameplayAbilityTargetActor_SingleLineTrace` 来追踪目标，`Reticle` 只会在敌人直接处于追踪路径上时出现。如果你转移视线，敌人就不再是有效的目标，`Reticle` 也会消失。如果你想让 `Reticle` 始终停留在最后一个有效的目标上，你需要自定义 `TargetActor` 以便记住最后一个有效的目标并持续在它们身上显示 `Reticle`。我将这些称为持久目标，因为它们将持续存在直到 `TargetActor` 收到确认或取消信号，或者 `TargetActor` 在其追踪/重叠中找到新的有效目标，或者目标不再有效（被销毁）。GASShooter 在其火箭发射器的次级能力追踪火箭的目标选择中使用了持久目标。

**[⬆ 回到顶部](#目录)**

#### 4.11.5 Gameplay Effect Containers Targeting
[`GameplayEffectContainers`](#concepts-ge-containers) 提供了一种可选且高效的产生 [`TargetData`](#concepts-targeting-data) 的方式。这种目标选择在客户端和服务器上应用 `EffectContainer` 时立即进行。它比使用 [`TargetActors`](#concepts-targeting-actors) 更高效，因为它在目标对象的 CDO 上运行（无需生成和销毁 `Actors`），但它缺乏玩家输入，立即发生而无需确认，无法取消，并且不能从客户端向服务器发送数据（在客户端和服务器上都产生数据）。它适用于即时追踪和碰撞重叠。Epic 的 [Action RPG 示例项目](https://www.unrealengine.com/marketplace/en-US/product/action-rpg) 包含了两种使用容器的目标选择示例——针对能力拥有者以及从事件中提取 `TargetData`。它还在蓝图中实现了一种即时球体追踪，追踪位置相对于玩家有一定偏移（由子蓝图类设定）。你可以在 C++ 或蓝图中继承 `URPGTargetType` 来创建自己的目标选择类型。

**[⬆ 回到顶部](#目录)**

## 5. Commonly Implemented Abilities and Effects

### 5.1 Stun
通常对于眩晕效果，我们希望取消角色的所有活动 `GameplayAbilities`，阻止新的 `GameplayAbility` 激活，并在整个眩晕期间禁止移动。示例项目中的 Meteor `GameplayAbility` 在击中目标时施加眩晕效果。

为了取消目标的活动 `GameplayAbilities`，当眩晕 [`GameplayTag` 被添加](#concepts-gt-change)时，我们调用 `AbilitySystemComponent->CancelAbilities()`。

为了防止在眩晕期间激活新的 `GameplayAbilities`，`GameplayAbilities` 在它们的 [`Activation Blocked Tags` `GameplayTagContainer`](#concepts-ga-tags) 中被赋予眩晕 `GameplayTag`。

为了在眩晕期间禁止移动，我们在 `CharacterMovementComponent` 的 `GetMaxSpeed()` 函数中进行覆盖，在拥有眩晕 `GameplayTag` 时返回 0。

**[⬆ 回到顶部](#目录)**

### 5.2 Sprint
示例项目提供了一个如何冲刺的例子——在按住 `Left Shift` 时跑得更快。

更快的移动是由 `CharacterMovementComponent` 通过网络向服务器发送一个标志来进行预测处理的。详情请参阅 `GDCharacterMovementComponent.h/cpp`。

`GA` 处理响应 `Left Shift` 输入，告诉 `CMC` 开始和停止冲刺，并在按住 `Left Shift` 时预测性地消耗耐力。详情请参阅 `GA_Sprint_BP`。

**[⬆ 回到顶部](#目录)**

### 5.3 Aim Down Sights
示例项目处理这种方式与冲刺完全相同，只是减少移动速度而不是增加它。

有关预测性减少移动速度的详细信息，请参阅 `GDCharacterMovementComponent.h/cpp`。

有关处理输入的详细信息，请参阅 `GA_AimDownSight_BP`。瞄准时没有耐力成本。

**[⬆ 回到顶部](#目录)**

### 5.4 Lifesteal
我在伤害的 [`ExecutionCalculation`](#concepts-ge-ec) 内部处理生命偷取。`GameplayEffect` 将具有一个像 `Effect.CanLifesteal` 这样的 `GameplayTag`。`ExecutionCalculation` 检查 `GameplayEffectSpec` 是否具有该 `Effect.CanLifesteal` `GameplayTag`。如果存在该 `GameplayTag`，`ExecutionCalculation` [创建一个动态的 `Instant` `GameplayEffect`](#concepts-ge-dynamic)，其中修改量为要给予的生命值，并将其重新应用于 `Source` 的 `ASC`。

```c++  
if (SpecAssetTags.HasTag(FGameplayTag::RequestGameplayTag(FName("Effect.Damage.CanLifesteal"))))  
{  
    float Lifesteal = Damage * LifestealPercent;  
    UGameplayEffect* GELifesteal = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Lifesteal")));    
    GELifesteal->DurationPolicy = EGameplayEffectDurationType::Instant;  
    int32 Idx = GELifesteal->Modifiers.Num();    
    GELifesteal->Modifiers.SetNum(Idx + 1);    
    FGameplayModifierInfo& Info = GELifesteal->Modifiers[Idx];    
    Info.ModifierMagnitude = FScalableFloat(Lifesteal);    
    Info.ModifierOp = EGameplayModOp::Additive;    
    Info.Attribute = UPAAttributeSetBase::GetHealthAttribute();  
    SourceAbilitySystemComponent->ApplyGameplayEffectToSelf(GELifesteal, 1.0f, SourceAbilitySystemComponent->MakeEffectContext());
}  
```

**[⬆ 回到顶部](#目录)**

### 5.5 Generating a Random Number on Client and Server
有时你需要在 `GameplayAbility` 内生成一个“随机”数字，比如用于子弹后坐力或散布。客户端和服务器都需要生成相同的随机数。为此，我们必须在激活 `GameplayAbility` 时设置相同的 `随机种子`。每次激活 `GameplayAbility` 时，你都需要设置 `随机种子`，以防客户端错误预测激活而导致其随机数序列与服务器不同步。

| 设置种子的方法                                                            | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 使用激活预测键                                                             | `GameplayAbility` 的激活预测键是一个 int16 类型的值，保证在客户端和服务器的 `Activation()` 中同步可用。你可以在客户端和服务器上都将其设置为 `随机种子`。这种方法的缺点是，预测键在每次游戏开始时总是从零开始，并且在生成键之间始终递增使用值。这意味着每场比赛都会有完全相同的随机数序列。这可能足够随机也可能不够随机，取决于你的需求。 |
| 通过事件负载发送种子，当你激活 `GameplayAbility` 时                      | 通过事件激活你的 `GameplayAbility` 并通过复制的事件负载从客户端向服务器发送随机生成的种子。这允许更多的随机性，但客户端很容易通过作弊使其每次都发送相同的种子值。此外，通过事件激活 `GameplayAbilities` 会阻止它们通过输入绑定激活。                                                                                                                                                                    |

如果你的随机偏差很小，大多数玩家可能不会注意到每局游戏的序列都是相同的，使用激活预测键作为 `随机种子` 应该能满足你的需求。如果你正在做更复杂的事情，需要防作弊功能，或许使用 `Server Initiated` `GameplayAbility` 会更好，这样服务器可以创建预测键或生成 `随机种子` 并通过事件负载发送。

**[⬆ 回到顶部](#目录)**

### 5.6 Critical Hits
我在伤害的 [`ExecutionCalculation`](#concepts-ge-ec) 内部处理暴击。`GameplayEffect` 将具有一个像 `Effect.CanCrit` 这样的 `GameplayTag`。`ExecutionCalculation` 检查 `GameplayEffectSpec` 是否具有该 `Effect.CanCrit` `GameplayTag`。如果存在该 `GameplayTag`，`ExecutionCalculation` 生成一个对应于暴击几率的随机数（从 `Source` 捕获的 `Attribute`）并在成功时加上暴击伤害（也是从 `Source` 捕获的 `Attribute`）。因为我没有预测伤害，所以我不必担心同步客户端和服务器上的随机数生成器，因为 `ExecutionCalculation` 只会在服务器上运行。如果你尝试使用 `MMC` 预测性地执行伤害计算，则必须从 `GameplayEffectSpec->GameplayEffectContext->GameplayAbilityInstance` 获取 `随机种子` 的引用。

看看 [GASShooter](https://github.com/tranek/GASShooter) 是如何处理爆头的。概念相同，只是它不依赖随机数来确定几率，而是检查 `FHitResult` 的骨骼名称。

**[⬆ 回到顶部](#目录)**

### 5.7 Non-Stacking Gameplay Effects but Only the Greatest Magnitude Actually Affects the Target
在 Paragon 中，减速效果不会叠加。每个减速实例正常应用并跟踪其生命周期，但只有最大减速效果实际上影响 `Character`。GAS 通过 `AggregatorEvaluateMetaData` 为这种情况提供了开箱即用的支持。有关详细信息和实现，请参阅 [`AggregatorEvaluateMetaData()`](#concepts-as-onattributeaggregatorcreated)。

**[⬆ 回到顶部](#目录)**

### 5.8 Generate Target Data While Game is Paused
如果你需要在等待从玩家的 `WaitTargetData` `AbilityTask` 生成 [`TargetData`](#concepts-targeting-data) 时暂停游戏，我建议不要真正暂停游戏，而是使用 `slomo 0`。

**[⬆ 回到顶部](#目录)**

### 5.9 One Button Interaction System
[GASShooter](https://github.com/tranek/GASShooter) 实现了一个一键交互系统，玩家可以通过按下或按住 'E' 键与可交互对象进行交互，如复活玩家、打开武器箱以及开关滑动门。

**[⬆ 回到顶部](#目录)**

## 6. Debugging GAS
在调试 GAS 相关问题时，你通常想知道以下信息：
> * “我的属性值是多少？”
> * “我有哪些 gameplay 标签？”
> * “我当前有哪些 gameplay 效果？”
> * “我被授予了哪些能力？哪些正在运行？哪些被阻止激活？”。

GAS 提供了两种在运行时解答这些问题的技术 —— [`showdebug abilitysystem`](#debugging-sd) 和在 [`GameplayDebugger`](#debugging-gd) 中的钩子。

**提示：** Unreal Engine 喜欢优化 C++ 代码，这使得调试某些函数变得困难。当你深入追踪代码时，这种情况很少发生。如果你将 Visual Studio 解决方案配置设置为 `DebugGame Editor` 仍然无法追踪代码或检查变量，你可以通过 `UE_DISABLE_OPTIMIZATION` 和 `UE_ENABLE_OPTIMIZATION` 宏或者 CoreMiscDefines.h 中定义的其他变体来禁用所有优化，从而包裹优化过的函数。除非你从源代码重新编译插件，否则不能在插件代码中使用这种方法。这可能对内联函数有效，也可能无效，具体取决于它们的功能和位置。调试完成后，请务必移除这些宏！

```c++  
UE_DISABLE_OPTIMIZATION  
void MyClass::MyFunction(int32 MyIntParameter)  
{  
    // My code}  
UE_ENABLE_OPTIMIZATION  
```

**[⬆ 回到顶部](#目录)**

### 6.1 showdebug abilitysystem
在游戏内的控制台中输入 `showdebug abilitysystem`。此功能分为三个“页面”。这三个页面都会显示你当前拥有的所有 `GameplayTags`。在控制台中输入 `AbilitySystem.Debug.NextCategory` 来切换不同的页面。

第一页显示了你所有 `Attributes` 的 `CurrentValue`：

![First Page of showdebug abilitysystem](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/showdebugpage1.png)

第二页显示了所有作用于你的 `Duration` 和 `Infinite` `GameplayEffects`，包括它们的层数、赋予的 `GameplayTags` 以及赋予的 `Modifiers`。

![First Page of showdebug abilitysystem](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/showdebugpage2.png)

第三页显示了所有已授予你的 `GameplayAbilities`，包括它们是否正在运行、是否被阻止激活，以及当前运行中的 `AbilityTasks` 的状态。

![First Page of showdebug abilitysystem](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/showdebugpage3.png)

要在目标间循环（目标由围绕 Actor 的绿色矩形棱柱表示），使用 `PageUp` 键或 `NextDebugTarget` 控制台命令前往下一个目标，使用 `PageDown` 键或 `PreviousDebugTarget` 控制台命令前往上一个目标。

**注意：** 为了让能力系统信息根据当前选定的调试 Actor 更新，你需要在 `AbilitySystemGlobals` 中将 `bUseDebugTargetFromHud=true` 设置为如下，在 `DefaultGame.ini` 文件中：

```  
[/Script/GameplayAbilities.AbilitySystemGlobals]  
bUseDebugTargetFromHud=true  
```

**注意：** 要使 `showdebug abilitysystem` 命令生效，必须在 GameMode 中选择一个实际的 HUD 类。否则，该命令将找不到，并返回 "Unknown Command"。

**[⬆ 回到顶部](#目录)**

### 6.2 Gameplay Debugger
GAS 为 Gameplay Debugger 添加了功能。通过按 Apostrophe (') 键访问 Gameplay Debugger。通过在数字小键盘上按 3 来启用 Abilities 类别。类别可能会因你安装的插件而有所不同。如果你的键盘（例如笔记本电脑键盘）没有数字小键盘，那么你可以在项目设置中更改快捷键绑定。

当你想查看 **其他** `Characters` 上的 `GameplayTags`、`GameplayEffects` 和 `GameplayAbilities` 时，可以使用 Gameplay Debugger。不幸的是，它不会显示目标 `Attributes` 的 `CurrentValue`。它会针对屏幕中心的任何 `Character`。你可以通过在编辑器的 World Outliner 中选择目标来更改目标，或者通过看向另一个 `Character` 并再次按 Apostrophe (') 键来更改。当前检查的 `Character` 在其上方有一个最大的红色圆圈。

![Gameplay Debugger](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/gameplaydebugger.png)


**[⬆ 回到顶部](#目录)**

### 6.3 GAS Logging
GAS 源代码包含许多不同详细程度的日志记录语句。你最有可能看到的是 `ABILITY_LOG()` 语句。默认的日志详细程度级别是 `Display`。更高级别的日志默认情况下不会在控制台中显示。

要更改日志类别的详细程度级别，可以在控制台中输入：

```  
log [category] [verbosity]  
```

例如，要开启 `ABILITY_LOG()` 语句，你可以在控制台中输入：

```  
log LogAbilitySystem VeryVerbose  
```

要将其重置回默认值，请键入：

```  
log LogAbilitySystem Display  
```

要显示所有日志类别，请键入：

```  
log list  
```

值得注意的 GAS 相关日志类别：

| 日志类别                  | 默认详细程度级别 |
| ------------------------- | ----------------------- |
| LogAbilitySystem          | Display                 |
| LogAbilitySystemComponent | Log                     |
| LogGameplayCueDetails     | Log                     |
| LogGameplayCueTranslator  | Display                 |
| LogGameplayEffectDetails  | Log                     |
| LogGameplayEffects        | Display                 |
| LogGameplayTags           | Log                     |
| LogGameplayTasks          | Log                     |
| VLogAbilitySystem         | Display                 |

更多信息请参阅 [关于日志的 Wiki](https://unrealcommunity.wiki/logging-lgpidy6i)。

**[⬆ 回到顶部](#目录)**

## 7. Optimizations

### 7.1 Ability Batching
能够在一帧内激活、可选地向服务器发送 `TargetData` 并结束的 [`GameplayAbilities`](#concepts-ga) 可以通过 [批处理来将两到三个 RPC 凝缩为一个 RPC](#concepts-ga-batching)。这类能力通常用于命中扫描枪械。

### 7.2 Gameplay Cue Batching
如果你同时发送多个 [`GameplayCues`](#concepts-gc)，考虑 [将它们批处理为一个 RPC](#concepts-gc-batching)。目的是减少 RPC 的数量（`GameplayCues` 是不可靠的 NetMulticast）并尽可能发送最少的数据。

### 7.3 AbilitySystemComponent Replication Mode
默认情况下，[`ASC`](#concepts-asc) 处于 [`Full Replication Mode`](#concepts-asc-rm)。这将把所有 [`GameplayEffects`](#concepts-ge) 复制给每个客户端（这对于单人游戏来说是可以接受的）。在多人游戏中，将玩家拥有的 `ASCs` 设置为 `Mixed Replication Mode`，并将 AI 控制的角色设置为 `Minimal Replication Mode`。这将使得应用在玩家角色上的 `GEs` 只复制给该角色的所有者，而应用在 AI 控制的角色上的 `GEs` 将永远不会向客户端复制 `GEs`。[`GameplayTags`](#concepts-gt) 仍将复制，而 [`GameplayCues`](#concepts-gc) 仍将以不可靠的 NetMulticast 方式发送给所有客户端，无论 `Replication Mode` 如何。这样可以减少不需要看到 `GEs` 的所有客户端上 `GEs` 的网络数据复制。

### 7.4 Attribute Proxy Replication
在像《堡垒之夜大逃杀》（Fortnite Battle Royale, FNBR）这样的大型多人游戏中，会有许多始终相关的 `PlayerStates` 上的 [`ASCs`](#concepts-asc) 复制大量的 [`Attributes`](#concepts-a)。为了优化这一瓶颈，《堡垒之夜》在 `PlayerState::ReplicateSubobjects()` 中完全禁用了 **模拟玩家控制代理** 上的 `ASC` 及其 [`AttributeSets`](#concepts-as) 的复制。自主代理和 AI 控制的 `Pawns` 仍然根据它们的 [`Replication Mode`](#concepts-asc-rm) 完全复制。而不是在始终相关的 `PlayerStates` 上的 `ASC` 上复制 `Attributes`，FNBR 在玩家的 `Pawn` 上使用了一个复制代理结构。当服务器上的 `ASC` 上的 `Attributes` 发生变化时，这些变化也会反映在代理结构上。客户端从代理结构接收复制的 `Attributes` 并将这些变化推送到本地的 `ASC`。这使得 `Attribute` 的复制能够利用 `Pawn` 的相关性和 `NetUpdateFrequency`。这个代理结构还复制了一个经过白名单过滤的小型 `GameplayTags` 集合，使用位掩码进行存储。这种优化减少了网络上的数据量，并允许我们利用 `Pawn` 的相关性。AI 控制的 `Pawns` 已经在其 `Pawn` 上具有 `ASC`，并且已经利用了其相关性，因此这种优化对它们来说不是必需的。

> 我不确定现在是否仍然有必要这样做，因为自那以后已经进行了其他服务器端优化（如 Replication Graph 等），而且这不是最易于维护的模式。

*Epic 的 Dave Ratti 对 [社区问题 #3](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89) 的回答*

### 7.5 ASC Lazy Loading
《堡垒之夜大逃杀》（FNBR）中有大量可被破坏的 `AActors`（树木、建筑物等），每个都有一个 [`ASC`](#concepts-asc)。这会在内存成本上累积。FNBR 通过仅在需要时（即首次被玩家破坏时）懒加载 `ASCs` 来优化这一点。这减少了总体内存使用量，因为有些 `AActors` 在一场比赛中可能永远不会受到损害。

**[⬆ 回到顶部](#目录)**

## 8. Quality of Life Suggestions

### 8.1 Gameplay Effect Containers
[GameplayEffectContainers](#concepts-ge-containers) 将 [`GameplayEffectSpecs`](#concepts-ge-spec)、[`TargetData`](#concepts-targeting-data)、[简单的目标定位](#concepts-targeting-containers) 及相关功能组合成易于使用的结构。这些结构非常适合将 `GameplayEffectSpecs` 传递给从能力中生成的弹体，这些弹体随后会在碰撞时应用效果。

### 8.2 Blueprint AsyncTasks to Bind to ASC Delegates
为了提高设计友好性的迭代时间，特别是在设计 UMG Widgets 用于 UI 时，可以创建 Blueprint AsyncTasks（用 C++ 编写），直接从你的 UMG Blueprint 图表绑定到 `ASC` 上的常用变更委托。唯一的限制是它们必须手动销毁（例如，在小部件被销毁时），否则它们将永远驻留在内存中。示例项目包含了三个 Blueprint AsyncTasks。

监听 `Attribute` 的变化：

![Listen for Attributes Changes BP Node](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/attributeschange.png)

监听冷却时间的变化：

![Listen for Cooldown Change BP Node](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/cooldownchange.png)

`GE` 堆栈的列表更改：

![Listen for GameplayEffect Stack Change BP Node](https://raw.githubusercontent.com/theMeiLin/GASDocumentation5.3_CN/main/Images/gestackchange.png)

**[⬆ 回到顶部](#目录)**

## 9. Troubleshooting

### 9.1 `LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`
你需要在客户端 [初始化 `ASC`](#concepts-asc-setup)。

**[⬆ 回到顶部](#目录)**

### 9.2 `ScriptStructCache` errors
你需要调用 [`UAbilitySystemGlobals::InitGlobalData()`](#concepts-asg-initglobaldata)。

**[⬆ 回到顶部](#目录)**

### 9.3 Animation Montages are not replicating to clients
确保在你的 [GameplayAbilities](#concepts-ga) 中使用 `PlayMontageAndWait` Blueprint 节点而不是 `PlayMontage`。这个 [AbilityTask](#concepts-at) 会自动通过 `ASC` 复制动画序列，而 `PlayMontage` 节点则不会。

**[⬆ 回到顶部](#目录)**

### 9.4 Duplicating Blueprint Actors is setting AttributeSets to nullptr
Unreal Engine 中存在一个 [bug](https://issues.unrealengine.com/issue/UE-81109)，它会导致从现有 Blueprint Actor 类复制得到的 Blueprint Actor 类中的 `AttributeSet` 指针被设置为 nullptr。这个问题有几个解决方法。我成功地避免在我的类中创建自定义的 `AttributeSet` 指针（不在 .h 文件中声明指针，不在构造函数中调用 `CreateDefaultSubobject`），而是直接在 `PostInitializeComponents()` 中将 `AttributeSets` 添加到 `ASC`（示例项目中未展示）。复制的 `AttributeSets` 仍然会存在于 `ASC` 的 `SpawnedAttributes` 数组中。实现方式大致如下：

```c++  
void AGDPlayerState::PostInitializeComponents()  
{  
    Super::PostInitializeComponents(); 
     
    if (AbilitySystemComponent)    
    {       
        AbilitySystemComponent->AddSet<UGDAttributeSetBase>();      
         
        // ... any other AttributeSets that you may have    
    }
}  
```

在这种情况下，你应该使用 `ASC` 上的函数来读取和设置 `AttributeSet` 中的值，而不是 [通过宏生成的函数来调用 `AttributeSet` 上的方法](#concepts-as-attributes)。

```c++  
/** Returns current (final) value of an attribute */  
float GetNumericAttribute(const FGameplayAttribute &Attribute) const;  
  
/** Sets the base value of an attribute. Existing active modifiers are NOT cleared and will act upon the new base value. */  
void SetNumericAttributeBase(const FGameplayAttribute &Attribute, float NewBaseValue);  
```

那么 `GetHealth()` 方法可能会是这样的：

```c++  
float AGDPlayerState::GetHealth() const  
{  
    if (AbilitySystemComponent)    
    {
        return AbilitySystemComponent->GetNumericAttribute(UGDAttributeSetBase::GetHealthAttribute());    
    }
      
    return 0.0f;
}  
```

设置（初始化）健康值 `Attribute` 可能看起来像这样：

```c++  
const float NewHealth = 100.0f;  
if (AbilitySystemComponent)  
{  
    AbilitySystemComponent->SetNumericAttributeBase(UGDAttributeSetBase::GetHealthAttribute(), NewHealth);
}  
```

作为提醒，`ASC` 期望每种 `AttributeSet` 类最多只有一个 `AttributeSet` 对象。

**[⬆ 回到顶部](#目录)**

### 9.5 unresolved external symbol UEPushModelPrivate::MarkPropertyDirty(int,int)
如果收到类似以下内容的编译器错误：

```  
error LNK2019: unresolved external symbol "__declspec(dllimport) void __cdecl UEPushModelPrivate::MarkPropertyDirty(int,int)" (__imp_?MarkPropertyDirty@UEPushModelPrivate@@YAXHH@Z) referenced in function "public: void __cdecl FFastArraySerializer::IncrementArrayReplicationKey(void)" (?IncrementArrayReplicationKey@FFastArraySerializer@@QEAAXXZ)  
```

这是尝试在 `FFastArraySerializer` 上调用 `MarkItemDirty()` 时出现的问题。我在更新 `ActiveGameplayEffect` 时遇到了这个问题，比如更新冷却时间的持续时间。

```c++  
ActiveGameplayEffects.MarkItemDirty(*AGE);  
```

发生的情况是 `WITH_PUSH_MODEL` 在多个地方被定义。`PushModelMacros.h` 将其定义为 0，而在多个其他地方将其定义为 1。`PushModel.h` 将其视为 1，但 `PushModel.cpp` 将其视为 0。

解决方案是在项目的 `Build.cs` 文件中的 `PublicDependencyModuleNames` 添加 `NetCore`。

**[⬆ 回到顶部](#目录)**

### 9.6 Enum names are now represented by path name
如果您收到类似以下内容的编译器警告：

```  
warning C4996: 'FGameplayAbilityInputBinds::FGameplayAbilityInputBinds': Enum names are now represented by path names. Please use a version of FGameplayAbilityInputBinds constructor that accepts FTopLevelAssetPath. Please update your code to the new API before upgrading to the next release, otherwise your project will no longer compile.  
```

Unreal Engine 5.1 已弃用在 `BindAbilityActivationToInputComponent()` 构造函数中使用 `FString`。相反，我们必须传入一个 `FTopLevelAssetPath`。

旧的、已弃用的方式：

```c++  
AbilitySystemComponent->BindAbilityActivationToInputComponent(InputComponent, 
                                                              FGameplayAbilityInputBinds(FString("ConfirmTarget"),  
                                                              FString("CancelTarget"), FString("EGDAbilityInputID"), 
                                                              static_cast<int32>(EGDAbilityInputID::Confirm), 
                                                              static_cast<int32>(EGDAbilityInputID::Cancel))
);  
```

新方式：

```c++  
FTopLevelAssetPath AbilityEnumAssetPath = FTopLevelAssetPath(FName("/Script/GASDocumentation"), 
                                                             FName("EGDAbilityInputID")
);  

AbilitySystemComponent->BindAbilityActivationToInputComponent(InputComponent, 
                                                              FGameplayAbilityInputBinds(FString("ConfirmTarget"),
                                                              FString("CancelTarget"), 
                                                              AbilityEnumAssetPath, 
                                                              static_cast<int32>(EGDAbilityInputID::Confirm), 
                                                              static_cast<int32>(EGDAbilityInputID::Cancel))
);  
```

更多信息，请参阅 `Engine\Source\Runtime\CoreUObject\Public\UObject\TopLevelAssetPath.h`。

**[⬆ 回到顶部](#目录)**

## 10. Common GAS Acronyms

| 名称                                                                                                   | 缩写            |
|------------------------------------------------------------------------------------------------------- | ------------------- |
| AbilitySystemComponent                                                                                 | ASC                 |
| AbilityTask                                                                                            | AT                  |
| [Epic 的 Action RPG 示例项目](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)      | ARPG, ARPG Sample   |
| CharacterMovementComponent                                                                             | CMC                 |
| GameplayAbility                                                                                        | GA                  |
| GameplayAbilitySystem                                                                                  | GAS                 |
| GameplayCue                                                                                            | GC                  |
| GameplayEffect                                                                                         | GE                  |
| GameplayEffectExecutionCalculation                                                                     | ExecCalc, Execution |
| GameplayTag                                                                                            | Tag, GT             |
| ModifierMagnitudeCalculation                                                                           | ModMagCalc, MMC     |

**[⬆ 回到顶部](#目录)**

## 11. Other Resources
* [官方文档](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html)
* 源代码！
   * 特别是 `GameplayPrediction.h`
* [Epic 的 Lyra 示例项目](https://unrealengine.com/marketplace/en-US/learn/lyra)
* [Epic 的 Action RPG 示例项目](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)
* [Unreal Slackers Discord](https://unrealslackers.org/) 有一个专门讨论 GAS 的文字频道 `#gameplay-ability-system`
   * 查看置顶消息
* [Dan 'Pan' 的 GitHub 资源库](https://github.com/Pantong51/GASContent)
* [SabreDartStudios 的 YouTube 视频](https://www.youtube.com/channel/UCCFUhQ6xQyjXDZ_d6X_H_-A)

### 11.1 Q&A With Epic Game's Dave Ratti
[Dave Ratti 回应了 Unreal Slackers Discord Server 社区关于 GAS 的问题](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89):

我们列出了八个问题：
1. 我们如何根据需要在 GameplayAbilities 之外或与 GameplayAbilities 无关的情况下创建范围预测窗口？例如，当一个发射后不管的射弹击中敌人时，如何在本地预测 GameplayEffect 造成的伤害？

PredictionKey 系统实际上并不是为此而设计的。从根本上讲，该系统的工作原理是客户端启动预测操作，使用密钥将其告知服务器，然后客户端和服务器都运行相同的操作并将预测副作用与给定的预测密钥相关联。例如，“我正在预测性地激活一项能力”或“我已经生成了目标数据，并将在 WaitTargetData 任务之后预测性地运行能力图的一部分”。

使用这种模式，PredictionKey 从服务器“反弹”并通过 UAbilitySystemComponent::ReplicatedPredictionKeyMap（复制属性）返回到客户端。一旦密钥从服务器复制回来，客户端就可以撤消所有本地预测副作用（GameplayCues、GameplayEffects）：复制的版本*将存在*，如果不存在，则为错误预测。准确知道​​何时撤消预测副作用在这里至关重要：如果太早，您会看到间隙，如果太晚，您将得到“双重”。 （请注意，这是指状态预测，例如基于持续时间的游戏效果的循环 GameplayCue。“突发”GameplayCues 和即时游戏效果永远不会“撤消”或回滚。如果与它们关联的预测密钥，它们只会在客户端上被跳过）。

为了进一步说明这一点：至关重要的是，预测操作是服务器不会自行执行的操作，而只有在客户端告诉它们时才会执行。因此，除非“某事”是服务器在客户端通知后才会执行的操作，否则通用的“按需创建密钥并告知服务器以便我可以运行某事”是行不通的。

回到最初的问题：类似于发射后就不管的射弹。Paragon 和 Fornite 都有使用 GameplayCues 的射弹演员类。但是，我们不使用预测密钥系统来执行这些操作。相反，我们有一个关于非复制 GameplayCues 的概念。GameplayCues 只是在本地触发并被服务器完全跳过。本质上，所有这些都是直接调用 UGameplayCueManager::HandleGameplayCue。它们不通过 UAbilitySystemComponent 路由，因此不会进行预测密钥检查/早期返回。

非复制 GameplayCues 的缺点是，它们不会被复制。因此，这取决于射弹类/蓝图，以确保调用这些函数的代码路径在每个人身上都运行。我们有启动提示（在 BeginPlay 中调用）、爆炸、击中墙壁/角色等。

这些类型的事件已经在客户端生成，因此调用非复制的游戏提示并不是什么大问题。复杂的蓝图可能很棘手，作者需要确保他们了解在哪里运行什么。

2. 当使用 WaitNetSync AbilityTask 和 OnlyServerWait 在本地预测的 GameplayAbility 中创建范围预测窗口时，由于服务器正在等待带有预测密钥的 RPC，玩家是否可以通过延迟向服务器发送数据包来控制 GameplayAbility 时间，从而进行作弊？这在 Paragon 或 Fortnite 中是否曾出现过问题？如果是，Epic 做了什么来解决这个问题？

是的，这是一个合理的担忧。在服务器上运行的任何等待客户端“信号”的能力蓝图都可能容易受到延迟切换类型漏洞的攻击。

Paragon 有一个类似于 UAbilityTask_WaitTargetData 的自定义定位任务。在这个任务中，我们有超时或“最大延迟”，我们会在客户端等待即时定位模式。如果定位模式正在等待用户确认（按下按钮），那么它将被忽略，因为用户可以慢慢来。但是对于即时确认目标的技能，我们只会等待一段时间，然后 A) 在服务器端生成目标数据 B) 取消该技能。

我们从未有过 WaitNetSync 这样的机制，我们很少使用。

不过，我认为 Fortnite 不会使用任何类似的东西。Fortnite 中的武器技能是特殊情况，批量处理到单个 Fortnite 特定的 RPC：一个 RPC 用于激活技能、提供目标数据和结束技能。因此，在 Battle Royale 中，武器技能本质上不会受到此影响。

我的看法是，这可能是可以在整个系统范围内解决的问题，但我认为我们不会很快做出改变。针对您提到的情况，对 WaitNetSync 进行局部修复以包含最大延迟可能是一项合理的任务，但同样，我们不太可能在不久的将来在我们这边做到这一点。

3. Paragon 和 Fortnite 使用了哪种 EGameplayEffectReplicationMode，Epic 对何时使用它们有什么建议？

这两款游戏基本上都对玩家控制的角色使用混合模式，对 AI 控制的角色（AI 小兵、丛林小兵、AI Husk 等）使用最小模式。这是我建议大多数人在多人游戏中使用该系统的方式。越早设置这些，越好。

Fortnite 在优化方面更进一步。它实际上根本不为模拟代理复制 UAbilitySystemComponent。在拥有 Fortnite 玩家状态类的 ::ReplicateSubobjects() 中跳过组件和属性子对象。我们确实将能力系统组件中的最低限度复制数据推送到 pawn 本身的结构（基本上是属性值的子集和我们在位掩码中复制的标签白名单子集）。我们称之为“代理”。在接收端，我们获取在 pawn 上复制的代理数据，并将其推回到玩家状态的能力系统组件中。因此，FNBR 中确实为每个玩家提供了一个 ASC，但它并不直接复制：而是通过 pawn 上的最小代理结构复制数据，然后路由回接收端的 ASC。这是有优势的，因为它 A) 是一组更简单的数据 B) 利用了 pawn 相关性。

我不确定自那时以来已经完成的其他服务器端优化（复制图等）是否仍然有必要，而且它不是最易于维护的模式。

4. 由于我们无法根据 GameplayPrediction.h 预测 GameplayEffects 的移除，是否有任何策略可以减轻延迟对移除 GameplayEffects 的影响？例如，当移除移动速度慢的元素时，我们目前必须等待服务器复制 GameplayEffect 移除，从而导致玩家角色位置的突然变化。

这是一个棘手的问题，我没有好的答案。我们通常通过容差和平滑来绕过这些问题。我完全同意能力系统和与角色移动系统的精确同步并不好，我们确实想修复它。

我有一个允许预测移除 GE 的架子，但在继续前进之前永远无法解决所有边缘情况。但这并不能解决所有问题，因为角色移动仍然有一个内部保存的移动缓冲区，它对能力系统和可能的移动速度修改器等一无所知。即使无法预测 GE 的移除，仍然有可能进入校正反馈循环。

如果你认为你的情况确实很危急，你可以预测性地添加一个 GE，这会抑制你的移动速度 GE。我自己从来没有这样做过，但之前曾对此进行过理论研究。它可能能够帮助解决某一类问题。

5. 我们知道，AbilitySystemComponent 存在于 Paragon 和 Fortnite 中的 PlayerState 上，以及 Action RPG Sample 中的角色上。Epic 对于 AbilitySystemComponent 应该存在于何处的内部规则、指南或建议是什么——它的所有者应该是谁？

一般来说，我认为任何不需要重生的东西都应该有相同的所有者和 Avatar 角色。任何东西，如 AI 敌人、建筑物、世界道具等。

任何重生的东西都应该有不同的所有者和 Avatar，这样 Ability 系统组件就不需要在重生后保存/重新创建/恢复。

PlayerState 是合乎逻辑的选择，它会复制到所有客户端（而 PlayerController 则不是）。

缺点是 PlayerState 总是相关的，所以你可能会在 100 个玩家的游戏中遇到问题。（请参阅问题 #3 中 FN 所做的事情的注释）。

6. 是否有可能让多个 AbilitySystemComponents 拥有相同的所有者但拥有不同的化身（例如，在 pawn 和武器/物品/射弹上，所有者设置为 PlayerState）？

我看到的第一个问题是在拥有者 Actor 上实现 IGameplayTagAssetInterface 和 IAbilitySystemInterface。前者可能是可行的：只需聚合所有 ASC 中的标签（但要注意 - HasAlMatchingGameplayTags 只能通过跨 ASC 聚合来满足。仅仅将该调用转发给每个 ASC 并将结果一起进行 OR 是不够的）。但后者更加棘手：哪个 ASC 是权威的？如果有人想应用 GE - 哪一个应该接收它？也许你可以解决这些问题，但问题的这一方面将是最困难的：所有者将在其下拥有多个 ASC。

不过，将棋子和武器上的 ASC 分开是有意义的。例如，区分描述武器的标签和描述拥有棋子的标签。也许赋予武器的标签也“适用于”所有者，而不适用于其他任何东西，这确实有意义（例如，属性和 GE 是独立的，但所有者将像我上面描述的那样汇总拥有的标签）。我相信这可以奏效。但拥有同一个所有者的多个 ASC 可能会变得很危险。

7. 有没有办法阻止服务器覆盖拥有客户端上本地预测技能的冷却时间？在高延迟的情况下，当本地冷却时间到期但服务器上仍处于冷却状态时，这将允许拥有客户端“尝试”再次激活该技能。当拥有客户端的激活请求通过网络到达服务器时，服务器可能已冷却完毕，或者服务器可能能够将激活请求排队等待剩余的毫秒数。否则，延迟较高的客户端在重新激活技能之前会比延迟较低的客户端有更长的延迟。这在冷却时间非常短的技能（例如冷却时间可能不到一秒的基本攻击）中最为明显。如果没有办法阻止服务器覆盖本地预测技能的冷却时间，那么 Epic 减轻高延迟对重新激活技能的影响的策略是什么？换个例子来说，Epic 如何设计 Paragon 的基本攻击和其他能力，以便高延迟玩家能够以与低延迟玩家相同的速度通过本地预测进行攻击或激活？

简而言之，没有办法阻止这种情况，Paragon 肯定存在问题。
更高延迟的连接在基本攻击时 ROF 会更低。

我试图通过添加“GE 协调”来解决这个问题，在计算 GE 持续时间时会考虑延迟。本质上允许服务器占用部分总 GE 时间，以便 GE 客户端的有效时间与任何延迟量 100% 一致（尽管波动仍然可能导致问题）。然而，我从来没有让它处于可以发货的状态，项目进展很快，我们从来没有完全解决这个问题。

Fortnite 自己记录武器的射击率：它不使用 GE 来冷却武器。如果这对您的游戏来说是一个严重问题，我会推荐它。

8. Epic 的 GameplayAbilitySystem 插件路线图是什么？
Epic 计划在 2019 年及以后添加哪些功能？
我们觉得目前系统总体上相当稳定，我们没有人负责开发主要的新功能。偶尔会针对 Fortnite 或
UDN/pull 请求进行错误修复和小改进，但目前就是这样。
从长远来看，我认为我们最终会推出“V2”或进行一些重大更改。我们从编写这个系统中学到了很多东西，觉得我们做对了很多，也做错了很多。我很想有机会纠正
这些错误并改进上面指出的一些致命缺陷。
如果 V2 真的要推出，提供升级路径将是至关重要的。我们永远不会制作 V2 并永远将 Fortnite 留在 V1 上：将有一些路径或程序会尽可能自动迁移，尽管几乎肯定仍需要手动重新制作。

优先修复将是：
● 与角色移动系统的更好互操作性。统一客户端预测。
● GE 移除预测（问题 #4）
● GE 延迟协调（问题 #8）
● 通用网络优化，例如批处理 RPC 和代理结构。大部分是我们为 Fortnite 所做的工作，但找到了将其分解为更通用形式的方法，至少这样游戏就可以更轻松地编写自己的游戏特定优化。

我会考虑进行更一般的重构类型的更改：
● 我想从根本上摆脱让 GE 直接引用
电子表格值的做法，相反，它们将能够发出参数，并且这些
参数可以由与电子表格
值绑定的某个更高级别的对象填充。当前模型的问题在于，由于 GE 与曲线表行紧密耦合，因此无法共享。我认为可以编写一个通用的参数化系统，并将其作为 V2 系统的基础。
● 减少 UGameplayAbility 上的“策略”数量。我会删除 ReplicationPolicy
InstancingPolicy。在我看来，Replication 几乎从未真正需要过，而且会造成
混乱。应该通过将
FGameplayAbilitySpec 设为可以子类化的 UObject 来替换 InstancingPolicy。这应该是具有事件且可蓝图化的
“非实例化能力对象”。
UGameplayAbility 应该是“每次执行时实例化”的对象。如果您需要实际实例化，它可以是可选的：相反，“非实例化”能力将通过新的 UGameplayAbilitySpec 对象实现。
● 系统应提供更多“中级”构造，例如“过滤的 GE 应用程序容器”（数据驱动将哪些 GE 应用于具有更高级别游戏逻辑的哪些参与者）、“重叠体积支持”（根据碰撞原始重叠事件应用“过滤的 GE 应用程序容器”）等。这些是每个项目最终以自己的方式实现的构建块。正确处理它们并非易事，因此我认为我们应该更好地提供一些基本实现。
● 总体而言，减少启动和运行项目所需的样板。可能是一个单独的模块“Ex 库”或任何可以提供开箱即用的被动能力或基本命中扫描武器之类的东西。此模块是可选的，但可以让你快速启动并运行。
● 我想将 GameplayCues 移至与能力系统不耦合的单独模块。我认为这里可以做出很多改进。

这只是我的个人意见，不代表任何人的承诺。我认为最现实的行动方案是，随着新引擎技术计划的实施，能力系统将需要更新，那时就是做这种事情的时候。这些计划可能与脚本、网络或物理/角色运动有关。不过，这一切都是长远考虑，所以我无法对时间表做出承诺或估计。

**[⬆ 回到顶部](#目录)**

#### 11.1.2 Community Questions 2




