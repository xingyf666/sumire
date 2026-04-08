# QMake

## 基本介绍

QMake 用于创建 Qt 工程，用于管理应用程序、库和其他组件的构建过程。qmake 将每个项目文件中的信息扩展为 Makefile，该 Makefile 执行编译和链接所需的命令。



### 资源变量

与 cmake 命令不同，qmake 直接使用变量来保存资源信息，它处理 .pro 文件。项目文件通过 SOURCES 变量保存

```properties
SOURCES += \
    main.cpp \
    mainwindow.cpp
```

头文件通过 HEADERS 变量保存

```properties
HEADERS += \
    mainwindow.h
```

生成目标自动配置与项目文件名称相同

```properties
TARGET = helloworld
```

如果想要产生 debug 版本，直接添加

```properties
CONFIG += debug
```



### 平台资源

如果想要为特定的平台引入不同的资源，使用

```properties
win32 {
    SOURCES += hellowin.cpp
}
unix {
    SOURCES += hellounix.cpp
}
```

这是条件格式

```properties
condition { ... }
```

如果条件为真，则进入内部。



### 停止条件

可以让 qmake 在某个文件不存在时立即停止

```properties
!exists( main.cpp ) {
    error( "No main.cpp file found" )
}
```

使用 exists() 函数判断文件是否存在，如果前面条件为真，则会调用 error 输出错误信息。



还可以嵌套使用条件判断，例如要求 win32 平台和 debug 选项

```properties
win32 {
    debug {
        CONFIG += console
    }
}
```

可以合并条件判断

```properties
win32:debug {
    CONFIG += console
}
```



### 常用变量

下面是常用变量的列表

