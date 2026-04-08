# UML

## plantUML

plantUML 是开源的 UML 图绘制工具，支持通过文本来生成图形，使用起来非常高效。可以支持时序图、类图、对象图、活动图、思维导图等图形的绘制。



### 安装插件

我们在 VSCode 中下载 plantUML 插件和 Graphviz Preview 插件，Ctrl + Shift + P 快捷键打开命令面版，输入 plantuml 即可看到相关指令。常用指令是 Preview Current Diagram 预览图表和 Export Current File Diagrams 导出图表。

![[image-20220716160155569.png]]



### [Java 环境](https://blog.csdn.net/pythony_d/article/details/124806543)

使用 plantUML 需要配置 Java 环境，下载 jdk 安装包（已经存放在 D 盘 Java 文件夹中），然后安装完毕。开始配置环境变量，在系统变量中：

1. 新建 `JAVA_HOME` 变量，路径设置为 `D:\Java`
2. 新建 `CLASSPATH` 变量，路径设置为 `.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar`
3. 点击 Path 环境变量，新建环境变量如下：

![[image-20220716161033571.png]]

完成后打开 cmd 输入 `java, javac, java -version` 检查是否正常。



## 部署图

plantUML 提供了一系列元素用于设计结构，部署图如下

```uml
@startuml

actor actor
agent agent
artifact artifact
boundary boundary
card card
cloud cloud
component component
control control
database database
entity entity
file file
folder folder
frame frame
interface  interface
node node
package package
queue queue
stack stack
rectangle rectangle
storage storage
usecase usecase

@enduml
```

![[image-20220716173512783.png]]



## 时序图

### 箭头

我们直接通过代码说明各种可以使用的箭头

```uml
@startuml test

Alice --> Bob : 认证申请
Alice -> Bob

Alice --x Bob
Alice -x Bob

Alice -\\ Bob
Alice -// Bob

Alice ->> Bob

Alice ->o Bob

@enduml
```

效果图如下：

![[image-20220716162026669.png]]

上面的箭头也可以写成 `x->` 等形式，表示将形状放在箭头的左端；还有 `<->` 表示双向箭头。



可以调整箭头的颜色

```uml
@startuml test

Bob -[#red]> Alice : Hello
Alice -[#0000FF]> Bob : Ok

@enduml
```

![[image-20220716172343962.png]]



### 参与者

使用 participant 定义参与者，order 设定参与者顺序

```uml
@startuml test

participant Alice order 30
participant Bob order 20
participant Tom order 10

@enduml
```

![[image-20220716175059051.png]]

也可以使用如下关键字声明，这会改变参与者的外观：

* actor 角色
* boundary 边界
* control 控制
* entity 实体
* database 数据库
* collections 集合
* queue 队列

使用 RGB 颜色或者颜色名就可以改变参与者的背景颜色

```uml
@startuml test

actor Bob #Red
participant Alice
participant "很长很长很长\n的名字" as L #99FF99

Alice -> Bob : 认证请求
Bob -> Alice : 认证响应
Bob -> L : 记录事务日志

@enduml
```

![[image-20220716214433402.png]]



### 别名

可以用 `[` 和 `]` 来表示进入和发出消息，可以配合各种箭头使用，例如

```uml
@startuml test

[--> Bob
Alice -->]

@enduml
```

效果图如下：

![[image-20220716190915435.png]]



有时我们输入的名字很长，就可以用 `as` 语法指定别名

```uml
@startuml test

Alice --> Long as "This is very\nlong"
' 也可以写成 "This is very\nlong" as Long
Long --> "Alice"

@enduml
```

![[image-20220716162753537.png]]

其中双引号用于括住有空格的内容，而单引号开头的文字表示注释。



### 文本对齐

使用 skinparam 命令调整响应信息的位置

```uml
@startuml test

skinparam ResponseMessageBelowArrow true
Bob -> Alice : hello
Alice -> Bob : ok

@enduml
```

![[image-20220716214743782.png]]



### 编号

使用 autonumber 编号格式示例如下：

```uml
@startuml test

' 3 个 0 表示编号有 3 位
autonumber "[000]"
Bob --> Alice : Hello
Bob --> Alice : Hello

' <b> 表示字体加粗，<u> 标签表示下划线
autonumber 155 "<b>(<u>##</u>)"
Bob --> Alice : Hello
Bob --> Alice : Hello

' 10 表示编号间隔，红色字体+加粗
autonumber 40 10 "<font color=red><b>Message 0"
Bob --> Alice : Hello
Bob --> Alice : Hello

@enduml
```

![[image-20220716171725692.png]]

使用 autonumber stop 表示停止编号，resume 用于表示接着停止之前的位置开始编号

```uml
@startuml test

autonumber 10 10 "[000]"
Bob --> Alice : Hello
Bob --> Alice : Hello
autonumber stop

Bob --> Alice : Stop

autonumber resume "<b>(<u>##</u>)"
Bob --> Alice : Hello
Bob --> Alice : Hello
autonumber stop

Bob --> Alice : Stop

autonumber resume 1 "<font color=red><b>Message 0"
Bob --> Alice : Hello
Bob --> Alice : Hello

@enduml
```

