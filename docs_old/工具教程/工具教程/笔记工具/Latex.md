# Latex

## 安装与配置

先[安装 texlive](https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/Images/)，打开 `.iso` 文件后，找到 install-tl 脚本文件执行安装程序。注意 texlive 安装时间很长，并且只有当安装界面出现图示表示安装完成才可以退出，否则可能会有文件缺失。



### texstudio

在 texstudio 中更改配置文件链接 texlive 的编译器如下

![](image-20220715112346525.png)



### vscode

在 vscode 中编译 latex，需要安装 latex workshop 插件。然后在设置 `settings.json` 中添加

```json
{
	// 增加编译命令，"%DOCFILE%" 表示文件路径可为中文
	"latex-workshop.latex.tools": [
	    {
	        "name": "pdflatex",
	        "command": "pdflatex",
	        "args": [
	            "-synctex=1",
	            "-interaction=nonstopmode",
	            "-file-line-error",
	            "%DOCFILE%"
	        ]
	    },
	    {
	        "name": "xelatex",
	        "command": "xelatex",
	        "args": [
	            "-synctex=1",
	            "-interaction=nonstopmode",
	            "-file-line-error",
	            "%DOCFILE%"
	        ]
	    },
	    {
	        "name": "biber",
	        "command": "biber",
	        "args": [
	            "%DOCFILE%"
	        ]
	    },
	    {
	        "name": "bibtex",
	        "command": "bibtex",
	        "args": [
	            "%DOCFILE%"
	        ]
	    }
	],
	// 定义编译顺序	
	"latex-workshop.latex.recipes": [
	    {
	        "name": "xelatex",
	        "tools": [
	            "xelatex"
	        ]
	    },
	    {
	        "name": "pdflatex",
	        "tools": [
	            "pdflatex"
	        ]
	    },
	    {
	        "name": "xe->biber->xe->xe",
	        "tools": [
	            "xelatex",
	            "biber",
	            "xelatex",
	            "xelatex"
	        ]
	    },
	    {
	        "name": "pdf->biber->pdf->pdf",
	        "tools": [
	            "pdflatex",
	            "biber",
	            "pdflatex",
	            "pdflatex"
	        ]
	    },
	    {
	        "name": "xe->bibtex->xe->xe",
	        "tools": [
	            "xelatex",
	            "bibtex",
	            "xelatex",
	            "xelatex"
	        ]
	    },
	    {
	        "name": "pdf->bibtex->pdf->pdf",
	        "tools": [
	            "pdflatex",
	            "bibtex",
	            "pdflatex",
	            "pdflatex"
	        ]
	    }
	],
	// 清除辅助文件格式	
	"latex-workshop.latex.clean.fileTypes": [
	    "*.aux",
	    "*.bbl",
	    "*.blg",
	    "*.idx",
	    "*.ind",
	    "*.lof",
	    "*.lot",
	    "*.out",
	    "*.toc",
	    "*.acn",
	    "*.acr",
	    "*.alg",
	    "*.glg",
	    "*.glo",
	    "*.gls",
	    "*.ist",
	    "*.fls",
	    "*.log",
	    "*.fdb_latexmk"
	],
	"latex-workshop.view.pdf.viewer": "tab",				// 使用内置浏览器
	"latex-workshop.latex.autoBuild.run": "onFileChange",	// 文件更改自动编译
	"latex-workshop.showContextMenu": true,					// 在右键菜单中显示
	"latex-workshop.message.error.show": false,				// 不显示错误
	"latex-workshop.message.warning.show": false,			// 不显示警告
	"latex-workshop.intellisense.package.enabled": true,	// 启用自动补全
	"latex-workshop.latex.autoClean.run": "never",			// 永不清除辅助文件
	"latex-workshop.latex.recipe.default": "lastUsed",		// 默认选择上一次使用的编译链
	"latex-workshop.view.pdf.internal.synctex.keybinding": "double-click",	// 反向定位
}
```

由于 json 文件不支持注释，需要去掉注释后添加。



### 入门手册

可以参考 [入门手册](Latex.assets/Latex-Guide.pdf) 查询基本指令。



### 通用模板

给出一个通用的模板，能够满足基本需求

```tex
\documentclass[twoside,a4paper]{article}
\usepackage{geometry}
\geometry{margin=1.5cm, vmargin={0pt,1cm}}
\setlength{\topmargin}{-1cm}
\setlength{\paperheight}{29.7cm}
\setlength{\textheight}{25.3cm}

% useful packages.
\usepackage[UTF8]{ctex}	% 中文支持
\usepackage{listings} 	% 插入代码
\usepackage{xcolor} 	% 语法高亮
\usepackage{amsfonts}
\usepackage{amsmath}
\usepackage{amssymb}
\usepackage{amsthm}
\usepackage{enumerate}
\usepackage{graphicx}
\usepackage{multicol}
\usepackage{fancyhdr}
\usepackage{layout}

% some common command
\newcommand{\dif}{\mathrm{d}}
\newcommand{\avg}[1]{\left\langle #1 \right\rangle}
\newcommand{\difFrac}[2]{\frac{\dif #1}{\dif #2}}
\newcommand{\pdfFrac}[2]{\frac{\partial #1}{\partial #2}}

\newcommand{\R}{\mathbb{R}}
\renewcommand{\proofname}{Proof.}
\newtheorem{theorem}{Theorem}[section]
\newtheorem{lemma}{Lemma}[section]
\newtheorem{exercise}{Exercise}[section]
\newtheorem{definition}{Definition}[section]

\lstset{
	columns=fixed,
	numbers=left,                                        % 在左侧显示行号
	numberstyle=\tiny\color{gray},                       % 设定行号格式
	basicstyle=\small,
	tabsize=4,
	frame=single,                                        % 背景边框
	rulecolor=\color[rgb]{0.8,0.8,0.8},                  % 边框颜色
	extendedchars=false,                                 % 解决换页问题
	backgroundcolor=\color[RGB]{245,245,244},            % 设定背景颜色
	keywordstyle=\color[RGB]{40,40,255},                 % 设定关键字颜色
	identifierstyle=\color{black},                       % 普通标识符颜色
	commentstyle=\it\color[RGB]{0,96,96},                % 设置代码注释的格式
	stringstyle=\rmfamily\slshape\color[RGB]{128,0,0},   % 设置字符串格式
	showstringspaces=false,                              % 不显示字符串中的空格
	language=matlab,                                     % 设置语言
	escapeinside=``,                                     % 逃逸字符
	xleftmargin=1em,                                     % 边距设置
	xrightmargin=1em,
	aboveskip=1em
}

\begin{document}
	
	\pagestyle{fancy}
	\fancyhead{}
	\lhead{left tiltle}
	\chead{center title}
	\rhead{\today}
	
\end{document}
```



## 文档元素

我们首先介绍一些基本的简单元素。



### Latex 中的字符

空格键和 Tab 键输入的空白字符视为“空格”。连续的若干个空白字符视为一个空格。一行开头的空格忽略不计。行末的换行符视为一个空格；但连续两个换行符，也就是空行，会将文字分段。多个空行被视为一个空行；也可以在行末使用 `\par` 命令分段。

```tex
连续两个换行符会将文字分段。例如

这是里开始另一段。也可以使用 par 命令进行分段\par
新的一段文字。
```



以下字符在 LATEX 里有特殊用途，如 `%` 表示注释，`$、^、_` 等用于排版数学公式，`&` 用于排版表格，等等。直接输入这些字符得不到对应的符号，还往往会出错：

```tex
# $ % & { } _ ^ ~ \
```

如果想要输入以上符号，需要使用以下带反斜线的形式输入，类似编程语言里的“转义”符号：

```tex
\# \$ \% \& \{ \} \_

\^{} \~{} \textbackslash

\# $ % & { } _ ^ ~ \
```

这些“转义”符号事实上是一些 LATEX 命令。其中 `^` 和 `~` 两个命令需要一个参数，加一对花括号的写法相当于提供了空的参数，否则它们可能会将后面的字符作为参数，形成重音效果。`\\` 被直接定义成了手动换行的命令，输入反斜线就需要用 `\textbackslash` 。



如果需要输出命令，使用 `\verb` 命令

```latex
\begin{document}
\verb|\textit|
\verb|\begin{*enviroment*}|
\end{document}
```

用 `|` 包围的内容将直接输出。



### 导入宏包

有时需要通过一些扩展来增强功能，就需要导入宏包

```tex
\usepackage[⟨options⟩]{⟨package-name⟩}
```

也可以同时导入多个包，但是不能指定选项

```tex
% 一次性调用三个排版表格常用的宏包
\usepackage{tabularx, makecell, multirow}
```



### 虚拟文本

可以向 latex 中插入虚拟的文本，用作占位符

```tex
\usepackage{lipsum}

\begin{document}
\lipsum[1]\\
\lipsum[2-3]
\end{document}
```

还有其它风格的虚拟文本，例如 `kantlipsum` 宏包

```tex
\documentclass[14pt]{memoir}

\usepackage{fontspec}
\setmainfont[Ligatures=TeX]{Segoe Print}

\usepackage{kantlipsum}

\begin{document}

\chapterstyle{weloveducks}

\setlength{\parskip}{1.5\baselineskip}
\setlength{\parindent}{0pt}
\OnehalfSpacing

\chapter{The journey begins}

Hi, I am a duck. Quack!

\kant[1]

\chapter{The journey continues}

\kant[2]

\chapter{The journey ends}

\kant[3]

\end{document}
```



### 排版表格

排版表格最基本的 tabular 环境用法为：

```tex
\begin{tabular}[⟨align⟩]{⟨column-spec⟩} 
    ⟨item1⟩ & ⟨item2⟩ & ... \\
    \hline
    ⟨item1⟩ & ⟨item2⟩ & ... \\
\end{tabular}
```

其中 `column-spec` 是列格式标记；`&` 用来分隔单元格；`\\` 用来换行；`\hline` 在行之间绘制横线。



直接使用 tabular 环境的话，会和周围的文字混排。此时可用一个可选参数 `align` 控制垂直对齐：`t` 和 `b` 分别表示按表格顶部、底部对齐，其他参数或省略不写（默认）表示居中对齐

```latex
\begin{document}
\begin{tabular}{|c|}
	center-\\ aligned \\
\end{tabular},

\begin{tabular}[t]{|c|}
	top-\\ aligned \\
\end{tabular},

\begin{tabular}[b]{|c|}
	bottom-\\ aligned\\
\end{tabular} tabulars.
\end{document}
```

但是通常情况下 tabular 环境很少与文字直接混排，而是会放在 table 浮动体环境中，并用 `\caption` 命令加标题。



#### 列格式

tabular 环境使用 `column-spec` 参数指定表格的列数以及每列的格式。基本的列格式见表

| 列格式         | 说明                    |
| ----------- | --------------------- |
| `l/c/r`     | 单元格内容左对齐、居中、右对齐，不折行   |
| `p{width}`  | 单元格宽度固定为 width ，可自动折行 |
| \|          | 绘制竖线                  |
| `@{string}` | 自定义内容 string          |

在 `column-spec` 参数位置直接使用

```latex
\begin{document}

\begin{tabular}{lcr|p{6em}}
    \hline
    left & center & right
    & par box with fixed width\\
    L & C & R & P \\
    \hline
\end{tabular}

\end{document}
```

使用 `@` 格式可在单元格前后插入任意的文本，但同时它也消除了单元格前后额外添加的间距。使用 `@` 格式可以充当“竖线”。使用 `@{}` 可消除单元格前后的间距

```latex
\begin{document}

\begin{tabular}{@{} r@{:}lr @{}}
    \hline
    1 & 1 & one \\
    11 & 3 & eleven \\
    \hline
\end{tabular}

\end{document}
```



另外 LATEX 还提供了简便的将格式参数重复的写法 `*{⟨n⟩}{⟨column-spec⟩}`，比如以下两种写法等价

```tex
\begin{tabular}{|c|c|c|c|c|p{4em}|p{4em}|}
...
\end{tabular}

\begin{tabular}{|*{5}{c|}*{2}{p{4em}}}
...
\end{tabular}
```

有时需要为整列修饰格式，比如整列改变为粗体，如果每个单元格都加上 `\bfseries` 命令会比较麻烦。array 宏包提供了辅助格式 `>` 和 `<`，用于给列格式前后加上修饰命令

```tikz
\usepackage{array}

\begin{document}

\begin{tabular}{>{\itshape}r<{*}l}
    \hline
    italic & normal \\
    column & column \\
    \hline
\end{tabular}

\end{document}

```

支持插入 `\centering` 等命令改变 `p` 列格式的对齐方式，还要加额外的命令 `\arraybackslash` 以免出错

```latex
\usepackage{array}

\begin{document}
\begin{tabular}%
{>{\centering\arraybackslash}p{9em}}
    \hline
    Some center-aligned long text. \\
    \hline
\end{tabular}
\end{document}
```



array 宏包提供类似 `p` 格式的 `m` 格式和 `b` 格式，分别在垂直方向上靠顶端对齐、居中以及底端对齐

```latex
\usepackage{array}
\newcommand\txt{a b c d e f g h i}

\begin{document}
\begin{tabular}{cp{2em}m{2em}b{2em}}
    \hline
    pos & \txt & \txt & \txt \\
    \hline
\end{tabular}
\end{document}
```



#### 横线

我们已经在之前的例子见过许多次绘制表格线的 `\hline` 命令。另外 `\cline{⟨i⟩-⟨j⟩}` 用来绘制跨越部分单元格的横线

```latex
\begin{document}

\begin{tabular}{|c|c|c|}
    \hline
    4 & 9 & 2 \\ \cline{2-3}
    3 & 5 & 7 \\ \cline{1-1}
    8 & 1 & 6 \\ \hline
\end{tabular}

\end{document}
```

广泛应用的表格形式是三线表，由 booktabs 宏包支持，提供 `\toprule, \midrule, \bottomrule` 命令用以排版三线表的三条线，以及和 `\cline` 对应的 `\cmidrule` 命令。除此之外，最好不要用其它横线以及竖线

```latex
\usepackage{booktabs}

\begin{document}
\begin{tabular}{cccc}
    \toprule
    & \multicolumn{3}{c}{Numbers} \\
    \cmidrule{2-4}
    & 1 & 2 & 3 \\
    \midrule
    Alphabet & A & B & C \\
    Roman & I & II& III \\
    \bottomrule
\end{tabular}
\end{document}
```



#### 斜线表头

使用 diagbox 宏包可以在表格中添加斜线

```latex
\usepackage{diagbox}

\begin{document}

\begin{tabular}{|c|c|c|c|}
	\hline
	\diagbox{N}{Error}{T} & 5 & 10 & 20\\
	\hline
	20 & 0.669878 & 0.296885 & 2.86198 \\
	40 & 2.02443 & 4.15128 & 3.354e+009 \\
	80 & 11.2192 & 4.80466e+009 & 4.12673e+027\\
	\hline
\end{tabular}

\end{document}
```



#### 合并单元格

LATEX 是一行一行排版表格的，横向合并单元格较为容易，由 `\multicolumn` 命令实现

```tex
\multicolumn{⟨n⟩}{⟨column-spec⟩}{⟨item⟩}
```

其中 `n` 为要合并的列数，`column-spec` 为合并单元格后的列格式，只允许出现一个 `l/c/r` 或 `p` 格式。如果合并前的单元格前后带表格线|，合并后的列格式也要带 | 以使得表格的竖线一致

```latex
\begin{document}

\begin{tabular}{|c|c|c|}
    \hline
    1 & 2 & Center \\ \hline
    \multicolumn{2}{|c|}{3} &
    \multicolumn{1}{r|}{Right} \\ 
    \hline
    4 & \multicolumn{2}{c|}{C} \\ 
    \hline
\end{tabular}

\end{document}
```

上面的例子还体现了，形如 `\multicolumn{1}{⟨column-spec⟩}{⟨item⟩}` 的命令可以用来修改某一个单元格的列格式。纵向合并单元格需要用到 multirow 宏包提供的 `\multirow` 命令

```tex
\multirow{⟨n⟩}{⟨width⟩}{⟨item⟩} 
```

参数 `width` 为合并后单元格的宽度，可以填 `*` 以使用自然宽度。例如

```latex
\usepackage{multirow}

\begin{document}

\begin{tabular}{ccc}
    \hline
    \multirow{2}{*}{Item} &
    \multicolumn{2}{c}{Value} \\
    \cline{2-3}
    & First & Second \\ 
    \hline
    A & 1 & 2 \\ 
    \hline
\end{tabular}

\end{document}
```



#### 嵌套表格

相对于合并单元格，拆分单元格对于 LATEX 来说并非易事。在单元格中嵌套一个小表格可以起到“拆分单元格”的效果。在以下的例子中，注意要用 `\multicolumn` 命令配合 `@{}` 格式把单元格的额外边距去掉，使得嵌套的表格线能和外层的表格线正确相连

```latex
\begin{document}

\begin{tabular}{|c|c|c|}
    \hline
    a & b & c \\ \hline
    a & \multicolumn{1}{@{}c@{}|}
        {\begin{tabular}{c|c}
        e & f \\ \hline
        e & f \\
        \end{tabular}}
    & c \\ \hline
    a & b & c \\ \hline
\end{tabular}

\end{document}
```

如果不需要为“拆分的单元格”画线，并且只在垂直方向“拆分”的话，makecell 宏包提供的 `\makecell` 命令是一个简单的解决方案

```latex
\usepackage{makecell}

\begin{document}

\begin{tabular}{|c|c|}
    \hline
    a & \makecell{d1 \\ d2} \\
    \hline
    b & c \\
    \hline
\end{tabular}

\end{document}
```



