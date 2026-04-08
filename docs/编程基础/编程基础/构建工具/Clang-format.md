# Clang-format

## 基本介绍

Clang-format 描述了一组基于 LibFormat 构建的工具。它可以通过多种方式支持您的工作流程，包括独立工具和编辑器集成。



### 安装

可以从 Clang 的官方 GitHub 网站下载 Release 版本。更简单的方法是通过 python 安装

```shell
(.venv) $ pip install clang-format=16.0.6
```

然后可以在 Scripts 文件夹下找到 `.exe` 程序。



### 导出

Clang 支持包括 `LLVM, GNU, Google, Chromium, Microsoft, Mozilla, WebKit` 多种不同的代码风格。可以将格式化配置导出为 YAML 文件

```shell
$ clang-format -style=llvm -dump-config > .clang-format
```

将 `.clang-format` 文件放在项目目录下，就可以启用格式化配置。



## [Format.cmake](https://github.com/TheLartians/Format.cmake)

使用 CPM 配合 `Format.cmake` 可以自动对所有 cpp 文件和 cmake 文件进行格式化。



首先在主目录下配置 `.clang-format` 文件

```shell
$ clang-format -style=microsoft -dump-config > .clang-format
```

在主目录的 CMake 文件中引入

```cmake
# 包含 CPM 脚本，添加 cmake-format 包
include("${CMAKE_SOURCE_DIR}/cmake/CPM.cmake")
cpmaddpackage(NAME cmake-format VERSION 1.7.0 SOURCE_DIR D:/lib/Format.cmake)
```

这里通过 CPM 包来导入这个库。然后就可以生成项目，分别**生成** format 和 fix-format 两个项目即可格式化所有文件。