![[image-20220716172050310.png]]



### 标题和页面

使用 title 添加标题

```uml
@startuml test

title __Simple__ **communication** example
Alice -> Bon : Authentication Request
Bob -> Alice : Authentication Request

@enduml
```

![[image-20220716210930534.png]]

其中双下划线添加下划线，使用星号加粗；甚至可以添加 HTML 元素

```uml
@startuml test

title
    <u>Simple<\u> communication example
    on <i>several</i> lines and using <font color=red>html</font>
    This is hosted by <img:sourceforge.jpg>
end title

Alice -> Bob : Authentication Request
Bob -> Alice : Authentication Request

@enduml
```

![[image-20220716212126159.png]]

使用 header 增加页眉，footer 增加页脚

```uml
@startuml test

header Page Header
footer Page %page% of %lastpage%

title Example Title

Alice -> Bob : message 1
Alice -> Bob : message 2

@enduml
```

![[image-20220716215014794.png]]



### 分割示意图

使用 newpage 把一张图分割成多张，在其后添加文字，作为新的示意图的标题

```uml
@startuml test

Alice -> Bob : message 1
Alice -> Bob : message 2

newpage

Alice -> Bob : message 3
Alice -> Bob : message 4

newpage A title for the\nlast page

Alice -> Bob : message 5
Alice -> Bob : message 6

@enduml
```

![[image-20220716215217605.png]]

![[image-20220716215229457.png]]

![[image-20220716215259361.png]]



### 组合消息

使用如下关键词组合消息：

* alt/else
* opt
* loop
* par
* break
* critical
* group 后面紧跟消息内容

可以在 header 添加需要显示的文字，end 用来结束分组；分组可以嵌套使用

```uml
@startuml test

Alice -> Bob: 认证请求

alt 成功情况
	Bob -> Alice: 认证接受

else 某种失败情况
    Bob -> Alice: 认证失败

    group 我自己的标签
        Alice -> Log : 开始记录攻击日志

        loop 1000次
            Alice -> Bob: DNS 攻击
        end

        Alice -> Log : 结束记录攻击日志
    end

else 另一种失败
	Bob -> Alice: 请重复

end

@enduml
```

![[image-20220716220453058.png]]

对于 group 而言，在标头处的 `[]` 之间可以显示次级文本或标签

```uml
@startuml test

Alice -> Bob: 认证请求
Bob -> Alice: 认证失败

group 我自己的标签 [我自己的标签2]
    Alice -> Log : 开始记录攻击日志
    
    loop 1000次
    	Alice -> Bob: DNS攻击
    end
    
    Alice -> Log : 结束记录攻击日志
end

@enduml
```

![[image-20220716220807906.png]]



### 消息注释

使用 note 添加注释，还可以指定注释相对的位置

```uml
@startuml test

participant Alice
participant Bob

' 多行注释，同时修改背景色
note left of Alice #aqua
This is displayed 
left of Alice
end note

' 也可以使用 note left 或 note right
note right of Alice : This is displayed right of Alice
note over Alice : This is displayed over Alice

' 修改背景色，同时注释
note over Alice, Bob #FFAAAA : This is displayed\n over Bob and Alice
note over Bob, Alice
This is yet another
example of
a long note
end note

@enduml
```

![[image-20220716174933429.png]]

note 还有 hnote 六边形，rnote 矩形两个版本

```uml
@startuml test

caller -> server : conReq
hnote over caller : idle

caller <- server : conConf
rnote over server
"r" as rectangle
"h" as hexagon
end rnote

@enduml
```

![[image-20220716211549112.png]]

通过 note across 可以在所有参与者之间添加备注

```uml
@startuml test

Alice->Bob:m1
Bob->Charlie:m2

note over Alice, Charlie: 创建跨越所有参与者的备注的旧方法：\n ""note over //FirstPart, LastPart//"".

note across: 新方法：\n""note across""

Bob->Alice
hnote across: 跨越所有参与者的备注。

@enduml
```

![[image-20220716221144073.png]]

利用 \ 在同一级对齐多个注释

```uml
@startuml

note over Alice : Alice的初始状态
/ note over Bob : Bob的初始状态
Bob -> Alice : hello

@enduml
```

![[image-20220716221321106.png]]



### 分隔符

通过 == 关键字分割图表

```uml
@startuml

== 初始化 ==
Alice -> Bob: 认证请求
Bob --> Alice: 认证响应

== 重复 ==
Alice -> Bob: 认证请求
Alice <-- Bob: 认证响应

@enduml
```

![[image-20220716221426909.png]]



### 引用

使用 ref over 实现引用

```uml
@startuml

participant Alice
actor Bob

ref over Alice, Bob : init
Alice -> Bob : hello

ref over Bob
This can be on
several lines
end ref

@enduml
```

