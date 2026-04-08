# VSCode

## 基本介绍

### 命令面板

使用 `Ctrl + P` 调出命令窗口可以：

* 直接输入文件名，跳转到文件
* `?` 列出当前可执行的动作
* `@ ` 跳转到变量或函数 (C-S o)
* `#` 根据名字查找 (C t)
* `>` 输入命令

使用 `Ctrl + Shift + P` 可以直接进入输入命令的模式。



### Bash

设置命令行默认使用 Git Bash 终端。需要设置

```json
"terminal.integrated.defaultProfile.windows": "Git Bash"
```

由于默认 Git Bash 位置在

```shell
C:\Program Files\Git\bin\bash.exe
```

因此将 git 整个文件夹复制到 `C:\Program Files` 路径下，然后可以勾选 Git Bash 作为默认终端。

![[VSCode.assets/image-20231124233017734.png]]



### 代码片段

使用 `Ctrl + Shift + P` 进入命令面板，输入 snippets 配置代码片段

```json
{
	// Place your snippets for cpp here. Each snippet is defined under a snippet name and has a prefix, body and 
	// description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
	// $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
	// same ids are connected.
	// Example:
	// "Print to console": {
	// 	"prefix": "log",
	// 	"body": [
	// 		"console.log('$1');",
	// 		"$2"
	// 	],
	// 	"description": "Log output to console"
	// }

	// $1, $2 是每个 tab 停留的位置，而 $0 是 tab 最终到达的位置
	"comments": {
		"prefix": "comments",
		"body": [
			"/**",
			" * @file ${TM_FILENAME}",
			" * @author xingyifan",
			" * @date $CURRENT_YEAR-$CURRENT_MONTH-$CURRENT_DATE $CURRENT_HOUR:$CURRENT_MINUTE",
			" * ",
			" * @description: $1",
			" */",
			"$0"
		],
		"description": "auto comments"
	}
}
```



### 快捷方式

| 快捷键    | 作用                 | 快捷键  | 作用            |
| :----- | :----------------- | :--- | :------------ |
| `s-f7` | `cmake run target` | `f7` | `cmake build` |
| `s-f5` | `cmake debug`      | `f9` | `break point` |

在“文件-首选项-键盘快捷方式”下可以设置快捷方式和对应的命令

![[VSCode.assets/image-20240415235411849.png]]



如果需要设置具体的快捷方式，`Ctrl + Shift + P` 打开键盘快捷方式，输入快捷操作

```json
{
    "key": "alt+p",
    "command": "cursorMove",
    "args": {
        "to": "up",
        "by": "wrappedLine",
        "value": 10,
        "select": false
    },
    "when": "editorTextFocus"
},
{
    "key": "alt+n",
    "command": "cursorMove",
    "args": {
        "to": "down",
        "by": "wrappedLine",
        "value": 10,
        "select": false
    },
    "when": "editorTextFocus"
}
```



## Makefile 工程

安装 `C/C++ Project Generator` 插件，通过命令面板输入 create 选择 "Create C++ Project" 指定文件夹来创建工程目录。

├─.vscode
├─include
├─lib
├─output
└─src

各类文件目录包括配置文件以及 makefile 都已经给出，然后 `Ctrl + F5` 即可编译运行。需要注意在 Windows 下执行 Shell 命令可能会报错，因此在自动生成的 makefile 中注释掉 echo 代码确保正常运行。

