# CMake

## 基本介绍

CMake 用于创建工程，在 Windows 下将得到 VS 项目。



### 语法格式

* **基本语法格式：指令(参数 1 参数 2 ...)**
    * 参数使用 () 包含
    * 参数之间用空格或分号隔开
* 指令大小写无关，参数和变量大小写相关
* 变量使用 `${}` 取值，但是在 **IF 语句中直接使用变量名**



通过字符串可以连接变量，例如

```cmake
set(var hello)
set(var "${var} world")
```

将 var 变量赋值为 hello world 字符串。



变量名可以嵌套获得，例如

```cmake
set(hello 1)
set(hello_2 2)
message("say: ${${hello}_2}")
```



cmake 中的指令参数格式通常为

```cmake
<command_name>(arg_type1 arg1 arg_type2 arg2 ...)
```

即先**指定参数类型，然后给出参数**。对于只有一种或具有默认参数类型的指令，可以省去参数类型。



### 基本指令

#### cmake_minimum_required

指定 cmake 最小版本要求。例如

```cmake
# 指定最小版本要求为2.8.3
cmake_minimum_required(VERSION 2.8.3)
```

其中 VERSION 就是显式给出的参数类型。



#### project

定义工程名称，并指定工程支持的语言，默认支持所有语言。它的另一个作用是指定项目目录

```cmake
# 定义工程名称为 helloworld
project(helloworld)

# 定义工程名称为 helloworld 并指定语言 C, C++
project(helloworld C CXX)
```

该指定隐式定义了两个 CMAKE 变量

* `<projectname>_BINARY_DIR`：即前面例子中的 `HELLO_BINARY_DIR` 变量
* `<projectname>_SOURCE_DIR`：即前面例子中的 `HELLO_SOURCE_DIR` 变量

因此使用 project 后就可以直接使用这两个变量，它们分别是编译路径和工程路径。

> [!note]
> 最好同时指定 C/C++ 两种语言，如果不指定 C 语言就无法链接 `.c` 文件！



但是这会带来一个问题：如果项目名称改变，则变量名也会改变。更好的方法是使用预定义变量

* `PROJECT_BINARY_DIR` 或 `CMAKE_BINARY_DIR`
* `PROJECT_SOURCE_DIR` 或 `CMAKE_SOURCE_DIR`



还可以指定工程的版本，例如

```cmake
project(helloworld VERSION 2.5)
```

指定工程的版本后可以使用 `PROJECT_VERSION` 变量，它是工程的版本号。



#### set

显式定义变量，例如

```cmake
# 显式定义变量 SRC ，其值为 hello.cpp main.cpp
set(SRC hello.cpp main.cpp)
```

如果文件名中间有空格，就需要加上 `""` 区分，例如

```cmake
set(SRC "fu c.c")
```

如果不加 `""` 就会认为是列表，自动通过 `;` 分割，例如

```cmake
set(SRC fu c.c) # 等价于 "fu;c.c"
```



#### message

输出变量或者值，包含三种信息

* SEND_ERROR 产生错误，生成过程被跳过
* STATUS 输出前缀为 -- 的信息
* WARNING 输出警告信息
* FATAL_ERROR 立即终止所有 cmake 过程

例如

```cmake
message(STATUS "Hello World")	# 将输出 -- Hello World
message("Hello World")			# 将输出 Hello World
```

注意这里 STATUS 等信息都必须使用大写。



#### add_executable

生成可执行文件。例如

```cmake
# 生成可执行文件 main
add_executable(main main.cpp src.cpp)
```



#### target_sources

指定目标后，可以继续为目标链接源文件。例如

```cmake
add_executable(main)
target_sources(main PUBLIC main.cpp src.cpp)
```

其中 PUBLIC 参数在后面 `target_include_directories` 中提及。



#### set_source_files_properties

可以令指定源文件生成汇编代码，存放在可执行文件所在目录下

```cmake
set_source_files_properties(main.cpp PROPERTIES COMPILE_FLAGS "/FA")			# MSVC
set_source_files_properties(main.cpp PROPERTIES COMPILE_FLAGS "-S -masm=intel")	# GCC
```



#### 简单样例

例如源文件 main.cpp 为

```cpp
#include <iostream>

int main(int argc, char *argv[])
{
	std::cout << "Hello world!" << std::endl;
	return 0;
}
```

在当前目录下编写 CMakeLists.txt 文件

```cmake
cmake_minimum_required(VERSION 3.18)

project(HELLO)

set(SRC_LIST main.cpp)

message(STATUS "This is BINARY dir" ${HELLO_BINARY_DIR})
message(STATUS "This is SOURCE dir" ${HELLO_SOURCE_DIR})

add_executable(hello ${SRC_LIST})
```

命令行运行 `cmake ..` 即可生成 .sln 文件，打开后将 hello 项目设为启动项目，即可编译运行。



### 缓存机制

在使用 cmake 编译时，产生的 `CMakeCache.txt` 中存放了执行后的中间变量。当重复使用 cmake 时，就不需要重新扫描找到的库文件位置等信息。但是这也可能导致这些信息不能够及时更新，这时候就需要删除该文件

```shell
rm CMakeCache.txt
```

也可以指定缓存变量

```cmake
# 设置 var 缓存变量为 world 字符串
set(var "world" CACHE STRING "this is a string." FORCE)
```

其中 FORCE 表示强制更新该缓存变量；STRING 表示变量是字符串，也可以是

* FILEPATH 文件路径
* PATH 目录路径
* BOOL 布尔值，可取 ON 或 OFF

官方更建议使用 `-D` 参数来更新缓存，或者使用 `ccmake -B build` (Linux) 进入可视化界面修改。



对于 BOOL 类型的缓存，可以简写命令

```cmake
option(var "detail" ON)
```

等价于 set 指令

```cmake
set(var ON CACHE BOOL "detail")
```

因此也需要手动更新缓存。



### 生成器

使用生成器表达式简化指令

```cmake
target_compile_definitions(
	main
	PUBLIC $<$<PLATFORM_ID:Windows>:MY_NAME="Bill Gates">
       	$<$<PLATFORM_ID:Linux>:MY_NAME="Steve Jobs">
       	$<$<PLATFORM_ID:Darwin>:MY_NAME="Tim Cook">)
```

其中表达式语法为

```cmake
$<$<类型:值>:为真时的表达式>
```

在项目配置中，经常使用 `Config` 来配置不同模式下的文件。例如

```cmake
set(INPUT_FILES "gtest$<CONFIG:Debug>:d>.lib")
set(INPUT_FILES "gtest$<$<NOT:$<CONFIG:Debug>>:d>.lib")
```

用于配置 Debug 和非 Debug 模式下选取的库文件名。

> [!note]
> 由于 Config 命令是字符串的一部分，只有当 cmake 运行时才会计算值，因此纯粹的字符串操作是无效的。例如
> 
> ```shell
> set(OUT_PATH "path$<CONFIG:Debug>:d>")
> file(GLOB SRC CONFIGURE_DEPENDS ${OUT_PATH}/*.cpp)
> ```
>
> 这个操作完全没有用，无法根据是否为 Debug 模式来获取不同路径下的文件。



### 环境变量

环境变量可以在命令行中通过 -D 参数修改，例如

```shell
cmake -DCMAKE_MAKE_PROGRAM=D:/MinGW64/bin/make -DCMAKE_CXX_COMPILER=D:/MinGW64/bin/g++.exe
```

这里修改了要使用的 make 程序和 C++ 编译器。



#### 常用变量

| 变量名                        | 含义                                 |
| -------------------------- | ---------------------------------- |
| `CMAKE_CURRENT_BINARY_DIR` | 目标编译目录。使用 add_subdirectory 可以修改该变量 |
| `CMAKE_CURRENT_SOURCE_DIR` | 当前 cmake 文件路径                      |
| `CMAKE_CURRENT_LIST_DIR`   | 当前 cmake 文件完整路径                    |
| `CMAKE_CURRENT_LIST_LINE`  | 所在行标                               |
| `PROJECT_SOURCE_DIR`       | 当前项目源码目录（项目最外层目录）                  |
| `PROJECT_BINARY_DIR`       | 当前项目输出目录（`main.exe` 位置）            |
| `CMAKE_SOURCE_DIR`         | 最外层 cmake 源码根目录（不建议使用，项目不能作为子模块）   |
| `CMAKE_MODULE_PATH`        | 用于定义自定义模块的路径                       |
| `EXECUTABLE_OUTPUT_PATH`   | 可执行文件输出路径                          |
| `LIBRARY_OUTPUT_PATH`      | 库文件输出路径                            |
| `PROJECT_NAME`             | 返回 project 定义的项目名                  |
| `CMAKE_PREFIX_PATH`        | 指定 cmake 查找目录                      |
| `PROJECT_IS_TOP_LEVEL`     | 判断当前项目是否是（最顶层）根项目                  |



#### 开关选项

| 选项                          | 含义                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| `BUILD_SHARED_LIBS`           | 默认 OFF，add_library 指令默认会生成静态库；设为 ON 则默认生成动态库 |
| `CMAKE_CXX_STANDARD`          | 指定编译版本，例如 98 11 20 等设置 编译选项                  |
| `CMAKE_CXX_STANDARD_REQUIRED` | 默认 OFF 。如果设为 ON，只要不支持指定的 C++ 标准就会报错；否则会降低标准运行 |
| `CMAKE_CXX_EXTENSIONS`        | 默认 ON 。设为 OFF 会关闭 GCC 的扩展功能                     |

开关选项的设置应该在 project 指令之前，这样可以在 project 函数里对编译器进行检测。



#### CMAKE_CXX_FLAGS

g++ 编译选项

```cmake
# 在 CMAKE_CXX_FLAGS 编译选项后追加 -std=c++11
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
```

但是不推荐这样使用，因为 MSVC 环境下这个编译参数会出错。应当使用 `CMAKE_CXX_STANDARD` 来指定编译版本。



#### CMAKE_BUILD_TYPE

指定编译类型 (Debug, Release)

```cmake
# 设定编译类型为 debug ，调试时选择
set(CMAKE_BUILD_TYPE Debug)

# 设定编译类型为 release ，发布时选择
set(CMAKE_BUILD_TYPE Release)
```

在 Release 模式下追求程序的最佳性能表现，而 Debug 模式则带有调试信息。对应的编译参数为

* Debug: `-O0 -g`
* Release: `-O3 -DNDEBUG`
* MinSizeRel：最小体积发布，不完全优化
* RelWithDebInfo：带调试信息发布，不完全优化

这里 `NDEBUG` 宏会使得 assert 被去除。





## 基础构建

### 构建方法

#### 内部构建

直接在 `CMakeLists.txt` 目录下执行

```shell
cmake .
```

这就是内部构建，它会在当前目录下产生大量文件，非常麻烦。



#### 外部构建

将编译输出文件与源文件放到不同目录中

```shell
# 创建 build 文件夹
mkdir build
# 进入 build 文件夹
cd build
# 编译上级目录的 CMakeLists.txt ，生成 Makefile 和其他文件
cmake ..
# 执行 make 命令，生成 target
make
```

这样一来，如果需要重新生成 CMake 文件，只需要删除 build 文件夹。



需要注意，使用外部构建时

* `PROJECT_BINARY_DIR` 将会是编译路径，即 build 文件目录
* `PROJECT_SOURCE_DIR` 仍然是工程路径



#### 跨平台构建

在不同的系统平台下，cmake 默认产生不同的编译模式。例如 Unix 下产生 makefile，Windows 下产生 VS 项目。命令行输入

```shell
$ cmake --help
```

可以在下面的 Generators 中看到 `*` 开头的默认编译模式

