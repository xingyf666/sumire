# Xmake

## 基本知识

### 安装配置

```shell
irm https://xmake.io/psget.text | iex
```



安装后创建一个简单项目，依次执行

```shell
xmake create hello
cd hello
xmake
xmake run
```

如果要调试程序，改为

```shell
xmake config -m debug
xmake
xmake run -d hello
```



### 创建工程

#### 指定语言

通过 `-l` 参数指定

```shell
xmake create -l c hello
xmake create -l rust hello
```

通过 `-h` 查看帮助

```shell
xmake create -h
```



#### 指定目标类型

通过 `-t` 参数指定不同的项目类型

```shell
xmake create -t static hello	# 静态库
xmake create -t shared hello	# 动态库
```



#### 自定义模板

可以自定义模板。模板有两个来源

- 仓库模板：通过 `xmake repo` 添加到仓库的 `templates` 目录
- 全局模板：位于 `~/.xmake/templates`

列出和使用模板

```shell
xmake create --list
xmake create --list -l c++
xmake create -t mytemplate hello
```



### 编译配置

通过 `f` 参数配置编译命令

```shell
xmake f --help
xmake f -p windows [-a x86|x64]
```

清除配置

```shell
xmake f -c
```

还可以导出配置

```
xmake f --export=/tmp/config.txt
xmake f -m debug --xxx=y --export=/tmp/config.txt
```

导入配置

```
xmake f --import=/tmp/config.txt
xmake f -m debug --xxx=y --import=/tmp/config.txt
```

还可以通过 `--menu` 附带菜单

```shell
xmake f --menu --export=/tmp/config.txt
```



### 构建目标

#### 构建命令

使用 `xmake build` 构建目标，其中 `build` 一般会省略。只有当需要构建特定目标时需要全写

```shell
xmake build foo
```

使用 `-r` 或 `--rebuild` 重新构建

```shell
xmake -r
xmake --rebuild
```



#### 标注目标

目标通常都是默认构建的，除非它被标注为非默认

```lua
target("test")
	set_default(false)	-- 非默认
	add_files("src/*.c")
```

如果需要构建所有目标，使用 `-a` 或 `--all` 参数

```shell
xmake build -a
xmake build --all
```



#### 详细命令

如果需要查看详细的编译命令，使用

```shell
xmake -v
```



#### 调用堆栈

通过 `-vD` 参数查看 xmake 的函数调用堆栈，用于检查 xmake 中的配置问题

```lua
add_rules("mode.debug", "mode.release")

target("foo")
    set_kind("static")
    add_files("src/foo.cpp")

target("test")
    set_kind("binary")
    dd_deps("foo")   --------------- 不正确的接口
    add_files("src/main.cpp")
```

```shell
xmake -vD
error: @programdir/core/main.lua:329: @programdir/core/sandbox/modules/import/core/base/task.lua:65: @progr
amdir/core/project/project.lua:1050: ./xmake.lua:9: attempt to call a nil value (global 'dd_deps')
stack traceback:
    [./xmake.lua:9]: in main chunk  ----------------- 实际配置错误的地方

stack traceback:
        [C]: in function 'error'
        @programdir/core/base/os.lua:1075: in function 'os.raiselevel'
        (...tail calls...)
        @programdir/core/main.lua:329: in upvalue 'cotask'
        @programdir/core/base/scheduler.lua:406: in function <@programdir/core/base/scheduler.lua:399>
```



### 运行目标

#### 运行命令

使用 `xmake run` 运行目标，其中 `run` 一般会省略。只有当需要运行特定目标时需要全写

```shell
xmake run foo
```



#### 标注目标

目标通常都是默认运行的，除非它被标注为非默认

```lua
target("test")
	set_default(false)	-- 非默认
	add_files("src/*.c")
```

如果需要运行所有目标，使用 `-a` 或 `--all` 参数

```shell
xmake run -a
xmake run --all
```



#### 传递参数

还可以给内部程序传递参数

```shell
xmake run foo --arg1=xxx --arg2=yyy
```



#### 工作目录

默认工作目录是可执行文件所在的目录，通过 `-w workdir` 参数指定工作目录

```
xmake run -w /tmp foo
```



#### 调试程序

通过 debug 模式编译运行程序

```
xmake f -m debug
xmake
xmake run -d hello
```



### 清理构建

通过 `xmake clean` 命令清理构建过程中生成的临时文件。



#### 清理目标

默认清理所有目标。

```
xmake clean
```

指定清理特定的目标

```
xmake clean test
```



#### 清理所有文件

清理所有编译模式、所有架构生成的文件

```
xmake clean -a
xmake clean --all
```



#### 清理配置

清理配置缓存（重新执行检测和配置）

```
xmake f -c
```



### 安装卸载

通过 `xmake install` 命令将构建目标程序和库安装到系统环境，通常用于 linux, unix, bsd 等系统环境下的安装。



#### 安装到系统目录

默认的安装目录在 `/usr/local` 下

```
xmake install
```



#### 安装到指定目录

指定需要安装的根目录

```
$ xmake install -o /tmp/usr
installing foo ..
installing foo to /tmp/usr ..
installing test ..
installing test to /tmp/usr ..
install ok!
ruki:test ruki$ tree /tmp/usr
/tmp/usr
├── bin
│   └── test
└── lib
    └── libfoo.a

2 directories, 2 files
```

也可以通过 `--bindir`, `--libdir`, `--includedir` 等参数，分别配置不同类型文件的安装目录

```
xmake install -o /tmp/usr --bindir=mybin --libdir=mylib
```



#### 卸载程序

通过 `xmake uninstall` 执行反向的卸载操作

```
xmake uninstall
xmake uninstall --installdir=/tmp/usr
```