![[image-20220716221523774.png]]



### 延迟

使用 ... 表示等待，可以添加文本说明延时时间

```uml
@startuml test

Alice -> Bob : Authentication Request
...
Bob -> Alice : Authentication Request
...5 minutes later...
Bob -> Alice : Bye

@enduml
```

![[image-20220716211226632.png]]



### 增加空间

使用 ||| 增加两个消息之间的竖直空间，还可以指定增加的像素数量

```uml
@startuml test

Alice -> Bob : message 1
Bob -> Alice : ok
|||
Alice -> Bob : message 2
Bob -> Alice : ok
||45||
Alice -> Bob : message 3
Bob -> Alice : ok

@enduml
```

![[image-20220716211831481.png]]



### 生命线

使用 activate 表示激活元素，deactivate 表示回收元素

```uml
@startuml test

participant User

' User 激活 A
User -> A : Dowork
activate A #FFBBBB

' A 激活 A
A -> A : Internal call
activate A #DarkSalmon

' A 激活 B
A -> B : << createRequest >>
activate B

' B 返回消息同时销毁 A B
B --> A : RequestCreated
deactivate B
deactivate A

' A 返回并销毁 A
A -> User: Done
deactivate A

@enduml
```

![[image-20220716173826423.png]]

也可以使用自动激活关键字 autoactivate ，需要与 return 关键字配合

```uml
@startuml

' 开启自动生命周期
autoactivate on
alice -> bob : hello
bob -> bob : self call

bill -> bob #005500 : hello from thread 2
bob -> george ** : create

' 对应 bill->bob 因此返回到 bill
return done in thread 2

' 对应 bob->bob 因此返回到 bob
return rc

bob -> george !! : delete
' 对应 alice -> bob 因此返回到 alice
return success

@enduml
```

![[image-20220716221836153.png]]

可以看到自动激活按照消息传递顺序的相反顺序返回。



### 返回

return 可以用于生成一个带有可选文本标签的返回信息，返回的点是导致最近一次激活生命线的点。语法是简单的返回标签，其中标签（如果提供）可以是传统信息中可以接受的任何字符串

```uml
@startuml

Bob -> Alice : hello

activate Alice
Alice -> Alice : some action
return bye

@enduml
```

![[image-20220716222528272.png]]



### 创建参与者

可以使用 create 关键字创建参与者，放在消息之后表示本次消息创建了新的节点

```uml
@startuml test

Bob -> Alice : Hello
create Other
Alice -> Other : new
' 创建控制节点
create control String
Alice -> String
note right: 添加注释
Alice --> Bob : Ok

@enduml
```

![[image-20220716173124910.png]]



### 快捷语法

在指定目标参与者后，可以立即使用以下语法：

* ++ 激活目标（可选择在后面加上 #color） 
* -- 撤销激活源
* ** 创建目标实例
* !! 摧毁目标实例

```uml
@startuml

alice -> bob ++ : hello
bob -> bob ++ : self call
bob -> bib ++ #005500 : hello
bob -> george ** : create
return done
return rc
bob -> george !! : delete
return success

@enduml
```

![[image-20220716222708989.png]]



### 锚定和持续时间

使用 teoz 在图表中添加锚定，从而指定持续时间

```uml
@startuml

!pragma teoz true

{start} Alice -> Bob : start doing things during duration
Bob -> Max : something
Max -> Bob : something else
{end} Bob -> Alice : finish

' 指定 start 与 end 之间的时间
{start} <-> {end} : some time

@enduml
```

![[image-20220716223120525.png]]



### 构造类型

可以使用 << 和 >> 给参与者 participant 添加构造类型。使用 `(X,color)` 格式语法添加一个小圆圈圈起来的字符

```uml
@startuml test

participant "Famous Bob" as Bob << Generated >>
participant Alice << (C,#ADD1B2) Testable>>

Bob -> Alice : Message

@enduml
```

![[image-20220716172633626.png]]



### 盒子

使用 box 命令添加盒子

```uml
@startuml test

box "Internal Service" #LightBlue
    participant Bob
    participant Alice
end box

participant Other
Bob -> Alice : hello
Alice -> Other : hello

@enduml
```

![[image-20220716210705149.png]]



### 移除脚注

使用 hide footbox 移除脚注

```uml
@startuml

hide footbox
title Footer removed
Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response

@enduml
```

![[image-20220716223347372.png]]



## 用例图

### 用例

用例用圆括号括起来，也可以用 usecase 定义，as 可以定义别名

```uml
@startuml

' 圆括号定义
(First usecase)
(Another usecase) as (UC2)

' usecase 定义
usecase UC3
usecase (Last\nusecase) as UC4

@enduml
```

![[image-20220716223928459.png]]



### 角色

用两个冒号包裹角色，或者 actor 定义角色

```uml
@startuml

' 冒号定义
:First Actor:
:Another\nactor: as Man2

' actor 定义
actor Woman3
actor :Last actor: as Person1

@enduml
```

