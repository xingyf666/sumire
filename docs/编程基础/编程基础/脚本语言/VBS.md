# VBS

## 基本介绍

VBS，全称为 Visual Basic Scripting Edition，是一种基于 Microsoft Visual Basic 的脚本语言，用于在 Windows 操作系统下编写脚本以执行各种任务。它继承了 Visual Basic 的一些语法和特性，如变量声明、条件语句、循环结构等。



### 消息框

由于 VBS 没有控制台，因此需要直接通过可视化输出来进行调试。使用消息框实现

```vbscript
msgbox "content",,"title"
```

可以弹出一个输入框

```vbscript
inputbox("content")
```

弹出输入框，显示内容，它会返回输入值。



### 注释

使用单引号表示注释

```vbscript
'注释内容
```

注意行末不能有多余的空格！！！



### 操作符

算术运算：+、-、*、/、^（乘方）、mod（取余）

比较运算：<、>、<=、>=、=（等于）、<>（不等于）

逻辑运算：and、not、or

连接字符串：+、&

比较是否相同：is，用于比较对象是否是相同类型。



## 变量

VBS 中变量不区分大小写。



### 声明变量

有三种声明变量的方法

* 显式声明：Dim、Public、Private 语句进行声明；
    * dim：用于声明脚本、过程、类级作用域的变量
    * public：用于声明脚本、类级作用域使用Private语句
    * private：用于声明脚本、类级作用域
* 隐式声明：不声明直接使用；
* 强制声明：Option Explicit 语句强制显式声明所有变量；

 例如使用 dim 声明多个同类型变量，用逗号分隔

```vbscript
dim var1, var2 ...
```

不能在声明的同时进行赋值，但允许同时对两个变量赋值，用冒号分隔。例如

```vbscript
dim a, b
a = 1 : b = 2
```

使用 const 修饰符声明常量

```vbscript
const a = 5
```

注意常量必须在声明时立即赋值。



### 类型转换

VBScript 在定义时只有一种变量类型，在实际使用中需要**使用类型转换函数来将变量转换成相应的变量类型**：

| 函数             | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| Cbool            | 将变量转换成布尔值                                           |
| Cbyte            | 将变量转换为 0 到 255 之间的整数                             |
| Ccur, Cdbl, Csng | 将变量转换为浮点数值，前者只精确到小数点后四位，后两者要更加精确，数值的范围也要大的多 |
| Cdate            | 将变量转换为日期值                                           |
| Cint, Clng       | 将变量转换为整数，后者的范围比前者要大的多                   |
| Cstr             | 将变量转换为字符串                                           |



### 数组

数组声明格式为

```vbscript
dim arr(5)
```

上述代码声明**长度为 6 的数组**。通过 `()` 访问数组名，下标范围为 0-5 共 6 个元素。



可以通过 array 指定数组赋值

```vbscript
dim a
a = array(1, 2, 3)
```

这里指定了数组的 3 个元素为 1,2,3 。



可以定义二维数组

```vbscript
dim a(1, 2)
a(0, 0) = 15
```



### 重定义

如果需要定义动态数组，可以不预先指定数组长度

```vbscript
dim arr()
redim arr(n)
```



## 基本语法

### 条件语句

if 格式：

```vbscript
if condition1 then
    code
elseif condition2 then
	code
else
	code
end if
```



select case 格式：

```vbscript
select case variable
case condition1
    code
case condition2
	code
case else
	code
end select
```

 

### 循环语句

for next 循环：

```vbscript
for i＝start to end
	code
next
```



for each 循环：

```vbscript
dim arr(2)
arr(0) = 5
arr(1) = 10

for each i in arr
    code
    ' 使用 exit for 来退出循环
next
```



do while 结构：

```vbscript
do while condition
    code
    ' 使用 exit do 来退出循环
loop
```

do 开头的循环格式会先执行循环体，然后判断循环条件。



do until 结构：

```vbscript
do until condition
    code
    ' 使用 exit do 来退出循环
loop
```



条件循环（满足条件一直循环）：