#### 行距控制

LATEX 生成的表格看起来通常比较紧凑。修改参数 `\arraystretch` 可以得到行距更加宽松的表格

```latex
\renewcommand\arraystretch{1.8}

\begin{document}

\begin{tabular}{|c|}
    \hline
    Really loose \\ \hline
    tabular rows.\\ \hline
\end{tabular}

\end{document}
```



另一种增加间距的办法是给换行命令 `\\` 添加可选参数，适合用于在行间不加横线的表格

```latex
\begin{document}

\begin{tabular}{c}
    \hline
    Head lines \\[6pt]
    tabular lines \\
    tabular lines \\ \hline
\end{tabular}

\end{document}
```

但是这种换行方式的存在导致了一个缺陷——从第二行开始，表格的首个单元格不能直接使用中括号，否则 `\\` 会将下一行的中括号当作自己的可选参数，因而出错。如果要使用中括号，应当放在花括号 `{}` 里面，或者也可以选择将换行命令写成 `\\[0pt]` 填充可选参数。



### 排版图片

#### 加载图片

在调用了 graphicx 宏包以后，就可以使用 `\includegraphics` 命令加载图片

```tex
\includegraphics[⟨options⟩]{⟨filename⟩}
```

其中 filename 为图片文件名，与 `\include` 命令的用法类似，文件名可能需要用相对路径或绝对路径表示。图片文件的扩展名一般可不写。

> > 文件名里不要有空格（类似 `\include`），也不要有多余的英文点号，否则宏包在解析文件名的过程中会出错。



另外 graphicx 宏包还提供了 `\graphicspath` 命令，用于声明一个或多个图片文件存放的目录，使用这些目录里的图片时可不用写路径

```tex
% 假设主要的图片放在 figures 子目录下，标志放在 logo 子目录下
\graphicspath{{figures/}{logo/}}
```

在 `\includegraphics` 命令的可选参数 `options` 中可以使用 `key=value` 的形式，常用参数如下

| 参数                | 含义                      |
| ----------------- | ----------------------- |
| `width=⟨width⟩`   | 将图片缩放到宽度为 `⟨width⟩`     |
| `height=⟨height⟩` | 将图片缩放到高度为 `⟨height⟩`    |
| `scale=⟨scale⟩`   | 将图片相对于原尺寸缩放 `⟨scale⟩` 倍 |
| `angle=⟨angle⟩`   | 将图片逆时针旋转 `⟨angle⟩` 度    |

也支持 `draft/final` 选项。当 graphicx 宏包或文档类指定 `draft` 选项时，图片将不会被实际插入，取而代之的是一个包含文件名的与原图片等大的方框。



#### 单张图片

排版一张图片的基本格式如下

```tex
\begin{figure}
    \centering
    \includegraphics[width=0.5\linewidth]{figure/picture/4.jpg}
    \caption{左图：半径为 $ r $ 的闭球包围的点集；右图：生成的 Cech 复形 $ C(i) $ 。}
    \label{pic4}
\end{figure}
```



#### 图片并排

在一个浮动体里面放置多张图的用法就是直接并排放置，也可以通过分段或者换行命令 `\\` 排版多行多列的图片

```tex
\begin{figure}[htbp]
    \centering
    \includegraphics[width=...]{...}
    \qquad
    \includegraphics[width=...]{...} \\[...pt]
    \includegraphics[width=...]{...}
    \caption{...}
\end{figure}
```

由于标题是横跨一行的，用 `\caption` 命令为每个图片单独生成标题就需要借助 `\parbox` 或者 minipage 环境，将标题限制在盒子内

```tex
\begin{figure}[htbp]
    \centering
    \begin{minipage}{...}
        \centering
        \includegraphics[width=...]{...}
        \caption{...}
    \end{minipage}
	\qquad
    \begin{minipage}{...}
        \centering
        \includegraphics[width=...]{...}
        \caption{...}
    \end{minipage}
\end{figure}
```

当我们需要更进一步，给每个图片定义小标题时，就要用到 `subcaption` 宏包的功能

```tex
\begin{figure}[htbp]
    \centering
        \begin{subfigure}{...}
        \centering
        \includegraphics[width=...]{...}
        \caption{...}
    \end{subfigure}
    \qquad
    \begin{subfigure}{...}
        \centering
        \includegraphics[width=...]{...}
        \caption{...}
    \end{subfigure}
\end{figure}
```

> > 不要使用 `subfigure` 宏包，过于古老且会与 `subcaption` 宏包冲突，导致报错（无法指定 `subfigure` 宽度）。



### 浮动体

内容丰富的文章或者书籍往往包含许多图片和表格等内容。这些内容的尺寸往往太大，导致分页困难。LATEX 为此引入了浮动体的机制，令大块的内容可以脱离上下文，放置在合适的位置。LATEX 预定义了 figure 和 table 两类浮动体环境。

> 习惯上 figure 里放图片，table 里放表格。

以 table 环境的用法举例，figure 同理：

```tex
\begin{table}[⟨placement⟩] 
...
\end{table}
```

参数 `placement` 提供了一些符号用来表示浮动体允许排版的位置，如 `hbp` 允许浮动体排版在当前位置、底部或者单独成页。

> table 和 figure 浮动体的默认设置为 `tbp` 。 



###### 浮动位置

排版位置的选取与参数里符号的顺序无关，LATEX 总是以 `h-t-b-p` 的优先级顺序决定浮动体位置，也就是说 `[!htp]` 和 `[ph!t]` 没有区别。

| 参数  | 含义             |
| --- | -------------- |
| h   | 当前位置（代码所处的上下文） |
| t   | 顶部             |
| b   | 底部             |
| p   | 单独成页           |
| !   | 在决定位置时忽视限制     |



###### 跨栏排版

双栏排版环境下，LATEX 提供了 `table*` 和 `figure*` 环境用来排版跨栏的浮动体。用法与 table 和 figure 一样，不同之处为双栏的 placement 参数只能用 `tp` 两个位置。



###### 添加标题

图表等浮动体提供了 \caption 命令加标题，并且自动给浮动体编号

```tex
\begin{figure}[⟨placement⟩] 
\caption{title}
\end{figure}
```

在 `\caption` 命令之后可以紧跟 `\label` 命令标记交叉引用。



Table 和 figure 两种浮动体分别有各自的生成目录的命令

```tex
\listoftables
\listoffigures
```

它们类似 `\tableofcontents` 生成单独的章节。



### 交叉引用

交叉引用是 LATEX 强大的自动排版功能的体现之一。在能够被交叉引用的地方，如章节、公式、图表、定理等位置使用 `\label` 命令：

```tex
\label{⟨label-name⟩}
```

之后可以在别处使用 `\ref` 或 `\pageref` 命令，分别生成交叉引用的编号和页码：

```tex
\ref{⟨label-name⟩} \pageref{⟨label-name⟩}
```

为了生成正确的交叉引用，一般也需要多次编译源代码

```tex
\section{A}
	著名的欧拉公式是
	\begin{equation}\label{sec:this}
		e^{i\pi} + 1 = 0
	\end{equation}
	我们引用一下 section \ref{sec:this} 在 page \pageref{sec:this} 。
```



标签命令使用位置主要包括

| 位置   | 用法                                                    |
| :--- | :---------------------------------------------------- |
| 章节标题 | 在章节标题命令 `\section` 等之后紧接着使用                           |
| 行间公式 | 单行公式在公式内任意位置使用；多行公式在每一行公式的任意位置使用                      |
| 有序列表 | 在 enumerate 环境的每个 `\item` 命令之后、下一个 `\item` 命令之前任意位置使用 |
| 图表标题 | 在图表标题命令 `\caption` 之后紧接着使用                            |
| 定理环境 | 在定理环境内部任意位置使用                                         |

在使用不记编号的命令形式（`\section*`、`\caption*`、带可选参数的 `\item` 命令等）时不要使用 `\label` 命令，否则生成的引用编号不正确。



### 脚注和边注

使用 `\footnote` 命令可以在页面底部生成一个脚注

```tex
\footnote{⟨footnote⟩}
```

例如引用文章内容

```tex
“天地玄黄，宇宙洪荒。日月盈昃，辰宿列张。”\footnote{出自《千字文》。}
```

有些情况下（比如在表格环境、各种盒子内）使用 `\footnote` 并不能正确生成脚注。我们可以分两步进行，先使用 `\footnotemark` 为脚注计数，再在合适的位置用 `\footnotetext` 生成脚注。例如

```tex
\begin{tabular}{l}
\hline
“天地玄黄，宇宙洪荒。日月盈昃，辰宿列张。”\footnotemark \\
\hline
\end{tabular}
\footnotetext{表格里的名句出自《千字文》。}
```

使用 `\marginpar` 命令可在边栏位置生成边注：

```tex
\marginpar[⟨left-margin⟩]{⟨right-margin⟩}
```

如果只给定了 `right-margin`，那么边注在奇偶数页文字相同；如果同时给定了 `left-margin`，则偶数页使用 `left-margin` 的文字

```tex
\marginpar{\footnotesize 边注较窄，不要写过多文字，最好设置较小的字号。}
```



## [文档结构](https://www.overleaf.com/latex/examples)

### 中文排版

ctex 是最常用的排版中文的宏包，源代码须保存为 UTF-8 编码，并使用 xelatex 或 lualatex 命令编译。

```tex
\documentclass{ctexart}
\begin{document}
在\LaTeX{}中排版中文。
汉字和English单词混排，通常不需要在中英文之间添加额外的空格。
当然，为了代码的可读性，加上汉字和 English 之间的空格也无妨。
汉字换行时不会引入多余的空格。
\end{document}
```



最简单的中文排版代码

```tex
\documentclass{ctexart}
\begin{document}
“你好，世界！”来自 \LaTeX{} 的问候。
\end{document}
```

其中 `{}` 用于产生空格，因为 `\LaTex` 会忽略后面所有的连续空格。



### 文档类

文档整体的格式为

```tex
\documentclass{...} % ... 为某文档类
% 导言区
\begin{document}
% 正文内容
\end{document}
% 此后内容会被忽略
```

文档类的声明为

```tex
\documentclass[⟨options⟩]{⟨class-name⟩}
```

其中 `class-name` 表示类名，基础类名如下

| 类名      | 含义                                                              |
| ------- | --------------------------------------------------------------- |
| article | 文章格式的文档类，广泛用于科技论文、报告、说明文档等                                      |
| report  | 长篇报告格式的文档类，具有章节结构，用于综述、长篇论文、简单的书籍等                              |
| book    | 书籍文档类，包含章节结构和前言、正文、后记等结构                                        |
| proc    | 基于 article 文档类的一个简单的学术文档模板                                      |
| slides  | 幻灯格式的文档类，使用无衬线字体                                                |
| minimal | 一个极其精简的文档类，只设定了纸张大小和基本字号，用作代码测试的最小工作示例（Minimal Working Example） |

可选参数 `⟨options⟩` 为文档类指定选项，以全局地规定一些排版的参数，如字号、纸张大小、单双面等等。比如调用 article 文档类排版文章，指定纸张为 A4 大小，基本字号为 11pt，双面排版：

```tex
\documentclass[11pt,twoside,a4paper]{article}
```



所有标准文档类都提供了一个 `\appendix` 命令将正文和附录分开，使用 `\appendix` 后，最高一级章节改为使用拉丁字母编号，从 A 开始。book 文档类还提供了前言、正文、后记结构的划分命令

| 命令             | 作用                                    |
| :------------- | :------------------------------------ |
| `\frontmatter` | 前言部分，页码使用小写罗马数字；其后的 `\chapter` 不编号    |
| `\mainmatter`  | 正文部分，页码使用阿拉伯数字，从 1 开始计数；其后的章节编号正常     |
| `\backmatter`  | 后记部分，页码格式不变，继续正常计数；其后的 `\chapter` 不编号 |

以上三个命令还可和 `\appendix` 命令结合，生成有前言、正文、附录、后记四部分的文档。



book 文档类的基本结构

```tex
\documentclass{book}

% 导言区，加载宏包和各项设置，包括参考文献、索引等
\usepackage{makeidx} 		% 调用 makeidx 宏包，用来处理索引
\makeindex 					% 开启索引的收集
\bibliographystyle{plain} 	% 指定参考文献样式为 plain

\begin{document}
\frontmatter 				% 前言部分
\maketitle 					% 标题页
\include{preface} 			% 前言章节 preface.tex
\tableofcontents
\mainmatter 				% 正文部分
\include{chapter1} 			% 第一章 chapter1.tex
\include{chapter2} 			% 第二章 chapter2.tex
...
\appendix 					% 附录
\include{appendixA} 		% 附录 A appendixA.tex
...
\backmatter 				% 后记部分
\include{epilogue} 			% 后记 epilogue.tex
\bibliography{books} 		% 利用 BibTeX 工具从数据库文件 books.bib 生成参考文献
\printindex 				% 利用 makeindex 工具生成索引
\end{document}
```



### 标题和目录

#### 标题页

LATEX 支持生成简单的标题页。首先需要给定标题和作者等信息

```tex
\title{⟨title⟩} \author{⟨author⟩} \date{⟨date⟩}
```

其中前两个命令是必须的（不用 `\title` 会报错；不用 `\author` 会警告），`\date` 命令可选。

> LATEX 还提供了 `\today` 命令自动生成当前日期，`\date` 默认使用 `\today` 生成。在 `\title, \author` 等命令内可以使用 `\thanks` 命令生成标题页的脚注，用 `\and` 隔开多个人名。



在信息给定后，就可以使用 `\maketitle` 命令生成一个简单的标题页了

```tex
\title{Test title}
\author{ Mary\thanks{E-mail:*****@***.com}
\and Ted\thanks{Corresponding author}
\and Louis}
\date{\today}
```



article 文档类的标题默认不单独成页，而 report 和 book 默认单独成页。可在 `\documentclass` 命令调用文档类时指定 `titlepage / notitlepage` 选项以修改默认的行为。LATEX 标准类还提供了一个简单的 titlepage 环境，生成不带页眉页脚的一页。用户可以在这个环境中使用各种排版元素自由发挥，生成自定义的标题页以替代 `\maketitle` 命令。甚至可以利用 titlepage 环境重新定义 `\maketitle`

```tex
\renewcommand{\maketitle}{\begin{titlepage}
... % 用户自定义命令
\end{titlepage}}
```

为标准文档类指定了 titlepage 选项以后，使用 `\maketitle` 命令生成的标题页就是一个 titlepage 环境。



#### 产生目录

在合适的地方使用命令

```tex
\tableofcontents
```

就会生成单独的一章（report / book）或一节（article），标题默认为 `Contents`，后面会介绍定制标题。\tableofcontents 生成的章节默认不写入目录（`\section*` 或 `\chapter*`），可使用 tocbibind 等宏包修改设置。正确生成目录项，一般需要编译两次源代码。



有时我们使用 `\chapter*` 或 `\section*` 这样不生成目录项的章节标题命令，而又想手动生成该章节的目录项，可以在标题命令后面使用

```tex
\addcontentsline{toc}{⟨level⟩}{⟨title⟩}
```

其中 level 为章节层次 chapter 或 section 等，title 为出现于目录项的章节标题。



### 划分文档

#### 断行和断页

使用如下命令实现手动断行

```tex
\\[10pt] \\*[1cm]
\newline
```

其中 `\\` 可以带参数表示增加间距，而且可以用于表格、公式换行；`\newline` 不带参数，只能用于文本换行。带星号的 `\\` 表示禁止在断行处分页。



使用如下命令断页

```tex
\newpage
\clearpage
```

在双栏排版模式中，`\newpage` 会另起一栏，`\clearpage` 则会另起一页。



#### 章节划分

三个标准文档类 article、report 和 book 提供了划分章节的命令：

```tex
\chapter{} \section{} \subsection{}
\subsubsection{} \paragraph{} \subparagraph{}
```

其中 `\chapter` 只在 report 和 book 文档类有定义。这些命令生成章节标题，并能够自动编号。除此之外 LATEX 还提供了 `\part` 命令，用来将整个文档分割为大的分块，但不影响 `\chapter` 或 `\section` 等的编号。



上述命令除了生成带编号的标题之外，还向目录中添加条目，并影响页眉页脚的内容。每个命令有两种变体：

* 带可选参数的变体：`\section[⟨short title⟩]{⟨title⟩}`

标题使用 `title` 参数，在目录和页眉页脚中使用 `short title` 参数

* 带星号的变体：`\section*{⟨title⟩}`

标题不带编号，也不生成目录项和页眉页脚。

较低层次如 `\paragraph` 和 `\subparagraph` 即使不用带星号的变体，生成的标题默认也不带编号。事实上，除 `\part` 外：

* article 文档类带编号的层级为 `\section  \subsection  \subsubsection` 三级
* report / book 文档类带编号的层级为 `\chapter  \section  \subsection` 三级



#### 分割文档

当编写长篇文档时，例如当编写书籍、毕业论文时，单个源文件会使修改、校对变得十分困难。将源文件分割成若干个文件，例如将每章内容单独写在一个文件中，会大大简化修改和校对的工作。使用 `\include` 命令插入这些文档

```tex
\include{⟨filename⟩} % filename⟩ 为文件名（不带 .tex 扩展名）
```

值得注意的是 `\include` 在读之前会另起一页。有的时候我们并不需要这样，而是用 `\input` 命令，它纯粹是把文件里的内容插入

```tex
\input{⟨filename⟩}
```

当导言区内容较多时，常常将其单独放置在一个 .tex 文件中，再用 `\input` 命令插入。复杂的图、表、代码等也会用类似的手段处理。



