# NovelAI

## 安装流程

### 克隆源码

首先在 GitHub 克隆[源码](https://github.com/AUTOMATIC1111/stable-diffusion-webui)，然后点击 webui-user.bat 运行脚本。此时会下载安装需要的 Python 库，由于版本问题可能失败。过程中会生成 venv 文件夹作为虚拟环境，修改其中的 pyvenv.cfg 配置文件中的路径

```config
home = D:\Python310
```

然后重新运行脚本。如果报错找不到要求版本的 torch 库，则需要更新 pip 版本

```shell
E:\Desktop\gitee\stable-diffusion-webui\venv\Scripts\python.exe -m pip install --upgrade pip
```



当模型加载完成后命令行将会停止，可以等待一段时间发现命令行不动以后，关闭窗口。然后重新运行脚本，可以看到

![[image-20230824001638442.png]]

其中生成了本地 URL，复制链接打开即可。



### [gfpgan](https://github.com/TencentARC/GFPGAN)

如果 gfpgan 安装失败，首先启用虚拟环境，安装两个库

```shell
(venv) $ pip install basicsr facexlib
```

下载源码，进入 gfpgan 源码路径

```shell
(venv) $ cd E:\Desktop\GFPGAN-master
```

安装依赖环境，然后 setup 安装库

```shell
(venv) $ pip install -r requirements.txt
(venv) $ python setup.py develop
```



### [clip](https://github.com/pharmapsychotic/clip-interrogator)

同样下载源码，进入路径

```shell
(venv) $ cd E:\Desktop\clip-interrogator-main
```

安装所有环境

```shell
(venv) $ pip install -r requirements.txt
```

如果出现错误，找不到 Rust 编译器，那么先安装 Rust，然后执行

```shell
(venv) $ rustup default stable
```

获得默认的稳定 Rust 版本。有可能还需要安装

```shell
(venv) $ pip install setuptools-rust
```



现在执行 setup 安装

```shell
(venv) $ python setup.py build install
```



### stable-diffusion

如果克隆仓库到指定位置出错，例如

```shell
Cloning Stable Diffusion into E:\Desktop\gitee\stable-diffusion-webui\repositories\stable-diffusion-stability-ai...
```

错误提示

```shell
Command: "git" clone "https://github.com/Stability-AI/stablediffusion.git" "E:\Desktop\gitee\stable-diffusion-webui\repositories\stable-diffusion-stability-ai"
```

这是因为没有将 git 路径添加到环境变量，导致 cmd 不能运行对应的命令。

![[image-20230823230656461.png]]

如果出现超时错误，那么手动进行克隆

```shell
git clone https://github.com/Stability-AI/stablediffusion.git E:\Desktop\gitee\stable-diffusion-webui\repositories\stable-diffusion-stability-ai
```



### [generative-models](https://github.com/Stability-AI/generative-models.git)

如果正常克隆失败，改为使用 SSH 进行克隆

```shell
git clone git@github.com:Stability-AI/generative-models.git E:\Desktop\gitee\stable-diffusion-webui\repositoriesenerative-models
```

需要注意设置好 GitHub 的 SSH 公钥。



### [k-diffusion](https://github.com/crowsonkb/k-diffusion.git)

这个库也需要手动克隆

```shell
git clone https://github.com/crowsonkb/k-diffusion.git E:\Desktop\gitee\stable-diffusion-webui\repositories\k-diffusion
```



### Rust

首先需要添加两个环境变量 CARGO_HOME 和 RUSTUP_HOME 指定安装位置

![[image-20230823223503867.png]]

然后在 [Rust](https://www.rust-lang.org/tools/install) 官网下载 Windows-64bit 版本的 installer，双击运行，会显示安装目录

![[image-20230823223433856.png]]

输入 2 按回车进入自定义安装，接着一路回车，直到出现下面 3 个选项，输入 1 回车安装完成。

![[image-20230823223611079.png]]

在命令行输入

```shell
$ rustc --version
```

检查安装版本。



## 基本使用

点击 webui-user.bat 脚本运行，出现 URL 后将其复制后在浏览器打开即可看到 UI 界面。

![[image-20231207190116542.png]]

生成的图片自动保存在 outputs 目录下。注意最好不要挂梯子，否则可能不能生成图片。



### 汉化插件

点击 Extension 选项卡，点击 Available 选项卡，输入如下 URL 链接

```url
https://raw.githubusercontent.com/wiki/AUTOMATIC1111/stable-diffusion-webui/Extensions-index.md
```

或者使用原始链接也可以，两个都有点不稳定。使用

```sh
https://gitcode.net/rubble7343/sd-webui-extensions/raw/master/index.json
```

点击 Load from 即可加载插件。

![[image-20231207190351172.png]]

在搜索框输入 zh_CN 查找扩展并安装。在 Settings 选项卡左侧找到 User interface 选项，将 Localization 刷新后选项 zh_CN 后 Apply settings 然后 Reload UI，就可以得到中文界面。

![[image-20231207190808468.png]]



### xformers 组件

使用 xformers 加速图片生成。右键打开编辑 webui-user.bat，在命令行参数中添加 `--xformers` 参数

```bat
@echo off

set PYTHON=
set GIT=
set VENV_DIR=
set COMMANDLINE_ARGS=--xformers

call webui.bat
```

然后运行 webui-user.bat 即可自动安装。



### 网络部署

可以使用 `--share` 参数部署，这样会生成一个 72 小时的链接，可以访问。但是每次部署产生的链接都不同。

```bat
set COMMANDLINE_ARGS=--xformers --share
```

另一种方式是通过 `--listen` 部署，前提是具有公网 IP 。还需开放 8080 端口，因为一般云端都会开放 8080 端口的访问。

```bat
set COMMANDLINE_ARGS=--xformers --listen --port 8080
```



### 开启拓展

默认情况下会阻止拓展插件安装，需要添加参数

```bat
set COMMANDLINE_ARGS=--enable-insecure-extension-access
```



### 低显存优化

可以开启显存优化加速图片生成

```bat
set COMMANDLINE_ARGS=--medvram
```



### tag 反推

使用 deepdanbooru 来启用 tag 反推插件

```bat
set COMMANDLINE_ARGS=--deepdanbooru
```

然后在图生图功能中可以看到 Tag 反推功能，上传一张图片后，等待命令行中的安装进程完毕，之后就可以自由使用。



### tag 自动补全

通过 git 安装，在“扩展-从网址安装”输入项目路径（不太稳定）。直接在 extensions 文件夹下 git clone 就可以了。

```bat
https://github.com/DominikDoom/a1111-sd-webui-tagcomplete.git
```

需要在其中的 tags 文件夹下提供中文翻译文件，然后在设置自动补全中将标签文件设为 danbooru.csv，翻译文件设为该中文翻译文件。



### 修复模型

使用 adetailer 来修复变形的手部、脸部。由于插件会检查 huggingface，如果不挂梯子会影响启动时间。可以添加参数

```bat
set COMMANDLINE_ARGS=--ad-no-huggingface
```



### 教程优化

```embed
title: "粉毛女路人(换装人偶)的魔咒教室 · 语雀"
image: "https://cdn.nlark.com/yuque/0/2022/png/385283/1665588138232-26d2ca8d-082c-42c0-bc5b-a41d7927a7d1.png"
description: ""
url: "https://www.yuque.com/longyuye/lmgcwy/goa36x"
```