### 生成 doxygen 文档

在工程目录下运行

```
xmake doxygen
```



### 打包程序

Xmake 主要提供三种打包方式，用于将目标程序进行对外分发。主要介绍 XPack 打包程序。



#### 生成安装包 (XPack)

通过 `xmake pack` 插件命令实现，对标 CMake 的 CPack 打包，可以提供各种系统安装包的打包，来实现对目标程序的分发

- Windows NSIS 二进制安装包
- Windows WIX 二进制安装包
- runself (shell) 自编译安装包
- `zip/tar.gz/tar.xz` 二进制包
- `zip/tar.gz/tar.xz` 源码包
- RPM 二进制安装包
- SRPM 源码安装包
- DEB 二进制安装包

例如生成 Windows NSIS 安装包

```
xmake pack -f nsis
```



## 运行测试

通过 `xmake test` 命令运行单元测试和测试用例，通过 `add_tests` 在需要测试的目标上配置测试用例，然后自动执行所有测试。即使目标被设置为 `set_default(false)`，xmake 在执行测试时仍会自动编译它，然后运行所有测试。



### 配置测试

使用 `add_tests` 为目标配置测试用例

```lua
add_rules("mode.debug", "mode.release")

for _, file in ipairs(os.files("src/test_*.cpp")) do
    local name = path.basename(file)
    target(name)
        set_kind("binary")
        set_default(false)
        add_files("src/" .. name .. ".cpp")
        add_tests("default")
        add_tests("args", {runargs = {"foo", "bar"}})
        add_tests("pass_output", {trim_output = true, runargs = "foo", pass_outputs = "hello foo"})
        add_tests("fail_output", {fail_outputs = {"hello2 .*", "hello xmake"}})
end
```



### 运行测试

执行 `xmake test` 运行所有配置的测试用例

```
$ xmake test
running tests ...
[  2%]: test_1/args        .................................... passed 7.000s
[  5%]: test_1/default     .................................... passed 5.000s
[  8%]: test_1/fail_output .................................... passed 5.000s
[ 11%]: test_1/pass_output .................................... passed 6.000s
[ 13%]: test_2/args        .................................... passed 7.000s
[ 16%]: test_2/default     .................................... passed 6.000s
[ 19%]: test_2/fail_output .................................... passed 6.000s
[ 22%]: test_2/pass_output .................................... passed 6.000s
[ 25%]: test_3/args        .................................... passed 7.000s
[ 27%]: test_3/default     .................................... passed 7.000s
[ 30%]: test_3/fail_output .................................... passed 6.000s
[ 33%]: test_3/pass_output .................................... passed 6.000s
[ 36%]: test_4/args        .................................... passed 6.000s
[ 38%]: test_4/default     .................................... passed 6.000s
[ 41%]: test_4/fail_output .................................... passed 5.000s
[ 44%]: test_4/pass_output .................................... passed 6.000s
[ 47%]: test_5/args        .................................... passed 5.000s
[ 50%]: test_5/default     .................................... passed 6.000s
[ 52%]: test_5/fail_output .................................... failed 6.000s
[ 55%]: test_5/pass_output .................................... failed 5.000s
[ 58%]: test_6/args        .................................... passed 7.000s
[ 61%]: test_6/default     .................................... passed 6.000s
[ 63%]: test_6/fail_output .................................... passed 6.000s
[ 66%]: test_6/pass_output .................................... passed 6.000s
[ 69%]: test_7/args        .................................... failed 6.000s
[ 72%]: test_7/default     .................................... failed 7.000s
[ 75%]: test_7/fail_output .................................... failed 6.000s
[ 77%]: test_7/pass_output .................................... failed 5.000s
[ 80%]: test_8/args        .................................... passed 7.000s
[ 83%]: test_8/default     .................................... passed 6.000s
[ 86%]: test_8/fail_output .................................... passed 6.000s
[ 88%]: test_8/pass_output .................................... failed 5.000s
[ 91%]: test_9/args        .................................... passed 6.000s
[ 94%]: test_9/default     .................................... passed 6.000s
[ 97%]: test_9/fail_output .................................... passed 6.000s
[100%]: test_9/pass_output .................................... passed 6.000s

80% tests passed, 7 tests failed out of 36, spent 0.242s
```



### 测试目标

运行特定的测试

```
xmake test targetname/testname
```

通过模式匹配运行一个目标的所有测试或批量测试

```
xmake test targetname/*
xmake test targetname/foo*
```

运行所有目标中同名的测试

```
xmake test */testname
```



### 配置选项

`add_tests` 支持以下配置参数

|参数|描述|
|---|---|
|`runargs`|测试运行参数字符串或数组|
|`runenvs`|测试运行环境变量表|
|`timeout`|测试超时时间（秒）|
|`trim_output`|是否修剪输出空白字符|
|`pass_outputs`|测试通过的预期输出模式|
|`fail_outputs`|应导致测试失败的输出模式|
|`build_should_pass`|测试应该构建成功|
|`build_should_fail`|测试应该构建失败|
|`files`|额外的测试文件编译|
|`defines`|测试编译的额外定义|
|`realtime_output`|实时显示测试输出|



### 配置示例

#### 带参数的测试

```lua
target("mytest")
    set_kind("binary")
    add_files("src/*.cpp")
    add_tests("with_args", {runargs = {"--verbose", "--mode=test"}})
```

相关 API

