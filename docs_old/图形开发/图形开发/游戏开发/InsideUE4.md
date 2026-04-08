# InsideUE4

## 准备工作

### 关联账户

首先必须将虚幻引擎账户与 GitHub 账户关联，然后才可以访问源码。

![](image-20250610150309134.png)



### 编译源码

下载源码后，先点击 `Setup.bat` 在点击 `GenerateProjectFiles.bat`，然后打开项目，使用默认选项 `DevelopmentEditor` 等待编译完成。



#### 版本对应关系

**[VS2012](https://zhida.zhihu.com/search?content_id=116607435&content_type=Article&match_order=1&q=VS2012&zhida_source=entity) ([Visual C++](https://zhida.zhihu.com/search?content_id=116607435&content_type=Article&match_order=1&q=Visual+C%2B%2B&zhida_source=entity) 11.0)**

- UE4.0 可以使用 VS2012
- UE4.7 Windows 平台最低需求 [VS2013](https://zhida.zhihu.com/search?content_id=116607435&content_type=Article&match_order=1&q=VS2013&zhida_source=entity)

**VS2013 (Visual C++ 12.0)**

- UE4.0 可以使用 VS2013
- UE4.7 Windows 平台最低需求 VS2013
- UE4.10 launcher 版引擎不再支持 VS2013

**VS2015 (Visual C++ 14.0)**

- UE4.9 可以使用 VS2015(Beta)，默认禁用了对 VS2015 的支持
- UE4.10 正式支持 VS2015
- UE4.17 最低需求 [VS2015 Update3](https://zhida.zhihu.com/search?content_id=116607435&content_type=Article&match_order=1&q=VS2015+Update3&zhida_source=entity)

**VS2015 Update3(Visual C++ 14.0)**

- UE4.13 支持 VS2015 Update3
- UE4.22 停止对 VS2015 Update3 的支持

**[VS2017](https://zhida.zhihu.com/search?content_id=116607435&content_type=Article&match_order=1&q=VS2017&zhida_source=entity) (Visual C++ 15.0)**

- UE4.14 支持 VS2017
- UE4.20 默认使用 VS2017

**[VS2019](https://zhida.zhihu.com/search?content_id=116607435&content_type=Article&match_order=1&q=VS2019&zhida_source=entity) (Visual C++ 16.0)**

- UE4.22 添加对 VS2019 的支持，可以在 Editor 中设置为默认 IDE
- UE4.24 对 VS2019 16.3 兼容有问题，不要使用



#### 不兼容问题

如果 UE4 项目显示不兼容，可以先删除项目目录中的

- Binaries
- Intermediate
- .vs 文件夹

重新运行 `GenerateProjectFiles.bat` 打开新的 `UE4.sln` 项目。



#### Rider

使用 Rider 编译 UE4，首先可以通过 File 下的 Settings 从离线文件安装插件

![](image-20250611130557362.png|800)



## 基础概念

### 项目结构

在游戏 Games 文件夹下

- Config 保存配置
- Source 是代码部分

还有未显示在项目中的其它文件夹

- Binaries 编译二进制文件
- Content 保存所有资源和蓝图
- DerivedDataCache (DDC) 保存引擎针对平台特化后的资源版本
- Intermediate 临时文件
- Saved 自动保存文件，包括配置、日志、硬件信息

![](image-20250610154427476.png)



### 编译类型

UE4 包含网络模式和编辑器，因此工程部署时会有 Server 和 Client；开发时还有 Editor 和 Stand-alone 之分；还可以选择是否为 Engine 和 Game 生成调试信息，以及是否在游戏中内嵌控制台。编译选项因此分为

- 状态选项
- 目标选项

两者可以组合

![](image-20250610155029558.png|600)

![](image-20250610155116225.png|600)

一般选择 DebugEditor 加载游戏项目，当需要最简化流程时使用 Debug 运行。



### 代码规范

#### 命名规范

- 命名（如类型或变量）中的每个单词需大写首字母，单词间通常无下划线
- 类型名前缀需使用额外的大写字母，用于区分其和变量命名。例如：`FSkin` 为类型名，而 `Skin` 则是 `FSkin` 的实例。
    - 模板类的前缀为 T
    - 继承自 `UObject` 的类前缀为 U
    - 继承自 `AActor` 的类前缀为 A
    - 继承自 `SWidget` 的类前缀为 S
    - 抽象类的前缀为 I
    - Epic 提供的概念类型的类（用作 `TModels` 的第一个参数），其前缀为 C
    - 枚举的前缀为 E
    - 布尔变量必须以 b 为前缀（例如 `bPendingDestruction` 或 `bHasFadedIn`）
    - 其他多数类均以 F 为前缀，而部分子系统则以其他字母为前缀
    - Typedefs 应以任何与其类型相符的字母为前缀：
	    - 若为结构体的 Typedefs，则使用 F；
	    - 若为 `Uobject` 的Typedefs，则使用 U，以此类推；
	    - 特别模板实例化的Typedef不再是模板，并应加上相应前缀，例如 `typedef TArray<FMytype> FArrayOfMyTypes;	`
    - C# 中省略前缀
- 类型模板参数和基于这些模板参数的嵌套类型别名**不受上述前缀规则的约束**，因为类型的类别是未知的
    - 一般在描述性术语后添加一个 Type 后缀
    - 通过使用 In 前缀，将模板参数与别名区分开来

```cpp
template <typename InElementType>        
class TContainer
{
public:
	using ElementType = InElementType;
};
```

- 宏应该全部大写，用下划线隔开，并以 `UE_` 作为前缀（参见命名空间）
- 若函数参数通过引用传递，同时该值会写入函数，建议以"Out"做为函数参数命名的前缀（非必需）。此操作将明确表明传入该参数的值将被函数替换
- 若 In 或 Out 参数同样为布尔，以 b 作为 In/Out 的前缀，如 `bOutResult`

> >多数情况下，UnrealHeaderTool 需要正确的前缀，因此添加前缀至关重要。



#### 标准库

| 标准库                | 使用场景                                                                                |
| ------------------ | ----------------------------------------------------------------------------------- |
| `atomic`           | 应在新代码中使用，在迁移旧代码时也应该使用                                                               |
| `type_traits`      | 应在旧版 UE 特性（trait）和标准特性重叠的地方使用                                                       |
| `initializer_list` | 必须用于支持初始化器（initializer）语法                                                           |
| `regex`            | 可以直接使用，但其使用应封装在仅和编辑器有关的代码中                                                          |
| `limits`           | `std::numeric_limits` 可以完整使用。                                                       |
| `cmath`            | 这个头文件中只有浮点比较函数可以使用                                                                  |
| `cstring`          | 如果能产生明显的性能改进，建议使用 `memcpy()` 和 `memset()` 而不是 `FMemory::Memcpy` 和 `FMemory::Memset` |



#### 现代 C++ 语法

| 语法               | 解释                              |
| :--------------- | :------------------------------ |
| `static_assert`  | 需要编译时断言时，此关键字有效                 |
| `override final` | 推荐使用此类关键字                       |
| `nullptr`        | 几乎所有情况下均使用 `nullptr`            |
| `auto`           | 不应使用，也包括结构化绑定；除非需要匿名函数变量、迭代器、模板 |
| `range-for`      | 可以使用                            |
| `lambda`         | 可以使用                            |
| `enum class`     | 公开到蓝图的枚举必须基于 `uint8`            |
| `move`           | 移动语义均有效                         |



#### 默认构造

可以直接在类中定义默认参数

```cpp
UCLASS()
class UTeaOptions : public UObject
{
	GENERATED_BODY()

public:
	UPROPERTY()
	int32 MaximumNumberOfCupsPerDay = 10;

	UPROPERTY()
	float CupWidth = 11.5f;

	UPROPERTY()
	FString TeaType = TEXT("Earl Grey");

	UPROPERTY()
	EDrinkingStyle DrinkingStyle = EDrinkingStyle::PinkyExtended;
};
```

这更适用于游戏代码而非引擎代码。



### 编译系统

- UnrealBuildTool 自定义工具，编译逐个模块并处理依赖
- UnrealHeaderTool 代码解析工具，用于将宏生成反射信息

一般 UBT 会先调用 UHT 解析代码，然后用平台编译工具编译，最后启动 Editor 或 Game 。



## GamePlay 架构

![](image-20250611140045721.jpeg)



### Actor 和 Component

- 几乎所有对象继承自 UObject，后者提供基础的垃圾回收、反射、元数据、序列化等
- 世界由 Actor 构建
- 每个 Actor 包含多个 Component
- 通过 Controller 控制 Actor (Pawn) 的行为



#### Actor

Actor 派生自 UObject，并且可以相互嵌套，具有定时调用函数

![](image-20250610164535792.png|300)

为了表示更广泛的对象，例如游戏状态、玩家状态等，Actor 并不直接具有 Transform，而是在 `SceneComponent` 中封装 Transform 。不过大部分 Actor 都可以通过获取 `SceneComponent` 来得到 Transform 。



#### Component

UActorComponent 封装了所属的 Actor，而

- `TSet<UActorComponent*> OwnedComponents` 保存 Actor 拥有的所有 Component，其中会有一个 SceneComponent 作为根组件；
- `TArray<UActorComponent*> InstanceComponents` 保存 Actor 拥有的实例化 Component

一个 Actor 要放入 Level，就必须实例化 `USceneComponent` 作为根组件。

![](image-20250610165121665.png|500)



SceneComponent 提供两大能力

- Transform
- SceneComponent 相互嵌套

相对的，ActorComponent 却不能嵌套。

![](image-20250610171237713.jpeg|500)



#### 父子关系

Actor 之间的嵌套关系通过 Component 确定，即通过 `Child::AttachToActor` 或 `Child::AttachToComponent` 来创建父子连接。例如

```cpp
void AActor::AttachToActor(AActor* ParentActor, const FAttachmentTransformRules& AttachmentRules, FName SocketName)
{
    if (RootComponent && ParentActor)
    {
        USceneComponent* ParentDefaultAttachComponent = ParentActor->GetDefaultAttachComponent();
        if (ParentDefaultAttachComponent)
        {
            RootComponent->AttachToComponent(ParentDefaultAttachComponent, AttachmentRules, SocketName);
        }
    }
}

void AActor::AttachToComponent(USceneComponent* Parent, const FAttachmentTransformRules& AttachmentRules, FName SocketName)
{
    if (RootComponent && Parent)
    {
        RootComponent->AttachToComponent(Parent, AttachmentRules, SocketName);
    }
}
```

注意 `AttachToActor` 调用了 `AttachToComponent` 。

> >由于 Actor 可以具有多个 SceneComponent，因此也具有多个 Transform，不能简单地处理 Actor 的父子关系。因此，实际上 Actor 只作为容器，提供创建、销毁、网格复制、事件触发等逻辑功能，而父子关系维护交给 Component，因此其实是**不同 Actor 的 SceneComponent 之间有父子关系**。



#### ChildActorComponent

ChildActorComponent 提供在 Component 下叠加 Actor 的能力。



### Level 和 World
#### Level

Level 派生自 UObject，它允许 

- ALevelScriptActor 提供脚本支持，AInfo 提供规则属性，两者都派生自 AActor
- 在 Level 保存的 Actors 中，第一个元素就是 AWorldSettings，它派生自 AInfo，并且还单独保存在 WorldSettings 变量中

之所以将 WorldSettings 保存在第一个，是因为 UE 会将网格 Actors 排在后面，从而加速网格复制；而 WorldSettings 几乎不变，因此放在开头；而 LevelScriptActor 代表关卡蓝图，允许复制，可能被排在后面。

![](image-20250610190603259.png|500)



#### World

World 派生自 UObject，它包含多个 Level，其中 Persistent 表示在最开始加载到 World，只有一个；Streaming 表示后续动态加载。在 Levels 中保存当前已经加载的所有 Level，StreamingLevels 保存整个 World 的 Levels 配置列表。

![](image-20250610192145647.png|500)



### WorldContext、GameInstance 和 Engine

#### WorldContext

World 有很多种类型

```cpp
namespace EWorldType
{
	enum Type
	{
		None,		// An untyped world, in most cases this will be the vestigial worlds of streamed in sub-levels
		Game,		// The game world
		Editor,		// A world being edited in the editor
		PIE,		// A Play In Editor world
		Preview,	// A preview world for an editor tool
		Inactive	// An editor world that was loaded but not currently being edited in the level editor
	};
}
```



UE 使用 WorldContext 管理不同的世界

![](image-20250610193039434.png|500)

它保存当前 World 的信息，在需要切换时，就传入 WorldContext 参数。通常，独立运行的游戏只有唯一 WorldContext，而编辑器模式下需要给编辑器和 PIE (Play In Editor) 分别一个 WorldContext 。

> >Level 的切换信息并不放在 World 中，因为一个 World 只有一个 PersistentLevel，而切换 PersistentLevel 时，引擎会删掉之前的 World 重新创建，因此用 WorldContext 管理，可以避免重新构建 Level 。



#### GameInstance

GameInstance 保存当前 WorldContext 和整个游戏信息，因此用它保存与 Level 无关的逻辑和数据，防止 Level 切换导致数据丢失。

![](image-20250610194028901.png|500)



#### Engine

Engine 派生出编辑器 Engine 和游戏 Engine，它们都管理 GameInstance 。事实上，编辑器本身也可以看作是游戏。

> >GEngine 是 UEngine 的实例，是全局变量。

![](image-20250610194452736.png|800)

在基类中保存所有的 WorldContext，也就保存了所有 World 。由于 UE4 不支持同时运行多个 World，所以每一时刻的 GameInstance 都唯一。



### Pawn

Pawn 派生自 Actor，它提供交互反馈，游戏业务逻辑就编写在这里。它提供 3 块基本的模板方法接口

- 可被 Controller 控制
- PhysicsCollision 表示
- MovementInput 的基本响应接口

注意 Actor 也具有 InputComponent 来处理输入。

![](image-20250610195637361.png|400)



从 Pawn 又派生出 

- DefaultPawn 默认 Pawn，自带移动、碰撞、网格
- SpectatorPawn 提供观战能力，无重力漫游，且关闭网格显示
- Character 人形的 Pwan，带有人形移动组件 CharacterMovementComponent、尽量贴合的 CapsuleComponent、带有骨骼蒙皮

![](image-20250610200825748.png|600)



### Controller

对于游戏，需要抽象出三部分

- 显示：游戏 UI、显示的画面或手柄输入振动等玩家直接交互的载体
- 数据：Mesh, Material, Actor, Level 等各种元素组织起来的内存数据表示
- 算法：各种渲染算法、物理模拟、AI 寻路等

这样就得到 MVC 模式

![](image-20250611093609868.png|500)



#### AController

我们希望控制器能够

- 与一个或多个 Pawn 对应
- 可控制多个实例
- 可挂载释放
- 能够脱离 Pawn 存在
- 可以创建或销毁 Pawn
- 根据配置自动生成
- 响应事件
- 持续运行
- 具有状态
- 具有可扩展能力
- 保存数据状态
- 可移动
- 可探查世界中的对象
- 可同步

由于 Pawn 是 Actor 子类，希望独立于 Pawn，就意味着至少不依赖 Actor，因此控制器不能仅仅是 Component；如果直接从 UObject 派生，那么要实现移动，就还需要附加 Component 。因此从 Actor 派生出 Controller 是最佳选择，此时 Controller 与 Pawn 是平级关系，只在运行时引用关联，提高了灵活性。

> >Controller 不能层级嵌套，考虑到其“控制”概念，很难区分层级。



Controller 在其默认构造中禁用了显示，也可以开启。并且 Controller 可以保存位置信息，以便

- 控制 Pawn 移动
- 在当前位置重新生成 Pawn

UE 还提供了 bAttachToPawn，默认关闭。如果开启该选项，可以让 Controller 跟随 Pawn 移动。

> >Controller 用于实现可替换逻辑或智能决策，例如自动追击功能；而 Pawn 实现其自身的固有逻辑，例如移动、动画、碰撞检测等。并且 Controller 生命周期一般更长，例如玩家死亡时，Pawn 会被销毁，然后由 Controller 重建。



#### APlayerState

PlayerState 动态保存**真正的玩家状态**，它与 Pawn、Controller 平级。

> >注意 AInfo 派生自 AActor 。

![](image-20250611104240777.png|300)



#### 总体架构

UE 在 Pawn 这个层级演化出最基本的完善结构

![](image-20250611104709488.png)



#### APlayerController

PlayerController 实现了大量功能

- Camera 管理
- Input 系统
- UPlayer 关联
- HUD 显示：即 UI 显示
- Level 切换
- Voice：聊天语音控制

PlayerController 是玩家直接控制的实体。以马里奥游戏为例

- Pawn 是马里奥实体，处理与金币碰撞的信息，上报给 PlayerController
- 玩家通过 PlayerController 控制马里奥移动，根据碰撞信息更新 PlayerState
- PlayerState 保存玩家信息

因此 PlayerController 只需要在控制 Pawn 的同时处理输入信息。此外，如果有特殊的操作逻辑，例如按特殊按键可以增加金币，则可以通过 PlayerController 处理，在其成员变量中保存每次增加金币的数量。

> >注意 PlayerController 根据关卡的不同，可以替换。如果是联机游戏，那么只有 PlayerState 会被同步，本地不会有 PlayerContorller 。

可以认为 PlayerController 代表玩家意志，PlayerState 代表玩家状态。

![](image-20250611104949324.png|400)



#### AAIController

AI 玩家使用 AIController 作为载体，它包含一些必要组件

- Navigation 用于智能寻路
- AI 组件，启动行为树
- Task 系统，让 AI 完成任务，也是实现 GameplayAbilities 系统的接口

以及前面 PlayerController 的部分组件。

![](image-20250611110401557.png|600)



### GameMode 和 GameState

#### GameMode

UE 中 World 更多的是逻辑概念，代表玩法；Level 是资源场景表示。一个 World 就是一个 Game，玩法称为 Mode，于是 GameMode 从 AInfo 派生出来，承担游戏逻辑。

![](image-20250611110845073.png|600)

GameMode 包括如下功能

- Class 登记：记录游戏基本类型信息，有需要时通过 UClass 反射构建 Pawn 。Controller 类的登记也位于此处，因此 GameMode 比 Controller 更高一级
- 管理 Pawn 的生成和释放 (Spawn)
- 游戏进度
- Level 切换或 World 切换
- 多人游戏同步

一个 World 中只有一个 GameMode，多个 Level 包含一个 PersistentLevel 和多个 StreamingLevel，即使它们配置不同的 GameMode 类，UE 也只会为 PersistentLevel 创建 GameMode 。



#### GameState

GameState 保存游戏状态数据

![](image-20250611111752756.png|400)



#### GameSession

网络联机游戏中针对 Session 使用的方便的管理类，不储存数据。



### Player

#### UPlayer

UPlayer 直接从 UOBject 派生，它管理玩家使用的不同 PlayerController

> >Actor 必须存在于 World 中，而 Player 能够在 World 切换时不变，因此 Player 不能从 Actor 派生；此外，Player 也不需要摆放在 Level 中，也不需要 Component 组装。

![](image-20250611112210151.png|400)



#### ULocalPlayer

本地玩家从 UPlayer 直接派生

![](image-20250611112751844.png|800)



#### UNetConnection

UE 中网络连接也是 Player

![](image-20250611113315279.png|800)



### GameInstance

#### GameInstance

GameInstance 管理全局游戏数据，包括 WorldContext, LocalPlayer 等，它直接派生自 UObject 。

![](image-20250611113427561.png)

一般只有一个 GameInstance，但如果用户同时打开多个窗口，就可能需要多个运行实例。



GameInstance 中可以实现如下功能

- World 和 Level 切换发生在 Engine，而 GameInstance 作为全局变量，也可以管理 World 切换
- 动态添加删除 Players
- 控制 UI
- 全局配置
- 额外的第三方配置



#### SaveGame

得益于 UObject 的序列化机制，只需要继承 USaveGame，并添加需要的属性字段，这个结构就可以序列化保存下来。



### Subsystems

Subsystems 是在 GamePlay 框架基础上的增强功能。它提供 5 个类构成的框架，从它们派生子类，就得到自动实例化和释放的类。

![](image-20250611140141474.jpeg|800)