LATEX 还提供了一个 `\includeonly` 命令来组织文件，用于导言区，指定只载入某些文件。导言区使用了 `\includeonly` 后，正文中不在其列表范围的 `\include` 命令不会起效：

```tex
\includeonly{⟨filename1⟩,⟨filename2⟩,...}
```

> 使用 `\include` 和 `\input` 命令载入的文件名最好不要加空格和特殊字符，也尽量避免使用中文名，否则很可能会出错。



最后介绍一个实用的工具宏包 syntonly。加载这个宏包后，在导言区使用 `\syntaxonly` 命令，可令 LATEX 编译后不生成 DVI 或者 PDF 文档，只排查错误，编译速度会快不少

```tex
\usepackage{syntonly}
\syntaxonly
```

如果想生成文档，则用 % 注释掉 `\syntaxonly` 命令即可。



### 特殊环境

#### 列表环境

有序和无序列表环境 enumerate 和 itemize，两者的用法很类似，都用 `\item` 标明每个列表项。enumerate 环境会自动对列表项编号。

```tex
\begin{enumerate}
	\item first
	\item second
\end{enumerate}
```

其中 `\item` 可带一个可选参数，将有序列表的计数或者无序列表的符号替换成自定义的符号。列表可以嵌套使用，最多嵌套四层。



关键字环境 description 的用法与以上两者类似，不同的是 `\item` 后的可选参数用来写关键字，以粗体显示

```tex
\begin{description}
\item[⟨item title⟩] ...
\end{description}
```

它将生成关键字开头的列表

```tex
\begin{description}
\item[Enumerate] Numbered list.
\item[Itemize] Non-numbered list.
\end{description}
```



各级无序列表的符号由命令 `\labelitemi` 到 `\labelitemiv` 定义，可以简单地重新定义它们

```tex
\renewcommand{\labelitemi}{\ddag}
\renewcommand{\labelitemiv}{\dag}

\begin{itemize}
    \item First item
    
    \begin{itemize}
        \item Subitem
        \item Subitem
    \end{itemize}
    
    \item Second item
\end{itemize}
```

有序列表的符号由命令 `\labelenumi` 到 `\labelenumiv` 定义，重新定义这些命令需要用到计数器相关命令

```tex
\renewcommand{\labelenumi}%
{\Alph{enumi}>}

\begin{enumerate}
    \item First item
    \item Second item
\end{enumerate}
```



#### 对齐环境

center、flushleft 和 flushright 环境分别用于生成居中、左对齐和右对齐的文本环境

```tex
\begin{center} ... \end{center}
\begin{flushleft} ... \end{flushleft}
\begin{flushright} ... \end{flushright}
```

除此之外，还可以用以下命令直接改变文字的对齐方式：

```tex
\centering \raggedright \raggedleft
```

> center 等环境会在上下文产生一个额外间距，而 `\centering` 等命令不产生，只是改变对齐方式。在浮动体环境 table 或 figure 内用 `\centering` 命令即可实现居中对齐。



#### 引用环境

quote 用于引用较短的文字，首行不缩进；quotation 用于引用若干段文字，首行缩进。引用环境较一般文字有额外的左右缩进

```tex
Francis Bacon says:
\begin{quote}
	Knowledge is power.
\end{quote}
```



#### 代码环境

有时我们需要将一段代码原样转义输出，这就要用到代码环境 verbatim，它的带星号的版本 `verbatim*` 将空格显示成 `␣` 。例如

```tex
\begin{verbatim}
#include <iostream>
int main()
{
std::cout << "Hello, world!"
<< std::endl;
return 0;
}
\end{verbatim}
```

要排版简短的代码或关键字，可使用 `\verb` 命令

```tex
\verb⟨delim⟩⟨code⟩⟨delim⟩
```

delim 标明代码的分界位置，前后必须一致，除字母、空格或星号外，可任意选择使得不与代码本身冲突，习惯上使用 `|` 符号。



同 verbatim 环境，`\verb` 后也可以带一个星号，以显示空格

```tex
\verb|\LaTeX| \\
\verb+(a || b)+ \verb*+(a || b)+
```

> 由于 `\verb` 命令对符号的处理比较复杂，一般不能用在其它命令的参数里，否则多半会出错。



#### 多栏环境

可以创建多栏文本格式

```tex
\documentclass{article}

\usepackage{multicol}	% 多栏环境
\usepackage{lipsum}		% 虚拟文本

\begin{document}

\begin{multicols}{2}	% 生成 2 栏格式
    \lipsum[1-3]
\end{multicols}

\end{document}
```



### 创建盒子

#### 水平盒子

使用 `\mbox` 生成一个基本的水平盒子，内容只有一行，不允许分段（除非嵌套其它盒子）。使用 `\makebox` 可以加上可选参数

```latex
\begin{document}
|\mbox{Some words.}|
|\makebox[8em]{Some words.}|		% 默认居中 c
|\makebox[8em][l]{Some words.}|		% 左对齐
|\makebox[8em][r]{Some words.}|		% 右对齐
|\makebox[8em][s]{Some words.}|		% 分散对齐
\end{document}
```



#### 带框的水平盒子

使用 `\fbox` 和 `\framebox` 可以为水平盒子添加边框，通过 `\setlength` 命令边框的宽度 `\fboxrule` 和内边距 `\fboxsep`

```latex
\begin{document}
\framebox[10em][r]{Test box}\\[1ex]
\setlength{\fboxrule}{1.6pt}
\setlength{\fboxsep}{1em}
\framebox[10em][r]{Test box}
\end{document}
```



#### 垂直盒子

如果需要排版一个文字可以换行的盒子，LATEX 提供了两种方式

```tex
\parbox[⟨align⟩][⟨height⟩][⟨inner-align⟩]{⟨width⟩}{…}

\begin{minipage}[⟨align⟩][⟨height⟩][⟨inner-align⟩]{⟨width⟩} 
...
\end{minipage}
```

其中 `align` 为盒子和周围文字的对齐情况；`height` 和 `inner-align` 设置盒子的高度和内容的对齐方式

```tex
三字经：\parbox[t]{3em}%
{人之初 性本善 性相近 习相远}
\quad

% inner-align 可选参数 t,b,l,r
千字文：
\begin{minipage}[b][8ex][t]{4em}
天地玄黄 宇宙洪荒
\end{minipage}
```



如果在 minipage 里使用 `\footnote` 命令，生成的脚注会出现在盒子底部，编号是独立的，并且使用小写字母编号。而在 `\parbox` 里无法正常使用 `\footnote` 命令。

```latex
\begin{document}
\fbox{\begin{minipage}{15em}%
    Box test.
    \footnote{from minipage.}
\end{minipage}}
\qquad
\parbox[t]{15em}{
	Box test.
	\footnotemark{from parbox.}
}
\end{document}
```



#### 标尺盒子

\rule 命令用来画一个实心的矩形盒子，也可适当调整以用来画线（标尺）：

```tex
\rule[⟨raise⟩]{⟨width⟩}{⟨height⟩}
```

其中 `raise` 参数设置竖直方向的位置

```latex
\begin{document}
Black \rule{12pt}{4pt} box.
Upper \rule[4pt]{6pt}{8pt} and
lower \rule[-4pt]{6pt}{8pt} box.
A \rule[-.4pt]{3em}{.4pt} line.
\end{document}
```



## 数学公式

本章介绍的许多命令和环境依赖于 amsmath 宏包，这些命令和环境将以蓝色示意。以下示例都假定了导言区中写有

```tex
\usepackage{amsmath}
```



### 公式排版基础

#### 行内和行间公式

数学公式有两种排版方式：

1. 与文字混排，称为行内公式；
2. 单独列为一行排版，称为行间公式。

行内公式由一对 `$` 符号包裹；单独成行的行间公式在 LATEX 里由 equation 环境包裹。



equation 环境为公式自动生成一个编号，用 `\label` 和 `\ref` 生成交叉引用，amsmath 的 `\eqref` 命令为引用加上圆括号；可以用 `\tag` 命令手动修改公式的编号，或者用 `\notag` 命令取消为公式编号（与之基本等效的命令是 `\nonumber`）。

```tex
The Pythagorean theorem is:

\begin{equation}
a^2 + b^2 = c^2 \label{pythagorean}
\end{equation}

Equation \eqref{pythagorean} is
called `Gougu theorem' in Chinese.
```



如果需要直接使用不带编号的行间公式，则将公式用命令 `[]` 包裹，与之等效的是 displaymath 环境。有的人更喜欢 `equation*` 环境，体现了带星号和不带星号的环境之间的区别

```tex
\begin{equation*}
a^2 + b^2 = c^2
\end{equation*}

For short:
\[ a^2 + b^2 = c^2 \]
Or if you like the long one:

\begin{displaymath}
a^2 + b^2 = c^2
\end{displaymath}
```

行间公式的对齐、编号位置等性质由文档类选项控制，文档类的 `fleqn` 选项令行间公式左对齐；`leqno` 选项令编号放在公式左边。



#### 数学符号

我们只提及几个较为重要的符号指令

```tex
f'(x) f''(x)	% 导数符号
\sqrt[n]{...}	% n 次方根
\binom{}{}		% 二项式形式
\frac{}{}		% 分式
\dfrac{}{}		% 行间公式大小的分式
\tfrac{}{}		% 行中公式大小的分式
```



如果算符不够用的话，amsmath 允许用户在导言区用 \DeclareMathOperator 定义自己的算符，**带星号的命令定义带上下限的算符**：

```tex
\DeclareMathOperator{\argh}{argh}
\DeclareMathOperator*{\nut}{Nut}
```



巨算符 $\int,\sum$ 等符号的上下标位置可由 `\limits` 和 `\nolimits` 调整
$$
\sum\limits_{i=1}^n,\quad \sum\nolimits_{i=1}^n
$$
amsmath 宏包还提供了 `\substack`，能够在下限位置书写多行表达式；subarray 环境令多行表达式可选择居中 `(c)` 或左对齐 `(l)`
$$
\sum_{\substack{0\le i\le n \\
j\in \mathbb{R}}}
P(i,j) = Q(n),

\quad 

\sum_{\begin{subarray}{l}
0\le i\le n \\
j\in \mathbb{R}
\end{subarray}}
P(i,j) = Q(n)
$$



#### 数学重音和上下括号

数学符号可以像文字一样加重音，使用时要注意重音符号的作用区域，一般应当对某个符号而不是“符号加下标”使用重音：
$$
\bar{x}_{0} \quad \bar{x}_0\\[5pt]
\vec{x}_0 \quad \vec{x}_0\\[5pt]
\hat{\mathbf{e}}_x \quad
\hat{\mathbf{e}}_x
$$
LATEX 也能为多个字符加重音，包括

* 直接画线的 `\overline` 和 `\underline` 命令（可叠加使用）
* 宽重音符号 `\widehat`
* 表示向量的箭头 `\overrightarrow`

$$
0.\overline{3} =
\underline{\underline{1/3}}\\[5pt]
\hat{XY} \qquad \widehat{XY}\\[5pt]
\vec{AB} \qquad
\overrightarrow{AB}
$$
使用 `\overbrace` 和 `\underbrace` 命令用来生成上/下括号，各自可带一个上/下标公式
$$
\underbrace{\overbrace{(a+b+c)}^6 \cdot \overbrace{(d+e+f)}^7}_\text{meaning of life} = 42
$$



#### 箭头和上标

用的箭头包括 `\rightarrow` (*→*，或 `\to`)、`\leftarrow`（*←*，或 `\gets`）等。amsmath 的 `\xleftarrow` 和 `\xrightarrow` 命令提供了长度可以伸展的箭头，并且可以为箭头增加上下标
$$
a\xleftarrow{x+y+z} b\quad c\xrightarrow[x<y]{a*b*c}d
$$



#### 括号和定界符

LATEX 提供了多种括号和定界符表示公式块的边界，如小括号 $()$、中括号 $[]$、大括号 $\{\}$、尖括号 $\langle \rangle$ ；使用 `\left` 和 `\right` 命令可令括号（定界符）的大小可变，在行间公式中常用。LATEX 会自动根据括号内的公式大小决定定界符大小。
$$
1 + \left(\frac{1}{1-x^{2}}
\right)^3 \qquad
\left.\frac{\partial f}{\partial t}
\right|_{t=0}
$$
可以用 `\big, \bigg` 等命令生成固定大小的定界符。更常用的形式是类似 `\left` 的 `\bigl, \biggl` 等，以及类似 `\right` 的 `\bigr, \biggr` 等（`\bigl, \bigr` 不必成对出现）
$$
\Bigl((x+1)(x-1)\Bigr)^{2}\\
\bigl( \Bigl( \biggl( \Biggl( \quad
\bigr\} \Bigr\} \biggr\} \Biggr\} \quad
\big\| \Big\| \bigg\| \Bigg\| \quad
\big\Downarrow \Big\Downarrow
\bigg\Downarrow \Bigg\Downarrow
$$

> 使用 `\big` 和 `\bigg` 等命令的另外一个好处是——用 `\left` 和 `\right` 分界符包裹的公式块是不允许断行的（下文提到的 array 或者 aligned 等环境视为一个公式块），所以也不允许在多行公式里跨行使用，而 `\big` 和 `\bigg` 等命令不受限制。



### 多行公式

#### 长公式折行

通常来讲应当避免写出超过一行而需要折行的长公式。如果一定要折行的话，习惯上优先在等号之前折行，其次在加号、减号之前，再次在乘号、除号之前，其它位置应当避免折行。amsmath 宏包的 multline 环境提供了书写折行长公式的方便环境。它允许用 `\\` 折行，将公式编号放在最后一行。多行公式的首行左对齐，末行右对齐，其余行居中

```tex
\begin{multline}
a + b + c + d + e + f
+ g + h + i \\
= j + k + l + m + n\\
= o + p + q + r + s\\
= t + u + v + x + z
\end{multline}
```

与表格不同的是，公式的最后一行不写 `\\`，如果写了，反倒会产生一个多余的空行。



#### 多行公式

更多的情况是，我们需要罗列一系列公式，并令其按照等号对齐。目前最常用的是 align 环境，它将公式用 & 隔为两部分并对齐。
$$
\begin{align}
a & = b + c \\
& = d + e
\end{align}
$$
align 环境会给每行公式都编号，可以用 `\notag` 去掉某行的编号。为了对齐等号，添加一对括号 `{}` 以产生正常的间距
$$
\begin{align}
a ={} & b + c \\
={} & d + e + f + g + h + i
+ j + k + l \notag \\
& + m + n + o \\
={} & p + q + r + s
\end{align}
$$
align 还能够对齐多组公式，除等号前的 `&` 之外，公式之间也用 `&` 分隔
$$
\begin{align}
a &=1 & b &=2 & c &=3 \\
d &=-1 & e &=-2 & f &=-5
\end{align}
$$
如果我们不需要按等号对齐，只需罗列数个公式，gather 将是一个很好用的环境
$$
\begin{gather}
a = b + c \\
d = e + f + g \\
h + i = j + k \notag \\
l + m = n
\end{gather}
$$
align 和 gather 有对应的不带编号的版本 `align*` 和 `gather*` 。使用 array 环境能够实现更灵活的对齐

$$
% cl 分别表示两列居中对齐、左对齐
\begin{array}{cl}
a_1 &\verb|a_1|\\
x^2 &\verb|x^2|\\
e^{-\alpha t} &\verb|e^{-\alpha t}| \\
a^3_{ij} &\verb|a^3_{ij}| \\
e^{x^2}\neq e^{x^2} &\verb|e^{x^2}\neq e^{x^2}|
\end{array}
$$
通过 `r/c/l` 控制对齐方式。



#### 公用编号

另一个常见的需求是将多个公式组在一起公用一个编号，编号位于公式的居中位置。为此，amsmath 宏包提供了诸如 aligned、gathered 等环境，与 equation 环境套用。以 `-ed` 结尾的环境用法与前一节不以 `-ed` 结尾的环境用法一一对应。以 aligned 举例
$$
\begin{equation}
\begin{aligned}
a &= b + c \\
d &= e + f + g \\
h + i &= j + k \\
l + m &= n
\end{aligned}
\end{equation}
$$
split 环境和 aligned 环境用法类似，也用于和 equation 环境套用，区别是 split 只能将每行的一个公式分两栏，aligned 允许每行多个公式多栏。



### 数组和矩阵

为了排版二维数组，LATEX 提供了 array 环境，用法与 tabular 环境极为类似，也需要定义列格式，并用 `\\` 换行。数组可作为一个公式块，在外套用 `\left, \right` 等定界符
$$
\mathbf{X} = \left(
\begin{array}{cccc}
x_{11} & x_{12} & \ldots & x_{1n}\\
x_{21} & x_{22} & \ldots & x_{2n}\\
\vdots & \vdots & \ddots & \vdots\\
x_{n1} & x_{n2} & \ldots & x_{nn}\\
\end{array} \right)
$$
值得注意的是，上一节末尾介绍的 aligned 等环境也可以用定界符包裹。



我们还可以利用空的定界符排版出这样的效果
$$
|x| = \left\{
\begin{array}{rl}
-x & \text{if } x < 0,\\
0 & \text{if } x = 0,\\
x & \text{if } x > 0.
\end{array} \right.
$$
不过上述例子可以用 amsmath 提供的 cases 环境更轻松地完成
$$
|x| =
\begin{cases}
-x & \text{if } x < 0,\\
0 & \text{if } x = 0,\\
x & \text{if } x > 0.
\end{cases}
$$
我们当然也可以用 array 环境排版各种矩阵。amsmath 宏包还直接提供了多种排版矩阵的环境，包括不带定界符的 matrix，以及带各种定界符的矩阵 pmatrix $($、bmatrix $[$、Bmatrix $\{$、vmatrix $|$、Vmatrix $\|$ 等
$$
\begin{matrix}
1 & 2 \\ 3 & 4
\end{matrix} 
\qquad
\begin{bmatrix}
x_{11} & x_{12} & \ldots & x_{1n}\\
x_{21} & x_{22} & \ldots & x_{2n}\\
\vdots & \vdots & \ddots & \vdots\\
x_{n1} & x_{n2} & \ldots & x_{nn}\\
\end{bmatrix}
$$
在矩阵中的元素里排版分式时，一来要用到 `\dfrac` 等命令，二来行与行之间有可能紧贴着，这时要调节间距：
$$
\mathbf{H}=
\begin{bmatrix}
\dfrac{\partial^2 f}{\partial x^2} &
\dfrac{\partial^2 f}
{\partial x \partial y} \\[8pt]
\dfrac{\partial^2 f}
{\partial x \partial y} &
\dfrac{\partial^2 f}{\partial y^2}
\end{bmatrix}
$$
其中用 `[8pt]` 来增大行间距。



### 公式中的间距

数学公式中各元素的间距是根据符号类型自动生成的，需要手动调整的情况极少。在公式中我们还可能用到 `\,`、`\:`、`\;` 以及负间距 `\!`，其中后三个命令只用于数学环境

| 命令       | 效果          |
| -------- | ----------- |
| `\quad`  | $a\quad a$  |
| `\qquad` | $a\qquad a$ |
| `\,`     | $a\,a$      |
| `\:`     | $a\:a$      |
| `\;`     | $a\;a$      |
| `\!`     | $a\!a$      |



一个常见的用途是修正积分的被积函数 $f(x)$ 和微元 $dx$ 之间的距离。注意微元里的 d 用的是直立体
$$
\int_a^b f(x)\mathrm{d}x
\qquad
\int_a^b f(x)\,\mathrm{d}x
$$
另一个用途是生成多重积分号。如果我们直接连写两个 `\int`，之间的间距将会过宽，此时可以使用负间距 `\!` 修正。不过 amsmath 提供了更方便的多重积分号，如二重积分 `\iint`、三重积分 `\iiint` 等
$$
\newcommand\diff{\,\mathrm{d}}
\begin{gather*}
\int\int f(x)g(y)
\diff x \diff y \\
\int\!\!\!\int
f(x)g(y) \diff x \diff y \\
\iint f(x)g(y) \diff x \diff y \\
\iint\quad \iiint\quad \idotsint
\end{gather*}
$$



### 字体控制

#### 字母字体

LATEX 允许一部分数学符号切换字体，主要是拉丁字母、数字、大写希腊字母以及重音符号等，使用 amssymb 宏包
$$
% \usepackage{amssymb}
\mathcal{R} \quad \mathfrak{R}
\quad \mathbb{R}
\quad
\mathcal{L}
= -\frac{1}{4}F_{\mu\nu}F^{\mu\nu}\\
\mathfrak{su}(2)\text{ and }
\mathfrak{so}(3)\text{ Lie algebra}
$$
![](image-20220715173019247.png)



#### 加粗符号

使用 `\mathbf` 命令只能获得直立、加粗的字母。如果想得到粗斜体，可以使用 amsmath 宏包提供的 `\boldsymbol` 命令：
$$
\mu, M \qquad\boldsymbol{\mu}, \boldsymbol{M}
$$
也可以使用 bm 宏包提供的 `\bm` 命令：

```latex
\usepackage{bm}