```vbscript
while condition
    code
    ' 使用 Exit While 来退出循环
wend
```



## 函数

### 定义函数

函数的基本定义方法为：

```vbscript
function func(args)
    code
	
    ' 右侧为返回值
    func = value
end function
```

例如定义一个返回 1 的函数

```vbscript
function A(x)
    msgbox "x"
    A = 1
end function
```

函数只能出现在赋值语句的右边、或者表达式中，函数**不能直接使用**，如果必须直接使用函数，则必须使用 call 语句调用，并取消返回值

 ```vbscript
 call A(x)
 ```



### 子程序

没有返回值的函数称为子程序，子程序不能在表达式中使用。尽管在定义子程序的时候，参数列表要加括号，但在调用子程序的时候，参数列表不加括号。例如

```vbscript
function func()
    msgbox "hello"
end function

func
```



### 内置函数

VBS 库中预定义了大量有用的内置函数，需要使用时查阅手册即可。

| 函数  | 作用               | 函数  | 作用               |
| ----- | ------------------ | ----- | ------------------ |
| lcase | 将字符串转换成小写 | ucase | 将字符串转换成大写 |



##  面向对象

WSH 是用来解析 VBS 的宿主，本身包含了几个常用对象：

1. scripting.filesystemobject 提供一整套文件系统操作函数
2. scripting.dictionary 用来返回存放键值对的字典对象
3. wscript.shell 提供一套读取系统信息的函数，如读写注册表、查找指定文件的路径、读取 DOS 环境变量，读取链接中的设置
4. wscript.network 提供网络连接和远程打印机管理的函数

通过 wscript 还可以进行输出

```vbscript
Dim i
i = 0

' Do While 循环
Do While i < 5
    WScript.Echo i
    i = i + 1
Loop
```



### 打开记事本

可以通过对象实现启动程序

```vbscript
dim objshell
set objshell = createobject("wscript.shell")

objshell.run "notepad"
set objshell = nothing
' 设置对象变量为空
```

其中 createobject() 用于创建对象，set 将一个对象引用（除字符串、数值、布尔值之外的变量）赋给变量。



### 调用成员

正确引用的对象本身内置有函数和变量，通过 `.` 来调用函数。例如

```vbscript
objshell.run "file"
```

run 在运行解析时，遇到空格会停止。因此需要使用**三个双引号**输入路径

```vbscript
dim wsh
set wsh = wscript.createobject("wscript.shell")
wsh.run """D:\Program Files (x86)\360\360Chrome\Chrome\Application\360chrome.exe"""
```

run 有三个参数

1. 程序路径
2. 窗口形式：
    * 0 后台运行；
    * 1 正常运行；
    * 2 激活程序并最小化；
    * 3 激活程序并最大化；

3. 脚本等待或继续执行。如果为 true 就等待当前程序退出再继续执行

并且 run 有返回值，0 表示成功执行，不为 0 则为错误代码

```vbscript
a = objshell.run path
```



### 异常处理

```vbscript
On Error Resume Next
```

这行语句告诉 vbs 在运行时跳过发生错误的语句，执行跟在它后面的语句。发生错误时，该语句将会把相关的错误号、错误描述和相关源代码压入错误堆栈。

 

使用 err 对象处理错误。它具有两个方法：clear, raise 以及五个属性 description, helpcontext, helpfile, number, source 。它不用引用实例，可以直接使用

```vbscript
' 跳过错误语句
on error resume next

' 除以 0 错误
a = 11
b = 0
c = a / b

' 如果错误码不为 0，就汇报错误
if err.number <> 0 then
	wscript.echo err.number & err.description & err.source
end if
```

 

### 修改注册表

#### 读注册表的关键词和值

可以通过把关键词的完整路径传递给 wshshell 对象的 regread 方法获取

```vbscript
set ws = wscript.createobject("wscript.shell")
v = ws.regread("HKLM\Software\7-Zip\Path")
wscript.echo v
```



#### 写注册表

使用 wshshell 对象的 regwrite 方法

