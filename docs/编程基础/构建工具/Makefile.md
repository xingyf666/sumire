# Makefile

## 基本介绍

Makefile 里主要包含了**五个东西：显式规则、隐晦规则、变量定义、文件指示和注释**

1. 显式规则：说明了如何生成一个或多的的目标文件；
2. 隐式规则：make 有自动推导的功能，可以让我们简略地书写 Makefile 
3. 变量定义：变量一般都是字符串，使用时**直接替换**，类似于宏；
4. 文件指示：在一个 Makefile 中引用 Makefile；根据某些情况指定 Makefile 中的有效部分；定义多行的命令；
5. 注释：Makefile 中只有行注释，其注释是用 `#` 字符。如果要使用 `#`，输入 `\#` 进行转义；

在 Makefile 中的命令，必须要以 Tab 键开始。


### 编译流程

通常一个 `.cpp` 文件需要经过几个编译步骤

- `g++ -E main.cpp -o main.i` 生成预处理文件，此过程将 `#include` 内容和宏插入到程序文本中
- `g++ -S main.i -o main.s` 生成汇编文件
- `g++ -c main.s -o main.o` 生成可重定位目标程序
- `g++ main.o -o main` 生成可执行程序



### 基础语法

基本示例如下

```makefile
targets : prerequisites
	command
	...

# 也可以采用等价写法
targets : prerequisites ; command
	command
	...
```

* targets 是文件名，以空格分开，可以使用通配符。一般来说是一个文件，但也有可能是多个文件；
* command 是**命令行**，每行 command 必须用 Tab 开头；
* prerequisites 也就是目标所依赖的文件（或依赖目标）；
* 可以使用反斜框 `\` 作为换行符；

    ```makefile
    main: main.cpp
    	g++ -o \
    	main main.cpp
    ```

make 会以 UNIX 的标准 Shell ，也就是 `/bin/sh` 来执行命令。



### 工作流程

make 命令执行时，需要一个 Makefile 文件，以告诉 make 命令需要怎么样的去编译和链接程序。按照以下流程：

* 在当前目录下找到 Makefile 或 makefile 的文件
    * 按顺序寻找 **GNUmakefile 、 makefile 、 Makefile** 的文件。最好使用 Makefile 这个文件名，因为更明显；
* 将文件中第一个目标文件作为最终的目标文件

    ```makefile
    # 这是第一个目标，makefile 只实现这个目标
    main: main.o
    	g++ -o main main.o
    
    # 为了编译 main.o，makefile 会执行这条目标产生 main.o
    main.o: main.cpp
    	g++ -c main.cpp
    ```

* 如果不存在 main 或者它**依赖的某个 `.cpp` 文件被修改**，就会重新生成那个 `.o` 文件，然后生成 main 文件



核心：目标和依赖，目标文件一定要存在，依赖文件可以不要

* 依赖：一般指你要编译的文件对象，也就是 `*.cpp`
* 目标：一般指你要生成的执行文件

有时依赖文件很多，手动输入比较麻烦。可以查看一个 `.cpp` 文件的所有依赖文件

```shell
g++ -MM main.cpp
```

即可显示编译该文件所需要的头文件和依赖文件。



我们展示一个稍微复杂的例子

```makefile
main: main.o header1.o header2.o header3.o
	g++ -o main main.o

main.o: main.cpp header1.h header2.h header3.h
	g++ -c main.cpp

header1.o: header1.cpp header1.h
	g++ -c header1.cpp

header2.o: header2.cpp header2.h
	g++ -c header2.cpp

header3.o: header3.cpp header3.h
	g++ -c header3.cpp

clean:
	rm *.o
	rm *.exe
```

我们之所以分离编译 `.o` 文件，是为了减少每次重新编译的开销。



GNU 的 make 非常强大，它可以自动推导文件以及文件依赖关系后面的命令，于是我们就没必要去在每一个 `.o` 文件后都写上类似的命令， make 会自动识别，并自己推导命令。

```makefile
main: main.o header1.o header2.o header3.o
	g++ -o main main.o

main.o: main.cpp header1.h header2.h header3.h
header1.o: header1.cpp header1.h
header2.o: header2.cpp header2.h
header3.o: header3.cpp header3.h

clean:
	rm *.o
	rm *.exe
```



### 默认目标

正如我们前面提到的，makefile 将第一个目标作为主要目标。但是也可以指定默认目标，例如

```makefile
# 指定 main.o 为最终目标，这样就只会生成 main.o
.DEFAULT_GOAL = main.o

main: main.o header1.o header2.o header3.o
	g++ -o main main.o

main.o: main.cpp header1.h header2.h header3.h
	g++ -c main.cpp

header1.o: header1.cpp header1.h
	g++ -c header1.cpp

header2.o: header2.cpp header2.h
	g++ -c header2.cpp

header3.o: header3.cpp header3.h
	g++ -c header3.cpp

clean:
	rm *.o
	rm *.exe
```



### 伪目标

伪目标是不产生文件的目标，并且它一定会更新所有依赖。当我们想要单独执行某个目标，例如

```makefile
clean:
	rm *.o
```

由于它没有任何依赖，我们只有通过显示地指明这个目标才能让其生效

```makefile
make clean
```



伪目标的取名不能和文件名重名。为了避免和文件重名的这种情况，可以使用 .PHONY 来显式地指明一个目标是伪目标

```makefile
.PHONY : clean
clean:
	rm *.o
```



可以为伪目标指定所依赖的文件。例如我们想要通过一个 make 直接生成多个文件

```makefile
all: main header1.o header2.o header3.o

# 这个标签只是显式地声明，即使没有这一行，all 依然是伪目标
.PHONY: all

main: main.cpp
	g++ -o main main.cpp

header1.o: header1.cpp header1.h
	g++ -c header1.cpp

header2.o: header2.cpp header2.h
	g++ -c header2.cpp

header3.o: header3.cpp header3.h
	g++ -c header3.cpp

clean:
	rm *.o
	rm *.exe
```

我们知道， Makefile 中的第一个目标会被作为其默认目标。我们声明了一个 all 的伪目标，其依赖于其它三个目标。**伪目标的特性使得每次都会更新依赖的目标**。



伪目标同样也可成为依赖。例如

```makefile
.PHONY: cleanall cleanobj cleanexe

# 将清除命令划分为两个单独的子命令。它们都是伪目标。
cleanall: cleanobj cleanexe

cleanobj:
	rm *.o

cleanexe:
	rm *.exe
```



### 引用文件

在 Makefile 使用 include 关键字可以把别的 Makefile 包含进来。例如

```makefile
# makefile

main: main.o header1.o header2.o header3.o
	g++ -o main main.o

main.o: main.cpp header1.h header2.h header3.h
	g++ -c main.cpp

header1.o: header1.cpp header1.h
	g++ -c header1.cpp

header2.o: header2.cpp header2.h
	g++ -c header2.cpp

# 引入其它 make 文件
include header3.make clean.mk

# 这是 header3.make 中的部分
header3.o: header3.cpp header3.h
	g++ -c header3.cpp
	