![[image-20220716224044696.png]]



### 用例描述

如果想定义跨越多行的用例描述，可以用双引号包裹，也可以用分隔符：

* `--` 横线
* `. .` 虚线
* `==` 双横线
* `__` 下划线

并且还可以在分隔符中间放置标题

```uml
@startuml

usecase UC1 as "You can use
several lines to define your usecase.
You can also use separators.

--

Several separators are possible.

==

And you can add titles:
..Conclusion..
This allows large description."

@enduml
```

![[image-20220716224418140.png]]



### 使用包

使用包对用例进行分组

```uml
@startuml

left to right direction

actor Guest as g

package Professional {
    actor Chef as c
    actor "Food Critic" as fc
}

package Restaurant {
    usecase "Eat Food" as UC1
    usecase "Pay for Food" as UC2
    usecase "Drink" as UC3
    usecase "Review" as UC4
}

fc --> UC4
g --> UC1
g --> UC2
g --> UC3

@enduml
```

![[image-20220716224546148.png]]

可以使用 rectangle 改变包的外观

```uml
@startuml

left to right direction

actor "Food Critic" as fc

rectangle Restaurant {
    usecase "Eat Food" as UC1
    usecase "Pay for Food" as UC2
    usecase "Drink" as UC3
}

fc --> UC1
fc --> UC2
fc --> UC3

@enduml
```

![[image-20220716224637539.png]]



### 基础示例

用 --> 连接角色，横杆越多，箭头越长。通过在箭头定义的后面加一个冒号及文字的方式添加标签

```uml
@startuml

User -> (Start)
User --> (Use the application) : A small label
:Main Admin: ---> (Use the application) : This is\nyet another\nlabel

@enduml
```

![[image-20220716224808949.png]]



### 继承

如果一个角色或者用例继承于另一个，可以用 `<|--` 符号表示

```uml
@startuml

:Main Admin: as Admin
(Use the application) as (Use)
User <|-- Admin
(Start) <|-- (Use)

@enduml
```

![[image-20220716234626825.png]]



### 使用注释

通过 note 命令添加注释，还可以用 .. 连接其它对象

```uml
@startuml

:Main Admin: as Admin
(Use the application) as (Use)

User -> (Start)
User --> (Use)
Admin ---> (Use)

note right of Admin : This is an example.
note right of (Use)
    A note can also
    be on several lines
end note

' 定义注释的别名
note "This note is connected\nto several objects." as N2
' 连接对象
(Start) .. N2
N2 .. (Use)

@enduml
```

![[image-20220716234850479.png]]



### 构造类型

用 `<< >>` 定义角色或用例的构造类型

```uml
@startuml

User << Human >>
:Main Database: as MySql << Application >>

(Start) << One Shot >>
(Use the application) as (Use) << Main >>

User -> (Start)
User --> (Use)
MySql --> (Use)

@enduml
```

![[image-20220716234953633.png]]



### 箭头方向

默认用 -- 表示竖直连接，可以用 - 或 . 表示水平连接；通过反转箭头改变方向

```uml
@startuml

:user: --> (Use case 1)
:user: -> (Use case 2)

(Use case 1) <.. :user:
(Use case 2) <- :user:

@enduml
```

![[image-20220716235122740.png]]

也可以用 left, right, up, down 等关键字改变方向

```uml
@startuml

:user: -left-> (dummyLeft)
:user: -right-> (dummyRight)
:user: -up-> (dummyUp)
:user: -down-> (dummyDown)

@enduml
```

![[image-20220716235205203.png]]

也可以用关键字首字母或前两个字母缩写代替，但不建议使用。



### 分割图示

用 newpage 分割多个页面

```uml
@startuml

:actor1: --> (Usecase1)
newpage
:actor2: --> (Usecase2)

@enduml
```



### 改变方向

默认从上向下构建图示，也可以调整

```uml
@startuml

left to right direction
user1 --> (Usecase 1)
user2 --> (Usecase 2)

@enduml
```

![[image-20220716235546225.png]]



### 一个完整的例子

```uml
@startuml

left to right direction

actor customer
actor clerk

rectangle checkout {
customer -- (checkout)
(checkout) .> (payment) : include
(help) .> (checkout) : extends
(checkout) -- clerk
}

@enduml
```

![[image-20220716235533806.png]]



## 类图

### 元素声明

不同类图元素声明如下

```uml
@startuml test

abstract abstract
abstract class "abstract class"
annotation annotation
circle circle
() circle_short_form
class class
diamond diamond
<> diamond_short_form
entity entity
enum enum
interface interface

@enduml
```

![[image-20220716235650074.png]]



### 类关系

类之间的关系通过下面的符号定义：

* `<|--` 扩展
* `*--` 组合
* `o--` 聚合

使用 .. 来代替 -- 可以得到点线

```uml
@startuml

Class01 <|-- Class02
Class03 *-- Class04
Class05 o-- Class06
Class07 .. Class08
Class09 -- Class10

@enduml
```