```makefile
#
# 'make'        build executable file 'main'
# 'make clean'  removes all .o and executable files
#

# define the Cpp compiler to use
CXX = g++

# define any compile-time flags
CXXFLAGS	:= -std=c++17 -Wall -Wextra -g

# define library paths in addition to /usr/lib
#   if I wanted to include libraries not in /usr/lib I'd specify
#   their path using -Lpath, something like:
LFLAGS =

# define output directory
OUTPUT	:= output

# define source directory
SRC		:= src

# define include directory
INCLUDE	:= include

# define lib directory
LIB		:= lib

ifeq ($(OS),Windows_NT)
MAIN	:= main.exe
SOURCEDIRS	:= $(SRC)
INCLUDEDIRS	:= $(INCLUDE)
LIBDIRS		:= $(LIB)
FIXPATH = $(subst /,\,$1)
RM			:= del /q /f
MD	:= mkdir
else
MAIN	:= main
SOURCEDIRS	:= $(shell find $(SRC) -type d)
INCLUDEDIRS	:= $(shell find $(INCLUDE) -type d)
LIBDIRS		:= $(shell find $(LIB) -type d)
FIXPATH = $1
RM = rm -f
MD	:= mkdir -p
endif

# define any directories containing header files other than /usr/include
INCLUDES	:= $(patsubst %,-I%, $(INCLUDEDIRS:%/=%))

# define the C libs
LIBS		:= $(patsubst %,-L%, $(LIBDIRS:%/=%))

# define the C source files
SOURCES		:= $(wildcard $(patsubst %,%/*.cpp, $(SOURCEDIRS)))

# define the C object files
OBJECTS		:= $(SOURCES:.cpp=.o)

# define the dependency output files
DEPS		:= $(OBJECTS:.o=.d)

#
# The following part of the makefile is generic; it can be used to
# build any executable just by changing the definitions above and by
# deleting dependencies appended to the file from 'make depend'
#

OUTPUTMAIN	:= $(call FIXPATH,$(OUTPUT)/$(MAIN))

all: $(OUTPUT) $(MAIN)
	# @echo Executing 'all' complete!

$(OUTPUT):
	$(MD) $(OUTPUT)

$(MAIN): $(OBJECTS)
	$(CXX) $(CXXFLAGS) $(INCLUDES) -o $(OUTPUTMAIN) $(OBJECTS) $(LFLAGS) $(LIBS)

# include all .d files
-include $(DEPS)

# this is a suffix replacement rule for building .o's and .d's from .c's
# it uses automatic variables $<: the name of the prerequisite of
# the rule(a .c file) and $@: the name of the target of the rule (a .o file)
# -MMD generates dependency output files same name as the .o file
# (see the gnu make manual section about automatic variables)
.cpp.o:
	$(CXX) $(CXXFLAGS) $(INCLUDES) -c -MMD $<  -o $@

.PHONY: clean
clean:
	$(RM) $(OUTPUTMAIN)
	$(RM) $(call FIXPATH,$(OBJECTS))
	$(RM) $(call FIXPATH,$(DEPS))
	# @echo Cleanup complete!

run: all
	./$(OUTPUTMAIN)
	# @echo Executing 'run: all' complete!

```



### json 配置

VSCode 中 `launch.json` 和 `tasks.json` 配置文件分别起到调试可执行文件、编译程序的作用。我们接下来详细介绍这两个配置文件参数及其使用方法。常用的环境变量包括

* `${workspaceFolder}` vscode 打开的文件夹路径
* `${workspaceFolderBasename}` vscode 打开的文件夹
* `${file}` 当前打开的文件完整路径
* `${relativeFile}` 当前打开文件相对 vscode 打开的文件夹的路径
* `${relativeFileDirname}` 当前打开文件所在文件夹相对 vscode 打开的文件夹的路径
* `${fileBasenameNoExtension}` 当前打开的文件的文件名，不含后缀
* `${fileDirname}` 当前打开文件的文件夹完整路径
* `${fileExtname}` 当前打开文件的扩展名
* `${cwd}` 启动 task 时的工作目录

通过这些变量设置编译参数。



#### launch

通过 `Ctrl + F5` 快捷键会启用此配置文件，常用于调试和运行程序。 常用参数

* `preLaunchTask` 在调用 Launch 之前执行的 `tasks.json` 文件的 label 名；
* `externalConsole` 如果设为 true，将会在运行时打开终端；
* `name` 显示在“调试”侧边栏的名字；
* `type` 类型，不能修改；
* `request` 可选参数为 launch 或 attach，使用前者可以通过 `Ctrl + F5` 启用调试；
* `program` 程序所在路径和程序名
* `args` 命令行参数，对应 main 函数的两个形参，可以不填；
* `stopAtEntry` 设为 true 则会在开始执行时暂停；
* `cwd` 目标工作目录，指定调试目录；
* `environment` 手动添加环境变量；
* `MIMode` 指定调试器为 gdb 或 lldb；
* `miDebuggerPath` 指定调试器