```vbscript
path = "..."
set ws = wscript.createobject("wscript.shell")
t = ws.regwrite(path & "jj","hello")
```

上述代码将 `path\jj` 这个键值改成了`hello`，注意这个键值一定要预先存在。对于不存在的键值，也可以创建

```vbscript
path = "..."
set ws = wscript.createobject("wscript.shell")

val = ws.regwrite(path, "nenboy")
val = ws.regread(path)

wscript.echo val
```



#### 删除关键字和值

使用 regdelete 方法

```vbscript
val = ws.regdelete(path)
```

如果要删除关键词的值的话，一定要在路径最后加上 `\`，否则会删除整个关键词。



### 模拟按键

这个命令的作用就是模拟键盘操作，将一个或多个按键指令发送到指定 Windows 窗口来控制应用程序运行。

 

首先创建一个对象，然后通过 SendKeys 发送字符串，表示按键

```vbscript
Set WshShell = WScript.CreateObject("WScript.Shell") 
WshShell.SendKeys "x"
```

可发送的字符串包含

```vbscript
' 基本键：直接用该按键字符本身来表示
WshShell.SendKeys "xyz"		

' 特殊按键：
' Shift 对应 + 
' Ctrl 对应 ^ 
' Alt 对应 % 
' 使用 () 表示同时发送
WshShell.SendKeys "^e"  	' 发送 Ctrl + E
WshShell.SendKeys "^(ec)" 	' 发送 Ctrl + (E + C)
WshShell.SendKeys "^ec" 	' 发送 Ctrl + E + C
' 其余特殊按键用 {} 包含即可，例如 {DOWN}, {UP}, {LEFT}, {ENTER}, {SPACE}

' 重复发送
WshShell.SendKeys "{x 10}" 	' 发送 10 个 x
```



## 文件系统

FileSystemObject(FSO) 表示文件系统对象。它包含的常见对象有：

| 对象       | 作用                                                   |
| ---------- | ------------------------------------------------------ |
| Drive      | 包含储存设备的信息，包括硬盘、光驱、ram 盘、网络驱动器 |
| Drives     | 提供一个物理和逻辑驱动器的列表                         |
| File       | 检查和处理文件                                         |
| Files      | 提供一个文件夹中的文件列表                             |
| Folder     | 检查和处理文件夹                                       |
| Folders    | 提供文件夹中子文件夹的列表                             |
| Textstream | 读写文本文件                                           |

常见方法有：

| 方法                | 作用                                                         | 方法             | 作用                                         |
| ------------------- | ------------------------------------------------------------ | ---------------- | -------------------------------------------- |
| BulidPath           | 把文件路径信息添加到现有的文件路径上                         | CopyFile         | 复制文件                                     |
| CopyFolder          | 复制文件夹                                                   | CreateFolder     | 创建文件夹                                   |
| CreateTextFile      | 创建文本并返回一个 TextStream 对象                           | DeleteFile       | 删除文件                                     |
| DeleteFolder        | 删除文件夹及其中所有内容                                     | DriveExits       | 确定驱动器是否存在                           |
| FileExits           | 确定一个文件是否存在                                         | FolderExists     | 确定某文件夹是否存在                         |
| GetAbsolutePathName | 返回一个文件夹或文件的绝对路径                               | GetBaseName      | 返回一个文件或文件夹的基本路径               |
| GetDrive            | 返回一个 drive 对象                                          | GetDriveName     | 返回一个驱动器的名字                         |
| GetExtensionName    | 返回扩展名                                                   | GetFile          | 返回一个 file 对象                           |
| GetFileName         | 返回文件夹中文件名称                                         | GetFolder        | 返回一个文件夹对象                           |
| GetParentFolderName | 返回一个文件夹的父文件夹                                     | GetSpecialFolder | 返回指向一个特殊文件夹的对象指针             |
| GetTempName         | 返回一个可以被 createtextfile 使用的随机产生的文件或文件夹的名称 | MoveFile         | 移动文件                                     |
| MoveFolder          | 移动文件夹                                                   | OpenTextFile     | 打开一个存在的文件并返回一个 TextStream 对象 |



### fso 对象

fso 不是 wsh 的一部分，需要建立对象

```vbscript
set fs = wscript.createobject("scripting.filesystemobject")