```shell
Generators

The following generators are available on this platform (* marks default):
* Visual Studio 16 2019        = Generates Visual Studio 2019 project files.
                                 Use -A option to specify architecture.
  Visual Studio 15 2017 [arch] = Generates Visual Studio 2017 project files.
                                 Optional [arch] can be "Win64" or "ARM".
  Visual Studio 14 2015 [arch] = Generates Visual Studio 2015 project files.
                                 Optional [arch] can be "Win64" or "ARM".
  Visual Studio 12 2013 [arch] = Generates Visual Studio 2013 project files.
                                 Optional [arch] can be "Win64" or "ARM".
  Visual Studio 11 2012 [arch] = Generates Visual Studio 2012 project files.
                                 Optional [arch] can be "Win64" or "ARM".
  Visual Studio 10 2010 [arch] = Generates Visual Studio 2010 project files.
                                 Optional [arch] can be "Win64" or "IA64".
  Visual Studio 9 2008 [arch]  = Generates Visual Studio 2008 project files.
                                 Optional [arch] can be "Win64" or "IA64".
  Borland Makefiles            = Generates Borland makefiles.
  NMake Makefiles              = Generates NMake makefiles.
  NMake Makefiles JOM          = Generates JOM makefiles.
  MSYS Makefiles               = Generates MSYS makefiles.
  MinGW Makefiles              = Generates a make file for use with
                                 mingw32-make.
  Unix Makefiles               = Generates standard UNIX makefiles.
  Green Hills MULTI            = Generates Green Hills MULTI files
                                 (experimental, work-in-progress).
  Ninja                        = Generates build.ninja files.
  Ninja Multi-Config           = Generates build-<Config>.ninja files.
  Watcom WMake                 = Generates Watcom WMake makefiles.
  CodeBlocks - MinGW Makefiles = Generates CodeBlocks project files.
  CodeBlocks - NMake Makefiles = Generates CodeBlocks project files.
  CodeBlocks - NMake Makefiles JOM
                               = Generates CodeBlocks project files.
  CodeBlocks - Ninja           = Generates CodeBlocks project files.
  CodeBlocks - Unix Makefiles  = Generates CodeBlocks project files.
  CodeLite - MinGW Makefiles   = Generates CodeLite project files.
  CodeLite - NMake Makefiles   = Generates CodeLite project files.
  CodeLite - Ninja             = Generates CodeLite project files.
  CodeLite - Unix Makefiles    = Generates CodeLite project files.
  Sublime Text 2 - MinGW Makefiles
                               = Generates Sublime Text 2 project files.
  Sublime Text 2 - NMake Makefiles
                               = Generates Sublime Text 2 project files.
  Sublime Text 2 - Ninja       = Generates Sublime Text 2 project files.
  Sublime Text 2 - Unix Makefiles
                               = Generates Sublime Text 2 project files.
  Kate - MinGW Makefiles       = Generates Kate project files.
  Kate - NMake Makefiles       = Generates Kate project files.
  Kate - Ninja                 = Generates Kate project files.
  Kate - Unix Makefiles        = Generates Kate project files.
  Eclipse CDT4 - NMake Makefiles
                               = Generates Eclipse CDT 4.0 project files.
  Eclipse CDT4 - MinGW Makefiles
                               = Generates Eclipse CDT 4.0 project files.
  Eclipse CDT4 - Ninja         = Generates Eclipse CDT 4.0 project files.
  Eclipse CDT4 - Unix Makefiles= Generates Eclipse CDT 4.0 project files.
```



例如要生成 MinGW 的编译模式

```shell
$ cmake .. -G "MinGW Makefiles"
```



#### 现代构建

为了能够在不同平台下进行统一编译，可以使用更加现代的 cmake 命令。执行构建命令

```shell
$ cmake -S./ -Bbuild -DCMAKE_BUILD_TYPE=Release
```

其中 `-S` 指定要执行的 `CMakeLists.txt` 文件的位置，`-B` 指定构建目录。然后可执行编译命令

```shell
$ cmake --build build -j4
```

其中 `--build` 表示编译参数，`build` 是构建目录。现在项目就已经编译完成。



还可以通过 cmake 直接进行安装

```shell
$ cmake --install build
```

将会安装 `build` 目录下编译好的程序。



指定构建版本并安装到指定目录

```sh
$ cmake -B build -DCMAKE_INSTALL_PREFIX=.					# 安装目录为当前目录
$ cmake --build build --config Release --target install		# 以 Release 模式构建 build 下的项目，以 install 为生成目标
```



### 子工程

对于具有几个文件夹的工程，我们可以分别创建 `CMakeLists.txt` 文件。我们首先引入一些 cmake 指令。



#### include_directories

添加头文件搜索路径，相当于 g++ 的 -I 参数。例如

```cmake
include_directories(/usr/local /usr/share)
```

注意此命令将会为文件中的**所有目标和之后添加的子目录**都引入该头文件目录。



#### link_directories

添加库文件搜索路径，相当于 g++ 的 -L 参数，例如

```cmake
link_directories(/usr/local /usr/share)
```



#### target_include_directories

指定目标的头文件目录，例如

```cmake
target_include_directories(hello PUBLIC /usr/local /usr/share)
```

其中 PUBLIC 是目录参数

* INTERFACE：源文件只在依赖该目标的目标中可见，但不在定义的目标中使用。通常用于导出接口文件或头文件
* PRIVATE：源文件只在定义的目标中可见
* PUBLIC：源文件在定义的目标和所有依赖该目标的目标中可见

通过此命令引入的头文件可以使用 `<>` 包含。

> 如果使用了可见性参数，那么所有链接的内容都要使用此参数。



#### target_link_libraries

为目标添加需要链接的动态库，相当于指定 g++ 编译器 -l 参数

```cmake
# 为 main 目标添加需要链接的共享库 hello
target_link_libraries(main hello)
```



#### add_subdirectory

向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置。例如

```cmake
add_subdirectory(src bin)
```

即指定源文件目录为 src，然后**在 cmake 执行目录下**创建 bin 文件夹，存放中间文件和最终的可执行文件。



#### 项目文件结构

├─include (头文件目录)
├─lib (库文件目录)
└─src (源文件目录)

在 src 中定义源文件

```cpp
#include "header1.h"
#include "header2.h"
#include "header3.h"
#include "hello.h"

int main(int argc, char *argv[])
{
	SayHello();
	return 0;
}
```

在 include 中定义头文件

```cpp
#ifndef HELLO_H
#define HELLO_H

void SayHello();

#endif // HELLO_H
```

在 lib 中存放 SayHello() 函数实现的库文件 libhello.a 或 libhello.dll 。



现在在主目录下建立 cmake 文件

```cmake
cmake_minimum_required(VERSION 3.18)
project(HELLO)

# 增加子目录 src，以及目标文件夹 bin
add_subdirectory(src bin)
```

然后**还要在 src 文件夹下建立**

```cmake
set(SRC main.cpp header1.cpp header2.cpp header3.cpp)

include_directories(../include)	# 增加头文件目录
link_directories(../lib)        # 增加库文件目录

# 生成 hello.exe
add_executable(hello ${SRC})

# 链接动态库文件
target_link_libraries(hello libhello.dll)
```

注意**相对目录以当前目录**为准，另外**链接操作要在定义目标之后**。然后就可以在 build 目录下使用 cmake .. 来生成工程。



我们还可以更改最终可执行文件的存放位置，这需要更改生成 hello.exe 的 cmake 文件为

```cmake
set(SRC main.cpp header1.cpp header2.cpp header3.cpp)
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/output)

include_directories(../include)	# 增加头文件目录
link_directories(../lib)        # 增加库文件目录

# 生成 hello.exe
add_executable(hello ${SRC})

# 链接动态库文件
target_link_libraries(hello libhello.dll)
```

这里修改 `EXECUTABLE_OUTPUT_PATH` 变量为编译目录下的 output 文件夹。这样最终产生 .exe 文件会放到 output 文件夹下。



### 创建库文件

我们使用 cmake 来构建静态库和动态库，提供一个函数供其它程序使用。引入新的 cmake 指令。



#### add_library

生成库文件

```cmake
# 生成库文件，通过变量 SRC 生成 libhello.dll 动态库
add_library(hello SHARED ${SRC})
```

其中 SHARED 参数表示动态库，设为 STATIC 则表示静态库，设为 OBJECT 表示对象库。



当没有指定库类型时，会根据 `BUILD_SHARED_LIBS` 变量决定是动态库还是静态库。因此可以利用这个变量控制

```cmake
// 可以外部指定，如果没有指定则默认开启
if (NOT DEFINED BUILD_SHARED_LIBS)
	set(BUILD_SHARED_LIBS ON)
endif()

add_library(hello ${SRC})
```



如果多个库之间相互链接，例如动态库链接静态库，就需要指定 PIC (`POSITION_INDEPENDENT_CODE`) 选项

```cmake
// 设为 PIC 位置无关代码
add_library(otherlib STATIC otherlib.cpp)
set_property(TARGET otherlib PROPERTY POSITION_INDEPENDENT_CODE ON)

// 创建动态库，链接静态库
add_library(mylib SHARED mylib.cpp)
target_link_libraries(mylib PUBLIC otherlib)

add_executable(main main.cpp)
target_link_libraries(main PUBLIC mylib)
```



#### set_target_properties

设置目标的属性，还可指定动态库的版本和 API 版本

```cmake
# 将 hello 目标的输出名称设为 hi
set_target_properties(hello PROPERTIES OUTPUT_NAME "hi")

# 设置 hello 目标生成时清除同名（不含前缀）库的方法
set_target_properties(hello PROPERTIES CLEAN_DIRECT_OUTPUT 1)
```

注意 OUTPUT_NAME 和 CLEAN_DIRECT_OUTPUT 都是目标原本就有的属性。



#### get_target_properties

获得目标的属性，例如设置 hello 目标的 OUTPUT_NAME 属性为 hi，再获得 hello 的 OUTPUT_NAME 属性，结果存放在 value 变量中

```cmake
set_target_properties(hello PROPERTIES OUTPUT_NAME "hi")
get_target_property(value hello OUTPUT_NAME)
message(STATUS "This is the hello name:"${value})
```

如果不存在对应属性，返回 NOTFOUND 。



#### 项目文件结构

├─include (头文件目录)
└─lib (源文件目录)

在 lib 中分别定义函数声明和函数实现

```cpp
// hello.h

#ifndef HELLO_H
#define HELLO_H

void SayHello();

#endif // HELLO_H

// hello.cpp

#include "hello.h"
#include <iostream>

void SayHello()
{
    std::cout << "Hello World!" << std::endl;
}
```



在主目录下 cmake 为

```cmake
cmake_minimum_required(VERSION 3.18)
project(HELLO)

# 增加子目录 lib，以及目标文件夹 bin
add_subdirectory(lib bin)
```

在 lib 目录下 cmake 为

```cmake
set(LIBHELLO_SRC hello.cpp)
add_library(hello SHARED ${LIBHELLO_SRC})
```

同理于前一部分执行即可生成动态库。修改生成库文件的路径时，修改 `LIBRARY_OUTPUT_PATH` 变量即可。



如果要同时生成静态库和动态库，应当指定

```cmake
set(LIBHELLO_SRC hello.cpp)
add_library(hello SHARED ${LIBHELLO_SRC})
add_library(hello_static STATIC ${LIBHELLO_SRC})
```

注意由于目标名不能相同，所以为了区分，静态库目标修改了目标名。



要产生同名只是不同后缀的方法，应当设置

```cmake
set(LIBHELLO_SRC hello.cpp)

add_library(hello SHARED ${LIBHELLO_SRC})

set_target_properties(hello PROPERTIES OUTPUT_NAME "hello")
set_target_properties(hello PROPERTIES CLEAN_DIRECT_OUTPUT 1)

add_library(hello_static STATIC ${LIBHELLO_SRC})
set_target_properties(hello_static PROPERTIES OUTPUT_NAME "hello")
set_target_properties(hello_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)
```

这里设置 CLEAN_DIRECT_OUTPUT 属性，是因为默认情况下，每次生成目标都会清除掉同名（不含后缀）的文件。为了同时生成静态库和动态库，就必须设置保留同名文件。



另外，动态库生成通常需要版本号，增加设置