![[image-20220716235949050.png]]



```uml
@startuml

Class11 <|.. Class12
Class13 --> Class14
Class15 ..> Class16
Class17 ..|> Class18
Class19 <--* Class20

@enduml
```

![[image-20220717000111126.png]]

```uml
@startuml

Class21 #-- Class22
Class23 x-- Class24
Class25 }-- Class26
Class27 +-- Class28
Class29 ^-- Class30

@enduml
```

![[image-20220717000134895.png]]



### 关系上的标签

在关系之间使用标签来说明时，使用 : 后接标签文字；对元素的说明，可以在每一边使用 "" 来说明

```uml
@startuml

Class01 "1" *-- "many" Class02 : contains
Class03 o-- Class04 : aggregation
Class05 --> "1" Class06

@enduml
```

![[image-20220717000308562.png]]



在标签的开始或结束位置添加 < 或 > 以表明是哪个对象作用到哪个对象上

```uml
@startuml

class Car
Driver - Car : drives >
Car *- Wheel : have 4 >
Car -- Person : < owns

@enduml
```

![[image-20220717000423873.png]]



### 添加方法

要声明对象属性或方法，可以使用后段字段名或方法名；系统检查是否有括号来判断是方法还是字段

```uml
@startuml

Object <|-- ArrayList
' 带括号的是方法
Object : equals()
ArrayList : Object[] elementData
ArrayList : size()

@enduml
```

![[image-20220717000607306.png]]

更直观的是用 `{}` 将字段和方法括起来

```uml
@startuml

class Dummy {
    String data
    void methods()
}

class Flight {
    flightNumber : Integer
    departureTime : Date
}

@enduml
```

![[image-20220717000728442.png]]



可以显式地用 `{field}` 和 `{method}` 来覆盖解析器默认的识别

```uml
@startuml

class Dummy {
    {field} A field (despite parentheses)
    {method} Some method
}

@enduml
```

![[image-20220717000858327.png]]



### 定义可访问性

可以用不同字符确定可访问程度

* `-` private
* `#` protected
* `~` package private
* `+` public

```uml
@startuml

class Dummy {
    -field1
    #field2
    ~method1()
    +method2()
}

@enduml
```

![[image-20220717001031309.png]]



### 抽象与静态

使用 `{static}` 和 `{abstract}` 声明静态和抽象方法或属性

```uml
@startuml test

class Dummy {
    {static} String id
    {abstract} void methods()
}

@enduml
```

![[image-20220717001241949.png]]



### 高级类体

plantUML 默认自动将方法和属性分组，但是我们可以自定义分隔符来重排方法和属性

```uml
@startuml

class Foo1 {
    You can use
    several lines
    ..
    as you want
    and group
    ==
    things together.
    __
    You can have as many groups
    as you want
    --
    End of class
}

class User {
    .. Simple Getter ..
    + getName()
    + getAddress()
    .. Some setter ..
    + setName()
    __ private data __
    int age
    -- encrypted --
    String password
}

@enduml
```

![[image-20220717001418615.png]]



### 备注和模板

模板通过 `<<` 和 `>>` 定义；可以使用 note 添加备注，使用 .. 符号可以作出连接虚线

```uml
@startuml

class Object << general >>

Object <|--- ArrayList
note top of Object : In java, every class\nextends this one.
note "This is a floating note" as N1
note "This note is connected\nto several objects." as N2

' 备注 N2 插入到两个对象之间
Object .. N2
N2 .. ArrayList

class Foo
note left: On last defined class

@enduml
```

![[image-20220717001933537.png]]



可以对属性和方法添加注释

```uml
@startuml

class A {
    {static} int counter
    +void {abstract} start(int timeout)
}

note right of A::counter
This member is annotated
end note

note right of A::start
This method is now explained in a UML note
end note

@enduml
```

![[image-20220717002037895.png]]

对同名方法添加注释

```uml
@startuml

class A {
    {static} int counter
    +void {abstract} start(int timeoutms)
    +void {abstract} start(Duration timeout)
}

note left of A::counter
This member is annotated
end note

note right of A::"start(int timeoutms)"
This method with int
end note

note right of A::"start(Duration timeout)"
This method with Duration
end note

@enduml
```

![[image-20220717002124299.png]]



我们也可以对连接词进行注释

```uml
@startuml

class Dummy
Dummy --> Foo : A link

note on link #red: note that is red

Dummy --> Foo2 : Another link

note right on link #blue
this is my note on right link
and in blue
end note

@enduml
```

![[image-20220717002324301.png]]



### 抽象类和接口

用 abstract 或 abstract class 定义抽象类，也可以使用 interface, annotation, enum 关键字

```uml
@startuml

abstract class AbstractList
abstract AbstractCollection
interface List
interface Collection

List <|-- AbstractList
Collection <|-- AbstractCollection
Collection <|- List
AbstractCollection <|- AbstractList
AbstractList <|-- ArrayList

class ArrayList {
    Object[] elementData
    size()
}

enum TimeUnit {
    DAYS
    HOURS
    MINUTES
}

annotation SuppressWarnings

@enduml
```