...code

set fs = nothing
```

最后释放对象。

 

### 文件夹操作

#### 检查文件夹

在创建前，需要检查该文件夹是否存在

```vbscript
' 初始化
dim fs, s
set fs = wscript.createobject("scripting.filesystemobject") 

' 判断是否存在
if (fs.folderexists("c:\Users\lenovo\Desktop\new")) then
	' 存在就删除文件夹
    s = "is available"
	fs.deletefolder("c:\Users\lenovo\Desktop\new")
else
    ' 不存在就创建
    s= "not exist"
    set folder = fs.createfolder("c:\Users\lenovo\Desktop\new")
end if

set fs = nothing
```

 

#### 复制移动

复制函数是子进程，不能通过 `()` 调用，而要用 `,` 分隔参数

```vbscript
dim fs
set fs = wscript.createobject("scripting.filesystemobject") 

' 复制文件夹，注意目标位置 data 应该不存在，否则会出错
fs.copyfolder "c:\data","d:\data"

' 使用强制覆盖，增加 true 参数（注意目标目录不能是源目录的子目录）
fs.copyfolder "e:\Desktop\Cuda","e:\Desktop\Duda",true

set fs = nothing
```



移动操作类似，还可以使用通配符

```vbscript
dim fs
set fs = wscript.createobject("scripting.filesystemobject") 

fs.movefolder "c:\data","d:\data"
fs.movefolder "c:\data\te*","d:\working"

set fs = nothing
```

在目标路径最后没有添加 `\`，这样如果 `d:\working` 目录不存在，windows 就不会为我们自动创建这个目录。

 

#### 文件夹对象

可以使用 folder 文件夹对象

```vbscript
dim fs, f
set fs = wscript.createobject("scripting.filesystemobject")

set f = fs.getfolder("c:\data")

' 执行删除、复制和移动操作
f.delete
f.copy "d:\working", true
f.move "d:\temp"
```



#### 特殊文件夹

访问系统文件夹

```vbscript
set fs = wscript.createobject("scripting.filesystemobject")
set wshshell = wscript.createobject("wscript.shell")

' 获得环境变量
set osdir = wshshell.expandenvironmentstrings("%systemroot%")

set f = fs.getfolder(osdir)

wscript.echo f
```



还可以使用 getspecialfolder() 获得文件夹

```vbscript
set fs = wscript.createobject("scripting.filesystemobject")

set wfolder = fs.getspecialfolder(0)  ' 返回 windows 目录
set wfolder = fs.getspecialfolder(1)  ' 返回 system32\
set wfolder = fs.getspecialfolder(2)  ' 返回临时目录
```



### 文件操作

#### 文件属性

在 windows 中，文件的属性一般用数字来表示：

| 数字 | 作用                             | 数字 | 作用     |
| ---- | -------------------------------- | ---- | -------- |
| 0    | normal，即普通文件未设置任何属性 | 1    | 只读文件 |
| 2    | 隐藏文件                         | 4    | 系统文件 |
| 16   | 文件夹或目录                     | 32   | 存档文件 |
| 1024 | 链接或快捷方式                   |      |          |



通过 Attributes 方法获得文件属性

```vbscript
set fs = wscript.createobject("scripting.filesystemobject")
set f = fs.getfile("d:\index.txt")
msgbox f.Attributes
```

注意 msgbox 显示的结果往往不是上面说明的数字，而是有关属性代表数字的和。



#### 创建文件

创建前一般需要检查文件是否存在

```vbscript
set fso = wscript.createobject("scripting.filesystemobject")

if fso.fileexists("c:\kk.txt") then
	msgbox "文件已存在"
else
    set f = fso.createtextfile("c:\kk.txt")
end if
```

如需要强制覆盖已存在的文件，则在文件名后加 true 参数。



#### 复制、移动、删除

```vbscript
set fso = wscript.createobject("scripting.filesystemobject")