```cmake
set_target_properties(hello PROPERTIES VERSION 1.2 SOVERSION 1)
```

其中 VERSION 指代动态库版本，SOVERSION 指代 API 版本。



### 多目标工程

在一些大规模的项目中，可能存在多个编译目标，主目标依赖这些编译目标生成的库文件。引入新的 cmake 指令。



#### add_dependencies

定义目标依赖的其它目标，需要确保编译此目标之前，其它目标都已经构建

```cmake
# 主目标 main 依赖 hello 和 world 目标
add_dependencies(main hello world)
```



#### 项目文件结构

├─hello (存放 hello 库的头文件和源文件)
├─lib (用于存放库文件)
├─src (存放 main 源文件)
└─world (存放 world 库的头文件和源文件)

在 hello 目录下创建

```cpp
// hello.h

#ifndef HELLO_H
#define HELLO_H

void Hello();

#endif // HELLO_H

// hello.cpp

#include "hello.h"
#include <iostream>

void Hello()
{
    std::cout << "Hello ";
}
```



在 world 目录下创建

```cpp
// world.h

#ifndef WORLD_H
#define WORLD_H

void World();

#endif // !WORLD_H

// world.cpp

#include "world.h"
#include <iostream>

void World()
{
    std::cout << "World!" << std::endl;
}
```



我们按照如下思路构建工程：

* 用 hello world 两个目录下的文件分别生成 `libhello.a` 和 `libworld.a` 库文件，存放到 lib 目录下；
* 用 src 目录下的文件链接两个库文件生成可执行文件；

于是在 hello world 两个目录下的 cmake 文件为

```cmake
# hello

# 注意这里使用工程源文件的目录，然后进入 lib 目录作为输出路径
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

# 生成 hello 静态库
add_library(hello STATIC hello.cpp)

# world

# 注意这里使用工程源文件的目录，然后进入 lib 目录作为输出路径
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

# 生成 world 静态库
add_library(world STATIC world.cpp)
```

然后在 src 目录下链接这两个库文件

```cmake
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/output)

include_directories(../hello ../world)	# 增加两个头文件目录
link_directories(../lib)                # 增加库文件目录

# 生成 main.exe
add_executable(main main.cpp)

# 链接两个库文件
target_link_libraries(main libhello.a libworld.a)
```

最后在主目录下设置依赖关系

```cmake
cmake_minimum_required(VERSION 3.18)
project(MAIN)

# 添加三个子目录，并指定中间二进制和目标二进制存放的位置
add_subdirectory(src bin)
add_subdirectory(hello libhello)
add_subdirectory(world libworld)

# 设置 main 目标依赖 hello world 两个目标
add_dependencies(main hello world)
```

在 build 目录下同理于前面执行 cmake 和 make 即可在 output 目录下得到可执行文件。



#### 中间文件结构

在多目标工程下，我们可以分析上述代码生成的中间文件结构。

├─build
│  ├─bin (main 编译生成的文件)
│  ├─CMakeFiles
│  ├─libhello (hello 编译生成的文件)
│  ├─libworld (world 编译生成的文件)
│  └─output (存放最终可执行文件)
├─hello
├─lib (存放生成的两个库文件)
├─src
└─world

首先，在指定子目录时，我们设置了 bin, libhello, libworld 三个不同的目录，用于存放三类文件编译的中间文件。可以注意到在 build 目录下有一个 makefile，而 bin, libhello, libworld 目录下分别都有一个 makefile 文件。在单目标工程中，这两类 makefile 的作用相同，但在多目标时，build 下的 makefile 会编译所有目标，其它目录下的 makefile 只会编译对应的目标。



### 安装工程

安装主要有两种：一种是代码编译后 make install 安装，一种是打包指定目录安装。用于定义安装规则，安装的内容可以包括目标二进制、动态库、静态库以及文件、目录、脚本等。我们主要介绍可执行文件和库文件的安装。



#### install

目标文件的安装流程为

```cmake
install(TARGETS targets... 
	[
	[ARCHIVE|LIBRARY|RUNTIME]
	[DESTINATION <dir>]
	[PERMISSIONS permissions...]
	[CONFIGURATIONS [Debug|Release|...]]
	[COMPONENT <component>]
	[OPTIONAL]
	][...])
```

其中 TARGETS 后面的 targets 就是要安装的目标，目标类型有三种

* ARCHIVE 静态库
* LIBRARY 动态库
* RUNTIME 可执行目标二进制

DESTINATION 定义安装路径：**以 `/` 开头表示绝对路径，否则是相对路径**。如果不以 `/` 开头，那么安装路径为

```cmake
${CMAKE_INSTALL_PREFIX}/DESTINATION<dir>
```

其中 CMAKE_INSTALL_PREFIX 指定安装路径前缀，后面则是 DESTINATION 给出的路径。安装时输入

```shell
cmake -DCMAKE_INSTALL_PREFIX=some_dir
```



普通文件的安装流程为

```cmake
install(FILES files... 
	DESTINATION <dir>
	[PERMISSIONS permissions...]
 	[CONFIGURATIONS [Debug|Release|...]]
 	[COMPONENT <component>]
 	[RENAME <name>] [OPTIONAL])
```

目录的安装流程为

```cmake
install(DIRECTORY dirs... 
	DESTINATION <dir>
	[FILE_PERMISSIONS permissions...]
	[DIRECTORY_PERMISSIONS permissions...]
	[USE_SOURCE_PERMISSIONS]
	[CONFIGURATIONS [Debug|Release|...]]
	[COMPONENT <component>]
	[[PATTERN <pattern> | REGEX <regex>]
	[EXCLUDE] [PERMISSIONS permissions...]] [...])
```

注意 DIRECTORY 参数指定时，abc 和 abc/ 不同

* abc 表示安装目录下的文件
* abc/ 才表示安装目录



还可以指定执行安装过程中的指令

```cmake
install(SCRIPT <file> CODE <code>)
```

其中 SCRIPT 指定调用的 `.cmake` 文件，CODE 指定 cmake 指令。例如

```cmake
install(CODE "message(\"Sample install message.\")")
```

注意其中的两个 `\` 是对 `"` 符号进行转义。



#### 项目文件结构

├─doc (存放介绍文档)
├─hello
├─install (存放安装文件)
├─src
└─world

仍然沿用多目标工程中的结构，增加 doc 文件夹存放介绍文档，增加 `README.md` 文件。



我们修改多目标工程中主目录下的 cmake 文件

```cmake
cmake_minimum_required(VERSION 3.18)
project(MAIN)

add_subdirectory(src bin)
add_subdirectory(hello libhello)
add_subdirectory(world libworld)

add_dependencies(main hello world)

# 设置安装目录前缀为源文件目录
set(CMAKE_INSTALL_PREFIX ${PROJECT_SOURCE_DIR})

# 将 main 可执行文件，hello world 两个静态库安装到源文件目录下的 install/output 文件夹下
install(TARGETS main hello world
    RUNTIME DESTINATION install/output
    ARCHIVE DESTINATION install/output
    ARCHIVE DESTINATION install/output
    )

# 安装目录 doc 和文件 README.md 到 install 目录下
install(DIRECTORY doc DESTINATION install)
install(FILES README.md DESTINATION install)

# 输出完成信息
install(CODE "message(\"Install finished.\")")
```

执行 cmake 后，再执行 make install 即可完成安装。



### Find 模块

通常程序中需要引入其它模块时，例如引入头文件和库文件，都可以使用 include 指令调用。但是使用基本指令管理工程会非常复杂，因此 cmake 设计为可扩展架构，可以通过编写一些通用的模块来扩展 cmake 。我们首先引入新的指令。



#### include

用于载入其它 cmake 文件，例如

```cmake
include(CMakeLists.txt OPTIONAL)
```

相当于将 `CMakeLists.txt` 的内容直接插入。最后 OPTIONAL 表示即使文件不存在，也不报错。



#### 项目文件结构

├─src (存放 main 源文件)

我们这一部分的文件结构相当简单，在 src 下创建 main.cpp 源文件

```cpp
#include <Core>
#include <Dense>
#include <iostream>

using namespace std;
using namespace Eigen;

int main()
{
    int N = 10;
    MatrixXd A(N, N);

    // 设置一个严格对角占优矩阵，确保可逆
    for (int i = 1; i < N - 1; i++)
    {
        A(i, i) = 4;
        A(i, i - 1) = 1;
        A(i, i + 1) = 1;
    }
    A(0, 0) = A(N - 1, N - 1) = 4;
    A(0, 1) = A(N - 1, N - 2) = 1;

    // 可以求解多维向量矩阵
    MatrixXd b(N, 2);
    for (int i = 0; i < N - 1; i++)
    {
        b(i, 0) = 1;
        b(i, 1) = 2;
    }

    // 进行 Cholesky 分解
    LLT<MatrixXd> lltOfA(A);
    MatrixXd x = lltOfA.solve(b);
    MatrixXd L = lltOfA.matrixL();
    cout << "The matrix x is: \n" << x << endl;
    cout << "The matrix L is: \n" << L << endl;

    return 0;
}
```



在主目录下 cmake 为

```cmake
cmake_minimum_required(VERSION 3.18)
project(MAIN)

add_subdirectory(src bin)
```

在 src 目录下 cmake 为

```cmake
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/output)

add_executable(main main.cpp)
```

显然现在还无法编译工程，因为 Eigen3 库的头文件和库文件目录还未提供。



一种方法是通过 `include_directories` 和 `target_link_libraries` 添加。修改 src 下的 cmake 为

```cmake
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/output)

# 提供头文件目录
include_directories(D:/lib/Eigen3)

add_executable(main main.cpp)
```

因为 Eigen3 库没有库文件，因此只添加了头文件目录。



然而，我们现在要使用 cmake 提供的 Find 模块。这里使用 **Config 模式**，因此指定 EIGEN3_DIR 变量

```cmake
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/output)

# 提供 name_DIR 变量，是 Eigen3Config.cmake 文件所在的路径
set(EIGEN3_DIR "D:/lib/Eigen3/share/eigen3/cmake")

# 找到 EIGEN3 包
find_package(EIGEN3)

# 如果找到包，输出头文件目录，并包含该目录
if(EIGEN3_FOUND)
    message(STATUS ${EIGEN3_INCLUDE_DIR})
    include_directories(${EIGEN3_INCLUDE_DIR})
else(EIGEN3_FOUND)
    message(FATAL_ERROR "Eigen3 library not found.")
endif(EIGEN3_FOUND)

add_executable(main main.cpp)
```

由于我们只需要查找 `<name>Config.cmake` 文件，因此使用 `Eigen3_DIR` 作为路径变量，查找 `Eigen3` 包也是可以的。但是后面的

* EIGEN3_FOUND
* EIGEN3_INCLUDE_DIR

变量都必须大写才行。



#### 使用伪对象

然而，我们可以采用**更现代的写法**。修改 src 下的 cmake 文件

```cmake
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/output)

# 注意需要先指定 main 目标
add_executable(main main.cpp)

# 提供 name_DIR 变量，是 Eigen3Config.cmake 文件所在的路径
set(EIGEN3_DIR "D:/lib/Eigen3/share/eigen3/cmake")

# 找到 EIGEN3 包，利用 REQUIRED 参数实现包不存在时的报错效果
find_package(EIGEN3 REQUIRED)

# 链接伪对象到 main 目标
target_link_libraries(main PUBLIC Eigen3::Eigen)
```

其中 `Eigen3::Eigen` 是在 `Eigen3Targets.cmake` 文件中定义的**伪对象**。只要将其链接，就会自动链接相关的头文件和库文件。



#### 自定义模块

├─hello (模块目录)
│  ├─cmake (存放 `Find<name>.cmake` 文件)
│  ├─include (存放头文件)
│  └─lib (存放库文件)
└─src

假设我们有一个 hello 模块需要引入使用，考虑两种模式的引入方式。完成引入后，我们可以在 src 下的 `main.cpp` 中直接使用

