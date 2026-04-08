# GME 代码

## 编译配置

### 主项目配置

#### CMake

使用 CMake 最新版本进行配置，勾选除了 BUILD_SAMPLES 以外的其它 BUILD 选项，配置后修改 Qt6_PATH 变量， 然后生成项目。

![[GME 代码.assets/image-20231123145752295.png]]



#### 修改编码

选择 demo_acis 编译，将报错的所有文件的编码修改为 UTF-8 with BOM 格式。保险起见删除 build 文件夹中的所有文件，重新生成一次，然后编译 demo_acis 项目。



### 脚手架配置

直接用 CMake 编译生成项目，选择对应的项目进行开发即可。



## 脚手架代码框架

### 文件结构

在框架中，已经给出了作为示例的 base 模块。在源代码结构中，对于任意一个完整的模块，其头文件，源文件，测试文件，示例文件分属于不同的文件夹。其定位为：include 和 src 文件夹分别声明和实现功能类和功能函数，test 文件夹实现接口的测试用例，samples 文件夹实现接口的简单调用教程。

```sh
├─_deps
├─cmake  # cmake脚本
├─docs  # 文档
├─include  # 头文件
│  └─base
├─resources  # 数据文件
├─samples  # 示例文件
│  └─base
├─scripts  # 脚本
├─src  # 源文件
│  └─base
├─test  # 测试文件
│  └─base
└─tools  # 其他工具
```



### cmake

cmake 目录下提供了两个脚本：

```cmake
# Notes CMakeUtils.cmake

# 获取目录下的所有头文件
function(GET_ALL_HEADERS _dir_in, _list_out)
    file(GLOB_RECURSE __tmp_list "${ARGV0}/*.h" "${ARGV0}/*.hpp"
         "${ARGV0}/*.hxx")
    set(${ARGV1}
        ${__tmp_list}
        PARENT_SCOPE)
endfunction()

# 获取目录下的所有源文件
function(GET_ALL_SOURCES _dir_in, _list_out)
    file(GLOB_RECURSE __tmp_list "${ARGV0}/*.c" "${ARGV0}/*.cpp"
         "${ARGV0}/*.cxx")
    set(${ARGV1}
        "${__tmp_list}"
        PARENT_SCOPE)
endfunction()

```

这个脚本中的两个函数比较类似，`GET_ALL_HEADERS` 接受两个参数 `__dir_in,_list_out`，传入后

* `ARGV0` 表示第一个参数
* `ARGV1` 表示第二个参数

通过 `file` 命令将 `__dir_in` 对应路径下的头文件存放到 `__tmp_list` 中，然后使用 `set` 和 `PARENT_SCOPE` 选项传递给 `_list_out` 。注意

> `PARENT_SCOPE` 选项表示父作用域，这样就可以修改外部变量 `_list_out` 的值。

另一个是 CPM.cmake，是一个开源包管理脚本。



### docs

docs 目录下提供了生成文档的 doxygen 命令脚本

```cmake
cmake_minimum_required(VERSION 3.14)

project(Docs)

# ---- Doxygen variables ----

# 制定 Doxygenfile 变量
set(DOXYGEN_PROJECT_NAME "GME Documentation")
set(DOXYGEN_PROJECT_ROOT ${CMAKE_SOURCE_DIR})
set(DOXYGEN_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/out")

# 将 Doxygenfile 文件从 ${CMAKE_CURRENT_LIST_DIR} 移动到 ${CMAKE_CURRENT_BINARY_DIR} 目录下
configure_file(${CMAKE_CURRENT_LIST_DIR}/Doxyfile
               ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

# 添加自定义目标，这个目标不是生成目标，而是可以调用的命令
add_custom_target(
    GenerateDocs
    ${CMAKE_COMMAND} -E make_directory "${DOXYGEN_OUTPUT_DIRECTORY}"
    COMMAND "doxygen" "${CMAKE_CURRENT_BINARY_DIR}/Doxyfile"
    COMMAND echo "Docs written to: ${DOXYGEN_OUTPUT_DIRECTORY}"
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}")

```

新的目标由 4 条指令构成

* 调用 `cmake` 命令，创建 Doxygen 输出目录
* 调用命令行命令 `doxygen` 读取 Doxygenfile
* 调用命令行命令 `echo` 输出信息
* 指定工作目录



### include

include 目录下存放模块需要引用的头文件。



### resources