通过 `Create C++ Project` 生成的 `launch.json` 文件为

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug",
            "type": "cppdbg",
            "request": "launch",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "linux": {
                "MIMode": "gdb",
                "miDebuggerPath": "gdb",
                "program": "${workspaceFolder}/output/main"
            },
            "osx": {
                "MIMode": "lldb",
                "miDebuggerPath": "lldb-mi",
                "program": "${workspaceFolder}/output/main"
            },
            "windows": {
                "MIMode": "gdb",
                "miDebuggerPath": "gdb.exe",
                "program": "${workspaceFolder}/output/main.exe"
            },
            "preLaunchTask": "build"
        }
    ]
}
```

这里指定了不同平台的调试方法。



#### tasks

用于自动化执行任务，这里我们用它来编译程序，以便 `launch.json` 进行调试。常用参数

* `label` 在用户界面上展示的 Task 标签；
* `type` 类型，可选 shell（作为命令运行）或 process（作为进程运行）；
* `command` 要执行的命令；
* `windows` 指定对应系统下属性；
* `group` 规定该 task 所属的组；
* `presentation` 定义用户界面如何处理 task 输出；
* `options` 定义 cwd（当前目录）、env（环境变量）和 shell 的值；
* `runOptions` 定义 task 何时运行以及如何运行；

通过 `Create C++ Project` 生成的 `tasks.json` 文件为

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build",
            "type": "shell",
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "windows": {
                "command": "powershell",
                "args": [
                    "-c",
                    "make"
                ]
            },
            "linux": {
                "command": "bash",
                "args": [
                    "-c",
                    "make"
                ]
            },
            "osx": {
                "command": "bash",
                "args": [
                    "-c",
                    "make"
                ]
            }
        },
        {
            "label": "build & run",
            "type": "shell",
            "windows": {
                "command": "powershell",
                "args": [
                    "-c",
                    "'make run'"
                ]
            },
            "linux": {
                "command": "bash",
                "args": [
                    "-c",
                    "'make run'"
                ]
            },
            "osx": {
                "command": "bash",
                "args": [
                    "-c",
                    "'make run'"
                ]
            }
        },
        {
            "label": "clean",
            "type": "shell",
            "windows": {
                "command": "powershell",
                "args": [
                    "-c",
                    "'make clean'"
                ]
            },
            "linux": {
                "command": "bash",
                "args": [
                    "-c",
                    "'make clean'"
                ]
            },
            "osx": {
                "command": "bash",
                "args": [
                    "-c",
                    "'make clean'"
                ]
            }
        }
    ]
}
```



#### c_cpp_properties

在命令面板输入 configurations 配置 JSON 文件，例如我们使用 VS 的编译器和头文件路径，同时附加 CUDA 的头文件

```cpp
{
    "configurations": [
        {
            "name": "Win32",
            "includePath": [
                "${workspaceFolder}/**",
                "D:/Visual Studio/2022/Community/VC/Tools/MSVC/14.37.32822/include/**",
                "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.3/include/**"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"
            ],
            "windowsSdkVersion": "10.0.22621.0",
            "compilerPath": "D:/Visual Studio/2022/Community/VC/Tools/MSVC/14.37.32822/bin/Hostx64/x64/cl.exe",
            "cStandard": "c17",
            "cppStandard": "c++17",
            "intelliSenseMode": "windows-msvc-x64"
        }
    ],
    "version": 4
}
```

此面板用于定义 vscode 的包含路径，编译时的宏定义，编译器路径等参数。这里 `/**` 表示搜索所有子目录。



## CMake 工程

利用 launch 和 tasks 配置一个使用 CMake 编译的项目。



### 自定义工程
#### 模拟命令行

首先要用 tasks 模拟 CMake 命令的执行过程