![[image-20220717002444663.png]]



### 使用非字母字符

如果想在类中使用非字母符号，可以：

* 在定义中使用 as 关键字
* 在类名旁边加上 ""

```uml
@startuml

class "This is my class" as class1
class class2 as "It works this way too"
class2 *-- "foo/dummy" : use

@enduml
```

![[image-20220717002613322.png]]



### 泛型

使用 `< >` 定义类的泛型

```uml
@startuml

class Foo<? extends Element> {
    int size()
}

Foo *- Element

@enduml
```

![[image-20220717002708978.png]]



### 指定标记

通常 C, I, E, A 用于标记类 class，接口 interface，枚举 enum 和抽象类 abstract classes ，但是也可以自己增加对应标记

```uml
@startuml

class System << (S,#FF7700) Singleton >>
class Date << (D,orchid) >>

@enduml
```

![[image-20220717002914691.png]]



### 包

可以用 package 声明包，用于分割不同的种类的类，同时可以声明对应的背景色；包可以嵌套定义

```uml
@startuml

package "Classic Collections" #DDDDDD {
    Object <|-- ArrayList
}

package net.sourceforge.plantuml {
    Object <|-- Demo1
    Demo1 *- Demo2
}

@enduml
```

![[image-20220717003015814.png]]



可以通过如下参数调整改变包的样式：

```uml
@startuml

scale 750 width
package foo1 <<Node>> {
class Class1
}

package foo2 <<Rectangle>> {
class Class2
}

package foo3 <<Folder>> {
class Class3
}

package foo4 <<Frame>> {
class Class4
}

package foo5 <<Cloud>> {
class Class5
}

package foo6 <<Database>> {
class Class6
}

@enduml
```

![[image-20220717003133005.png]]



### 命名空间

使用包时，类名是类的唯一标识，因此不同包中的类不能使用相同的类名。使用命名空间定义类，然后用 . 来引用不同命名空间中的类

```uml
@startuml

class BaseClass

namespace net.dummy #DDDDDD {
	' 使用 . 表示访问全局命名空间中的类
    .BaseClass <|-- Person
    Meeting o-- Person
    .BaseClass <|- Meeting
}

namespace net.foo {
	' 使用 nte.dummy 前缀加 . 访问前一个命名空间中的类
    net.dummy.Person <|- Person
    .BaseClass <|-- Person
    net.dummy.Meeting o-- Person
}
BaseClass <|-- net.unused.Person

@enduml
```

![[image-20220717003354758.png]]

使用 set namespaceSeperator 自定义命名空间的分隔符

```uml
@startuml

' 使用 :: 分隔命名空间
set namespaceSeparator ::
class X1::X2::foo {
    some info
}

@enduml
```

![[image-20220717003620333.png]]



### 箭头方向

默认 `--` 显示竖直线，可以用 `-` 或 `.` 得到水平连接的线；倒置连接可以改变方向，也可以用 left, right, up, down 改变方向

```uml
@startuml

Room o- Student
Room *-- Chair

Student -o Room
Chair --* Room

foo -left-> dummyLeft
foo -right-> dummyRight
foo -up-> dummyUp
foo -down-> dummyDown

@enduml
```

![[image-20220717003813939.png]]



### 关系类

可以在定义了两个类之间的关系后，再定义一个关系类 association class

```uml
@startuml

class Student {
    Name
}

' 连接 Student 和 Course
Student "0..*" - "1..*" Course
' 中间连接 Enrollment
(Student, Course) .. Enrollment

class Enrollment {
    drop()
    cancel()
}

@enduml
```

![[image-20220717003923818.png]]

可以将不同的类连接到同一个类

```uml
@startuml

class Station {
    +name: string
}

class StationCrossing {
    +cost: TimeInterval
}

<> diamond
StationCrossing . diamond
diamond - "from 0..*" Station
diamond - "to 0..* " Station

@enduml
```

![[image-20220717233941258.png]]



## 活动图

活动图用于描述事件的整个流程，本质上就是流程图。当前活动图 (activity diagram) 的语法有诸多限制和缺点，比如代码难以维护。所以从 V7947 开始提出一种全新的、更好的语法格式和软件实现供用户使用 (beta 版)。就像序列图一样，新的软件实现的另一个优点是它不再依赖于 Graphviz。



### 简单活动图

活动标签**以冒号开始，以分号结束**

```uml
@startuml test

:Hello world;
:This is on defined on
several **lines**;

@enduml
```

![[image-20220717094419709.png]]



### 开始/结束

使用 start 和 stop/end 表示开始和结束

```uml
@startuml

start

:Hello world;
:This is on defined on
several **lines**;
stop

@enduml
```

![[image-20220717094500461.png]]



### 条件语句

使用 if, then, else 设置分支，标注文字放在括号内

