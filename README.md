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

**[⬆ Back to Top](#table-of-contents)**

## 3.使用 GAS 设置项目
使用 GAS 设置项目的基本步骤：
1. 在编辑器中启用 GameplayAbilitySystem 插件。
2. 编辑`YourProjectName.Build.cs`以将`"GameplayAbilities", "GameplayTags", "GameplayTasks"`添加到`PrivateDependencyModuleNames`中。注：如果后续有报错的情况下可以把`"GameplayAbilities"`移动到`PublicDependencyModuleNames`。
3. 刷新重新生成 Visual Studio 项目文件。
4. 从 4.24 开始，现在必须调用 `UAbilitySystemGlobals::Get().InitGlobalData()` 使用 [`TargetData`](#concepts-targeting-data)。示例项目在`UAssetManager::StartInitialLoading()`中执行此操作。有关更多信息，请参见 [`InitGlobalData()`](#concepts-asg-initglobaldata)。

这就是启用 GAS 所需要做的全部事情。从这里，将[`ASC`](#concepts-asc) 和 [`AttributeSet`](#concepts-as) 添加到您的`Character`或`PlayerState`中并开始制作[`GameplayAbilities`](#concepts-ga) 和 [``GameplayEffects``](#concepts-ge)！

**[⬆ Back to Top](#table-of-contents)**

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
>```
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

**[⬆ Back to Top](#table-of-contents)**

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

**[⬆ Back to Top](#table-of-contents)**

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

**[⬆ Back to Top](#table-of-contents)**

### 4.2.1 响应游戏标签的变化
`ASC`（能力系统组件）提供了当`GameplayTags`被添加或移除时的委托。它接收一个`EGameplayTagEventType`参数，可以指定仅在`GameplayTag`被添加/移除时触发，或者在`GameplayTag`的`TagMapCount`有任何改变时触发。

```c++  
AbilitySystemComponent->RegisterGameplayTagEvent(FGameplayTag::RequestGameplayTag(FName("State.Debuff.Stun")), EGameplayTagEventType::NewOrRemoved).AddUObject(this, &AGDPlayerState::StunTagChanged);  
```

回调函数有一个参数用于接收`GameplayTag`和一个新的`TagCount`值。

```c++  
virtual void StunTagChanged(const FGameplayTag CallbackTag, int32 NewCount);  
```

**[⬆ Back to Top](#table-of-contents)**

### 4.3 Attributes

#### 4.3.1 Attribute 定义
`Attributes`是由结构体[`FGameplayAttributeData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayAttributeData/index.html)定义的浮点数值，它可以代表从游戏角色的生命值到等级，再到药水充能次数等各种游戏相关的数值。如果某个数值是与游戏玩法紧密相关，并且属于某个`Actor`（游戏中的活动实体，如角色、物品等），那么应该考虑使用`Attribute`来表示它。`Attributes`一般应当只由[``GameplayEffects``](#concepts-ge)（游戏玩法效果）进行修改，这样ASC（能力系统组件）才能对这些变化进行[预测](#concepts-p)，确保游戏状态的一致性和正确性。

`Attributes`由[`AttributeSet`](#concepts-as)定义并存储其中。`AttributeSet`负责复制那些标记为需要复制的`Attributes`。关于如何定义`Attributes`，请参阅有关[`AttributeSets`](#concepts-as)的部分。

**提示:** 如果你不希望某个`Attribute`出现在编辑器的`Attributes`列表中，你可以使用`Meta = (HideInDetailsView)``属性修饰符`。

**[⬆ Back to Top](#table-of-contents)**

#### 4.3.2 `BaseValue` vs `CurrentValue`（基础值 vs 当前值）
一个`Attribute`由两个值组成——`BaseValue`（基础值）和`CurrentValue`（当前值）。`BaseValue`是该属性的永久值，而`CurrentValue`则是`BaseValue`加上来自`GameplayEffects`（游戏玩法效果）的临时修改。例如，你的角色可能有一个移动速度属性，其`BaseValue`为600单位/秒。由于目前没有`GameplayEffects`修改移动速度，因此`CurrentValue`也是600单位/秒。如果她获得了一个临时的50单位/秒的移动速度增益，`BaseValue`保持不变，仍为600单位/秒，而`CurrentValue`现在变为600+50，总计650单位/秒。当移动速度增益效果到期后，`CurrentValue`会回退到`BaseValue`的600单位/秒。

GAS（Gameplay Ability System，游戏玩法能力系统）的初学者常常会将`BaseValue`与某一`Attribute`的最大值混淆，并试图将其当作最大值来处理。这是一种错误的做法。对于那些可能变化或在能力（abilities）或用户界面（UI）中引用的`Attributes`的最大值，应当作为独立的`Attributes`来对待。对于硬编码的最大和最小值，可以通过定义带有`FAttributeMetaData`的`DataTable`来设置这些边界值，但是Epic在其结构体上方的注释中称这种方法仍在“开发中”。更多细节可以在`AttributeSet.h`文件中找到。为了避免混淆，我建议那些在能力或UI中可引用的最大值应作为独立的`Attributes`来创建，而仅用于限制`Attributes`范围的硬编码最大和最小值，则应在`AttributeSet`中定义为固定的浮点数。`Attributes`的限制（clamping）在[PreAttributeChange()](#concepts-as-preattributechange)方法中讨论，该方法处理的是`CurrentValue`的变化；而在[PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute)方法中则处理来自`GameplayEffects`的`BaseValue`变化。
>译者注：简单来说，`BaseValue`不应该被误认为是`Attribute`的最大值。如果`Attribute`的最大值是动态的或需要在游戏逻辑中引用，它应该作为一个独立的属性存在。而对于那些仅用于限制`Attribute`范围的固定最大和最小值，可以作为硬编码的浮点数在`AttributeSet`中定义。此外，通过[PreAttributeChange()](#concepts-as-preattributechange)和[PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute)方法，可以分别在`Attribute`的`BaseValue`和`CurrentValue`发生改变时进行限制处理。

对`BaseValue`的永久性改变来源于`Instant`（即时）的`GameplayEffects`，而`Duration`（持续时间）和`Infinite`（无限）的`GameplayEffects`则改变`CurrentValue`。周期性的`GameplayEffects`被视为类似于瞬发的`GameplayEffects`，并且它们也改变`BaseValue`。

**[⬆ Back to Top](#table-of-contents)**

#### 4.3.3 Meta Attributes

一些`Attributes`被当作临时值的占位符，这些值旨在与其它`Attributes`交互，这类属性被称为`Meta Attributes`。例如，我们通常将伤害定义为一种`Meta Attributes`。这样一来，GameplayEffect并不会直接改变我们的生命值`Attributes`，而是使用一个名为伤害的`Meta Attributes`作为占位符。这样做的好处是，伤害值可以在[`GameplayEffectExecutionCalculation`](#concepts-ge-ec)（游戏玩法效果执行计算）中通过增益和减益效果进行修改，并且可以在`AttributeSet`中进一步操作，例如先从当前护盾`Attributes`中扣除伤害值，然后再从生命值`Attributes`中扣除剩余的伤害。伤害`Meta Attributes`在不同的`GameplayEffects`之间不具备持久性，每次都会被新的`GameplayEffect`覆盖。通常情况下，`Meta Attributes`不会在网络间进行复制。

`Meta Attributes`为诸如伤害和治疗这类概念提供了良好的逻辑分离，区分了“我们造成了多少伤害？”和“我们如何处理这些伤害？”这两个问题。这种逻辑上的分离意味着我们的`Gameplay Effects`（游戏玩法效果）和`Execution Calculations`（执行计算）不需要了解目标是如何处理伤害的。以伤害为例，`Gameplay Effect`决定了造成多少伤害，然后`AttributeSet`（属性集）决定如何处理这些伤害。并非所有角色都拥有相同的`Attributes`，特别是当你使用继承自基类的`AttributeSet`时。基类的`AttributeSet`可能只包含生命值`Attributes`，而继承的`AttributeSet`子类可能还会添加护盾`Attributes`。具有护盾`Attributes`的子类`AttributeSet`会以不同于基类AttributeSet的方式来分配所受的伤害。

尽管使用`Meta Attributes`是一种很好的设计模式，但它并非强制性的。如果你的所有伤害实例都只使用同一个`Execution Calculation`（执行计算），并且所有角色共享同一个`AttributeSet`（属性集）类，那么你可能可以直接在`Execution Calculation`内部处理伤害分配至生命值、护盾等，并直接修改这些`Attributes`。这样做，你牺牲的只是灵活性，但这对你来说可能完全是可以接受的。

**[⬆ Back to Top](#table-of-contents)**

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

**[⬆ Back to Top](#table-of-contents)**

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

**[⬆ Back to Top](#table-of-contents)**

### 4.4 Attribute Set
#### 4.4.1 Attribute Set 定义
`AttributeSet` 用于定义、保存和管理对 `Attributes` 的更改。 开发人员应当从 [`UAttributeSet`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAttributeSet/index.html) 继承。 在 `OwnerActor` 的构造函数中创建一个 `AttributeSet` 会自动将其注册到对应的 `ASC` 中。**这项操作必须用 C++ 来完成。**

**[⬆ Back to Top](#table-of-contents)**

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
    DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, PrimaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));    DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, SecondaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));}  
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
    if (!AttributeSet)    {       AttributeSet = NewObject<UGSWeaponAttributeSet>(this);    }    //...}  
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

**[⬆ Back to Top](#table-of-contents)**