|                             变量                             |              内容              |
| :----------------------------------------------------------: | :----------------------------: |
| [CONFIG](https://doc.qt.io/qt-5/qmake-variable-reference.html#config) |            生成选项            |
| [DESTDIR](https://doc.qt.io/qt-5/qmake-variable-reference.html#destdir) |            目标路径            |
| [FORMS](https://doc.qt.io/qt-5/qmake-variable-reference.html#forms) |          UI 文件 .ui           |
| [HEADERS](https://doc.qt.io/qt-5/qmake-variable-reference.html#headers) |           头文件 .h            |
| [QT](https://doc.qt.io/qt-5/qmake-variable-reference.html#qt) |            Qt 模块             |
| [RESOURCES](https://doc.qt.io/qt-5/qmake-variable-reference.html#resources) |         资源文件 .qrc          |
| [SOURCES](https://doc.qt.io/qt-5/qmake-variable-reference.html#sources) |           源码 .cpp            |
| [TEMPLATE](https://doc.qt.io/qt-5/qmake-variable-reference.html#template) | 项目的模板，决定输出程序还是库 |



要获取变量内容，使用 `$$` 前缀，例如获得 SOURCES 变量

```properties
TEMP_SOURCES = $$SOURCES
```



### 空格

默认情况下，空格用来分割值。例如

```properties
VAR = hello world
```

如果需要输入带有空格的值，则需要用双引号

```properties
VAR = "hello world"
```

这一点在输入带有空格的路径时尤为重要。



### 注释

以 `#` 开头的行被视为注释

```properties
# Comments usually start at the beginning of a line, but they
# can also follow other content on the same line.
```



### 项目模板

模板变量 TEMPLATE 用来定义要构建的项目类型。如果没有指定此变量，默认构建应用程序。

|     模板      |                             输出                             |
| :-----------: | :----------------------------------------------------------: |
| app (default) |                         构建应用程序                         |
|      lib      |                            构建库                            |
|      aux      | 不生成任何内容。如果不需要调用编译器来创建目标，请使用此选项 |
|    subdirs    | 生成 SUBDIRS 指定的子目录下的项目，每个子目录下分别有一个 .pro |
|     vcapp     |                生成 VS 项目文件来构建应用程序                |
|     vclib     |                   生成 VS 项目文件来构建库                   |
|   vcsubdirs   |              生成 VS 项目文件构建子目录下的项目              |



### 通用配置

配置变量 CONFIG 指定项目选项。注意使用 `+=` 防止覆盖默认的选项

```properties
CONFIG += qt debug
```

默认情况下启用 qt release app gui core ... 选项。



如果想构建非 qt 项目，可以使用

```properties
CONFIG -= qt
```

移除 qt 选项。



可以判断是否指定了某个选项，例如是否启用 OpenGL

```properties
CONFIG(opengl) {
    message(Building with OpenGL support.)
} else {
    message(OpenGL support is not available.)
}
```



### 声明第三方库

如果需要添加其它库到 qt 项目中，需要使用 LIBS 和 INCLUDEPATH 变量

```properties
LIBS += -L/usr/local/lib -lmath
INCLUDEPATH = c:/msdev/include d:/stl/include
```





## QMake 语法

这一部分我们开始系统介绍 QMake 的基本语法。



### 赋值操作

变量可以通过多种方式赋值，包括直接赋值、增量赋值、删除赋值、唯一赋值、替代赋值

```properties
DEFINES = test				# 直接赋值
DEFINES	+= hello			# 增量赋值
DEFINES -= hello			# 删除赋值
DEFINES *= hello			# 只有 DEFINES 不包含 hello 时，才会增加 hello
DEFINES ~= s/QT_[DT].+/QT	# 正则表达式，将 QT_D 和 QT_T 开头的值替换为 QT
```

其中唯一赋值的目的是防止同一个值被增加多次。



### 变量扩展

使用 `$$` 操作符获得变量的值，可用来传值或作为函数参数

```properties
EVERYTHING = $$SOURCES $$HEADERS
message("The project contains the following files:")
message($$EVERYTHING)
```



还可以在 qmake 运行时获得环境变量的值，使用 `$$(..)` 操作符

```properties
DESTDIR = $$(PWD)
message(The project will be installed in $$DESTDIR)
```

这里获取了 `PWD` 环境变量的值。



使用 `$(..)` 操作符获得 makefile 运行时的环境变量

```properties
DESTDIR = $$(PWD)
message(The project will be installed in $$DESTDIR)

DESTDIR = $(PWD)
message(The project will be installed in the value of PWD)
message(when the Makefile is processed.)
```

其中第一个 `PWD` 在 qmake 运行时立即读取，而第二个 `PWD` 则在 makefile 运行时读取。



使用 `$$[..]` 操作符访问 qmake 属性

```properties
message(Qt version: $$[QT_VERSION])
message(Qt is installed in $$[QT_INSTALL_PREFIX])
message(Qt resources can be found in the following locations:)
message(Documentation: $$[QT_INSTALL_DOCS])
message(Header files: $$[QT_INSTALL_HEADERS])
message(Libraries: $$[QT_INSTALL_LIBS])
message(Binary files (executables): $$[QT_INSTALL_BINS])
message(Plugins: $$[QT_INSTALL_PLUGINS])
message(Data files: $$[QT_INSTALL_DATA])
message(Translation files: $$[QT_INSTALL_TRANSLATIONS])
message(Settings: $$[QT_INSTALL_CONFIGURATION])
message(Examples: $$[QT_INSTALL_EXAMPLES])
```

使用这些属性可以使第三方插件和组件能够集成在Qt中。例如将 Qt Designer 插件与 Qt Designer 内置插件一起安装

```properties
target.path = $$[QT_INSTALL_PLUGINS]/designer
INSTALLS += target
```



变量值可以直接连接，例如

```properties
TARGET = myproject_$${TEMPLATE}
```



### Scopes

Scopes 类似于编程语言中的 `if` 条件，如果条件为真则会执行 Scope 中的部分。基本语法为

```properties
<condition> {
    <command or definition>
    ...
}
```

注意 `{}` 的位置必须按照上面的方式放置。



例如根据平台类型添加源文件

```properties
win32 {
    SOURCES += paintwidget_win.cpp
}

!win32 {
    SOURCES -= paintwidget_win.cpp
}
```



#### 嵌套

Scopes 可以相互嵌套，例如在 mac 平台下，配置 debug 模式时才引入头文件

```properties
macx {
    CONFIG(debug, debug|release) {
        HEADERS += debugging.h
    }
}
```

通过 `:` 连接条件来简化写法

```properties
macx:CONFIG(debug, debug|release) {
	HEADERS += debugging.h
}
```



甚至可以写成一行

```properties
win32:DEFINES += USE_MY_STUFF
```



使用 `|` 表示或，例如 win32 或 mac 平台下引入头文件

```properties
win32|macx {
    HEADERS += debugging.h
}
```



#### 混合

如果需要混合 `|,:` 操作符，应该使用 `if` 函数指定

```properties
if(win32|macos):CONFIG(debug, debug|release) {
    # Do something on Windows and macOS,
    # but only for the debug configuration.
}
win32|if(macos:CONFIG(debug, debug|release)) {
    # Do something on Windows (regardless of debug or release)
    # and on macOS (only for debug).
}
```



可以进行字符串匹配，例如匹配以 `win32-` 开头的值

```properties
win32-* {
    # Matches every mkspec starting with "win32-"
    SOURCES += win32_specific.cpp
}
```



#### 分支

支持条件分支

```properties
win32:xml {
    message(Building for Windows)
    SOURCES += xmlhandler_win.cpp
} else:xml {
    SOURCES += xmlhandler.cpp
} else {
    message("Unknown configuration")
}
```



#### 配置

CONFIG 变量存放的值会被 qmake 特别处理，其中的每个选项都可以直接用来进行条件判断。例如用 opengl 选项判断

```properties
CONFIG += opengl
opengl {
    TARGET = application-gl
} else {
    TARGET = application
}
```

根据是否配置 opengl，决定目标名称。



### 函数语法

qmake 提供了两种函数语法进行复杂操作，分别处理变量值和作为条件测试。



#### Replace

处理变量值的函数体结构为

```properties
defineReplace(functionName){
    #function code
}
```

例如定义 `print` 函数，直接返回接收的值

```properties
defineReplace(print) {
    variable = $$1
    return($$variable)
}

A = $$print(hello world)
message($$A)
```

通过 `$$` 接收函数返回值。



#### Test

作为条件测试的函数体结构为

```properties
defineTest(functionName){
    #function code
}
```

例如定义一个返回 true 的函数，它可以直接作为条件

```properties
defineTest(sayHello) {
    variable = $$1
    return(true)
}

sayHello(world) {
    message(hello)
}
```



### 内置函数

正如前面提到的，qmake 提供两种函数类型。分别用来处理值和进行条件判断

* Replace 需要通过 `$$` 将返回值交给某个变量
* Test 可以直接使用，或是作为条件

我们介绍几种常用的内置函数。



#### [Replace](https://doc.qt.io/qt-5/qmake-function-reference.html)

| 函数                                    | 作用                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| absolute_path(path[, base])             | 返回 base/path 作为绝对路径，base 默认为当前路径             |
| basename(variablename)                  | 返回文件名                                                   |
| cat(filename[, mode])                   | 返回文件内容。mode 可选 blob, lines, true(default), false    |
| dirname(file)                           | 返回文件所在目录                                             |
| find(variablename, substr)              | 匹配子字符串。例如 `$$find(VAR, t.*)` 匹配 t 开头的部分      |
| files(pattern[, recursive=false])       | 匹配对应模式的文件，rescursive=true 会搜索子目录             |
| join(variablename, glue, before, after) | 将变量的每个值之间用 glue 连接后，添加 before 前缀和 after 后缀 |
| list(arg1 [, arg2 ..., argn])           | 将传入参数作为列表返回，可用于 `for` 函数                    |
| member(variablename [, start [, end]])  | 对变量进行切片，返回范围内的值                               |
| replace(string, old_string, new_string) | 字符串替换                                                   |
| size(variablename)                      | 返回变量中值的个数                                           |
| sorted(variablename)                    | 对变量的值根据 ASCII 码排序                                  |
| split(variablename, separator)          | 用 sep 分割变量值                                            |
| sprintf(string, arguments...)           | 将 string 中的 `%1` 到 `%9` 替换为对应的参数                 |
| str_member(arg [, start [, end]])       | 对字符串切片                                                 |
| str_size(arg)                           | 返回字符串的字符个数                                         |
| unique(variablename)                    | 去除变量中的重复值                                           |



函数的参数可以显式指定，例如

```properties
message($$format_number(BAD, ibase=16 width=6 zeropad))
```

否则默认按照参数顺序获得。



#### [Test](https://doc.qt.io/qt-5/qmake-test-function-reference.html#exists-filename)

| 函数                             | 作用                         |
| -------------------------------- | ---------------------------- |
| CONFIG(config)                   | 检查配置是否存在             |
| count(variablename, number)      | 检查变量是否有 number 个值   |
| equals(variablename, value)      | 检查是否相等                 |
| eval(string)                     | 计算 string 并返回 true      |
| export(variablename)             | 导出变量为全局变量           |
| greaterThan(variablename, value) | 检查是否大于                 |
| include(filename)                | 包含文件                     |
| lessThan(variablename, value)    | 检查是否小于                 |
| message(string)                  | 输出 string                  |
| if(condition)                    | 条件判断                     |
| isEmpty(variablename)            | 检查变量是否为空             |
| for(iterate, list)               | 循环 list 中的值赋给 iterate |
| defined(name[, type])            | 检查 name 是否定义           |
| exists(filename)                 | 检查文件是否存在             |
| contains(variablename, value)    | 检查是否包含 value           |