- `set_runargs`: [设置目标运行参数](https://xmake.io/zh/api/description/project-target.html#set-runargs)
- `set_rundir`: [设置运行工作目录](https://xmake.io/zh/api/description/project-target.html#set-rundir)
- `add_runenvs`: [添加运行环境变量](https://xmake.io/zh/api/description/project-target.html#add-runenvs)



#### 带预期输出的测试

```lua
target("mytest")
    set_kind("binary")
    add_files("src/*.cpp")
    add_tests("output_test", {pass_outputs = ".*success.*", trim_output = true})
```



#### 带超时的测试

```lua
target("mytest")
    set_kind("binary")
    add_files("src/*.cpp")
    add_tests("timeout_test", {timeout = 5}) -- 5秒超时
```



#### 带环境变量的测试

```lua
target("mytest")
    set_kind("binary")
    add_files("src/*.cpp")
    add_tests("env_test", {runenvs = {TEST_MODE = "1", DEBUG = "true"}})
```

相关 API

- `add_runenvs`: [添加运行环境变量](https://xmake.io/zh/api/description/project-target.html#add-runenvs)
- `set_runenv`: [设置运行环境变量](https://xmake.io/zh/api/description/project-target.html#set-runenv)



#### 构建失败测试

```lua
target("compile_fail_test")
    set_kind("binary")
    add_files("src/invalid.cpp")
    add_tests("should_fail", {build_should_fail = true})
```

相关 API

- `set_default`: [设置目标默认构建](https://xmake.io/zh/api/description/project-target.html#set-default)
- `set_kind`: [设置目标类型](https://xmake.io/zh/api/description/project-target.html#set-kind)



#### 添加额外代码文件

`add_tests` 支持通过 `files` 参数添加额外的代码文件进行编译

```lua
target("mytest")
    set_kind("binary")
    add_files("src/*.cpp")
    add_tests("test_with_stub", {
        files = "tests/stub_*.cpp",  -- 添加额外的测试文件
        defines = "TEST_MODE",
        remove_files = "src/main.cpp"  -- 移除不需要的文件
    })
```

使用场景

- **单元测试**：添加测试代码文件而不修改原始源码
- **桩代码**：为测试提供模拟的实现
- **条件编译**：通过 `defines` 添加测试特定的宏定义
- **文件替换**：使用 `remove_files` 移除不需要的文件（如 `main.cpp`）

以 doctest 为例，可以在不修改任何 `main.cpp` 的情况下外置单元测试

```lua
target("doctest")
    set_kind("binary")
    add_files("src/*.cpp")
    for _, testfile in ipairs(os.files("tests/*.cpp")) do
        add_tests(path.basename(testfile), {
            files = testfile,
            remove_files = "src/main.cpp",
            languages = "c++11",
            defines = "DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN"
        })
    end
end
```



### 第三方测试框架

`xmake test` 可以很好地集成 doctest、gtest 等第三方测试框架，实现更强大的测试功能。



#### gtest

```lua
add_rules("mode.debug", "mode.release")
add_requires("gtest")

target("mytest")
    set_kind("binary")
    add_files("src/*.cpp")
    for _, testfile in ipairs(os.files("tests/*.cpp")) do
        add_tests(path.basename(testfile), {
            files = testfile,
            remove_files = "src/main.cpp",
            packages = "gtest",  -- 集成 gtest 包
            defines = "TEST_MAIN"
        })
    end
```

框架优势

- **doctest**：轻量级，仅头文件，易于集成
- **gtest**：功能丰富，Google 维护，社区活跃
- **Catch2**：现代 C++ 风格，功能强大



#### 动态库测试

也可以对动态库进行测试

```lua
target("mylib")
    set_kind("shared")
    add_files("src/*.cpp")

target("mylib_test")
    set_kind("binary")
    add_deps("mylib")
    add_files("tests/*.cpp")
    add_packages("gtest")
    add_tests("default")

```

使用 `-v` 获取详细输出

```sh
xmake test -v
```


使用 `-vD` 查看详细的测试失败错误信息

```
xmake test -vD
```

`-vD` 参数会显示更详细的日志输出，包括每个测试的输出文件路径和错误信息

```
report of tests:
[  2%]: test_10/compile_fail .... passed 0.001s
[  4%]: test_11/compile_pass .... failed 0.001s
errors: build/.gens/test_11/macosx/x86_64/release/tests/test_11/compile_pass.errors.log
[  7%]: test_1/args ............. passed 0.045s
stdout: build/.gens/test_1/macosx/x86_64/release/tests/test_1/args.stdout.log
[  9%]: test_1/default .......... passed 0.046s
stdout: build/.gens/test_1/macosx/x86_64/release/tests/test_1/default.stdout.log
[ 11%]: test_1/fail_output ...... passed 0.046s
stdout: build/.gens/test_1/macosx/x86_64/release/tests/test_1/fail_output.stdout.log
[ 14%]: test_1/pass_output ...... passed 0.047s
stdout: build/.gens/test_1/macosx/x86_64/release/tests/test_1/pass_output.stdout.log
...
[100%]: test_timeout/run_timeout  failed 1.007s
errors: build/.gens/test_timeout/macosx/x86_64/release/tests/test_timeout/run_timeout.errors.log

Detailed summary:
Failed tests:
 - test_11/compile_pass
 - test_5/fail_output
 - test_5/pass_output
 - test_7/args
 - test_7/default
 - test_7/fail_output
 - test_7/pass_output
 - test_8/pass_output
 - test_timeout/run_timeout

78% tests passed, 9 test(s) failed out of 42, spent 1.212s
```



#### 日志文件说明

使用 `-vD` 时，xmake 会生成以下日志文件

- **stdout 日志**：`build/.gens/{target}/tests/{testname}.stdout.log` - 测试的标准输出
- **errors 日志**：`build/.gens/{target}/tests/{testname}.errors.log` - 测试的错误输出
- **构建错误日志**：编译失败时的详细错误信息

这些日志文件对于调试测试失败问题非常有用，可以查看具体的输出内容和错误信息。



### 分组测试

使用模式匹配对测试进行分组

```
# 运行所有以 "unit_" 开头的测试
xmake test */unit_*

# 运行所有以 "test_" 开头的目标的测试
xmake test test_*/*

# 运行所有目标中的特定测试
xmake test */default
```



### 持续集成

对于 CI/CD 环境，可以使用以下模式

```
# 运行测试，失败时返回错误码
xmake test

# 运行测试并提供详细输出用于 CI 日志
xmake test -v

# 运行特定测试组
xmake test unit_tests/*
xmake test integration_tests/*
```

测试命令在任何测试失败时都会返回非零退出码，这使其适合用于 CI 流水线。



## 语法描述

### 工程配置

基本上写个简单的工程构建描述，只需三行就能搞定，例如

```lua
target("test")
    set_kind("binary")
    add_files("src/*.c")
```



### 配置分离

Xmake 采用二八原则实现了描述域、脚本域两层分离式配置

- 80% 的情况下，使用基础的常规配置
- 20% 的配置通常比较复杂，为了避免污染 `xmake.lua`，需要隔离配置



#### 描述域

描述域形如

```lua
target("test")
    set_kind("binary")
    add_files("src/*.c")
    add_defines("DEBUG")
    add_syslinks("pthread")
```

支持条件判断和循环

```lua
target("test")
    set_kind("binary")
    add_files("src/*.c")
    add_defines("DEBUG")
    if is_plat("linux", "macosx") then
        add_links("pthread", "m", "dl")
    end
```

```lua
target("test")
    set_kind("binary")
    add_files("src/*.c")
    add_defines("DEBUG")
    for _, name in ipairs({"pthread", "m", "dl"}) do
        add_links(name)
    end
```

> >描述域虽然支持 lua 的脚本语法，但在描述域尽量不要写太复杂，比如一些耗时的函数调用和循环。由于 `xmake.lua` 会被解析多次，因此也不要用 `print` 显示信息。



#### 脚本域

形如 `on_xxx`, `after_xxx`, `before_xxx` 内部的脚本，都属于脚本域。脚本域用于做更加复杂的配置逻辑

```lua
target("test")
    set_kind("binary")
    add_files("src/*.c")
    on_load(function (target)
        if is_plat("linux", "macosx") then
            target:add("links", "pthread", "m", "dl")
        end
    end)
    after_build(function (target)
        import("core.project.config")
        local targetfile = target:targetfile()
        os.cp(targetfile, path.join(config.buildir(), path.filename(targetfile)))
        print("build %s", targetfile)
    end)
```



在脚本域中，用户可以干任何事。Xmake 提供 import 接口导入内置的 lua 模块，也可以导入用户提供的 lua 脚本。如果需要实现更加复杂的脚本，以把脚本分离到独立的 lu文件中去维护。例如

```lua
target("test")
    set_kind("binary")
    add_files("src/*.c")
    on_load("modules.test.load")
    on_install("modules.test.install")
```

其中自定义脚本位于 `modules/test/load.lua` 和 `modules/test/install.lua` 中。



### 配置类型

#### 配置域

目前提供的配置域有

- `target()`
- `option()`
- `task()`
- `package()`

每当一个配置域出现，上一个配置域自动结束。



#### 配置项

带有 `set_xxx` 和 `add_xxx` 字样的配置属于配置项，一个配置域里面可以设置多个配置项。



### 作用域

Xmake 的描述语法是按作用域划分的，主要分为

- 外部作用域
- 内部作用域
- 接口作用域

例如

```lua
-- 外部作用域
target("test")

    -- 外部作用域
    set_kind("binary")
    add_files("src/*.c")

    on_run(function ()
        -- 内部作用域
        end)

    after_package(function ()
        -- 内部作用域
        end)

-- 外部作用域
task("hello")

    -- 外部作用域
    on_run(function ()
        -- 内部作用域
        end)
```



#### 外部作用域

为了做到简洁、安全，在这个作用域内，很多 lua 内置 api 不开放。外部作用域开放的 lua 内置 api 有

- table
- string
- pairs
- ipairs
- print
- os

还有些辅助 api，例如

- dirs：扫描获取当前指定路径中的所有目录
- files：扫描获取当前指定路径中的所有文件
- format: 格式化字符串，`string.format` 的简写版本

变量定义、逻辑操作可以使用，例如通过 if 切换编译文件

```lua
target("test")
    set_kind("static")
    if is_plat("iphoneos") then
        add_files("src/test/ios/*.c")
    else
        add_files("src/test/*.c")
    end
```

变量定义分全局变量和局部变量，局部变量只对当前 `xmake.lua` 有效，不影响子 `xmake.lua`

```lua
-- 局部变量
local var1 = 0

-- 全局变量，影响所有之后 includes()
var2 = 1

includes("src")
```



#### 内部作用域

提供更加复杂、灵活的脚本支持，一般用于编写一些自定义脚本、插件开发、自定义task任务、自定义模块等等。例如

```lua
-- 自定义脚本
target("hello")
    after_build(function ()
        -- 内部作用域
        end)

-- 自定义任务、插件
task("hello")
    on_run(function ()
        -- 内部作用域
        end)
```

> >在内部作用域中，所有的调用都是启用异常捕获机制的，如果运行出错，会自动中断 xmake，并给出错误提示信息。



#### 切换作用域

在外部作用域中的所有描述api设置，本身也是有作用域之分的，在不同地方调用，影响范围也不相同，例如：

```lua
-- 全局根作用域，影响所有 target，包括 includes() 中的子工程 target 设置
add_defines("DEBUG")

-- 定义或者进入 demo 目标作用域（支持多次进入来追加设置）
target("demo")
    set_kind("shared")
    add_files("src/*.c")
    -- 当前 target 作用域，仅仅影响当前 target
    add_defines("DEBUG2")

-- 选项设置，仅支持局部设置，不受全局 api 设置所影响
option("test")
    -- 当前选项的局部作用域
    set_default(false)

-- 其他 target 设置，-DDEBUG 也会被设置上
target("demo2")
    set_kind("binary")
    add_files("src/*.c")

-- 重新进入 demo 目标作用域
target("demo")
    -- 追加宏定义，只对当前 demo 目标有效
    add_defines("DEBUG3")
```



### 作用域缩进

xmake 缩进，只是编写规范，用于更加清楚的区分作用域。



### 代码格式化

可以通过 `do end` 的写法来处理

```lua
target("bar") do
    set_kind("binary")
    add_files("src/*.cpp")
end

target("foo") do
    set_kind("binary")
    add_files("src/*.cpp")
end
```

这样，Lua LSP 就能把它作为标准的 lua 代码进行正确的格式化，是否需要这么做，看用户自己的需求。



### 多级配置

在描述域可以通过 `includes` 接口引入项目子目录下的 `xmake.lua` 配置。如下项目结构

```shell
projectdir
    - xmake.lua
    - src
      - xmake.lua
```

其中 `projectdir/xmake.lua` 内容：

```lua
add_defines("ROOT")

target("test1")
    set_kind("binary")
    add_files("src/*.c")
    add_defines("TEST1")

target("test2")
    set_kind("binary")
    add_files("src/*.c")
    add_defines("TEST2")

includes("src")	-- 引入子目录配置
```

其中 `src/xmake.lua` 内容

```lua
add_defines("ROOT2")

target("test3")
    set_kind("binary")
    add_files("src/*.c")
    add_defines("TEST3")
```



## 配置目标

### 默认配置

当创建空项目时，会得到最基础的配置

```lua
add_rules("mode.debug", "mode.release")

target("test")
    set_kind("binary")
    add_files("src/*.cpp")
```



### 宏定义

通过 `add_defines` 添加一个宏定义选项

```lua
target("test")
    set_kind("binary")
    add_files("src/*.cpp")
    add_defines("DEBUG")
```

在 test 目标作用域下配置了一个 `-DDEBUG` 的宏定义编译选项，对 test 这一个目标生效。



通过 `xmake -v` 命令验证配置是否生效

```
[ 23%]: cache compiling.release src/main.cpp
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -c -Q
unused-arguments -target x86_64-apple-macos15.2 -isysroot /Applications/Xcode.app/Contents/Develop
er/Platforms/MacOSX.platform/Developer/SDKs/MacOSX15.2.sdk -fvisibility=hidden -fvisibility-inline
s-hidden -O3 -DNDEBUG -o build/.objs/test/macosx/x86_64/release/src/main.cpp.o src/main.cpp
```



### 多个目标

在全局作用域中的配置对所有目标都生效

```lua
add_defines("DEBUG")

target("foo")
    set_kind("binary")
    add_files("src/*.cpp")

target("bar")
    set_kind("binary")
    add_files("src/*.cpp")
```



### 头文件搜索目录

```lua
add_includedirs("/tmp")
```



### 链接库搜索目录

```lua
add_linkdirs("/tmp")
```



### 链接库

```lua
add_links("foo")
```



### 系统链接库

`add_links` 通常用于链接用户生成的库，而 `add_syslinks` 可以添加系统库，它的链接顺序也相对靠后

```lua
add_links("foo")
add_syslinks("pthread", "dl")
```



### 目标类型

#### 设置目标类型

通过 `set_kind()` 可以设置不同的目标类型

```lua
target("app")
    set_kind("binary")      -- 可执行程序
    add_files("src/*.cpp")

target("lib")
    set_kind("static")      -- 静态库
    add_files("src/*.cpp")

target("dll")
    set_kind("shared")      -- 动态库
    add_files("src/*.cpp")

target("obj")
    set_kind("object")      -- 对象文件集合
    add_files("src/*.cpp")

target("header")
    set_kind("headeronly")  -- 纯头文件库
    add_headerfiles("include/*.h")
```



#### 目标类型说明

| 类型         | 描述     | 输出文件                 |
| ---------- | ------ | -------------------- |
| binary     | 可执行程序  | `app.exe, app`       |
| static     | 静态库    | `libapp.a, app.lib`  |
| shared     | 动态库    | `libapp.so, app.dll` |
| object     | 对象文件集合 | `*.o, *.obj`         |
| headeronly | 纯头文件库  | 无编译输出                |
| phony      | 虚拟目标   | 无输出，仅用于依赖管理          |



#### 虚拟目标 (phony)

虚拟目标不生成实际文件，仅用于管理依赖关系

```lua
target("test1")
    set_kind("binary")
    add_files("src/test1.cpp")

target("test2")
    set_kind("binary")
    add_files("src/test2.cpp")

target("all-tests")
    set_kind("phony")
    add_deps("test1", "test2")
```

执行 `xmake build all-tests` 会同时构建 test1 和 test2 。



### 配置目标依赖

通过 `add_deps` 接口配置两个目标程序间的依赖。通常可用于可执行程序依赖静态库（或者动态库）的场景

```lua
target("foo")
    set_kind("static")
    add_files("src/foo.cpp")

target("test")
    set_kind("binary")
    add_deps("foo")
    add_files("src/main.cpp")
```



### 目标文件配置

#### 添加源文件

```lua
target("app")
    add_files("src/*.cpp")           -- 添加所有 cpp 文件
    add_files("src/*.c")             -- 添加所有 c 文件
    add_files("src/*.asm")           -- 添加汇编文件
    add_files("src/*.m")             -- 添加 Objective-C 文件
    add_files("src/*.mm")            -- 添加 Objective-C++ 文件
```



#### 排除特定文件

```lua
target("app")
    add_files("src/*.cpp|test.cpp")  -- 排除 src 目录下测试文件
```



#### 添加头文件

```lua
target("header-lib")
    set_kind("headeronly")
    add_headerfiles("include/*.h")           -- 添加头文件
    add_headerfiles("include/*.hpp")         -- 添加 C++ 头文件
    add_headerfiles("include/*.h", {install = false})  -- 不安装的头文件
```



#### 添加安装文件

```lua
target("app")
    add_installfiles("assets/*.png", {prefixdir = "share/app"})  -- 安装资源文件
    add_installfiles("config/*.conf", {prefixdir = "etc"})       -- 安装配置文件
```



### 目标属性配置

#### 设置目标文件名

```lua
target("app")
    set_basename("myapp")  -- 生成的文件名为 myapp.exe
```



#### 设置目标目录

```lua
target("app")
    set_targetdir("bin")   -- 输出到 bin 目录
```



#### 设置安装目录

```lua
target("app")
    set_installdir("bin")  -- 安装到 bin 目录
```



#### 设置版本信息

```lua
target("app")
    set_version("1.0.0")   -- 设置版本号
```



### 目标可见性配置

#### 设置符号可见性

```lua
target("lib")
    set_kind("shared")
    set_symbols("hidden")  -- 隐藏符号，减少导出表大小
```



### 目标优化配置

#### 设置优化级别

```lua
target("app")
    set_optimize("fast")     -- 快速优化
    set_optimize("faster")   -- 更快速优化
    set_optimize("fastest")  -- 最快优化
    set_optimize("smallest") -- 大小优化
    set_optimize("none")     -- 无优化
```



#### 设置调试信息

```lua
target("app")
    set_symbols("debug")     -- 添加调试符号
    set_strip("debug")       -- 链接时去除调试符号
    set_strip("all")         -- 链接时去除所有符号
```



### 目标语言配置

#### 设置语言标准

```lua
target("app")
    set_languages("c++17")   -- 设置 C++ 标准
    set_languages("c11")     -- 设置 C 标准
```



#### 设置语言特性

```lua
target("app")
    set_languages("c++17", "c11")  -- 同时支持 C++17 和 C11
```



### 目标平台配置

#### 设置目标平台

```lua
target("app")
    set_plat("android")      -- 设置为 Android 平台
    set_arch("arm64-v8a")    -- 设置为 ARM64 架构
```



#### 条件配置

```lua
target("app")
    if is_plat("windows") then
        add_defines("WIN32")
        add_links("user32")
    elseif is_plat("linux") then
        add_defines("LINUX")
        add_links("pthread")
    end
```



### 目标选项配置

#### 关联选项

```lua
option("enable_gui")
    set_default(false)
    set_description("Enable GUI support")

target("app")
    set_options("enable_gui")  -- 关联选项
```



#### 条件选项

```lua
target("app")
    if has_config("enable_gui") then
        add_defines("GUI_ENABLED")
        add_links("gtk+-3.0")
    end
```



### 目标规则配置

#### 添加构建规则

```lua
target("app")
    add_rules("mode.debug", "mode.release")  -- 添加调试和发布模式
    add_rules("qt.widgetapp")                -- 添加 Qt 应用规则
    add_rules("wdk.driver")                  -- 添加 WDK 驱动规则
```



#### 自定义规则

```lua
rule("myrule")
    set_extensions(".my")
    on_build_file(function (target, sourcefile, opt)
        -- 自定义构建逻辑
    end)

target("app")
    add_rules("myrule")  -- 应用自定义规则
```



### 目标运行时配置

#### 设置运行时库

```lua
target("app")
    set_runtimes("MT")      -- 静态运行时 (MSVC)
    set_runtimes("MD")      -- 动态运行时 (MSVC)
```



#### 设置运行时路径

```lua
target("app")
    set_runtimes("MD")
    add_rpathdirs("$ORIGIN")  -- 设置相对路径查找
```



### 目标工具链配置

#### 设置工具链

```lua
target("app")
    set_toolset("clang")    -- 使用 Clang 工具链
    set_toolset("gcc")      -- 使用 GCC 工具链
    set_toolset("msvc")     -- 使用 MSVC 工具链
```



#### 设置编译器

```lua
target("app")
    set_toolset("cc", "clang")   -- 设置 C 编译器
    set_toolset("cxx", "clang++") -- 设置 C++ 编译器
```



### 目标分组配置

#### 设置目标分组

```lua
target("app")
    set_group("apps")       -- 设置分组为 apps

target("lib")
    set_group("libs")       -- 设置分组为 libs

target("test")
    set_group("tests")      -- 设置分组为 tests
```



### 目标默认配置

#### 设置默认目标

```lua
target("app")
    set_default(true)       -- 设为默认构建目标

target("test")
    set_default(false)      -- 不设为默认构建目标
```



#### 启用/禁用目标

```lua
target("app")
    set_enabled(true)       -- 启用目标

target("old")
    set_enabled(false)      -- 禁用目标
```



## 定义选项

### 自定义命令行选项

定义一个选项开关，用于控制内部的配置逻辑

```lua
option("tests")
	set_default(false)
	set_description("Enable Tests")

target("foo")
    set_kind("binary")
    add_files("src/*.cpp")
    if has_config("tests") then
        add_defines("TESTS")
    end
```

在命令行启用这个自定义的选项，使得 foo 目标编译时候自动加上 `-DTEST` 的宏定义

```shell
xmake f --tests=y
xmake
```



### 绑定选项到目标

直接使用 `add_options` 将选项绑定到指定的 target 。当选项被启用的时候，所有关联的设置都会自动设置到被绑定的目标中

```lua
option("tests")
    set_description("Enable Tests")
    set_default(false)
    add_defines("TEST")

target("foo")
    set_kind("binary")
    add_files("src/*.cpp")
    add_options("tests")
```



### 选项类型与常用接口

#### 选项类型

- **布尔型**：开关选项，常用于启用/禁用某特性
- **字符串型**：用于路径、模式等自定义值
- **多值型**：通过 `set_values` 提供可选项（配合菜单）



#### 常用接口

- `set_default(value)`：设置默认值（支持 bool 或 string）
- `set_showmenu(true/false)`：是否在 `xmake f --menu` 菜单中显示
- `set_description("desc")`：设置描述
- `set_values("a", "b", "c")`：设置可选值（菜单模式）
- `add_defines("FOO")`：启用时自动添加宏定义
- `add_links("bar")`：启用时自动链接库
- `add_cflags("-O2")`：启用时自动添加编译选项



### 选项依赖与条件控制

一个选项可以依赖其它选项决定是否启用

- `add_deps("otheropt")`：依赖其他选项，常用于 on_check/after_check 控制
- `before_check`/`on_check`/`after_check`：自定义检测逻辑，可动态启用/禁用选项

例如

```lua
option("feature_x")
    set_default(false)
    on_check(function (option)
        if has_config("feature_y") then
            option:enable(false)
        end
    end)
```



### 选项实例接口

在 `on_check`、`after_check` 等脚本中，可以通过 option 实例接口获取和设置选项状态：

- `option:name()` 获取选项名
- `option:value()` 获取选项值
- `option:enable(true/false)` 启用/禁用选项
- `option:enabled()` 判断选项是否启用
- `option:get("defines")` 获取配置值
- `option:set("defines", "FOO")` 设置配置值
- `option:add("links", "bar")` 追加配置值



### 选项与 target 的结合

- 通过 `add_options("opt1", "opt2")` 绑定选项到目标
- 选项启用时，相关配置会自动应用到目标
- 也可用 `has_config("opt")` 在 target 域内做条件判断



### 典型示例

#### 1. 布尔开关选项

```lua
option("enable_lto")
    set_default(false)
    set_showmenu(true)
    set_description("Enable LTO optimization")
    add_cflags("-flto")

target("foo")
    add_options("enable_lto")
```



#### 2. 路径/字符串选项

```lua
option("rootdir")
    set_default("/tmp/")
    set_showmenu(true)
    set_description("Set root directory")

target("foo")
    add_files("$(rootdir)/*.c")
```



#### 3. 多值菜单选项

```lua
option("arch")
    set_default("x86_64")
    set_showmenu(true)
    set_description("Select architecture")
    set_values("x86_64", "arm64", "mips")
```



## 添加依赖包

通过 `add_requires` 接口声明需要的依赖包，通过 `add_packages` 接口，将声明的包绑定到需要的编译目标

```lua
add_requires("tbox 1.6.*", "libpng ~1.16", "zlib")

target("foo")
    set_kind("binary")
    add_files("src/*.c")
    add_packages("tbox", "libpng")

target("bar")
    set_kind("binary")
    add_files("src/*.c")
    add_packages("zlib")
```

其中 `add_requires` 是全局接口，用于包的配置声明，Xmake 会根据声明的包来触发查找安装。



### API 详解

#### 指定包版本

```lua
add_requires("tbox 1.6.*", "libpng ~1.16", "zlib")
```



#### 可选包

```lua
add_requires("foo", {optional = true})
```



#### 禁用系统库

```lua
add_requires("foo", {system = false})
```



#### 指定别名

```lua
add_requires("foo", {alias = "myfoo"})
add_packages("myfoo")
```



#### 平台/架构限定

```lua
add_requires("foo", {plat = "windows", arch = "x64"})
```



#### 传递包配置参数

```lua
add_requires("tbox", {configs = {small = true}})
```



#### 传递给依赖包

```lua
add_requireconfs("spdlog.fmt", {configs = {header_only = true}})
```



### 包实例接口

在自定义规则、after_install 等脚本中可用

- `package:name()` 获取包名
- `package:version_str()` 获取包版本
- `package:installdir()` 获取包安装目录
- `package:get("links")` 获取链接库
- `package:get("includedirs")` 获取头文件目录



### 典型示例

#### 1. 依赖可选包

```lua
add_requires("foo", {optional = true})
target("bar")
    add_packages("foo")
```



#### 2. 依赖指定分支/commit

```lua
add_requires("tbox master")
add_requires("zlib 1.2.11")
```



#### 3. 传递参数给包

```lua
add_requires("spdlog", {configs = {header_only = true}})
```



#### 4. 依赖本地包

1. 在工程目录下新建本地包仓库目录（如 `local-repo/packages/foo/xmake.lua`）。
2. 在 `xmake.lua` 中添加本地仓库：

```lua
add_repositories("myrepo local-repo")
add_requires("foo")
```

3. 本地包描述文件结构示例：

```lua
local-repo/
  packages/
    foo/
      xmake.lua
```

4. 这样即可像官方包一样通过 `add_requires("foo")` 使用本地包。



## 插件和任务

### 基础概念

- **任务 (Task)**: 自定义的构建步骤或工具，可以在项目中调用
- **插件 (Plugin)**: 特殊的任务，通常提供更复杂的功能，通过 `set_category("plugin")` 分类
- **菜单**: 通过 `set_menu()` 设置，让任务可以通过命令行直接调用



### 创建简单任务

#### 基本语法

```lua
task("taskname")
    on_run(function ()
        -- 任务执行逻辑
        print("任务执行中...")
    end)
```



#### 示例：Hello 任务

```lua
task("hello")
    on_run(function ()
        print("hello xmake!")
    end)
```

这个任务只能在 `xmake.lua` 中通过 `task.run()` 调用

```lua
target("test")
    after_build(function (target)
        import("core.project.task")
        task.run("hello")
    end)
```



### 创建命令行任务

#### 设置菜单

通过 `set_menu()` 可以让任务通过命令行直接调用

```lua
task("echo")
    on_run(function ()
        import("core.base.option")
        
        -- 获取参数内容并显示
        local contents = option.get("contents") or {}
        local color = option.get("color") or "black"
        
        cprint("${%s}%s", color, table.concat(contents, " "))
    end)
    
    set_menu {
        usage = "xmake echo [options]",
        description = "显示指定信息",
        options = {
            {'c', "color", "kv", "black", "设置输出颜色"},
            {nil, "contents", "vs", nil, "要显示的内容"}
        }
    }
```

现在可以通过命令行调用

```shell
xmake echo -c red hello xmake!
```



### 任务分类

#### 设置任务分类

```lua
task("myplugin")
    set_category("plugin")  -- 分类为插件
    on_run(function ()
        print("这是一个插件")
    end)
```

分类说明

- **plugin**: 显示在 "Plugins" 分组中
- **action**: 内置任务默认分类
- **自定义**: 可以设置任意分类名称



### 任务参数处理

#### 参数类型

```lua
task("example")
    on_run(function ()
        import("core.base.option")
        
        -- 获取不同类型的参数
        local verbose = option.get("verbose")        -- 布尔值
        local color = option.get("color")            -- 键值对
        local files = option.get("files")            -- 多值参数
        local args = {...}                           -- 可变参数
    end)
    
    set_menu {
        options = {
            {'v', "verbose", "k", nil, "启用详细输出"},           -- 布尔选项
            {'c', "color", "kv", "red", "设置颜色"},              -- 键值选项
            {'f', "files", "vs", nil, "文件列表"},                -- 多值选项
            {nil, "args", "vs", nil, "其他参数"}                  -- 可变参数
        }
    }
```



#### 参数类型说明

- **k**: key-only，布尔值参数
- **kv**: key-value，键值对参数
- **v**: value，单值参数
- **vs**: values，多值参数



### 在项目中使用任务

#### 构建后执行任务

```lua
target("test")
    set_kind("binary")
    add_files("src/*.cpp")
    
    after_build(function (target)
        import("core.project.task")
        
        -- 构建完成后运行代码生成任务
        task.run("generate-code")
        
        -- 运行测试任务
        task.run("run-tests")
    end)
```



#### 自定义构建任务

```lua
-- 代码生成任务
task("generate-code")
    on_run(function ()
        print("生成代码...")
        -- 执行代码生成逻辑
        os.exec("protoc --cpp_out=src proto/*.proto")
    end)

-- 测试任务
task("run-tests")
    on_run(function ()
        print("运行测试...")
        os.exec("xmake run test")
    end)
```



#### 文件处理任务

```lua
task("process-assets")
    on_run(function ()
        import("core.base.option")
        
        local input_dir = option.get("input") or "assets"
        local output_dir = option.get("output") or "build/assets"
        
        -- 处理资源文件
        os.mkdir(output_dir)
        os.cp(path.join(input_dir, "*.png"), output_dir)
        os.cp(path.join(input_dir, "*.json"), output_dir)
        
        print("资源文件处理完成")
    end)
    
    set_menu {
        usage = "xmake process-assets [options]",
        description = "处理项目资源文件",
        options = {
            {'i', "input", "kv", "assets", "输入目录"},
            {'o', "output", "kv", "build/assets", "输出目录"}
        }
    }
```



### 复杂任务示例

#### 示例 1：代码格式化任务

```lua
task("format")
    on_run(function ()
        import("core.base.option")
        import("lib.detect.find_tool")
        
        local tool = find_tool("clang-format")
        if not tool then
            raise("clang-format not found!")
        end
        
        local files = option.get("files") or {"src/**/*.cpp", "src/**/*.h"}
        for _, pattern in ipairs(files) do
            local filelist = os.files(pattern)
            for _, file in ipairs(filelist) do
                os.execv(tool.program, {"-i", file})
                print("格式化文件:", file)
            end
        end
    end)
    
    set_menu {
        usage = "xmake format [options]",
        description = "格式化代码文件",
        options = {
            {'f', "files", "vs", nil, "要格式化的文件模式"}
        }
    }
```



#### 示例 2：项目清理任务

```lua
task("clean-all")
    on_run(function ()
        local patterns = {
            "build/**",
            "*.log",
            "*.tmp",
            "*.o",
            "*.a",
            "*.so",
            "*.dylib",
            "*.exe"
        }
        
        for _, pattern in ipairs(patterns) do
            os.tryrm(pattern)
        end
        
        print("项目清理完成")
    end)
    
    set_menu {
        usage = "xmake clean-all",
        description = "清理所有构建文件和临时文件"
    }
```



### 任务调用方式

#### 1. 命令行调用

```shell
xmake taskname [options] [args...]
```



#### 2. 脚本中调用

```lua
import("core.project.task")

-- 调用任务
task.run("taskname")

-- 传递参数
task.run("taskname", {option1 = "value1"}, "arg1", "arg2")
```



#### 3. 在构建流程中调用

```lua
target("test")
    before_build(function (target)
        task.run("prepare")
    end)
    
    after_build(function (target)
        task.run("post-process")
    end)
```

