# Python

## 基本介绍

### 配置安装

在[官网](https://www.python.org/downloads/release/python-3113/)下载 Windows 64 位版本的 python 安装文件，点击后设置将 `python.exe` 添加到环境路径，然后通过 Customize 安装。

![[image-20230512195451118.png]]

设置高级选项，为所有用户安装，从而赋予写入权限

![[image-20230512195714566.png]]

安装完成后在系统环境变量中可以看到 python 的安装目录已经添加

![[image-20230512195952648.png]]

也可以手动在用户环境变量中添加。注意安装后需要重启才能更新环境变量。



#### 多个版本

在官网找到 [Windows](https://www.python.org/downloads/windows/) 版本的 Release 列表，注意查找具有 installer 的版本，例如安装 `3.6` 版本时，只有 `3.6.8` 版本具有 installer，下载这个版本，安装到另一个文件夹中即可

![[image-20230820224945548.png]]

同样也需要重启才能更新环境变量。



在 Ubuntu 中安装 `Python3.10.6` 的步骤如下

```sh
# 下载解压
sudo apt-get update
sudo apt-get install zlib1g-dev libbz2-dev libssl-dev libncurses5-dev libsqlite3-dev libreadline-dev tk-dev libgdbm-dev libdb-dev libpcap-dev xz-utils libexpat1-dev liblzma-dev libffi-dev libc6-dev
wget https://www.python.org/ftp/python/3.10.6/Python-3.10.6.tgz 
tar -xzf Python-3.10.6.tgz

# 移动到系统目录下
sudo mv Python-3.10.6 /usr/local/share/
cd /usr/local/share/Python-3.10.6
./configure --prefix=/usr/local/python3.10.6 --enable-optimizations

# 编译安装
make
sudo make install

# 链接程序
sudo ln -s /usr/local/python3.10.6/bin/python3.10 /usr/bin/python3.10
sudo ln -s /usr/local/python3.10.6/bin/python3.10-config /usr/bin/python3.10-config
```

然后可以运行

```sh
python3.10
```

进入 Python 命令行模式。



#### 查看版本

在 Windows 下执行命令

```powershell
py --list
```

获得当前系统中所有可用的 python 程序的路径。在 Ubuntu 中执行

```sh
ls /usr/bin/python*
```

实现相同的效果。



#### 修改版本

可以修改默认的 python 版本

```shell
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.10 1
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.10 1
```

可以输入 `python` 或 `python3` 验证已经为 `3.10.6` 版本。



在 Windows 上，只需要在 `C:\Windows` 下提供 `py.ini` 文件

```ini
[defaults]
python=3.12
```

即可切换默认版本。



#### 安装 pip

有时因为环境错误，导致 pip 无法启动。这时候需要重新安装 pip，首先复制 [get-pip.py](https://bootstrap.pypa.io/get-pip.py) 到 Script 目录下，执行

```shell
$ ..\python.exe get-pip.py
```

就可以针对该版本的 python 获得 pip 程序。在 Ubuntu 下通过 apt 安装

```shell
sudo apt install python3-pip
```



#### 移除所有库

如果想要删除所有库，首先可以获得环境中的库，然后根据库来移除

```shell
pip freeze > requirements.txt
pip uninstall -r requirements.txt -y
```

其中 `-y` 参数自动 `yes` 避免手动操作。



### 虚拟环境

通常进行项目开发时，为了确保外部环境中的包的版本变化不影响工程，会在项目目录中创建虚拟环境。



#### 创建环境

执行命令

```shell
python -m venv .venv
```

在当前目录下创建 `.venv` 配置文件夹用于放置虚拟环境。此时其中的 Scripts 文件夹中已经有 `pip.exe` 安装程序和 `python.exe` 解释器。



如果想要指定版本，例如 `3.6.8` 版本，使用

```shell
py -3.6 -m venv .venv
```



#### 激活环境

在 `.venv\Scripts` 目录下有一个 `activate.bat` 脚本，通过命令行运行

```shell
.venv\Scripts\activate.bat
```

即可激活虚拟环境。在 Powershell 中运行

```powershell
.venv\Scripts\Activate
```

在 Ubuntu 下进入虚拟环境需要执行

```shell
source .venv/bin/activate
```



此时可以通过 pip 或 pip3 安装想要的第三方库

```shell
pip3 install pygame
```



#### 退出环境

要退出环境，执行

```python
deactivate.bat
```

如果移动了文件夹位置，就要修改 `pyvenv.cfg activate activate.bat` 中虚拟环境的路径。如果使用 pyinstaller 出现问题，应该先卸载然后重新安装。


在 Ubuntu 下退出虚拟环境执行命令

```sh
deactivate
```



#### VSCode

在 vscode 编辑器右下角 `3.11.3 64-bit` 位置可以修改解释器为 `.venv` 文件夹中的解释器

![[image-20230512203916131.png]]

右键点击 `Run Python File in Terminal` 即可运行 `.py` 代码。



还可以使用 tasks 配置

```json
{
    "tasks": [
        {
            "type": "shell",
            "label": "run",
            "command": ".venv/Scripts/python.exe",
            "args": [
                "${file}"
            ]
        }
    ],
    "version": "2.0.0"
}
```

利用在 Open Keyboard Shortcuts 中绑定的快捷键

```json
// 将键绑定放在此文件中以覆盖默认值
[
    {
        "key": "ctrl+f5",
        "command": "workbench.action.tasks.runTask",
        "args": "run"
    }
]
```

就可以通过 `Ctrl + F5` 运行程序。



#### 导出环境

可以将环境中的库版本导出为文本文件

```shell
(.venv) $ pip freeze >requirements.txt
```

这样以后可以根据 `requirements.txt` 文件来安装所需的依赖

```shell
(.venv) $ pip install -r requirements.txt
```

如果当前目录下没有虚拟环境，可以打开 vscode，选择右下角的“选择解释器”，然后可以创建新的虚拟环境，导入 requirements 文件。



### UV

#### 安装工具

UV 是一个 Rust 编写的 Python 包管理工具，可以用 pip 安装

```shell
pip install uv
```

```embed
title: "uv"
image: "https://github.com/astral-sh/uv/assets/1309177/629e59c0-9c6e-4013-9ad4-adb2bcf5080d#only-light"
description: "uv is an extremely fast Python package and project manager, written in Rust."
url: "https://docs.astral.sh/uv/"
```



#### 配置环境

在指定 `pyproject.toml` 文件的情况下，执行

```shell
uv sync
```

自动安装其中的所有依赖。

>[!note]
>这个命令的另一个作用是在修改配置文件后，重新执行它来刷新环境。



也可以在目录下执行

```shell
uv init -p 3.12 proj
```

完成项目初始化，会自动生成项目所需的配置文件

![[image-20260311114358498.png]]

然后创建虚拟环境

```shell
uv venv
```



#### 添加依赖

只需要在项目目录下创建 `pyproject.toml` 文件

```toml
[project]
name = "proj"
version = "0.1.0"
```

然后执行

```shell
uv add flask>=3.1.3
```

安装 `flask` 库，并且会自动管理直接依赖和间接依赖。只有直接依赖会写入到 `toml` 文件中

```toml
[project]
name = "proj"
version = "0.1.0"
dependencies = [
    "flask>=3.1.3",
]
```

可以查询依赖树

```shell
uv tree
```

>[!note]
>注意 uv 可以在全局环境中运行，添加的库自动安装在虚拟环境中。



#### 移除依赖

执行

```shell
uv remove flask
```

移除依赖，并修改 `toml` 文件。



#### 标记依赖

可以将依赖标记为开发依赖，避免将其一并打包

```shell
uv add ruff --dev
```



#### 全局工具

也可以将其安装为全局工具

```shell
uv tool install ruff
uv tool uninstall ruff
```

之后可以执行如下三种指令来运行 tool

```shell
ruff
uvx ruff
uv tool run ruff
```

>[!note]
> `uvx` 是 `uv tool run` 的简写。



#### 运行代码

在全局安装 uv 后，可以直接执行

```shell
uv run main.py
```

或者执行

```shell
uv run .
```

就会自动在当前目录的虚拟环境下运行程序，不需要再激活环境。



可以将代码标记为脚本

```shell
uv init --script main.py
```

这样会在文件开头定义脚本描述，在其中指定依赖

```python
# /// script
# requires-python = ">=3.12"
# dependencies = ["requests"]
# ///

import requests

def main() -> None:
    print(requests.__version__)


if __name__ == "__main__":
    main()

```

然后可以直接运行

```shell
uv run main.py
```

不需要任何本地依赖。



#### 打包文件

在 `toml` 文件中提供 `project.scripts` 字段，表明打包文件名 `proj`，调用的函数为 `main.py` 中的 `test` 函数

```toml
[project]
name = "proj"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = []
[project.scripts]
proj = "main:test"
```

执行构建和安装命令

```shell
uv build
uv tool install .\dist\proj-0.1.0-py3-none-any.whl
```

其中安装文件可以在项目的 `dist` 目录下找到。然后可以运行打包程序

```shell
uvx proj
```

>[!note]
>注意项目名和打包文件名应当相同，这样才能正常执行程序，否则需要在调用时指定额外的参数。



#### 配合 pip

使用 uv 配合 pip 可以加速传统 pip 的运行

```shell
uv pip install -r requirements.txt
```



### 格式化

使用 ruff 进行格式化和修复。只需要安装 ruff 插件，然后在命令面板可以执行命令

![[image-20260311164151861.png]]

还可以在 `toml` 文件中给出具体的格式化配置

```toml
[tool.ruff]
preview = true
fix = true
unsafe-fixes = false
lint.select = [
    "E",   # 错误
    "F",   # pyflakes
    "W",   # 警告
    "N",   # 命名规范
    "B",   # 潜在错误
    "I",   # 导入排序
    "C4",  # 列表推导式优化
    "C90", # 复杂度检查
    "UP",  # 自动升级旧语法
    "SIM", # 代码简化
]

[tool.ruff.format]
quote-style = 'single'
```



### 类型检查

使用 pyright 进行静态类型检查。只需要安装 pylance 插件，其中内置了 pyright 。在用户设置中搜索 `type checking mode` 切换检查模式

![[image-20260311162942352.png]]



### Jupyter

安装 Jupyter 插件，通过命令面板输入 `Create: New Jupyter Notebook` 创建一个 `.ipynb` 文件，修改解释器为虚拟环境中的解释器

![[image-20230610110105230.png]]

然后可以通过 `Ctrl + Enter` 运行代码块，`Alt + Enter` 创建新的代码块。



在 Jupyter 中可以使用

```python
%timeit A @ B
```

自动执行代码多次，并输出平均执行时间和标准差。



### 打开方式

在 `D:\Python\Lib\idlelib` 目录下找到 `idle.bat` 设为 `.py` 文件的默认打开方式即可。



### 程序打包

需要安装 pyinstaller 库，然后执行

```shell
pyinstaller -w -F xxx.py -p xxx.py -p xxx.py -i xxx.ico
```

其中 -F 参数后是主程序文件，将会打包产生同名的 `.exe` 文件；-p 参数后是依赖的其它程序文件；-i 参数后是程序图标。注意

* 64 位版本打包的 exe 只能在 64 位操作系统使用
* 打包文件夹和文件的名称不能用中文
* 打包 pygame 时，字体 `pygame.font.SysFont("宋体", 32)` 换成 `pygame.font.SysFont('arial',32)`

生成文件目录为

├─build
│  └─main
│      └─localpycs
├─dist
└─\_\_pycache\_\_

目标可执行文件位于 dist 目录下，注意调整导入资源的路径。



PyInstaller 库的其余参数包括

| 参数                         | 含义                              |
| -------------------------- | ------------------------------- |
| -F                         | 指定打包后只生成一个 exe 文件               |
| -D (–onedir)               | 创建一个目录，包含 exe 文件，但会依赖很多文件（默认选项） |
| -c (–console, –nowindowed) | 使用控制台，无界面（默认选项）                 |
| -w (–windowed, –noconsole) | 使用窗口，无控制台                       |
| -p                         | 添加搜索路径，让其找到对应的库                 |
| -i                         | 设置生成程序的 icon 图标                 |



## 基本语法

### 操作符

* 算术运算符：除去 `+,-,*,/，%` 以外，需要强调使用的有

    * `**` 计算幂
    * `//` 取整除

* 逻辑运算符：`and,or,not`
* 成员运算符：`in,not in` 判断数据中是否包含指定的成员

    ```python
    gList = ['x', 'y', 'z']
    'x' in gList	# 将返回 true
    ```

还可以判断字符串是否包含指定的子字符串。



#### 特殊操作符

还有一些特殊的操作符

* is 判断两个类型是否相同
* pass 什么都不做，用于占位
* del 删除变量，释放内存



#### 三元操作符

还存在由分支语句得到的三元运算符

```python
a if a > b else c
```

即若 a > b 则取 a，否则取 c 。该运算符可以嵌套使用。



### 内置函数

| 函数                | 作用                                       | 函数            | 作用                                      |
| ------------------- | ------------------------------------------ | --------------- | ----------------------------------------- |
| abs()               | 返回绝对值                                 | all()           | 参数都是 True 时返回 True，否则返回 False |
| any()               | 参数都是 False 时返回 False，否则返回 True | bin()           | 转化整数为二进制字符串                    |
| chr()               | 返回整数对应的 Unicode 字符                | divmod(a,b)     | 返回元组 `(a//b,a%b)`                     |
| eval()              | 将**参数去掉引号**后，返回表达式的值       | exec()          | 返回程序运行后的结果                      |
| hash()              | 哈希运算，对任意固定数据类型产生一个哈希值 | hex()           | 转化整数为十六进制字符串                  |
| id()                | 查看变量地址                               | isinstance()    | 判断对象是否为给定类型                    |
| map()               | 返回函数对多个序列操作的迭代器             | ord()           | 返回 Unicode 字符对应的整数               |
| oct()               | 转化整数为八进制字符串                     | round(data[,n]) | 保留 n 位精度                             |
| zip([iterable,...]) | 返回序列对应元素组成的元组的列表           | type()          | 查看数据类型                              |



#### print

用于输出，写入文件。语法为

```python
print(value, ..., sep=' ', end='\n', file=sys.stdout, flush=False)
```

其中 sep 表示多个值之间的间隔，end 表示末尾输出，file 表示输出目标，flush 表示是否直接刷入内存，默认 False 会先写入缓冲。



可以使用 print 向文件中输出，例如写入 csv 表格文件

```python
csv = open('test.csv', 'a')

print('名称', '数量', '内容', sep=',', file=csv)
print('Li', '10', 'hi', sep=',', file=csv)
print('Lin', '5', 'hello', sep=',', file=csv)
print('Hu', '6', 'yes', sep=',', file=csv)

csv.close()
```

与 xlsx 是二进制文件不同，csv 文件是文本文件，相邻列之间用 `,` 分隔，只能保存一个 sheet，不会保存格式。



#### chr / ord

利用 Unicode 字符与整数的转换关系，我们可以编写一个加密程序

```python
s = input('请输入文字：')
r = input('请输入加密或解密：')

# 加密函数
def si(x):
    try:
        for i in s:
            print(ord(i),end=' ')
    except:
        print('输入有误。')

# 解密函数
def so(x):
    try:
        l = x.split(' ')
        for i in l:
            print(chr(int(i)),end='')
    except:
        print('输入有误。')

# 通过字典保存函数
d = {'加密':si,'解密':so}
d[r](s)
```



#### map

`map()` 将函数作用于可迭代对象中的每个元素。例如

```cpp
a,b,c = map(int, input().split(' '))
```

将输入的三个元素分别转换为 `int` 类型。



#### zip

`zip()` 会将输入的序列中的元素分别组合，直到某个序列元素被全部使用。例如

```python
list(zip('abcdefg', range(3), range(4)))
```

这里是将三个序列

```python
'abcdefg'
[0,1,2]
[0,1,2,3]
```

其中的元素按照顺序分别组合，得到 `(a,0,0),(b,1,1),(c,2,2)`，由于 `[0,1,2]` 这个序列被用完了，因此得到

```python
[(a,0,0),(b,1,1),(c,2,2)]
```



#### input

使用 input 命令获得用户输入的字符串

```python
input(str)
```

将会先输出 str，然后读取输入的内容，以换行符作为终止。可以追加 strip 函数去除输入中的字符串

```python
input("input: ").strip('x')
```

将会去掉输入中的所有 `x` 字符。



### 分支循环

可以搭配循环和 else 指令来实现循环是否全部完成的判断，例如

```python
for i in range(10):
    print(i)
    if i == 5:
        break
    # 当循环正常结束（没有 break, exit）时执行
else:
    print("loop done...")
    
i = 0
while i < 10:
    print(i)
    if i == 5:
        break
    i += 1
    # 当循环正常结束（没有 break, exit）时执行
else:
    print("loop done...")
```



### 异常处理

使用 `try` 语句捕获异常

```python
try:
    # 测试代码
except:
	# 出现错误后执行
else:
    # 没有错误时执行
finally:
    # 最后都会执行
```

可以根据不同的错误类型处理

```python
try:
    # 测试代码
except OSError:
	# 出现 OSError 后执行
except AssertionError:
	# 出现 AssertionError 后执行
except (TypeError, IndexError):
	# 出现 TypeError 和 IndexError 后执行
except Exception:
    # 捕获任意异常，但效率很低
```

可以获得错误信息

```python
try:
    # 测试代码
except OSError as reason:
	print('文件错误，原因是：' + str(reason))
except AssertionError as reason:
	print('断言错误，原因是：' + str(reason))
```



可以主动发起异常，例如使用

```python
try:
    # 主动发起 OSError 异常，原因为 yes
    raise OSError('yes')
except Exception as reason:
    # 输出 yes
    print(str(reason))
```

使用断言表达式，如果表达式错误，抛出 AssertionError

```python
try:
    assert(0)
except AssertionError as reason:
    print('出错了')
```



### with-as

with-as 是简化版的 try-except-finally 语句。结构为

```python
with expr [as var]:
    # block
```

其中表达式 expr 的值会赋给 var 变量。在这个过程中，能够处理 expr 和 block 中出现的异常，并且会自动释放资源。



常用的情景是打开编辑文件

```python
with open('file.txt', 'wb') as f:
	# block
```

能够解决打开失败，以及忘记关闭的问题。当 block 部分代码执行完成，文件会自动关闭。



这种结构等价于

```python
try:
    # 执行 expr 对象的 __enter__ 函数，返回值给 var
    # 执行 block 代码
finally:
    # 执行 expr 对象的 __exit__ 函数
```

要求对象拥有 `__enter__` 和 `__exit__` 两个成员函数，前者用于初始化并返回自身的引用，后者用于善后和处理异常。



## 数据类型

* 数字类型 int
* 字符串 str
* 列表 list
* 布尔 bool
* 集合 set
* 字典 dict
* 字节 bytes

可以使用 `type` 函数查看变量的类型。使用 `help` 函数查看变量类型的描述

```python
help(str)
```

因此 help 常用于查看某个数据类型的使用方法。



### 字符串

字符串是用 `""` 或 `''` 包含的字符序列，它具有几个重要特性

* 不可修改：字符串赋值后不能改动，重新赋值相当于另外申请了内存，原先的值没有变，而是被回收

* 可索引：可以用 `[]` 索引其中的单个字符，也可以获得切片

```python
x = 'world'
x[2:4]
```

* 使用 `'''` 创建多行字符串
```python
x = '''
hello,
my name is Li.
It is a good day.
'''
```

* 字符串拼接
```python
x = 'hello'
y = 'world'   
x + ' ' + y
```

* 格式化输出

```python
name = 'Alex'
age = 10
hobby = 'apple'

msg = '''
----------------- Student Info -----------------
name: %s
age: %d
hobby: %s
------------------------------------------------
''' % (name, age, hobby)

print(msg)
```



基本格式化规则为

| 占位符 | 作用       | 占位符 | 作用         |
| --- | -------- | --- | ---------- |
| %d  | 整型替换字符串  | %f  | 保留精度       |
| %s  | 字符串替换字符串 | %c  | 单个字符       |
| %u  | 无符号整数    | %o  | 八进制数       |
| %x  | 十六进制数    | %X  | 字母大写的十六进制数 |
| %e  | 科学计数法    | %E  | 字母大写的科学计数法 |


还可以使用 `f` 前缀直接使用变量

```python
msg = f'''
----------------- Student Info -----------------
name: {name}
age: {age}
hobby: {hobby}
------------------------------------------------
'''
```



使用 `format` 显式地进行格式化

```python
'{} {}'.format("hello", "world")		# 得到 'hello world'
'{1} {0}'.format("hello", "world")		# 得到 'world hello'
"{a} {b}".format(b="hello", a="world")	# 得到 'world hello'
```



### 列表

列表用于存放各种类型的数据。例如

```python
gList = ['x', 1, 2, 'y']
```

允许通过索引访问，并且可以通过负索引从后向前访问

```python
gList[0]	# 第一个元素
gList[-1]	# 最后一个元素
```



可以使用 `del` 删除指定索引的元素

```python
del gList[1]
```



可以指定起始、终止以及步长来进行切片

```python
gList[1:6:2]	# 从 1 元素到 5 元素为止，索引步长为 2
```

其中的参数可以省略，用 `:` 分隔即可

```python
gList[::2]	# 从 0 元素开始，步长为 2，直到结束
```

并且列表可以进行嵌套

```python
gList = ['x', [1, 2, 3]]
```

通过 list 函数将字符串转化为列表

```python
a = list('abc')
```



#### 深浅 Copy

在使用嵌套列表时，需要注意复制问题。例如

```python
a = ['A', [1, 2], 'B']
b = a
```

此时 a, b 两个列表相同，修改任意单个元素

```python
a[0] = 'C'
a[1] = 'D'
```

此时不会有任何问题；然而，当我们**修改嵌套列表**就会出现问题

```python
a[1][0] = 3
```

此时 `b[1][0]` 也会被修改。这是因为在列表中，每个元素都是单独储存，**列表只是分别指向这些元素**。因此 a[1], b[1] 实际上指向同一个列表，修改该列表就会同时更改 a, b 中的列表元素。这就是浅 copy 。



如果需要进行深 copy，引入 copy 库

```python
import copy
b = copy.deepcopy(a)
```



#### 列表生成式

可以快速生成列表，基本语法为

```python
[ expression for i in seq if condition ]
```

例如构造字符串列表

```python
a = [f"str_{i}" for i in range(10) if i > 4]
# ['str_5', 'str_6', 'str_7', 'str_8', 'str_9']
```



### 元组

元组是**只读列表**，一旦创建就不能更改。例如

```python
a = (1, 2, 3)
```

注意如果要创建只有一个元素的元组，应该按照如下格式

```python
a = (1,)
```



另外，元组中的列表中的元素是可以修改的。例如

```python
a = (1, [2, 3])
a[1][0] = 5
```



### 字典

字典类型用于快速查找数据。定义方式为

```python
info = {key1 : value, key2 : value, ...}
```

其中 key 必须为不可变数据类型（字符串、数字、**元组**），且不能重复。也就是**可以将列表转换为元组，从而将其作为 key 使用**。



字典可以嵌套使用，例如

```python
info = {'a' : 1, 'b' : {'c' : 2}}
```

在 Python 3.7 之后的版本字典是有序的。



我们甚至可以用字典来保存函数

```python
d = {'字典':dict,'字符串':str,'列表':list,'元组':tuple,
      '集合':set,'整型':int,'输出':print,'输入':input}
```



### 集合

集合存放无序不重复的数据。定义方式为

```python
a = {1, 2, 3, 'a'}
```

使用 `{}` 默认表示字典，如果要定义空集合，应该使用

```python
a = set()
```



可以对两个集合进行基本运算

```python
a & b	# a 交 b
a | b	# a 并 b
a - b	# a 差 b
a ^ b	# 对称差
```



## 文件处理

对文件的操作有两种：文本文件、二进制文件（视频、图片等）



### open

文件读写函数，格式为

```python
f = open(file, mode='rt', encoding=None)
```

其中可以指定打开模式

| 模式 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| r    | 只读模式（默认），当文件不存在时返回异常                     |
| w    | 覆盖写模式（新创建或重写内容），文件不存在时创建             |
| a    | 追加模式（新创建或追加内容），文件不存在时创建，存在时返回异常 |
| x    | 创建写模式（只能新创建内容，否则报错），文件不存在时创建     |
| +    | 与r/w/a/x使用，增加读写功能                                  |
| t    | 文本类型（默认）                                             |
| b    | 二进制类型                                                   |

以及指定文件编码，例如

```python
f = open("test.txt", 'rt', 'uft-8')
```



可以查看打开后的文件编码

```python
f.encoding()
```



### 读取文本

常用的读取方法包括

```python
f.read(size)		# 读取 size 个字符，如果不指定 size 就全部读取，文件指针移动到文件尾
f.readline(size) 	# 读取一行前 size 个字符，若未给定读取整行
f.readlines(size) 	# 读取 size 行并返回以它们为元素组成的列表，若未给定则读取所有行，同时文件指针定位到文件尾
```

注意即使读取中文文本，也是按照字符数计算，而非字节数。



可以使用循环语句查找文本内容

```python
f = open("test.txt", 'rt')

# 遍历每一行，查找 hello 字符串
for line in f:
    if 'hello' in line:
        print(line)

f.close()
```



### 写入文本

常用的写入方法包括

```python
f.write(s) 			# 把 s 写入文件
f.writelines(s) 	# 向文件写入字符串的列表
```

注意 writelines 换行需要自己加入换行符。例如

```python
f = open("test.txt", 'wt')
f.writelines(['hi', '\n', 'yes'])
f.close()
```



### 读写位置

在读写过程中，我们操作的字节位置会发生移动。通过 tell 函数可以看到当前所在的字节位置

```python
f.tell()	# 获得字节位置
```

当需要修改之前的内容时，可以使用

```python
f.seek(offset)	# 指定移动到 offset 字节位置
```

移动后就可以从指定**字节位置**开始重新写入。另外，**在追加模式下，即使指定当前位置，内容始终会写入到最后**。



### 缓存操作

通过 write 写入的内容会先存放到缓存中，直到

```python
f.close()
```

调用时将内容写入硬盘。



如果需要确保内容立即写入，例如输入内容过多，超出缓存范围，就需要执行强制写入

```python
f.flush()
```

可以截断指定位置以后的所有内容

```python
f.truncate()
```

如果不指定，会截断当前位置之后的内容。**截断后的内容会立即写入硬盘**。



### 修改模式

在 `r+` 模式下修改文件

```python
f = open("test.txt", 'r+')
```

但是如果修改的文件中存在中文，可能会出现问题。例如文本内容

```txt
名称----日期
```

在文件开头写入

```python
f.write("天气不错")
```

得到的结果会是

```python
天气不错--鏃ユ湡
```

这是因为中文字符占 3 字节，之前的内容中 "名称----" 共占用了 10 字节，写入的内容占 12 字节，其中多出的 2 字节覆盖了 "日" 的部分内存，导致无法正常读取。



推荐的修改方法是先将文本作为字符串读取，然后通过字符串替换操作进行修改。例如

```python
f = open("test.txt", 'r+')

# 读取并替换
data = f.read()
data_new = data.replace("名称", "天气不错")

# 清空之前的所有内容
f.seek(0)
f.truncate()

# 写入修改的内容
f.write(data_new)

f.close()
```



### 二进制编码

以二进制格式打开的文件，也需要通过二进制格式读写。例如

```python
f = open("test.txt", 'wb')

s = "你好"
s = s.encode("utf-8")
f.write(s)

f.close()
```

这里将字符串以 utf-8 格式编码为二进制格式写入。



相对地，可以对编码内容解析来读取内容。例如

```python
f = open("test.txt", 'rb')

s = f.read().decode("utf-8")
print(s)

f.close()
```

指定编码对读取内容进行解码，获得字符串。

![[Screenshot_20230517_202119.jpg|500]]



## 函数基础

Python 函数的基本定义格式为

```python
def f(x, y):
    print(x, y)
```

其中 `x,y` 都是传入的形式参数，在函数内部对 `x,y` 的**值修改**只在函数内部有效。



但是也有例外。对于列表，传入的是参数地址，因此可以修改列表的内容

```python
def f(x, y):
    x[0] = 3	# 修改 x 列表的元素
    y = 6		# 只在函数内部有效
```



### return

使用 return 返回值并退出函数

```python
def f(x):
    return x * 10
```

还可以定义多个返回值

```python
def f(x, y):
    return x * 10, y * 6
```

返回值类型是一个元组，因此这实际上是返回了元组。同理可以返回列表

```python
def f(x, y):
    return [x * 10, y * 6]
```



### yield

通常序列都具有迭代性质，例如

```python
string = 'hello'
for i in string:
    print(i)
```

在这个过程中就用到了迭代器，它等价于

```python
string = 'hello'

# 获得迭代对象
it = iter(string)
for i in range(len(string)):
    # 将迭代器移动到下一个位置，返回该位置的值
    c = next(it)
    print(c)
```

在使用 in 操作符循环时，会自动调用 `next()` 方法移动迭代器。



#### 生成器

**生成器**是一种可迭代容器，每次迭代时返回一个值，直到抛出 StopIteration 异常。构造方法是

```python
(expr for var in iterator if condition)
```

例如构造如下生成器

```python
a = (x + 3 for x in range(10))
```

生成器可以转换为列表使用，但它自身不能直接输出。使用 next 函数进行迭代

```python
next(a)	# 3
next(a)	# 4
```

可以通过循环获得元素，过程中会自动调用 next 函数

```python
for num in a:
    print(num)
```

我们可以指定迭代结束时的返回值

```python
x = next(a, None)
```

这样可以通过是否返回 `None` 判断是否结束。



使用 yield 关键字，将会返回值，同时使函数“暂停”，当函数被再次调用，将会从“暂停”位置开始。例如

```python
def generator_func():
    print('hello')
    yield 1
    print('world')
    yield 2
    print('A')

# 使用 yield 关键字的函数会产生“生成器”
g = generator_func()
```

通过 next 函数获得返回值

```python
a = next(g)	# 获得 1
b = next(g)	# 获得 2
c = next(g)	# 报错
```



我们可以使用循环来获得可变长度的生成器。例如

```python
def generator_func():
    # 最开始有 700mL 酒
    wine = 700
    while wine > 0:
        # 如果剩下的多于 200mL，就获得 200mL 酒
        if wine > 200:
            wine = wine - 200
            get_wine = 200
        # 否则就获得剩下所有的酒
        else:
            get_wine = wine
            wine = 0
		
        # 返回获得的酒量
        yield get_wine

# 获得生成器
g = generator_func()
for i in g:
    print(f'I got {i}mL wine.')
```

每次 `i` 获得返回的酒量，输出为

```python
I got 200mL wine.
I got 200mL wine.
I got 200mL wine.
I got 100mL wine.
```

利用循环，反复执行循环体；利用 yield 来跳过 wine 变量的初始化过程，确保总的酒量不断减少。



#### send

使用 send 方法向生成器发送消息。例如

```python
def generator_func():
    name = yield 1
    age = yield 0
    print(f'My name is {name}, my age is {age}')
    yield
    
g = generator_func()
next(g)			# 移动到 yield 1 位置
g.send('Li')	# 发送 Li 赋值给 name，移动到 yield 0 位置
g.send(10)		# 发送 10 赋值给 age，移动到最后一个 yield 位置
```

上述代码分别输出

```python
1
0
My name is Li, my age is 10
```



#### close

使用 close 方法结束一个生成器。例如

```python
def generator_func():
    print('hello')
    yield 1
    print('world')
    yield 2
    print('A')

g = generator_func()
next(g)
g.close()
```

一旦调用 close 方法，则之后使用 next 就会报错。



### 参数类型

Python 有如下几种函数参数类型

* 必备参数（位置参数）
* 关键字参数
* 默认参数
* 不定长参数



#### 必备参数

必须传入的参数，否则会报错。并且传入顺序固定，从左到右。例如

```python
def f(name, age, major, phone):
    info = f'''
    Name : {name},
    Age : {age},
    Major : {major},
    Phone: {phone}
    '''
    print(info)


f("Li", 10, 'math', 198)
```



#### 关键字参数

可以通过指定参数名不按照顺序传入。例如

```python
f("Li", age=10, phone=198, major='math')
```

需要注意**关键字参数只能放在必备参数后面**。



可以通过添加 `/` 参数，表示它**左侧的参数只能直接传入**。例如

```python
def func(a, /, b):
    print(a + b)

# a 在 / 左侧，这种传参方式会报错
func(a=5, b=6)
```



#### 默认参数

可以在定义函数时指定参数的默认值，该参数可以不传入。例如

```python
def f(name, age, major, phone='198'):
    info = f'''
    Name : {name},
    Age : {age},
    Major : {major},
    Phone: {phone}
    '''
    print(info)
    

f("Li", age=10, major='math')
```

注意**默认参数只能放在后面定义**。



#### 不定长参数

可以指定传入不定长的参数。例如

```python
def f(name, age, major, phone, *args):
    info = f'''
    Name : {name},
    Age : {age},
    Major : {major},
    Phone: {phone}
    '''
    print(info)
    print("不定长参数元组：", args)	# ('RGB', 'Yellow')


f("Li", 10, 'math', 198, 'RGB', 'Yellow')
```

其中 `args` 是一个元组，存放了**额外传入**的参数。



还可以以指定参数名的方式传入字典类型的参数。例如

```python
def f(name, age, major, phone, *args, **kwargs):
    info = f'''
    Name : {name},
    Age : {age},
    Major : {major},
    Phone: {phone}
    '''
    print(info)
    print("不定长参数元组：", args)		# ('RGB', 'Yellow')
    print("不定长参数字典：", kwargs)	# {'habbit': 'football', 'school': 'zju'}


f("Li", 10, 'math', 198, 'RGB', 'Yellow', habbit='football', school='zju')
```

其中通过**不是函数原本参数名**进行赋值的参数，就会被存放到 kwargs 字典当中。



### 实参拆包

任何可迭代的容器前面增加 `*` 就表示对该变量进行**拆包**，会把其中储存的变量分离为单独的实参。例如

```python
gList = ['x', 1, 2, 'y']
print(*gList)
```

将会把 gList 中的变量分别取出，然后输出。



实参拆包的重要应用是作为函数参数传入

```python
def f(name, age, major, phone):
    info = f'''
    Name : {name},
    Age : {age},
    Major : {major},
    Phone: {phone}
    '''
    print(info)

my_info = ["Li", 10, 'math', 198]
f(*my_info)
```

当我们需要传入一整个列表中的元素时，就可以利用实参拆包简化流程。



如果要**将字典的元素以指定参数名的方式传入**，则使用

```python
def f(name, age, major, phone, *args, **kwargs):
    info = f'''
    Name : {name},
    Age : {age},
    Major : {major},
    Phone: {phone}
    '''
    print(info)
    print("不定长参数元组：", args)		# ()
    print("不定长参数字典：", kwargs)	# {'habbit': 'football', 'school': 'zju'}


my_info = {
    "name" : "Li",
    "age" : 10,
    "major" : "math",
    "phone" : 198,
    "habbit" : 'football',
    "school" : 'zju'
}

# 传入字典
f(**my_info)
```



### 全局变量

在**函数**中定义的变量称为局部变量，在程序一开始定义的变量称为**全局变量**。

* 前者只能在函数内使用，后者则可以在整个程序中使用；
* 当使用重名的局部变量和全局变量时，会**优先使用局部变量**；
* **不能在函数中直接修改全局变量**；

如果想要在函数中引用并修改全局变量，使用 global 声明。例如

```python
name = "Li"


def change_name():
    global name
    name = "Liu"


change_name()
print(name)
```

通常不建议在函数中直接修改全局变量。



可以使用 nolocal 声明，表示某个外部变量不是局部变量

```python
def func():
    a = 5
    def hi():
        nonlocal a
        a = 10

    # 可以在 hi 内部修改外部的变量
    hi()
    print(a)
```



### 匿名函数

匿名函数（lambda 表达式）可以不显式地指定函数。基本语法为

```python
func = lambda arg1,arg2,... : do something
```

其目的仅仅是为了让简单函数的代码能够在一行写完。例如

```python
func = lambda x, y : x * y

def func(x, y):
    return x * y
```

上面两个函数是等价的。



### 高阶函数

函数可以赋值给变量，因此也可以将函数作为参数传入给另一个函数，这就称为**高阶函数**。例如

```python
def add(x, y, f):
    return f(x) + f(y)


def square(x):
    return x * x


print(add(2, 3, square))
```



### 递归函数

函数在定义中调用自身的做法称为递归，这种写法能够极大地简化代码的复杂度。例如下面代码输出汉诺塔游戏的解法

```python
def hanoi(n,x,y,z):
	if n == 1:
		print(x,"-->",z)
	else:
		hanoi(n-1,x,z,y) # 将 n-1 个圆盘从 x 移动到 y
		print(x,"-->",z) # 将第 n 个圆盘从 x 移动到 z
		hanoi(n-1,y,x,z) # 将 n-1 个圆盘从 y 移动到 z
		
n = int(input("请输入汉诺塔层数："))
hanoi(n,"X","Y","Z")
```



### 装饰器

使用装饰器可以在不修改原先函数的情况下增加函数功能。例如

```python
import time

def time_master(func):
    # 定义一个函数，为 func 添加描述
    def call_func():
        print('开始运行程序')
        start = time.time()
        func()
        stop = time.time()
        print('结束程序运行')
        print(f'运行耗时{(stop-start):.2f}秒')
        
    return call_func

# 装饰函数
@time_master
def myfunc():
    time.sleep(2)
    print('Hi')

# 将 myfunc 传入 time_master，执行返回的函数
myfunc()
```

上述代码调用的等价形式是

```python
# 获得计算 myfunc 运行时间的 call_func 函数
myfunc = time_master(myfunc)
myfunc()
```



可以为一个函数增加多个装饰器，例如

```python
def add(func):
    def inner():
        x = func()
        return x + 1

    return inner


def cube(func):
    def inner():
        x = func()
        return x**3

    return inner


def square(func):
    def inner():
        x = func()
        return x**2

    return inner


@add
@cube
@square
def test():
    return 2


print(test())
```

这里会先将 test 传入 square，然后将返回的函数传入 cube，再将返回的函数传入 add，最后调用这个返回的函数。



装饰器可以带有参数，例如

```python
# 最外层传入其它参数
def logger(msg):
    # 这一层需要传入 func，返回内层函数
    def inner1(func):
        # 最内层输出描述信息，调用 func
        def inner2():
            print(f'This is {msg} called.')
            func()

        return inner2
    return inner1


@logger(msg='A')
def funcA():
    print('funcA is called.')


@logger(msg='B')
def funcB():
    print('funcB is called.')


funcA()
funcB()
```

上述代码调用的等价形式是

```python
# 先传入参数到 logger，然后用返回的函数包装 funcA
funcA = logger(msg='A')(funcA)
funcB = logger(msg='B')(funcB)

funcA()
funcB()
```



### 函数文档

我们知道通过 help 函数可以查看某个函数的使用方法，但事实上可以自定义函数文档。例如

```python
def power(x, n=2):
    '''
    功能：计算 x 的 n 次幂
    参数：
    - x 底数
    - n 次数，默认 2 次
    返回值：
    - x^n
    '''
    return x**n


print(power(2, 5))
help(power)
```

通过 help 就可以输出函数中描述的字符串内容。



### 类型注释

Python 允许显式地表明一个函数的参数类型和返回值类型。例如

```python
def times(s:list[int], n:int) -> list:
    return s * n

print(times([1, 2, 3, 5], 5))
```

它表明一个函数**最好应该具有**怎样的参数类型，不过传入参数不会产生错误时，也不会报错。



## 面向对象

面向对象程序设计 (OOP) 由五个最基本的概念组成：类(class)、对象(object)、方法(method)、消息(message)、继承(inheritance) 。

 

类的定义格式为

```python
class MyClass:
    # 成员属性
    name = 'Li'
    age = 10
    
    # 成员函数
    def sayHello(self):
        # 通过 self 获得成员变量
        print(f'Hello, My name is {self.name}.')
        
    def sayBye(self):
        print('Bye')

```

其中 self 参数和 C++ 中类的 this 指针类似，只在定义时传入，调用类方法时并不需要。例如

```python
myClass = MyClass()
myClass.sayHello()
```



### 访问控制

在Python中，以下划线开头的方法名和变量名有特殊的含义， 尤其是在类的定义中。 

* `_xxx` 当做内部名，不应该在外部使用，不能用 `from module import *` 导入；
* `__xxx` 解释器会重命名，不能使用对象直接访问到这个成员；
* `__xxx__` 系统定义的特殊成员；不要创建这种标识符；



Python 通过重命名指定的成员变量实现对成员变量访问的控制。例如

```python
class MyClass:
    __name = 'Li'
    
    def sayHello(self):
        print(f'Hello, My name is {self.__name}.')

```

此时无法访问 `__name` 变量。但是实际上，Python 是将 `__name` 重命名为 `_MyClass__name`，因此可以访问

```python
myClass._MyClass__name
```

即使用 `_ClassName__varName` 可以获得“私有”变量。



### 类信息

可以使用一些函数获得、修改类具有的信息。

| 函数                             | 作用                                                 |
| -------------------------------- | ---------------------------------------------------- |
| issubclass(class, classinfo)     | 判断 class 类是否是对象元组 classinfo 中某个类的子类 |
| isinstance(object, classinfo)    | 判断 object  是否是对象元组 classinfo 中某个类的实例 |
| hasattr(object, name)            | 判断 object 是否有属性 name                          |
| getattr(object, name[ ,default]) | 获得 object 的 name 属性值，如果不存在，返回 default |
| setattr(object, name, value)     | 设置 object 的 name 属性值                           |
| delattr(object, name)            | 删除 object 的 name 属性，不存在则会报错             |

注意这里 **name 是变量名字符串**。



#### property

使用 property 方法设置一个属性的获取、修改、删除函数。例如

```python
class MyClass:
    def __init__(self, size):
        self.size = size

    def getSize(self):
        return self.size

    def setSize(self, value):
        self.size = value

    def delSize(self):
        del self.size
	
    # 增加变量 x，它调用 getSize 获得值，setSize 修改值，delSize 删除值
    x = property(getSize, setSize, delSize)


c = MyClass(10)
c.x = 10
print(c.x)
del c.x
```

通过这种方式，能够操作 `c.x` 来操作 `c.size` 变量值的获得、修改和删除。



利用装饰器，可以实现类的只读属性。例如

```python
class A:
    def __init__(self):
        self._x = 50
        
    @property
    def x(self):
        return self._x


a = A()
print(a.x)	# 输出 50
a.x = 5		# 将会报错
```

根据装饰器的原理，上述定义等价于

```python
class A:
    def __init__(self):
        self._x = 50
        
    def x(self):
        return self._x
    
    # 由于只传入了获得值的方法，没有修改和删除的方法，因此 x 成为只读变量
    x = property(x)
```



#### 描述符

描述符指将某种特殊类型的类的实例作为另一个类（被描述的类）的属性。基本方法为

```python
__get__(self, instance, owner)	# 访问属性，返回属性值
__set__(self, instance, value)	# 分配属性值
__delete__(self, instance)		# 删除属性
```

这里 instance 是被描述的类的实例，owner 是被描述的类的名称。



通过描述符，我们可以实现 property 方法

```python
class MyProperty:
    # 传入三个函数作为属性
    def __init__(self, fget=None, fset=None, fdel=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
    
    # 当访问属性时，会调用 fget 函数
    def __get__(self, instance, owner):
        return self.fget(instance)

    # 当设置属性时，会调用 fset 函数
    def __set__(self, instance, value):
        self.fset(instance, value)

	# 当删除属性时，会调用 fdel 函数
    def __delete__(self, instance):
        self.fdel(instance)


class C:
    def __init__(self):
        self._x = None

    def getX(self):
        return self._x

    def setX(self, value):
        self._x = value

    def delX(self):
        del self._x
	
    # 创建描述类实例，传入三个函数
    x = MyProperty(getX, setX, delX)


c = C()
c.x = 100
print(c.x)
print(c._x)
del c.x
```

这里 MyProperty 是描述类，C 是被描述类。



使用描述符，我们可以将算法分别写在几个类中，然后将它们统一放在一个类中描述属性。例如下面的温度转换代码

```python
# 转换为摄氏度的方法
class Celsius:
    # 保存并初始化温度值
    def __init__(self,value = 26.0):
        self.value = float(value)
	
    # 正常获得值
    def __get__(self,instance,owner):
        return self.value
    
    # 正常设置值
    def __set__(self,instance,value):
        self.value = float(value)

# 转换为华氏度的方法
class Fahrenheit:
    # 当获得此值，会用 instance 中 cel 的值计算
    def __get__(self,instance,owner):
        return float(instance.cel) * 1.8 + 32
    
    # 当修改此值，会用传入的值转化为摄氏度，修改 instance 中 cel 的值
    def __set__(self,instance,value):
        instance.cel = (float(value) - 32) /1.8
	
class Temp:
    cel = Celsius()
    fah = Fahrenheit()

    
temp = Temp()
temp.cel = 30
print(temp.fah)
temp.fah = 100
print(temp.cel)
```

在 Temp 类中的 cel 实例保存了温度值，而 fah 实例用于转换温度值。当修改 cel，然后获得 fah 时，就会调用 `__get__` 计算对应的华氏度；当修改 fah 时，会调用 `__set__` 计算并修改对应的摄氏度。



### 构造方法

#### 构造器

Python 类的构造函数统一为 `__init__`，如果没有定义，则会使用默认的空构造函数。例如

```python
class Turtle:
    # 在构造函数中定义成员变量
    def __init__(self, x):
        self.num = x

class Fish:
    def __init__(self, x):
        self.num = x

class Pool:
    def __init__(self, x, y):
        self.turtle = Turtle(x)
        self.fish = Fish(y)

    def print_num(self):
        print('水池里总共有乌龟 %d 只，小鱼 %d 条！' % (self.turtle.num,self.fish.num))

        
pool = Pool(5, 6)
pool.print_num()
```



#### 析构器

Python 类的析构函数统一为 `__del__`，如果没有定义，则会使用默认的空析构函数。例如

```python
class MyClass:
    def __init__(self, name):
        self.name = name

    def __del__(self):
        print(f'My name is {self.name}, I was deleted.')

        
myClass = MyClass('Li')
del myClass
```

如果**对某个实例的所有引用都被删除**，则会调用析构函数。



#### new

Python 实例化对象时，会先调用这个函数构造类的实例，为其分配内存，然后返回实例，传入构造函数。例如

```python
class Object:
    pass

class Person(Object):
    def __new__(cls):
        print("__new__ called")
        # 这里调用父类的 __new__ 方法获得实例，返回值作为 __init__ 的参数
        return super().__new__(cls)

    def __init__(self):
        print("__init__ called")


a = Person()
```

上述代码输出

```python
__new__ called
__init__ called
```

如果 `__new__` 没有返回值，则不会调用 `__init__` 方法。



如果 `__new__` 返回一个其它类的实例，则不会调用它自身的 `__init__` 方法，而是调用对应类的 `__init__` 方法

```python
class Object:
    pass

class Animal(Object):
    def __init__(self):
        print("Animal __init__ called")

class Person(Object):
    def __new__(cls):
        print("__new__ called")
        return Animal()

    def __init__(self):
        print("Person __init__ called")


a = Person()
```

此时将会输出

```python
__new__ called
Animal __init__ called
```



默认情况下，`__new__` 方法的其它参数与 `__init__` 方法的其它参数相同。但是如果要自定义 `__new__`，需要确保参数一致

```python
class Object:
    pass

class Person(Object):
    def __new__(cls, name):
        print("__new__ called")
        return super().__new__(cls)

    def __init__(self, name):
        print("__init__ called")
        self.name = name


a = Person('Li')
print(a.name)
```

或者使用可变长度参数

```python
class Person(Object):
    def __new__(cls, *args, **kwargs):
        print("__new__ called")
        return super().__new__(cls)

    def __init__(self, name):
        print("__init__ called")
        self.name = name
```



使用 `__new__` 的一个场景是修改那些不可修改的属性。例如字符串 str 不可修改，我们可以重写 `__new__` 来预处理字符串，将它们转换为大写，然后再进行初始化

```python
class CapStr(str):
    def __new__(cls, string):
        string = string.upper()
        return super().__new__(cls, string)
    
   
a = CapStr("I am Li.")
print(a)
```



### 类属性

#### 静态属性

Python 中的类内可以直接定义变量，可以看作**静态变量**。例如

```python
class MyClass:
    count = 10
```

此时 count 是类 `MyClass` 的静态成员。可以直接通过类名修改

```python
MyClass.count = 100
```



#### 动态添加变量

实例化的类对象甚至可以**随意增加成员变量**，例如

```python
c = MyClass()
c.x = 5
```

这里 x 这个成员变量是在实例化后创建出来的，但是显然不建议这样使用。并且这种创建方法可能会覆盖类的成员，例如

```python
class MyClass:
    count = 10
    
    def x(self):
        print(x)
        
c = MyClass()
c.x = 5			# 对实例对象，新创建的 x 覆盖了成员函数 x
c.count = 100	# 对实例对象，新创建的 count 覆盖了成员变量 count
```



#### 属性字典

通常类会带有预定义的成员变量，用 `__` 包围。例如

| 变量       | 含义                 |
| ---------- | -------------------- |
| `__dict__` | 类或实例的属性的字典 |

我们甚至可以通过它来修改类属性，例如

```python
class A:
    def __init__(self):
        self.name = 'Li'
        
    def sayHello(self):
        print(f'Hello, My name is {self.name}.')

a = A()
a.sayHello()
a.__dict__['name'] = 'Liu'
a.sayHello()
```



使用 `__slots__` 属性可以限制动态添加属性，例如

```python
class A:
    __slots__ = ['x', 'y']
    
    def __init__(self, x, y):
        self.x = x
        self.y = y
        
a = A()
a.z = 10	# 将会报错
```



### 类继承

Python 中类继承的格式为

```python
class BaseClass1:
    ...

class BaseClass2:
    ...
 
# 继承两个父类
class ChildClass(BaseClass1, BaseClass2):
    ...
```

如果派生类没有定义构造函数，则定义派生对象时，会自动调用基类的构造函数；对于析构函数同理。



#### 基类构造

如果需要增加派生类的构造函数，应当在其中调用基类的构造函数

```python
class MyClass:
    def __init__(self, name):
        self.name = name

    def sayHello(self):
        print(f'Hello, My name is {self.name}.')
    

class ChildClass(MyClass):
    def __init__(self):
        # 调用父类构造函数初始化
        MyClass.__init__(self, 'child')

        
child = ChildClass()
child.sayHello()
```

子类可以直接调用父类的成员函数，获得父类的成员变量。



#### super

在子类中更建议使用 `super()` 方法调用父类函数

```python
import random as r

class Fish:
    def __init__(self):
        # 生成随机坐标
        self.x = r.randint(0,10)
        self.y = r.randint(0,10)

    def move(self):
        self.x -=1
        print('我的位置是：', self.x, self.y)

class Goldfish(Fish):
    # 占位符，什么都不做，因此具有和 Fish 相同的属性和方法
    pass

class Carp(Fish):
    pass

class Salmon(Fish):
    pass

class Shark(Fish):
    def __init__(self):
        # 通过 super() 调用父类函数
        super().__init__()
        self.hungry = True

    def eat(self):
        if self.hungry:
            print('吃货的梦想就是天天有的吃^_^')
            self.hungry = False
        else:
            print('太撑了，吃不下了！')

```

使用 `super()` 能够更直接地看到父类的成员函数，如果直接通过类名调用，可能会产生混淆，因为**即使没有继承关系，也可以在一个类中调用另一个类的方法**。注意**`super()` 方法不需要传入 self 参数**。



#### 类参数

继承类传入的父类实际上作为一种参数传入。例如

```python
class A:
    def __init_subclass__(cls):
        print('This is A.')
        cls.x = 520
        
class B(A):
    x = 250
    
print(B.x)
```

这里 A 实际上作为参数传入 `__init_subclass__` 方法，其中 `cls.x` 的赋值覆盖了 B 原本 x 的值。



甚至于我们可以这样写

```python
class A:
    def __init_subclass__(cls, value):
        print('This is A.')
        cls.x = value
        
class B(A, value=10):
    x = 250
    
print(B.x)
```

此时我们把一个值“当做子类”传入了，但是也可以运行，得到 x 的值是 10 。



### 实例方法

类的方法，即类中定义的函数。类中定义的所有函数都是静态函数，即用一块内存专门存放该函数，即使删除了类的定义，仍然可以通过对象实例调用。例如

```python
class MyClass:
    def x(self):
        print('Hello')
        
        
c = MyClass()
del MyClass
c.x()
```

这种形式定义的函数称为**实例方法**，它由 `self` 参数开头，是最常用的定义方法的方式。



### 默认方法

#### 运算方法

Python 提供了各种运算符对应的操作函数作为类的成员函数。例如

| 方法                      | 作用     | 方法                     | 作用     |
| ------------------------- | -------- | ------------------------ | -------- |
| `__add__(self, obj)`      | 加法运算 | `__sub__(self,obj)`      | 减法运算 |
| `__mul__(self,obj)`       | 乘法运算 | `__truediv__(self, obj)` | 除法运算 |
| `__floordiv__(self, obj)` | 整除运算 | `__mod__(self, obj)`     | 取模运算 |
| `__lt__(self, obj)`       | 小于     | `__le__(self, obj)`      | 小于等于 |
| `__gt__(self, obj)`       | 大于     | `__ge__(self, obj)`      | 大于等于 |
| `__eq__(self, obj)`       | 等于     | `__ne__(self, obj)`      | 不等于   |

类似地大部分基本运算符都可以通过重定义对应的方法设置。需要注意避免在重定义时产生递归调用，例如

```python
class NewInt(int):
    def __add__(self, obj):
        return self + obj


a = NewInt(5)
b = NewInt(3)
c = a + b
print(c)
```

这里重定义 `int` 子类的加法，函数内部又调用了加法，因此产生了无限递归。



另外，还有 `__radd__` 等对应**反运算符**，表示“被加”运算，当“加”运算不支持时，就会调用。例如

```python
class NewInt(int):
    def __radd__(self, obj):
        return super().__sub__(obj)
    

a = NewInt(5)
b = NewInt(3)
print(a + b)	# NewInt 有继承的 __add__ 方法，得到 5 + 3 = 8
print(1 + b)	# int 和 NewInt 之间没有 __add__ 方法，调用 __radd__ 得到 3 - 1 = 2
```



#### 类型转换

Python 提供默认的方法，重写后用于类型转换。例如

| 方法              | 作用       | 方法                | 作用         |
| ----------------- | ---------- | ------------------- | ------------ |
| `__int__(self)`   | 转换为整型 | `__complex__(self)` | 转换为复数   |
| `__float__(self)` | 转换为实数 | `__str__(self)`     | 转换为字符串 |



#### 属性方法

在类信息一节，我们介绍了获得类实例的相关属性的函数。Python 允许在类中重定义相关的属性成员函数，例如

```python
class C:    
    def __init__(self):
        self.x = 10
    
    # 当试图获得类不存在的属性时，会调用此函数
    def __getattr__(self,name):
        print('getattr')
        
    # 当获得属性时调用此函数，返回属性值
    def __getattribute__(self,name):
        print('getattribute')
        return super().__getattribute__(name)
        
    # 当修改属性时调用此函数，设置属性值
    def __setattr__(self,name,value):
        print('setattr')
        super().__setattr__(name,value)
        
    # 当删除属性时调用此函数，删除属性值
    def __delattr__(self,name):
        print('delattr')
        super().__delattr__(name)

        
c = C()
getattr(c, 'x')
setattr(c, 'x', 100)
delattr(c, 'x')
getattr(c, 'x')
```

注意这里 `super()` 能**调用原本的默认方法**，防止出现无限递归。



在重定义过程中有可能出现死循环，例如

```python
class Rectangle:
    def __init__(self,width=0,height=0):
        # 这里 name == 'width' 和 'height' 分别传入 __setattr__
        self.width = width
        self.height = height

    def __setattr__(self,name,value):
        # 进入 if 分支后，又会将 name == 'width' 和 'height' 分别传入 __setattr__，将会进入 else 分支
        if name == 'square':
            self.width = value
            self.height = value
        # 初始化赋值将会进入 else 分支
        else:
            # 如果这样写就会再次进入 __setattr__，此时 name == 'square'，将会进入 if 分支
            self.square = value 

    def getArea(self):
        return self.width * self.height

    
# 构造函数报错
r = Rectangle(10, 10)
```

出现死循环的问题在于 `if` 和 `else` 两个分支调用的 `__setattr__` 方法又分别进入对方的分支中。我们修改为

```python
class Rectangle:
    def __init__(self,width=0,height=0):
        # 这里 name == 'width' 和 'height' 分别传入 __setattr__
        self.width = width
        self.height = height

    def __setattr__(self,name,value):
        # 进入 if 分支后，又会将 name == 'width' 和 'height' 分别传入 __setattr__
        if name == 'square':
            self.width = value
            self.height = value
        # 初始化赋值将会进入 else 分支
        else:
            # 调用默认赋值，就不会进入当前类的 __setattr__
            super().__setattr__(name,value)
            # 另一种方法是 self.__dict__[name] = value

    def getArea(self):
        return self.width * self.height

    
r = Rectangle(10, 10)
setattr(r, 'square', 5)
print(r.getArea())
```



#### 自我描述

重定义 `__repr__` 方法用于提供类的自我描述，它将对象转换为字符串作为参数传入。例如

```python
class A:
    def __repr__(self):
        return "Hello"
   

a = A()
print(a)
```

打印函数会输出类的自我描述信息。



#### 容器类型

协议（Protocols）与其它编程语言中的接口相似，它规定哪些方法必须要定义。

* 如果希望定制不可变容器，只需要定义 `__len__()` 和 `__getitem__` 方法；
* 如果希望定制可变容器，还需要定义 `__setitem__()` 和 `__delitem__` 方法；

如果需要更高级的操作，可以定义 `__reversed__` (反转操作)，`__contains__` (in, not in 操作) 等方法。



利用我们定制一个不可变容器，记录每个元素被访问的次数

```python
class CountList:
    # 传入数据作为列表 values，同时用 count 记录每个数据被访问的次数
    def __init__(self,*args):
        self.values = [x for x in args]
        self.count = {}.fromkeys(range(len(self.values)),0)

	# 定义此方法，可以用 len() 获得容器长度
    def __len__(self):
        return len(self.values)

    # 定义此方法，可以用 [] 访问容器元素
    def __getitem__(self,key):
        self.count[key] += 1
        return self.values[key]

    
c = CountList(1, 5, 6, 8)
print(len(c))
print(c[1])
print(c[3])
print(c.count)
```



#### 迭代方法

我们可以为一般类提供迭代属性，只需要重定义 `__iter__, __next__` 方法

```python
class Fibs:
    def __init__(self, maxn):
        self.a = 0
        self.b = 1
        self.maxn = maxn

    def __iter__(self):
        return self

    def __next__(self):
        self.a, self.b = self.b, self.a + self.b
        
        # 当超过 maxn 时，发起特定的异常，从而停止迭代
        if self.a > self.maxn:
            raise StopIteration

        return self.a

# 输出不超过 50 的斐波那契数列
fibs = Fibs(50)
for each in fibs:
    print(each)
```

需要注意这里迭代器移动后，fibs 中的 a, b 属性都已经修改，如果想再次输出不超过 50 的斐波那契数列，需要重新初始化。



#### 仿函数

重定义 `__call__` 方法可以让类像函数一样调用，例如

```python
class Power:
    def __init__(self, exp):
        self.exp = exp
        
    def __call__(self, base):
        return base ** self.exp
    
square = Power(2)
print(square(4))
```

这里定义一个幂类，产生一个平方实例对象，这样调用这个对象获得 2 的 4 次方。



### 删除方法

上面介绍的通过双下划线定义的方法通常是类自带的方法，可以去掉这些方法。例如

```python
class A:
    __contains__ = None
```

在类继承时，也可以在子类中删除不需要的父类方法。



### 类方法与静态方法

类方法是类拥有的方法，需用修饰器 @classmethod 来标识，第一个参数通常用 cls 表示，表示类对象。使用时不需要传入 cls 参数，建议通过类名直接调用。例如

```python
class A:
    @classmethod
    def info(cls):
        print('类方法被调用',cls)


a = A()
a.info()
A.info()
```



静态方法需要用 @staticmethod 来标识，不需要 self 或 cls 参数。静态方法只能获得类的静态成员，调用类方法或静态方法。例如

```python
class Test:
    name = 'show'
    
    def __init__(self,x):
        self.x = x
        
    @classmethod
    def getName(cls):
        print('getname:',Test.name)
        
    @staticmethod
    def display():
        Test.getName()
        print('display:',Test.name)
        
    def say(self):
        print('say:',self.x)
        print('say:',Test.name)

        
test = Test(10)
test.say()
Test.display()
```



类方法和静态方法区别就在于是否会传入对象。利用这一区别，前者用于处理对象属性，后者处理与对象无关的属性。我们主要介绍类方法特性的妙用，例如

```python
class A:
    count = 0
    
    def __init__(self):
        self.add()
    
    @classmethod
    def add(cls):
        cls.count += 1
        
    @classmethod
    def getCount(cls):
        print(f'{cls.__name__} 类实例化了 {cls.count} 个对象')
        
        
class B(A):
    # 覆盖 A 中的 count
    count = 0

class C(A):
    # 覆盖 A 中的 count
    count = 0


a = A()
b1, b2 = B(), B()
c1, c2, c3 = C(), C(), C()
a.getCount()
b1.getCount()
c1.getCount()
```

每当子类产生了实例化对象，就会调用 add，同时**传入该子类 cls**，然后增加该类的计数。因此每个类都是单独计数，输出结果也是每个类各自的统计结果。



### 装饰器

#### 函数装饰类

可以用函数装饰类，在类实例化之前进行处理。例如

```python
def report(cls):
    def onCall(*args, **kwargs):
        print('即将开始实例化')
        c = cls(*args, **kwargs)
        print('实例化完成')
        return c
    
    return onCall
    

@report
class A:
    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z
        print('调用构造函数')

a = A(1, 2, 3)
```

这里上面构造代码等价于

```python
class A:
    def __init__(self):
        print('调用构造函数')

A = report(A)	# 这里 A 变成 onCall 函数
a = A(1, 2, 3)	# 调用 onCall 函数，返回 A 的实例化对象
```



#### 类装饰器

甚至可以用类作为装饰器，例如

```python
class Counter:
    def __init__(self, func):
        self.func = func

	# 定义 call，使得类可以像函数一样调用
    def __call__(self, *args, **kwargs):
        print('Counter Called')
        return self.func(*args, **kwargs)


@Counter
def sayHi():
    print('Hi')

sayHi()
```

这里上面构造代码等价于

```python
sayHi = Counter(sayHi)	# 这里 sayHi 变成 Counter 类实例
sayHi()					# 调用 Counter 类实例，此时 __call__ 被调用，然后调用传入的 sayHi 函数
```

类似地，这种具有仿函数的类也可以用来装饰类。



### 元类

#### type

我们知道通过 type 函数可以获得变量的类型，并用于类型检测，例如

```python
type(250)
type('A') is str
```

这里我们发现可以直接用 type 获得的类型定义变量

```python
a = type('A')(520)	# 等价于 str(520)
print(a)
```

但是我们可以注意到

```python
class A:
    pass

type(A) is type		# True
type(type) is type	# True
```

这是因为 Python 中的所有类本身也是对象，是由 type 类创建出的类。



使用 type 可以创造类，语法为

```python
type(name, bases, dict, **kwds)
```

参数分别是类名、父类元组、属性和方法的字典，以及 `__init_subclass__` 的参数。例如

```python
def func(self, name):
    print(f'Hi, {name}')

# 创建 A 类，没有父类（默认为 Object），添加属性 x,y 和方法 sayHi
A = type('A', (), dict(x=250, y=520, sayHi=func))
a = A()
print(a.x)
a.sayHi('Li')

# 创建 B 类，继承 A 类
B = type('B', (A,), dict(z=666))
b = B()
print(b.x)
print(b.z)
```



最后一个参数使用案例如下

```python
class A:
    def __init_subclass__(cls, value):
        print('This is A.')
        cls.x = value
        
B = type('B', (A,), dict(x=666), value=10)
print(B.x)
```

此时 value 传入 `__init_subclass__` 覆盖了 x 的赋值。



#### metaclass

我们已经知道 type 可以创建所有类，实际上 type 是一个元类，它是类创建的根基。我们可以自定义元类

```python
# 继承 type 得到元类
class MetaC(type):
    pass
```

通过这个元类创建类

```python
class C(metaclass=MetaC):
    pass
```

通过 type 可以看到

```python
type(C) is MetaC
```

因此自定义元类介于 type 和一般类之间。



我们可以通过元类拦截类的定义过程，例如

```python
class MetaC(type):
    def __new__(mcls, name, bases, attrs):
        print('__new__ in MetaC')
        return type.__new__(mcls, name, bases, attrs)
    
    def __init__(cls, name, bases, attrs):
        print('__init__ in MetaC')
        type.__init__(cls, name, bases, attrs)
        
        
class C(metaclass=MetaC):
    def __new__(cls):
        print('__new__ in C')
        return super().__new__(cls)
    
    def __init__(self):
        print('__init__ in C')
        
        
```

可以注意到，上述代码没有任何调用，但是会输出 MetaC 中的信息。这是因为 MetaC 在创建 C 类时，调用了自定义的两个方法。



通过元类，甚至可以拦截类的实例化过程，例如

```python
class MetaC(type):
    def __call__(cls, *args, **kwargs):
        print('__call__ in MetaC')
        
        
class C(metaclass=MetaC):
    pass
        
c = C()			# 输出 __call__ in MetaC
print(type(c))	# c 是 None
```

实际上，类的实例化过程就是将元类作为仿函数调用，因此我们重写了 `__call__` 后，就不会得到类的实例，此时 c 是 None 。



#### 控制属性

可以通过元类控制类的创建和构造过程。例如

```python
class MetaC(type):
    def __new__(mcls, name, bases, attrs):
        # 给实例对象添加成员属性
        attrs['name'] = 'Li'
        return type.__new__(mcls, name, bases, attrs)
    
    def __init__(cls, name, bases, attrs):
        # 给类添加属性
        cls.age = 10
        type.__init__(cls, name, bases, attrs)
        
        
class C(metaclass=MetaC):
    pass
        
c = C()
print(c.name)	# 实例属性 name
print(C.age)	# 类属性 age
```



#### 规范类名

可以通过元类检查类名，当类名不满足要求时报错。例如

```python
class MetaC(type):
    def __init__(cls, name, bases, attrs):
        if not name.istitle():
            raise TypeError('类名不是大写字母开头！')
        type.__init__(cls, name, bases, attrs)
        
        
class c(metaclass=MetaC):
    pass
        
# 此类定义将会报错
```



#### 修改属性

可以通过元类修改属性值。例如

```python
class MetaC(type):
    def __call__(cls, *args, **kwargs):
        # 将所有字符串属性值变成大写
        new_args = [each.upper() for each in args if isinstance(each, str)]
        return type.__call__(cls, *new_args, **kwargs)
        
        
class C(metaclass=MetaC):
    def __init__(self,name):
        self.name = name
        
c = C('abc')
print(c.name)
```



#### 限制传参

可以让类构造时只能通过关键字传参。例如

```python
class MetaC(type):
    def __call__(cls, *args, **kwargs):
        # 要求 args 为空，只能用 kwargs 传参
        if args:
            raise TypeError('只能通过关键字传参')
        return type.__call__(cls, *args, **kwargs)
        
        
class C(metaclass=MetaC):
    def __init__(self,name):
        self.name = name
        
c = C(name='abc')
```



#### 禁止实例化

可以禁止一个类实例化。例如

```python
class NoInstance(type):
    def __call__(cls, *args, **kwargs):
        raise TypeError('不允许实例化！')
        
        
class C(metaclass=NoInstance):
    @classmethod
    def classOK(cls):
        print('可以调用类方法')
    
    @staticmethod
    def staticOK():
        print('可以调用静态方法')
        
C.classOK()
C.staticOK()
c = C()	# 将会报错
```



这种写法的主要用途之一就是**单例模式**。例如

```python
class SimpleInstance(type):
    def __init__(cls, name, bases, attrs):
        # 定义一个类实例属性为空
        cls.__instance = None
        type.__init__(cls, name, bases, attrs)
    
    def __call__(cls, *args, **kwargs):
        # 只有当类实例不存在，才能够构造类实例
        if cls.__instance is None:
            cls.__instance = type.__call__(cls, *args, **kwargs)
            
        return cls.__instance
        
        
class C(metaclass=SimpleInstance):
    pass
        
c1 = C()
c2 = C()
print(c1 is c2)
```

这里 c1 和 c2 是同一个实例。



#### 抽象基类

通过阻止类实例化，可以实现类似 C++ 的抽象基类功能。需要导入 abc 模块，然后用它们定义抽象基类

```python
from abc import ABCMeta, abstractmethod

class Fruit(metaclass=ABCMeta):
    # 抽象方法
    @abstractmethod
    def health(self):
        pass
    
    
class Banana(Fruit):
    def health(self):
        print('Banana is good for health.')
        
        
banana = Banana()
banana.health()
```

与 C++ 类似，抽象基类不可实例化，并且子类必须定义抽象基类中的抽象方法后，才可以实例化。



## 模块与包

### 模块

在 python 中可以通过 import 来导入 python 文件，从而增强代码功能。语法为

```python
import mod 						# 完全导入一个 mod 模块，通过 mod. 调用函数
import mod as name				# 导入 mod 模块，用 name 命名，通过 name. 调用函数
from mod import *				# 导入模块中所有函数，直接调用函数
from mod import func			# 引入模块中一个函数，只能调用 func
from mod import func as name	# 引入模块中一个函数，命名为 name，只能调用 name
```

这里的 name 实际上相当于命名空间。



任何模块都有 `__name__` 属性，它就是模块的名称。只有模块名为 `__main__` 的文件才会被执行。通过这种方式来区分哪些代码需要执行，哪些不需要执行。例如

```python
# TemperatureConversion.py 模块

def c2f(cel):
    fah = cel *1.8 + 32
    return fah
    
def f2c(fah):
    cel = (fah - 32) / 1.8
    return cel

def test():
    print('0摄氏度 = %.2f华氏度' % c2f(0))
    print('0华氏度 = %.2f摄氏度' % f2c(0))

if __name__ == '__main__':
    test()
```

如果直接执行这个模块，则名称就是 `__main__`，测试代码将会执行；如果导入到其它文件

```python
import TemperatureConversion as TC

print(TC.__name__)
print('32摄氏度 = %.2f华氏度' % TC.c2f(32))
print('99华氏度 = %.2f摄氏度' % TC.f2c(99))
```

则 `TemperatureConversion.py` 的名称是 `TemperatureConversion`，因此不会执行测试代码。



### 包

在 sys 库中的 path 属性存放了 Python 搜索模块的路径。我们当然可以通过向 path 中添加路径来配置模块，但是这样会全局修改路径。更好的方法是用一个文件夹存放模块，然后通过文件夹名称导入模块。例如

└─pack

在 pack 文件夹下创建空的 `__init__.py` 文件，以及

```python
# add.py

def add(a, b):
    return a + b

# minus.py
def minus(a, b):
    return a - b
```

两个文件作为模块，然后在主目录下引入包的模块

```python
import pack.add as a
import pack.minus as m

print(a.add(1, 1))
print(m.minus(1, 1))
```

这里 pack 就是一个**包**，其中包含了所有的模块。



如果需要在包中调用包中的模块，就直接引入当前目录下的模块，例如 pack 目录下新建的 `test.py` 文件引用 `add.py` 模块

```python
# test.py
from . import add
```



如果调用

```python
import pack
```

此时将会调用 pack 中的 `__init__.py` 文件，决定要导入哪些模块。需要注意，使用

```python
from pack import *
```

**不能导入所有模块**，因为这里 `__init__.py` 文件是空的，我们应该增加

```python
__all__ = ['add', 'minus']	# 设置需要导入的模块
```

此时就可以调用模块的所有函数

```python
from pack import *

print(add.add(1, 1))
print(minus.minus(1, 1))
```



## 第三方库

除了 Python 自带的库，还可以通过 pip 在命令行安装第三方库

```shell
pip install ...
```



### random

随机数库

```python
import random
```

常用方法如下

| 函数                               | 作用                                      | 函数         | 作用                                                   |
| ---------------------------------- | ----------------------------------------- | ------------ | ------------------------------------------------------ |
| shuffle()                          | 使列表元素随机排列                        | choice()     | 从序列中随机取一个元素                                 |
| random()                           | 返回一个 [0,1) 上的浮点数                 | randint(a,b) | 返回一个 [a,b) 上的整型                                |
| seed(a)                            | 以 a 为随机数公式的输入（默认为系统时间） | uniform(a,b) | 返回一个 [a,b) 上的浮点数                              |
| randrange([start=0],stop[,step=1]) | 返回指定范围上的一个数                    | sample(a,k)  | 返回 a 中一个长度为 k 的序列的元素随机排列后得到的序列 |



### os

系统目录和文件操作库，用于文件名和路径操作。包括

* 创建、修改和删除文件、目录的函数
* 修改当前目录的函数
* 运行 shell 命令的函数

其中的 path 模块可以

* 对文件名称进行操作
* 进行路径判断操作



下面是一个使用 oc 库输出给定目录下的目录结构的程序

```python
import os as o

# 输出函数
def list_files(startpath):
    # 遍历路径中的目录和文件，root 是当前路径字符串，dirs、files 均为列表
    for root,dirs,files in o.walk(startpath):    
        # o.sep 得到路径的分隔符 “\\”，level 获得路径中 “\\” 的个数，即路径的深度
        level = root.replace(startpath,'').count(o.sep)
        
        # 绘制缩进结构
        dir_indent = '|    ' * (level) + '|--'
        file_indent = '|    ' * (level + 1) + '|--'
        
        if not level:
            print('{}'.format(startpath[0]))		# 输出 D
            print('|--{}'.format(startpath[3:]))	# 输出 Cmder
        else:
            # 输出子目录名
            print('{}{}'.format(dir_indent,o.path.basename(root)))
        
        # 遍历输出所有文件名
        for f in files:
            print('{}{}'.format(file_indent,f))

# 目标路径
startpath = 'D:\Cmder'
list_files(startpath)
```



### pathlib

在 Python 3.4 以后，pathlib 作为标准库模板，可以

* 统一管理
* 跨平台操作
* 面向对象，路径处理更灵活方便
* 简化操作，简单易用



与 os 操作的详细对比如下：

| pathlib 操作           | os 操作              | 功能描述                                                     |
| ---------------------- | -------------------- | ------------------------------------------------------------ |
| Path.resolve()         | os.path.abspath()    | 获得绝对路径                                                 |
| Path.chmod()           | os.chmod()           | 修改文件权限和时间戳                                         |
| Path.mkdir()           | os.mkdir()           | 创建目录                                                     |
| Path.rename()          | os.rename()          | 文件或文件夹重命名，如果路径不同，会移动并重新命名           |
| Path.replace()         | os.replace()         | 文件或文件夹重命名，如果路径不同，会移动并重新命名，如果存在，则破坏现有目标 |
| Path.rmdir()           | os.rmdir()           | 删除目录                                                     |
| Path.unlink()          | os.remove()          | 删除一个文件                                                 |
| Path.unlink()          | os.unlink()          | 删除一个文件                                                 |
| Path.cwd()             | os.getcwd()          | 获得当前工作目录                                             |
| Path.exists()          | os.path.exists()     | 判断是否存在文件或目录 name                                  |
| Path.home()            | os.path.expanduser() | 返回电脑的用户目录                                           |
| Path.is_dir()          | os.path.isdir()      | 检验给出的路径是一个目录                                     |
| Path.is_file()         | os.path.isfile()     | 检验给出的路径是一个文件                                     |
| Path.is_symlink()      | os.path.islink()     | 检验给出的路径是一个符号链接                                 |
| Path.stat()            | os.stat()            | 获得文件属性                                                 |
| PurePath.is_absolute() | os.path.isabs()      | 判断是否为绝对路径                                           |
| PurePath.joinpath()    | os.path.join()       | 连接目录与文件名或目录                                       |
| PurePath.name          | os.path.basename()   | 返回文件名                                                   |
| PurePath.parent        | os.path.dirname()    | 返回文件路径                                                 |
| Path.samefile()        | os.path.samefile()   | 判断两个路径是否相同                                         |
| PurePath.suffix        | os.path.splitext()   | 分离文件名和扩展名                                           |



使用 Path 和 PurePath 来生成路径

```python
from pathlib import *

path = PurePath('A', 'B', 'C')
print(path)

path = Path('A', 'B', 'C')
print(path)
```

它们都会将传入的字符串合并为路径。



通过 Path 的成员属性可以直接获得路径信息

```python
from pathlib import *

path = r'D:\Python\Scripts\pip.exe'
path = Path(path)

print(path.name)    # 返回文件名 + 后缀
print(path.stem)    # 返回文件名
print(path.suffix)  # 返回后缀
print(path.root)    # 返回根目录
print(path.parts)   # 返回目录的每一部分
print(path.parent)  # 返回父目录
```



还可以进行路径判断

```python
from pathlib import *

path = r'D:\Python\Scripts'
path = Path(path)

if path.exists():
    if path.is_file():
        print('是文件！')
    elif path.is_dir():
        print('是目录！')
else:
    print('路径不存在！')
```



进行路径建立、删除

```python
from pathlib import *

path = 'newdir'
path = Path(path)

if path.exists():
    Path.rmdir(path)
    print('删除文件夹')
else:
    Path.mkdir(path)
    print('创建文件夹')
```

还可以删除文件

```python
Path.unlink(path)
```



可以查找指定路径下的文件，使用 iterdir 方法可以获得目录下的所有子目录和文件的生成器。下面代码对结果进行过滤，获得文件列表

```python
from pathlib import *

path = r'D:\Python\Scripts'
path = Path(path)

# 获得指定路径下的文件的列表
files = [str(f) for f in path.iterdir() if Path(f).is_file()]
print(files)
```



甚至可以通过正则表达式匹配，使用 glob 指定匹配模式，获得目录下的指定类型的文件

```python
from pathlib import *

path = r'D:\Python\tcl'
path = Path(path)

# 获得指定路径下的 .lib 文件的列表
files = [str(f) for f in path.glob(r'*.lib') if Path(f).is_file()]
print(files)
```

可以递归匹配子目录

```python
from pathlib import *

path = r'D:\Python\tcl'
path = Path(path)

# 获得指定路径及其子目录下的 .lib 文件的列表
files = [str(f) for f in path.rglob(r'*.lib') if Path(f).is_file()]
print(files)
```



还可以读写文件，与基本文件操作类似

* `Path.open(mode='r')` 以 "r" 格式打开 Path 路径下的文件，若文件不存在即创建后打开。
* `Path.read_bytes()` 打开 Path 路径下的文件，以字节流格式读取文件内容，等同 open 操作文件的 "rb" 格式。
* `Path.read_text()` 打开 Path 路径下的文件，以 str 格式读取文件内容，等同 open 操作文件的 "r" 格式。
* `Path.write_bytes()` 对 Path 路径下的文件进行写操作，等同 open 操作文件的 "wb" 格式。
* `Path.write_text()` 对 Path 路径下的文件进行写操作，等同 open 操作文件的 "w" 格式。

其中 open 方法的模式与一般文件读取函数 open 的模式通用。使用它打开指定路径下的文件

```python
from pathlib import *

path = 'test.txt'
path = Path(path)

# 创建并打开文件
with path.open('w') as f:
	f.write('Hello World')

f = open(path, 'r')  # 读取内容
print(f"读取的文件内容为：{f.read()}")
f.close()
```



### pickle

pickle 模块能够将大多数数据类型与二进制数据进行转换操作。例如将列表数据保存为二进制文件

```python
import pickle

write_list = [123, 3.14, '你好', ['a', 'b']]
list_file = open('list.pkl', 'wb')
pickle.dump(write_list, list_file)
list_file.close()
```

反之可以从二进制文件中读取

```python
list_file = open('list.pkl', 'rb')
read_list = pickle.load(list_file)
list_file.close()

print(read_list)
```

与一般的文件读写相比，使用这种方法**不需要考虑数据类型的问题**。前者读写文本文件得到字符串，读写二进制文件得到视频和图片；后者则可以应对任何数据类型。



### zipfile

zipfile 模块用于处理压缩文件。

| 函数           | 作用                     | 函数             | 作用                          |
| ------------ | ---------------------- | -------------- | --------------------------- |
| `ZipFile()`  | 创造一个zipfile文件对象，同时打开文件 | `is_zipfile()` | 测试文件是否为有效的 zip 文件，要正确指明文件路径 |
| `read()`     | 读取压缩文件内容               | `namelist()`   | 返回 zip 文件中的所有文件夹名、文件名的列表    |
| `printdir()` | 将 zip 文件的目录结构输出到控制台    | `write()`      | 将指定文件添加到 zip 文件中            |
| `close()`    | 完成文件操作                 |                |                             |



结合 os 和 zipfile 模块可以实现解压文件

```python
import os
import zipfile

# 这里使用原始字符串，不需要转义
filedir = r'F:\python'
filename = r'file.zip'

# 切换到文件目录
os.chdir(filedir) 

# 获得 .zip 后缀，然后创建新文件夹
new_filedir = filename[:-4]
os.mkdir(new_filedir)

# 以 'r' 模式打开 zip 文件，然后得到一个列表，列表第一个元素是文件夹名，应当用 [1:] 去掉
zipfp = zipfile.ZipFile(filename,'r')
zipfile_list = zipfp.namelist()[1:]

# 切换当前路径为新建文件夹，便于把加压后文件放入
os.chdir(os.path.join(filrdir,new_filedir))

# 遍历压缩文件里面的所有文件
for name in zipfile_list:
    # 把路径分割成 dirname 和 basename，返回一个元组
    zipfile_list = os.path.split(name)
    
    # 解压后的文件以原文件名命名
    file = open(zipfile_list[1],'wb')
    
    # 读取文件数据，写入新文件中
    data = zipfp.read(name)
    file.write(data)
    
    # 关闭文件保存
    file.close()

```



### time

时间模块提供时间格式以及转换为字符串的函数。利用该我们，我们可以写一个简单计时器

```python
import time as t

class MyTimer():
    def __init__(self):
        self.unit = ['年','月','天','小时','分钟','秒']
        self.prompt = '未开始计时!'
        self.lasted = []
        self.begin = 0
        self.end = 0

    def __str__(self):
        return self.prompt
    
    # 将运行时间相加，然后返回信息
    def __add__(self, other):
        x = '总共运行了'
        result = []
        for i in range(6):
            result.append(self.lasted[i] + other.lasted[i])
            if result[i]:
                x += (str(result[i]) + self.unit[i])
        return x
              
    # 使自我描述函数与字符串函数一致
    __repr__ = __str__
    
    # 开始计时
    def start(self):
        self.prompt = '请先调用 stop() 停止计时！'
        self.begin = t.localtime()
        print('计时开始...')

    # 停止计时
    def stop(self):
        if not self.begin:
            print('请先调用 start() 开始计时！')
        else:
            # 计算运行时间
            self.end = t.localtime()
            self._calc()
            print('计时结束！')

    # 内部方法，计算运行时间
    def _calc(self):
        self.lasted = []
        self.prompt = '总共运行了'
        
        for i in range(6):
            self.lasted.append(self.end[i] - self.begin[i])
            if self.lasted[i]:
                self.prompt += (str(self.lasted[i]) + self.unit[i])
                
        #为下一轮计时初始化变量
        self.begin = 0
        self.end = 0


```



### warning

警告模块，用于显示警告信息

```python
import warnings

def month_warning(m):
    if not 1<= m <= 12:
        msg = "month (%d) is not between 1 and 12" % m
        warnings.warn(msg, RuntimeWarning)
		
# 将会警告 RuntimeWarning
month_warning(13)
```



### functools

这是一个提供函数方法的库，用于对函数进行操作。例如

```python
import functools

# reduce 将可迭代序列中的参数传入，然后把返回值作为新的参数传入
# 等价于 x = (((1 * 2) * 3) * 4)
x = functools.reduce(lambda x, y : x * y, range(1, 5))
print(x)


def func(a, b):
    print(a + b)

# partial 为指定函数提供显式的参数，然后将包装后的函数返回
# funcA = func(a, b=5)
funcA = functools.partial(func, b=5)
funcA(3)
```



它还提供了用来装饰装饰器指令。先前使用装饰器的函数，其 `__name__` 属性值不是原先的函数名，为了解决这个问题，引入

```python
import time
import functools

def time_master(func):
    # 使用这个装饰器，修改函数名为原先的函数名
    @functools.wraps(func)
    def call_func():
        print('开始运行程序')
        start = time.time()
        func()
        stop = time.time()
        print('结束程序运行')
        print(f'运行耗时{(stop-start):.2f}秒')
        
    return call_func

@time_master
def myfunc():
    time.sleep(2)
    print('Hi')

# 此时函数名为 myfunc
print(myfunc.__name__)
```



### turtle

turtle 提供了快速绘图的方法，通过直接调用相应的方法实现绘图操作。

```python
import turtle
```

使用 turtle 的绘图与一般的绘图 API 不同，它使用一根画笔在屏幕上移动来绘图，因此会显示绘图过程。



通过 `turtle.` 调用方法来控制画笔移动和属性

| 方法                  | 作用                  | 方法                     | 作用               |
| ------------------- | ------------------- | ---------------------- | ---------------- |
| speed               | 控制画笔速度              | pensize                | 控制画笔宽度           |
| pencolor            | 调整画笔颜色              | colormode              | 设置颜色模式           |
| penup()/pu()        | 抬起画笔                | pendown()/pd()         | 放下画笔             |
| setheading()/seth() | 将行进方向变为以 x 正向为基准的角度 | right                  | 设置右转角度           |
| left                | 设置左转角度              | fillcolor              | 设置填充颜色           |
| color               | 设置画笔颜色和填充颜色         | tracer                 | 是否展示过程           |
| begin_fill          | 准备填充颜色（在绘制封闭图形前）    | end_fill               | 结束填充颜色（在绘制封闭图形后） |
| hideturtle          | 隐藏画笔                | showturtle             | 显示画笔             |
| forward()/fd()      | 向前移动                | backward()/back()/bk() | 向后移动             |
| goto(x,y)           | 移动到 (x,y)           | home                   | 返回原点             |
| setx                | 设置 x 坐标             | sety                   | 设置 y 坐标          |
| circle              | 画圆                  | dot                    | 绘制实心圆点           |
| title               | 设置窗口标题              | clear                  | 清除窗口             |
| reset               | 重置窗口                | undo                   | 撤销上一步            |
| isvisible           | 返回当前画笔是否可见          | write                  | 将文本输出到当前位置       |

其中较为复杂的方法有

```python
circle(radius[,extent[,step]])	# 以 radius 为半径画圆弧，extent 为圆弧度数，step 为内接多边形边数
dot(size=None,*color) 			# 绘制实心圆点
```

以及绘制文本的方法

```python
write(s[,move=False/True[,align='left'/'right'/'center'[,font=(fontname,fontsize,fonttype)]]])
```



例如绘制多边形并填充，绘制文本

```python
import turtle
turtle.title('画图吧')

# 移动画笔时，让画笔抬起，到达绘图位置后放下
turtle.pu()
turtle.goto(-200,200)
turtle.pd()

# 说明要填充接下来绘制的图像
turtle.begin_fill()
turtle.color('red','blue')  # 设置画笔颜色和填充颜色
turtle.forward(100)         # 向前移动
turtle.right(120)           # 右转 120 度
turtle.forward(100)
turtle.right(120)
turtle.forward(100)
turtle.right(120)
turtle.end_fill()

# 移动画笔
turtle.pu()
turtle.goto(100,100)
turtle.pd()

# 画一个圆并填充
turtle.begin_fill()
turtle.color('blue','red')
turtle.circle(50, steps=5)
turtle.end_fill()
turtle.circle(50)

# 移动画笔
turtle.pu()
turtle.goto(-125,50)

# 绘制文字不需要放下画笔
turtle.write('你觉得好看吗？',align='center',font=('微软雅黑',18,'bold'))

# 隐藏画笔
turtle.hideturtle()
```



### json

使用 json 模块实现文件的序列化和反序列化。例如将对象转换为 json 字符串（序列化）

```python
import json

names = ['Li', 'Liu']
names = json.dumps(names)

fp = open('test.txt', 'w')
fp.write(names)
fp.close()
```

也可以用 dump 直接写入

```python
import json

names = ['Li', 'Liu']

fp = open('test.txt', 'w')
json.dump(names, fp)

fp.close()
```



反之，可以将 json 字符串转换为对象（反序列化）

```python
import json

fp = open('test.txt', 'r')
content = fp.read()

result = json.loads(content)
print(type(result))	# 类型已经是列表

fp.close()
```

也可以用 load 直接转换

```python
import json

fp = open('test.txt', 'r')

result = json.load(fp)
print(type(result))	# 类型已经是列表

fp.close()
```



### sqlite3

sqlite3 是一个 python 自带的用于与 sqlite 数据库交互的库，使用方法如下

```python
import sqlite3

conn = sqlite3.connect('book.db')
cur = conn.cursor()

# 插入数据(1)
cur.execute('INSERT INTO student VALUES(10008,"貂蝉",86);')
conn.commit()

# 插入数据(2)
with conn:
    cur.execute('INSERT INTO student VALUES(10009,"蔡文姬",99);')

# 查询数据
with conn:
    cur.execute('SELECT * FROM student')
    print(cur.fetchall())

# 删除数据
with conn:
    cur.execute('DELETE FROM student WHERE id = 10008 or id = 10009;')

conn.close()
```



### [fitz](https://pymupdf.readthedocs.io/en/latest/)

fitz 库用来对 pdf 文件进行操作。首先需要安装

```shell
(.venv) $ pip install pymupdf
```



可以创建 fitz 对象，得到一个空白的对象，需要向其中插入数据后才能保存

```python
doc = fitz.open()
```

例如我们将一张图片转换为 pdf，然后插入到 doc 中

```python
import fitz

# 创建 fitz 对象
doc = fitz.open()

# 读取图片，转换为 pdf 获得二进制数据，然后用它创建一个 pdf
pdfbytes = fitz.open('test.jpg').convert_to_pdf()
imgpdf = fitz.open('pdf', pdfbytes)
# 如果只有一张图片，可以直接保存 imgpdf

# 插入 pdf，然后保存
doc.insert_pdf(imgpdf)
doc.save('test.pdf')
doc.close()
```



反过来也可以将 pdf 中的页面转换为图片

```python
import fitz

# 创建 fitz 对象
doc = fitz.open()

pdf = fitz.open('test.pdf')
for i in range(len(pdf)):
    # 索引获得 pdf 中的第 i 页面
    page = pdf[i]
    pix = page.get_pixmap()
    pix.save(f"page{i}.png")

pdf.close()
```



也可以打开 pdf 文件，然后插入页面，实现 pdf 合并功能

```python
import fitz

# 创建 fitz 对象
doc = fitz.open()

pdf1 = fitz.open('test.pdf')
pdf2 = fitz.open('test2.pdf')

# 插入 pdf
doc.insert_pdf(pdf1)
doc.insert_pdf(pdf2)

pdf1.close()
pdf2.close()

doc.save('merge.pdf')
doc.close()
```



可以读取页面中的文本

```python
import fitz

# 创建 fitz 对象
doc = fitz.open()

pdf = fitz.open('test.pdf')
for i in range(len(pdf)):
    page = pdf[i]
    print(page.get_text())

pdf.close()
```



### [ISR](https://idealo.github.io/image-super-resolution/)

Image Super Resolution 是一个对图像进行超分辨率的深度学习模型库。在 Python 3.7.9 版本下可以通过 pip 直接安装

```shell
(.venv) $ pip install ISR
```

此时安装的代码版本实际上已经过时，需要对其中的 protobuf 库回退版本

```shell
(.venv) $ pip install protobuf==3.20.1
```

处理兼容性问题

```shell
(.venv) $ pip install h5py==2.10.0 --force-reinstall
```



现在可以使用

```python
import numpy as np
from PIL import Image
from ISR.models import RDN

# 使用 Pillow 读取并保存为 numpy 数组
img = Image.open('xiang.png')
lr_img = np.array(img)

# 读取预训练模型，进行超分辨率，然后转换为 Pillow
rdn = RDN(weights='noise-cancel')
sr_img = rdn.predict(lr_img)
dst = Image.fromarray(sr_img)

dst.save("output.png")
```

可以使用 3 种不同的 RDN 预训练模型

```python
['psnr-small', 'psnr-large', 'noise-cancel']
```

下面是对 360 x 360 分辨率图像（左图）分别使用上述 3 种模型处理后输出 720 x 720 分辨率的效果对比

![[xiang.png]]
![[output1.png]]
![[output2.png]]
![[output3.png]]




还可以使用 RRDN 预训练模型

```python
import numpy as np
from PIL import Image
from ISR.models import RRDN

img = Image.open('xiang.png')
lr_img = np.array(img)

rdn = RRDN(weights='gans')
sr_img = rdn.predict(lr_img)
dst = Image.fromarray(sr_img)

dst.save("output.png")
```

处理后输出 1440 x 1440 分辨率

![[output4.png|275]]

第一次使用模型时会下载模型，要挂梯子才行。



### tikzplotlib

这是一个将 matplotlib 的 2 维绘图结果转换为 tikz 代码的辅助库。通过 pip 安装

```shell
pip install tikzplotlib
```

首先用 matplotlib 正常绘图

```python
import matplotlib.pyplot as plt
import numpy as np

plt.style.use("ggplot")

t = np.arange(0.0, 2.0, 0.1)
s = np.sin(2 * np.pi * t)
s2 = np.cos(2 * np.pi * t)
plt.plot(t, s, "o-", lw=4.1)
plt.plot(t, s2, "o-", lw=4.1)
plt.xlabel("time (s)")
plt.ylabel("Voltage (mV)")
plt.title("Simple plot $\\frac{\\alpha}{2}$")
plt.grid(True)
```

然后调用 tikzplotlib 导出

```python
import tikzplotlib

tikzplotlib.save("test.tex")
```

可以在导出前清理图

```python
tikzplotlib.clean_figure()
```

该命令将删除轴限制之外的点，简化曲线并降低指定目标分辨率的点密度。



### ddddorc

这是一个验证码识别库。通过 pip 安装

```shell
pip install ddddocr
```



### gtest-parallel

这是一个将 Google-Test 中的每个测试并行化执行的脚本，使用方式如下

```shell
python gtest_parallel.py E:\Desktop\github\GME-starter-main\build\Debug\tests_acis.exe
```

通过 `--help` 选项获得更多信息。



如果需要只执行部分测试，使用

```shell
python gtest_parallel.py test.exe --gtest-filter=Foo.*:Bar.*
```

其中 `--gtest-filter` 的参数可以使用正则匹配测试名。



可以重复执行测试并指定更多线程

```c++
python gtest_parallel.py test.exe --repeat=1000 --workers=128
```



## 数据分析

### numpy

NumPy 提供了许多高级的数值编程工具，其重要特征是它的数组计算功能。首先导入

```python
import numpy as np
```

numpy 定义了各类数学函数和常量可供直接调用，我们主要介绍数组对象。



#### 数组运算

使用 numpy 定义数组，可以很容易对数组进行高效的修改、求和等操作

```python
import numpy as np

a = np.array([1, 2, 3, 4])
b = np.array([3, 4, 5, 6])

print(a + 1)
print(a + b)
```



甚至可以对数组元素进行判断

```python
b = a > 10	# array([False, False, False, False], dtype=bool)
```

将得到一个布尔数组，每个元素是判断 a 对应元素大于 10 的结果。



#### 数组类型

创建数组时可以指定类型

```python
a = np.array([1.5, -3], dtype='float')
```



可以对数组进行类型转换

```python
a = a.astype('float')			# 返回新数组，转换为 float 类型
np.asarray(a, dtype='bool')		# 原地修改类型为 bool
```

使用 astype 会申请新的内存。



#### 生成数组

生成特定数组的方法

| 方法                            | 作用                           |
| ----------------------------- | ---------------------------- |
| zeros(length, dtype='float')  | 全零数组                         |
| ones(length, dtype='float')   | 全一数组，可修改元素类型                 |
| fill(value)                   | 用指定值填充数组，value 将会转换与数组类型一致   |
| arange(start, end, step)      | 创建等差数列，不包含 end               |
| linspace(start, end, n)       | 创建 n 个数的序列，均匀划分 [start, end] |
| random.rand(n, m=1)           | 创建 0-1 上均匀分布的 n*m 个随机数       |
| random.randn(n, m=1)          | 创建 0-1 上正态分布的 n*m 个随机数       |
| random.randint(start, end, n) | 创建 [start, end] 上的 n 个随机整数   |



#### 数组属性

使用 `type` 查看数组类型。使用 dtype 获得其中的数据类型

```python
print(a.dtype)
```

使用 shape 获得一个元组，其中每个元素表示这一维的元素数量

```python
print(a.shape)
```

或者使用

```python
print(a.size)
```

可以查看数组维度

```python
print(a.ndim)
```



#### 索引与切片

numpy 数组的索引和切片方法与 python 自带的 list 基本相同。使用切片实现错位相减

```python
ob = np.array([1, 5, 6, 10])
db = ob[1:] - ob[:-1]
print(db)
```

用后一位减去前一位得到的差数组。



可以用列表（元组）进行索引，例如

```python
a = np.arange(0, 100, 10)
index = [1, 2, -3]
b = a[index]
print(b)
```

还可以用布尔数组来索引

```python
a = np.arange(0, 100, 10)
mask = np.array([0, 1, 1, 0, 0, 1, 0, 0, 1, 0], dtype='bool')
b = a[mask]
print(b)
```

这里创建 mask 元素为 True/False，将会获得对应 True 的元素的数组。



#### 引用复制

在切片过程中，使用的是引用机制，即新生成的数组仍然引用之前的数组中的内存，在列表中则没有这种情况。这样做的好处是避免了内存浪费，但是可能会导致修改一个值将改变所有引用的数组。这时候需要使用 copy 来复制

```python
b = a[2:4].copy()
```

复制会申请新的内存。**需要注意列表索引返回的是复制而不是引用**。



#### 多维数组

只需要嵌套两个列表就可以生成二维数组，先行后列

```python
a = np.array([[1, 2, 3, 4], [5, 6, 7, 8]])
print(a.shape)	# (2, 4) 2 行 4 列
print(a.ndim)	# 2 维
```



二维数组使用逗号分隔的数字索引和赋值

```python
a[1, 3] = 10
```

也可以用单个索引获得一整行或一整列

```python
b = a[1, :]	# 获得第 2 行，可以简写为 a[1]
d = a[:3]	# 获得前 3 行
c = a[:, 1]	# 获得第 2 列
```

其操作方法与 matlab 中几乎相同。



多维数组的切片方法则是分别对行列按照之前的切片格式

```python
d = a[0, 1:3]
```

使用列表索引

```python
a = np.array([[1, 2, 3, 4], [5, 6, 7, 8], [8, 9, 10, 11], [10, 11, 12, 13]])
d = a[[0,1,2], [1,2,3]]
print(d)
```

获得次对角线的元素数组。当然也可以用布尔数组索引。



#### where

使用 where 获得所有非零元素的索引。对于一维数组

```python
ind = np.where(a>10)
```

获得一个元组，包含大于 10 的元素的索引。这是因为 a>10 返回一个布尔数组，where 接收这个布尔数组返回索引元组。然后就可以取出所有满足条件的元素

```python
b = a[ind]	# 也可以用 b = a[a>10]
```



#### 数组形状

可以改变数组的形状

```python
a = np.array([0, 1, 2, 3, 4, 5])
a.shape = 2,3
print(a)
```

此时一维数组将变成 2 行 3 列数组。也可以使用 reshape，它不会修改原本的数组，而是返回新的数组

```python
b = a.reshape(2, 3)
```

注意行数乘列数必须等于数组的元素数。



如果想指定列数，行数根据列数计算，则使用

```python
a = np.array([[0, 1, 2, 3, 4, 5], [0, 1, 2, 3, 4, 5], [0, 1, 2, 3, 4, 5]])
b = a.reshape(-1, 3)
```

这表示重组为 3 列，根据计算会得到 6 行。注意列数必须**整除**数组的元素数。



使用 T 属性直接转置数组

```python
a.T
```

也可以用 transpose，它不会修改原本的数组

```python
b = a.transpose()
```



#### 数组连接

使用 concatenate 函数将一列数组按照给定的轴连接。语法为

```python
np.concatenate((a0, a1, ..., aN), axis=0)
```

默认按照第一维连接。例如

```python
a = np.array([[0, 1, 2], [3, 4, 5]])
b = np.array([[0, -1, -2], [-3, -4, -5]])
c = np.concatenate((a, b))
print(c)
```

将获得 `[[0, 1, 2], [3, 4, 5], [0, -1, -2], [-3, -4, -5]]`；如果按照第二维拼接

```python
c = np.concatenate((a, b), axis=1)
```

将得到 `[[0, 1, 2, 0, -1, -2], [3, 4, 5, -3, -4, -5]]` 。



如果两个数组形状相同，可以使用

```python
d1 = np.vstack((a, b))	# 纵向堆叠，对应 axis=0
d2 = np.hstack((a, b))	# 横向堆叠，对应 axis=1
d3 = np.dstack((a, b))	# 维数堆叠，会增加一个维数
```



#### 数组方法

更多常用的数组方法有

| 方法      | 作用                 | 方法    | 作用                             |
| --------- | -------------------- | ------- | -------------------------------- |
| sort      | 返回递增排序后的数组 | argsort | 返回递增排序后对应元素的索引数组 |
| sum       | 返回和               | max     | 返回最大值                       |
| min       | 返回最小值           | mean    | 返回均值                         |
| std       | 返回标准差           | cov     | 返回协方差矩阵                   |
| transpose | 转置                 | abs     | 返回绝对值数组                   |
| exp       | 返回 e 的幂数组      | median  | 返回中位数                       |
| cumsum    | 返回累积和数组       |         |                                  |



### pandas

用于解决数据分析任务，其中纳入了大量库和一些标准的数据模型，提供操作大型数据集的工具。

```python
import pandas as pd
```



pandas 有两种常用的基本数据结构

* Series 一维数组，与 numpy 中的一维数组类似，但是能够保存不同种数据类型；
* DataFrame 二维表格型数据结构；

它们的许多方法和 numpy 数组的方法类似。



#### Series

可以用一维列表初始化

```python
s = pd.Series([1, 3, 4, np.nan, 6, 8, 1])
```

其中 `np.nan` 表示空值。默认情况下通过连续数字索引，但是也可以修改索引

```python
s = pd.Series([1, 3, 4, np.nan, 6, 8, 1], index=['a', 'b', 'c', 'd', 'e', 'f', 'g'])
```

当需要进行索引，先通过 index 获得标签数组，也可以直接修改索引列表

```python
s.index = list('abcdefg')
```

可以设定索引的名字

```python
s.index.name = '索引'
```



切片操作与之前 numpy 数组相同，区别在于索引元素

```python
s = pd.Series([1, 3, 4, np.nan, 6, 8, 1], index=['a', 'b', 'c', 'd', 'e', 'f', 'g'])
d = s['a':'c']
```

根据索引类型设定切片范围。需要注意，**切片后的索引不一定是连续的**。



使用 values 返回 numpy 类型的数组

```python
type(s.values)
```



#### DataFrame

这是一个二维结构，首先构造一组时间序列，作为第一维的下标

```python
date = pd.date_range('20180101', periods=6)
```

将根据输入的日期格式，获得连续 6 天日期的下标数组。然后可以创建 DataFrame 结构

```python
df = pd.DataFrame(np.random.randn(6, 4), index=date, columns=list('ABCD'))
```

创建 6 x 4 随机数，用 date 作为索引，指定列标为 `ABCD` 。



可以使用字典传入数据，**字典的每个 key 对应一列，其 value 是各种可以转换为 Series 的对象**。例如

```python
df = pd.DataFrame(
    {'A': 1., 													# 只有一个元素，会自动填充
     'B': pd.Timestamp('20180101'),								# 时间格式，会自动填充
     # 创建长度为 4 的索引，得到 1 填充的长度为 4 的数组
     'C': pd.Series(1, index=list(range(4)), dtype='float'),
     'D': np.array([3]*4, dtype='int'),                         # numpy 中的数组
     'E': pd.Categorical(['test', 'train', 'test', 'train']),   # 类别数据
     'F': 'abc'                                                 # 字符串，算一个元素，会自动填充
     }
)
```

DataFrame 只要求每列数据类型相同。



#### excel

使用 pandas 读取 excel 文件

```python
df = pd.read_excel('score.xlsx')
```

数据保存在 DataFrame 中。也可以进行保存

```python
dp.to_excel('data.xlsx')
```



#### 访问数据

通过属性访问数据

```python
df.iloc[0:3]	# 获得 0 行到 2 行的数据
```

还可以通过标签选择数据

```python
df.loc[1, 'name']								# 获得 1 行 name 列数据
df2 = df.loc[[1, 3, 5], ['name', 'type']]		# 获得 1,3,5 行 name,type 列的数据表
```



要查看 DataFrame 的数据。可以使用

```python
df.dtypes	# 查看各列数据类型
df.index	# 查看下标
df.columns	# 查看列标
df.values	# 查看数据值
df.head()	# 查看前 5 行数据（默认 5 行）
df.tail(3)	# 查看后 3 行数据
```



#### 行列操作

由于 append 方法被弃用，**如果要添加新的行，需要创建一个新的表**，然后用 concat 方法将两个表连接

```python
df2 = pd.concat([df, df], ignore_index=True)
```

其中 ignore_index 设置是否重置行标。



使用 drop 方法删除指定的行/列，例如

```python
df = df.drop([0, 1])			# 删除 0,1 行
df = df.drop('name', axis=1)	# 删除 name 列
```



要获得指定范围的数据，使用列名称索引列，用行标索引行

```python
df['name'][:5]			# 将获得 name 列的前 5 行
df[['name', 'type']]	# 取出 name, type 列
```

要创建一列元素，使用

```python
df['new_col'] = range(1, 10)	# 创建 new_col 列，赋值为一个列表
```



#### 条件选择

通过直接的条件判断可以获得索引数组，然后使用它获得满足条件的数据

```python
ind = df['name']=='Li'	# 获得 name 是 Li 的列索引列表
a = df[ind][:5]			# 获得对应列的前 5 行数据
```

通过**布尔值的位运算**可以增加条件

```python
ind = (df['name']=='Li') & (df['type']=='file')		# 获得 name 是 Li 且 type 是 file 的列索引列表
ind = (df['name']=='Li') | (df['type']=='file')		# 获得 name 是 Li 或 type 是 file 的列索引列表
```



#### 缺失值

首先要判断是否有缺失值

```python
a = df.isnull()		# 返回整个表的缺失情况布尔值的表
```

一般对某一列或行进行判断。找到缺失情况后，可以进行填充。例如当某一列是数字，可以用均值填充缺失值

```python
df['score'].fillna(np.mean(df['score']), inplace=True)
```

其中 inplace 设置是否原地填充，如果为 False，将会创建新的表返回。



也可以直接删除空行或列

```python
df.dropna(how='all', inplace=True, axis=0)
```

其中 hwo 默认删除全为空的行或列，inplace 是否原地填充，axis 选择行或列。



#### 格式转换

首先要查看一列的数据格式

```python
df['name'].dtype
```

然后可以使用 astype 转换格式

```python
df['name'] = df['name'].astype('str')
```



#### 统计分析

可以获得对数据的基本分析结果，调用

```python
df.describe()
```

将得到数据的计数、均值、标准差等信息的表格。



可以进一步获得具体的信息

| 方法   | 作用       | 方法         | 作用           |
| ------ | ---------- | ------------ | -------------- |
| sum    | 返回和     | max          | 返回最大值     |
| min    | 返回最小值 | mean         | 返回均值       |
| std    | 返回标准差 | corr         | 返回协方差矩阵 |
| median | 返回中位数 | var          | 返回方差       |
| unique | 获得唯一值 | value_counts | 数值计数       |



可以根据给定的列数据进行排序

```python
df.sort_values(by='name', ascending=False)
```

其中 by 设置排序数据的列，ascending 设置升序/降序，默认 True 为升序。可以根据多列数据，依次排序

```python
df.sort_values(by=['name', 'type'])
```

也可以根据索引排序

```python
df = df.sort_index()
```



使用 replace 方法替换数据

```python
df['name'].replace('Li', 'Liu', inplace=True)
df['name'].replace(['Li', 'Hua'], ['Liu', 'Hu'], inplace=True)
```



#### 数据透视

在 Excel 中数据透视表可以用于以不同的角度分析数据。在 pandas 中使用

```python
pd.pivot_table(df, index=['name'])
```

以 name 为关键字，进行数据聚合，返回以 name 为核心的数据表。还可以指定需要统计的数据

```python
pd.pivot_table(df, index=['name', 'type'], value=['score'])
```

以 name, type 为关键字，统计 score 数据。



可以指定聚合函数

```python
pd.pivot_table(df, index=['name'], value=['score'], aggfunc=[np.sum, np.mean])
```

将会根据 name 对 score 进行分类，然后计算求和、均值等数据。可以对不同标签使用不同函数

```python
pd.pivot_table(df, index=['name'], value=['score', 'age'], aggfunc=['score':np.sum, 'age':np.mean])
```

这里对两种数据使用不同的聚合函数。



对于表中的缺失数据，可以提供 fill_value 参数填充

```python
pd.pivot_table(df, index=['name'], fill_value=0)
```

加入 margins 参数可以在最下方增加总和数据

```python
pd.pivot_table(df, index=['name'], fill_value=0, margins=True)
```



### matplotlib

matplotlib 是 Python 的 2D 图形包。其中 pyplot 模块封装了画图函数。

```python
import matplotlib.pyplot as plt
```



#### 绘制线条

调用 show 函数来显示图像

```python
plt.show()
```



最常用的绘图命令是 plot 命令

```python
plt.plot([1, 2, 3, 4], [5, 6, 7, 8])
```

可以用字符串指定颜色和线型，例如

```python
plt.plot([1, 2, 3, 4], [5, 6, 7, 8], 'ro')
```

显示红色圆点。



可选颜色例如

| 标识符 | 作用 | 标识符 | 作用 |
| ------ | ---- | ------ | ---- |
| w      | 白色 | r      | 红色 |
| g      | 绿色 | b      | 蓝色 |
| y      | 黄色 | k      | 黑色 |
| c      | 青色 | m      | 品红 |



可选线型例如

| 标识符 |  意义  | 标识符 | 意义 |
| :----: | :----: | :----: | :--: |
|   -    |  实线  |   +    | 加号 |
|   -.   | 点划线 |   x    | 叉号 |
|   --   |  虚线  |   *    | 星号 |
|   :    |  点线  |   .    |  点  |
|   o    |   圆   |   d    | 菱形 |



#### 图像保存

绘制完成后，调用

```matlab
plt.savefig('test.png')
```

保存当前窗口绘制的图像。



#### 传入数组

在 matplotlib 中通常传入 numpy 数组，即使传入列表，内部也会转换为 numpy 数组使用。例如

```python
t = np.arange(0, 5, 0.2)
plt.plot(t, t*t, 'r--')
```

使用 numpy 的方法创建数组并绘制

```python
x = np.linspace(-np.pi, np.pi, 100)
y = np.sin(x)
plt.plot(x, y)
```



#### 线条属性

可以通过参数设置线条属性

```python
plt.plot(x, y, linewidth=4, color='r')	# 设置线宽和颜色
```

plot 函数返回 Line2D 对象的列表，例如

```python
line, = plt.plot(x, y, linewidth=4, color='r')	# 获得线条对象
```

由于只绘制了一条线，返回列表只有一个元素，注意**左边有一个 `,` 表明这是一个接收元组，让 line 获得线条对象而不是列表**。



获得线条对象后，就可以直接修改属性

```python
line.set_antialiased(False)		# 取消反走样
```

更方便的是使用 setp 方法

```python
plt.setp(line, color='g')
```



#### 坐标标签

可以为坐标增加标签

```python
plt.xlabel('x', fontsize=10)
plt.ylabel('y', fontsize=10)
```

可以调整坐标刻度值的字体大小

```python
plt.tick_params(labelsize=14)
```

可以修改坐标刻度，并调整显示角度

```python
plt.xticks([1, 2, 3], rotation=90)	# 以 1 2 3 为刻度值，旋转 90 度
```



#### 坐标范围

利用 axis 函数对坐标重新设定。其调用格式为

```python
plt.axis([xmin, xmax, ymin, ymax])
```

也可以使用 xlim, ylim 函数修改坐标范围

```python
plt.xlim([0, 2])
plt.ylim([1, 3])
```



我们可以直接传入值来设置坐标

```python
plt.axis(False)		# 取消坐标轴
plt.axis('equal')	# 纵横坐标轴采用等长刻度
plt.axis('square')	# 产生正方形坐标系（默认为矩形）
```

给坐标加网格线可以用 grid 命令来控制

```python
plt.grid(True)	# 显示网格
```



#### 对数坐标

在实际应用中，经常用到对数坐标

```python
plt.semilogx(...)
plt.semilogy(...)
plt.loglog(...)
```

其中 semilogx 函数使用半对数坐标，loglog 函数使用全对数坐标，用法与 plot 相同。



#### 图形窗口

通过 figure 来创建窗口

```python
f = plt.figure()				# 创建新窗口
f = plt.figure(n)				# 创建编号为 n 的窗口
f = plt.figure(figsize=(10, 6))	# 创建窗口，设置尺寸
```

调用后会切换到新窗口，然后可以调用

```python
plt.subplot(rows, cols, i)
```

将**当前窗口**分割为 rows 行 cols 列，并且将其中第 i 个窗口设为当前窗口，这里 i 从  1 开始。



更好的是**通过 figure 对象获得 axes 对象**，用来绘制

```python
f = plt.figure()
axes = f.add_subplot(2, 2, 3)	# 在 2x2 网格中添加第 3 个子图
axes.plot(...)					# 可以使用 axes 在 f 上绘图
```

这种方法可以避免不能确定 plt 绘制的是哪一个窗口的问题。



#### 图例标注

使用 title 方法为整个图增加标题

```python
plt.title('Figure', fontsize=20)
```



使用 legend 按照顺序为每条绘制的线增加图例

```python
x = np.linspace(-np.pi, np.pi, 100)
y = np.sin(x)
plt.plot(x, y, linewidth=4, color='r')
plt.plot(x, y/2, linewidth=4, color='r')

plt.legend(['One', 'Two'], fontsize=10)
```



#### 文本标注

使用 text 函数为图像添加文字说明，例如在 (a,b) 位置绘制文本

```python
plt.text(a, b, ('%.2f' % b), ha='center', va='bottom', fontsize=10)
```

其中 ha, va 分别表示水平对齐和垂直对齐。



如果需要显示中文，应该修改字体

```python
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

plt.xlabel('x 轴')
plt.ylabel('y 轴')
```

同样修改 rc 参数可以支持衬线体的 LaTex 显示

```python
plt.rc('text', usetex=True)
plt.rc('font', family='serif')

# 使用原始字符串，不用 \ 转义
plt.text(1, 1, r'$\alpha^2+\beta^2=5^2$')
```



#### 创建注释

使用 annotation 函数创建注释。例如

```python
plt.annotate('Hello',
             xy=(1, 1),                 # 箭头指向的位置
             xytext=(1.1, 1.1),         # 注释文本的位置
             arrowprops=dict(
                 facecolor='black',     # 表面颜色
                 edgecolor='red',       # 边框颜色
                 headwidth=10           # 箭头底部宽度（像素）
             )
             )
```

还可以在 arrowprops 中添加 arrowstyle 参数，注意调整这个参数时，headwidth 参数可能无效，添加此参数会报错。



#### 柱状图

使用 bar 绘制柱状图

```python
x = np.linspace(0, np.pi, 10)
y = np.sin(x) + 0.1

plt.figure(figsize=(10, 6))				# 设置尺寸
plt.bar(x, y, color='g', width=0.1)		# 绘制柱状图，设置颜色和宽度

# 遍历 x,y 的序列包，获得柱状图的顶部位置
for a,b in zip(x, y):
    plt.text(a, b, ('%.2f' % b), ha='center', va='bottom', fontsize=10)

plt.show()
```



#### 饼图

使用 pie 绘制饼图

```python
x = np.linspace(0, np.pi, 5)
y = np.sin(x) + 0.1

plt.pie(y,									# 数据列，自动归一化
        labels=['a', 'b', 'c', 'd', 'e'],	# 每块的标签
        colors='bygrm',						# 每块颜色序列
        autopct='%1.1f%%',					# 百分比显示格式，%% 表示百分号
        startangle=45,						# 起始角度 45
        textprops=dict(color='w')			# 字体颜色为白色
        )
plt.legend()

plt.legend()
```



#### 直方图

使用 hist 绘制直方图

```python
x = np.linspace(0, np.pi, 1000)
y = np.sin(x)

plt.hist(y,						# 数据列
         bins=10,				# 划分区间数
         facecolor='red',		# 柱面颜色
         edgecolor='white',		# 边框颜色
         alpha=0.8				# 透明度
         )
```

直方图显示的是数据在区间段的分布频率。



## 图像处理

PIL 支持图像储存、显示和处理，能够处理几乎所有的图片格式，可以完成对图像的缩放、裁剪、叠加以及向图像添加线条和文字操作。

PIL 主要可以满足图像归档和图像处理两方面的功能需求

* 图像归档：对图像进行批处理、生成图像预览、转换图像格式等
* 图像处理：包括图像基本处理、像素处理、颜色处理

 

需要安装 Pillow 库

```shell
pip3 install Pillow
```



### Image

用 Image 模块导入图像文件

```python
from PIL import Image
pil_im = Image.open('empire.jpg')
```



也可以自己创建 Image 对象

```python
newIm = Image.new('RGB', (640, 480), (255, 0, 0)) 
```

新建一个大小为 (640, 480) 的红色 RGB 图像。对象的属性包括

* mode 模式
* size 尺寸
* color 颜色
* format 格式



使用 save 保存图像

```python
img.save('img1.png', 'png')
```



#### convert

通过 convert 方法将图像转换模式。例如转换为灰度图像

```python
pil_im = Image.open('empire.jpg').convert('L')
```

共有 9 种模式 1(黑白), L(灰度), P, RGB, RGBA, CMYK, YCbCr, I, F 用于对应类型的转换。



#### show

可以调用 show 直接显示图像

```python
pil_im.show()
```



#### crop

用于裁剪和复制

```python
im = Image.open('1.jpg')
box = (100, 100, 400, 400)
region = im.crop(box)
```

获得 box 指定范围的图像。



#### paste

将图像粘贴到指定位置

```python
im.paste(region, box)
```

将 region 放回 im 的 box 区域。



#### transpose

对图片进行旋转、反转操作

```python
region = region.transpose(Image.ROTATE_180)
```

逆时针旋转 180 度。



#### resize

调整图片大小

```python
out = im.resize((128, 128))
```



#### rotate

逆时针旋转图片

```python
out = im.rotate(45)
```

逆时针旋转 45 度。



#### 像素操作

| 方法                    | 作用                                                 |
| ----------------------- | ---------------------------------------------------- |
| getpixel(x, y)          | 获取指定像素的颜色，如果图像为多通道，则返回一个元组 |
| putpixel((x, y), color) | 改变指定像素的颜色                                   |



例如我们可以将图像转换为灰度图像，然后根据像素点的灰度值，用字符替代，实现字符图像

```python
from PIL import Image

img = Image.open('turtle.png')
out = img.convert('L')

width, height = out.size

# 纵向伸缩图像，使得字符替换后图像比例正确
zoom = 1
vscale = 0.5
width = int(width * zoom)
height = int(height * zoom * vscale)

out = out.resize((width, height))

asciis = '@%#*+=-. '
texts = ''

# 根据灰度值选择替换的字符
for row in range(height):
    for col in range(width):
        gray = out.getpixel((col, row))
        texts += asciis[int(gray / 255 * 8)]
    
    texts += '\n'

with open('asciis.txt', 'w') as file:
    file.write(texts)
```



### ImageChops

ImageChops 模块包含一些算术图形操作，称为**通道操作**。这些操作可用于诸多目的，如图像特效、图像组合、算法绘图等。通道操作只用于位图像（如 L 模式和 RGB 模式）。每张图片都由一个或者多个数据通道构成，以 RGB 图像为例，每张图片都由三个数据通道构成，分别为 R、G 和 B 通道。而对于灰度图像，则只有一个通道。大多数通道操作有一个或者两个图像参数，返回一个新的图像。



例如可以复制图像，然后将两张图像逐像素做差取绝对值

```python
from PIL import Image
from PIL import ImageChops

im = Image.open('1.jpg')
im_dup = ImageChops.duplicate(im)			# 复制图像
im_diff = ImageChops.difference(im, im_dup)	# 逐像素差取绝对值
im_diff.show()
```



### ImageDraw

此模块进行图形处理，添加几何图形。例如

```python
from PIL import Image, ImageDraw

im = Image.open('1.jpg')

# 在图像上画线
draw = ImageDraw.draw(im)
draw.line((0, 0)+im.size, fill=128)
draw.line((0, im.size[1], im.size[0], 0), fill=128)

im.show()
```

 

### ImageEnhance

用于图像增强，分为 Color 类，Brightness 类，Contrast 类和Sharpness 类。可以对图像进行高级处理，例如

```python
enhancer = ImageEnhance.Brightness(im)	# 建立增强对象
im0 = enhancer.enhance(0.5)				# 提高亮度
im0.show()
```



## Excel 处理

我们主要介绍 openpyxl 和 xlwings 模块

* openpyxl 模块简单易用、功能广泛，具有单元格格式、图片、表格、公式、筛选、批注、文件保护等功能，缺点是对 VBA(Excel 的编程语言) 的支持不够好；
* xlwings 模块可以结合 VBA 实现对 Excel 编程，拥有丰富的接口，结合 pandas/numpy/matplotlib 等模块进行数据处理工作；



### openpyxl

使用 openpyxl 可以快速操作和创建 excel 文件，例如

```python
import openpyxl
import datetime

wb = openpyxl.Workbook()			# 创建工作簿
ws = wb.active						# 获得工作表 sheet
print(ws.title)

ws['A1'] = 520                      # 设置 A1 位置
ws.append([1, 2, 3])                # 设置 A2 B2 C2 位置
ws['A3'] = datetime.datetime.now()  # 设置 A3 位置

wb.save('demo.xlsx')
```

可以看到按照纵向顺序赋值，每次设置一行的内容。



使用 load_workbook 打开一个 excel 文件

```python
wb = openpyxl.load_workbook('demo.xlsx')
```



#### 工作表

获得所有 sheet 的列表，然后获得需要的 sheet

```python
sheets = wb.sheetnames	# sheet 名称的列表
ws = wb[sheets[0]]		# 通过名称获得 sheet
ws = wb.active			# 获得活动表对应的 sheet 对象
```

进一步获得 sheet 的信息

```python
ws.title		# 表名
ws.max_row		# 行数
ws.max_column	# 列数
ws.rows			# 每行单元格
ws.columns		# 每列单元格
```



可以创建、复制和删除 sheet

```python
wb.create_sheet(title, index)	# 指定标题，插入位置
ns = wb.copy_worksheet(ws)		# 复制 ws 工作表，自动创建
wb.remove(sheet)				# 删除 sheet
```



使用 sheet_properties 修改 sheet 属性，例如修改标签颜色

```python
ws.sheet_properties.tabColor = 'FF0000'
```



可以修改行高和列宽

```python
ws.row_dimensions[2].height = 100		# 第 2 行高 100
ws.column_dimensions['C'].width = 50	# 第 C 列宽 50
```



可以合并和拆分单元格

```python
ws.merge_cells('A1:C3')		# 将矩形区域合并
ws['A1'] = 'Hello'			# 将会输出到整个区域中
ws.unmerge_cells('A1:C3')	# 拆分合并的单元格
```

注意拆分的单元格必须完全等于合并的区域。



可以冻结和解冻单元格，让行列标题始终显示

```python
ws.freeze_panes = 'B8'	# 冻结
ws.freeze_panes = None	# 解冻
```



#### 单元格

得到 sheet 后，通过行列索引获得单元格

```python
ws['A1']		# A - 列标，1 - 行标
ws.cell(1, 1)
```

注意在 Z 列之后的列标是 AA，因此建议通过 cell 来获得单元格。



对于单元格对象，可以通过其属性获得信息

```python
c = ws.cell(1, 1)
c.value			# 获得值
c.row			# 行号
c.column		# 列号
c.coordinate	# 坐标
```



使用 offset 方法移动单元格位置

```python
d = c.offset(2, 1)	# 向下移动 2 行，向右移动 1 列
```



#### utils 模块

使用它处理行列索引。例如

```python
import openpyxl.utils as utils

wb = openpyxl.load_workbook('demo.xlsx')
ws = wb.active

col = utils.get_column_letter(10)			# 获得第 10 列标的字符串格式，即 J
ind = utils.column_index_from_string(col)	# 将字符串列表转换为数字列表，即 10
```



#### 遍历表格

可以遍历获得表的内容

```python
for row in range(1, ws.max_row + 1):
    cell = ws.cell(row, 1)
    print(cell.value)
```

输出表格第一列的所有内容，注意行列序号从 1 开始。也可以按照行遍历

```python
# 获得行的生成器
iter = ws.iter_rows()
for cell in iter:
    # cell 是一行单元格的元组
    for each in cell:
        print(each.value)
        
# 可以指定范围
iter = ws.iter_rows(min_row=2, min_col=2, max_row=5, max_col=5)
for cell in iter:
    # cell 是一行单元格的元组
    for each in cell:
        print(each.value)
```

使用 iter_cols 则可以按照列遍历。



使用切片可以获得指定区域范围的单元格。例如

```python
import openpyxl
from openpyxl.utils import get_column_letter

wb = openpyxl.load_workbook('demo.xlsx')
ws = wb.active

# 获得第一行的所有单元格对象
col = get_column_letter(ws.max_column)
cells_1 = ws['A1' : '%s1' % col]

for cells in cells_1:
    for cell in cells:
        print(cell.value)

# 获得第一列的所有单元格对象
row = ws.max_row
cells_A = ws['A1' : 'A%d' % row]

for cells in cells_A:
    for cell in cells:
        print(cell.value)

# 获得矩形区域的所有单元格对象（先行后列）
cells_A1C3 = ws['A1' : 'C3']

for cells in cells_A1C3:
    for cell in cells:
        print(cell.value)
```



#### styles 模块

使用该模块定制单元格风格。例如设置字体

```python
import openpyxl
from openpyxl.styles import Font

wb = openpyxl.load_workbook('demo.xlsx')
ws = wb.active

b2 = ws['A4']
b2.value = 'Hello'
bold_red_font = Font(bold=True, color='FF0000')
b2.font = bold_red_font

wb.save('demo.xlsx')
```

Font 对象可选参数有

* name 字体名
* size 字体大小
* bold 是否加粗
* italic 是否斜体
* vertAlign 可选 None / superscipt(上标) / subscript(下标)
* underline 可选 None / single(单下划线) / double(双下划线)
* strike 是否显示删除线
* color 颜色



另一个常用风格是文本对齐，例如

```python
import openpyxl
from openpyxl.styles import Alignment

wb = openpyxl.load_workbook('demo.xlsx')
ws = wb.active

ws.merge_cells('A1:C3')
ws['A1'].value = 'Hello World'

center_alignment = Alignment(horizontal='center', vertical='center')
ws['A1'].alignment = center_alignment

wb.save('demo.xlsx')
```

Alignment 对象可选参数有

* horizontal 可选 general(常规) / justify(两端对齐) / right(靠右对齐) / centerContinuous(跨列居中) / distributed(分散对齐) / fill(填充) / center(居中) / left(靠左)
* vertical 可选 center(垂直居中) / top(靠上) / bottom(靠下) / justify(两端对齐) / distributed(分散对齐)
* text_rotation 文本旋转角度
* wrap_text 是否自动换行
* shrink_to_fit 是否缩小字体填充
* indent 指定缩进



为了方便赋予格式，使用 NamedStyle 对象封装上面这些格式

```python
import openpyxl
from openpyxl.styles import Font
from openpyxl.styles import Alignment
from openpyxl.styles import NamedStyle

wb = openpyxl.load_workbook('demo.xlsx')
ws = wb.active

# 命名风格
highlight = NamedStyle(name='highlight')
highlight.font = Font(bold=True, color='FF0000')
highlight.alignment = Alignment(horizontal='center', vertical='center')

# 为工作簿增加风格
wb.add_named_style(highlight)

# 设置单元格风格
ws['A1'].style = highlight

wb.save('demo.xlsx')
```



#### 数字格式

在 Excel 中的数据有两种类型，分别是**文本**和**数字**，文本是左对齐的，数字是右对齐的。对于数据而言，文本格式是数字格式的一种。在 openpyxl 中提供了不同的文本格式，并通过变量存储。但是使用自定义格式能够帮助我们了解格式的写法，例如

| 格式           | 效果                                           | 格式          | 效果       |
| -------------- | :--------------------------------------------- | ------------- | ---------- |
| General        | 数字                                           | @             | 文本       |
| 0000           | 显示 4 位数字，不足 4 位则补 0                 | 0.00          | 两位小数   |
| #,##0.00       | '#' 是占位符，不会自动补充；`,` 是千分位分隔符 | 0%            | 增加百分号 |
| yyyy-mm-dd     | 4 位年，2 位月，2 位日                         | dd/mm/yy      | 日/月/年   |
| m/d/yy h:mm:ss | 月/日/年 时/分/秒                              | h:mm:ss AM/PM | 增加 AM/PM |



通过 number_format 设置格式

```python
import openpyxl
import datetime

wb = openpyxl.load_workbook('demo.xlsx')
ws = wb.active

ws['A1'] = 88.8
ws['A1'].number_format = '#,###.00元'

ws['A2'] = datetime.datetime.today()
ws['A2'].number_format = 'yyyy-mm-dd'

wb.save('demo.xlsx')
```



可以对一个单元格设置不同数据类型的格式，分号格式为 正值;负值;零;文本，可以设置 4 种格式，分别对应这些数据类型

```python
import openpyxl
import openpyxl.styles.colors as colors

wb = openpyxl.load_workbook('demo.xlsx')
ws = wb.active

# 创建颜色对象
RED = colors.RGB('FF0000')
GREEN = colors.RGB('00FF00')
BLUE = colors.RGB('0000FF')
YELLOW = colors.RGB('FFFF00')

# 正值;负值 格式，99 是红色
ws['A1'] = 99
ws['A1'].number_format = '[RED]+#,###.00;[GREEN]-#,###.00'

# 正值;负值 格式，99 是绿色
ws['A2'] = -99
ws['A2'].number_format = '[RED]+#,###.00;[GREEN]-#,###.00'

# 正值;负值;零;文本 格式，0 是蓝色
ws['A3'] = 0
ws['A3'].number_format = '[RED];[GREEN];[BLUE];[YELLOW]'

# 正值;负值;零;文本 格式，Hello 是黄色
ws['A4'] = 'Hello'
ws['A4'].number_format = '[RED];[GREEN];[BLUE];[YELLOW]'

wb.save('demo.xlsx')
```



在 `[]` 中可以附加条件，让单元格按照条件显示格式，例如

```python
import openpyxl
import openpyxl.styles.colors as colors

wb = openpyxl.load_workbook('demo.xlsx')
ws = wb.active

RED = colors.RGB('FF0000')
GREEN = colors.RGB('00FF00')
BLUE = colors.RGB('0000FF')
YELLOW = colors.RGB('FFFF00')

# 显示 女
ws['A1'] = 0
ws['A1'].number_format = '[=1]男;[=0]女'

# 显示 男
ws['A2'] = 1
ws['A2'].number_format = '[=1]男;[=0]女'

# 显示 红色不及格
ws['A3'] = 26
ws['A3'].number_format = '[<60][RED]不及格;[>=60][GREEN]及格'

# 显示 绿色及格
ws['A4'] = 68
ws['A4'].number_format = '[<60][RED]不及格;[>=60][GREEN]及格'

wb.save('demo.xlsx')
```



#### 函数公式

导入函数公式集合

```python
from openpyxl.utils import FORMULAE
```

这个 frozenset 中保存了大量函数类型。



例如我们可以将指定范围的单元格中的值相加，填充到一个单元格中

```python
import openpyxl
from openpyxl.utils import FORMULAE

wb = openpyxl.load_workbook('demo.xlsx')
ws = wb.active

for row in ws.iter_rows(min_col=2, min_row=2, max_col=5, max_row=5):
    ws[row[3].coordinate] = '=SUM(%s:%s)' % (row[0].coordinate, row[2].coordinate)

wb.save('demo.xlsx')
```

其中 row 存放了 2, 3, 4, 5 单元格，我们将 2, 3, 4 的值相加，填充到 5 中，因此 `row[3].coordinate` 获得 5 的位置，右边获得 2-4 的范围调用 SUM 函数求和。



再比如调用条件判断函数

```python
for row in ws.iter_rows(min_col=2, min_row=2, max_col=5, max_row=5):
    ws[row[3].coordinate] = '=IF(%s>250, "A", "B")' % (row[2].coordinate)
```

当 4 列值大于 250 时，5 列显示 A，否则显示 B 。注意这里外部**必须用单引号包围**，因为 IF 函数需要传入双引号包围的字符串。



使用 VLOOKUP 函数查找匹配数据

```python
ws['I2'] = '=VLOOKUP(H2, D2:D5, 1, FALSE)'
```

在 D2:D5 范围内寻找等于 H2 中值的行标，返回 **D2:D5 范围内第 1 列**对应行的值，FALSE 表示不使用近似匹配。



## [Tkinter](https://tkdocs.com/tutorial/index.html)

TKinter 是标准库，但是我们需要着重介绍。它可以实现简单的窗口交互程序，作为 GUI 编程的入门练习。



### 简单示例

创建一个显示文本标签的窗口

```python
import tkinter as tk

# 创建 tk 对象，设置标题
app = tk.Tk()
app.title("FishC Demo")

# 创建文本标签，并打包
thelabel = tk.Label(app, text="窗口程序")
thelabel.pack()

# 进入消息循环
app.mainloop()
```



为了使用方便，最好对窗口程序进行封装

```python
import tkinter as tk

# 封装 tkinter
class APP:
    def __init__(self,master):
        # 获得窗口框架，设置对齐方式和窗口尺寸
        frame = tk.Frame(master)
        frame.pack(side=tk.LEFT,padx=10,pady=10)

        # 创建按钮，传入响应函数 sayHello
        self.helloButton = tk.Button(
            frame,text="Say Hello",
            bg="black",
            fg="white",
            command=self.sayHello
        )

        # 将按钮打包
        self.helloButton.pack()

    def sayHello(self):
        print("Hello World")

# 用 tkinter 对象初始化
root = tk.Tk()
app = APP(root)

root.mainloop()
```



### 主题美化

安装第三方库 [ttkbootstrap](https://ttkbootstrap.readthedocs.io/en/version-0.5/) 实现对 tkinter 窗口风格的美化

```shell
(.venv) $ pip install ttkbootstrap
```



简单使用的示例如下

```python
from ttkbootstrap import Style
from tkinter import ttk

style = Style(theme='sandstone')
root = style.master

ttk.Button(root, text="Submit", style='success.TButton').pack(side='left', padx=5, pady=10)
ttk.Button(root, text="Submit", style='success.Outline.TButton').pack(side='left', padx=5, pady=10)

root.mainloop()
```

可选多种主题

```python
['vista', 'classic', 'cyborg', 'journal', 'darkly', 'flatly', 'clam', 'alt', 'solar', 'minty', 'litera', 'united', 'xpnative', 'pulse', 'cosmo', 'lumen', 'yeti', 'superhero', 'winnative', 'sandstone', 'default']
```



### 基本对象

#### Tk

Tk 是最基本的组件函数，创建一个窗口对象

```python
root = tk.Tk()
```

窗口对象的方法有

* quit() 是退出窗口的函数，可以通过 Button 的 command 参数来调用（这一函数会与 IDLE 的 quit 函数冲突）
* destroy() 是一个直接消灭窗口的函数，当无法用 root.quit 方法退出时可以使用这一函数
* register(func) 用于包装函数，用于 Entry 组件的 validatecommand 隐藏参数
* geometry() 用于设置窗口大小和位置
* title() 设置标题
* update() 刷新窗口



创建居中窗口

```python
import tkinter as tk

root = tk.Tk()

width = 300
height = 300

screen_width = root.winfo_screenwidth()
screen_height = root.winfo_screenheight()

x = int(screen_width / 2 - width / 2)
y = int(screen_height / 2 - height / 2)
size = '{}x{}+{}+{}'.format(width, height, x, y)

root.geometry(size)
root.mainloop()
```



#### StringVar

用于设置一个可变文本对象。例如

```python
var = StringVar()
var.set('Hello')
```

通过 set 可以修改文本，它用于 Tkinter 的其它控件。



#### IntVar

用于创建一个可变整型对象。例如

```python
v = IntVar()
v.set(1)
```

它用来存放控件的状态信息。



### 几何管理

Tkinter 控件有特定的几何状态管理方法，管理整个控件区域组织，以下是 Tkinter 公开的几何管理类：包、网格、位置

| 几何方法 | 描述 |
| -------- | ---- |
| pack()   | 包装 |
| grid()   | 网格 |
| place()  | 位置 |

注意不要在同一个父组件中使用 pack 和 grid，否则程序会一直计算两者的优先顺序卡死。



#### pack

用于将组件添加到窗口中，任何一个组件（包括框架）都必须经过 pack 的添加才能够显示。例如

```python
import tkinter as tk

app = tk.Tk()
thelabel = tk.Label(app, text="窗口程序")
thelabel.pack()

app.mainloop()
```

pack 方法有默认参数，因此可以无参数调用。通过 `help(thelabel.pack)` 可以看到其参数格式

```python
pack_configure(cnf={}, **kw)
```

所有的参数都是可选的，常用参数有

* side 可设为 LEFT, RIGHT, TOP, BOTTOM，决定这个组件显示的初始位置；
* fill 填充模式，可设为 X 横向填充，Y 纵向填充，BOTH 全部填充；
* expand 设置组件大小是否随窗口变化，可选 True/False；
* padx, pady 设置位置；
* ipadx, ipady 增量设置位置；
* anchor=NSEW 设置沿着指定方向延伸控件，可选 N,W,E,S,NE,NW,SE,SW；



例如我们设置标签填充 x 轴，组件大小随窗口变化

```python
from tkinter import *

root = Tk()

Label(root, text='red', bg='red', fg='white').pack(fill=X, expand=1)
Label(root, text='green', bg='blue', fg='white').pack(fill=X, expand=1)
Label(root, text='blue', bg='green', fg='black').pack(fill=X, expand=1)

mainloop()
```



#### forget

可以调用 forget() 来删除 pack

```python
import tkinter as tk

app = tk.Tk()
thelabel = tk.Label(app, text="窗口程序")
thelabel.pack()
thelabel.forget()

app.mainloop()
```



#### grid

通过网格的方式设置控件所在位置，基本形式为

```python
Label(root, ...).grid(row=0, column=0, sticky=W)
```

常用参数有

* row, column 参数设置行/列位置；
* sticky 参数设置所在方位，可设参数为 N,W,E,S,NE,NW,SE,SW；
* rowspan, columnspan 参数设置跨越的行/列数；
* padx, pady 参数设置位置

可以调用 grid_remove() 方法删除 grid 。



例如通过 grid 配置组件位置

```python
from tkinter import *

root = Tk()

Label(root, text='用户名').grid(row=0, column=0, sticky=W)
Label(root, text='密码').grid(row=1, column=0, sticky=W)

Entry(root).grid(row=0, column=1)
Entry(root, show='*').grid(row=1, column=1)

def show():
    print('提交成功！')

Button(text='提交', command=show).grid(row=2, column=0, columnspan=3)

mainloop()
```



#### grid_forget

使用 grid_forget 来隐藏通过 grid 配置的组件

```python
label = Label(root, text='用户名')
label.grid(row=0, column=0, sticky=W)
label.grid_forget()
```



#### place

指定组件的大小和位置

```python
Label(root, text='123').place(relx=1, rely=1, anchor=CENTER)
```

其中 relx, rely 参数设置相对于这一组件的位置，0~1 表示从左到右/从上到下的相对位置，relheight, relwidth 参数设置内容（如背景颜色等）相对于这一组件的高度和宽度。例如

```python
from tkinter import *

root = Tk()

Label(root, bg='red').place(relx=0.5, rely=0.5, relheight=0.75, relwidth=0.75, anchor=CENTER)
Label(root, bg='yellow').place(relx=0.5, rely=0.5, relheight=0.5, relwidth=0.5, anchor=CENTER)
Label(root, bg='green').place(relx=0.5, rely=0.5, relheight=0.25, relwidth=0.25, anchor=CENTER)

mainloop()
```



### 基本控件

Tkinter 提供各种控件，如按钮，标签和文本框。任何一个控件都可以通过键来修改参数。例如

```python
b = Button(root, text='A')
b['text'] = 'B'

l = Label(root, text='A')
l['text'] = 'B'
```



#### Label

显示标签，可以设置文本和图像

```python
tk.Label(master, option=value, ...)
```

其中 master: 是按钮的父容器（如窗口对象），options 是可选参数，例如

```python
from tkinter import *

root = Tk()

photo = PhotoImage(file="test.png")
theLabel = Label(
    root,
    text='显示在图片上面',
    justify=LEFT,
    image=photo,
    compound=CENTER,
    font=('微软雅黑', 20),
    fg='white'
)
theLabel.pack()

mainloop()
```

以给定的图片为背景，在其上绘制白色文字。



可以通过 textvariable 参数指定可修改的文本

```python
from tkinter import *

root = Tk()

# Tkinter 的字符串变量
var = StringVar()
var.set('Good morning!')

# 文本使用 var
theLabel = Label(root, textvariable=var)
theLabel.pack()

# 定义修改文本的函数
def putout():
    var.set('Ok, that\'s good.')

# 增加按钮，按下后调用 putout 修改文本
thebutton = Button(root, text='I\'m fine, thanks', command=putout)
thebutton.pack(side=BOTTOM, pady=50)

mainloop()
```

 

#### Frame

框架控件：在屏幕上显示一个矩形区域，多用来作为容器

```python
frame = Frame(root)
```

其它控件可以用它作为容器

```python
theLabel = Label(frame, text='Hi')
```



#### PhotoImage

用于导入图片，只能识别 gif 格式

```python
from tkinter import *

root = Tk()

textLabel = Label(root, text="这是一张图片！")
textLabel.pack(side=LEFT)

photo = PhotoImage(file="test.png")
imgLabel = Label(root, image=photo)
imgLabel.pack(side=RIGHT)

mainloop()
```

如果要显示其它格式的图片，应该使用 Canvas 配合 Pillow 库

```python
from PIL import Image, ImageTk

# 读取图片
src = Image.open(name)
            
# 注意一定要保存 srcPhoto，防止内存释放
srcPhoto = ImageTk.PhotoImage(self.src)
srcCanvas.create_image(0, 0, image=self.srcPhoto, anchor="nw")
srcCanvas.pack()
```



#### Button

按钮控件，可以点击。通过指定按钮函数，可以在点击时调用该函数。例如

```python
from tkinter import *

root = Tk()

def show():
    print('按了一下')

Button(root, text='按钮', width=10, command=show).pack()

root.mainloop()
```

按钮可用的参数包括

| 参数             | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| activebackground | 当鼠标放上去时，按钮的背景色                                 |
| activeforeground | 当鼠标放上去时，按钮的前景色                                 |
| bd               | 按钮边框的大小，默认为 2 个像素                              |
| bg               | 按钮的背景色                                                 |
| fg               | 按钮的前景色（按钮文本的颜色）                               |
| command          | 函数名按钮关联的函数，当按钮被点击时，执行该函数             |
| highlightcolor   | 要高亮的颜色                                                 |
| image            | 按钮上要显示的图片                                           |
| justify          | 显示多行文本的时候,设置不同行之间的对齐方式，可选项包括 LEFT, RIGHT, CENTER |
| relief           | 边框样式，设置控件 3D 效果，可选的有：FLAT、SUNKEN、RAISED、GROOVE、RIDGE 。默认为 FLAT |
| state            | 设置按钮组件状态，可选的有 NORMAL、ACTIVE、 DISABLED 。默认 NORMAL |
| underline        | 下划线。默认按钮上的文本都不带下划线。取值就是带下划线的字符串索引，为 0 时，第一个字符带下划线，为 1 时，前两个字符带下划线，以此类推 |
| wraplength       | 限制按钮每行显示的字符的数量                                 |

这些参数也可能在其它控件中可用。

 

#### Checkbutton

勾选选项组件，它的参数与前面的都差不多

```python
from tkinter import *

root = Tk()

# 要显示的文本
GIRLS = ['貂蝉','西施','王昭君','杨玉环']

# IntVar 列表
v = []

# 对每个文本，添加一个 IntVar 变量，然后将列表中最后一个变量（也就是新创建的变量）传入
for girl in GIRLS:
    v.append(IntVar())
    b = Checkbutton(root, text=girl, variable=v[-1])
    b.pack(anchor=W)

mainloop()
```



#### Radiobutton

单选组件，参数都差不多

```python
from tkinter import *

root = Tk()

LANGS = [
    ('Python', 1),
    ('Java', 2),
    ('C', 3),
    ('Perl', 4),
    ('Lua', 5)]

v = IntVar()
v.set(1)

for lang, num in LANGS:
    b = Radiobutton(root, text=lang, variable=v, value=num, indicatoron=False)
    b.pack(fill=X)

root.mainloop()
```

注意对每个 Radiobutton 我们传入的 IntVar 是同一个，因此它的值会反映第几个按钮被选择。另外，这里 indicatoron 参数设置按钮的形式，True 时为圆点，False 时为普通按钮。



以下为常用的方法：

| 方法       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| deselect() | 清除单选按钮的状态                                           |
| flash()    | 在激活状态颜色和正常颜色之间闪烁几次单选按钮，但保持它开始时的状态 |
| invoke()   | 可以调用此方法来获得与用户单击单选按钮以更改其状态时发生的操作相同的操作 |
| select()   | 设置单选按钮为选中                                           |



#### LabelFrame

文本框架组件，用于设计简单文本及其框架。使用它来修改上面的单选代码

```python
from tkinter import *

root = Tk()

group = LabelFrame(root, text="世界上最好的语言是？", padx=5, pady=5)
group.pack(padx=10, pady=10)

LANGS = [
    ('Python', 1),
    ('Java', 2),
    ('C', 3),
    ('Perl', 4),
    ('Lua', 5)]

v = IntVar()

for (lang, num) in LANGS:
    b = Radiobutton(group, text=lang, variable=v, value=num)
    b.pack(anchor=W)

root.mainloop()
```

 

#### Entry

输入框组件，用来让用户输入一行文本字符串，Tab 键可以将输入光标移动到下一个输入框，通过 get 方法获得输入内容

```python
from tkinter import *

root = Tk()

Label(root, text='作品：').grid(row=0, column=0)
Label(root, text='作者：').grid(row=1, column=0)

e1 = Entry(root)
e2 = Entry(root, show='*')
e1.grid(row=0, column=1, padx=10, pady=5)
e2.grid(row=1, column=1, padx=10, pady=5)

def show():
    print('作品：《{}》'.format(e1.get()))
    print('作者：{}'.format(e2.get()))

Button(root, text='获取信息', width=10, command=show)\
             .grid(row=3, column=0, sticky=W, padx=10, pady=5)
Button(root, text='退出', width=10, command=root.quit)\
            .grid(row=3, column=1, sticky=E, padx=10, pady=5)

mainloop()
```

其中 show 参数设置输入文本的显示内容。



可以禁用输入框

```python
e = Entry(buttonFrame, textvariable=self.srcHeight, width=10)
e.config(state='disabled')
e.pack(side=LEFT, pady=20)
```



使用 insert 方法插入字符串

```python
e.insert(index, 'hello')	# 指定位置插入
e.insert(END, 'hello')		# 末尾插入
e.insert(INSERT, 'hello')	# 光标位置插入
```

使用 delete 方法删除指定范围的字符串

```python
e.delete(index, END)	# 删除从 index 到末尾
e.delete(ACTIVE)  		# 删除当前选中的字符串
```

这两个方法在有关文本内容的控件中也经常使用。



Entry 控件支持验证输入合法性操作。通过 validate 参数设置

| 参数值   | 作用                 |
| -------- | -------------------- |
| focus    | 获得或失去焦点时验证 |
| focusin  | 获得焦点时验证       |
| focusout | 失去焦点时验证       |
| key      | 输入框被编辑时验证   |
| all      | 出现前四种情况时验证 |
| none     | 关闭验证             |

还要进一步设置 validatecommand 参数设置验证函数，此函数要返回 True 或者 False 作为验证结果。还可以设置 invalidatecommand 参数对应的函数，当 validatecommand 返回 False 就会调用。



例如设置两个输入框，当第一个输入框失去焦点时，检查内容是否为 Hello，如果不是就删除内容

```python
from tkinter import *

master = Tk()

v = StringVar()

def test1():
    if e1.get() == 'hello':
        print('正确！')
        return True
    else:
        print('错误！')
        e1.delete(0, END)
        return False

def test2():
    print('invalidcommand 调用……')
    return True

e1 = Entry(master, textvariable=v, validate='focusout', \
           validatecommand=test1, invalidcommand=test2)

e2 = Entry(master)

e1.pack(padx=10, pady=10)
e2.pack(padx=10, pady=10)

mainloop()
```



还可以实现更高级的检测方法，validatecommand 可以**传入带参数的函数**，使用

```python
validatecommand=(f[,options])
```

其中 f 是经过包装的函数，后面的字符提供额外的高级选项，它们对应参数值。例如

```python
def f(p, v, w):
    # p 对应 %P，为输入框的最新文本内容
    # v 对应 %v，为当前的 validate 选项的值
    # W 对应 %W，为组件的名字

validatecommand=(f, '%P', '%v', '%W')
```



所有的可选参数有

| 选项 | 参数值                                                       |
| :--: | :----------------------------------------------------------- |
|  %d  | 0 表示删除操作；1 表示插入操作；2 表示获得、失去焦点或 textvariable 变量的值被修改 |
|  %i  | 当用户尝试插入或修改操作时，表示插入或删除的位置；当获得、失去焦点或 textvariable 变量的值被修改而调用验证函数，那么它是 -1 |
|  %P  | 当输入框的值允许改变时，为输入框的最新文本内容               |
|  %s  | 该值为调用验证函数前输入框的文本内容                         |
|  %S  | 当插入或删除操作触发验证函数时，表示文本被插入和删除的内容   |
|  %v  | 表示该组件当前的 validate 选项的值                           |
|  %V  | 表示调用验证函数的原因，可以是 textvariable选项指定的变量值中的一个 |
|  %W  | 表示组件的名字                                               |



例如写一个加法器，使用高级检测方法使得输入只能是数字

```python
from tkinter import *

master = Tk()

# 使用框架包装
frame = Frame(master)
frame.pack(padx=10, pady=10)

v1 = StringVar()
v2 = StringVar()
v3 = StringVar()

# 检测函数，%P 作为参数，获得文本内容
def test(content):
    return content.isdigit()

# 使用 register 包装函数
testCMD = master.register(test)

# 输入框被编辑时验证，
e1 = Entry(frame, width=10, textvariable=v1, validate='key', \
           validatecommand=(testCMD, '%P')).grid(row=0, column=0)
e2 = Entry(frame, width=10, textvariable=v2, validate='key', \
           validatecommand=(testCMD, '%P')).grid(row=0, column=2)

# 结果输入框，设为只读
e3 = Entry(frame, width=10, textvariable=v3, state='readonly').grid(row=0, column=4)

# 显示符号
Label(frame, text='+').grid(row=0, column=1)
Label(frame, text='=').grid(row=0, column=3)

# 计算函数
def calc():
    result = int(v1.get()) + int(v2.get())
    v3.set(str(result))

# 按下按钮计算
Button(frame, text='计算结果', command=calc).grid(row=1, column=2, pady=5)

mainloop()
```



#### Listbox

列表框控件，用来显示一个字符串列表给用户

```python
theLB = Listbox(master, selectmode=BROWSE, ...)
```

其中 selectmode 提供四种选择模式：

* SINGLE 单选
* BROWSE 单选，但可以拖动鼠标或方向键改变选项
* MULTIPLE 多选
* EXTENDED 多选，但需要同时按住 Shift 键或 Ctrl 键或拖动鼠标实现

当要显示的内容较多，可以设置 height (行数) 参数。



例如显示一个可删除的序列

```python
from tkinter import *

master = Tk()

theLB = Listbox(master, selectmode=SINGLE, height=10)
theLB.pack()

# 在末尾插入元素
for item in ['鸡蛋','鸭蛋','鹅蛋','恐龙蛋']:
    theLB.insert(END, item)

# 这里使用 lambda 表达式传入匿名函数
theButton = Button(master, text='删除它', command=lambda x=theLB: x.delete(ACTIVE))
theButton.pack()

mainloop()
```



#### Scrollbar

滚动条控件，当内容超过可视化区域时使用，如列表框

```python
from tkinter import *

root = Tk()

sb = Scrollbar(root)
sb.pack(side=RIGHT, fill=Y)

# 为列表盒增加纵向滚动条
lb = Listbox(root, yscrollcommand=sb.set)

for i in range(1000):
    lb.insert(END, i)

theButton = Button(root, text='删除', command=lambda x=lb: x.delete(ACTIVE))
theButton.pack(side=BOTTOM, pady=5)

# 最后打包，注意打包顺序会影响控件位置
lb.pack(side=RIGHT, fill=X)

# 配置滚动条纵向滚动
sb.config(command=lb.yview)

mainloop()
```



#### Scale

范围控件，显示一个数值刻度，为输出限定范围的数字区，类似于滚动条控件。例如

```python
from tkinter import *

root = Tk()

# 刻度范围 0-45，刻度精度 10，滚动步长 5
s1 = Scale(root, from_=0, to=45, tickinterval=10, resolution=5)
s1.pack()

# 刻度范围 0-200，横向显示
s2 = Scale(root, from_=0, to=200, orient=HORIZONTAL, length=200)
s2.pack()

def show():
    # get 方法获得滑块位置
    print(s1.get())
    print(s2.get())

theB = Button(root, text='显示位置', command=show)
theB.pack()

mainloop()
```

注意其中 `from_` 参数，是因为 `from` 是关键字，所以才使用这个参数名。



#### Canvas

画布控件，用来绘制图形。

| 方法               | 作用     |
| ------------------ | -------- |
| create_line()      | 画直线   |
| create_rectangle() | 画矩形   |
| create_oval()      | 画椭圆   |
| create_polygon()   | 画多边形 |
| create_text()      | 画文本   |

新的画布会覆盖旧的，这时需要使用 lift() 和 lower() 方法调整优先级。



在 Canvas 方法中可以指定画布上的对象。例如

* Item handles  指定某个画布对象的整形数字（ID）
* Tags  附在画布对象上的标签
* ALL 指定所有画布对象
* CURRENT  表示鼠标指针下的画布对象



测试几个绘图函数

```python
from tkinter import *

root = Tk()

# 创建画布
w = Canvas(root, width=200, height=100)
w.pack()

# 绘制直线，移动位置
line1 = w.create_line(0, 50, 200, 50, fill='blue')
w.coords(line1, 0, 25, 200, 25)

# 绘制直线，然后删除
line2 = w.create_line(100, 0, 100, 100, fill='red', dash=(1, 1))
w.delete(line2)

# 绘制矩形
rect1 = w.create_rectangle(50, 25, 150, 75, fill='black')
w.itemconfig(rect1, fill='green')

# 绘制文本
w.create_text(100, 50, text='FishC')

# 注意这里 ALL 表示画布的所有对象，点击按钮删除所有图像
Button(root, text='删除全部', command=(lambda x=ALL: w.delete(x))).pack()

mainloop()
```



Canvas 没有绘制点的函数，但是可以用绘制椭圆来替代。因此我们可以创建一个自由画布

```python
from tkinter import *

root = Tk()
root.title('画布')

w = Canvas(root, width=500, height=400)
w.pack()

def paint(event):
    x1, y1 = (event.x - 1), (event.y - 1)
    x2, y2 = (event.x + 1), (event.y + 1)
    w.create_oval(x1, y1, x2, y2, fill='red')

def clear():
    w.delete(ALL)

w.bind('<B1-Motion>', paint)

Label(root, text='画布', font='宋体').pack(side=TOP, pady=30)
Label(root, text='现在开始你的绘画历程吧！', font='微软雅黑').pack(side=BOTTOM,pady=5)
Button(root, text='清除', command=clear, width=30).pack(side=BOTTOM,pady=10)

mainloop()
```



#### Menu

菜单组件可以显示在窗口上，也可以设为右键弹出。Menu 组件不需要 pack()，通过

```python
m = Menu(root)
root.config(menu=m)
```

将菜单设置到窗口上，如果不这样设置该菜单就不会显示在窗口上（可以利用这个制作右击呼出的窗口）。



可以将一个菜单作为另一个菜单的下拉菜单

```python
m1 = Menu(m, tearoff=False)
```

其中 tearoff 参数设置菜单是否可以从窗口分离，默认为 True 。



常用的方法有

| 方法              | 作用                             |
| ----------------- | -------------------------------- |
| add_command()     | 设置一般的点击菜单项，可绑定函数 |
| add_separator()   | 添加分割线                       |
| add_cascade()     | 设置菜单归属                     |
| add_checkbutton() | 设置单选菜单                     |
| add_radiobutton() | 设置多选菜单                     |
| post()            | 该方法被调用就会呼出菜单         |



我们构建一个常见的菜单样式

```python
from tkinter import *

root = Tk()

menubar = Menu(root)

# 存放菜单状态
openVar = IntVar()
saveVar = IntVar()
quitVar = IntVar()
editVar = IntVar()

# 点击菜单后调用的函数
def callback():
    print("你好~")
    
# 创建“文件”菜单
filemenu = Menu(menubar, tearoff=False)
filemenu.add_checkbutton(label="打开", command=callback, variable=openVar)
filemenu.add_checkbutton(label="保存", command=callback, variable=saveVar)

# 增加一个分隔符
filemenu.add_separator()

filemenu.add_checkbutton(label="退出", command=root.destroy, variable=quitVar)
menubar.add_cascade(label="文件", menu=filemenu)

# 创建“编辑”菜单
editmenu = Menu(menubar, tearoff=False)
editmenu.add_radiobutton(label="剪切", command=callback, variable=editVar, value=1)
editmenu.add_radiobutton(label="拷贝", command=callback, variable=editVar, value=2)
editmenu.add_radiobutton(label="粘贴", command=callback, variable=editVar, value=3)
menubar.add_cascade(label="编辑", menu=editmenu)

root.config(menu=menubar)

mainloop()
```



如果不调用 config 绑定，可以通过事件调用 post 方法呼出菜单

```python
from tkinter import *

root = Tk()

def callback():
    print('你好~')

menubar = Menu(root)
menubar.add_command(label='撤销', command=callback)
menubar.add_command(label='重做', command=root.destroy)

frame = Frame(root, width=512, height=512)
frame.pack()

def popup(event):
    menubar.post(event.x_root, event.y_root)

frame.bind('<Button-3>', popup)

mainloop()
```



#### Menubutton

一个可以产生下拉菜单的按钮控件，并不常用

```python
from tkinter import *

root = Tk()

def callback():
    print("你好~")

menubar = Menu(root)

# 额外创建一个菜单按钮
mb = Menubutton(root, text='点我', relief=RAISED)
mb.pack()

filemenu = Menu(mb, tearoff=False)

filemenu.add_command(label="打开", command=callback)
filemenu.add_command(label="保存", command=callback)
filemenu.add_separator()
filemenu.add_command(label="退出", command=root.destroy)

# 注意要把菜单配置到按钮上
mb.config(menu=filemenu)

mainloop()
```



#### OptionMenu

设置选项菜单，菜单显示可以修改

```python
from tkinter import *

root = Tk()

variable = StringVar()
variable.set('one')

w = OptionMenu(root, variable, 'one', 'two', 'three')
w.pack()

mainloop()
```

如果选项较多，则通过可变参数传入

```python
OPTIONS = ['a', 'b', 'c', 'd']
w = OptionMenu(root, variable, *OPTIONS)
```



#### Message

与 Label 差不多的组件，但是它具有自动换行等特性，可以自行调整

```python
from tkinter import *

root = Tk()

w1 = Message(root, text='输出一段文字', width=200)
w1.pack()

w2 = Message(root, text='我将会输出一段十分长长长长长长长长长长长长长的文字', width=200)
w2.pack()

mainloop()
```



#### Spinbox

可以点击箭头切换显示的小组件

```python
from tkinter import *

root = Tk()

w = Spinbox(root, values=['a', 'b', 'c'], wrap=True)
w.pack()

mainloop()
```

其中 wrap 设置循环，可以从最后一个选项回到第一个选项。

 

#### PanedWindow

嵌入式窗口，可以独立于 root 存在。它允许自由调节窗口比例，窗口按照创建顺序逐次嵌入

```python
from tkinter import *

root = Tk()
root.title('分割')

# 显示可拖动标签，分割样式为 SUNKEN
m1 = PanedWindow(showhandle=True, sashrelief=SUNKEN)
m1.pack(fill=BOTH, expand=1)

# 添加标签
left = Label(m1, text='left pane')
m1.add(left)

# 纵向可变，显示可拖动标签，分割样式为 SUNKEN
m2 = PanedWindow(orient=VERTICAL, showhandle=True, sashrelief=SUNKEN)
m1.add(m2)

# 添加两个标签
top = Label(m2, text='top pane')
m2.add(top)

bottom = Label(m2, text='bottom pane')
m2.add(bottom)

mainloop()
```

其中 expand 参数设置嵌入窗口是否随主窗口变化而变化。



#### Toplevel

创建顶级窗口，与 root 容器差不多，只是这是额外的窗口。例如

```python
from tkinter import *

root = Tk()

def create():
    top = Toplevel()
    top.title('这是一个病毒')

    msg = Message(top, text='开始分裂')
    msg.pack()
    
Button(root, text='创建顶级窗口', command=create).pack()

mainloop()
```



可以修改顶部窗口的属性

```python
top.attributes('-alpha', 0.5)
```

可定义的属性包括

| 属性       | 含义                                    |
| ---------- | --------------------------------------- |
| alpha      | 控制窗口透明度（0.0-1.0）               |
| disabled   | 禁用整个窗口（只能从任务管理器关闭）    |
| fullscreen | 如果设置为 True，则全屏显示窗口         |
| toolwindow | 如果设置为 True，该窗口采用工具窗口样式 |
| topmost    | 如果设置为 True，该窗口永远置顶         |



### Text 控件

文本控件，用于显示多行文本，功能非常复杂，因此单独开一章来介绍部分功能。常用方法包括

| 方法                   | 作用                                              | 方法                  | 作用                                  |
| ---------------------- | ------------------------------------------------- | --------------------- | ------------------------------------- |
| get(start, end)        | 获得索引范围的内容                                | search(s, start, end) | 获得所找字符串的索引                  |
| config(cursor='arrow') | 设置鼠标的样式，arrow, xterm, circle, cross, plus | insert()              | 插入文本                              |
| index()                | 将传入的索引格式转换为行列索引返回                | bind(key, f)          | 通过按键绑定调用以 event 为参数的函数 |
| window_create()        | 在文本指定位置中插入窗口                          | image_create()        | 插入图片                              |



例如我们在一个文本框中插入一个按钮

```python
from tkinter import *

root = Tk()

text = Text(root, height=10, width=10)
text.pack()

text.insert(INSERT, 'I love ')
text.insert(END, 'FishC.com')

def show():
    print('啊，我被点了一下！')

text.insert('1.2 + 5 chars', '你好')

# 将按钮插入到文本框中
b = Button(root, text='点我点我', command=show)
text.window_create(INSERT, window=b)

mainloop()
```



#### 文本索引

Tkinter 提供一系列索引类型。最简单的索引方式是

* INSERT 插入光标的位置
* CURRENT 鼠标位置
* END 字符串末尾



用 `.` 分隔的**行列索引**，例如

```python
text.get('1.2', '1.6')	# 获得 1 行 2 列到 1 行 6 列的字符串
```

注意**行号从 1 开始，列号从 0 开始**。并且可以使用浮点 `1.2` 代替对应的字符串作为索引。使用

```python
text.get('1.2', '1.end')	# 获得直到 1 行结尾的字符串
```



**Mark 标记索引**，可以设置它的位置，当插入文本后，Mark 会移动。例如

```python
text.mark_set('here', 1.3)	# 标记 here 表示 1.3
text.insert('here', 'Hi')	# 在 1.3 位置的左边插入 Hi，然后标记向右移动
```

插入 Hi 后，此时 `here` 等价于行列索引 `1.5` 。需要注意**Mark 只会记录它右边文本的位置，因此左边文本的变化会影响它的位置**。如果它左右的文本都被删除，则 Mark 初始化为行列索引 `1.0` 。



当然，可以修改插入位置

```python
text.mark_gravity('here', LEFT)
```

这样插入文本会放在右边，从而 Mark 就不会移动。可以删除标记

```python
text.mark_unset('here')
```



#### 文本查找

Tkinter 可以使用表达式计算和移动索引。

| 表达式      | 操作               | 表达式    | 操作               |
| ----------- | ------------------ | --------- | ------------------ |
| '+nc'       | 向后移动 n 个字符  | '-nc'     | 向前移动 n 个字符  |
| '+nl'       | 向后移动 n 行      | '-nl'     | 向前移动 n 行      |
| 'linestart' | 移动到本行开头     | 'lineend' | 移动到本行末尾     |
| 'wordstart' | 移动到这个单词开头 | 'wordend' | 移动到这个单词末尾 |



使用 search 方法查找范围内指定的字符串，例如

```python
text.insert(INSERT, 'I love FishC.com!')

# 开始位置
start = '1.0'
while True:
    # 搜索范围内 o 的位置
    pos = text.search('o', start, stopindex=END)
    if not pos:
        break
    
    # 转换为行列索引输出
	print(text.index(pos))
    
    # 向后移动一个字符
    start = pos + '+1c'
```

获得位置存放在 pos 中，然后通过 index 方法转换为行列索引。



#### Tags 标签

通常用于改变 Text 组件中内容的样式和功能，修改字体、颜色、尺寸，还允许你将文本、嵌入的组件和图片与键盘和鼠标等事件相关联。除了自定义 Tags（user-defined tags），还有一个预定义的特殊 Tag：SEL，用于表示对应的选中内容。



例如我们在指定索引范围增加 tag，然后设置 tag 的背景、前景色以及下划线

```python
text.insert(INSERT,'I love FishC.com')
text.tag_add('tag','1.7','1.12')
text.tag_config('tag',background='red',foreground='yellow',underline=True)
```

其中索引范围是两两配对的，也就是可以指定多对索引范围。



默认新的 tag 会覆盖旧的 tag 重叠的部分。但是可以通过修改 tag 优先级调整覆盖关系

```python
text.tag_raise('tag')
text.tag_lower('tag')
```

我们可以将 tag 插入到文本中

```python
text.insert(INSERT, 'Hello', ('tag1', 'tag2'))
```



此外 tag 还可以与事件绑定，用于实现超链接等功能。例如

```python
from tkinter import *
import webbrowser

root = Tk()

text = Text(root)
text.pack()

# 指定两对索引范围作为一个 tag
text.insert(INSERT,'I love FishC.com')
text.tag_add('link','1.2','1.9','1.12','1.16')
text.tag_config('link',foreground='blue',underline=True)

# 修改鼠标移动到 text 上的样式
def show_arrow_cursor(event):
    text.config(cursor='arrow')

def show_xterm_cursor(event):
    text.config(cursor='xterm')

# 跳转指定网址
def click(event):
    webbrowser.open("http://www.fishc.com")

# 将函数与事件绑定
text.tag_bind('link', '<Enter>', show_arrow_cursor)
text.tag_bind('link', '<Leave>', show_xterm_cursor)
text.tag_bind('link', '<Button-1>', click)

mainloop()
```

注意绑定的事件函数要以 event 作为参数。



可以绑定的鼠标事件包括

* `<Enter>` 鼠标移入标签
* `<Leave>` 鼠标离开标签
* `<Button-1>` 点击鼠标左键
* `<Button-2>` 点击鼠标滚轮
* `<Button-3>` 点击鼠标右键



利用 hashlib 库实现文本修改提醒

```python
from tkinter import *
import hashlib

# 定义哈希函数将内容转换为数字，用来识别
def getSig(contents):
    m = hashlib.md5(contents.encode())
    return m.digest()

root = Tk()

text = Text(root)
text.pack()

text.insert(INSERT, 'I love FishC.com')

# 将文本内容转换为数字
contents = text.get('1.0', END)
sig = getSig(contents)

# 检查当前文本内容通过 Hash 后的值是否与 sig 相同
def check():
    contents = text.get('1.0', END)
    if sig != getSig(contents):
        print('警报，内容发生变动！')
    else:
        print('风平浪静~')

Button(root, text='检查', command=check).pack()

mainloop()
```



#### 撤销文本

创建文本对象时，可以通过参数指定是否可以撤销文本操作。默认情况下，使用换行作为分隔符，即每两次换行之间的操作视为一步操作，这可能导致一次撤销的内容太多。可以取消自动分割，自定义分割操作。例如

```python
from tkinter import *

root = Tk()

# 开启 undo，关闭自动分割
text = Text(root, width=30, height=5, undo=True, autoseparators=False)
text.pack()

text.insert(INSERT, 'I love FishC.com')

# 自定义分割函数
def callback(event):
    text.edit_separator()

# 每次按键都会创建一个分隔符
text.bind('<Key>', callback)

# 撤销操作
def show():
    text.edit_undo()

Button(root, text='撤销', command=show).pack()

mainloop(
```



### ttk

ttk 是 tkinter 的补充组件包，实现了控件和样式的分离，可以改善应用程序的外观和感觉。

```python
import tkinter.ttk as ttk
```

为了防止与 tkinter 的原生组件重名，最好将其导入后命名为 ttk 。



#### Progressbar

进度条组件包括如下参数

| 参数     | 作用                                          |
| -------- | --------------------------------------------- |
| length   | 进度条的长度，默认 100 像素                   |
| mode     | 模式，可选 determinate(默认)，indeterminate   |
| maximum  | 最大值，默认 100 像素                         |
| name     | 进度条名称                                    |
| orient   | 进度条方向，可选 HORIZONTAL(默认) 和 VERTICAL |
| value    | 当前值                                        |
| variable | 记录当前进度值                                |



我们主要介绍 mode 参数，如果知道工作时间，那么使用默认的 determinate 模式即可。如果不知道，那么使用 indeterminate 模式将会显示来回移动的方块，表示正在运行。例如

```python
from tkinter import *
import tkinter.ttk as ttk

app = Tk()
app.title("Demo")

prog = ttk.Progressbar(app, mode="indeterminate")
prog.pack()

# 每次点击按钮，进度条移动，刷新界面
def show():
    prog['value'] += 10
    app.update()

but = Button(app, text="确定", command=show)
but.pack()

app.mainloop()
```



#### ComboBox

下拉框组件 ComboBox 是 Entry 的子类，因此可以使用 Entry 相关的属性和方法。ComboBox 常用的参数展示如下

| 参数        | 作用                                                |
| ----------- | --------------------------------------------------- |
| height      | 显示的选项数，如果可选项多于 height，就会显示滚动条 |
| values      | 传入选项列表                                        |
| postcommand | 当弹出下拉框时调用的函数                            |

可调用的方法主要有两个

| 方法           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| set(value)     | 设置当前显示的内容，可以不在 values 中                       |
| current()      | 获得当前显示内容在 values 中的索引，如果不在 values 中，则返回 -1 |
| current(index) | 更新当前显示内容为 values[index] 的值，不返回值              |

例如通过 postcommand 动态调整下拉框选项，并通过按钮显示当前内容的索引

```python
from tkinter import *
import tkinter.ttk as ttk

app = Tk()
app.title("Demo")

values = ['1','2','3','4','5','6']

# 每次弹出下拉框，都会增加一项
def change():
    values.append(str(int(values[-1]) + 1))
    comb['values'] = values

comb = ttk.Combobox(
    app,
    height=3,
    values=values,
    postcommand=change
)
comb.pack()

# 定义 Button 按下时刷新文本
def current():
    but['text'] = 'Current' + str(comb.current())

but = ttk.Button(app, text='Current', command=current)
but.pack()

app.mainloop()
```



#### TreeView

TreeView 控件是一个分层列表显示控件。常用参数有

| 参数       | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| columns    | 列名称的列表                                                 |
| height     | 可见行数                                                     |
| padding    | 内边距                                                       |
| selectmode | 可选 extended(默认多选), browse(单选) 或 none(不可选)        |
| show       | 指定显示元素。可选 `["tree"],["headings"],["tree", "headings"](默认)`，其中 tree 表示显示列标，headings 表示显示行标 |
| text       | 文本标签                                                     |
| image      | 显示在文本左边的图片                                         |



例如创建 TreeView

```python
tree = ttk.Treeview(root)
tree["columns"] = ["column1", "column2"]

tree.heading("#0", text="Name")				# 设置左上角标题名称
tree.heading("column1", text="Column 1")	# 设置 column1 标题名
tree.heading("column2", text="Column 2")	# 设置 column2 标题名
```



使用 insert 方法向树中插入项

```python
tree.insert("", "end", text="Item 1", values=("Value 1-1", "Value 1-2"))
```

其中第一个参数指定所属项，第二个参数指定插入位置，text 指定行标题，values 指定传入值。因此可以插入子项

```python
item = tree.insert("", "end", text="Item 1", values=("Value 1-1", "Value 1-2"))
tree.insert(item, "end", text="Item 2", values=("Value 2-1", "Value 2-2"))
```



使用 delete 删除项及其子项

```python
item = tree.insert("", "end", text="Item 1", values=("Value 1-1", "Value 1-2"))
tree.insert(item, "end", text="Item 2", values=("Value 2-1", "Value 2-2"))
tree.delete(item)
```



### 消息事件

#### 基本事件

任何一个组件都可以绑定一个消息事件。可触发的事件有

| 事件         | 含义     |
| ------------ | -------- |
| `<Button-1>` | 左键点击 |
| `<Button-2>` | 滚轮点击 |
| `<Button-3>` | 右键点击 |
| `<Key>`      | 键盘输入 |
| `<Motion>`   | 鼠标移动 |



event 对象包含事件的信息

| 属性           | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
| widget         | 产生该事件的组件                                             |
| x, y           | 当前鼠标位置坐标（窗口左上角）                               |
| x_root, y_root | 当前鼠标位置坐标（屏幕左上角）                               |
| char           | 按键对应的字符（键盘事件专属）                               |
| keysym         | 按键名，key names（键盘事件专属）获取具体按键的名称（Alt-L 左边的 Alt 键） |
| keycode        | 按键码，key codes（键盘事件专属）获取具体按键的代码（64 表示 Alt-L） |
| num            | 按钮数字（鼠标事件专属）                                     |
| width, height  | 组件新尺寸（Configure 事件专属）                             |
| type           | 该事件类型                                                   |

 

只有获得焦点的组件才能收到事件

```python
from tkinter import *

root = Tk()

# 定义事件消息函数，输出事件包含的字符
def callback(event):
    print(event.char)

# frame 绑定键盘按键，设为焦点
frame = Frame(root, width=200, height=200)
frame.bind("<Key>", callback)
frame.focus_set()
frame.pack()

mainloop()
```



鼠标移动事件可以不设置焦点，只要点击控件即可获得焦点

 ```python
 from tkinter import *
 
 root = Tk()
 
 # 定义事件消息函数，输出事件位置
 def callback(event):
     print('当前位置：', event.x, event.y)
 
 frame = Frame(root, width=200, height=200)
 frame.bind('<Motion>', callback)
 frame.pack()
 
 mainloop()
 ```



#### 自定义事件

用户可自定义事件，格式为

```python
'<modifier-type-detail>'
```

其中 type 描述普通事件类型，modifier 描述组合键，detail 描述具体按键。例如

```python
'<Button-1>' 					# 点击鼠标左键
'<KeyPress-H>' 					# 点击 H 键
'<Control-Shift-KeyPress-H>' 	# 同时点击 Ctrl+Shift+H 键
'<Double-Button-1>' 			# 双击鼠标左键
```



modifier 可选类型

| modifier | 含义                       |
| -------- | -------------------------- |
| Alt      | 当按下 Alt 键时            |
| Any      | 任何按键被按下时           |
| Control  | 当按下 Ctrl 键时           |
| Double   | 当后续两个事件被连续触发时 |
| Lock     | 当打开大写字母锁定键时     |
| Shift    | 当按下 Shift 键时          |
| Triple   | 当后续三个事件被连续触发时 |

 

type 可选类型

| type          | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| Activate      | 当组件状态从未激活到激活时触发                               |
| Button        | 点击鼠标时触发，detail 部分指定按键：1 左键，2 滚轮，3 右键，4 滚轮上滚，5 滚轮下滚 |
| ButtonRelease | 释放鼠标按键触发                                             |
| Configure     | 当组件尺寸改变时触发                                         |
| Deactivate    | 当组件状态从激活到未激活时触发                               |
| Destroy       | 组件被销毁时触发                                             |
| Enter         | 鼠标进入组件时触发                                           |
| Expose        | 当窗口或组件的某部分不再被覆盖时触发                         |
| FocusIn       | 组件获得焦点时触发。可以用 Tab 键将焦点转移到该组件上（需要组件的 takefocus 选项为 True） |
| FocusOut      | 当组件失去焦点时触发                                         |
| Key           | 当按下键盘按键时触发，detail 指定具体按键                    |
| KeyRelease    | 当用户释放键盘按键时触发                                     |
| Leave         | 当鼠标离开组件时触发                                         |
| Map           | 当组件在应用程序中显示时触发                                 |
| Motion        | 当鼠标在组件中移动时触发                                     |
| MouseWheel    | 当鼠标滚轮滚动时触发                                         |
| Unmap         | 当组件在应用程序中不再显示时触发                             |
| Visibility    | 当应用程序至少有一部分在屏幕上可见时触发                     |



使用 after() 函数刷新界面

```python
def func():
	print('s')

# 每隔 1000 ms 刷新界面
root.after(1000, func)
```

要实现循环调用，需要在函数内外都调用 after 。



### 对话框

#### messagebox

弹窗组件，用于提示

```python
from tkinter import *
from tkinter.messagebox import *
```



主要包括

| 消息框                            | 作用               |
| --------------------------------- | ------------------ |
| showinfo('tip', 'detail')         | 当按下“是”，返回ok |
| showwarning('warining', 'detail') | 当按下“是”，返回ok |
| showerror('error', 'detail')      | 当按下“是”，返回ok |



| 对话框                          | 作用                         |
| ------------------------------- | ---------------------------- |
| askokcancel('tip', 'detail')    | 确定/取消，返回值 True/False |
| askquestion('tip', 'detail')    | 是/否，返回值 yes/no         |
| askyesno('tip', 'detail')       | 是/否，返回值 True/False     |
| askretrycancel('tip', 'detail') | 重试/取消，返回值 True/False |

 

还可以添加可选 options 参数

| 选项    | 含义                                                         |
| ------- | ------------------------------------------------------------ |
| default | 设置按回车选择的按钮（ 默认是第一个按钮）可选值：CANCEL, IGNORE, OK, NO, RETRY, YES |
| icon    | 指定对话框显示的图标，可选值 ERROR, INFO, QUESTION, WARNING3 |
| parent  | 设置父窗口，默认为根窗口                                     |

 

#### filedialog

文件对话框，用于打开和保存文件

```python
from tkinter import *
from tkinter.filedialog import *

root = Tk()

def callback():
    filename = askopenfilename()
    print(filename)

Button(root, text='打开文件', command=callback).pack()

mainloop()
```



常用方法有

| 方法                | 作用                 |
| ------------------- | -------------------- |
| asksaveasfilename() | 保存文件，返回文件名 |
| asksaveasfile()     | 会创建文件           |
| askopenfilename()   | 打开文件，返回文件名 |
| askopenfile()       | 返回文件流对象       |
| askdirectory()      | 返回目录名           |
| askopenfilenames()  | 可以返回多个文件名   |
| askopenfiles()      | 多个文件流对象       |



方法的可选参数

| 选项             | 含义                                                         |
| ---------------- | ------------------------------------------------------------ |
| defaultextension | 指定文件的后缀。当用户输入一个文件名时，会自动添加给定后缀。输入文件名包含后缀时该选项不生效 |
| filetypes        | 指定筛选文件类型的下拉菜单选项，该选项的值是二元组构成的列表 |
| initialdir       | 指定打开/保存文件的默认路径                                  |
| parent           | 设置父窗口，默认为根窗口                                     |
| title            | 指定文件对话框的标题栏文本                                   |

例如设置打开文件类型为 PNH 类型

```python
filedialog.askopenfilename(filetypes=[('PNG','png')])
```



## Pygame

Pygame 是一个用 Python 写的 SDL 库。SDL 是一个能访问计算机多媒体硬件组件（包括声卡、视频卡、输入组件等）的跨平台库。通过使用 Pygame 我们可以快捷地创建图形化界面，进行图像、音频操作，并进行灵活的消息处理。



例如我们绘制移动的图像，碰到边界就反转并返回

```python
import pygame
import sys

# 初始化 pygame
pygame.init()

# 用于控制帧率，从而控制速度
clock = pygame.time.Clock()

# 创建指定大小的窗口
size = width, height = 600, 400
screen = pygame.display.set_mode(size)

# 设置窗口标题
pygame.display.set_caption('初次见面，请大家多多关照！')

# 加载图片，获取图像的位置矩形
turtle = pygame.image.load('turtle.png')
position = turtle.get_rect()

# 初始化速度和背景色
speed = [-2, 1]
bg = (255, 255, 255)

# 循环移动
while True:
    # 消息循环，捕获退出消息
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            sys.exit()

    # 移动图像
    position = position.move(speed)

    # 边界检测，超出边界就反向
    if position.left < 0 or position.right > width:
        turtle = pygame.transform.flip(turtle, True, False)
        speed[0] = -speed[0]

    if position.top < 0 or position.bottom > height:
        speed[1] = -speed[1]

    screen.fill(bg)                 # 填充背景
    screen.blit(turtle, position)   # 更新图像
    pygame.display.flip()           # 更新界面

    # 延迟 10 毫秒，用于控制刷新速度
    pygame.time.delay(10)
    clock.tick(200)
    
```



### 初始化

导入 pygame 局部模块，然后调用初始化函数

```python
import pygame
from pygame.locals import *

pygame.init()
```



### 窗口方法

#### set_mode

用于设置窗口属性，格式为

```python
pygame.display.set_mode(resolution=(0, 0), flags=0, depth=0)
```

其中 resolution 设置窗口尺寸，flags 设置窗口模式，可选

| 选项       | 模式                         |
| ---------- | ---------------------------- |
| FULLSCREEN | 全屏模式                     |
| DOUBLEBUF  | 双缓冲模式                   |
| HWSURFACE  | 硬件加速支持（全屏模式使用） |
| OPENGL     | 使用 OpenGL 渲染             |
| RESIZABLE  | 使得窗口可以调整大小         |
| NOFRAME    | 使得窗口没有边框和控制按钮   |

多种模式可以用 `|` 连接设置。



#### set_caption

设置窗口标题

```python
pygame.display.set_caption('Hello')
```



#### list_modes

获取可以设置的窗口参数（长度, 宽度）

```python
l = pygame.display.list_modes()
```



#### fill

填充窗口背景

```python
bg = 255, 255, 255
screen = pygame.display.set_mode(200, 100)
screen.fill(bg)
```

其中 bg 参数为 RGB 颜色。



#### filp

更新界面，完成图形图像绘制后，需要调用此方法来重绘界面

```python
pygame.display.flip()
```



#### quit

退出窗口

```python
pygame.quit()
```



### 事件处理

#### 获取事件

使用 get 方法获得事件的序列

```python
pygame.event.get()
```

事件对象 `pygame.event.Event()` 通过 `type` 属性保存事件类型。例如

```python
for event in pygame.event.get():
	if event.type == pygame.QUIT:
		sys.exit()
```



可以写一个将所有事件信息输出到屏幕上的程序

```python
import pygame
import sys

pygame.init()

size = width, height = 600, 400
screen = pygame.display.set_mode(size)
pygame.display.set_caption('初次见面，请大家多多关照！')

# 设置字体，并获得高度
font = pygame.font.Font(None, 20)
line_height = font.get_linesize()

bg = (0, 0, 0)
screen.fill(bg)

position = 0

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            sys.exit()

        # 在屏幕上打印事件信息，然后移动位置
        screen.blit(font.render(str(event), True, (0, 255, 0)), (0 , position))
        position += line_height

        # 当屏幕被写满，就用指定颜色刷新屏幕
        if position > height:
            position = 0
            screen.fill(bg)

        pygame.display.flip()

```



#### 事件类型

pygame 可以处理不同的消息，因而有多种不同的事件类型。

| 事件                   | 含义             |
| ---------------------- | ---------------- |
| pygame.QUIT            | 退出事件         |
| pygame.VIDEORESIZE     | 改变窗口大小触发 |
| pygame.MOUSEMOTION     | 鼠标移动事件     |
| pygame.MOUSEBUTTONDOWN | 鼠标按下事件     |
| pygame.MOUSEBUTTONUP   | 鼠标松开事件     |
| pygame.KEYDOWN         | 键盘按下事件     |

由于我们引入了 `pygame.locals`，因此上面的事件类型就不必使用 `pygame` 前缀。



#### 键盘事件

如果检测到

```python
event.type == KEYDOWN
```

那么就可以通过 key 属性获得按钮事件。

```python
event.key
```

可以设置是否重复按键事件，用于处理按键长按

```python
pygame.key.set_repeat(delay, interval)
```

其中 delay 指定第一次发送事件的延迟，interval 指定重复发送事件的时间间隔。如果不带任何参数，表示取消重复发送按键事件。



所有键盘事件以 pygame.locals 中的变量定义

| 变量     | 含义                 |
| -------- | -------------------- |
| K_A      | A 键                 |
| K_F11    | F11 键               |
| K_LEFT   | 左箭头               |
| K_SPACE  | 空格                 |
| K_EQUALS | 等号（不是数字键盘） |
| K_MINUS  | 减号（不是数字键盘） |



可以获取所有键盘按键的布尔值

```python
# 获得按键及其布尔值的字典
key_pressed = pygame.key.get_pressed()

# 检测该按键是否为 True
if key_pressed[K_a]:
    ...
```



例如通过事件控制的图像移动和反转代码

```python
import pygame
from pygame.locals import *
import sys

#初始化pygame
pygame.init()

size = width, height = 600, 400
speed = [-2, 1]
bg = (255, 255, 255)

#用于控制帧率，从而控制速度
clock = pygame.time.Clock()

# 控制是否全屏
fullscreen = False

screen = pygame.display.set_mode(size, RESIZABLE)
pygame.display.set_caption('初次见面，请大家多多关照！')

# 设置放大缩小的比率
ratio = 1.0

# 加载图片
oturtle = pygame.image.load('turtle.png')
turtle = oturtle

# 获取图像的位置矩形
oturtle_rect = oturtle.get_rect()
position = turtle_rect = oturtle_rect

# 创建原始图像和左右反转的图像
l_head = turtle
r_head = pygame.transform.flip(turtle, True, False)

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            sys.exit()

        # 根据方向键设置图像方向和移动方向
        if event.type == KEYDOWN:
            if event.key == K_LEFT:
                turtle = l_head
                speed = [-1, 0]
            if event.key == K_RIGHT:
                turtle = r_head
                speed = [1, 0]
            if event.key == K_UP:
                speed = [0, -1]
            if event.key == K_DOWN:
                speed = [0, 1]

            # 全屏（F11）
            if event.key == K_F11:
                fullscreen = not fullscreen
                if fullscreen:
                    screen = pygame.display.set_mode((1024, 768), FULLSCREEN | HWSURFACE)
                    width, height = 1024, 768
                else:
                    screen = pygame.display.set_mode(size)

            # 放大、缩小小乌龟（=、-），空格键恢复原始尺寸
            if event.key == K_EQUALS or event.key == K_MINUS or event.key == K_SPACE:
                # 最大只能放大一倍，缩小 0.5
                if event.key == K_EQUALS and ratio < 2:
                    ratio += 0.1
                if event.key == K_MINUS and ratio >0.5:
                    ratio -= 0.1
                if event.key == K_SPACE:
                    ratio = 1.0

                # 缩放图像
                turtle = pygame.transform.smoothscale(oturtle, \
                                             (int(oturtle_rect.width * ratio), \
                                             int(oturtle_rect.height * ratio)))

                # 用缩放图像重新初始化原始图像和反转图像
                l_head = turtle
                r_head = pygame.transform.flip(turtle, True, False)

        # 用户调整窗口尺寸，输出改变后的尺寸
        if event.type == VIDEORESIZE:
            size = event.size
            width, height = size
            print(size)
            screen = pygame.display.set_mode(size, RESIZABLE)
            

    # 移动图像
    position = position.move(speed)

    # 碰到边界反向并反转
    if position.left < 0 or position.right > width:
        turtle = pygame.transform.flip(turtle, True, False)
        speed[0] = -speed[0]

    if position.top < 0 or position.bottom > height:
        speed[1] = -speed[1]

    
    screen.fill(bg)                 # 填充背景
    screen.blit(turtle, position)   # 更新图像
    pygame.display.flip()           # 更新界面

    # 延迟 10 毫秒，用于控制刷新速度
    pygame.time.delay(10)
    clock.tick(200)
    

```



#### 鼠标事件

如果检测到

```python
event.type == MOUSEMOTION
event.type == MOUSEBUTTONDOWN
event.type == MOUSEBUTTONUP
```

就说明触发对应的鼠标事件。此时可获得和设置鼠标信息

```python
pygame.mouse.get_pos() 			# 获取鼠标位置
pygame.mouse.set_visible(True) 	# 设置鼠标是否可见
```



也可以直接通过属性获得。例如通过 event.button 获取鼠标操作的事件

* 1 左键
* 2 滚轮
* 3 右键
* 4 滚轮上滚
* 5 滚轮下滚

通过 event.pos 获取鼠标操作的位置。

 

### time

#### 定时器

使用 time 模块可以设置定时器，例如设置 1000 ms 触发一次的事件

```python
timer = pygame.time.set_timer(USEREVENT, 1000)
```

其中 USEREVENT 用于标记定时器，所有定时事件从它开始按照顺序标记

```python
timer2 = pygame.time.set_timer(USEREVENT + 1, 2000)
```



定时器发送的事件类型就是其标记，即

```python
event.type == USEREVENT + 1
```

说明 timer2 被触发。



#### 延时控制

可以设置两次刷新之间的延时

```python
pygame.time.delay(10)
```



#### 帧率控制

可以控制刷新帧数

```python
clock = pygame.time.Clock()
clock.tick(200)
```

此时刷新率是每秒 200 次。



### Surface

Surface 是用于表示图像的对象。事实上，所有与绘制相关的操作都与它有关。我们创建的窗口对象其实也是一个 Surface 对象

```python
# 返回的是 Surface 对象
screen = pygame.display.set_mode(200, 100)
```

也可以自己创建

```python
temp = pygame.Surface((200, 100))
```



Surface 对象的常用方法有

| 方法            | 作用                                  |
| --------------- | ------------------------------------- |
| blit()          | 将一个图像重叠到另一个图像上方        |
| convert()       | 修改图像的像素格式                    |
| convert_alpha() | 修改图像的像素格式，包含 alpha 通道   |
| set_alpha()     | 设置整个图像的透明度                  |
| get_alpha()     | 获取整个图像的透明度                  |
| copy()          | 创建一个 Surface 对象的拷贝           |
| fill()          | 使用纯色填充 Surface 对象             |
| subsurface()    | 根据父对象创建一个新的子 Surface 对象 |
| get_rect()      | 获取位置矩形                          |
| set_colorkey()  | 去除指定颜色                          |
| get_at()        | 获得指定位置的颜色和透明度数据        |
| set_at()        | 设置指定位置的颜色和透明度数据        |
| get_width()     | 获取宽度                              |
| get_height()    | 获取高度                              |



例如我们可以通过 pygame 的 Rect 对象复制 Surface 的一部分

```python
select_rect = pygame.Rect(0, 0, 100, 100)
capture = image.subsurface(select_rect).copy()
```



#### copy

通过复制操作，实现框选复制的操作

```python
import pygame
from pygame.locals import *
import sys
import math

pygame.init()

clock = pygame.time.Clock()

bg = 255, 255, 255
size = width, height = 600, 400
screen = pygame.display.set_mode(size)
pygame.display.set_caption('框选乌龟')

turtle = pygame.image.load('turtle.png')

# 0-未选择，1-选择中，2-完成选择
select = 0
select_rect = pygame.Rect(0, 0, 0, 0)

# 0-未拖拽，1-拖拽中，2-完成拖拽
drag = 0

# 记录图案中心位置
position = turtle.get_rect()
position.center = width / 2, height // 2

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            sys.exit()

        elif event.type == MOUSEBUTTONDOWN:
            if event.button == 1:
                # 第一次点击，选择范围
                if select == 0 and drag == 0:
                    pos_start = event.pos
                    select = 1
                # 第二次点击，拖拽图像
                elif select == 2 and drag == 0:
                    capture = screen.subsurface(select_rect).copy()
                    cap_rect = capture.get_rect()
                    drag =1
                # 第三次点击，初始化
                elif select == 2 and drag == 2:
                    select = 0
                    drag = 0

        elif event.type == MOUSEBUTTONUP:
            if event.button == 1:
                # 第一次释放，结束选择
                if select == 1 and drag ==0:
                    pos_stop = event.pos
                    select = 2
                if select == 2 and drag == 1:
                    drag = 2

    # 刷新背景
    screen.fill(bg)
    screen.blit(turtle, position)

    # 实时绘制选择框
    if select:
        mouse_pos = pygame.mouse.get_pos()
        if select == 1:
            pos_stop = mouse_pos

        select_rect.left, select_rect.top = pos_start
        select_rect.width, select_rect.height = math.fabs(pos_stop[0] - pos_start[0]), math.fabs(pos_stop[1] - pos_start[1])
        pygame.draw.rect(screen, (0, 0, 0), select_rect, 1)

    # 拖拽裁剪的图像
    if drag:
        if drag == 1:
            cap_rect.center = mouse_pos
        screen.blit(capture, cap_rect)

    # 更新界面
    pygame.display.flip()

    clock.tick(30)
    

```



#### alpha

我们还可以修改图像的透明度，但是可能遇到一些问题。例如要修改 PNG 图像的透明度，对于那些已经是“空”的部分，透明度是零，如果直接用 set_alpha 处理，会导致原本透明的部分不透明，因此需要逐个像素修改

```python
for i in range(position.width):
	for j in range(position.height):
		temp = turtle.get_at((i, j))
        
        # 检查透明度，不为零才修改
		if temp[3] != 0:
			temp[3] = 200
            
		turtle.set_at((i, j), temp)
```



但这样效率非常低，可以通过将背景和图像都进行绘制，然后调整透明度来实现

```python
# target 输出目标，source 需要修改透明度的图像，location 图像矩形，opacity 目标透明度
def blit_alpha(target, source, location, opacity):
    x = location[0]
    y = location[1]
    
    # 先创建一个同等大小的 surface，然后将 target 和 source 分别都绘制到上面
    temp = pygame.Surface((source.get_width(), source.get_height())).convert()
    
    # 目标是屏幕，相对于 surface 的位置是 -x, -y
    # 源文件相对 surface 的位置是 0,0
    temp.blit(target, (-x, -y))
    temp.blit(source, (0, 0))
    
    # 然后转换透明度，将 surface 绘制到目标上
    temp.set_alpha(opacity)
    target.blit(temp, location)
```



#### 绘制图形

Rect 对象可以直接绘制在 Surface 上，例如

```python
pygame.draw.rect(Surface, color, pos, width=0)
```

类似地可以绘制更多图案

```python
pygame.draw.polygon(Surface, color, points, width=0)
pygame.draw.circle(Surface, color, pos, radius, width=0)
pygame.draw.ellipse(Surface, color, Rect, width=0)
pygame.draw.arc(Surface, color, Rect, start_angle, stop_angle, width=0)
pygame.draw.line(Surface, color, start_pos, stop_pos, width=1)
pygame.draw.lines(Surface, color, closed, pointlist, width=1)
pygame.draw.aaline(Surface, color, start_pos, stop_pos, blend=1)
pygame.draw.aalines(Surface, color, closed, pointlist, blend=1)
```

其中 aa 前缀表示反走样。



### 绘图方法

#### 绘制文字

首先要载入字体

```python
aFont = pygame.font.Font('font.ttf', 63)
```

然后调用其绘制方法

```python
text = aFont.render('bomb', True, WHITE)	# 指定绘制文字，反走样，颜色
screen.blit(text, (0, 0))					# 将文字绘制到指定位置
```



#### 绘制图像

首先要导入图像

```python
turtle = pygame.image.load(path)
bg = turtle.convert()			# 进行转换，这样可以使得在屏幕上绘制的速度更快
bg = turtle.convert_alpha()		# 含有 alpha 通道的图片（支持部分位置透明，如 PNG 图像）的转换
```

 

使用 transform 对图像进行操作，例如

```python
turtle = pygame.transform.flip(turtle, True, False)
```

将 turtle 图像左右翻转后重新赋值。



更多的方法有

| 方法        | 作用                 |
| ----------- | -------------------- |
| flip        | 上下、左右翻转图像   |
| scale       | 缩放图像（快速）     |
| rotate      | 旋转图像             |
| rorozoom    | 缩放并旋转图像       |
| scale2x     | 快速放大一倍图像     |
| smoothscale | 平缓放大图像（精准） |
| chop        | 裁剪图像             |



利用 Rect 对象的方法，可以移动图像位置

```python
speed = [2, 1]
position = turtle.get_rect()
position = position.move(speed)
```

这里修改 position 就会对应地修改图像的位置。



### 动画精灵

pygame 提供的动画精灵可以用于关键的碰撞检测操作。使用时，创建类继承动画精灵

```python
class Ball(pygame.sprite.Sprite):
    def __init__(self):
        # 调用动画精灵的初始化方法
        pygame.sprite.Sprite.__init__(self)
```



继承了动画精灵的类就可以使用它的碰撞检测方法。首先要建立一个需要检测的组

```python
group = pygame.sprite.Group()
```

将需要检测的类添加/删除

```python
group.add(obj1)
group.remove(obj2)
```



#### collide_circle

用于对两个圆形物体进行碰撞检测

```python
pygame.sprite.collide_mask(obj1, obj2)
```

使用这个方法需要在类中定义 radius 属性。



#### collide_mask

对两个物体的非透明部分进行碰撞检测

```python
pygame.sprite.collide_mask(obj1, obj2)
```



#### spritecollide

使用它让一个物体与一组物体进行碰撞检测

```python
pygame.sprite.spritecollide(obj, group, dokill, collided=None)
```

其中 obj 为检测的图像，group 为其它图像，dokill 参数设置碰撞后是否消除对象，collided 参数设置检测函数。



### 音频处理

使用 mixer 模块进行音频处理。首先需要进行初始化

```python
pygame.mixer.init()
```

 

#### 播放音效

创建一个音效的对象，可以设置音量

```python
cat_sound = pygame.mixer.Sound('cat_sound.mp3')
cat_sound.set_volume(0.2)
```

音效可以同时播放多个。



播放音效的方法包括

| 方法               | 作用                           | 方法               | 作用                       |
| ------------------ | ------------------------------ | ------------------ | -------------------------- |
| play()             | 播放音效（参数为-1时循环播放） | stop()             | 停止播放                   |
| fadeout()          | 淡出                           | set_volume()       | 设置音量                   |
| get_volume()       | 获取音量                       | set_num_channels() | 设置同时播放的最大音效数量 |
| get_num_channels() | 计算该音效播放次数             | get_length()       | 获取音效长度               |

 

#### 播放音乐

使用其中的 music 模块播放音乐

```python
pygame.mixer.music.load('bgm.mp3')
pygame.mixer.music.play()
```

音乐以队列的形式，按顺序播放。



播放音乐的方法包括

| 方法           | 作用                             | 方法           | 作用                           |
| -------------- | -------------------------------- | -------------- | ------------------------------ |
| load()         | 载入音乐                         | play()         | 播放音乐（参数为-1时循环播放） |
| stop()         | 停止播放                         | rewind()       | 重新播放                       |
| pause()        | 暂停播放                         | unpause()      | 恢复播放                       |
| fadeout()      | 淡出                             | set_volume()   | 设置音量                       |
| get_volume()   | 获取音量                         | get_busy()     | 检测音乐是否正在播放           |
| set_pos()      | 设置开始播放的位置               | get_pos()      | 获取已经播放的时间             |
| queue()        | 将音乐文件放入待播放列表中       | set_endevent() | 在音乐播放完毕时发送事件       |
| get_endevent() | 获取音乐播放完毕时发送的事件类型 |                |                                |



## Yaml

### 基本介绍

YAML 是一种数据格式，类似于 JSON，支持注释、换行、多行字符串、裸字符串等。它具有比 JSON 更高的性能。YAML 用于作为配置文件，配置全局的数据：环境变量、数据库信息、账号信息、日志格式等。用于写测试用例（接口自动化测试用例）。



### 语法规则

#### 基本语法

* 大小写敏感（区分大小写）
* 使用缩进（**空格，而不是 Tab**）表示层级关系
* 缩进不管空格的数量，只要层级的左边对齐
* 用 `#` 表示注释



#### 对象

对象是一组键值对，用 `:` 表示，每个 `:` 后面**必须有一个空格**。

```yaml
msxy:
  name: Li
  age: 10
```

另一种写法

```yaml
msxy: {name: Li,age: 10}
```



同一层级不能有相同的 key

```yaml
# 这种写法是错误的
name: Li
name: wang
```

同一层级的键必须对齐

```yaml
A:
  B: 1
  C: 2
```

如果没有对齐，就可能导致错误的层级

```yaml
A:
  B:
    C: 2
```



#### 数组

以 `-` 开头表示数组的元素。例如

```yaml
# name, age 是 msxy 数组中的两个元素
msxy:
  - name: Li
  - age: 10
```

另一种写法

```cpp
msxy: [{'name': 'Li', 'age': 10}]
```



#### 锚点和引用

使用 `&` 定义锚点

```yaml
prod:
    driver: &name abcdefghijklmn
	
dev:
    driver: *name
```

这等价于 json 格式

```json
{
  "prod": {
    "driver": "abcdefghijklmn"
  },
  "dev": {
    "driver": "abcdefghijklmn"
  }
}
```



还可以将整个结构复制

```yaml
prod: &name
    driver: abcdefghijklmn
	
dev:
    driver: *name
```

这等价于 json 格式

```json
{
  "prod": {
    "driver": "abcdefghijklmn"
  },
  "dev": {
    "driver": {
      "driver": "abcdefghijklmn"
    }
  }
}
```



如果需要直接将整个结构进行拼接，使用

```yaml
prod: &name
    driver: abcdefghijklmn
	
dev:
    <<: *name
```

这等价于 json 格式

```json
{
  "prod": {
    "driver": "abcdefghijklmn"
  },
  "dev": {
    "driver": "abcdefghijklmn"
  }
}
```



#### JSON

Map 对象：键值对

```json
{'name': 'Li', 'age': 10}
```

数组：

```json
'msxy': [{'name': 'Li', 'age': 10}]
```



### 配置文件处理

安装第三方库

```shell
(.venv) $ pip install pyyaml
```

然后封装读写操作

```python
import yaml


class ReadConfig:
    def __init__(self, yaml_file) -> None:
        self.yaml_file = yaml_file

    def read_yaml(self):
        with open(self.yaml_file, encoding="utf-8") as f:
            msxy_dict = yaml.load(stream=f.read(), Loader=yaml.FullLoader)
            print(msxy_dict)

    def write_yaml(self):
        with open(self.yaml_file, encoding="utf-8", mode='w') as f:
            data = {'y': [{'name': '李'}, {'age' : 12}]}
            yaml.dump(data, stream=f, allow_unicode=True)

```