```cmake
#include "hello.h"

int main()
{
	Hello();
    return 0;
}
```

在 build 下执行 cmake 命令即可生成。



##### Module 模式

修改 src 下的 cmake 文件

```cmake
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/output)

# 指定生成目标
add_executable(main main.cpp)

# 设置模块搜索目录
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/hello/cmake)

# 找到 HELLO 包
find_package(HELLO)

# 如果找到包，输出头文件目录，并包含该目录
if(HELLO_FOUND)
    message(STATUS "hello world headers: ${HELLO_INCLUDE_DIR}")
    message(STATUS "hello world libraries: ${HELLO_LIBRARY}")
    include_directories(${HELLO_INCLUDE_DIR})
    target_link_libraries(main ${HELLO_LIBRARY})
else(HELLO_FOUND)
    message(FATAL_ERROR "hello world library not found.")
endif(HELLO_FOUND)
```

注意设置模块搜索目录，确保能够找到 .cmake 文件；另外，**目标文件需要放在前面生成**，因为后面链接库文件时需要目标名。



在 hello/cmake 下增加 Findhello.cmake 文件

```cmake
# 获得头文件路径，存放在 HELLO_INCLUDE_DIR 中
find_path(HELLO_INCLUDE_DIR hello.h ${CMAKE_MODULE_PATH}/../include)

# 获得库文件路径，存放在 HELLO_LIBRARY 中
find_library(HELLO_LIBRARY hello ${CMAKE_MODULE_PATH}/../lib)

# 如果找到了头文件和库文件，就设置 HELLO_FOUND 为真
if(HELLO_INCLUDE_DIR AND HELLO_LIBRARY)
    set(HELLO_FOUND TRUE)
endif(HELLO_INCLUDE_DIR AND HELLO_LIBRARY)

# 如果 HELLO_FOUND 为真，就输出找到了的信息
if(HELLO_FOUND)
    message(STATUS "Found hello world: ${HELLO_INCLUDE_DIR} and ${HELLO_LIBRARY}")
else(HELLO_FOUND)
    message(FATAL_ERROR "Could not find hello world library")
endif(HELLO_FOUND)
```

注意这里我们利用了 CMAKE_MODULE_PATH 变量存放了 .cmake 文件路径的特点，获得上级目录下的 include 和 lib 目录路径。



##### Config 模式

修改 src 下的 cmake 文件

```cmake
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/output)

# 指定生成目标
add_executable(main main.cpp)

# 设置 Config 搜索目录
set(HELLO_DIR ${PROJECT_SOURCE_DIR}/hello/cmake)

# 找到 HELLO 包
find_package(HELLO)

# 如果找到包，输出头文件目录，并包含该目录
if(HELLO_FOUND)
    message(STATUS "hello world headers: ${HELLO_INCLUDE_DIR}")
    message(STATUS "hello world libraries: ${HELLO_LIBRARY}")
    include_directories(${HELLO_INCLUDE_DIR})
    target_link_libraries(main ${HELLO_LIBRARY})
else(HELLO_FOUND)
    message(FATAL_ERROR "hello world library not found.")
endif(HELLO_FOUND)
```

在 hello/cmake 下增加 `HelloConfig.cmake` 文件

```cmake
find_path(HELLO_INCLUDE_DIR hello.h ${HELLO_DIR}/../include)
find_library(HELLO_LIBRARY hello ${HELLO_DIR}/../lib)

if(HELLO_INCLUDE_DIR AND HELLO_LIBRARY)
    set(HELLO_FOUND TRUE)
endif(HELLO_INCLUDE_DIR AND HELLO_LIBRARY)

if(HELLO_FOUND)
    message(STATUS "Found hello world: ${HELLO_INCLUDE_DIR} and ${HELLO_LIBRARY}")
else(HELLO_FOUND)
    message(FATAL_ERROR "Could not find hello world library")
endif(HELLO_FOUND)
```

注意查找路径和库时，使用的变量是 HELLO_DIR 而不是模块路径。



### 测试工程

工程在编译时可能需要进行单元测试，我们可以构建 test 目标，使用 cmake 提供的 ctest 工具。首先引入新的指令。



#### enable_testing

执行此命令启用测试，它应当出现在主目录下的 cmake 文件中

```cmake
enable_testing()
```



#### add_test

执行此命令增加测试

```cmake
add_test(NAME <name>
	COMMAND <command> [<arg> ...]
	[CONFIGURATIONS <config> ...]
	[WORKING_DIRECTORY <dir>]
	[COMMAND_EXPAND_LISTS])
```

其中参数类型

* NAME 指定测试名称
* COMMAND 指定测试命令行。如果 `<command>` 指定了一个可执行文件目标，它自动替换生成该目标
* CONFIGURATIONS 指定 Debug/Release 下是否测试
* WORKING_DIRECTORY 指定执行测试的工作目录。默认为当前目录对应的生成目录
* COMMAND_EXPAND_LISTS 命令参数中的列表将展开，**可通过此参数传入外部参数到 argv 中**



#### ctest

使用 ctest 命令来执行目录下的测试程序。默认情况下只会输出 Passed/Failed 提示信息，可以增加编译参数

* `-C DEBUG` 以 DEBUG 编译类型测试
* `--output-on-failure` 将测试输出的内容打印到屏幕上
* `-v` 启用测试的详细输出
* `-vv` 启用更详细的输出



#### 项目文件结构

├─hello (引入模块)
├─lib (存放库文件)
├─src (存放源文件)
└─test (存放测试文件及其头文件)

首先是 hello 模块，其中包含头文件和源文件

```cpp
// hello.h

#ifndef HELLO_H
#define HELLO_H

void Hello();

#endif // HELLO_H

// hello.cpp

#include "hello.h"
#include <iostream>

void Hello()
{
    std::cout << "Hello ";
}
```

以及 cmake 文件

```cmake
# hello

# 注意这里使用工程源文件的目录，然后进入 lib 目录作为输出路径
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

# 生成 hello 静态库
add_library(hello STATIC hello.cpp)
```

将会生成静态库存放到 lib 目录下。



然后是 src 目录，执行主程序 `main.cpp`

```cpp
#include "hello.h"

int main()
{
	Hello();
    return 0;
}
```

对应的 cmake 文件

```cmake
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/output)

include_directories(../hello)	# 增加头文件目录
link_directories(../lib)        # 增加库文件目录

# 指定生成目标
add_executable(main main.cpp)

target_link_libraries(main libhello.a)
```



添加 test 目录，并增加两个测试文件

```cpp
// test1.cpp

#define CATCH_CONFIG_MAIN
#include "hello.h"
#include "catch.hpp"

TEST_CASE("First Test")
{
    SECTION("test")
    {
        REQUIRE(1 == 0);
    }
}

// test2.cpp

#define CATCH_CONFIG_MAIN
#include "hello.h"
#include "catch.hpp"

TEST_CASE("Second Test")
{
    SECTION("test")
    {
        REQUIRE(1 == 0);
    }
}
```

这里使用了 catch.hpp 作为测试头文件，它只用来测试，因此也放在 test 目录下。



在 test 目录下增加 cmake 文件

```cmake
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/test)

include_directories(../hello)	# 增加头文件目录
link_directories(../lib)        # 增加库文件目录

# 指定生成目标
add_executable(test1 test1.cpp)
add_executable(test2 test2.cpp)

add_test(NAME mytest1 COMMAND $<TARGET_FILE:test1>)
add_test(NAME mytest2 COMMAND $<TARGET_FILE:test2>)
```

这里设置 test 子目录产生的中间文件存放目录为 test，而之前设置 test1 test2 目标生成目录也是 test，也就是说我们将中间文件和可执行文件放在同一个目录下，方便执行测试。



然后在主目录下 cmake 改为

```cmake
cmake_minimum_required(VERSION 3.18)
project(MAIN)

# 启用 ctest
enable_testing()

add_subdirectory(hello libhello)
add_subdirectory(src bin)
add_subdirectory(test test)

# 构建依赖关系
add_dependencies(main hello)
```

增加了 test 子目录。



进入 build 文件夹，依次执行

```shell
cmake .. -G "MinGW Makefiles"
make test test2
cd test\
ctest -C DEBUG --output-on-failure
```

其中 make 生成 test 和 test2 文件可以分成两次执行。为了确保 catch 测试的提示信息被输出，需要 `--output-on-failure` 参数。当然，在使用 make 生成测试文件后，也可以分别执行两个测试程序，查看输出。



### 自定义目标

#### add_custom_target

使用此命令创建自定义目标，通常可以用来创建运行命令。例如

```cmake
add_executable(main main.cpp)
add_custom_target(run COMMAND $<TARGET_FILE:main>)	# 生成表达式会让 run 依赖 main
# 也可以手动指定依赖 add_dependencies(run main)
```

然后可以使用

```shell
cmake --build build --target run
```

构建并执行自定义目标来运行可执行文件。



再例如添加 configure 伪目标，用于可视化地修改缓存变量

```cmake
if(CMAKE_EDIT_COMMAND)
  add_custom_target(configure COMMAND ${CMAKE_EDIT_COMMAND} -B
                                      ${CMAKE_BINARY_DIR})
endif()
```

然后可以使用

```cmake
cmake --build --target configure
```

启动 ccmake 修改缓存。这在 Linux 上等价于 `ccmake -B build`，在 Windows 上等价于 `cmake-gui -B build` 。



#### add_custom_command

使用此命令创建自定义命令，可以指定它在某个目标构建前后执行

```cmake
// 在目标构建后，执行 cmake 复制命令
add_custom_command(
	TARGET ${PROJECT_NAME}
	POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy ${INPUT_FILES} ${OUTPUT_DIR})

// 执行命令行命令
add_custom_command(
	OUTPUT generated_file.txt 
	COMMAND echo "Hello, World!" > generated_file.txt 
	WORKING_DIRECTORY ${CMAKE_BINARY_DIR} 
	COMMENT "Generating file..." )

// 添加自定义目标，依赖文件，从而在构建目标时执行命令
add_custom_target(generate_file ALL DEPENDS generated_file.txt)
```



## 工程构建
### VSCode 工程