\begin{document}
$\mu, M \qquad \bm{\mu}, \bm{M}$
\end{document}
```

在 LATEX 默认的数学字体中，一些符号本身并没有粗体版本，使用 `\boldsymbol` 也得不到粗体。此时 `\bm` 命令会生成“伪粗体”，尽管效果比较粗糙，但在某些时候也不失为一种解决方案。



#### 符号尺寸

数学符号按照符号排版的位置规定尺寸，从大到小包括行间公式尺寸、行内公式尺寸、上下标尺寸、次级上下标尺寸。除了字号有别之外，行间和行内公式尺寸下的巨算符也使用不一样的大小。LATEX 为每个数学尺寸指定了一个切换的命令
$$
\displaystyle\sum a,\quad \textstyle\sum a,\quad \scriptstyle a,\quad \scriptscriptstyle a
$$
我们通过以下示例对比行间公式和行内公式的区别。在分式中，分子分母默认为行内公式尺寸，示例中将分母切换到行间公式尺寸：
$$
r = \frac
{\sum_{i=1}^n (x_i- x)(y_i- y)}
{\displaystyle \left[
\sum_{i=1}^n (x_i-x)^2
\sum_{i=1}^n (y_i-y)^2
\right]^{1/2} }
$$



### 定理环境

#### 原始定理环境

使用 LATEX 排版数学和其他科技文档时，会接触到大量的定理、证明等内容。LATEX 提供了一个基本的命令 `\newtheorem` 提供定理环境的定义：

```tex
\newtheorem{⟨theorem environment⟩}{⟨title⟩}[⟨section-level⟩]
\newtheorem{⟨theorem environment⟩}[⟨counter⟩]{⟨title⟩} 
```

theorem environment 为定理环境的名称。原始的 LATEX 里没有现成的定理环境，不加定义而直接使用很可能会出错。title 是定理环境的标题（“定理”，“公理”等）。



定理的序号由两个可选参数之一决定，它们不能同时使用 

* section-level 为章节级别，如 chapter、section 等，定理序号成为章节的下一级序号
* counter 为用 \newcounter 自定义的计数器名称，定理序号由这个计数器管理

如果两个可选参数都不用的话，则使用默认的与定理环境同名的计数器。



我们定义一个 mythm 环境，其序号设为 section 的下一级序号

```tex
\newtheorem{mythm}{My Theorem}[section]

\begin{mythm}\label{thm:light}
The light speed in vacuum
is $299,792,458\,\mathrm{m/s}$.
\end{mythm}

\begin{mythm}[Energy-momentum relation]
The relationship of energy,
momentum and mass is
\[E^2 = m_0^2 c^4 + p^2 c^2\]
where $c$ is the light speed
described in theorem \ref{thm:light}.
\end{mythm}
```

![](image-20220715173958778.png)



#### amsthm 宏包

amsthm 提供了 `\theoremstyle` 命令支持定理格式的切换，在用 `\newtheorem` 命令定义定理环境之前使用。amsthm 预定义了三种格式

* plain 和 LATEX 原始的格式一致
* definition 使用粗体标签、正体内容
* remark 使用斜体标签、正体内容

另外 amsthm 还支持用带星号的 `\newtheorem*` 定义不带序号的定理环境

```tex
\theoremstyle{definition} \newtheorem{law}{Law}
\theoremstyle{plain} \newtheorem{jury}[law]{Jury}
\theoremstyle{remark} \newtheorem*{mar}{Margaret}
```

以下例子定义的 jury 环境与 law 环境共用编号，mar 环境不编号

```tex
\begin{law}\label{law:box}
Don't hide in the witness box.
\end{law}

\begin{jury}[The Twelve]
It could be you! So beware and
see law~\ref{law:box}.\end{jury}

\begin{jury}
You will disregard the last
statement.\end{jury}

\begin{mar}No, No, No\end{mar}

\begin{mar}Denis!\end{mar}
```

![](image-20220715174147015.png)

amsthm 还支持使用 `\newtheoremstyle` 命令自定义定理格式，更为方便使用的是 ntheorem 宏包。



#### 证明环境和证毕符号

amsthm 还提供了一个 proof 环境用于排版定理的证明过程。proof 环境末尾自动加上一个证毕符号

```tex
\begin{proof}
For simplicity, we use
\[
E=mc^2
\]
That's it.
\end{proof}
```

如果行末是一个不带编号的公式， 符号会另起一行，这时可使用 `\qedhere` 命令将符号放在公式末尾

```tex
\begin{proof}
For simplicity, we use
\[
E=mc^2 \qedhere
\]
\end{proof}
```

\qedhere 对于 `align*` 等环境也有效

```tex
\begin{proof}
Assuming $\gamma
= 1/\sqrt{1-v^2/c^2}$, then
\begin{align*}
E &= \gamma m_0 c^2 \\
p &= \gamma m_0v \qedhere
\end{align*}
\end{proof}
```

在使用带编号的公式时，建议最好不要在公式末尾使用 `\qedhere` 命令。对带编号的公式使用 `\qedhere` 命令会使符号放在一个难看的位置。



证毕符号本身被定义在命令 `\qedsymbol` 中，如果有使用实心符号作为证毕符号的需求，需要自行用 `\renewcommand` 命令修改。可以利用标尺盒子来生成一个适当大小的“实心矩形”作为证毕符号

```tex
\renewcommand{\qedsymbol}%
{\rule{1ex}{1.5ex}}

\begin{proof}
For simplicity, we use
\[
E=mc^2 \qedhere
\]
\end{proof}
```



## 特色工具和功能

### 参考文献

biblatex 宏包是一套基于 LATEX 宏命令的参考文献解决方案，提供了便捷的格式控制和强大的排序、分类、筛选、多文献表等功能。biblatex 宏包也因其对 UTF-8 和中文参考文献的良好支持，被国内较多 LATEX 模板采用。


#### BibTex

传统方式是使用 bibtex 编译，编译步骤为

```tex
xelatex demo
bibtex demo
xelatex demo
xelatex demo
```

例如按照下面的方式定义 bib 数据库，并在 tex 文件中使用

```tex
% File: egbibdata.bib

@book{caimin2006,
    title = {UML 基础和 Rose 建模教程},
    address = {北京},
    author = {蔡敏 and 徐慧慧 and 黄柄强},
    publisher = {人民邮电出版社},
    year = {2006},
    month = {1}
}

% File: demo.tex

\documentclass{ctexart}

% 使用符合 GB/T 7714-2015 规范的参考文献样式
\usepackage[style=gb7714-2015]{biblatex}
\addbibresource{egbibdata.bib}

\begin{document}

见文献\cite{caimin2006}。

% 指定样式，以及 egbibdata.bib 文件
\bibliographystyle{splncs04}
\bibliography{egbibdata}

\end{document}
```



#### BibLaTex

与基于 BIBTEX 的传统方式不同的是，biblatex 宏包使用 biber 程序处理参考文献。因此上述文档的编译步骤为

```tex
xelatex demo
biber demo
xelatex demo
xelatex demo
```

如果要在 TexStudio 中编译，需要在“设置-构建”中将默认文献编译工具改为 Biber 。 



例如按照下面的方式定义 bib 数据库，并在 tex 文件中使用

```tex
% File: egbibdata.bib

@book{caimin2006,
    title = {UML 基础和 Rose 建模教程},
    address = {北京},
    author = {蔡敏 and 徐慧慧 and 黄柄强},
    publisher = {人民邮电出版社},
    year = {2006},
    month = {1}
}

% File: demo.tex

\documentclass{ctexart}

% 使用符合 GB/T 7714-2015 规范的参考文献样式
\usepackage[style=gb7714-2015]{biblatex}
\addbibresource{egbibdata.bib}

\begin{document}

见文献\cite{caimin2006}。

\printbibliography

