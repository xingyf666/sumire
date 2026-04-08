# InsideUE5

## GameFeatures 架构

### 发展由来

GameFeatures 是 GamePlay 发展出的框架。



#### 内建 GamePlay 框架

![](image-20250611102601499.jpeg)



#### GameAbilitySystem 技能框架

![](image-20250611102846596.jpeg|800)



#### Subsystem 系统

Subsystems 给出预定义的 5 个类，允许用户从它们继承出子类，并自动实例化和托管生命周期。

![](image-20250611103104224.jpeg|800)



#### GameFeatures

GameFeatures 是比较特殊的插件，这些插件共同组成游戏的整体玩法。普通插件提供游戏基础功能的封装，而游戏玩法上的插件 GameFeature 就可以提供玩法的实现，通过动态开关插件来切换玩法。

![](image-20250611103309433.jpeg|500)

GameFeature 依托 Plugin 机制实现

![](image-20250611103457972.jpeg|600)

GameFeature 的出现是为了解决 Pawn 中游戏机制过于复杂的问题，所提出的一种模块化组织方式。虽然 Subsystem 和 GAS 在框架的某些方面都提供了解耦的作用，但是 GameFeature 可以更进一步允许在“游戏功能”上解耦。



### 基础用法

#### 开启插件

首先要开启 GameFeatures 和 ModularGameplay 插件。在“编辑-插件”中找到 GamePlay，并启用两个插件

![](image-20250611151329238.png|800)



#### 创建 GameFeature

点击左上角“添加”，然后选择“游戏功能（仅内容）”创建。

> >注意不能修改 GameFeature 的路径。

![](image-20250611151717384.png|800)

此时在内容浏览器中可以看到新建的 GameFeature

![](image-20250611151914024.png|800)

双击 MyGameFeature 可以查看插件状态，点击“激活”来启用插件

![](image-20250611153116726.png|800)