```json
{
	"options": {
        "cwd": "${workspaceFolder}/build"	// 设置 task 执行目录
    },
    "tasks": [
        {
            "type": "shell",	// 说明这是 shell 命令
            "label": "cmake",	// 设置命令标签
            "command": "cmake",	// 执行 cmake 命令
            "args": [
                "-DCMAKE_MAKE_PROGRAM=D:/MinGW/bin/make.exe",
                "-G",
                "MinGW Makefiles",
                ".."
            ],
        },
        {
			"label": "make",	// 设置命令标签
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "command": "make",	// 执行 make 命令
            "dependsOn": [
                "cmake"
            ]
        },
		{
            "label": "build",	// 命令标签
            "dependsOn": [
                "make"
            ]
        }
    ],
    "version": "2.0.0"
}
```

其中引入了 "dependsOn"，说明命令之间的依赖关系：`build` 依赖 `make`，`make` 依赖 `cmake`，因此会先执行依赖的命令，然后执行自身，这样指定命令的执行顺序。



由于我们使用 MinGW 编译，因此需要指定 MinGW 下的 make.exe 路径，同时指明使用 MinGW Makefiles 生成器

```json
"args": [
	"-DCMAKE_MAKE_PROGRAM=D:/MinGW/bin/make.exe",
	"-G",
	"MinGW Makefiles",
	".."
]
```

这里使用 `-D` 参数修改 `CMAKE_MAKE_PROGRAM` 变量；注意我们先把**参数项放在前面**，将 `..` 放在后面，为了确保参数中的空格不会识别错误。这样上述命令等价于

```shell
cd ${workspaceFolder}/build
cmake -DCMAKE_MAKE_PROGRAM=D:/MinGW/bin/make.exe -G "MinGW Makefiles" ..
make
```



#### 配置调试器

接着配置 launch 文件，在 Create C++ Project 的基础上进行修改

```json
{
    "version": "2.0.0",
    "configurations": [
        {
            "name": "Debug",
            "type": "cppdbg",
            "request": "launch",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "gdb32.exe",
            "program": "${workspaceFolder}/output/main.exe",
            "preLaunchTask": "build"
        }
    ]
}
```