\end{document}
```

需要注意，bib 文件只能导入并编译一次，之后修改 bib 无法进行编译。如果要调整参考文献，有几种方式：

* 删除编译生成的 `.bbl` ，这样才会重新调用 Biber 编译新的 bib 文件
* 添加新的参考文件引用
* 修改 bib 文件的路径



#### biblatex 样式和其它选项

biblatex 使用的参考文献样式分为著录样式（bibliography style）和引用样式（citation style），分别以 `.bbx`  和 `.cbx`  为扩展名。参考文献的样式在调用宏包时使用 style 选项指定，或者使用 bibstyle 或 citestyle 分别指定

```tex
% 同时调用 gb7714-2015.bbx 和 gb7714-2015.cbx
\usepackage[style=gb7714-2015]{biblatex}
% 著录样式调用 gb7714-2015.bbx，引用样式调用 biblatex 宏包自带的 authoryear
\usepackage[bibstyle=gb7714-2015,citestyle=authoryear]{biblatex}
```



以下总结一些常用的参考文献样式，除 biblatex 宏包自带的样式外，许多样式以单独的宏包在发行版内发布。

* **authoryear** biblatex 自带样式，类似 natbib 默认的引用样式效果
* **authortitle** biblatex 自带样式，采用作者-题名（shorttitle 字段）的引用样式
* **verbose** biblatex 自带样式，引用样式中包含作者、题名、书名、页码等字段的信息
* **alphabetic** biblatex 自带样式，著录样式与 BIBTEX 的 alpha 样式类似
* **trad-alpha** biblatex-trad 样式包，移植自 BIBTEX 默认的 alpha 样式。另外还包括 trad-abbrv、trad-plain 和 trad-unsrt
* **gb7714-2015** 符合中文文献著录标准 GB/T 7714—2015 的样式，著录按顺序编码排版。另外还包括按作者—年份顺序排版著录的样式 gb7714-2015ay
* **caspervector** 以中文文献著录标准 GB/T 7714—2015 为基础的一个样式
* **ieee** 兼容 IEEEtran 风格的样式，著录按顺序编码排版。另外还包括按作者—年份顺序排版著录的样式 ieee-alphabetic



### 使用颜色

原始的 LATEX 不支持使用各种颜色。color 宏包或者 xcolor 宏包提供了对颜色的支持，给 PDF 输出生成颜色的特殊指令。



#### 颜色的表达方式

调用 color 或 xcolor 宏包后，我们就可以用如下命令切换颜色

```tex
\color[⟨color-mode⟩]{⟨code⟩}
\color{⟨color-name⟩}
```

颜色的表达方式有两种，其一是使用色彩模型和色彩代码，代码用 `0∼1` 的数字代表成分的比例。color 宏包支持 rgb、cmyk 和 gray 模型，xcolor 支持更多的模型如 hsb 等

```latex
\begin{document}
\large\sffamily
{\color[gray]{0.6}
60\% gray} \quad
{\color[rgb]{0,1,1}
cyan}
\end{document}
```

其二是直接使用名称代表颜色，前提是已经定义了颜色名称（没定义的话会报错）

```latex
\begin{document}
\large\sffamily
{\color{red} red} \quad
{\color{blue} blue}
\end{document}
```

color 宏包仅定义了 8 种颜色名称，xcolor 补充了一些，总共有 19 种

![](image-20220715211921524.png)



xcolor 还支持将颜色通过表达式混合或互补

```latex
\begin{document}
\large\sffamily
{\color{red!40} 40\% red}\quad
{\color{blue}blue\quad
\color{blue!50!black}blue-black\quad
\color{black}black}\quad
{\color{-red}complementary}
\end{document}
```

我们还可以通过命令自定义颜色名称，这里的 ⟨color-mode⟩ 是必选参数

```tex
\definecolor{⟨color-name⟩}{⟨color-mode⟩}{⟨code⟩}
```

如果调用 color 或 xcolor 宏包时指定 `dvipsnames` 选项，就有额外的 68 种颜色名称可用。



#### 带颜色的文本和盒子

原始的 `\color` 命令类似于字体命令 `\bfseries`，它使之后排版的内容全部变成指定的颜色，所以直接使用时通常要加花括号分组。color / xcolor 宏包都定义了一些方便用户使用的带颜色元素。



输入带颜色的文本可以用类似 `\textbf` 的命令

```tex
\textcolor[⟨color-mode⟩]{⟨code⟩}{⟨text⟩}
\textcolor{⟨color-name⟩}{⟨text⟩}
```

以下命令构造一个带背景色的盒子，`⟨material⟩` 为盒子中的内容

```tex
\colorbox[⟨color-mode⟩]{⟨code⟩}{⟨material⟩}
\colorbox{⟨color-name⟩}{⟨material⟩}
```

以下命令构造一个带背景色和有色边框的盒子，`⟨fcode⟩` 或 `⟨fcolor-name⟩` 用于设置边框颜色

```tex
\fcolorbox[⟨color-mode⟩]{⟨fcode⟩}{⟨code⟩}{⟨material⟩}
\fcolorbox{⟨fcolor-name⟩}{⟨color-name⟩}{⟨material⟩}
```

`\fcolorbox` 也可以像 `\fbox` 那样调节 `\fboxrule` 和 `\fboxsep`；对于 `\colorbox`，调整 `\fboxsep` 是有效的

```latex
\begin{document}
\sffamily
Text is \textcolor{red}{red}\quad
\colorbox[gray]{0.95}{gray background} \quad
\fcolorbox{blue}{yellow}{%
\textcolor{blue}{blue border + text,%
	yello background} 
}
\end{document}
```



### 使用超链接

LATEX 中实现这一功能的是 hyperref 宏包。



#### hyperref 宏包

hyperref 宏包涉及的链接遍布 LATEX 的每一个角落——目录、引用、脚注、索引、参考文献等等都被封装成超链接。但这也使得它与其它宏包发生冲突的可能性大大增加。

> 为减少可能的冲突，习惯上将 hyperref 宏包放在其它宏包之后调用。 



hyperref 宏包提供了命令 `\hypersetup` 配置各种参数。参数也可以作为宏包选项，在调用宏包时指定

```tex
\hypersetup{⟨option1⟩,⟨option2⟩={value},...}
\usepackage[⟨option1⟩,⟨option2⟩={value},...]{hyperref}
```

当选项值为 true 时，可以省略“=true”不写。可选参数如下表

![](image-20220715212758551.png)



#### 超链接

hyperref 宏包提供了直接书写超链接的命令，用于在 PDF 中生成 URL

```tex
\url{⟨url⟩}
\nolinkurl{⟨url⟩}
```

命令  `\url` 和 `\nolinkurl` 都输出一个 URL，为 URL 加上了超链接。在 `\url` 等命令的参数 `⟨url⟩` 里，可直接输入如 `%、&` 这样的特殊符号。



我们也可以像 HTML 中的超链接一样，把一段文字作为超链接

```latex
\begin{document}
\href{https://wikipedia.org}{link}
\url{https://wikipedia.org} \\
\nolinkurl{https://wikipedia.org} \\
\href{https://wikipedia.org}{Wiki}
\end{document}
```



使用 hyperref 宏包后，文档中所有的引用、参考文献、索引等等都转换为超链接。用户也可对某个 `\label` 命令定义的标签 `⟨label⟩` 作超链接（这里的 `⟨label⟩` 虽然是可选参数的形式，但通常是必填的）

```tex
\hyperref[⟨label⟩]{⟨text⟩} 
```



默认的超链接在文字外边加上一个带颜色的边框（在打印 PDF 时边框不会打印）

* 指定 `colorlinks` 参数修改为将文字本身加上颜色
* 修改 `pdfborder` 参数调整边框宽度以“去掉”边框
* 指定 `hidelinks` 参数则令超链接既不变色也不加边框

```tex
\hypersetup{hidelinks}
% or:
\hypersetup{pdfborder={0 0 0}}
```



#### PDF 书签

hyperref 宏包另一个强大的功能是为 PDF 生成书签。对于章节命令 `\chapter, \section` 等，默认情况下会为 PDF 自动生成书签。和交叉引用、索引等类似，生成书签也需要多次编译源代码，第一次编译将书签记录写入 .out 文件，第二次编译才正确生成书签。



hyperref 还提供了手动生成书签的命令：

```tex
\pdfbookmark[⟨level⟩]{⟨bookmark⟩}{⟨anchor⟩} 
```

其中 `bookmark` 为书签名称，`anchor` 为书签项使用的锚点（类似交叉引用的标签）。可选参数 `level` 为书签的层级，默认为 0 。书签的一些属性在前面表格 `6.4` 中。



章节命令里往往有 LATEX 命令甚至数学公式，而 PDF 书签是纯文本，对命令和公式的处理很困难，有出错的风险。hyperref 宏包已经为我们处理了许多常见命令，如 `\LaTeX` 和字体命令 `\textbf` 等，对于未被处理的命令或数学公式，就要在章节标题中使用如下命令，分别提供 LATEX 代码和 PDF 书签可用的纯文本

```tex
\texorpdfstring{⟨LATEX code⟩}{⟨PDF bookmark text⟩}
```

比如在章节名称里使用公式 $E=mc^2$ ，而书签则使用纯文本形式的 `E=mc^2`

```tex
\section{质能公式 \texorpdfstring{$E=mc^2$}{E=mc\textasciicircum 2}}
```



## TikZ 绘图

除了排版文字，LATEX 也支持用代码表示图形。不同的扩展已经极大地丰富了 LATEX 的图形功能，[TikZ](https://texample.net/tikz/examples/) 就是其中之一。



### 使用格式

在导言区调用 tikz 宏包，就可以用以下命令和环境使用 TikZ 的绘图功能了

```tex
\tikz[...] ⟨tikz code⟩;
\tikz[...] {⟨tikz code 1⟩; ⟨tikz code 2⟩; ...}
\begin{tikzpicture}[...]
⟨tikz code 1⟩; ⟨tikz code 2⟩;
...
\end{tikzpicture}
```

前一种用法为 `\tikz` 带单条绘图命令，以分号结束，一般用于在文字之间插入简单的图形；后两种用法较为常见，使用多条绘图命令，可以在 figure 等浮动体中使用。


使用 `\filldraw` 命令绘制实心点，使用 `help lines` 参数绘制辅助线

```latex
\begin{document}
\begin{tikzpicture}
\filldraw [gray] (0,0) circle (2pt)
				(1,1) circle (2pt)
				(2,1) circle (2pt)
				(2,0) circle (2pt);
\draw [help lines] (0,0) -- (1,1) -- (2,1) -- (2,0);
\end{tikzpicture}
\end{document}
```

如果需要放大绘图结果，可以添加 `scale` 参数

```latex
\begin{document}
\begin{tikzpicture}[scale=2]
\filldraw [gray] (0,0) circle (2pt)
				(1,1) circle (2pt)
				(2,1) circle (2pt)
				(2,0) circle (2pt);
\draw [help lines] (0,0) -- (1,1) -- (2,1) -- (2,0);
\end{tikzpicture}
\end{document}
```



### 绘制路径
#### 直线路径

在 TikZ 中绘制直线路径只需要通过 `\draw` 命令并连接多个点坐标

```latex
\begin{document}
\begin{tikzpicture}
\draw (-1.5,0) -- (1.5,0) -- (0,-1.5) -- (0,1.5);
\end{tikzpicture}
\end{document}
```

连续使用连线时，可以使用 `cycle` 令路径平滑地回到起点

```latex
\begin{document}
\begin{tikzpicture}[line width=5pt]
\draw (0,0) -- (1,0) -- (1,1) -- cycle;
\draw (2,0) -- (3,0) -- (3,1) -- (2,0);
\useasboundingbox (0,1.5);		% 增高包围盒
\end{tikzpicture}
\end{document}
```

多条路径可用于同一条画图命令中，以空格分隔

```latex
\begin{document}
\begin{tikzpicture}
\draw (0,0) -- (0,1)
(1,0) -- (1,1) -- (2,0) -- cycle;
\end{tikzpicture}
\end{document}
```



#### 曲线路径

通过 `.. controls ... and ... ..` 语法指定控制点，可以绘制 Bezier 曲线

```latex
\begin{document}
\begin{tikzpicture}
\filldraw [gray] (0,0) circle (2pt)
				(1,1) circle (2pt)
				(2,1) circle (2pt)
				(2,0) circle (2pt);
\draw (0,0) .. controls (1,1) and (2,1) .. (2,0);
\end{tikzpicture}
\end{document}
```



#### 基本形状

使用 `circle, ellipse, rectangle` 参数来绘制基本形状

```latex
\begin{document}

\begin{tikzpicture}
\draw (0,0) rectangle (1.5,1);						% 矩形：指定左下角和右上角
\draw (2.5,0.5) circle [radius=0.5];				% 圆：通过 [] 显式指定半径
\draw (2.5,0.5) circle (1cm);						% 圆：通过 () 隐式指定半径
\draw (4.5,0.5) ellipse [x radius=1,y radius=0.5];	% 椭圆：通过 [] 显式指定长短轴
\draw (4.5,0.5) ellipse (20pt and 10pt);			% 椭圆：通过 () 隐式指定长短轴
\end{tikzpicture}

\end{document}
```

可以通过 `()` 隐式依次指定参数，或者通过 `[]` 和参数名显示指定参数值。



要绘制圆弧，就需要指定极坐标表示

```latex
\begin{document}

\begin{tikzpicture}
\draw (3cm,0cm) arc (0:30:3cm);
\draw (6,0) arc (0:135:1 and 0.5);
\end{tikzpicture}

\end{document}
```

其中 `(3cm,0cm)` 指定圆心位置，`(0:30:3cm)` 指定角度为 `0~30` 度半径为 `3cm` 的圆弧。如果指定两个半径 `1 and 0.5` 就会得到椭圆弧。



#### 特殊曲线

正弦、余弦曲线（1/4 周期）使用 `sin, cos` 参数指定曲线的起点和终点，这些命令可以并用。例如指定 5 个特殊端点生成一段连续曲线

```latex
\begin{document}
\begin{tikzpicture}
\draw [x=1.57cm,y=1cm] (0,0) sin (1,1) cos (2,0) sin (3,-1) cos (4,0);
\end{tikzpicture}
\end{document}
```

抛物线用 `bend` 参数设定一个抛物线上的点来绘制

```latex
\begin{document}
\begin{tikzpicture}
\draw (0,0) parabola (1,2);
\draw (2,0) parabola
bend (2.25,-0.25) (3,2);
\draw (4,0) parabola
bend (4.75,2.25) (5,2);
\end{tikzpicture}
\end{document}
```



#### 函数曲线

用 `domain` 参数控制定义域，由 `plot` 参数指定表达式

```latex
\begin{document}
\begin{tikzpicture}
\draw [domain=-1:1] plot(\x,{\x*\x*2 -1});
\end{tikzpicture}
\end{document}
```



#### 网格路径

使用 `grid` 绘制网格，通过 `step` 参数指定间隔，同时指定样式

```latex
\begin{document}
\begin{tikzpicture}
\draw [step=0.5,gray,very thin] (-1.4,-1.4) grid (1.4,1.4);
\end{tikzpicture}
\end{document}
```



#### 裁剪路径

可以用 `\clip` 指定路径用于裁剪图像

```latex
\begin{document}
\begin{tikzpicture}
\clip[draw] (0.0,0.0) circle (1.6cm);
\draw [step=0.5,gray,very thin] (-1.4,-1.4) grid (1.4,1.4);
\end{tikzpicture}
\end{document}
```

这里 `[draw]` 参数表示绘制裁剪路径，如果省去则不会出现上面的圆形边框。

> 事实上 `\draw` 是 `\path[draw]` 的简写，`\clip` 是 `\path[clip]` 的简写。



#### 填充路径

使用 `\fill` 命令填充封闭路径

```latex
\begin{document}
\begin{tikzpicture}
\fill [green!20!white] (0,0) -- (3,0) arc (0:30:3) -- cycle;
\end{tikzpicture}
\end{document}
```

其中 `[green!20!white]` 表示 20% 的绿色和 80% 的白色混合。



使用 `\filldraw` 命令在填充的同时绘制边界

```latex
\begin{document}
\begin{tikzpicture}[thick]
\draw [blue] (0,0) rectangle (1,1);
\filldraw [fill=yellow,draw=red] (2,0.5) circle [radius=0.5];
\end{tikzpicture}
\end{document}
```



### 绘图样式
#### 定义样式

可以定义和修改样式

```tex
\tikzstyle help lines=[very thin]		% 定义样式
\tikzstyle help lines+=[color=blue!50]	% 添加样式：50% 的蓝色
[help lines/.style={blue,thick,->}]		% 定义样式
```

例如自定义样式

```latex
\begin{document}
\tikzstyle Karl's grid=[style=help lines,color=blue!50]

\begin{tikzpicture}
\draw [step=0.5,style=Karl's grid] (0,0) grid (2.5,2.5);
\end{tikzpicture}
\end{document}
```

其中 `style=` 可以省略，tikz 会自动检查一个未知选项是否是样式名。



#### 线型样式

线型风格主要包括

| 样式               | 线型   | 样式               | 线型   |
| :--------------- | :--- | :--------------- | :--- |
| `very thin`      | 较细   | `thin`           | 细    |
| `ultra thin`     | 极细   | `very thick`     | 非常粗  |
| `thick`          | 粗    | `ultra thick`    | 极粗   |
| `semithick`      | 中等   | `dashed`         | 虚线   |
| `dotted`         | 点线   | `loosely dashed` | 稀疏虚线 |
| `densely dashed` | 密集虚线 | `loosely dotted` | 稀疏点线 |
| `densely dotted` | 密集点线 |                  |      |

指定线条的粗细

```latex
\begin{document}
\begin{tikzpicture}
\draw[ultra thin] (0,0)--(0,2);
\draw[very thin] (0.5,0)--(0.5,2);
\draw[thin] (1,0)--(1,2);
\draw[semithick] (1.5,0)--(1.5,2);
\draw[thick] (2,0)--(2,2);
\draw[very thick] (2.5,0)--(2.5,2);
\draw[ultra thick] (3,0)--(3,2);
\end{tikzpicture}
\end{document}
```

指定线条类型（实线、虚线、点划线等）

```latex
\begin{document}
\begin{tikzpicture}
\draw[dashed] (0,0) -- (0,2);
\draw[dotted] (0.5,0) -- (0.5,2);
\draw[dash dot] (1,0) -- (1,2);
\draw[dash dot dot] (1.5,0) -- (1.5,2);
\draw[densely dotted]
(2,0) -- (3,2) -- (4,0) -- cycle;
\end{tikzpicture}
\end{document}
```



#### 箭头样式

通过 `[]` 参数指定路径首尾的箭头样式。例如

```latex
\begin{document}
\begin{tikzpicture}[thick]
\draw[->] (0,4) -- (3,4);
\draw[->>] (0,3.5) -- (3,3.5);
\draw[->|] (0,3) -- (3,3);
\draw[<-] (0,2.5) -- (3,2.5);
\draw[<->] (0,2) -- (3,2) -- (3,2.25);
\draw[>->|] (0,1.5) -- (3,1.5);
\draw[-stealth] (0,1) -- (3,1);
\draw[-latex] (0,0.5) -- (3,0.5);
\draw[-to] (0,0) -- (3,0);
\end{tikzpicture}
\end{document}
```

复杂的箭头形式需要在导言区使用 ` \usetikzlibrary{arrows.meta}`，例如指定 `>=stealth` 绘制特殊的箭头尖端

```latex
\begin{document}
\begin{tikzpicture}[thick]
\draw[<<-,>=stealth] (0,4) -- (3,4);
\draw[<<-] (0,4.5) -- (3,4.5);
\end{tikzpicture}
\end{document}
```



#### 圆角样式

参数 `radius` 控制圆角的半径。可以对某一段路径直接使用

```latex
\begin{document}
\begin{tikzpicture}[thick]
\draw [rounded corners] (0,0) rectangle (1,1);
\draw (2,0) -- (2,1) [rounded corners=.3cm] -- (3,1) -- (3.5,0) [sharp corners] -- cycle;
\end{tikzpicture}
\end{document}
```



#### 作用域

如果希望某个样式对一些路径有效，而对另一些路径使用不同的样式，就可以借助作用域

```latex
\begin{document}
\begin{tikzpicture}[ultra thick]
\draw (0,0) -- (0,1);
\begin{scope}[thin]
\draw (1,0) -- (1,1);
\draw (2,0) -- (2,1);
\end{scope}
\draw (3,0) -- (3,1);
\end{tikzpicture}
\end{document}
```



### 坐标系

TikZ 用直角坐标系或者极坐标系描述点的位置

* 直角坐标下，点的位置写作 `(x,y)`，坐标可以用 LATEX 支持的任意单位表示，默认为 cm
* 极坐标下，点的位置写作 `(θ:r)`。θ 为极角，单位是度

通过 `x,y` 参数指定坐标轴的单位长度。



#### 相对位置

通过 `+` 号设定增量坐标，例如

```latex
\begin{document}
\begin{tikzpicture}
\draw [red,very thick] (30:1) -- +(0,-0.5);
\end{tikzpicture}
\end{document}
```

通过 `+(0,-0.5)` 指定下一个点的位置是**前一个指定位置**向左偏移 `0.5` 后的位置。



通过 `++` 在指定偏移量的同时，将偏移后的位置作为新的指定位置。例如

```latex
\begin{document}
\begin{tikzpicture}
\def\incpath{-- ++(1,0) -- ++(0,1) -- ++(-1,0) -- cycle}
\def\relpath{-- +(1,0) -- +(1,1) -- +(0,1) -- cycle}
\draw (0,0) \incpath;
\draw (1.5,0) \relpath;
\end{tikzpicture}
\end{document}
```

其中定义了 `\incpath, \relpath` 两个新的指令，分别通过 `++, +` 实现。注意到 `++` 每次的增量都是相对于前一个点，而 `+` 的增量相对于第一个点。这就是由于 `++` 会将增量后的点作为新的相对位置，而 `+` 只会将最后一个显式指定的位置作为相对位置。



#### 交点位置

使用语法 `(p |- q)` 表示经过 `p` 的竖线与经过 `q` 的横线的交点，而 `(p -| q)` 则恰好相反。例如

```latex
\begin{document}
\begin{tikzpicture}
\draw[red] (0,0) -- (0,0 -| 2,2);
\draw[blue] (0,0) -- (0,0 |- 2,2);
\end{tikzpicture}
\end{document}
```

通过 `intersection of ... and` 指定两条线的交点位置

```latex
\begin{document}
\begin{tikzpicture}
\draw [very thick,orange] (1,0) -- (intersection of 1,0--1,1 and 0,0--30:1);
\draw [very thick,blue] (0,0) -- (intersection of 1,0--1,1 and 0,0--30:1);
\end{tikzpicture}
\end{document}
```



#### 坐标点

可以用 `\coordinate` 为某个坐标命名，也可以在路径中的点后面使用 `coordinate` 命名，便于之后使用

```latex
\begin{document}
\begin{tikzpicture}
\draw (0,0) -- (30:1);
% 命名坐标位置 (2,1) 为 A
\draw (1,0) -- (2,1) coordinate (A);
% 命名坐标位置 (0,1) 为 S
\coordinate (S) at (0,1);
\draw (S) -- (1,1);
\end{tikzpicture}
\end{document}
```



#### 坐标变换

借助 `xshift, yshift` 参数可以对路径做偏移

```latex
\begin{document}
\begin{tikzpicture}[x=10pt,y=50pt,thick]
\draw (0,0) -- (0,0.5) [xshift=2pt] (0,0) -- (0,0.5);
\end{tikzpicture}
\end{document}
```

在 `[]` 中的参数偏移了它后面的路径。




下面是一个更复杂的例子，分别使用 `shift, rotate` 变换它后面的路径。样式使用 `even odd rule` 实现嵌套间隔填充背景

```latex
\begin{document}
\begin{tikzpicture}[even odd rule,rounded corners=2pt,x=10pt,y=10pt]
\filldraw [fill=green] (0,0) rectangle (1,1)
		[xshift=5pt,yshift=5pt] (0,0) rectangle (1,1)
		[rotate=30] (-1,-1) rectangle (2,2);
\end{tikzpicture}
\end{document}
```

类似的还有 `xscale, yscale` 伸缩变换和 `xslant, yslant` 剪切变换

```latex
\begin{document}
\begin{tikzpicture}
\draw[help lines](0,0) rectangle (1,1);
\draw[scale=1.5] (0,0) rectangle (1,1);
\draw[rotate=30] (0,0) rectangle (1,1);
\draw[help lines](2,0) rectangle (3,1);
\draw[yshift=4pt](2,0) rectangle (3,1);
\draw[help lines](4,0) rectangle (5,1);
\draw[xslant=0.4](4,0) rectangle (5,1);
\end{tikzpicture}
\end{document}
```



### 循环绘图

TikZ 通过 pgffor 功能宏包实现了简单的循环功能。例如标记坐标单位

```latex
\begin{document}
\begin{tikzpicture}
\draw [->] (-1.5,0) -- (1.5,0);
\draw [->] (0,-1.5) -- (0,1.5);
\foreach \x in {-1cm,-0.5cm,1cm}
{ 
	\draw (\x,-1pt) -- (\x,1pt);
}
\foreach \y in {-1cm,-0.5cm,0.5cm,1cm}
{ 
	\draw (-1pt,\y) -- (1pt,\y);
}
\end{tikzpicture}
\end{document}
```

可以用 `...` 来省略中间数值

```latex
\begin{document}
\begin{tikzpicture}
\foreach \x in {1,...,10}
{ 
	\draw (\x,0) circle (0.4cm);
}
\end{tikzpicture}
\end{document}
```

指定前两个数值来设定步长

```latex
\begin{document}
\begin{tikzpicture}
\foreach \x in {-1,-0.5,...,1}
{ 
	\draw (\x cm,-1pt) -- (\x cm,1pt);
}
\end{tikzpicture}
\end{document}
```

允许嵌套循环以及在循环值中插入其它值来修改循环

```latex
\begin{document}
\begin{tikzpicture}
% 5,7,8 这里跳过了 6
\foreach \x in {1,2,...,5,7,8,...,12}
{ 
	\foreach \y in {1,...,5}
	{
		\draw (\x,\y) +(-0.5,-0.5) rectangle ++(0.5,0.5);
		\draw (\x,\y) node{\x,\y};
	}
}
\end{tikzpicture}
\end{document}
```

可使用变量对参与循环

```latex
\begin{document}
\begin{tikzpicture}
% \n, \t 是两个变量
\foreach \n/\t in {0/\alpha,1/\beta,2/\gamma}
{
	\node [circle,fill=lightgray,draw] at (\n,0) {$\t$};
}
\end{tikzpicture}
\end{document}
```

这种变量对常用于绘制坐标单位

```latex
\begin{document}
\begin{tikzpicture}
\draw [->] (-1.5,0) -- (1.5,0);
\foreach \x/\xtext in {-1, -0.5/-\frac{1}{2}, 1}
{
	\draw (\x cm,1pt) -- (\x cm,-1pt) node [anchor=north] { $\xtext$ };
}
\end{tikzpicture}
\end{document}
```


### 操作节点

TikZ 允许在路径中间添加 `node` 节点，在指定位置放置文本并设置样式

```latex
\begin{document}
\begin{tikzpicture}
\draw (0,0) rectangle (2,2);
\draw (0.5,0.5) node [fill=green] {Text at \verb!node 1!} -- (1.5,1.5) node {Text at \verb!node 2!};
\end{tikzpicture}
\end{document}
```

也可以用 `\node` 命令指定节点

```latex
\begin{document}
\begin{tikzpicture}
\node[draw] at (0,0) {$E=mc^2$};
\end{tikzpicture}
\end{document}
```

可以渲染 LaTex 数学文本，只需要将文本用 `$` 包围。



#### 节点位置

如果希望将文本放在位置的指定方位，可以使用 `anchor, below` 参数

```latex
\begin{document}
\begin{tikzpicture}
\coordinate (A) at (1,1);
\fill (A) circle[radius=2pt];
\node[draw,anchor=south] at (A) {a};
\node[draw,below right=4pt] at (A) {b};
\end{tikzpicture}
\end{document}
```

注意到 `south` 表示节点位置的下方是指定点（实心点），这与直观上的方位不同，因此更建议使用 `centered/below/above/right/left` 参数。



给出一个较为复杂的例子

```latex
\begin{document}
\begin{tikzpicture}[scale=3]
\clip (-2,-0.2) rectangle (2,0.8);
\draw [step=0.5cm,gray,very thin] (-1.4,-1.4) grid (1.4,1.4);
\filldraw [fill=green!20,draw=green!50!black] (0,0) -- (3mm,0mm) arc (0:30:3mm) -- cycle;

% 定义两个坐标点 x axis 和 y axis
\draw [->] (-1.5,0) -- (1.5,0) coordinate (x axis);
\draw [->] (0,-1.5) -- (0,1.5) coordinate (y axis);
\draw (0,0) circle (1cm);

\draw [very thick,red] (30:1cm) -- node [left=1pt,fill=gray!20] {$\sin\alpha$} (30:1cm |- x axis);
\draw [very thick,blue] (30:1cm |- x axis) -- node [below=2pt,fill=gray!20] {$\cos\alpha$} (0,0);
\draw [very thick,orange] (1,0) -- node [right=1pt,fill=gray!20] 
{
	% 这一部分是 LaTex 文本
	$\displaystyle \tan\alpha \color{black}=\frac{{\color{red}\sin\alpha}}{\color{blue}\cos\alpha}$
} (intersection of 0,0--30:1cm and 1,0--1,1) coordinate (t);

\draw (0,0) -- (t);

\foreach \x/\xtext in {-1, -0.5/-\frac{1}{2}, 1}
{
	\draw (\x cm,1pt) -- (\x cm,-1pt) node [anchor=north,fill=gray!20] { $\xtext$ };
}
\foreach \y/\ytext in {-1, -0.5/-\frac{1}{2}, 0.5/\frac{1}{2}, 1}
{
	\draw (1pt,\y cm) -- (-1pt,\y cm) node [anchor=east,fill=gray!20] { $\ytext$ };
}
\end{tikzpicture}
\end{document}
```



还可以将文本标签写在曲线上，通过 `near start/near end` 来控制位置，通过 `above/below` 控制在曲线的上下方，加入 `sloped` 使文本沿曲线切向

```latex
\begin{document}
\begin{tikzpicture}
\draw (0,0) .. controls (6,1) and (9,1) .. 
node [near start,sloped,above] {near start}
node {midway}
node [very near end,sloped,below] {very near end} (12,0);
\end{tikzpicture}
\end{document}
```



#### 节点形状

通过 `shape` 参数修改节点形状。通过 `draw` 参数指定节点在路径完成后绘制

```latex
\begin{document}
\begin{tikzpicture}
\path (0,2) node [shape=circle,draw] {}
	(0,1) node [shape=circle,draw] {}
	(0,0) node [shape=circle,draw] {}
	(1,1) node [shape=rectangle,draw] {}
	(-1,1) node [shape=rectangle,draw] {};
\end{tikzpicture}
\end{document}
```




#### 放置节点

通过路径指定节点较为麻烦，可以使用 `\path node` 的缩写 `\node` 来放置节点

```tex
\begin{document}
\begin{tikzpicture}
\node at (0,2) [circle,draw] {};
\node at (0,1) [circle,draw] {};
\node at (0,0) [circle,draw] {};
\node at (1,1) [rectangle,draw] {};
\node at (-1,1) [rectangle,draw] {};
\end{tikzpicture}
\end{document}
```



#### 节点样式

通过 `fill` 参数指定填充颜色，`draw` 参数指定边框颜色。为了让节点含义更清楚，将样式指定名称后使用

```latex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick]

\begin{tikzpicture}
\node at (0,2) [place] {};
\node at (0,1) [place] {};
\node at (0,0) [place] {};
\node at (1,1) [transition] {};
\node at (-1,1) [transition] {};
\end{tikzpicture}
\end{document}
```



#### 节点大小

节点具有默认的空格，即使没有填写文本，也具有一定大小。通过 `inner sep` 参数指定

```latex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick]

\begin{tikzpicture}[inner sep=2mm]
\node at (0,2) [place] {};
\node at (0,1) [place] {};
\node at (0,0) [place] {};
\node at (1,1) [transition] {};
\node at (-1,1) [transition] {};
\end{tikzpicture}
\end{document}
```

不过更建议使用 `minmum size` 参数，确保节点具有相同的最小大小。同时将 `inner sep` 设为 0，确保空节点确实达到最小大小

```tex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

\begin{tikzpicture}
\node at (0,2) [place] {};
\node at (0,1) [place] {};
\node at (0,0) [place] {};
\node at (1,1) [transition] {};
\node at (-1,1) [transition] {};
\end{tikzpicture}
\end{document}
```



#### 节点命名

通过括号指定节点名称，由于节点操作的语法顺序可交换，因此重新排列

```tex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

\begin{tikzpicture}
\node [place] 		(waiting 1) 		at (0,2)  	{};
\node [place] 		(critical 1) 		at (0,1) 	{};
\node [place] 		(semaphore) 		at (0,0) 	{};
\node [transition] 	(leave critical) 	at (1,1) 	{};
\node [transition] 	(enter critical) 	at (-1,1) 	{};
\end{tikzpicture}
\end{document}
```



#### 相对位置

通过节点之间的相对位置放置能够更容易确定节点关系

```tex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

\begin{tikzpicture}
\node [place] 		(waiting) 									{};
\node [place] 		(critical) 			[below of=waiting] 		{};
\node [place] 		(semaphore) 		[below of=critical] 	{};
\node [transition] 	(leave critical) 	[right of=critical] 	{};
\node [transition] 	(enter critical) 	[left of=critical] 		{};
\end{tikzpicture}
\end{document}
```



#### 添加标签

通用方法是利用节点的相对位置，指定 `semaphore.north` 表示节点位于 `semaphore` 的北方

```latex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

\begin{tikzpicture}
\node [place] 		(waiting) 									{};
\node [place] 		(critical) 			[below of=waiting] 		{};
\node [place] 		(semaphore) 		[below of=critical] 	{};
\node [transition] 	(leave critical) 	[right of=critical] 	{};
\node [transition] 	(enter critical) 	[left of=critical] 		{};

\node [red,above] at (semaphore.north) {$s\le 3$};
\end{tikzpicture}
\end{document}
```

另一种等价的方式是直接在节点参数中指定 `label` 参数

```tex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

\begin{tikzpicture}
\node [place] 		(waiting) 									{};
\node [place] 		(critical) 			[below of=waiting] 		{};
\node [place] 		(semaphore) 		[below of=critical,
										label=above:$s\le3$] 	{};
\node [transition] 	(leave critical) 	[right of=critical] 	{};
\node [transition] 	(enter critical) 	[left of=critical] 		{};
\end{tikzpicture}
\end{document}
```

可以指定多个 `label` 参数

```latex
\begin{document}
\begin{tikzpicture}
\node [circle,draw,label=60:$60^\circ$,label=below:$-90^\circ$] {my circle};
\end{tikzpicture}
\end{document}
```

其中 `60` 表示逆时针转动 60 度后的位置。



#### 标签颜色

可以重定义标签颜色，语法格式 `label=[color]` 会导致 `[]` 嵌套，因此需要用 `label={[color]}` 替代

```latex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

\begin{tikzpicture}
\node [place] 		(waiting) 									{};
\node [place] 		(critical) 			[below of=waiting] 		{};
\node [place] 		(semaphore) 		[below of=critical,
										label={[blue]above:$s\le3$}] 	{};
\node [transition] 	(leave critical) 	[right of=critical] 	{};
\node [transition] 	(enter critical) 	[left of=critical] 		{};
\end{tikzpicture}
\end{document}
```

可以选择直接重定义 `every label` 样式

```tex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

\begin{tikzpicture}
% 重定义 every label
\tikzstyle{every label}=[blue]
\node [place] 		(waiting) 									{};
\node [place] 		(critical) 			[below of=waiting] 		{};
\node [place] 		(semaphore) 		[below of=critical,
										label=above:$s\le3$] 	{};
\node [transition] 	(leave critical) 	[right of=critical] 	{};
\node [transition] 	(enter critical) 	[left of=critical] 		{};
\end{tikzpicture}
\end{document}
```



#### 连接节点

我们从 `enter critical` 节点的右侧连接到 `critical` 节点的左侧，然后将 `waiting` 的左侧用一个 Bezier 曲线连接到 `enter critical` 节点的上方

```latex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

\begin{tikzpicture}
\node [place] 		(waiting) 									{w};
\node [place] 		(critical) 			[below of=waiting] 		{c};
\node [place] 		(semaphore) 		[below of=critical] 	{s};
\node [transition] 	(leave critical) 	[right of=critical] 	{lc};
\node [transition] 	(enter critical) 	[left of=critical] 		{ec};

\draw [->] (enter critical.east) -- (critical.west);
\draw [->] (waiting.west) .. controls +(left:5mm) and +(up:5mm) .. (enter critical.north);

% 绘制控制点
\filldraw [gray] (waiting.west) circle (2pt);
\filldraw [gray] (waiting.west)+(left:5mm) circle (2pt);
\filldraw [gray] (enter critical.north)+(up:5mm) circle (2pt);
\filldraw [gray] (enter critical.north) circle (2pt);
\end{tikzpicture}
\end{document}
```

注意到每个节点的 4 个方位点都可以绘制出来，并且相对偏移 `+(left:5mm)` 都可以参与运算。上述代码可以简化

```tex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

\begin{tikzpicture}
\node [place] 		(waiting) 									{w};
\node [place] 		(critical) 			[below of=waiting] 		{c};
\node [place] 		(semaphore) 		[below of=critical] 	{s};
\node [transition] 	(leave critical) 	[right of=critical] 	{lc};
\node [transition] 	(enter critical) 	[left of=critical] 		{ec};

\draw [->] (enter critical) -- (critical);
\draw [->] (waiting) .. controls +(left:8mm) and +(up:8mm) .. (enter critical);
\end{tikzpicture}
\end{document}
```

节点方位可由 TikZ 自动推导得到。更简化的版本是

```tex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

\begin{tikzpicture}
\node [place] 		(waiting) 									{w};
\node [place] 		(critical) 			[below of=waiting] 		{c};
\node [place] 		(semaphore) 		[below of=critical] 	{s};
\node [transition] 	(leave critical) 	[right of=critical] 	{lc};
\node [transition] 	(enter critical) 	[left of=critical] 		{ec};

\draw [->] (enter critical) to 					(critical);
\draw [->] (waiting) 		to [out=180,in=90] 	(enter critical);
\end{tikzpicture}
\end{document}
```

使用 `to` 指定连接关系，参数 `out,in` 分别指定起点和终点的切向角度。



使用 `bend` 参数指定节点弯曲方向角

```latex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

\begin{tikzpicture}
\node [place] 		(waiting) 									{w};
\node [place] 		(critical) 			[below of=waiting] 		{c};
\node [place] 		(semaphore) 		[below of=critical] 	{s};
\node [transition] 	(leave critical) 	[right of=critical] 	{lc};
\node [transition] 	(enter critical) 	[left of=critical] 		{ec};

\draw [->] (enter critical) to 					(critical);
\draw [->] (waiting) 		to [bend right=45] 	(enter critical);
\draw [->] (enter critical)	to [bend right=45] 	(semaphore);
\end{tikzpicture}
\end{document}
```

其中 `[bend right=45] 	(semaphore)` 使得连接 `semaphore` 的曲线向右弯折。



#### 边路径操作

在节点路径上添加边可以直接连接其它节点

```latex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

\begin{tikzpicture}
\node [place] 		(waiting) 									{w};
\node [place] 		(critical) 			[below of=waiting] 		{c};
\node [place] 		(semaphore) 		[below of=critical] 	{s};
\node [transition] 	(leave critical) 	[right of=critical] 	{lc};
\node [transition] 	(enter critical) 	[left of=critical] 		{ec}
	edge [->]				(critical)
	edge [<-,bend left=45] 	(waiting)
	edge [->,bend right=45]	(semaphore);
\end{tikzpicture}
\end{document}
```

每条边 `edge` 都会构造一条新的路径，连接起始节点和目标节点。



最后我们将连接样式封装为 `pre, post` 并指定 `bend angle` 得到最终结果

```latex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

% 连接样式
\tikzstyle{pre}=[<-,shorten <=1pt,>=stealth,semithick]
\tikzstyle{post}=[->,shorten >=1pt,>=stealth,semithick]

\begin{tikzpicture}[bend angle=45]
\node [place] 		(waiting) 									{w};
\node [place] 		(critical) 			[below of=waiting] 		{c};
\node [place] 		(semaphore) 		[below of=critical] 	{s};

\node [transition] 	(leave critical) 	[right of=critical] 	{lc}
	edge [pre]				(critical)
	edge [post,bend right]	(waiting)
	edge [pre,bend left]	(semaphore);
\node [transition] 	(enter critical) 	[left of=critical] 		{ec}
	edge [post]				(critical)
	edge [pre,bend left] 	(waiting)
	edge [post,bend right]	(semaphore);
\end{tikzpicture}
\end{document}
```



#### 连线标签

通过添加 `auto` 边界，TikZ 自动在连线上定位节点。使用 `swap` 指定两个节点沿曲线镜像放置

```latex
\begin{document}
\begin{tikzpicture}[auto,bend right]
\node (a) at (0:1) {$0^\circ$};
\node (b) at (120:1) {$120^\circ$};
\node (c) at (240:1) {$240^\circ$};

% 连接 (a) (b) 的路径上增加 node {1} 和 node {1'}，它们通过 [swap] 对称摆放
\draw 	(a) to node {1} node [swap] {1'} (b)
		(b) to node {2} node [swap] {2'} (c)
		(c) to node {3} node [swap] {3'} (a);
\end{tikzpicture}
\end{document}
```

过程中新节点放在 `to` 指定的方向上，自动选项使新节点移动到曲线旁边。



标签节点放置在 `edge` 路径上

```latex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

% 连接样式
\tikzstyle{pre}=[<-,shorten <=1pt,>=stealth,semithick]
\tikzstyle{post}=[->,shorten >=1pt,>=stealth,semithick]

\begin{tikzpicture}[bend angle=45]
\node [place] 		(waiting) 									{w};
\node [place] 		(critical) 			[below of=waiting] 		{c};
\node [place] 		(semaphore) 		[below of=critical] 	{s};

\node [transition] 	(leave critical) 	[right of=critical] 	{lc}
	edge [pre]										(critical)
	% 中间加入一个 node
	edge [post,bend right]	node [auto,swap] {2}	(waiting)
	edge [pre,bend left]							(semaphore);
\node [transition] 	(enter critical) 	[left of=critical] 		{ec}
	edge [post]				(critical)
	edge [pre,bend left] 	(waiting)
	edge [post,bend right]	(semaphore);
\end{tikzpicture}
\end{document}
```



#### 蛇形路径

设置 `snake` 参数将路径修改为蛇形，并可指定更多样式

```latex
\begin{document}
\begin{tikzpicture}
\draw [->,snake=snake] (0,1) -- (2,1);
\draw [->,snake=snake,
		segment amplitude=0.4mm,
		segment length=2mm,
		line after snake=1mm] (0,0) -- (3,0);
\end{tikzpicture}
\end{document}
```

要在蛇形路径上排列多行文本，需要指定文本节点宽度和居中

```latex
\begin{document}
\begin{tikzpicture}
\draw [->,snake=snake,
		segment amplitude=0.4mm,
		segment length=2mm,
		line after snake=1mm] (0,0) -- (3,0)
	node [above,text width=3cm,text centered,midway]
	{
		replacement of the \textcolor{red}{capacity} by
		\textcolor{red}{two places}
	};
\end{tikzpicture}
\end{document}
```

由于文本长度超出给定宽度，因此得到多行文本效果。



#### 图层

通过 `pgfonlayer` 环境设置图层环境。我们通常希望在绘图完成后添加背景，避免提前计算背景矩形的位置尺寸。借助节点位置，可以很容易构造出背景矩形所需的位置参数

```latex
\begin{document}
\tikzstyle{place}=[circle,draw=blue!50,fill=blue!20,thick,inner sep=0pt,minimum size=6mm]
\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

% 连接样式
\tikzstyle{pre}=[<-,shorten <=1pt,>=stealth,semithick]
\tikzstyle{post}=[->,shorten >=1pt,>=stealth,semithick]

\begin{tikzpicture}[bend angle=45]
\node [place] 		(waiting) 									{w};
\node [place] 		(critical) 			[below of=waiting] 		{c};
\node [place] 		(semaphore) 		[below of=critical] 	{s};

\node [transition] 	(leave critical) 	[right of=critical] 	{lc}
	edge [pre]										(critical)
	% 中间加入一个 node
	edge [post,bend right]	node [auto,swap] {2}	(waiting)
	edge [pre,bend left]							(semaphore);
\node [transition] 	(enter critical) 	[left of=critical] 		{ec}
	edge [post]				(critical)
	edge [pre,bend left] 	(waiting)
	edge [post,bend right]	(semaphore);

% 背景图层
\begin{pgfonlayer}{background}
\filldraw [fill=black!30,draw=red]
	(semaphore.south -| enter critical.west) rectangle (waiting.north -| leave critical.east);
\end{pgfonlayer}
\end{tikzpicture}
\end{document}
```



#### Petri 网

Petri 网绘制库提供了类似于前面的定义、风格，例如 `place, pre, post` 等，还定义了 `tokens` 作为节点内部的实心点样式。

```latex
\begin{document}
\tikzstyle{every place}=		[minimum size=6mm,thick,draw=blue!75,fill=blue!20]
\tikzstyle{every transition}=	[thick,draw=black!75fill=black!20]
\tikzstyle{red place}=			[place,draw=red!75,fill=red!20]
\tikzstyle{every label}=		[red]

\tikzstyle{transition}=[rectangle,draw=black!50,fill=black!20,thick,inner sep=0pt,minimum size=4mm]

\begin{tikzpicture}[node distance=1.3cm,>=stealth,bend angle=45,auto]

% 构造节点和位置
\node [place,tokens=1] 	(w1)										{};
\node [place] 			(c1) 	[below of=w1]						{};
\node [place] 			(s) 	[below of=c1,label=above:$s\le 3$]	{};
\node [place]			(c2) 	[below of=s]						{};
\node [place,tokens=1] 	(w2) 	[below of=c2]						{};

\node [transition] (e1) [left of=c1] {}
	edge [pre,bend left]					(w1)
	edge [post,bend right]					(s)
	edge [post]								(c1);
\node [transition] (e2) [left of=c2] {}
	edge [pre,bend right]					(w2)
	edge [post,bend left]					(s)
	edge [post]								(c2);
\node [transition] (l1) [right of=c1] {}
	edge [pre]								(c1)
	edge [pre,bend left]					(s)
	edge [post,bend right] node [swap] {2}	(w1);
\node [transition] (l2) [right of=c2] {}
	edge [pre]								(c2)
	edge [pre,bend right]					(s)
	edge [post,bend left] node {2}			(w2);

% 平移后的绘制新的图
\begin{scope}[xshift=6cm]

% 构造节点和位置
\node [place,tokens=1] 		(w1')														{};
\node [place] 				(c1') 	[below of=w1']										{};
\node [red place] 			(s1') 	[below of=c1',xshift=-5mm] [label=left:$s$]			{};
\node [red place,tokens=3] 	(s2') 	[below of=c1',xshift=5mm] [label=right:$\bar s$]	{};
\node [place]				(c2') 	[below of=s1',xshift=5mm]							{};
\node [place,tokens=1] 		(w2') 	[below of=c2']										{};

\node [transition] (e1') [left of=c1'] {}
	edge [pre,bend left]					(w1')
	edge [post]								(s1')
 	edge [pre]								(s2')
	edge [post]								(c1');
\node [transition] (e2') [left of=c2'] {}
	edge [pre,bend right]					(w2')
	edge [post]								(s1')
 	edge [pre]								(s2')
	edge [post]								(c2');
\node [transition] (l1') [right of=c1'] {}
	edge [pre]								(c1')
	edge [pre]								(s1')
 	edge [post]								(s2')
	edge [post,bend right] node [swap] {2}	(w1');
\node [transition] (l2') [right of=c2'] {}
	edge [pre]								(c2')
	edge [pre]								(s1')
	edge [post]								(s2')
 	edge [post,bend left] node {2}			(w2');

\end{scope}

% 绘制蛇形线
\draw [-to,thick,snake=snake,segment amplitude=0.4mm,segment length=2mm,line after snake=1mm]
	% 节点 s 右移位置，节点 s1' 左移位置
	([xshift=5mm]s -| l1) -- ([xshift=-5mm]s1' -| e1')
	node [above=1mm,midway,text width=3cm,text centered]
	{
		replacement of the \textcolor{red}{capacity} by \textcolor{red}{two places}
 	};

% 背景图层
\begin{pgfonlayer}{background}
\filldraw [line width=4mm,join=round,black!10]
	(w1.north -| l1.east) rectangle (w2.south -| e1.west)
	(w1'.north -| l1'.east) rectangle (w2'.south -| e1'.west);
\end{pgfonlayer}

\end{tikzpicture}

\end{document}
```



## 自定义命令

自定义宏包的好处在于，一旦想要修改命令代码的样式，比如更换颜色、加边框等等，可以通过改变环境的定义来很容易地创建新的外观，而不是挨个修改每个命令示例。



#### 定义命令

使用如下命令可以定义你自己的命令

```tex
\newcommand{\⟨name⟩}[⟨num⟩]{⟨definition⟩}
```

需要两个必选参数，参数 `⟨name⟩` 是要定义的命令名称（带反斜线），参数 `⟨definition⟩` 是命令的具体定义。参数 `⟨num⟩` 是可选的，用于指定新命令所需的参数数目（最多 9 个）。如果缺省可选参数，默认就是 0，也就是新定义的命令不带任何参数。



定义一个新的命令

```latex
\newcommand{\tnss}{The not so Short Introduction to \LaTeXe}

\begin{document}
% 直接使用 \tnss 就可以输出上面文字
This is ``\tnss'' \ldots{} ``\tnss''
\end{document}
```

在命令的定义中，标记 `#1` 代表指定的参数。如果想使用多个参数，可以依次使用 `#2,...,#9` 等标记

```tex
\newcommand{\txsit}[1]{This is the \emph{#1} Short Introduction to \LaTeXe}

\begin{document}
% in the document body:
\begin{itemize}
\item \txsit{not so}
\item \txsit{very}
\end{itemize}
\end{document}
```

LATEX 不允许使用 `\newcommand` 定义一个与现有命令重名的命令。如果需要修改命令定义的话，使用 `\renewcommand` 命令。

> 在某些情况之下，使用 `\providecommand` 命令是一种比较理想的方案：在命令未定义时，它相当于 \newcommand；在命令已定义时，沿用已有的定义。



#### 定义环境

可以用 `\newenvironment` 定义新的环境。它的语法如下所示

```tex
\newenvironment{⟨name⟩}[⟨num⟩]{⟨before⟩}{⟨after⟩}
```

同样地，`\newenvironment` 命令有一个可选的参数。在 `⟨before⟩` 中的内容将在此环境包含的文本之前处理，而在 `⟨after⟩` 中的内容将在遇到 `\end{⟨name⟩}` 命令时处理。例如

```latex
\newenvironment{king}{\rule{1ex}{1ex}}

\begin{document}
\begin{king}
	My humble subjects \ldots
\end{king}
\end{document}
```

参数 `⟨num⟩` 的使用方式与 `\newcommand` 命令相同。LATEX 还同样保证你不会不小心新建重名的环境。如果你确实希望改变一个现有的环境，你可以使用命令 `\renewenvironment`，它使用和命令 `\newenvironment` 相同的语法。



#### xparse 宏包

通过 `\newcommand` 和 `\newenvironment` 定义的命令或环境格式比较固定。如果需要定义带有多个可选参数、或者带星号的命令或环境，可以使用 xparse 宏包。它提供了 `\NewDocumentCommand` 和 `\NewDocumentEnvironment` 等命令，具体语法如下

```tex
\NewDocumentCommand\⟨name⟩{⟨arg spec⟩}{⟨definition⟩}
\NewDocumentEnvironment{⟨name⟩}{⟨arg spec⟩}{⟨before⟩}{⟨after⟩} 
```

> 此定义需要放在 `\begin{document}` 和 `\end{document}` 之间。



xparse 通过 `{⟨arg spec⟩}` 来指定参数的个数和格式，其中的空格可以忽略

![](image-20220715233745041.png|500)

不同输入值在解析后的结果如下

![](image-20220715233829531.png|500)

-NoValue- 标记可以用 `\IfNoValueTF` 等命令来判断

```tex
\IfNoValueTF{⟨argument⟩}{⟨true code⟩}{⟨false code⟩}
\IfNoValueT{⟨argument⟩}{⟨true code⟩}
\IfNoValueF{⟨argument⟩}{⟨false code⟩}
```

举例如下

```latex
\begin{document}
% 百分号用于注释掉不必要的空格和换行符
\NewDocumentCommand\hello{om}
{%
\IfNoValueTF{#1}%
{Hello, #2!}%
{Hello, #1 and #2!}%
}

\hello{Alice}
\hello[Bob]{Alice}
\end{document}
```



可以用 `\IfBooleanTF` 等命令来判断 `\BooleanTrue, \BooleanFalse`

```tex
\IfBooleanTF{⟨argument⟩}{⟨true code⟩}{⟨false code⟩}
\IfBooleanT{⟨argument⟩}{⟨true code⟩}
\IfBooleanF{⟨argument⟩}{⟨false code⟩}
```

举例如下

```latex
\begin{document}
\NewDocumentCommand\hereis{sm}
{Here is \IfBooleanTF{#1}{an}{a} #2.}

\hereis{banana}
\hereis*{apple}
\end{document}
```



需要注意的是，与命令不同，环境在定义时名字里面可以包含 `*`

```tex
\NewDocumentEnvironment {mytabular} { o +m } {...} {...}
\NewDocumentEnvironment {mytabular*} { m o +m } {...} {...}
```

用 `s` 标记的 `*` 则应该放在 `\begin{⟨env⟩}` 的后面

```latex
\begin{document}
\NewDocumentEnvironment { envstar } { s }
{\IfBooleanTF {#1} {star} {no star}} {}

\begin{envstar}*
\end{envstar}
\end{document}
```

与 `\renewcommand, \providecommand` 等命令类似，xparse 宏包也允许在命令或环境已有定义时做出相应的处理，具体见表

![](image-20220715234245738.png|650)



### 编写宏包

#### 编写宏包

如果定义了很多新的环境和命令，文档的导言区将变得很长，在这种情况下，可以建立一个新的 LATEX 宏包来存放所有你自己定义的命令和环境，然后在文档中使用 \usepackage 命令来调用自定义的宏包。



写一个宏包的基本工作就是将原本在你的文档导言区里很长的内容拷贝到另一个文件中去，这个文件需要以 .sty 作扩展名。你还需要加入一个宏包专用的命令

```tex
\ProvidesPackage{⟨package name⟩}
```

这个命令应该放在你的宏包的最前面，并且

> `⟨package name⟩` 需要和宏包的文件名一致。 

命令 `\ProvidesPackage` 让 LATEX 记录宏包的名称，从而在 `\usepackage` 命令再次调用同一个宏包的时候忽略。例如

```tex
% Demo Package by Tobias Oetiker
\ProvidesPackage{demopack}

\newcommand{\tnss}{The not so Short Introduction
to \LaTeXe}
\newcommand{\txsit}[1]{The \emph{#1} Short
Introduction to \LaTeXe}

\newenvironment{king}{\begin{quote}}{\end{quote}}
```



当需要编写自己的文档类，如论文模板等。首先，自己的文档类以 .cls 作扩展名，开头使用 \ProvidesClass 命令

```tex
\ProvidesClass{⟨class name⟩}
```

这里 `{⟨class name⟩}` 也需要和文档类的文件名一致。在文档类中调用其它文档类

```tex
\LoadClass[⟨options⟩]{⟨package name⟩}
```



#### 调用宏包

如果想进一步把各种宏包的功能汇总到一个文件里，LATEX 允许在自己编写的宏包中调用其它宏包

```tex
\RequirePackage[⟨options⟩]{⟨package name⟩
```



### 计数器

章节符号、列表、图表……它们都是依靠 LATEX 提供的计数器功能完成的。



#### 定义和修改计数器

定义一个计数器的方法为

```tex
\newcounter{⟨counter name⟩}[⟨parent counter name⟩]
```

但如果你以不同的选项多次引入宏包，则有可能会引起错误。`⟨counter name⟩` 为计数器的名称。计数器可以有上下级的关系，可选参数 `⟨parent counter name⟩` 定义为 `⟨counter name⟩` 的上级计数器。



以下命令修改计数器的数值

* `\setcounter` 将数值设为 `⟨number⟩`
* `\addtocounter` 将数值加上 `⟨number⟩`
* `\stepcounter` 将数值加一，并将所有下级计数器归零

```tex
\setcounter{⟨counter name⟩}{⟨number⟩}
\addtocounter{⟨counter name⟩}{⟨number⟩}
\stepcounter{⟨counter name⟩}
```



#### 计数器的输出格式

计数器 `⟨counter⟩` 的输出格式由 `\the⟨counter⟩` 表示。这个值默认以阿拉伯数字形式输出，如果想改成其它形式，需要重定义 `\the⟨counter⟩`，如将 equation 计数器的格式定义为大写字母

```tex
\renewcommand\theequation{\Alph{equation}}
```

命令 `\Alph` 控制计数器 `⟨counter⟩` 的值以大写字母形式显示。下表列出所有可用于修改计数器格式的命令。

> 这些命令只能用于计数器，不能直接用于数字，如 `\roman{1}` 这样的命令会出错。

![](image-20220715235058059.png|500)



计数器的输出格式还可以利用其它字符，甚至其它计数器的输出格式与之组合。标准文档类里对 `\subsection` 相关的计数器的输出格式的定义相当于

```tex
\renewcommand\thesubsection{\thesection.\arabic{subsection}}
```



#### Latex 中的计数器

* 所有章节命令 `\chapter, \section` 等分别对应计数器 chapter、section 等等，而且有上下级的关系。而计数器 part 是独立的
* 有序列表 enumerate 的各级计数器为 enumi, enumii, enumiii, enumiv，也有上下级的关系
* 图表浮动体的计数器就是 table 和 figure；公式的计数器为 equation。这些计数器在 article 文档类中是独立的，而在 report 和 book 中以 chapter 为上级计数器
* 页码、脚注的计数器分别是 page 和 footnote

我们可以利用前面介绍过的命令，修改计数器的样式以达到想要的效果，比如把页码修改成大写罗马数字，左右加横线，或是给脚注加上方括号

```tex
\renewcommand\thepage{--~\Roman{page}~--}
\renewcommand\thefootnote{[\arabic{footnote}]}
```

它的内部机制是修改 page 计数器的格式 `\thepage`，并将计数器的值重置为 1 。



## GNUPlot

GNUPlot 能以极高效率产生各种图形, 并且能够对接各种可能的应用。它可以和 latex 和 tikz 配合使用。通过命令行安装

```shell
sudo apt install gnuplot
```

我们可以直接在命令行中输入 gnuplot 开始绘图

```shell
gnuplot
```

也可以先编辑绘图文件，以 .gp 结尾，新建文件 test.gp ，执行

```shell
gnuplot test.gp
```

运行生成绘图结果，通常是对应某种应用的绘图格式。



### 组合输出

可以将 GNUPlot 的输出格式设定为 latex

```shell
set terminal latex
```

生成的绘图结果就是 latex 的绘图格式；更有效率的绘图格式是 tikz

```shell
set terminal tikz
```

相较于原生的 latex 终端图形绘制效果更好，但是这种方式输出的格式需要使用 gnuplot-lua-tikz.sty 文件，配置难以处理。



### 绘图命令

使用 set output 制定输出文件名，plot 命令进行绘图

```shell
set terminal latex
set output "eg1.tex"
plot [-3.14:3.14] sin(x)
```

执行命令得到 eg1.tex 文件，打开后得到

```tex
\begin{pictrue}
...
\end{pictrue}
```

将此段内容插入到 tex 文档中即可得到图形

![](image-20220715101451969.png|500)

plot 命令格式为

```shell
plot [x-range] [y-range] expression1 with style1, expression2 with style2, ...
```

常用 style 如下

| style       | 作用           |
| ----------- | -------------- |
| lines       | 默认格式，实线 |
| points      | 叉点           |
| linespoints | 直线叉点       |
| dot         | 小点           |
| impulses    | 绘制垂线       |
| errorbars   | 绘制错误       |



### 调整格式

我们通过 `size` 命令调整图像大小，`format` 使得 xy 坐标的标记为 Latex 数学格式，其中 `%g` 是 C 风格的 `printf` 类型的格式化记号

```shell
set terminal latex size 7cm, 5cm
set output "eg2.tex"
set format xy "$%g$"
set title "This is a plot of $y=\sin(x)$"
set xlabel "This is the $x$ axis"
set ylabel "This is the $y$ axis"
plot [0:6.28] [0:1] sin(x)
```

![](image-20220715094735386.png|500)



使用 `set key` 命令可以调整图解的位置；如果使用两次 `plot` 绘制曲线，后者会清空图像重新绘制，因此要绘制两条曲线，就需要使用 `,` 分割同时绘制

```shell
set terminal latex
set output "eg3.tex"
set format xy "$%g$"
set title "This is another plot"
set xlabel "$x$ axis"
set ylabel "$y$ axis"
set key at 1, -0.5
plot [-pi/2:pi/2] [-1:1] x with lines, sin(x) with linespoints
```

![](image-20220715101844796.png|500)

有时我们需要用 $\pi$ 作为坐标单位，使用 `set xtics` 设置 $x$ 坐标的单位间隔

```shell
set terminal latex
set output "eg4.tex"
set format y "$%g$"
set format x "$%.2f$"
set title "This is $\\sin(x)$"
set xlabel "This is the $x$ axis"
set ylabel "$\\sin(x)$"
unset key
set xtics -pi, pi/4
plot [-pi:pi] [-1:1] sin(x)
```

![](image-20220715103243526.png|500)

注意 `set xtics` 表示从 $-\pi$ 开始间隔 $\pi/4$ 递增，这里没有指定上限；`set format` 设置 $x$ 轴坐标保留 2 位小数，它使用 C 风格的标记；使用 `unset key` 取消图解；最后，在双引号中的部分使用 `\` 时要通过 `\` 进行转义。



使用更精确的 `set xtics` 可以优化坐标表示

```shell
set terminal latex
set output "eg4.tex"
set format y "$%g$"
set format x "$%.2f$"
set title 'This is $\sin(x)$'
set xlabel "This is the $x$ axis"
set ylabel "$\\sin(x)$"
unset key
set xtics ("$-\\pi$" -pi, "$-\\frac{\\pi}{2}$" -pi/2, "0" 0, "$\\frac{\\pi}{2}$" pi/2, "$\\pi$" pi)
plot [-pi:pi] [-1:1] sin(x)
```

![](image-20220715104243218.png|500)

注意我们使用 `''` 表示字符串时，不需要进行转义。



### 复杂案例

最后我们给出一个较为复杂的例子

```shell
set terminal latex size 5.0, 3.0
set output "eg6.tex"
set format y "$%g$"
set format x "$%5.1f\mu$"
set title "This is a title"
set xlabel "This is the $x$ axis"
set ylabel 'This is\\a longer\\version\\ of\\the $y$\\ axis' offset -1
set label "Data" at -5,-5 right
set arrow from -5,-5 to -3.3,-6.7
set key top left
set xtic -10,5,10
plot [-10:10] [-10:10] "eg3.dat" title "Data File" with linespoints lt 1 pt 7,\
3*exp(-x*x)+1 title ’$3e^{-x^{2}}+1$’ with lines lt 4
```

![](image-20220715104921739.png|500)





## [伪代码](https://zhuanlan.zhihu.com/p/166418214)

我们使用 Algorithm2e 宏包来书写伪代码，导入格式为

```tex
\usepackage[ruled,linesnumbered,lined,boxed,commentsnumbered]{algorithm2e} 
```

使用宏包时中括号的参数含义

- `ruled` 是让标题显示在上面，否则算法的标题则在下面
- `linesnumbered` 让算法中显示行号
- `boxed` 让算法排版时插入在一个盒子里



### 基本语法

| 代码                                        | 含义               |
| ----------------------------------------- | ---------------- |
| `\;`                                      | 在行末添加分号，并自动换行    |
| `\caption{}`                              | 插入标题             |
| `\KwData{info}`                           | 效果："Data:info"   |
| `\Kwln{info}`                             | 效果："In:info"     |
| `\KwOut{info}`                            | 效果："Out:info"    |
| `\KwResult{info}`                         | 效果："Result:info" |
| `\For {condition}{cycle code}`            | 循环语句             |
| `\If {condition}{code}`                   | 条件语句             |
| `\While {condition}{cycle code}`          | 循环语句             |
| `\tcc{note}`                              | `/*注释*/`         |
| `\tcp{note}`                              | `//注释`           |
| `\elf {condition}{true code}{false code}` | 条件语句             |



如果要更改伪代码编号格式 `Algorithm num` 可以使用如下方式修改：

```tex
\renewcommand{\algorithmcfname}{算法名}
```

除了 `\If, \Else, \ElseIf 之外`，还有 `\uIf, \lIf, \uElse, \lElse, \uElseIf, \lElseIf` 等命令，他们的区别在于

* `\If, \Else, \ElseIf` 都是会以 end 结尾
* `\uIf, \uElse, \uElseIf` 是不以 end 结尾的块级元素
* `\lIf, \lElse, \lElseIf` 是不以 end 为结尾的行内元素
* 在 If-else 结构中，`\eIf` 自带 else（即 if 和 else 共用一个 end），而只是用 `\If, \Else` 的话则会多出一个 end 给 Else



官方案例

```latex
\begin{document}
    \begin{algorithm}[H]
      \SetAlgoLined
      \KwData{this text}
      \KwResult{how to write algorithm with \LaTeX2e }

      initialization\;
      \While{not at end of this document}{
        read current\;
        \eIf{understand}{
          go to next section\;
          current section becomes this one\;
          }{
          go back to the beginning of current section\;
          }
        }
      \caption{How to write algorithms}
    \end{algorithm}
\end{document}
```



牛顿法迭代的伪代码

```tex
\begin{algorithm}
	\caption{牛顿法}\label{algorithm}
	给定初始猜测点 $ x_0 $，迭代次数 $ k=0 $ 和精度要求 $ \epsilon>0 $\;
    \While{$\|\nabla r(x_k)r(x_k)\|>\epsilon$}
    {
        $ p_k\leftarrow -(\nabla r(x_k)\nabla r(x_k)^T + S(x_k))^{-1}\nabla r(x_k)r(x_k)$\;
        $ \alpha_k\leftarrow \arg\min_{\alpha>0} f(x_k+\alpha p_k) $\;
        $ x_{k+1}=x_k+\alpha_kp_k $\;
        $k\leftarrow k+1$\;
    }
\end{algorithm}
```



### 自定义宏指令

Algorithm2e 本身不支持 Do-While 结构（支持的是While-Do），需要自行定义。不过自行定义并不难，因为宏包中内置了 Repeat-Until 结构，在Algorithm2e中是“宏指令（Repeat macros）”的一种

```tex
\SetKwRepeat{Do}{do}{while}
```

定义完之后，就可以在伪代码块中使用如下命令调用

```tex
\Do{<结束条件>}{<执行命令>}
```

使用案例

```latex
\SetKwRepeat{Do}{do}{while}%

\begin{document}

\begin{algorithm}[H]
  \KwData{this text}
  \KwResult{how to write algorithm with \LaTeX2e }
  initialization\;
  \While{not at end of this document}{
    read current\;
    \Repeat{this end condition}{
      do these things\;
    }
    \eIf{understand}{
      go to next section\;
      current section becomes this one\;
    }{
      go back to the beginning of current section\;
    }
    \Do{this end condition}{
      do these things\;
    }
  }
  \caption{How to write algorithms}
\end{algorithm}

\end{document}
```



## [Beamer](https://latex-beamer.com/)

Beamer 可用于生成幻灯片格式的 pdf 文件，简单的示例如下

```tex
% Creating a simple Title Page in Beamer
\documentclass{beamer}

% Theme choice:
\usetheme{AnnArbor}

% Title page details: 
\title{Your First \LaTeX{} Presentation}
\author{latex-beamer.com}
\date{\today}

\begin{document}
	
	% Title page frame
	\begin{frame}
		\titlepage
	\end{frame}
	
\end{document}
```

将产生一张幻灯片开头

![](test_00.png|459)



### 帧 frame

每一页幻灯片就是一帧，有 3 种帧：

* 标题页帧 `\frame{\titlepage}`
* 普通帧

```tex
\begin{frame}[对齐方式]\frametitle{标题}
内容
\end{frame}
```

* 空白帧

```tex
\begin{frame}[plain]
\end{frame}
```



标题页帧是第一帧，可以展示主要信息：

```tex
\title[short title]{long title}										% 标题
\subtitle[short subtitle]{long subtitle}							% 子标题
\author[short name]{long name}										% 作者
\date[short date]{long date}										% 日期
\institute[short name]{long name}									% 研究所
\titlegraphic{\includegraphics[width=0.17\textwidth]{icon.png}}		% logo
```

这一部分放在 document 之前

```tex
% Title page details: 
\title{Computer Animation}
\author{Xing Yifan}
\date{\today}
\institute{Zhejiang University}
\titlegraphic{\includegraphics[width=0.17\textwidth]{logo.png}}
```



### 章节目录

使用 section、subsection 来产生带有变号的章节，用 `section*` 可以产生不在目录中出现的章节。

```tex
\begin{document}
	
	% 标题页
	\begin{frame}
		\titlepage
	\end{frame}
	
	% 目录页
	\begin{frame}\frametitle{Outline}
		\tableofcontents[pausesections]
	\end{frame}
	
	% 第一章节
	\section{First Section}
	\begin{frame}[c]\frametitle{first}
		This is the first page.
	\end{frame}
	
	% 第二章节
	\section{Second Section}
	\begin{frame}[c]\frametitle{first}
		This is the second page.
	\end{frame}

\end{document}
```



### 文本字体

常用的改变文本样式的命令如下

| 命令                    | 样式                      |
| ----------------------- | ------------------------- |
| \textit{text}           | $\textit{text}$           |
| \textrm{text}           | $\textrm{text}$           |
| \textsf{text}           | $\textsf{text}$           |
| \textcolor{green}{text} | $\textcolor{green}{text}$ |

由于默认格式下，数学公式的字体不是 Latex 字体，因此我们需要修改字体主题

```tex
\usefonttheme[onlymath]{serif}
```

其中 only-math 表示只修改数学字体，serif 表示衬线字体。



在 documentclass 中修改字体大小

```tex
\documentclass[11pt]{beamer}
```

可选项为 10-11-12pt 。



### 页面分栏

使用多栏环境 columns 或子页环境 minipage 来在一旁插入图片或表格。

```tex
% 分成宽度为一半的两列
\begin{columns}
\column{0.5\textwidth}
	First column text and/or code
\column{0.5\textwidth}
	Second column text and/or code
\end{columns}
```



### 区块环境

区块环境可以与普通文本很好地区分开来，适用于各种定理、引理以及示例。Beamer 自带了一些区块环境：

* block 普通环境
* theorem 定理环境
* lemma 引理环境
* proof 证明环境
* corollary 推论环境
* example 示例环境
* alertblock 警示环境

```tex
\begin{frame}[c]\frametitle{first}
	\begin{theorem}[Cauchy's theorem for a disc]
		If $ f $ is holomorphic in a disc, then
		\begin{equation}\label{key}
			\int_{\gamma}f(z)dz = 0
		\end{equation}
		for any closed curve $ \gamma $ in that disc.
	\end{theorem}
\end{frame}
```



### 动态效果

在 `\begin{frame}` 后面加上如下命令可以设置切换到这一页时的动态效果

```tex
% \transblindshorizontal<1> % 水平百叶窗效果
% \transblindsvertical<2> % 竖直百叶窗效果
% \transboxin<3> % 从中心到四角
% \transboxout<4> % 从四角到中心
% \transdissolve<5> % 溶解效果
% \transglitter<6> % 闪烁
% \transsplitverticalin<7> % 竖直撕开(向内)
% \transsplitverticalout<8> % 竖直撕开(向外)
% \transsplithorizontalin % 水平撕开(向内)
% \transsplithorizontalout % 水平撕开(向外)
% \transwipe<9> % 涂抹
```



### 补充设置

Beamer 中插入图片和表格的方式和 Latex 中完全一致。但是需要注意使用 caption 添加标题时默认没有编号，需要手动设置

```tex
\setbeamertemplate{caption}[numbered]
```

默认会产生右下角的导航栏，但是并不常用，可以去除

```tex
\setbeamertemplate{navigation symbols}{}
```

使用如下命令添加参考文件，最好放在 `\appendix` 后面

```tex
\appendix
	\begin{frame}[allowframebreaks]{References}
	\def\newblock{}
	\bibliographystyle{plain}
	\bibliography{mybib}
\end{frame}
```

有时我们需要在每一节的开头添加目录，来让观众得知当前的进度，在前面添加

```tex
\setbeamerfont{myTOC}{series=\bfseries}
\AtBeginSection[]{\frame{\frametitle{Outline}%
	\usebeamerfont{myTOC}\tableofcontents[current]}}
```



### 基本模板

官方收集了大量自定义模板可供免费下载。这里给出自定义的模板格式

```tex
% Creating a simple Title Page in Beamer
\documentclass{beamer}

% Use Package
\usepackage{graphicx}

% Theme choice:
\usetheme{Madrid}
\usefonttheme[onlymath]{serif}
\setbeamertemplate{caption}[numbered]
\setbeamertemplate{navigation symbols}{}
\setbeamerfont{myTOC}{series=\bfseries}
\AtBeginSection[]{\frame{\frametitle{Outline}%
	\usebeamerfont{myTOC}\tableofcontents[current]}}

% Title page details: 
\title{Computer Animation}
\author{Xing Yifan}
\date{\today}
\institute{Zhejiang University}
\titlegraphic{\includegraphics[width=0.17\textwidth]{logo.png}}

% 设置章节开头显示目录
\AtBeginSection[]
{
	\begin{frame}{主要内容}
		\transfade % 淡入淡出效果
		\tableofcontents[sectionstyle=show/shaded,subsectionstyle=show/shaded/hide] % 突出显示当前章节，而其它章节都进行了淡化处理
		\addtocounter{framenumber}{-1}  % 目录页不计算页码
	\end{frame}
}

\AtBeginSubsection[]
{
	\begin{frame}{主要内容}
		\transfade % 淡入淡出效果
		\tableofcontents[sectionstyle=show/shaded,subsectionstyle=show/shaded/hide] % 突出显示当前章节，而其它章节都进行了淡化处理
		\addtocounter{framenumber}{-1}  %目录页不计算页码
	\end{frame}
}

\begin{document}
	
	% 标题页
	\begin{frame}
		\titlepage
	\end{frame}
	
	% 目录页
	\begin{frame}\frametitle{Outline}
		\tableofcontents[pausesections]
	\end{frame}
	
	\section{First Section}
	\begin{frame}[c]\frametitle{First}
		This is the first page.
	\end{frame}

	\section{Second Section}
	\begin{frame}[c]\frametitle{Second}
		This is the second page.
	\end{frame}

	\section{Third Section}
	\begin{frame}[c]\frametitle{Third}
	\end{frame}

	\appendix
	\begin{frame}[allowframebreaks]{References}
		\def\newblock{}
		\bibliographystyle{plain}
		\bibliography{mybib}
	\end{frame}
	
\end{document}
```