```uml
@startuml

start

if (Graphviz installed?) then (yes)
    :process all\ndiagrams;
else (no)
    :process only
    __sequence__ and __activity__ diagrams;
endif
stop

@enduml
```

![[image-20220717094611761.png]]

使用 elseif 进行多个分支测试

```uml
@startuml

start

if (condition A) then (yes)
    :Text 1;
elseif (condition B) then (yes)
    :Text 2;
    stop
elseif (condition C) then (yes)
    :Text 3;
elseif (condition D) then (yes)
    :Text 4;
else (nothing)
    :Text else;
endif

stop

@enduml
```

![[image-20220717094740206.png]]



如果要在某个条件终止，如上在分支中使用 stop ；如果想表示在此步结束，使用 kill 或 detach

```uml
@startuml

' 在此步条件结束
if (condition?) then
    #pink:error;
    kill
endif

#palegreen:action;

@enduml
```

![[image-20220717094949328.png]]



### 循环语句

使用 repeat 和 repeatwhile 重复循环，backward 设置回归描述

```uml
@startuml

start

repeat :foo as starting label;
    :read data;
    :generate diagrams;
backward :This is backward;
repeat while (more data?)

stop

@enduml
```

![[image-20220717095312788.png]]

要在某一步循环退出使用 break

```uml
@startuml

start

repeat
:Test something;
    if (Something went wrong?) then (no)
        #palegreen:OK;
        break
    endif

    ->NOK;
    :Alert "Error with long text";
repeat while (Something went wrong with long text?) is (yes) not (no)

->//merged step//;
:Alert "Sucess";

stop

@enduml
```

![[image-20220717095638482.png]]

还可以用 while 和 end while 实现 while 循环，用 is 添加标注

```uml
@startuml

while (check filesize ?) is (not empty)
    :read file;
endwhile (empty)
:close file;

@enduml
```

![[image-20220717095806052.png]]



### 并行处理

使用 fork, fork again 和 end fork 表示并行处理

```uml
@startuml

start

if (multiprocessor?) then (yes)
    fork
        :Treatment 1;
    fork again
        :Treatment 2;
    end fork
else (monoproc)
    :Treatment 1;
    :Treatment 2;
endif

@enduml
```

![[image-20220717095953974.png]]



### 划分进程

使用 split, split again 和 split keywords 划分进程

```uml
@startuml

start

split
    :A;
split again
    :B;
split again
    :C;
split again
    :a;
    :b;
end split

:D;

end

@enduml
```

![[image-20220717100157566.png]]



### 注释

使用 note 语法添加注释，floating 关键字可以使 note 浮动

```uml
@startuml

start

:foo1;
floating note left: This is a note
:foo2;

note right
    This note is on several
    //lines// and can
    contain <b>HTML</b>
    ====
    * Calling the method ""foo()"" is prohibited
end note

stop
```

![[image-20220717100341052.png]]



### 箭头

使用 -> 标记箭头，可以添加文字或修改颜色，同时也可以调整点状 dotted，条状 dashed，加粗 bold，隐式 hidden

```uml
@startuml

:foo1;
-[hidden]-> You can put text on arrows;

if (test) then
    -[#blue]->
    :foo2;
    -[#green,dashed]-> The text can
    also be on several lines
    and **very** long...;
    :foo3;
else
    -[#black,dotted]->
    :foo4;
endif

-[#gray,bold]->
:foo5;

@enduml
```

![[image-20220717100648592.png]]



### 连接器

使用括号定义连接器

```uml
@startuml

start

:Some activity;
#Blue:(A)
detach

#Red:(A)
:Other activity;

@enduml
```

![[image-20220717100836970.png]]



### 组合

使用 partition 定义分区，将多个活动组合在一起

```uml
@startuml

start

partition Initialization {
    :read config file;
    :init internal variable;
}

partition Running {
    :wait for user interaction;
    :prin
    t information;
}
stop

@enduml
```

![[image-20220717100940277.png]]



### 泳道

使用管道 | 定义泳道，还可以改变颜色

```uml
@startuml

|Swimlane1|
    start
    :foo1;
|#AntiqueWhite|Swimlane2|
    :foo2;
    :foo3;
|Swimlane1|
    :foo4;
|Swimlane2|
    :foo5;
    stop

@enduml
```

![[image-20220717101056826.png]]





### 活动形状

修改活动标签的分号分隔符 `;` 设置不同形状

```uml
@startuml

:Ready;
:next(o)|
:Receiving;

split
    :nak(i)<
    :ack(o)>
split again
    :ack(i)<
    :next(o)
    on several line|
    :i := i + 1]
    :ack(o)>
split again
    :err(i)<
    :nak(o)>
split again
    :foo/
split again
    :i > 5}
    stop
end split

:finish;

@enduml
```

![[image-20220717101410907.png]]



## 状态图

### 简单状态

使用 `[*]` 开始和结束状态图，使用 `-->` 添加箭头，使用 hide empty description 隐藏空的状态