fso.copyfile "c:\kk.txt","d:\1\kk.txt",true
fso.movefile "c:\kk.txt","d:\"
fso.deletefile "c:\kk.txt"
```



#### 文件读写

首先要打开文件

```vbscript
set ts = fso.opentextfile("c:\kk.txt",1,true)
```

其中第二个参数是访问模式，1 为只读、2 为写入、8 为追加。第三个参数指定如文件不存在则创建。



有多种读取方式

```vbscript
value = ffile.read(20)		' 读 20 个字符
line = ffile.readline		' 读一行
contents = ffile.readall	' 全部读取
```



使用 skip(x) 跳过 x 个字符；skipline 跳过一行

```vbscript
ffile.skip(10)
ffile.skipline
```



写入方法包括

```vbscript
ffile.write("hello")		' 写入 hello
ffile.writeline("hi")		' 写入 hi 然后换行
ffile.writeblanklines(2)	' 写入 2 个空行
```



读写完成后要使用 close 方法关闭文件

```vbscript
ffile.close
```



#### 指针变量

通过 atendofstream 属性循环检测是否到达文件末尾。例如：

```vbscript
do while ffile.atendofstream <> true
	ffile.read(10)
loop
```

通过 atendofline 属性检测是否到了行末尾

```vbscript
do while ffile.atendofline <> true
	ffile.read(10)
loop
```

还有 Column 属性获得当前字符位置的列号，line 属性获得文件当前行号。



## 实践样例

### 记事本输入

打开记事本输入 Hello World 保存的脚本

```vbscript
Dim WshShell
Set WshShell = WScript.CreateObject("WScript.Shell")

WshShell.Run "notepad"

' 脚本暂停 2 秒，给 notepad 一个打开的时间，时间太短可能导致后面的字符无法进入编辑区
WScript.Sleep 2000

' 获得焦点
WshShell.AppActivate "无标题 - 记事本" 

' 输入文本
WshShell.SendKeys "Hello World"

' Alt + F4 退出记事本
WshShell.SendKeys “%{F4}” 

WshShell.SendKeys “{RIGHT}{ENTER}”
```

Fn 是由硬件驱动直接控制的，不能通过键盘模拟。

 

### 快速登陆 QQ

假设 QQ 号码是：10001，密码是：123456，隐身登陆：

```vbscript
set ws = wscript.createobject("wscript.shell")

ws.run "C:\Progra~1\Tencent\QQ\QQ.exe",0

' 等待开启
wscript.Sleep 2000

' 获得焦点
ws.AppActivate "QQ用户登录"

ws.SendKeys "10001"
ws.SendKeys "{TAB}"
ws.SendKeys "123456"
ws.SendKeys "{ENTER}"
```



### 文本自动保存

创建一个文本文件，每隔一段时间保存

```vbscript
dim wsh, autosave, filename

' 保存间隔
autosave = 5000

set wsh = wscript.createobject("wscript.shell")
filename = inputbox("输入需要编辑的文本文件名：")

' 打开记事本
wsh.run "notepad"
wscript.sleep 200

' 注意这里激活的窗口名要与实际创建的新记事本的标题相同
' 在 win10 中是 “无标题 - 记事本”
' 在 win11 中是 “无标题”
wsh.appactivate "无标题"

' 按 Ctrl + s, Shift 切换输入法
wsh.sendkeys "^s"
wscript.sleep 200
wsh.sendkeys "+"
wscript.sleep 200

' 输入文件名，回车并保存
wsh.sendkeys filename
wscript.sleep 200
Wsh.SendKeys "{ENTER}"
WScript.Sleep autosave

' 注意这里加 * 是因为修改内容后文件标题会改变
' 另外获取焦点的顺序不能改，否则焦点会丢失
while (wsh.appactivate (filename) or wsh.appactivate ("*"+filename))=true  
	wsh.sendkeys "^s"         
	wscript.sleep autosave
wend

' 退出脚本
wscript.quit
```