resources 目录下存放数据文件。



### samples

samples 目录下存放不同模块的示例文件。还包含一个 cmake 文件，从模块列表变量 `${SUBMODULES}` 中获得所有子目录，添加到项目

```cmake
foreach(module ${SUBMODULES})
    add_subdirectory(${module})
endforeach()
```



### scripts

scripts 目录下存放脚本。首先是删除 out 文件夹的脚本

```bat
RD /s %~dp0\..\\out
RD /s %~dp0\..\\test\\out
RD /s %~dp0\..\\samples\\out
pause
```

其中 `RD` 是删除命令，`/s` 表示递归删除，`%~dp0` 表示当前目录，因此第一条命令递归删除当前目录的上级目录的子目录 out 及其内容。最后一条命令会暂停命令行，显示 `Press any key to continue...`，允许在脚本运行结束后查看输出。



在 gtest-parallel 目录下提供允许 gtest 并行运行的库。



### src

src 目录下存放不同模块的源文件。还包含一个 cmake 文件，从模块列表变量 `${SUBMODULES}` 中获得所有子目录，添加到项目

```cmake
foreach(module ${SUBMODULES})
    add_subdirectory(${module})
endforeach()
```



### test

test 目录下存放不同模块的测试文件。还包含一个 cmake 文件

```cmake
cmake_minimum_required(VERSION 3.14)

project(tests CXX C)

# 包含前面的 cmake 脚本
include(${CMAKE_CURRENT_SOURCE_DIR}/../cmake/CMakeUtils.cmake)
# ---- Import Google Test ----

# 声明空变量，存放当前目录下的所有模块下的 *.cpp 文件格式 
set(TEST_DIR)
foreach(module ${SUBMODULES})
    set(TEST_DIR ${TEST_DIR} "${CMAKE_CURRENT_SOURCE_DIR}/${module}/*.cpp")
endforeach()

# 额外添加几个文件
set(TEST_DIR
    ${TEST_DIR} "${CMAKE_CURRENT_SOURCE_DIR}/utils.hxx"
    "${CMAKE_CURRENT_SOURCE_DIR}/utils.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/main.cpp" ${GME_DEPS_DIR}/acis/acis_utils.cpp)

# 将前面保存的所有目录下的文件存放在 TEST_SOURCE_FILES 变量中；将 src 目录下的源文件存放到 SRC_SOURCE_FILES 变量中
# 注意 cmake 的命令大小写不敏感，因此这里 get_all_sources 其实就是前面 CMakeUtils 中自定义的命令
file(GLOB_RECURSE TEST_SOURCE_FILES CONFIGURE_DEPENDS ${TEST_DIR})
get_all_sources("${CMAKE_CURRENT_SOURCE_DIR}/../src" SRC_SOURCE_FILES)

# 添加可执行文件
add_executable(${PROJECT_NAME} ${SRC_SOURCE_FILES} ${TEST_SOURCE_FILES})

# 增加包含路径
target_include_directories(
    ${PROJECT_NAME}
    PUBLIC ${GME_DEPS_DIR}/gtest/include
    PUBLIC ${GME_DEPS_DIR}
    PUBLIC ${GME_INCLUDE_DIR})

# 链接依赖库
target_link_libraries(${PROJECT_NAME} debug "${BASE_DEPS}d.lib" optimized
                      "${BASE_DEPS}.lib")

target_link_libraries(
    ${PROJECT_NAME} debug ${GME_DEPS_DIR}/gtest/lib/gtestd.lib optimized
    ${GME_DEPS_DIR}/gtest/lib/gtest.lib)

# 启用设置
set(gtest_force_shared_crt
    ON
    CACHE BOOL "Always use msvcrt.dll" FORCE)

# 安装命令
install(TARGETS ${PROJECT_NAME} DESTINATION ${GME_INSTALL_BIN_DIR})

# 指定输出路径
set(OUTPUT_DIR
    "$<$<CONFIG:Debug>:"
    "${EXECUTABLE_OUTPUT_PATH}/Debug"
    ">"
    "$<$<CONFIG:Release>:"
    "${EXECUTABLE_OUTPUT_PATH}/Release"
    ">"
    "$<$<CONFIG:RelWithDebInfo>:"
    "${EXECUTABLE_OUTPUT_PATH}/RelWithDebInfo"
    ">")
string(REPLACE ";" "" OUTPUT_DIR ${OUTPUT_DIR})

# 指定输入文件（都是 dll）
set(INPUT_FILE
    "$<$<CONFIG:Debug>:" ${GME_DEPS_DIR}/gtest/dll/gtestd.dll ">"
    "$<$<NOT:$<CONFIG:Debug>>:" ${GME_DEPS_DIR}/gtest/dll/gtest.dll ">")
string(REPLACE ";" "" INPUT_FILE ${INPUT_FILE})

# 增加自定义命令，关联目标 PROJECT_NAME，选项 POST_BUILD 指定在目标构建后执行
# COMMAND 指定执行 cmake 命令，提供后面的参数
# 作用是将输入文件移动到输出文件夹下
add_custom_command(
    TARGET ${PROJECT_NAME}
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} ARGS -E copy ${INPUT_FILE} ${OUTPUT_DIR}/.)

set(INPUT_FILE2 "$<$<CONFIG:Debug>:" "${BASE_DEPS}d.dll" ">"
                "$<$<NOT:$<CONFIG:Debug>>:" "${BASE_DEPS}.dll" ">")
string(REPLACE ";" "" INPUT_FILE2 ${INPUT_FILE2})

# 拷贝dll文件到可执行文件输出目录
add_custom_command(
    TARGET ${PROJECT_NAME}
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy ${INPUT_FILE2} ${OUTPUT_DIR}/.
)

```



