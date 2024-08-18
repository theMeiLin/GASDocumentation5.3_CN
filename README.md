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

从 4.24 版本开始，`PossessedBy()`方法现在会将 `Pawn` 的拥有者设置为新的 `Controller`。

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