```uml
@startuml

hide empty description
[*] --> State1
State1 --> [*]
State1 : this is a string
State1 : this is another string
State1 -> State2
State2 --> [*]

@enduml
```

![[image-20220717101843684.png]]



### 合成状态

使用 state 和花括号定义合成状态

```uml
@startuml

scale 350 width
[*] --> NotShooting

state NotShooting {
    [*] --> Idle
    Idle --> Configuring : EvConfig
    Configuring --> Idle : EvConfig
}

state Configuring {
    [*] --> NewValueSelection
    NewValueSelection --> NewValuePreview : EvNewValue
    NewValuePreview --> NewValueSelection : EvNewValueRejected
    NewValuePreview --> NewValueSelection : EvNewValueSaved
    state NewValuePreview {
        State1 -> State2
    } 
}

@enduml
```

![[image-20220717101950941.png]]



### 长名字

也可以使用 state 定义长名字状态

```uml
@startuml

scale 600 width
[*] -> State1
State1 --> State2 : Succeeded
State1 --> [*] : Aborted
State2 --> State3 : Succeeded
State2 --> [*] : Aborted

state State3 {
    state "Accumulate Enough Data\nLong State Name" as long1
    long1 : Just a test
    [*] --> long1
    long1 --> long1 : New Data
    long1 --> ProcessData : Enough Data
}

State3 --> State3 : Failed
State3 --> [*] : Succeeded / Save Result
State3 --> [*] : Aborted

@enduml
```

![[image-20220717103036853.png]]



### 分支和合并

使用 fork 和 join 实现分支和合并

```uml
@startuml

state fork_state <<fork>>

[*] --> fork_state
fork_state --> State2
fork_state --> State3

state join_state <<join>>

State2 --> join_state
State3 --> join_state
join_state --> State4
State4 --> [*]

@enduml
```

![[image-20220717103218219.png]]



### 并发状态

用 `--` 或 `||` 作为分隔符来合成并发状态

```uml
@startuml

[*] --> Active

state Active {
    [*] -> NumLockOff
    NumLockOff --> NumLockOn : EvNumLockPressed
    NumLockOn --> NumLockOff : EvNumLockPressed

    --

    [*] -> CapsLockOff
    CapsLockOff --> CapsLockOn : EvCapsLockPressed
    CapsLockOn --> CapsLockOff : EvCapsLockPressed

    --
    
    [*] -> ScrollLockOff
    ScrollLockOff --> ScrollLockOn : EvCapsLockPressed
    ScrollLockOn --> ScrollLockOff : EvCapsLockPressed
}

@enduml
```

![[image-20220717103531336.png]]



### 条件选择

使用 `<<choice>>` 说明条件选择

```uml
@startuml

' 定义别名
state "Req(Id)" as ReqId <<sdlreceive>>
state "Minor(Id)" as MinorId
state "Major(Id)" as MajorId

' 定义 c 为选择节点
state c <<choice>>

Idle --> ReqId
ReqId --> c
c --> MinorId : [Id <= 10]
c --> MajorId : [Id > 10]

@enduml
```

![[image-20220717103821988.png]]



### 模块化全部案例

使用 choice, fork, join, end 的完全案例

```uml
@startuml

state choice1 <<choice>>
state fork1 <<fork>>
state join2 <<join>>
state end3 <<end>>

[*] --> choice1 : from start\nto choice
choice1 --> fork1 : from choice\nto fork
choice1 --> join2 : from choice\nto join
choice1 --> end3 : from choice\nto end
fork1 ---> State1 : from fork\nto state
fork1 --> State2 : from fork\nto state
State2 --> join2 : from state\nto join
State1 --> [*] : from state\nto end
join2 --> [*] : from join\nto end

@enduml
```

![[image-20220717104110314.png]]



### 箭头方向

使用 `->` 定义水平箭头，也可以使用 left, right, up, down 改变方向

```uml
@startuml

[*] -up-> First
First -right-> Second
Second --> Third
Third -left-> Last

@enduml
```

![[image-20220717104321774.png]]



### 注释

使用 note 定义注释以及浮动注释

```uml
@startuml

[*] --> Active
Active --> Inactive

note left of Active : this is a short\nnote
note right of Inactive
    A note can also
    be defined on
    several lines
end note

@enduml
```

![[image-20220717105941324.png]]



使用 note on link 在连接线上添加注释

```uml
@startuml

[*] -> State1
State1 --> State2

note on link
    this is a state-transition note
end note

@enduml
```

![[image-20220717110053781.png]]

可以在合成状态中放置注释

```uml
@startuml

[*] --> NotShooting

state "Not Shooting State" as NotShooting {
    state "Idle mode" as Idle
    state "Configuring mode" as Configuring
    [*] --> Idle
    Idle --> Configuring : EvConfig
    Configuring --> Idle : EvConfig
}

note right of NotShooting : This is a note on a composite state

@enduml
```

![[image-20220717110145778.png]]