注意由于调试需要，我们从[官网](http://www.equation.com/servlet/equation.cmd?fa=gdb)下载 gdb 的 32 位版本，重命名为 gdb32.exe 放在 MinGW 的 bin 目录下。



#### 绑定任务

我们可以将 tasks 绑定到指定的快捷键上。例如在 tasks.json 中设置

```json
{
    "tasks": [
        {
            "type": "shell",
            "label": "hello",	// 通过标签指定命令名称
            "command": "echo",
            "args": [
                "hello world"
            ]
        }
    ],
    "version": "2.0.0"
}
```

然后打开命令面板，输入 Open Keyboard Shortcuts，选择编辑 JSON 的命令。这里可以自定义用户快捷键，新定义的会覆盖默认的快捷键。编辑如下

```json
// 将键绑定放在此文件中以覆盖默认值
[
    {
        "key": "ctrl+r",
        "command": "workbench.action.tasks.runTask",
        "args": "hello"	// hello 命令
    }
]
```

这里设置 `Ctrl + r` 将会调用 hello 命令。



#### 项目文件结构

├─build
├─include (存放头文件)
├─output (存放输出文件)
└─src (存放源文件)

在主目录下创建 CMake 文件，启用 Debug 模式

```cmake
cmake_minimum_required(VERSION 3.18)
project(MAIN)

# 设置为 Debug 构建
set(CMAKE_BUILD_TYPE Debug)

add_subdirectory(src bin)
```

在 src 目录下创建编译代码

```cmake
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/output)
file(GLOB_RECURSE SRC CONFIGURE_DEPENDS *.cpp)
file(GLOB_RECURSE INC CONFIGURE_DEPENDS ../*.h)

include_directories(../include)

add_executable(main ${SRC} ${INC})
```

只需要通过 `F5` 快捷键即可启用编译调试。



### CMake 插件

#### 自动配置

安装 CMake Tools 插件，就可以识别 `CMakeLists.txt` 文件进行项目配置。例如创建

```cmake
cmake_minimum_required(VERSION 3.14)
project(MAIN)
add_executable(main main.cpp)
```

以及源文件

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello World" << std::endl;
    return 0;
}
```

通过 `Ctrl + Shift + P` 打开命令面板，输入 `cmake:config` 进行配置

![[VSCode.assets/image-20230911171620396.png]]

选择要使用的编译器后就会自动配置，在最下方可以看到操作按钮

![[VSCode.assets/image-20230911171730816.png]]

点击“生成”就会执行 Build 操作，点击 "CMake: [Debug]" 可以选择生成 Debug 或 Release 项目，还可以点击右边的两个按钮 Debug 和运行。 



#### 自动生成

还可以自动生成 CMake 项目。例如

![[VSCode.assets/image-20230925222110903.png]]

可以指定项目名称，就会自动生成一个 CMake 项目。



#### 生成器

默认使用 `Unix Makefiles` 生成器，而在 Windows 上更应该使用 `Visual Studio` 生成，因此在 `settings.json` 中添加

```json
"cmake.generator": "Visual Studio 17 2022",
```



### 单元测试

我们使用 CMake 工程进行单元测试。进入项目以后，可以在下方看到“运行 CTest”的选项

![[VSCode.assets/image-20230927112841706.png]]

先生成项目，然后运行 CTest，如果出现找不到 CTest 的错误，应该选择 cmake 文件重新生成。

![[VSCode.assets/image-20230927113110410.png]]





## Git

首先在设置中搜索 Git Path 找相关的设置选项，点击在 settings.json 中编辑

![[VSCode.assets/image-20230515105237564.png]]

添加 Git 所在目录

```json
"git.path": "D:/cmder/vendor/git-for-windows/cmd/git.exe"
```

这里因为是使用 cmder 中安装的 git，所以需要指定。默认情况下会输出 Git 日志，为了界面简洁，进一步添加设置

```json
"gitlens.currentLine.enabled": false
```



### 初始化仓库

有两种方法：一种是找到一个空文件夹，在源代码管理位置点击“初始化仓库”，即可生成 .git 文件夹初始化。之后选择添加远程储存库

![[VSCode.assets/image-20230515210807196.png]]

另一种是直接 `Ctrl + Shift + P` 调出命令面板，输入 `git clone` 选择克隆远程库。



### 本地操作

这里我们选择初始化仓库后添加远程储存库，这时候两个分支没有联系。我们需要在命令行输入

```shell
git pull --rebase Test master
```

先将远程库的内容拉取到本地。注意确保本地文件夹是空的，防止文件内容出现冲突。



本地添加的文件会显示 `U`，表明该文件还没有被 git 追踪；已经储存到远程库的本地文件被修改后显示 `M`；

![[VSCode.assets/image-20230515211426970.png]]

这时候需要点击 `+` 号将修改提交到暂存区，此时 `U` 的文件将显示 `A`，表明文件在暂存区。然后在消息框中输入 commit 信息，点击提交

![[VSCode.assets/image-20230515211634802.png]]

此时所有改动已经通过 commit 提交。然后点击“发布 Branch” 将内容推送到远程库。

![[VSCode.assets/image-20230515211929909.png]]

注意这个按钮只会出现在第一次将内容推送到远程库时，之后推送都会显示“同步更改”，点击后将会先拉取然后再推送。



### 分支操作

在源代码管理中显示管理储存库信息，其中点击 "master" 可以创建新的分支，也可以查看项目所有的分支，并点击切换分支。

![[VSCode.assets/image-20230515212311948.png]]

所有操作都可以点击 `···` 找到相关的菜单。



### Graph 插件

为了方便查看提交历史、分支关系以及版本回溯，最好安装 Git Graph 插件。点击“源代码管理储存库”下方最右边的按钮即可打开视图，在这里可以查看推送状态，点击不同的版本查看具体信息；右键点击可以进行 reset 操作，还可以切换、合并分支。

![[VSCode.assets/image-20230515212638231.png]]