利用 launch 和 tasks 配置一个使用 CMake 编译的项目。



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
            "label": "hello",
            "command": "echo",
            "args": [
                "hello"
            ]
        }
    ],
    "version": "2.0.0"
}
```

然后打开命令面板，输入 open keyboard shutcuts，选择编辑 JSON 的命令。这里可以自定义用户快捷键，新定义的会覆盖默认的快捷键。编辑如下

```json
// 将键绑定放在此文件中以覆盖默认值
[
    {
        "key": "ctrl+r",
        "command": "workbench.action.tasks.runTask",
        "args": "hello"
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

只需要通过 `F5` 快捷键即可启用编译调试。**注意最好将头文件和源文件都作为依赖，这样后面编译 VS 工程可以读取这些文件**。



### VS 工程

#### 基本配置

在 Windows 下，CMake 将会生成 .sln 文件。通过

```cmake
include_directories(dir)
link_directories(dir)
```

来引入库文件和头文件的搜索目录。注意**这些目录要在设置目标之前引入**。



#### 调试环境

在足够高版本的 CMake 下可以使用

```cmake
set_target_properties(${PROJECT_NAME} PROPERTIES VS_DEBUGGER_ENVIRONMENT "PATH=C:/;%PATH%")
```

设置 VS 项目设置中调试选项下的环境路径，主要用于引入动态库的目录，仅在调试时可用。



需要注意，多个路径之间通过 `;` 分割，但是 `;` 在 cmake 中本身就有分割作用，因此需要转义。例如

```cmake
set_target_properties(${PROJECT_NAME} PROPERTIES VS_DEBUGGER_ENVIRONMENT 
	"PATH=C:/dir\;D:/dir\;%PATH%")
```

这种问题在连接字符串时，也会出现

```cmake
set(var "a;" "b;")		# var 将会是 ab
set(var "a\;" "b\;")	# var 将会是 a;b;
```

典型的添加所有动态库路径的方法如下

```cmake
# 添加动态库路径
set(DLL_DIR "%PATH%")
set(ALL_DLL_DIR)

# 链接第三方库
get_all_cmakes(${CMAKE_SOURCE_DIR}/deps DEPS_CMAKES)
foreach(deps_cmake ${DEPS_CMAKES})
  include(${deps_cmake})

  target_include_directories(test_cagd PRIVATE ${DEPS_INC_DIR})
  target_link_libraries(test_cagd PRIVATE ${DEPS_LIBS})
  set(ALL_DLL_DIR ${ALL_DLL_DIR} ${DEPS_DLL_DIR})
endforeach()

# 创建链接动态库
foreach(dll_dir ${ALL_DLL_DIR})
  set(DLL_DIR "${dll_dir} ${DLL_DIR}")
endforeach()
string(REPLACE " " "\;" DLL_DIR ${DLL_DIR})

set_target_properties(test_cagd PROPERTIES VS_DEBUGGER_ENVIRONMENT
                                           "PATH=${DLL_DIR}")
```



#### 运行库

在 VS 项目的“属性-配置属性-C/C++-代码生成-运行库”中可以找到运行库的设置，可选

* MT 多线程（静态）
* MTd 多线程调试（静态）
* MD 多线程（动态）
* MDd 多线程调试（动态）

默认情况下 VS 使用 MD/MDd，因此在链接 MT/MTd 编译的库时就会出现运行库错误。



我们可以在 cmake 中修改项目的运行库设置，例如

```cmake
set_property(TARGET main PROPERTY MSVC_RUNTIME_LIBRARY "MultiThreaded")
```

表示将 main 项目的运行库设为 MT 模式。可选值为

- `MultiThreaded` ：对应 `MT`
- `MultiThreadedDLL` ：对应 `MD`
- `MultiThreadedDebug`：对应 `MTd`
- `MultiThreadedDebugDLL`：对应 `MDd`

默认设置 `MultiThreadedDebugDLL` 模式。



更好的写法是

```cmake
set_property(TARGET main PROPERTY MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
```

后面跟着的表达式在项目 Config 为 Debug 时会自动添加 Debug，得到 `MultiThreadedDebug`，因此更为灵活。



#### 程序类型

对于窗口程序，需要设置入口点

```cmake
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /SUBSYSTEM:WINDOWS")	# 全局修改
set_target_properties(main PROPERTIES WIN32_EXECUTABLE TRUE)				# 只影响目标
```

配置目标为 WIN32 程序。



#### 修改字符集

默认情况下生成的项目使用**多字节字符集**，如果有需要，可以手动调整

```cmake
# 设置使用 unicode 字符集
add_definitions(-DUNICODE -D_UNICODE)
```



#### 增加分组

在项目中可以根据文件夹对源文件进行分类，我们可以通过 cmake 来设置

```cmake
# 当前目录下的源文件
file(GLOB SRC CONFIGURE_DEPENDS *.cpp)
file(GLOB INC CONFIGURE_DEPENDS *.h)

# 分组为 FileA
source_group(FileA FILES ${SRC} ${INC})
```



#### MFC 项目

首先创建如下 cmake 项目

```cpp
#include <afxwin.h>
#include <afxext.h>

// 文档类，管理数据的类，封装了对数据的管理，并和视图类进行数据交互
class CMyDoc : public CDocument
{
	DECLARE_DYNCREATE(CMyDoc)
};
IMPLEMENT_DYNCREATE(CMyDoc, CDocument)

// 视图类，负责显示数据
class CMyView : public CView
{
	DECLARE_DYNCREATE(CMyView)
public:
	virtual void OnDraw(CDC* pDC)
	{
		pDC->TextOut(100, 100, L"视图窗口");
	}
};
IMPLEMENT_DYNCREATE(CMyView, CView)

// 框架类，作为文档显示的框架
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_DYNCREATE(CMyFrameWnd)
};
IMPLEMENT_DYNCREATE(CMyFrameWnd, CFrameWnd)

// 应用程序类
class CMyWinApp : public CWinApp
{
public:
	virtual BOOL InitInstance()
	{
		return TRUE;
	}
};

// 应用程序对象的全局变量
CMyWinApp theApp;
```

以及对应的 cmake 文件

```cmake
cmake_minimum_required(VERSION 3.22)
project(Main)

# 增加 AFX 动态库
add_definitions(-D_AFXDLL)

# 修改子系统为 Windows 窗口模式（默认为控制台程序）
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /SUBSYSTEM:WINDOWS")

# 设置 MFC 库类型
set(CMAKE_MFC_FLAG 2)
# 0 不使用
# 1 静态库
# 2 动态库
# 当在 x64 环境下编译，应当使用动态库。

add_executable(main main.cpp)
```

生成得到 VS 项目后，创建菜单资源文件，就得到 `resource.h` 和 `main.rc` 两个资源文件。



然后将这两个文件复制到 `main.cpp` 同级目录下，修改 cmake 文件

```cmake
cmake_minimum_required(VERSION 3.22)
project(Main)

# 增加 AFX 动态库
add_definitions(-D_AFXDLL)

# 修改子系统为 Windows 窗口模式（默认为控制台程序）
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /SUBSYSTEM:WINDOWS")

# 设置 MFC 库类型
set(CMAKE_MFC_FLAG 2)

# 增加 rc 文件
add_executable(main main.cpp main.rc)
```

修改源文件

```cpp
#include <afxwin.h>
#include <afxext.h>
#include "resource.h"

// 文档类，管理数据的类，封装了对数据的管理，并和视图类进行数据交互
class CMyDoc : public CDocument
{
	DECLARE_DYNCREATE(CMyDoc)
};
IMPLEMENT_DYNCREATE(CMyDoc, CDocument)

// 视图类，负责显示数据
class CMyView : public CView
{
	DECLARE_DYNCREATE(CMyView)
public:
	virtual void OnDraw(CDC* pDC)
	{
		pDC->TextOut(100, 100, L"视图窗口");
	}
};
IMPLEMENT_DYNCREATE(CMyView, CView)

// 框架类，作为文档显示的框架
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_DYNCREATE(CMyFrameWnd)
};
IMPLEMENT_DYNCREATE(CMyFrameWnd, CFrameWnd)

// 应用程序类
class CMyWinApp : public CWinApp
{
public:
	virtual BOOL InitInstance()
	{
        // 创建流程
		CSingleDocTemplate* pTemplate = new CSingleDocTemplate(
			IDR_MENU1,
			RUNTIME_CLASS(CMyDoc),
			RUNTIME_CLASS(CMyFrameWnd),
			RUNTIME_CLASS(CMyView)
        );

        // 添加模板
		AddDocTemplate(pTemplate);

        // 创建新的窗口
		OnFileNew();
		m_pMainWnd->ShowWindow(SW_SHOW);
		m_pMainWnd->UpdateWindow();
		return TRUE;
	}
};

// 应用程序对象的全局变量
CMyWinApp theApp;
```

最后重新生成 VS 项目，就可以正常运行。



### Qt 工程

#### 文件结构

└─src
    └─QtFrame

为了配置方便，我们预先准备 Qt GUI 项目生成的基本文件结构，首先是 ui 文件

```html
<UI version="4.0" >
 <class>MainWindowClass</class>
 <widget class="QMainWindow" name="MainWindowClass" >
  <property name="objectName" >
   <string notr="true">MainWindowClass</string>
  </property>
  <property name="geometry" >
   <rect>
    <x>0</x>
    <y>0</y>
    <width>600</width>
    <height>400</height>
   </rect>
  </property>
  <property name="windowTitle" >
   <string>MainWindow</string>
  </property>  <widget class="QMenuBar" name="menuBar" />
  <widget class="QToolBar" name="mainToolBar" />
  <widget class="QWidget" name="centralWidget" />
  <widget class="QStatusBar" name="statusBar" />
 </widget>
 <layoutDefault spacing="6" margin="11" />
 <pixmapfunction></pixmapfunction>
 <resources>
   <include location="MainWindow.qrc"/>
 </resources>
 <connections/>
</UI>

```



然后是窗口文件

```cpp
// MainWindow.h
#pragma once

#include <QtWidgets/QMainWindow>
#include "ui_MainWindow.h"

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget* parent = nullptr);
    ~MainWindow();

private:
    Ui::MainWindowClass ui;
};



// MainWindow.cpp
#include "MainWindow.h"

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    ui.setupUi(this);
}

MainWindow::~MainWindow()
{}
```



以及 `.qrc` 资源文件

```html
<RCC>
    <qresource prefix="MainWindow">
    </qresource>
</RCC>
```



最后是主程序 `main.cpp`

```cpp
#include "MainWindow.h"
#include <QtWidgets/QApplication>

int main(int argc, char* argv[])
{
    QApplication a(argc, argv);
    MainWindow w;
    w.show();
    return a.exec();
}

```



#### 基本配置

需要添加路径到环境变量

```shell
D:\Qt\Qt5.14.2\5.14.2\msvc2017_64\bin
```



#### 提升控件

有时需要自定义控件功能，就需要右键控件提升。设置提升后的类名和引入的头文件，即可在对应的头文件中自定义类的行为。提升类的名称与该控件的名称不能一样，例如提升类为 `OCCWidget`，控件名也为 `OCCWidget`，就会得到

```cpp
OCCWidget = new OCCWidget(...);
```

编译无法通过。



#### 添加资源

当需要定制窗口内部的子窗口控件时，可以添加新的 `.ui` 文件。使用 Qt 原生的设计师软件来添加，将会自动生成对应的头文件和源文件。



#### Qt 5

在主目录下创建 CMakelist 文件

```cmake
cmake_minimum_required(VERSION 3.1.0)

project(MAIN VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

if(CMAKE_VERSION VERSION_LESS "3.7.0")
    set(CMAKE_INCLUDE_CURRENT_DIR ON)
endif()

add_subdirectory(src bin)
```



在 src 文件夹下创建

```cmake
file(GLOB QT_UI CONFIGURE_DEPENDS QtFrame/*.ui)
file(GLOB QT_QRC CONFIGURE_DEPENDS QtFrame/*.qrc)

# 编译所有 .ui 文件为 .h 文件
foreach(SRC ${QT_UI})
    string(REGEX MATCH "[a-zA-Z]+.ui" TAR ${SRC})
    string(REPLACE "ui" "h" TAR ${TAR})
    exec_program(uic ARGS "${SRC} -o ${CMAKE_CURRENT_SOURCE_DIR}/QtFrame/ui_${TAR}")
endforeach(SRC ${QT_UI})

# 编译所有 .qrc 文件为 .cpp 文件
foreach(SRC ${QT_QRC})
    string(REGEX MATCH "[a-zA-Z]+.qrc" TAR ${SRC})
    string(REPLACE "qrc" "cpp" TAR ${TAR})
    exec_program(rcc ARGS "${SRC} -o ${CMAKE_CURRENT_SOURCE_DIR}/QtFrame/qrc_${TAR}")
endforeach(SRC ${QT_QRC})

# 包含 Qt .h 和 .cpp 文件
file(GLOB QT_SRC CONFIGURE_DEPENDS QtFrame/*.cpp)
file(GLOB QT_INC CONFIGURE_DEPENDS QtFrame/*.h)

source_group(QtFrame FILES ${QT_SRC} ${QT_INC} ${QT_UI} ${QT_QRC})
include_directories(QtFrame)

# Qt
set(Qt5_DIR D:/Qt/Qt5.14.2/5.14.2/msvc2017_64/lib/cmake/Qt5)
find_package(Qt5 COMPONENTS Widgets REQUIRED)

add_executable(main main.cpp
    ${QT_SRC} ${QT_INC} ${QT_UI} ${QT_QRC}
)

# 链接 Qt
target_link_libraries(main Qt5::Widgets)
set_target_properties(main PROPERTIES VS_DEBUGGER_ENVIRONMENT "PATH=D:/Qt/Qt5.14.2/5.14.2/msvc2017_64/bin;%PATH%")
```



#### Qt 6

在主目录下创建 CMakelist 文件

```cmake
cmake_minimum_required(VERSION 3.16)

project(MAIN VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 配置 Qt6
set(CMAKE_PREFIX_PATH "D:/Qt/Qt6.6/6.6.0/msvc2019_64/lib/cmake")
find_package(Qt6 REQUIRED COMPONENTS Widgets OpenGL OpenGLWidgets)
qt_standard_project_setup()

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

add_subdirectory(src bin)
```



在 src 文件夹下创建

```cmake
# 主程序和附加头文件
file(GLOB MAIN_SRC CONFIGURE_DEPENDS *.cpp *.hpp)
file(GLOB MAIN_INC CONFIGURE_DEPENDS *.h)
include_directories(.)

# 封装程序
file(GLOB QT_SRC CONFIGURE_DEPENDS QtFrame/*.cpp)
file(GLOB QT_INC CONFIGURE_DEPENDS QtFrame/*.h)
file(GLOB QT_RC CONFIGURE_DEPENDS QtFrame/*.qrc QtFrame/*.ui)

source_group(QtFrame FILES ${QT_SRC} ${QT_INC})
include_directories(QtFrame)

# 创建 Qt 应用
qt_add_executable(main
    ${MAIN_SRC} ${MAIN_INC}
    ${QT_RC} ${QT_SRC} ${QT_INC}
)

# 链接图形库
target_link_libraries(main PRIVATE Qt6::Widgets)

# 设为 Win32 应用
set_target_properties(main PROPERTIES
    WIN32_EXECUTABLE ON
)
```



#### 动态库

最新版本的 Qt 似乎需要将 `<QtDirectory>/plugins/platforms/` 整个目录复制到应用程序目录下才能确保程序正常启动

```shell
- YourApp.exe
- platforms/
    - qwindows.dll
- Other Qt DLLs like Qt5Core.dll, Qt5Gui.dll, etc.
```

```embed
title: "How to fix qt.qpa.plugin: Could not find the Qt platform plugin “windows” in ”″ error"
image: "https://ddgobkiprc33d.cloudfront.net/fdabc3bf-87df-44e6-90c8-11d09667ec3c.png"
description: "An answer from ChatGPT just in case someone else finds it useful: **It seems like the application cannot find the Qt platform plugin, specifically the “windo…"
url: "https://forum.qt.io/topic/134667/how-to-fix-qt-qpa-plugin-could-not-find-the-qt-platform-plugin-windows-in-error/27"
```



### CUDA 工程

首先给出示范代码

```cpp
#include <stdio.h>

__global__ void hello_from_gpu()
{
    printf("Hello World from the the GPU\n");
}

int main(void)
{
    // 4 个线程块，每个线程块有 4 个线程
    hello_from_gpu<<<4, 4>>>();
    
    // 同步主机与设备（等待 GPU 执行完成），促进缓冲区刷新
    cudaDeviceSynchronize();

    return 0;
}
```

可以直接通过命令行编译

```sh
$ nvcc hello.cu -o hello
```

注意需要将 `D:\Visual Studio\2022\Community\VC\Tools\MSVC\14.37.32822\bin\Hostx64\x64` 添加到环境变量。



给出 cmake 文件

```cmake
cmake_minimum_required(VERSION 3.23)
project(hello LANGUAGES CXX CUDA)
add_executable(hello hello.cu)
set_property(TARGET hello PROPERTY CUDA_ARCHITECTURES 75) # 设置 CUDA 架构
```



### 现代工程

#### 项目结构

现代 cmake 使用目标依赖和关系传递来构建项目。考虑下面的项目结构

modern_export
│
├─include
│  └─modern
│      ├─liba
│      └─libb
├─src
│  ├─demo
│  ├─liba
│  └─libb
└─test

在根目录下给出

```cmake
cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(modern_export)

enable_testing()

# 设置安装路径前缀
set(CMAKE_INSTALL_PREFIX ${PROJECT_SOURCE_DIR}/install)

add_subdirectory(src)
add_subdirectory(test)
```

在 src 目录下给出

```cmake
# 定义导出库
add_library(liba STATIC ${CMAKE_CURRENT_SOURCE_DIR}/liba/a.cpp)
add_library(libb STATIC ${CMAKE_CURRENT_SOURCE_DIR}/libb/b.cpp)

# 定义导出库的别名
add_library(modern::liba ALIAS liba)
add_library(modern::libb ALIAS libb)

# 使用 PUBLIC 关键字将头文件目录添加到目标的 include_directories 列表中
target_include_directories(
  liba PUBLIC $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include> # 源目录下的头文件
              $<INSTALL_INTERFACE:${CMAKE_INSTALL_PREFIX}/include> # 安装后的头文件目录
)

target_include_directories(
  libb PUBLIC $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include> # 源目录下的头文件
              $<INSTALL_INTERFACE:${CMAKE_INSTALL_PREFIX}/include> # 安装后的头文件目录
)

# 添加可执行文件
add_executable(modern ${CMAKE_CURRENT_SOURCE_DIR}/demo/main.cpp)

# 链接库文件
target_link_libraries(modern modern::liba)
target_link_libraries(modern modern::libb)

# 安装库文件
install(
  TARGETS liba
  EXPORT modernTargets
  LIBRARY DESTINATION lib)

install(
  TARGETS libb
  EXPORT modernTargets
  LIBRARY DESTINATION lib)

# 安装头文件
install(DIRECTORY ${CMAKE_SOURCE_DIR}/include/ DESTINATION include)

# 指定导出目标的配置
install(
  EXPORT modernTargets
  FILE modernConfig.cmake
  NAMESPACE modern::
  DESTINATION lib/cmake/modern)
```

在 test 目录下给出

```cmake
add_executable(test_modern test.cpp)

target_link_libraries(test_modern modern::liba)
target_link_libraries(test_modern modern::libb)

add_test(NAME test1 COMMAND $<TARGET_FILE:test_modern>)
```

> [!note]
> 指定安装路径时需要注意 `/`，例如 `${CMAKE_SOURCE_DIR}/include/` 后的 `/` 如果去掉，就会将 `include` 多嵌套一层。



#### 结构分析

利用命名空间，指定安装库对应的别名，其中保存了库的基本信息，包括依赖关系。由于使用 `PUBLIC` 链接头文件路径，库的头文件信息将会传递给链接库的目录。在当前项目中，库文件的头文件路径是 `${CMAKE_SOURCE_DIR}/include`；在使用当前项目库的其它项目中，库的头文件路径则是安装路径。因此需要针对不同情况提供不同的头文件路径。

```cmake
target_include_directories(
  liba PUBLIC $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include> # 源目录下的头文件
              $<INSTALL_INTERFACE:${CMAKE_INSTALL_PREFIX}/include> # 安装后的头文件目录
)
```

执行安装命令

```shell
cmake --build build --target install
```

在安装头文件和库文件的同时，会生成 `modernConfig.cmake` 文件。



#### 链接项目库

链接当前项目库的 cmake 文件

```cmake
cmake_minimum_required(VERSION 3.10)
project(main)

add_executable(main main.cpp)

set(modern_DIR "../modern_export/install/lib/cmake/modern")
find_package(modern REQUIRED)

target_link_libraries(main modern::liba)
target_link_libraries(main modern::libb)
```

引入项目头文件

```cpp
#include <modern/liba/a.h>
#include <modern/libb/b.h>

int main() {
  a_func();
  b_func();
  return 0;
}
```



#### 封装头文件

给定一个头文件库

├─include
   └─modern

我们可以提供它的现代封装库。在根目录给出

```cmake
cmake_minimum_required(VERSION 3.10)
project(header LANGUAGES CXX)

# 设置安装路径前缀
set(CMAKE_INSTALL_PREFIX ${PROJECT_SOURCE_DIR}/install)

# 创建一个 INTERFACE 库
add_library(header INTERFACE)

# 别名
add_library(modern::header ALIAS header)

# 头文件目录
target_include_directories(
  header INTERFACE $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
                   $<INSTALL_INTERFACE:${CMAKE_INSTALL_PREFIX}/include>)

# 安装头文件
install(DIRECTORY ${CMAKE_SOURCE_DIR}/include/ DESTINATION include)

# 安装 INTERFACE 库
install(TARGETS header EXPORT headerTargets)

# 导出配置
install(
  EXPORT headerTargets
  FILE modernConfig.cmake
  NAMESPACE modern::
  DESTINATION lib/cmake/modern)
```

执行安装命令

```shell
cmake --build build --target install
```

在安装头文件的同时，会生成 `modernConfig.cmake` 文件。



在其它项目中链接

```cmake
cmake_minimum_required(VERSION 3.10)
project(main)

add_executable(main main.cpp)

set(modern_DIR "../header_export/install/lib/cmake/modern")
find_package(modern REQUIRED)

target_link_libraries(main modern::header)
```

引入头文件

```cpp
#include <modern/header.h>

int main() {
    hello();
    return 0;
}
```



## 指令操作

我们进一步开始介绍 cmake 中常用的指令以及语法操作。



### 基本指令

#### add_definitions

用于向编译器中加入 -D 定义，例如前面子工程中，将 main.cpp 改为

```cpp
#include <iostream>
#include "header1.h"
#include "header2.h"
#include "header3.h"

int main(int argc, char *argv[])
{
	std::cout << "Hello world!" << std::endl;

	#ifdef OPEN_DEBUG
		std::cout << "Open Debug!" << std::endl;
	#endif // OPEN_DEBUG

	#ifdef ABC
		std::cout << "ABC!" << std::endl;
	#endif // ABC

	return 0;
}
```

然后修改 src 下的 cmake 文件为

```cmake
set(SRC main.cpp header1.cpp header2.cpp header3.cpp)
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/output)

# 添加定义 OPEN_DEBUG 和 ABC
add_definitions(-DOPEN_DEBUG -DABC)

include_directories(../include)
add_executable(hello ${SRC})
```

于是编译工程将会输出上面三段文字。



#### aux_source_directory

发现一个目录下所有源代码文件并将列表存储在一个变量中，这个指令临时被用来自动构建源文件列表

```cmake
# 发现当前目录下的所有变量，存放在变量 SRC_LIST 中
aux_source_directory(. SRC_LIST)
add_executable(hello ${SRC_LIST})
```



#### exec_program

用于在指定目录下运行 shell 命令。例如在当前目录下执行 `ls` 命令，设置

* ARGS：命令参数为 `-l`；
* OUTPUT_VARIABLE：输出值，存放在 output_value 中；
* RETURN_VALUE：返回值，存放在 return_value 中；

```cmake
exec_program(ls ARGS "-l" OUTPUT_VARIABLE output_value RETURN_VALUE return_value)
message(STATUS "ls result: " ${output_value})
```

此命令等价于执行 `ls -l` 后，结果存放在 output_value 中，最后通过 message 输出。



#### add_compile_options

添加编译参数，例如

```cmake
add_compile_options(-std=c++11 -g -Wall)
```

其作用类似于 `CMAKE_CXX_FLAGS` 环境变量。



#### string

用于进行字符串操作，有多种模式。

* Find 模式

```cmake
string(FIND <str> <substr> <out-var> [REVERSE])
```

在 str 中查找 substr，返回第一次出现的位置到 out-var 中。若指定 REVERSE 标记，则返回最后一次出现的位置。



* Replace 模式

```cmake
string(REPLACE <match-str> <replace-str> <out-var> <input>)
```

在 input 中匹配 match-str，置换为 replace-str，输出到 out-var 中。



* 正则表达式

```cmake
string(REGEX MATCH <match-str> <out-var> <input>)
string(REGEX REPLACE <matchstr> <replace-str> <out-var> <input>)
```



利用字符串处理，我们可以实现将除 `main.cpp` 外的源文件编译为静态库

```cmake
# 找到所有 .cpp 文件，存放到 SRC 中
file(GLOB_RECURSE SRC CONFIGURE_DEPENDS *.cpp)
foreach(F ${SRC})
	# 匹配路径和后缀，将其去除
    string(REPLACE "${CMAKE_CURRENT_LIST_DIR}/" "" NAME ${F})
    string(REPLACE ".cpp" "" NAME ${NAME})
	
	# 将 main.cpp 以外的文件编译为静态库
    if(NOT NAME STREQUAL "main")
        add_library(${NAME} STATIC ${F})
    endif(NAME STREQUAL "main")
endforeach(F ${SRC})
```



### 目标指令

#### target_compile_definitions

指定目标中定义的宏

```cmake
target_compile_definitions(main PUBLIC FOO)		# 定义 FOO 宏
target_compile_definitions(main PUBLIC -DFOO)	# 取消定义 FOO 宏
```



#### target_compile_options

指定目标编译时的选项

```cmake
target_compile_options(<target> [BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```



#### target_complie_features

指定目标编译时的特性

```cmake
target_compile_features(main INTERFACE cxx_std_11)
```



#### target_link_options

指定目标链接时的选项

```cmake
target_link_options(<target> [BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```

其中 `BEFORE` 参数指定选项添加在前面而不是追加。



#### target_source

指定目标的源文件

```cmake
target_sources(<target>
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```



### file 指令

文件操作命令，可以执行多种不同类型命令。

* 读写命令

```cmake
file(OP filename message/variable)
```

其中 OP 指定对文件 filename 的操作

* WRITE 写入文件。如果该文件存在，它会覆盖它，如果不存在，它会创建该文件；
* APPEND 追加文件；
* READ 读取文件；

例如

```cmake
file(WRITE test.txt "This is a test.\n")
file(APPEND test.txt "This is another test.")
file(READ test.txt text LIMIT 10 OFFSET 10)
message(STATUS ${text})
```

其中 READ 对应的参数类型 LIMIT 表示最大读取字节数，OFFSET 表示向后的偏移字节数。



* 匹配文件

使用 GLOB 参数匹配 src 目录下的 `.cpp` 文件

```cmake
file(GLOB SRC CONFIGURE_DEPENDS src/*.cpp)
message(STATUS ${SRC})
```

使用 GLOB_RECURSE 参数匹配 src 目录**及其子目录**下的 `.cpp` 文件

```cmake
file(GLOB_RECURSE SRC CONFIGURE_DEPENDS src/*.cpp)
message(STATUS ${SRC})
```

其中 `CONFIGURE_DEPENDS` 参数使得我们创建新的 `.cpp` 文件时，能够自动更新 SRC 变量。

> [!note]
> 慎用 `GLOB_RECURSE`，尤其是在使用 Qt 时，可能会一次性把 Qt 生成的 ui 文件也包含进来，导致莫名其妙的错误。



最后的匹配表达式可以是

```cmake
*.cxx		# 匹配 .cxx 文件
*.vt?   	# 匹配 .vta .vtb ... .vtz 等任何文件
f[3-5].txt	# 匹配 f3.txt f4.txt f5.txt 文件
```



* 移动文件

将 src 下的 `main.cpp` 移动到当前目录

```cmake
file(RENAME src/main.cpp main.cpp)
```



* 删除文件

```cmake
file(REMOVE src/main.cpp)	# 删除文件
file(REMOVE_RECURSE src)	# 删除目录
```



* 创建目录

在指定目录下创建目录，如果对应目录的父目录不存在，会创建父目录

```cmake
file(MAKE_DIRECTORY src/src2/src3)
```



### list 指令

使用 list 指令创建列表

```cmake
set(SOURCE)
list(APPEND SOURCE "main.cpp")
```

列表中的元素自动由 `;` 分隔，相较于通过 `set` 连接字符串，便于管理。



### find 指令

find 是一系列查找指令，前面的章节我们已经介绍了部分，这里再给出剩余比较简单的部分。



#### find_file

用于在指定路径下查找指定名称的文件，例如在 src/ world/ 目录下查找

```cmake
find_file(name main.cpp src/ world/)
message(STATUS ${name})
```

其中 name 变量存放了 `main.cpp` 文件的全路径。



#### find_library

用于在指定路径下查找指定名称的库，例如在 src/ lib/ 目录下查找

```cmake
find_library(name world src/ lib/)
message(STATUS ${name})
```

注意对于库 `libworld.a`，查找名称可以是 `libworld.a` 或 libworld 或 world 等。



#### find_path

用于在指定路径下获得指定名称文件的路径，例如在 src/ world/ 目录下查找

```cmake
find_path(path main.cpp src/ world/)
message(STATUS ${path})
```



#### find_program

用于在指定路径下获得指定名称的程序的路径，例如在 build/output 目录下查找

```cmake
find_program(path main build/output)
message(STATUS ${path})
```




#### find_package

用来调用预定义在 `CMAKE_MODULE_PATH` 目录下的 `Find<name>.make` 模块，语法为

```cmake
find_package(<name> [major.minor] [QUIET] [NO_MODULE] [[REQUIRED|COMPONENTS] [componets...]])
```



find_package 具有两种查找模式

* Module 模式
* Config 模式

在 Module 模式下的格式为

```cmake
find_package(
	<name> 
	[version] 
	[EXACT] 
	[QUIET] 
	[MODULE] 
	[[REQUIRED|COMPONENTS] [componets...]]
	[OPTIONAL_COMPONENTS componets...])
```

其中可选参数分别为

* version 版本
* EXACT 要求版本完全匹配
* QUIET 出错时不输出
* MODULE 只在 Module 模式下查找
* REQUIRED 一定要找到包，否则报错并停止执行
* COMPONENTS 一定要找到包中要求的组件，否则报错并停止执行
* OPTIONAL_COMPONENTS 可选组件，找不到也不停止执行

它查找 `Find<name>.make` 文件，其中定义了头文件和库文件的路径。默认查找目录是 cmake 的安装目录，或者

```cmake
set(CMAKE_MODULE_PATH dir)
```

指定查找目录。如果该模式查找失败，就会进入 Config 模式。



在 Config 模式下的格式为

```cmake
find_package(
	<name> 
	[version] 
	[EXACT] 
	[QUIET] 
	[REQUIRED] 
	[[COMPONENTS] [componets...]]
	[OPTIONAL_COMPONENTS componets...]
	[CONFIG|NO_MODULE])
```

比 Module 模式更加丰富。它查找 `<name>Config.cmake` 文件，同理于前者。使用

```cmake
set(<name>_DIR dir)
```

指定查找目录。



不管使用哪一种模式，只要找到包，就会定义变量

* `<name>_FONUD` 判断模块是否找到
* `<name>_INCLUDE_DIR` 头文件目录
* `<name>_LIBRARY` 库文件

例如 `Eigen3` 的 `Eigen3Config.cmake` 文件内容为

```cmake
# This file exports the Eigen3::Eigen CMake target which should be passed to the
# target_link_libraries command.


####### Expanded from @PACKAGE_INIT@ by configure_package_config_file() #######
####### Any changes to this file will be overwritten by the next CMake run ####
####### The input file was Eigen3Config.cmake.in                            ########

get_filename_component(PACKAGE_PREFIX_DIR "${CMAKE_CURRENT_LIST_DIR}/../../../" ABSOLUTE)

macro(set_and_check _var _file)
  set(${_var} "${_file}")
  if(NOT EXISTS "${_file}")
    message(FATAL_ERROR "File or directory ${_file} referenced by variable ${_var} does not exist !")
  endif()
endmacro()

####################################################################################

if (NOT TARGET eigen)
  include ("${CMAKE_CURRENT_LIST_DIR}/Eigen3Targets.cmake")
endif ()

# Legacy variables, do *not* use. May be removed in the future.

set (EIGEN3_FOUND 1)
set (EIGEN3_USE_FILE    "${CMAKE_CURRENT_LIST_DIR}/UseEigen3.cmake")

set (EIGEN3_DEFINITIONS  "")
set (EIGEN3_INCLUDE_DIR  "${PACKAGE_PREFIX_DIR}/include/eigen3")
set (EIGEN3_INCLUDE_DIRS "${PACKAGE_PREFIX_DIR}/include/eigen3")
set (EIGEN3_ROOT_DIR     "${PACKAGE_PREFIX_DIR}")

set (EIGEN3_VERSION_STRING "3.4.0")
set (EIGEN3_VERSION_MAJOR  "3")
set (EIGEN3_VERSION_MINOR  "4")
set (EIGEN3_VERSION_PATCH  "0")
```

注意这里 `Eigen3` 的头文件目录的写法，使用 find_package 得到目录后，就可以直接使用 `<Dense>` 等头文件，而不需要增加路径。



在 Linux 系统下，默认在 `/usr/lib/cmake` 目录下查找库的配置文件；在 Windows 下，可以为

- `<prefix>/`
- `<prefix>/cmake`
- `<prefix>/<name>*/cmake`
- `<prefix>/<name>*/lib/cmake`

其中 `prefex` 是变量 `${CMAKE_PREFIX_PATH}`，默认为 `C:/Program Files`，因此可以通过修改这一变量来调整搜索路径。



### 控制指令

#### 条件指令

使用 if 指令实现条件分支，基本语法为

```cmake
if(expression)
	COMMAND1(ARGS ...)
	COMMAND1(ARGS ...)
else(expression)
	COMMAND1(ARGS ...)
	COMMAND1(ARGS ...)
endif(expression)
```

如果 expression 不是：空，0，N，NO，OFF，FALSE，NOTFOUND 或 `<var>_NOTFOUND`，则表达式为真。例如

```cmake
set(var 0)

if(var)
    message(STATUS "Hello")
else(var)
    message(STATUS "World")
endif(var)
```

需要注意 **if() 直接使用 var 作为表达式**。



**出现 IF 的地方，一定要有对应的 ENDIF** 。例如判断平台差异

```cmake
if(WIN32)
    message(STATUS "This is windows.")
else(WIN32)
    message(STATUS "This is not windows.")
endif(WIN32)
```

注意到每次分支都出现一次 WIN32 变量，可以开启变量

```cmake
set(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS ON)

if(WIN32)
    message(STATUS "This is windows.")
else()
    message(STATUS "This is not windows.")
endif()
```

这样就不需要反复指定变量了。



可以使用 ELSEIF 来得到更多分支，例如

```cmake
if(WIN32)
    message(STATUS "This is windows.")
elseif(UNIX)
    message(STATUS "This is unix.")
elseif(APPLE)
    message(STATUS "This is apple.")
endif(WIN32)
```



表达式可以具有更多形式

* `if(NOT var)`：与 `if(var)` 条件相反；
* `if(var1 AND var2)`；
* `if(var1 OR var2)`；
* `if(DEFINED var)`：如果变量被定义则为真。例如

```cmake
set(var 1)
if(DEFINED var)
	message(STATUS "Hello.")
endif(DEFINED var)
```

* 可以使用正则表达式，例如

```cmake
if("hello" MATCHES "ell")
	message(STATUS "Hello.")
endif("hello" MATCHES "ell")
```

* 还可以比较数字，例如

```cmake
if(1 LESS 2)
	message(STATUS "Hello.")
endif(1 LESS 2)
```

可使用 LESS GREATER EQUAL 操作符。

* 可以比较字符串的字典序，例如

```cmake
if("abc" STRLESS "bcd")
	message(STATUS "Hello.")
endif("abc" STRLESS "bcd")
```

可使用 `STRLESS STRGREATER STREQUAL` 操作符。



#### 循环指令

有两种用于循环的指令。



while 指令根据条件进行判断，例如

```cmake
set(var 1)

while(var)
    set(var 0)
    message(STATUS "Hello")
endwhile(var)
```



foreach 指令用途更广

* 遍历列表

```cmake
# 获得当前目录下的源文件列表
aux_source_directory(. SRC_LIST)

# 遍历列表输出文件名
foreach(F ${SRC_LIST})
	message(STATUS ${F})
endforeach(F ${SRC_LIST})
```



* 范围循环

```cmake
# 从 0 开始依次输出到 10
foreach(i RANGE 10)
	message(STATUS ${i})
endforeach(i RANGE 10)

# 从 5 开始，每次增加 3，不超过 10
foreach(i RANGE 5 10 3)
	message(STATUS ${i})
endforeach(i RANGE 5 10 3)
```

注意 foreach 指令直到遇到 endforeach 后才会开始执行。



### 自定义指令

可以自己定义指令操作，例如定义 get_all_headers 指令，获取指定目录下的所有头文件

```cmake
# 获取目录下的所有头文件
function(GET_ALL_HEADERS _dir_in, _list_out)
  file(GLOB __tmp_list "${ARGV0}/*.h")
  set(${ARGV1}
      ${__tmp_list}
      PARENT_SCOPE)
endfunction()
```

其中 ARGV0，ARGV1 分别表示传入变量，选项 `PARENT_SCOPE` 允许值传出函数。



## [CMakeRC](https://github.com/vector-of-bool/cmrc)

为了方便检索路径和读取文件，我们可以将资源文件（着色器、图片）编译到项目中，这样以后就不需要额外存放这些资源，它们将完全嵌入生成的 exe 文件中。这就需要使用 CMakeRC 库实现。



使用时需要引入对应的文件

```cmake
include(D:/lib/CMakeRC/CMakeRC.cmake)
```



### 创建资源库

使用 cmrc_add_resource_library 添加一个资源库

```cmake
cmrc_add_resource_library(ShaderLib ALIAS Shader::rc NAMESPACE Shader)
```

这表示创建一个叫 ShaderLib 的资源库，别名（ALIAS）为 Shader::rc，指定命名空间 Shader 。



接着使用 cmrc_add_resources 向库中添加资源

```cmake
cmrc_add_resources(ShaderLib "Shader/shader.vert")
```

通过这种方式添加的资源可以在 cpp 代码中按照如下形式访问

```cpp
cmrc::open("Shader/shader.vert")
```



我们也可以通过 WHENCE 指定位置。这里 WHENCE 默认情况下为 CMAKE_CURRENT_SOURCE_DIR，如果指定

```cmake
cmrc_add_resources(ShaderLib WHENCE Shader "Shader/shader.vert")
```

那么就会以 CMAKE_CURRENT_SOURCE_DIR 目录下的 Shader 作为根目录，访问方式变为

```cpp
cmrc::open("shader.vert")
```



如果有同名的资源文件，可以通过 PREFIX 添加前缀

```cmake
cmrc_add_resources(ShaderLib PREFIX resources "Shader/shader.vert")
```

访问方式变为

```cpp
cmrc::open("resources/Shader/shader.vert")
```



### 链接资源

当资源添加完毕后，和其它库一起链接到目标

```cmake
target_link_libraries(main Shader::rc ...)
```



### 访问资源

在 cmake 中构建完成后，可以在 cpp 文件中按照如下方式访问

```cpp
#include <cmrc/cmrc.hpp>

// 声明命名空间 Shader
CMRC_DECLARE(Shader);

// 获得 Shader 文件库，然后通过 path 文件名打开文件
auto fs = cmrc::Shader::get_filesystem();
auto file = fs.open(path);
```



对于文本文件，可以转换为 string 类型

```cpp
std::string code = std::string(file.begin(), file.end());
```

对于图片文件，获得其二进制内容，下面代码说明它与通过文件流读取的效果相同

```cpp
#include <cmrc/cmrc.hpp>

#include <algorithm>
#include <fstream>
#include <iostream>
#include <iterator>

CMRC_DECLARE(flower);

int main() {
    // 通过文件流读取 path 路径的图片
    std::ifstream flower_fs{path, std::ios_base::binary};
    using iter         = std::istreambuf_iterator<char>;
    const auto fs_size = std::distance(iter(flower_fs), iter());
    flower_fs.seekg(0);
	
    // 通过文件系统读取 path 路径的图片
    auto       fs        = cmrc::flower::get_filesystem();
    auto       flower_rc = fs.open(path);
    const auto rc_size   = std::distance(flower_rc.begin(), flower_rc.end());
    
    // 说明读取大小相同，内容也相同
    if (rc_size != fs_size) {
        std::cerr << "Flower file sizes do not match: FS == " << fs_size << ", RC == " << rc_size
                  << "\n";
        return 1;
    }
    if (!std::equal(flower_rc.begin(), flower_rc.end(), iter(flower_fs))) {
        std::cerr << "Flower file contents do not match\n";
        return 1;
    }
}
```



### 可用方法

然后我们介绍文件系统可以使用的成员函数

| 函数              | 作用                                                         |
| ----------------- | ------------------------------------------------------------ |
| open              | 打开指定路径的文件。如果错误，抛出 `std::system_error()` 异常 |
| is_file           | 检查指定路径是否是文件，返回 bool                            |
| is_directory      | 检查指定路径是否是目录，返回 bool                            |
| exists            | 检查指定路径是否存在，返回 bool                              |
| iterate_directory | 返回指定路径下的文件迭代器                                   |

例如可以通过文件迭代器获得路径下的资源

```cpp
auto fs = cmrc::iterate::get_filesystem();

// 访问文件系统中的所有内容
for (auto&& entry : fs.iterate_directory(""))
    std::cout << entry.filename() << '\n';

// 访问文件系统中指定路径下的所有内容
for (auto&& entry : fs.iterate_directory("subdir_a/subdir_b"))
    std::cout << entry.filename() << '\n';
```

其中 entry 有如下成员函数

| 函数         | 作用           |
| ------------ | -------------- |
| filename     | 返回文件名     |
| is_file      | 返回是否是文件 |
| is_directory | 返回是否是目录 |



## CPM

CPM.cmake 是一个跨平台的 CMake 脚本，用于向 CMake 添加依赖项管理功能。它作为 CMake 的 FetchContent 模块构建的瘦包装器，该模块添加了版本控制、缓存、简单的 API 等。



### FetchContent

首先需要介绍 FetchContent 命令，它是在 cmake 3.14 中添加的新命令，可以用来直接从 GIT 库拉取依赖。

```cmake
include(FetchContent)
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG        release-1.8.0
)
FetchContent_Declare(
  Catch2
  GIT_REPOSITORY https://github.com/catchorg/Catch2.git
  GIT_TAG        v2.5.0
)

# After the following call, the CMake targets defined by googletest and
# Catch2 will be defined and available to the rest of the build
FetchContent_MakeAvailable(googletest Catch2)
```

但是 FetchContent 具有较慢的速度，而 CPM 则作为一个更好的选择方案。



### 导入 CPM

可以直接将其 include 进项目 cmake 文件

```cmake
include(CPM.cmake)
```

或者通过如下代码

```cmake
set(CPM_DOWNLOAD_VERSION 0.27.2) 
set(CPM_DOWNLOAD_LOCATION "${CMAKE_BINARY_DIR}/cmake/CPM_${CPM_DOWNLOAD_VERSION}.cmake")

if(NOT (EXISTS ${CPM_DOWNLOAD_LOCATION}))
    message(STATUS "Downloading CPM.cmake")
    file(DOWNLOAD https://github.com/TheLartians/CPM.cmake/releases/download/v${CPM_DOWNLOAD_VERSION}/CPM.cmake ${CPM_DOWNLOAD_LOCATION}) 
endif()

include(${CPM_DOWNLOAD_LOCATION})
```

它每次检查 CPM 是否存在，如果不存在则会下载。



### 添加库

有两种方法可以添加库：GitHub 地址，例如加载 nlohmann/json 库

```cmake
CPMAddPackage(
    NAME nlohmann_json
    GITHUB_REPOSITORY nlohmann/json
    VERSION 3.6.1)
```

或者使用链接

```cmake
CPMAddPackage(
    NAME nlohmann_json	 
    VERSION 3.6.1 	 
    URL https://github.com/nlohmann/json/releases/download/v3.6.1/include.zip	 
    URL_HASH SHA256=69cc88207ce91347ea530b227ff0776db82dcb8de6704e1a3d74f4841bc651cf)
    
if(nlohmann_json_ADDED)	 
    add_library(nlohmann_json INTERFACE)	 
    target_include_directories(nlohmann_json INTERFACE ${nlohmann_json_SOURCE_DIR})	 
endif()
```



添加库后需要将其链接到项目中，完整的流程如下：

```cmake
cmake_minimum_required(VERSION 3.14 FATAL_ERROR)

# create project
project(MyProject)

# add executable
add_executable(main main.cpp)

# add dependencies
include(cmake/CPM.cmake)

CPMAddPackage("gh:fmtlib/fmt#7.1.3")
CPMAddPackage("gh:nlohmann/json@3.10.5")
CPMAddPackage("gh:catchorg/Catch2@3.4.0")

# link dependencies
target_link_libraries(main fmt::fmt nlohmann_json::nlohmann_json Catch2::Catch2WithMain)
```

这里 `CPMAddPackage` 使用了 GitHub 方式导入库，并且导入方式进行了简写。



### 指令介绍

导入库的命令格式为

```cmake
CPMAddPackage(
  NAME          # 唯一依赖名称，作为导出名称
  VERSION       # 可选，作为最小版本要求，默认为 0
  OPTIONS       # 可选，配置传递给依赖的选项
  DOWNLOAD_ONLY # 可选，如果指定，则会下载依赖，但不会配置
  [...]         # 转发给 FetchContent 的参数
)
```

还可以使用紧凑格式

```cmake
# A git package from a given uri with a version
CPMAddPackage("uri@version")
# A git package from a given uri with a git tag or commit hash
CPMAddPackage("uri#tag")
# A git package with both version and tag provided
CPMAddPackage("uri@version#tag")
```

其中 `uri` 可以是 `gh` 即 GitHub 或 `bb` 即 BitBucket 或 `gl` 即 GitLab 。例如 `gh:user/name` 将其解释为 GitHub URI 并转换为

```sh
https://github.com/user/name.git
```



也可以对 URL 使用这种语法

```cmake
# An archive package from a given url. The version is inferred
CPMAddPackage("https://example.com/my-package-1.2.3.zip")
# An archive package from a given url with an MD5 hash provided
CPMAddPackage("https://example.com/my-package-1.2.3.zip#MD5=68e20f674a48be38d60e129f600faf7d")
# An archive package from a given url. The version is explicitly given
CPMAddPackage("https://example.com/my-package.zip@1.2.3")
```

调用 `CPMAddPackage` 后，局部作用域中会定义以下变量，其中 `<dependency>` 是依赖项的名称：

- `<dependency>_SOURCE_DIR` 是依赖项源的路径。
- `<dependency>_BINARY_DIR` 是依赖项的构建目录的路径。
- `<dependency>_ADDED` 如果之前未添加依赖项，则设置为 `YES`，否则设置为 `NO` 。
- `CPM_LAST_PACKAGE_NAME` 设置为最后添加的依赖项的确定名称（相当于 `<dependency>` ）。

若要将 `CPM.cmake` 项目与外部包管理器（如 conan 或 vcpkg）一起使用，设置变量 `CPM_USE_LOCAL_PACKAGES` 将使 `CPM.cmake` 首先尝试添加 `find_package` 包，如果不成功，则从源添加包。如果使用命令 `CPMFindPackage`，则会先调用 `find_package` 然后调用 `CPMAddPackage` 。



### 选项

* CPM 选项 `CPM_SOURCE_CACHE`，通过 `-DCPM_SOURCE_CACHE=<path to an external download directory>` 传递给 CMake，避免重新下载依赖项。这也将允许离线配置项目，只要之前已将依赖项添加到缓存中即可。
* CPM 选项 `CPM_USE_LOCAL_PACKAGES` 可以要求 `CPMAddPackage` 来搜索本地安装的依赖项，输入的参数会转发给 `find_package` 。
* 设置 CPM 选项 `CPM_LOCAL_PACKAGES_ONLY`，则如果在本地找不到依赖项，CPM 将发出错误。