### tools

tools 目录下提供 doxygen 和 clang-format 程序用于调用。



### 主目录

主目录下存放 cmake 和 clang 格式配置文件。以及核心 cmake 文件

```cmake
cmake_minimum_required(VERSION 3.14)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_EXTENSIONS OFF)

# 编译类型包括 Debug, Release 和 RelWithDebInfo
# CACHE 指定变量缓存为字符串，并 FORCE 强制执行
set(CMAKE_CONFIGURATION_TYPES
    "Debug;Release;RelWithDebInfo"
    CACHE STRING "" FORCE)

# 设置 DEBUG 后缀 d，使用它区分发布版本和调试版本
set(CMAKE_DEBUG_POSTFIX d)

# 启用开发者模式，缓存为 BOOL 变量，提供描述
set(ENABLE_DEVELOPER_MODE
    TRUE
    CACHE BOOL "Enable 'developer mode'")
set(OPT_WARNINGS_AS_ERRORS_DEVELOPER_DEFAULT TRUE)

# 提供项目信息
project(
    GME
    VERSION 0.0.1
    DESCRIPTION ""
    HOMEPAGE_URL "https://thu-cad.github.io/gme/"
    LANGUAGES CXX C)

# 定义 UNICODE 和 _UNICODE 和 SPA_NO_AUTO_LINK 宏
add_definitions(-DUNICODE -D_UNICODE -DSPA_NO_AUTO_LINK)

# 指定测试数据路径
add_definitions(-DTEST_DATA_PATH="${CMAKE_SOURCE_DIR}/data")

# 设置一些会使用到的变量
set(CMAKE_INSTALL_PREFIX ${PROJECT_BINARY_DIR})
set(GME_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/include)
set(GME_DEPS_DIR "${CMAKE_SOURCE_DIR}/_deps")
set(GME_SAMPLES_DIR "${CMAKE_SOURCE_DIR}/samples")
set(GME_INSTALL_BIN_DIR ${CMAKE_BINARY_DIR}/bin)
set(GME_INSTALL_LIB_DIR ${CMAKE_BINARY_DIR}/lib)
set(GME_BUILD_DIRECTORY ${CMAKE_BINARY_DIR})

set(LIBRARY_OUTPUT_PATH ${GME_BUILD_DIRECTORY})
set(EXECUTABLE_OUTPUT_PATH ${GME_BUILD_DIRECTORY})

# doxygen 编译开关
set(BUILD_DOXYGEN
    "ON"
    CACHE BOOL "Choose whether to build the DocGenerate module or not")

# 指定模块和工具是否构建
set(BUILD_TEST
    "ON"
    CACHE BOOL "Choose whether to build the test module or not")
set(BUILD_FORMAT
    "ON"
    CACHE BOOL "Choose whether to build the format tool or not")
set(DECOUPLE_ACIS
    "OFF"
    CACHE BOOL "Choose whether to decouple from ACIS or not")
set(BUILD_SAMPLES
    "ON"
    CACHE BOOL "Choose whether to build the sample module or not") # NOTES
                                                                   # sample
                                                                   # option

# 如果解耦 ACIS，就定义 DECOUPLE_ACIS 宏
if(${DECOUPLE_ACIS})
    add_definitions(-DDECOUPLE_ACIS)
endif()

# 如果设置了环境变量 CI 或者启用了 BUILD_FORMAT
if(($ENV{CI}) OR (${BUILD_FORMAT}))
	# 就包含 CPM 脚本，添加 cmake-format 包
    include("${CMAKE_SOURCE_DIR}/cmake/CPM.cmake")
    cpmaddpackage(
        NAME cmake-format VERSION 1.7.0 SOURCE_DIR
        ${GME_DEPS_DIR}/Format.cmake-1.7.0 OPTIONS "FORMAT_SKIP_CMAKE YES")
endif()

# 列出所有子模块文件夹名字,倒序写，自顶向下遍历
set(OPTION_SUBMODULES "子模块" "BASE")

# 用于记录本次编译所编译的模块
set(SUBMODULES)

# 各模块编译开关 base 必须编译
set(BUILD_BASE
    "ON"
    CACHE BOOL "Choose whether to build the base module or not")
# set(BUILD_EXAMPLE "ON" CACHE BOOL "Choose whether to build the example module
# or not")

# 各模块的依赖模块，依赖会传递
set(BASE_DEPS
    ${GME_DEPS_DIR}/acis/lib/SpaACIS
    CACHE INTERNAL "Path to SpaACIS lib")

# 遍历所有子模块（倒序列表，自顶向下）
foreach(module ${OPTION_SUBMODULES})
	# 检查是否 BUILD 这个模块，并且这个模块不是 BASE
    if(${BUILD_${module}} AND NOT ${module} STREQUAL "BASE")
    	# 连接模块名和 DEPS，然后将它们全都变为大写，存放在 UPPER_MODULE 中
    	# 启用 BUILD 这个模块
        foreach(dep_module ${${module}_DEPS})
            string(TOUPPER ${dep_module} UPPER_MODULE)
            set(BUILD_${UPPER_MODULE}
                "ON"
                CACHE BOOL
                      "Choose whether to build the ${dep_module} module or not"
                      FORCE)
        endforeach()
    endif()
endforeach()

# 遍历所有子模块（倒序列表，自顶向下）
foreach(module ${OPTION_SUBMODULES})
	# 如果 BUILD 这个模块，将模块名小写，存放在 lower_module 中，然后将其放在 SUBMOUDULES 变量的前面
    if(${BUILD_${module}})
        string(TOLOWER ${module} lower_module)
        set(SUBMODULES ${lower_module} ${SUBMODULES})
    endif()
endforeach()

# 添加源文件目录
add_subdirectory(src)

# 如果构建 TEST，添加 test 目录
if(${BUILD_TEST})
    add_subdirectory(test)
endif()

# 如果构建 samples，添加 samples 目录
if(${BUILD_SAMPLES})
    add_subdirectory(samples)
endif()

# 如果构建 doxygen，添加 docs 目录
if(${BUILD_DOXYGEN})
    add_subdirectory(docs)
endif()
```



## GME 源码框架

源码包括

* _deps 依赖第三方库文件目录
* data 存放数据文件
* doc 存放 doxygen 文件
* include 包含头文件
* scripts 存放脚本
* src 存放与 include 相关的源文件
* src_acis 存放与 acis 中的 include 相关的源文件
* test 存放测试样例
* tools 存放格式化工具
* utils 存放工具脚本

主要关注 include 中的模块，它与 src 中的源码对应。



### acisadaptor

此模块存放 gme 与 acis 数据格式的转换声明。



### base

此模块存放辅助函数的声明。



### boolean

此模块存放布尔操作的声明。



### clearance

此模块存放删除操作的声明。



### constructor

此模块存放构造基本元素的函数声明。



### defeature

此模块存放拆分实体操作的声明。



### euler

此模块存放欧拉操作的声明。



### faceter

此模块存放 facet 操作的声明。



### intersector

此模块存放求交操作的声明。



### kernel

此模块存放各种基本数据结构的声明。



### law

此模块存放不同维欧几里得空间之间的变换声明。



### query

此模块存放查询操作的声明。



### spdadaptor

此模块存放 spd 格式与 gme 格式之间的适配器声明。



### 协程生成器


