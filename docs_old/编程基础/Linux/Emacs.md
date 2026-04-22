# Emacs

## 基本介绍

### 安装步骤

使用 Spacemacs 来配置 [emacs](https://www.gnu.org/software/emacs/download.html#gnu-linux) 获得良好的编辑体验。首先需要安装 `emacs 27.1` 版本及以上

```shell
sudo add-apt-repository ppa:kelleyk/emacs -y
sudo apt-get update
```

然后在新立得包管理器中安装。安装最新版本的 emacs，添加库

```shell
sudo add-apt-repository ppa:ubuntuhandbook1/emacs
```

然后在新立得包管理器中安装。只需要一行就可以安装 spacemacs

```sh
git clone https://github.com/syl20bnr/spacemacs $HOME/.emacs.d
```

现在打开 emacs，注意预先清除家目录下的 `.emacs` 和 `.emacs.d` 配置。打开 emacs 后，会检查家目录下是否有 `.spacemacs` 文件，如果有就会直接使用其中的配置；如果没有，就会进行提问，根据回答进行配置。




如果安装 emacs 时出现错误

```shell
dpkg: 处理归档 /var/cache/apt/archives/emacs-common_1%3a29.3+1-0build1~focal_all.deb (--unpack)时出错：
```

则强制进行覆盖，然后修复

```shell
sudo dpkg -i --force-overwrite /var/cache/apt/archives/emacs-common_1%3a29.3+1-0build1~focal_all.deb
sudo apt-get -f install
```



### 配置字体

配置完成后需要重启。如果出现字体问题，可以到[开源字体库](https://github.com/adobe-fonts/source-code-pro.git)将源码克隆，然后

```shell
sudo cp /path/to/font-file.ttf /usr/share/fonts/
```

将 ttf 字体复制到系统字体目录下。例如下载 Nerd 字体库中的 `Ubuntu Nerd font`，解压后移动到字体文件夹下。

```embed
title: "Nerd Fonts - Iconic font aggregator, glyphs/icons collection, & fonts patcher"
image: "https://www.nerdfonts.com/assets/img/sankey-glyphs-combined-diagram.png"
description: "Iconic font aggregator, collection, & patcher: 9,000+ glyph/icons, 60+ patched fonts: Hack, Source Code Pro, more. Popular glyph collections: Font Awesome, Octicons, Material Design Icons, and more"
url: "https://www.nerdfonts.com/font-downloads"
```

命令行步骤为

```shell
unzip Ubuntu.zip
sudo mv *.ttf /usr/local/share/fonts/
fc-list
fc-cache -fv
```

如果想要查询字体名称，例如 `DejaVuSansM` 字体，可以输入

```shell
fc-list | grep DejaVuSansM*
```

会得到所有字体文件，例如

```shell
/usr/local/share/fonts/DejaVuSansMNerdFontMono-Regular.ttf: DejaVuSansM Nerd Font Mono:style=Regular
```

其中 `.ttf:` 后面就是字体名称。



### 配置图标

如果出现图标错乱的问题，需要安装对应的 icon 。执行 `M-x all-the-icons-install-fonts` 等待安装完成，重新启动即可。

> 对于自己配置的 emacs, 使用 `M-x nerd-icons-install-fonts` 解决所有乱码问题。



### 编辑风格

第一次打开 spacemacs 时，会引导创建配置文件 `.spacemacs` 。如果希望修改编辑风格，可以修改其中的

```lisp
dotspacemacs-editing-style 'vim
```

可选风格有 `vim, emacs, hybrid`，最后一种表示混合风格。



### 签名过期

当出现签名过期，`Failed to verify signatures ...` 错误，应该更新签名。首先在 `.spacemacs` 配置文件中找到 `dotspacemacs/user-init` 函数，在最开头添加 `setq package-check-signature nil)`，然后进入 emacs 安装插件。过程中可能提示缺少 `libtool`，退出安装

```shell
sudo apt install libtool
```

然后重新进入 emacs 完成安装。输入 `M-x package-install` 回车，再输入 `gnu-elpa-keyring-update` 更新签名。最后删除配置文件中的 `setq package-check-signature nil)`，重新进入 emacs 即可。



### 错误提示

* 如果提示某个配置在 layers 外面，这时候定位到该配置，将其移动到 layers 里面即可。如果提示变量未赋值，同样定位到该变量，赋值即可。
* 如果出现 `symbol's value as variable is void: ispell-menu-map-needed` 错误，将 `/etc/emacs/site-start.d/50dictionaries-common.el` 移除即可。



### 模式与钩子

在 emacs 中存在模式概念，例如 C++ 代码编写时进入 `c++-mode` 。模式分为主模式和次模式

* 主模式根据 Buffer 的文件类型选择，每个 Buffer 只能对应一个主模式
* 同一个 Buffer 可以有多个次模式，用于进一步调整、增加配置

每个主模式对应一个 Mode hook，当启动一个主模式时，自动直接挂钩到此主模式的函数或次模式。例如

```lisp
(add-hook 'text-mode-hook 'flyspell-mode)
```

将检查拼写的次模式挂钩到文本模式。再例如

```lisp
(add-hook 'prog-mode-hook #'hs-minor-mode)
```

将代码折叠功能挂钩到编程模式。



### 快捷键

在 Spacemacs 中，vim 模式通过 Space 来唤起快捷键提示；emacs 模式通过 `M-x` 唤起快捷键提示。Vim 模式下实际上也可以使用 `M-x` 来执行命令。



#### 键位映射

安装 `gnome-tweaks`，然后命令行运行，设置 Alt 行为和 Ctrl 的位置，重启即可。



#### 移动操作

| 快捷键     | 作用          | 快捷键   | 作用        |
| ------- | ----------- | ----- | --------- |
| `C-a`   | 行首          | `C-e` | 行尾        |
| `C-b`   | 后退一个字符      | `C-f` | 前进一个字符    |
| `C-n`   | 下一行         | `C-p` | 上一行       |
| `C-l`   | 页面移动使光标位于中央 | `C-v` | 向下滚屏      |
| `M-g g` | 前往指定行       | `M-r` | 光标移动到屏幕中央 |
| `M-v`   | 向上滚屏        |       |           |
按下 `C-u` 或按住 `M` 然后按数字，再按其它快捷键就可以重复执行快捷键。



#### 编辑操作

| 快捷键       | 作用                             | 快捷键       | 作用         |
| --------- | ------------------------------ | --------- | ---------- |
| `C-x h`   | 全选                             | `C-x C-s` | 储存文件       |
| `C-x C-c` | 退出 emacs                       | `C-g`     | 中断命令       |
| `C-z`     | 挂起 emacs                       | `C-/`     | 撤销         |
| `M-w`     | 复制                             | `C-w`     | 剪切         |
| `C-y`     | 粘贴                             | `M-y`     | 召回上一次移除的内容 |
| `M-;`     | 注释                             | `C-d`     | 删除后一个字符    |
| `M-d`     | 删除后一个词                         | `C-k`     | 删除光标后的内容   |
| `C-s`     | 搜索当前位置以后                       | `C-r`     | 搜索当前位置之前   |
| `M-up`    | 选中内容上移                         | `M-down`  | 选中内容下移     |
| `M-%`     | 替换（ y 确认替换  n 跳过本处的替换  ! 全部替换） |           |            |

安装 `multiple-cursors` 后，可以首先用 `C-space` 标记位置，然后向下移动几行，再通过快捷键 `M-m` 开启多行输入。输入完成后，通过 `C-g` 退出多行输入。



#### 文件操作
| 快捷键       | 作用         | 快捷键       | 作用             |
| --------- | ---------- | --------- | -------------- |
| `C-x C-s` | 储存文件       | `C-x C-f` | 打开当前目录下的文件     |
| `C-x C-r` | 只读打开文件     | `C-x C-v` | 打开当前文件所在目录下的文件 |
| `C-x C-j` | 打开当前文件所在目录 | `C-x s`   | 保存所有缓冲区        |



#### 文档操作

| 快捷键     | 作用                 | 快捷键     | 作用               |
| ------- | ------------------ | ------- | ---------------- |
| `C-h k` | 可以输入一个快捷键，查看其对应的函数 | `C-h f` | 查看一个 lisp 函数的源代码 |
| `C-h t` | 查看导航               | `C-h ?` | 查看帮助             |
| `C-h v` | 查看变量               |         |                  |



#### 窗口操作

| 快捷键     | 作用                           | 快捷键     | 作用      |
| ------- | ---------------------------- | ------- | ------- |
| `C-x 0` | 关闭当前窗口                       | `C-x 1` | 只保留当前窗口 |
| `C-x 2` | 上下拆分                         | `C-x 3` | 左右拆分    |
| `C-x 4` | 在新窗口执行操作（`C-x 4 f` 在新窗口打开文件） | `C-x o` | 切换窗口    |



#### 缓冲区操作

| 快捷键       | 作用        | 快捷键     | 作用          |
| --------- | --------- | ------- | ----------- |
| `C-x C-b` | 列出缓冲区     | `C-x k` | 杀死当前 Buffer |
| `C-x b`   | 切换 buffer | `M-k`   | 关闭当前 buffer |



## 配置文件

### 文件位置

默认情况下按照顺序

1. `.emacs`
2. `~/.emacs.el`
3. `.emacs.d/init.el`

寻找配置文件。



### 基础配置

#### 工具栏

在 `.emacs` 中添加配置

```lisp
;; 关闭菜单、工具、滚动条
(menu-bar-mode -1)
(tool-bar-mode -1) 
(scroll-bar-mode -1)

;; 关闭启动欢迎栏
(setq inhibit-startup-screen t)
```



#### 文字编码

设置文字编码

```lisp
(prefer-coding-system 'utf-8) 
(unless *is-windows* 
	(set-selection-coding-system 'utf-8))
```



#### 垃圾回收

设置垃圾回收

```lisp
(setq gc-cons-threshold most-positive-fixnum)
```



#### 简化输入

未保存文件退出时，会提示输入 `yes` 或 `no`，可以简化输入

```lisp
(defalias 'yes-or-no-p 'y-or-n-p)
```



#### 显示行号

```lisp
(add-hook 'prog-mode-hook 'display-line-numbers-mode)
```



### 软件源

进入清华镜像站，搜索 `elpa`，点击问号进入帮助界面

```embed
title: "elpa | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror"
image: "https://mirrors.tuna.tsinghua.edu.cn/static/img/logo-share.png"
description: "elpa 使用帮助 | 镜像站使用帮助 | 清华大学开源软件镜像站，致力于为国内和校内用户提供高质量的开源软件镜像、Linux 镜像源服务，帮助用户更方便地获取开源软件。本镜像站由清华大学 TUNA 协会负责运行维护。"
url: "https://mirrors.tuna.tsinghua.edu.cn/help/elpa/"
```

将配置内容复制到配置文件中即可使用

```lisp
(setq package-archives '(("gnu"    . "http://mirrors.tuna.tsinghua.edu.cn/elpa/gnu/")
                         ("nongnu" . "http://mirrors.tuna.tsinghua.edu.cn/elpa/nongnu/")
                         ("melpa"  . "http://mirrors.tuna.tsinghua.edu.cn/elpa/melpa/")))
(package-initialize) ;; You might already have this line
```



### 扩展管理

首先安装扩展管理工具 `use-pacakge` 用来管理软件包

```lisp
;;个别时候会出现签名检验失败 
(setq package-check-signature nil) 

;; 初始化软件包管理器 
(require 'package) 

(unless (bound-and-true-p package--initialized)
		(package-initialize)) 

;; 刷新软件源索引 
(unless package-archive-contents (package-refresh-contents)) 

;; 第一个扩展插件：use-package，用来批量统一管理软件包 
(unless (package-installed-p 'use-package) 
		(package-refresh-contents) 
		(package-install 'use-package))
```

基本配置

```lisp
;; 确认包已经安装，启动延迟加载（使用时再加载）
(setq use-package-always-ensure t 
	  use-package-always-defer t 
	  use-package-enable-imenu-support t 
	  use-package-expand-minimally t)
```



### 主题

使用一个好看的主体，并配置智能模式

```lisp
(use-package gruvbox-theme 
			 :init (load-theme 'gruvbox-dark-soft t))
(use-package smart-mode-line 
			 :init (setq sml/no-confirm-load-theme t 
			 			sml/theme 'respectful) 
			 (sml/setup))
```



### 启动配置

使用 `benchmark-init` 包测试启动耗时

```lisp
(use-package benchmark-init 
			 :init (benchmark-init/activate) 
			 :hook (after-init . benchmark-init/deactivate))
```

然后可以执行命令

```shell
M-x benchmark-init/show-durations-tree
M-x benchmark-init/show-durations-tabulated
```

查看不同组件的耗时。



### sudo

当使用 `sudo` 打开 emacs 时可能不会加载配置文件，这就需要将配置文件复制到根目录下

```shell
sudo cp ~/.emacs /root/
sudo cp -r ~/.emacs.d /root/
```



### ace-window

使用 `ace-window` 提高分屏操作效率

```lisp
(use-package ace-window 
			 :bind (("M-o" . 'ace-window)))
```

可以按下 `M-o`，再按数字键切换窗口。也可以按 `M-o` 后选择操作键

| 操作  | 作用         | 操作  | 作用      |
| :-- | :--------- | :-- | :------ |
| `x` | 删除窗口       | `m` | 交换窗口    |
| `M` | 移动窗口       | `c` | 复制窗口    |
| `j` | 选择缓冲       | `n` | 选择上一个窗口 |
| `u` | 选择其它窗口中的缓冲 | `c` | 分割窗口    |
| `v` | 垂直分割       | `b` | 水平分割    |
| `o` | 最大化当前窗口    | `?` | 显示命令提示  |
最后按数字键对指定窗口执行操作。



### Neotree

配置 neotree 插件

```lisp
;; 安装并配置 all-the-icons
(use-package all-the-icons
  :ensure t)

;; 安装字体 (需要手动安装字体文件，按提示操作)
(all-the-icons-install-fonts)

;; 安装并配置 neotree
(use-package neotree
  :ensure t
  :bind (("C-x t" . neotree-toggle))  ;; 绑定快捷键
  :config
  ;; 配置主题
  (setq neo-theme (if (display-graphic-p) 'icons 'arrow))
  ;; 设置 Neotree 在项目根目录打开
  (setq neo-smart-open t)
  ;; 每次打开 Neotree 自动刷新
  (setq neo-autorefresh t)
  ;; 隐藏隐藏文件
  (setq-default neo-show-hidden-files nil)
  ;; 设置 Neotree 宽度
  (setq neo-window-width 30))
```

重新进入 emacs 后，首先连接外网下载图标，然后退出外网完成安装。重新进入时又会提示安装字体，现在注释掉安装命令即可。



### Vterm

Vterm 是一个可以嵌入 emacs 的终端，需要先通过 apt 安装

```cpp
sudo apt install libvterm-dev
```

然后配置

```lisp
(use-package vterm
			 :bind ("M-s" . 'vterm))
```



### Consult

Consult-imenu 可用于获得文件中所有函数的列表

```lisp
(use-package consult
	:bind ("M-l" . 'consult-imenu))
```

如果出现安装 `Not Found`，可以先执行命令

```lisp
package-refresh-contents
```

然后重启安装。



Consult 可以配合 ripgrep 实现强化搜索。首先安装

```shell
sudo apt install ripgrep
rg --version
```



### Treesit

Treesit 提供现代且高效的语法高亮效果。首先安装

```lisp
(use-package treesit-auto 
			 :demand t 
			 :config (setq treesit-auto-install 'prompt) 
			 (setq treesit-font-lock-level 4)
			 (global-treesit-auto-mode))
```

然后进入 emacs 执行 `M-x treesit-auto-install-all`，将会安装所有可能的语言的高亮代码。有一部分语言会安装失败，可以忽略。



## 配置环境

### Org

在配置文件中开启 org 模板

```lisp
(require 'org-tempo)
```

然后在 org 文件中可以使用模板：输入 `<s`，然后 tab 键（要取消补全），就会得到一个代码段

```org
#+begin_src cpp

#+end_src
```

可以在里面写代码。使用快捷键 `C-c '` 打开一个单独的 buffer 用于编写代码，重复使用快捷键退出 buffer 并将内容填写到代码段。



### eglot

使用 eglot 配置 IDE 环境

```lisp
(use-package eglot 
			 :hook ((c-mode 
			 		c++-mode 
			 		go-mode 
			 		java-mode 
			 		js-mode 
			 		python-mode 
			 		rust-mode 
			 		web-mode) . eglot-ensure) 
			 :bind (("C-c e f" . #'eglot-format) 
			 		("C-c e a" . #'eglot-code-actions) 
			 		("C-c e i" . #'eglot-code-action-organize-imports) 
			 		("C-c e q" . #'eglot-code-action-quickfix)) 
			 :config 
			 ;; (setq eglot-ignored-server-capabilities '(:documentHighlightProvider)) 
			 (add-to-list 'eglot-server-programs '(web-mode "vls")) 
			 (defun eglot-actions-before-save() 
			 	(add-hook 'before-save-hook 
			 		(lambda () 
			 			(call-interactively #'eglot-format) 
			 			(call-interactively #'eglot-code-action-organize-imports)))) 
			 (add-hook 'eglot--managed-mode-hook #'eglot-actions-before-save))
```

还需要安装对应语言的服务。



### Python

安装最新版本

```embed
title: "Index of /ftp/python/"
image: "https://www.python.org/favicon.ico"
description: ""
url: "https://www.python.org/ftp/python/"
```

下载源码后解压

```shell
tar -xzf Python-3.11.9.tgz
cd Python-3.11.9
mkdir
../configure --enable-optimizations
make
sudo make install
```

然后安装 pyright 服务

```shell
sudo pip3 install pyright
```

或者使用 `snap` 安装

```shell
sudo snap install pyright --classic
```



### C++

#### Clang

首先需要安装 clangd 和 clang-format 用于编译和格式化

```shell
sudo apt install clang
sudo apt install clangd
sudo apt install clang-format
```

还需要 bear 为 clangd 生成配置文件

```shell
sudo apt-get install bear
```



如果 clangd 没有源，需要配置源

```shell
wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -
sudo apt-add-repository "deb http://apt.llvm.org/$(lsb_release -cs)/ llvm-toolchain-$(lsb_release -cs) main"
sudo apt update
sudo apt install clangd
sudo apt install clang
```



在安装 gcc/g++ 时，需要注意参考 clang 的版本

```shell
clang++ -v

# 找到 clang 选择的版本（它会选择最高的版本）
Selected GCC installation: /usr/bin/../lib/gcc/x86_64-linux-gnu/13
```

然后安装对应版本

```cpp
sudo apt install gcc-13
sudo apt install g++-13
```

安装 CMake 新版

```shell
cd cmake-3.30.2
mkdir build
../configure
make
sudo make install
cmake --version
```



#### gdb

下载最新的 [gdb-14.2.tar.xz](https://ftp.gnu.org/gnu/gdb/gdb-14.2.tar.xz) 版本

```embed
title: "Download GDB"
image: "https://www.sourceware.org/gdb/images/archer.svg"
description: "Please send FSF & GNU inquiries & questions to gnu@gnu.org. There are also other ways to contact the FSF."
url: "https://www.sourceware.org/gdb/download/"
```

依次执行

```shell
tar -xf gdb-14.2.tar.xz 
cd gdb-14.2
mkdir build
cd build
../configure
make
```

如果提示 `configure: error: Building GDB requires GMP 4.2+, and MPFR 3.1.0+.`，则需要先安装这两个库

```shell
sudo apt-get install libgmp-dev libmpfr-dev
```

如果执行 `make install` 就会覆盖原本的 gdb，因此需要根据情况使用。这里创建软链接

```shell
sudo mv ./gdb /opt/gdb
sudo ln -s /opt/gdb /usr/local/bin/gdb
gdb --version
```

这种方式是通过本地配置来覆盖系统配置，系统 gdb 位于 `/usr/bin/gdb`，它被 local 配置覆盖，但仍然存在。只要删除软链接就会还原

```shell
sudo rm /usr/local/bin/gdb
```



要执行调试，依次执行

1. `M-x compile`, 输入 `g++ -g -o test.o test.cpp`
2. `M-x gud-gdb` 输入 `gdb ./test.o`

现在可以使用调试指令。

| name              | function                                       |
| ----------------- | ---------------------------------------------- |
| `list`            | 显示源代码                                          |
| `break`           | 新增断点， `break main`, `break 12`（行号）             |
| `info`            | 查看断点或者局部变量信息 `info breakpoints`, `info locals` |
| `run`             | 开始调试                                           |
| `next`            | 类似 `step over`                                 |
| `step`            | 跳转到函数内部                                        |
| `continue`        | 继续运行到下一个断点                                     |
| `quit`            | 退出调试                                           |
| `watch`           | 内存断点                                           |
| `display`         | 类似 `IDE` 里面的 `watch` 功能                        |
| `break 11 if xxx` | 条件断点                                           |

也可以使用 realgud 提供鼠标交互打断点

```embed
title: "RealGUD"
image: ""
description: "An extensible, modular GNU Emacs front-end for interacting with external debuggers, brought to you by Rocky Bernstein and Clément Pit-Claudel."
url: "https://github.com/realgud/realgud"
```



### Rust

安装 Rust

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

检验安装

```shell
rustc --version
```

安装 rust-analyzer

```shell
rustup component add rust-analyzer
```



### HTML+CSS+JS

依次安装语言服务

```shell
sudo npm install -g typescript-language-server typescript
sudo npm install -g  vscode-html-languageserver
sudo npm install -g  vscode-css-languageserver
```

目前 29 版本的 eglot 似乎还有问题。



### 其它语言
#### CMake

通过 `M-x Package-Install` 安装 `cmake-mode` 即可提供语法高亮，然后还要安装 `cmake-language-server` 提供自动补全和检查

```shell
pip install cmake-language-server
```



#### Lua

通过 `M-x Package-Install` 安装 `lua-mode` 即可提供语法高亮。



#### Markdown

通过 `M-x Package-Install` 安装 `markdown-mode` 即可提供语法高亮。



## 基本语法

Emacs lisp 是使用 Lisp 为 Emacs 打造的方言，可以使用它为 Emacs 编写逻辑。



### Scratch

进入 emacs 后按 `q` 关闭欢迎界面，进入 `scratch` 缓冲区。每个缓冲区都具有 emacs 的内建命令，目前的 `scratch` 缓冲区则处于 `Lisp Interaction` 模式。在此模式下可以执行 Lisp 命令，例如

```lisp
(+ 2 2)
```

将光标移动到末尾，然后按 `C-j` 将结果插入到缓冲区。也可以使用 `C-x C-e` 将结果输出到底部提示区。



Lisp 是 List Processing 的缩写，其核心就是列表。每一对括号表达了一个列表，列表元素用空格分割。在执行时，会把列表的第一个元素作为函数名，后面的元素都是函数的参数。元素可以是一个词，也可以是另一个列表。例如

```lisp
(+ 2 3 4) ;; 2+3+3
```

这表示 `+` 函数以后面的 `2 3 4` 为参数；再例如

```lisp
(+ 4 (- 3 2)) ;; 4+(3-2)
```



### 变量

使用 `setq` 创建变量

```lisp
(setq my-name "Bastien")
```

执行 `insert` 将参数内容插入到光标位置

```lisp
(insert "Hello")
(insert "Hello" " world!")
(insert "Hello, I am " my-name)
```



### 函数

使用 `defun` 关键字定义函数。例如

```lisp
(defun ivy-set-prompt (caller prompt-fn)
	(setq ivy--prompts-list
      (plist-put ivy--prompts-list caller prompt-fn)))
```

这大致等价于

```cpp
void ivy_set_prompt(CallerType caller, FnType prompt_fn) 
{
    ivy__prompts_list = plist_put(ivy__prompts_list, caller, prompt_fn);
}
```



函数可以调用

```lisp
(defun hello (name) (insert "Hello " name))

(hello "you")
```



### 组合

可以组合多个函数

```lisp
(progn
	(switch-to-buffer-other-window "*test*")
	(hello "you"))

(progn
	(switch-to-buffer-other-window "*test*")
	(erase-buffer)
	(hello "you")
	(other-window 1))
```

使用 `let` 将值绑定到局部变量

```lisp
;; you 绑定到 local-name，并在后续命令 (hello local-name) 中使用
(let ((local-name "you"))
	(switch-to-buffer-other-window "*test*")
	(erase-buffer)
	(hello local-name)
	(other-window 1))
```

这里 `let` 也会将命令组合到一起，变量在 `let` 规定的作用域内使用。



### 格式化

使用 `format` 格式化字符串

```lisp
(format "Hello %s!\n" "visitor")
```

其中 `%s` 是字符串占位符。


### 列表

可以将一组值绑定到变量

```lisp
;; 注意 () 前面的 '
(setq list-of-names '("Sarah" "Chloe" "Mathilde"))

(car list-of-names)					;; 获得第一个元素
(cdr list-of-names)					;; 获得除了第一个元素的所有元素
(push "Stephanie" list-of-names)	;; 将元素添加到头部

;; 为每个元素执行 hello 函数
(mapcar 'hello list-of-names)
```

单引号表达了后面的元素不进行执行而直接返回它本身，例如

```lisp
'ivy-set-prompt
```

表示把 `ivy-set-prompt` 这个函数作为一个对象传递给其它部分，也没有执行这个函数。



### 条件

使用 `(while (x, y))` 构造条件句：如果 `x` 返回值，则对 `y` 求值。例如

```lisp
(defun replace-hello-by-bonjour ()
	(switch-to-buffer-other-window "*test*")
	(goto-char (point-min))
	(while (search-forward "Hello" nil t) ;; 传入 nil 表示搜索没有边界，传入 t 表示失败后静默
  		(replace-match "Bonjour"))
	(other-window 1))
```



### 搜索

改为使用正则搜索

```lisp
(defun boldify-names ()
	(switch-to-buffer-other-window "*test*")
	(goto-char (point-min))
	(while (re-search-forward "Bonjour \\(.+\\)!" nil t)
  		(add-text-properties (match-beginning 1)
                       (match-end 1)
                       (list 'face 'bold)))
	(other-window 1))
```



### 导出模块

可以编写一个 elisp 文件，其中定义一个函数

```lisp
;;; hello -- Echo "Hello, world!"
;;; Commentary:
;;; Code:

(defun hello-world ()
  (interactive)					; 将函数暴露给用户，允许显式调用
  (message "Hello, world!"))	; 输出字符串

(provide 'hello) ; 意为“导出本模块，名为 hello”。这样就可以在其它地方进行 require 

;;; hello.el ends here
```

在主配置文件中引入

```lisp
(require 'hello)
```

就可以通过 `M-x hello-world` 调用函数。