# 这是 clean.mk 中的部分
clean:
	rm *.o
	rm *.exe
```

理论上引用的文件可以是任何后缀，但是最常用的是 `.mk` 和 `.make`，因为能够区分出它们是 makefile 文件。



make 命令开始时，会寻找 include 所指出的其它 Makefile。按照顺序：

* 当前目录下，以及系统目录下；
* make 执行时 `-I` 或 `--include-dir` 参数给出的目录；



### 常用范例

* 头文件目录 ├─include
* 库文件目录 ├─lib
* 可执行文件 ├─bin
* 源文件目录 └─src

```makefile
# 需要引用的头文件路径和编译参数
OUTPUT_DIR = ./output/
BUILD_DIR = ./src/
HEADER_PATH = -I./include

# 编译资源
CXX_SRC = $(wildcard ./src/*.cpp)
CXX_OBJS = $(patsubst %cpp, %o, $(CXX_SRC))

# 中间资源
CXX__O = $(wildcard ./src/*.o)
OUTPUT__OBJS = $(wildcard ./output/*.o)

# 参数设置
MV = mv
CXX_BIN = main
CXX_CLEAN = clean
CXXFLAGS = '-std=c++14'

# 生成 main，然后将 .o 文件都移动到 output 目录下
$(CXX_BIN): $(CXX_OBJS)
	$(CXX) $^ -o $@ $(LIBS) && $(MV) $(CXX_OBJS) $(OUTPUT_DIR)

# 生成 .o 文件，然后将它们移动到 src 目录，然后才能进行下一步编译
$(CXX_OBJS): $(CXX_SRC)
	$(CXX) $^ -c $(LIBS) $(CXXFLAGS) $(HEADER_PATH) && $(MV) $(notdir $(CXX_OBJS)) $(BUILD_DIR)

.PHONY: $(CXX_CLEAN)
$(CXX_CLEAN):
	$(RM) $(CXX_BIN) $(OUTPUT__OBJS)
```



## 命令基础

### 显示命令

通常 make 会把其要执行的命令行在命令执行前输出到屏幕上，我们可以在命令前加上 `@` 来禁止输出命令行

```makefile
@echo 正在编译XXX模块......
```

当 make 执行时，会输出“正在编译XXX模块......”字串，但不会输出命令。我们还可以直接禁止命令显示

```makefile
.SILENT: target1 target2 ...
```

其后跟随的目标都被禁止显示命令（如果后面为空，则禁止所有命令显示）。



### 命令执行

理论上，make 的每行命令都单独运行在一个 shell 窗口中，因此不能保证按照顺序执行的命令能够互相衔接。如果希望上一条命令的结果应用于下一条命令，应当在一行中书写并用 `;` 分割命令。例如

```makefile
all:
	cd .. ; pwd
```

先返回上一级目录，然后输出上一级目录路径。如果分成两行写，pwd 会输出当前目录路径。



### 命令出错

当某个命令出现错误时，make 会终止执行之后的命令，但我们并不总是希望如此。例如 mkdir 命令要建立一个不存在的目录，当目录存在时会执行失败。但是我们只是希望存在这个目录，并不希望因此终止命令。这时可以在命令前增加 `-` 表示忽略错误

```makefile
clean:
	-rm -f *.o
	rm *.exe
```

如果没有 `-`，只要不存在 .o 文件，make 就会报错并停止执行；加上 `-` 后，即使没有 `.o` 文件，也能确保 `.exe` 文件被删除。



## 变量基础

make 中的变量本质上都是字符串，在声明时需要给予初值。在使用时，需要给在变量名前加上 `$` 符号

```makefile
a = 2
all:
	@echo $a
```

但是直接这样使用会出现一些问题，例如

```makefile
a = 3
abc = 2
all:
	echo $abc
```

运行将输出 `3bc`，这是因为 make 将 `a` 单独看作一个变量替换掉了。因此最好用小括号 `()` 或是大括号 `{}` 把变量给包括起来

```makefile
a = 3
abc = 2
all:
	echo $(abc)
```

如果要使用真实的 `$` 字符，需要用 `$$` 来表示

```makefile
all:
	echo $$
```



我们也可以取消变量的定义。例如

```makefile
objs = 10
undefine objs

all:
	echo $(objs)
```

其输出结果将是空的。



### 执行过程

GNU make 分两个阶段执行

* 读取阶段
    * 读取 makefile 文件内容
    * 根据内容建立变量
    * 构建显式规则、隐式规则
    * 建立目标与依赖之间的依赖图
* 目标更新阶段
    * 确定哪个目标需要更新，执行对应的更新方法

变量和函数的展开如果发生在第一阶段，称为**立即展开**，否则称为**延迟展开**。后者在用到的时候进行展开，有两种情况

* 延迟展开变量或函数在立即展开的表达式中用到
* 在第二阶段用到

**显式规则中，目标和依赖部分都是立即展开，在更新方法中延迟展开。**



### 变量赋值

#### 递归展开赋值

使用 `=` 进行赋值称为递归展开赋值，是延迟展开。如果赋值时**右端是其他变量或函数调用，将不做处理**，直到使用到该变量时才展开得到变量值。例如

```makefile
# 注意这一步不是延迟，而是立即赋值，但它仍属于递归展开赋值
objs = main.cpp header1.cpp

# 这一步右边出现了变量，是延迟展开
files = $(objs)

all:
	@echo $(files)

objs = main.o header1.o
```

其中 files 变量右端的表达式会被保留，由于 objs 被后面 .o 字符串覆盖，最后 files 延迟展开为 .o 字符串。



由于延迟展开的性质，赋值时右端变量可以使用后面定义的值。例如：

```makefile
a = $(b)
b = $(c)
c = 2

all:
	@echo $(a)
```

但这种形式也有不好的地方，那就是递归定义，例如：

```makefile
CFLAGS = $(CFLAGS) -O
A = $(B)
B = $(A)
```

这会让 make 陷入无限的变量展开过程中。当然，这种代码会报错，但是如果在变量中使用函数，类似的形式会导致 make 执行缓慢。



另一种自递归定义是

```makefile
files = 1
files = $(files)

all:
	@echo $(files)
```

由于延迟展开的特性，第二行 files 会无限递归展开，此时会报错。解决方案是使用下面将介绍的简单赋值

```makefile
files = 1
files := $(files)

all:
	@echo $(files)
```

在第二步直接展开获得 files 的值。



#### 简单赋值

使用 `:=` 进行赋值称为简单赋值，是立即展开。无论右端是变量还是函数，都会立即获得值。例如

```makefile
objs = main.cpp header1.cpp

files := $(objs)

all:
	@echo $(files)

objs = main.o header1.o
```

其中 files 立即展开为 objs 当前的值 .cpp 字符串。



这种方法只能使用前面已定义好了的变量。例如：

```makefile
y := $(x) bar
x := foo
```

则 y 的值是 `bar` ，而不是 `foo bar` 。



利用 `:=` 可以定义出一个值为**空格**的变量

```makefile
empty :=
space := $(empty) # end of the line

all:
	@echo a$(space)b
```

注意这里 `#` 是必须出现的，先用一个 Empty 变量来标明变量的值开始了，而后面采用 `#` 注释符来表示变量定义的终止。



注释符 `#` 的这种特性值得我们注意，例如：

```makefile
dir := /foo/bar # nothing
```

dir 这个变量的值是` /foo/bar` ，后面还跟了 4 个空格。如果不加注意的话，就会产生问题。



#### 条件赋值

使用 `?=` 表示条件赋值，如果变量没有被**定义**，就立即获得右端的值。例如

```makefile
foo = 100
foo ?= 200

all:
	@echo $(foo)
```

将会获得 100 而非 200；如下代码说明条件赋值是延迟赋值

```makefile
foo ?= $(a)
a = 100

all:
	@echo $(foo)
```



#### 追加赋值

使用 `+=` 操作符给变量追加值，如：

```makefile
objs = main.o
objs += foo.o
```




如果变量之前没有定义过，那么 `+=` 会自动变成 `=` ；如果前一次的是 `:=` ，那么会正常追加

```makefile
a := hello
a += world
```

上述代码获得 `hello world`；然而，如果前一次是 `=`，例如

```makefile
a = hello
a += world
```

由于延迟赋值，`+=` 会变成 `=` 产生递归问题，不过 make 执行时会解决这种问题。



#### Shell 运行赋值

使用 `!=` 运行 Shell 指令后将返回值赋给变量，例如

```makefile
version != g++ -v

all:
	@echo $(version)
```

获得 g++ 版本信息并输出。



### 模式规则

由 `%` 组成的目标或依赖就会应用模式规则。`%` 的展开发生在变量和函数的展开之后，变量和函数的展开发生在 make 载入 Makefile 时，而模式规则中的 `%` 则发生在运行时。



标中的 `%` 定义表示对文件名的匹配， `%` 表示长度任意的非空字符串。例如

* `%.c` 表示以 .c 结尾的文件名（文件名的长度至少为 3 ）
* `s.%.c` 则表示以 s. 开头，.c 结尾的文件名（文件名的长度至少为 5 ）

例如获取当前目录下的所有文件的字符串，然后将 h 开头 .h 结尾的字符串替换为 x 开头 .a 结尾的字符串

```makefile
files != ls
objs := $(files:h%.h=x%.a)

all:
	@echo $(objs)
```

注意这里前后替换的模式要相同。



利用模式规则可以**替换变量结尾共有**的部分，其格式是

```makefile
# 1
$(var:a=b) 

# 2
${var:a=b}
```

把变量 var 中所有以 a 字串结尾的 a 替换成 b 字串。例如

```makefile
foo := a.o b.o c.o
bar := $(foo:.o=.c)
```

将 foo 中的 .o 结尾替换为 .c 结尾。利用 `!=` 操作符，可以快速实现替换，例如

```makefile
files != ls *.cpp
objs := $(files:%.cpp=%.o)
```

这样我们就获得了当前目录下所有的 .cpp 文件的字符串转换为 .o 字符串的结果。



### 变量替换


由于变量本质上是字符串替换，可以把变量的值再当成变量

```makefile
z = 2
x = y
y = z
a := $($(x))
```

这里 `$x` 就是 `y`，因此括号展开后 `a` 的值就是 `$z`，也就是 2 。



在这种方式中，可以使用多个变量来组成一个变量的名字，然后再取其值：

```makefile
first_second = Hello
a = first
b = second
c = $($a_$b)
```

这里的 `$a_$b` 组成了 `first_second`，因此 `c` 的值就是 Hello 。



把变量的值再当成变量，同样可以用在操作符的左边：

```makefile
ab = Hello
c = a
d = b
$(c)$(d) += World
```

其中 `ab` 的值会是 Hello World 。



### 多行变量

使用 define 定义换行变量，最后以 endef 结束。例如

```makefile
# 定义两行变量
define objs
World
Hello
endef

all:
	$(info $(objs)) 
```

注意多行变量只能通过 info 函数输出，使用 echo 会报错。



默认情况下，多行变量定义与 `=` 是相同的。但是也可以改为使用其它赋值

```makefile
objs = Yes
define objs +=
World
Hello
endef

all:
	$(info $(objs)) 
```

类似地可以改为 `!=` 或 `?=` 等赋值操作。



### 环境变量

make 中可以直接使用系统变量。输入命令行

```shell
set
```

可以查看当前系统的所有环境变量，其中任何一个变量都可以直接使用。



makefile 中也有预定义的环境变量

```makefile
CURDIR := /home/zht 		# 记录当前路径
SHELL = /bin/sh				# shell 程序
MAKEFILE_LIST := Makefile	# 所有查找到的 makfile 文件
.DEFAULT_GOAL := all		# 默认目标
CC = cc 					# C语言编译器的名称
CPP = $(CC) -E  			# C语言预处理器的名称 $(CC) -E
CXX = g++       			# C++语言的编译器名称
RM = rm -f					# 删除文件程序的名称
CFLAGS						# C语言编译器的编译选项，无默认值
CPPFLAGS  					# C语言预处理器的编译选项，无默认值
CXXFLAGS					# C++语言编译器的编译选项，无默认值
```



### 变量覆盖

可以在命令行中为 makefile 中定义的变量赋值，从而覆盖文件中原本的值。例如

```makefile
MSG = Start Linking ...

all:
	@echo $(MSG)
```

在命令行输入

```shell
make MSG=123456
```

即在执行 make 时用 123456 覆盖 MSG 变量原本的值；如果中间有空格，需要用 `""` 包围

```shell
make MSG="Hello World"
```



如果不希望某个变量在外部被修改，可以增加前缀

```makefile
override MSG = Start Linking ...

all:
	@echo $(MSG)
```

这样外部无法再修改 MSG 的值。



### 局部变量

先前定义变量的方式都得到的是全局变量，但有时我们希望某个变量只在一定范围内有效。这就需要局部变量，例如

```makefile
# 全局变量
global_var = 2

# 分别对 test.1, test.2 目标定义局部变量
test.1: local_var1 = 3
test.2: local_var2 = 4

test.1:
	@echo global_var=$(global_var)
	@echo local_var1=$(local_var1)
	@echo local_var2=$(local_var2)

test.2:
	@echo global_var=$(global_var)
	@echo local_var1=$(local_var1)
	@echo local_var2=$(local_var2)
```

在 test.1 中无法访问 local_var2，在 test.2 中无法访问 local_var1，而 global_var 都可以访问。



通过模式规则可以为满足模式的目标定义局部变量

```makefile
global_var = 2

test.1: local_var1 = 3
test.2: local_var2 = 4

# 为具有此模式的目标定义它们可以访问的局部变量
test.%: local_var3 = 5

test.1:
	@echo global_var=$(global_var)
	@echo local_var1=$(local_var1)
	@echo local_var2=$(local_var2)
	@echo local_var3=$(local_var3)

test.2:
	@echo global_var=$(global_var)
	@echo local_var1=$(local_var1)
	@echo local_var2=$(local_var2)
	@echo local_var3=$(local_var3)
```



### 自动变量

自动变量的值会依据规则中的目标和依赖自动计算其值。例如

| 变量 | 含义                               |
| ---- | ---------------------------------- |
| `@`  | 目标文件的名称，包含扩展名         |
| `^`  | 所有的依赖文件，以空格隔开，不重复 |
| `<`  | 第一个依赖文件的名称               |
| `+`  | 所有的依赖文件，空格隔开，可以重复 |
| `*`  | 目标文件的名称，不包含扩展名       |
| `?`  | 被更新的依赖文件                   |

在上述自动变量之后附加 D 表示对应变量所在的目录，结尾不带 /；附加 F 表示对应变量除去目录后的文件名。例如

```makefile
# 输出 main 生成的目录
main: main.cpp header1.cpp
	@echo $(@D)

# 输出 main.cpp header1.cpp 的文件名
main: main.cpp header1.cpp
	@echo $(^F)
```

注意目录是相对于 makefile 所在目录的相对目录。



### 二次展开

我们在 make 执行过程中提到目标和依赖都是**立即展开**，但有时我们希望能够延迟展开。例如

```makefile
objs = main.cpp header1.cpp

main: $(objs)
	@echo $^

objs = main.o header1.o
```

我们可能更希望 main 使用 objs 最后的值，这时可以增加前缀 `.SECONDEXPANSION:` 表明需要二次展开

```makefile
objs = main.cpp header1.cpp

.SECONDEXPANSION:
main: $$(objs)
	@echo $^

objs = main.o header1.o
```

注意使用 `$$` 标记，说明要展开两次。如果不加前缀，那么 `$$(objs)` 会被解析为 `(objs)` 字符串，从而报错。



### 条件判断

使用条件判断，可以让 make 根据运行时的不同情况选择不同的执行分支。



#### ifdef / ifndef

判断变量是否定义/未定义，如果定义/未定义就执行之后的命令。例如

```makefile
a = 2

# 如果定义了 a，就输出 def 2
ifdef a
$(info def $(a))
endif

# 如果未定义 a，就输出 def 2
ifndef a
$(info def $(a))
endif
```



还可以产生 else 分支，例如

```makefile
a = 2
b = 3

ifdef a
$(info def $(a))
else ifndef b
$(info def $(b))
endif
```



#### ifeq / ifneq

判断两个值是否相等/不等。例如

```makefile
a = 2

ifeq ($(a), 2)
$(info def $(a))
endif
```



## 函数基础

函数以 `$` 来标识，其语法如下：

```makefile
$(<function> <arguments>)
${<function> <arguments>}
```

这里 function 是函数名，arguments 是参数，参数间以逗号分隔，而函数名和参数之间以空格分隔。



### 信息提示函数

#### error

提示错误信息，包括行号等，并终止运行。例如

```makefile
$(error Something went wrong)
```



#### warning

提示警告信息，包括行号等，但仍继续运行。例如

```makefile
$(warning Something not good)
```



#### info

用于输出提示

```makefile
a = hello world
all:
	$(info $(a))
```

上述代码输出 `a` 的值。info 函数相较于 shell 的 echo 函数最大的优势是不必考虑后者的语法问题，例如

```makefile
# 1
objs = 2
a = $$(objs)
all:
	echo $(a)

# 2
objs = 2
a = $$(objs)
all:
	$(info $(a))
```

这里我们用两个 `$` 来表示 `$`，因此给 `a` 赋值 `$(objs),` 而不是 2 。但是第一段代码会报错，因为 `$` 在 shell 语法中有意义；而第二段代码可以正常输出 `$(objs)` 。



### 通用函数

#### if

条件判断，如果 condition 不是空，返回 then-part，否则返回 else-part；语法为

```makefile
$(if <condition>,<then-part> )
$(if <condition>,<then-part>,<else-part> )
```

例如当 src/ 下有 .cpp 文件时，下面代码返回这些文件名

```makefile
files = $(wildcard src/*.cpp)
result = $(if $(files),$(files),File not exists)

all:
	@echo $(result)
```

如果不存在 .cpp 文件，就返回 File not exists 。



#### or

返回条件中第一个不为空的部分。例如当 src/ 下有 .cpp 文件，include/ 下有 .h 文件时

```makefile
file1 = $(wildcard src/*.h)
file2 = $(wildcard src/*.cpp)
file3 = $(wildcard include/*.h)
result = $(or $(file1),$(file2),$(file3))

all:
	@echo $(result)
```

上述代码获得 .cpp 文件字符串。



#### and

如果所有条件不为空，返回最后一个条件，否则返回空。例如在前面的条件下

```makefile
file1 = $(wildcard src/*.h)
file2 = $(wildcard src/*.cpp)
file3 = $(wildcard include/*.h)
result = $(and $(file1),$(file2),$(file3))

all:
	@echo $(result)
```

上述代码返回空。



#### file

用于读写文件。语法为

```makefile
$(file op filename[, text])
```

其中 op 是操作符，包括

* `>` 覆盖
* `>>` 追加
* `<` 读取

text 是写入的内容，读取时不需要。例如覆盖加上追加

```makefile
$(file > m.txt,Something)
$(file >> m.txt, more)
```

注意追加会换行然后写入，并且参数前面的空格也会写入；读取操作可能因为版本问题，没有成功。



#### foreach

foreach 函数和别的函数非常的不一样。因为这个函数是用来做循环用的， Makefile 中的 foreach 函数几乎是仿照于 Unix 标准 Shell 中的 for 语句而构建的。它的语法是：

```makefile
$(foreach <var>,<list>,<progress> )
```

其中 `<var>` 是指定的每项变量，`<list>` 是要处理的字符串列表，`<progress>` 是对每一项的处理

```makefile
objs = src/header1.cpp include/header2.cpp header3.cpp 
result := $(foreach each,$(objs),$(patsubst %.cpp,%.o,$(notdir $(each))))

all:
	$(info $(result))
```

上述代码将 objs 中每一项 each 的目录去除，然后将 .cpp 后缀替换为 .o 后缀。



#### call

此函数用于自定义函数，语法为

```makefile
$(call <expression>,<parm1>,<parm2>,<parm3>...)
```

在函数中可以用 `$(n)$` 来访问第 n 个参数。例如

```makefile
func = Hello World
result = $(call func)
```

将会返回 Hello World 。



使用参数的函数

```makefile
func = var1 = $(1), var2 = $(2), var3 = $(3)
result = $(call func,Hello,My,)

all:
	$(info $(result))
```

由于我们传入了两个参数，因此得到

```shell
var1 = Hello, var2 = My, var3 =
```



#### value

对于不是立即展开的变量，获得其原始定义，否则返回变量值。例如

```makefile
var = hello
var2 = $(var)
result = $(value var2)

all:
	$(info $(result))
```

其中 var2 不是立即展开，result 是其原始定义 $(var) 而不是 hello 。



#### origin

该函数用于查看变量的来源。语法为

```makefile
$(origin <variable> )
```

注意 variable 是变量名而非引用，所以最好不要在 variable 中使用 `$` 字符。 其返回值说明变量的信息

* 未定义：返回 undefined
* 默认变量：返回 default
* 环境变量：返回 environment
* Makefile 中的变量：返回 file
* 命令行变量：返回 command line
* override 变量：返回 override
* 自动变量：返回 automatic

例如自动变量

```makefile
result = $(origin @)

all:
	$(info $(result))
```



#### flavor

用于查看一个变量的赋值方式

* 未定义：返回 undefined
* 递归展开赋值：返回 recursive
* 简单赋值：返回 simple

例如

```makefile
f1 = Hello
f2 = $(f1)

result = $(flavor f1) $(flavor f2) $(flavor f3)

all:
	$(info $(result))
```

三个变量分别得到 recursive recursive undefined 。



#### eval

将文本作为 make 目标插入。例如

```makefile
# 定义一个多行变量
define eval_target =
eval:
	@echo Eval Target
endef

# 将变量的值作为一个目标运行
$(eval $(eval_target))
```



#### shell

该函数执行新生成一个 Shell 程序来执行命令。例如

```makefile
result = $(shell ls)

all:
	$(info $(result))
```

注意如果 Makefile 中有一些比较复杂的规则，并大量使用了这个函数，可能导致低效率。



### 字符串处理函数

#### subst

把字串 text 中的 from 字符串替换成 to，语法为

```makefile
$(subst <from>,<to>,<text>)
```

例如将所有 .o 字符替换为 .h 字符

```makefile
objs = header1.oheader2.o header3.o
result = $(subst .o,.h,$(objs))
	
# 得到 header1.hheader2.h header3.h
```

注意这里的**替换是任意符合 from 的字符串都会替换，而模式规则只能替换头尾**。



我们还可以去掉一些字符，例如

```makefile
objs = header1.o header2.o header3.o
result = $(subst .o,,$(objs))
	
# 得到 header1 header2 header3
```

所以**使用函数时要注意参数位置不要加空格**。



#### patsubst

查找 text 中的单词是否符合模式 pattern，以 replacement 替换，语法为

```makefile
$(patsubst <pattern>,<replacement>,<text>)
```

例如将所有 .o 结尾替换为 .h 结尾

```makefile
objs = header1.oheader2.o header3.o
result = $(patsubst %.o,%.h,$(objs))
	
# 得到 header1.oheader2.h header3.h
```

注意中间的 .o 就没有被替换。



#### strip

去掉 string 字串中**开头和结尾**的空字符，语法为

```makefile
$(strip <string>)
```

例如

```makefile
objs = space header1.o header2.o header3.o space

# 在头尾将 space 替换为大量空格
result = $(subst space,   ,$(objs))

# 注意这里使用 := 立即展开变量，防止递归定义
result := $(strip $(result))
```



#### findstring

在字串 in 中查找 find 字串，如果找到，那么返回 find ，否则返回空字符串。语法为

```makefile
$(findstring <find>,<in>)
```

例如

```makefile
objs = space header1.o header2.o header3.o space

result := $(findstring ea,$(objs))
```

由于 ea 存在，因此会得到 ea 字符串。



#### filter

以 pattern 模式过滤 text 字符串中的单词，保留符合模式 pattern 的单词。语法为

```makefile
$(filter <pattern...>,<text>)
```

例如返回 .c 和 .s 结尾的字串

```makefile
objs = header1.o header2.s header3.k

result := $(filter %.o %.s,$(objs))
```

多种模式之间用空格分开。**注意只能匹配头尾**。



#### filter-out

以 pattern 模式过滤 text 字符串中的单词，去除符合模式 pattern 的单词。语法为

```makefile
$(filter-out <pattern...>,<text>)
```

例如返回不含 .c 和 .s 结尾的字串

```makefile
objs = header1.o header2.s header3.k

result := $(filter-out %.o %.s,$(objs))
```

多种模式之间用空格分开。**注意只能匹配头尾**。



#### sort

字符串 list 中的单词排序（升序）。语法为

```makefile
$(sort <list>)
```

例如

```makefile
$(sort foo bar bar lose)
```

返回 bar foo lose，会去掉 list 中相同的单词。



#### word

取字符串 text 中第 n 个单词并返回。如果 n 比 text 中的单词数要大，那么返回空字符串。语法为

```makefile
$(word <n>,<text>)
```

例如

```makefile
objs = header1.o header2.s header3.k

result := $(word 1,$(objs))
```



#### wordlist

从字符串 text 中取从 s 开始到 e 的单词串并返回。如果 s 比 text 中的单词数要大，那么返回空字符串；如果 e 大于 text 的单词数，那么返回从 s 开始，到 text 结束的单词串。语法为

```makefile
$(wordlist <s>,<e>,<text>)
```

例如

```makefile
objs = header1.o header2.s header3.k

result := $(wordlist 2,4,$(objs))
```



#### words

统计 text 中字符串中的单词个数并返回。语法为

```makefile
$(words <text>)
```



#### firstword

取字符串 text 中的第一个单词。语法为

```makefile
$(firstword <text>)
```

这个函数可以用 word 函数来实现：

```makefile
$(word 1,<text>)
```



#### lastword

取字符串 text 中的最后一个单词。语法为

```makefile
$(lastword <text>)
```



### 文件名操作函数

#### dir

从文件名序列 names 中取出目录部分，即最后一个反斜杠之前的部分，然后返回。如果没有反斜杠，那么返回 `./` ；语法为

```makefile
$(dir <names...>)
```

例如

```makefile
objs = src/header1.o include/header2.h header3.cpp

result := $(dir $(objs))
	
# 获得 arc/ include/ ./
```



#### notdir

从文件名序列 names 中取出非目录部分，即最后一个反斜杠之后的部分。语法为

```makefile
$(notdir <names...>)
```

例如

```makefile
objs = src/header1.o include/header2.h header3.cpp

result := $(notdir $(objs))
```



#### suffix

从文件名序列 names 中取出各个文件名的后缀并返回，如果文件没有后缀，则返回空字串。语法为

```makefile
$(suffix <names...>)
```

例如

```makefile
objs = src/header1.o include/header2.h header3.cpp

result := $(suffix $(objs))
```

获得三种后缀。



#### basename

从文件名序列 names 中取出各个文件名的前缀部分并返回；如果文件没有前缀，则返回空字串。语法为

```makefile
$(basename <names...>)
```

例如

```makefile
objs = src/header1.o include/header2.h header3.cpp

result := $(basename $(objs))
```

得到的结果去除了后缀。



#### addsuffix

把后缀 suffix 加到 names 中的每个单词后面。语法为

```makefile
$(addsuffix <suffix>,<names...>)
```

例如

```makefile
objs = src/header1.o include/header2.h header3.cpp

result := $(addsuffix .exe,$(objs))
```

为所有字符串增加了 .exe 后缀。



#### addprefix

把前缀 prefix 加到 names 中的每个单词签前缀。语法为

```makefile
$(addprefix <prefix>,<names...>)
```

例如

```makefile
objs = src/header1.o include/header2.h header3.cpp

result := $(addprefix make/,$(objs))
```

每个字符串增加了 make/ 前缀。



#### join

把 list2 中的单词对应地加到 list1 的单词后面。如果 list1 的单词个数要比 list2 的多，那么 list1 中的多出来的单词将保持原样；如果 list2 的单词个数要比 list1 多，那么 list2 多出来的单词将被复制到 list1 中。语法为

```makefile
$(join <list1>,<list2>)
```

例如

```makefile
objs = src/header1.o include/header2.h header3.cpp header4.cpp
suf = .a .b .c

result := $(join $(objs),$(suf))
```

只有前三个 objs 中的字符串被添加了 .a .b .c 后缀；

```makefile
objs = src/header1.o include/header2.h header3.cpp 
suf = .a .b .c .d

result := $(join $(objs),$(suf))
```

后缀数量更多，于是多出只有 .d 后缀的字符串。



#### wildcard

在某一路径下寻找匹配的文件。语法为

```makefile
$(wildcard arg)
```

例如获得当前目录和 src/ 目录下的所有 .cpp 文件

```makefile
result := $(wildcard *.cpp) $(wildcard src/*.cpp)
```



#### realpath

获得文件的绝对路径，如果文件不存在则得到空。语法为

```makefile
$(realpath file)
```

例如

```makefile
result := $(realpath include/header2.h)
```



#### abspath

获得文件的绝对路径，**如果文件不存在则得到当前目录**。语法为

```makefile
$(abspath file)
```

例如

```makefile
result := $(abspath include/header2.h)
```





## 进阶操作

### 命令参数

指定读取 makefile 的目录

```shell
-C path
--directory=path
```

如果有多个 `-C` 参数， 则后面的路径以前面的作为相对路径，并以最后的目录作为被指定目录。如：

```shell
make –C ~hchen/test –C prog
```

此命令等价于

```shell
make –C ~hchen/test/prog
```



输出 make 的调试信息

```shell
—debug[=<options>]
```

它有几种不同的级别可供选择，如果没有参数，那就是输出最简单的调试信息。

| options | 作用                                                         |
| ------- | ------------------------------------------------------------ |
| a       | all ，输出所有的调试信息。（会非常的多）                     |
| b       | basic ，只输出简单的调试信息                                 |
| v       | verbose ，比 b 选项输出更多的调试信息                        |
| i       | implicit ，输出所有的隐含规则                                |
| j       | jobs ，输出执行规则中命令的详细信息                          |
| m       | makefile ，输出 make 读取 makefile ，更新 makefile ，执行 makefile 的信息 |

命令 `--debug=a` 可以简写为

```shell
-d
```



输出 makefile 中的所有数据，包括所有的规则和变量

```shell
-p
--print-data-base
```

如果想输出信息而不想执行 makefile ，可以使用

```shell
make -q -p
```

查看执行 makefile 前的预设变量和规则，可以使用

```
make –p –f /dev/null
```

这个参数输出的信息会包含 makefile 文件的文件名和行号。



假定目标 file 需要更新，如果和 `-n` 选项使用，下面参数会输出该目标更新时的运行动作

```shell
-W <file>
--what-if=<file>
--new-file=<file>
--assume-file=<file>
```

如果没有 `-n` 就使得 file 的修改时间为当前时间。



| 参数                                     | 作用                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| `-b` 或 `-m`                             | 忽略和其它版本 make 的兼容性                                 |
| `-B` 或 `--always-make`                  | 认为所有的目标都需要更新（重编译）                           |
| `-e` 或 `--environment-overrides`        | 指明环境变量的值覆盖 makefile 中定义的变量的值               |
| `-f=<file>` 或 `--file=<file>`           | 指定需要执行的 makefile                                      |
| `-h` 或 `-help`                          | 显示帮助信息                                                 |
| `-i` 或 `--ignore-errors`                | 在执行时忽略所有的错误                                       |
| `-I <dir>` 或 `--include-dir=<dir>`      | 指定一个被包含 makefile 的搜索目标。可以使用多个 `-I` 参数来指定多个目录 |
| `-j [<jobsnum>]` 或 `--jobs[=<jobsnum>]` | 同时运行命令的个数，默认运行所有命令。最后一个 `-j` 才是有效的 |
| `-k` 或 `--keep-going`                   | 出错也不停止运行                                             |
| `-l <load>` 或 `--load-average[=<load]`  | 指定 make 运行命令的负载                                     |
| `-n` 或 `--just-print`                   | 仅输出执行过程中的命令序列，但并不执行                       |
| `-o <file>` 或 `--old-file=<file>`       | 不重新生成的指定的 file ，即使这个目标的依赖文件新于它       |
| `-q` 或 `--question`                     | 不运行命令，也不输出，仅仅是检查所指定的目标是否需要更新     |
| `-r` 或 `--no-builtin-rules`             | 禁止 make 使用任何隐含规则                                   |
| `-R` 或 `--no-builtin-variabes`          | 禁止 make 使用任何作用于变量上的隐含规则                     |
| `-s` 或 `--silent`                       | 在命令运行时不输出命令的输出                                 |
| `-S` 或 `--no-keep-going`                | 取消 `-k` 选项的作用                                         |
| `-t` 或 `--touch`                        | 把目标的修改日期变成最新的，也就是阻止生成目标的命令运行     |
| `-v` 或 `--version`                      | 输出 make 程序的版本、版权等关于 make 的信息                 |
| `-w` 或 `--print-directory`              | 输出运行 makefile 之前和之后的信息                           |
| `--no-print-directory`                   | 禁止 `-w` 选项                                               |
| `--warn-undefined-variables`             | 只要 make 发现有未定义的变量，那么就输出警告信息             |



### 多目标

如果多个独立目标具有相同依赖和相同写法的命令，例如

```makefile
all: t1 t2 t3 t4

t1: prepare
	@echo $@ is done.

t2: prepare
	@echo $@ is done.

t3: prepare
	@echo $@ is done.

t4: prepare
	@echo $@ is done.

prepare:
	@echo $@ is done.
```

其中 `t1 t2 t3 t4` 都依赖 perpare，并且它们的命令写法相同，都是输出目标的名称。这时候可以将它们合并为一条

```makefile
all: t1 t2 t3 t4

# 与前面的代码等价
t1 t2 t3 t4: prepare
	@echo $@ is done.

prepare:
	@echo $@ is done.
```



使用独立多目标还可以简化复杂的编译过程。例如我们在最开始引入的代码，先用自动变量和模式规则进行替换得到

```makefile
main: main.o header1.o header2.o header3.o
	g++ -o $@ $^

# 将 main.o 替换为 main.cpp
main.o: main.cpp header1.h header2.h header3.h
	g++ -c $(@:%.o=%.cpp)

header1.o: header1.cpp header1.h
	g++ -c $(@:%.o=%.cpp)

header2.o: header2.cpp header2.h
	g++ -c $(@:%.o=%.cpp)

header3.o: header3.cpp header3.h
	g++ -c $(@:%.o=%.cpp)

.PHONY: clean
clean:
	rm *.o *.exe
```

发现后面 4 个规则有相同的命令，因此可以简写为

```makefile
objs = main.o header1.o header2.o header3.o

main: $(objs)
	g++ -o $@ $^

$(objs):
	g++ -c $(@:%.o=%.cpp)

.PHONY: clean
clean:
	rm *.o *.exe
```

我们用 objs 存放要产生的多个目标，依赖通过 make 自动推导，然后使用一条命令即可。



### 静态模式

在多目标中，我们将具有不同依赖的规则进行合并，例如

```makefile
objs = main.o header1.o header2.o header3.o

main: $(objs)
	g++ -o $@ $^

$(objs):
	g++ -c $(@:%.o=%.cpp)

.PHONY: clean
clean:
	rm *.o *.exe
```

但是这样会导致依赖问题：由于显式的依赖为空，当某个 `.cpp` 或 `.h` 文件被更新，不会更新对应的 `.o` 文件。



静态模式可以定义多目标的规则，其语法为

```makefile
<targets...>: <target-pattern>: <prereq-patterns ...>
	<commands>
```

* target-parrtern 是目标的模式规则；
* prereq-parrterns 是依赖的模式规则，可以有多个用空格分开；

由于每个 `.o` 文件一般由唯一 `.cpp` 文件编译得到，因此我们将之前的代码改写为

```makefile
objs = main.o header1.o header2.o header3.o

main: $(objs)
	g++ -o $@ $^

# 目标模式为 .o 文件，依赖模式为 .cpp 文件
$(objs): %.o : %.cpp
	g++ -c $<

.PHONY: clean
clean:
	rm *.o *.exe
```

其中静态模式可以直接作为依赖 `$<` 。这样将每个 `.o` 目标和一个 `.cpp` 文件对应，当 `.cpp` 文件被修改，就会更新对应的 `.o` 文件。



进一步，我们增加对 `.h` 文件的依赖

```makefile
objs = header1.o header2.o header3.o

main: $(objs) main.o 
	g++ -o $@ $^

# 增加 %.h 模式
$(objs): %.o : %.cpp %.h
	g++ -c $<

main.o: %.o : %.cpp
	g++ -c $<

.PHONY: clean
clean:
	rm *.o *.exe
```

注意到我们将 `main.o` 单独列出，因为它没有对应的 `.h` 文件依赖。此时，修改 `.h` 文件后，make 也会更新对应的 `.o` 文件。



### 文件路径

当有许多依赖文件都存在不同目录下时，就需要指定文件路径。



可以使用变量 VPATH 指定所有可以找到依赖文件和目标文件的路径，路径之间用 `:` 分隔。例如

* 头文件目录 ├─include
* 源文件目录 └─src

在这种情况下，写法为

```makefile
objs = header1.o header2.o header3.o

# 指定 .cpp 和 .h 文件的路径
VPATH = src:include

main: $(objs) main.o 
	g++ -o $@ $^

# 还要具体指定命令中头文件的路径
$(objs): %.o : %.cpp %.h
	g++ -c $< -Iinclude

main.o: %.o : %.cpp
	g++ -c $< -Iinclude

.PHONY: clean
clean:
	rm *.o *.exe
```

这里需要注意，当指定了 VPATH 后，`$<` 的内容是 `src/*.cpp`，也就是带有路径的 `.cpp` 文件。如果还使用

```makefile
$(objs): %.o : %.cpp %.h
	g++ -c $(@:%.o=%.cpp) -Iinclude
```

这里 `$@` 的内容是 objs，即不带路径的 `.o` 文件，那么替换后编译文件就是 `.cpp`，它们并不存在，会报错。



更为灵活的方法是小写的 vpath 关键字，语法为

```makefile
vpath <pattern> <directories>
```

它指定某个模式文件的寻找路径。例如

```makefile
objs = header1.o header2.o header3.o

vpath %.h include
vpath %.cpp src

main: $(objs) main.o 
	g++ -o $@ $^

$(objs): %.o : %.cpp %.h
	g++ -c $< -Iinclude

main.o: %.o : %.cpp
	g++ -c $< -Iinclude

.PHONY: clean
clean:
	rm *.o *.exe
```



### 隐式规则

回到使用静态模式简化 makefile 的情况，我们的代码是

```makefile
objs = header1.o header2.o header3.o

main: $(objs) main.o 
	g++ -o $@ $^

# 增加 %.h 模式
$(objs): %.o : %.cpp %.h
	g++ -c $<

main.o: %.o : %.cpp
	g++ -c $<

.PHONY: clean
clean:
	rm *.o *.exe
```

然而 make 有自动推导依赖关系以及编译命令的能力，上述代码可以简化为

```makefile
objs = main.o header1.o header2.o header3.o
CC = g++

main: $(objs)

.PHONY: clean
clean:
	rm *.o *.exe
```

这里有两个关键点：

* 需要指定目标为 main，让 make 知道目标与依赖的关系
* 由于默认编译使用 cc，需要修改环境变量 CC 为 g++ 进行编译



#### 编译规则

在隐式规则下，C 语言编译 `.c` 到 `.o` 为

```makefile
$(CC) $(CPPFLAGS) $(CFLAGS) -c
```

C++ 编译 `.cc .cpp .C` 到 `.o` 为

```makefile
$(CXX) $(CPPFLAGS) $(CXXFLAGS) -c
```

其中 CPPFLAGS 是预处理参数，CFLAGS 和 CXXFLAGS 分别指定编译选项。在前面环境变量中已经介绍，它们默认为空。



最后由 `.o` 链接到可执行文件为

```makefile
$(CC) $(LDFLAGS) *.o $(LOADLIBES) $(LDLIBS)
```



如果我们想要阻止某个隐式规则，例如阻止 `.cpp` 到 `.o` 的隐式编译规则，使用

```makefile
objs = main.o header1.o header2.o header3.o
CC = g++

main: $(objs)

# 什么都不写，对应的编译规则就为空
%.o : %.cpp

.PHONY: clean
clean:
	rm *.o *.exe
```

如果要修改这条规则，只需要在后面附加命令

```makefile
objs = main.o header1.o header2.o header3.o
CC = g++

main: $(objs)

# 修改编译规则为 g++
%.o : %.cpp
	$(CXX) -c $<

.PHONY: clean
clean:
	rm *.o *.exe
```



#### 隐式变量

所有的隐式规则用到的变量如下：

* 命令变量

    * AS —— 汇编语言编译程序。默认命令是 as 
    * CC —— C 语言编译程序。默认命令是 cc 
    * CXX —— C++ 语言编译程序。默认命令是 g++ 
    * CPP —— C 程序的预处理器（输出是标准输出设备）。默认命令是 `$(CC) –E` 
    * MAKEINFO —— 转换 Texinfo 源文件（ `.texi` ）到 Info 文件程序。默认命令是 makeinfo
    * TEX —— 从 TeX 源文件创建 TeX DVI 文件的程序。默认命令是 tex 
    * TEXI2DVI —— 从 Texinfo 源文件创建 TeX DVI 文件的程序。默认命令是 texi2dvi 
    * RM —— 删除文件命令。默认命令是 rm –f  

* 命令参数变量

    * SFLAGS —— 汇编语言编译器参数。（当明显地调用 `.s` 或 `.S` 文件时）
    * CFLAGS —— C 语言编译器参数
    * CXXFLAGS —— C++ 语言编译器参数
    * CPPFLAGS —— C 预处理器参数。（ C 和 Fortran 编译器也会用到）
    * LDFLAGS —— 链接选项，如 -L
    * LDLIBS —— 链接用到的库文件

    如果没有指明其默认值，那么其默认值都是空。命令参数变量通常使用追加的方式设置，例如

    ```makefile
    objs = main.o header1.o header2.o header3.o
    
    vpath %.h include
    vpath %.cpp src
    
    # 重设编译器
    CC = g++
    
    # 追加头文件目录
    CXXFLAGS += -Iinclude
    
    # 设置程序使用的字符集，可以解决乱码
    CXXFLAGS += -fexec-charset=GBK -finput-charset=UTF-8
    
    main: $(objs)
    
    .PHONY: clean
    clean:
    	rm *.o *.exe
    ```



### 自动生成依赖

有时依赖关系会包含一系列头文件。例如我们之前的 makefile 文件

```makefile
objs = main.o header1.o header2.o header3.o
CC = g++

main: $(objs)

%.o : %.cpp
	$(CXX) -c $<

.PHONY: clean
clean:
	rm *.o *.exe
```



使用编译器的这一功能，我们可以将每个源文件自动生成的依赖关系放到一个文件中，从而增加头文件依赖。例如

```makefile
objs = main.o header1.o header2.o header3.o
CC = g++

main: $(objs)

# 在隐式规则中，加入了输出目标和依赖关系的命令
%.o : %.cpp
	@echo $@ --- $^
	$(CXX) -c $<

# 引入 .d 文件，前面加 - 避免报错导致停止执行；如果不存在，就会通过下面规则生成
-include $(objs:%.o=%.d)

# 生成 .d 文件的规则
%.d : %.cpp
	@-rm $@
	$(CXX) -MM $< >$@.tmp
	@sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.tmp > $@
	@-rm $@.tmp

.PHONY: clean
clean:
	-rm *.o *.exe
```

上述代码生成 `.d` 文件的规则为：

* 移除之前的 `.d` 文件 `@-rm $@`

* 利用 `-MM` 获得源文件的依赖，存放到 `.d.tmp` 文件中 `$(CXX) -MM $< >$@.tmp`，此时内容为

    ```makefile
    main.o: main.cpp header1.h header2.h header3.h
    ```

* 使用正则表达式，读取 `.d.tmp` 文件中的文本，修改格式为

    ```makefile
    main.o main.d : main.cpp header1.h header2.h header3.h
    ```

    结果存放在 `.d` 文件中

* 移除临时文件 `.d.tmp` `@-rm $@.tmp`

执行 make 时会 include `.d` 文件

* 由于开始不存在 `.d` 文件，就会使用 `%.d : %.cpp` 规则生成 `.d` 文件；

* 之后引入 .d 文件的内容，也就是引入所有形如

    ```makefile
    main.o main.d : main.cpp header1.h header2.h header3.h
    ```

* 这时 `.o .d` 两个目标的依赖合并，于是就为 `.o` 文件增加了 `.h` 文件依赖关系；



### 嵌套执行

当代码结构较为复杂时，可以将不同模块的文件单独放在某些目录下，每个目录下都单独提供一个 makefile，在主目录下直接引用这些子目录下的 makefile，从而简化模块。



例如考虑文件结构

├─bin (存放可执行文件)
├─include (存放头文件)
├─lib (存放其它 `.cpp` 文件)
└─src (存放 `main.cpp`)

我们现在将 lib 下的 `.cpp` 文件编译为库文件，将 src 下的 `main.cpp` 编译为 `.o` 文件，然后链接库文件和 `.o` 文件生成可执行文件。



在 lib 目录下，我们编译 `.cpp` 为 `.o`，然后链接为静态库文件 `header.lib`，最后增加删除命令

```makefile
cpps := $(wildcard *.cpp)
objs := $(cpps:%.cpp=%.o)

# 指定头文件目录和编码
CXXFLAGS += -I../include -fexec-charset=GBK -finput-charset=UTF-8

# 编译 .o 文件为静态库 header.lib
header.lib: $(objs)
	$(AR) rs $@ $^

.PHONY: clean
clean:
	$(RM) *.o *.lib
```

在 src 目录下，我们编译 `main.cpp` 为 `.o`，然后链接静态库文件，在 bin 目录下生成 `.exe` 文件

```makefile
# 指定 .lib 文件的目录
vpath %.lib ../lib

# 指定头文件目录和编码
CXXFLAGS += -I../include -fexec-charset=GBK -finput-charset=UTF-8

# 指定库文件目录
LDFLAGS += -L../lib

# 指定库文件
LDLIBS += -lheader

# 在 bin 目录下生成 main.exe
../bin/main: main.o header.lib
	$(CXX) -o $@ $^ $(LDFLAGS) $(LDLIBS)

.PHONY: clean
clean:
	$(RM) *.o ../bin/*.exe
```

最后我们在主目录下提供一个 makefile 来整合两个子模块

```makefile
.PHONY: subsrc sublib clean

# 执行 src 下的 makefile
subsrc: sublib
	$(MAKE) -C src

# 执行 lib 下的 makefile
sublib:
	$(MAKE) -C lib

# 执行 src 和 lib 下的 clean 命令
clean:
	$(MAKE) clean -C src
	$(MAKE) clean -C lib
```

这里 -C 参数在前面**命令参数**部分有描述。



可以通过 export 从主 makefile 向子项目的 make 传递变量

```makefile
# 传递 var
export var = 2

# 等价的写法
var = 2
export var
```

如果不写变量，就表示传递所有变量

```makefile
export
```

使用 unexport 可以取消某个变量的传递，用法与 export 相同。



需要注意的是 SHELL 和 MAKEFLAGS 变量一定会传递到下层 Makefile 中，但是 make 命令中的有几个参数并不往下传递，它们是 `-C` 、 `-f` 、 `-h` 、 `-o` 和 `-W` ；如果你不想往下层传递参数，写法为

```makefile
subsystem:
	cd subdir && $(MAKE) MAKEFLAGS=
```

当我们使用 `-C` 来执行下层 makefile 时，会自动打开 `-w` 参数，输出目录的信息。例如

```makefile
make[1]: Entering directory 'F:/Desktop/learnmake/src'
```

但是如果参数中有 `-s` 就会导致 `-C` 参数无效。
