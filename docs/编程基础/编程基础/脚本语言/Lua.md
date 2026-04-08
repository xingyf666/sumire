# Lua

## 基本介绍

### 配置安装

在官网找到预编译二进制版本，下载 Win64 对应的 exe 工具包，直接使用解压后的文件夹。

```embed
title: "Lua Binaries"
image: "https://luabinaries.sourceforge.net/lua.gif"
description: "LuaBinaries is a distribution of the Lua libraries and executables compiled for several platforms."
url: "https://luabinaries.sourceforge.net/"
```

命令行输入

```shell
lua54 -v
```

查看安装版本。可以复制一份 `lua54.exe` 保存为 `lua.exe`，方便 VSCode 调用。



## 基本语法

### 声明变量

直接赋值来声明

```lua
local a = 1
local b = 2
local a, b = 1, 2
print(a, b)
```

如果不加 `local` 修饰，则默认为全局变量（变量命名最好为大写）。



### 空值

未赋值的变量为 `nil` 空值

```lua
local a
```



### 数值型

可以使用十六进制、科学计数法

```lua
local a = 1
local b = 0x10
local c = 1.23
local d = 2e10
```



### 运算符

支持常见运算符

```lua
local a = 1
a = a << 1
a = a + 1
a = a * a
```



### 字符串

使用 `"` 和 `'` 声明字符串

```lua
local a = "hello"
local b = 'world'
```



#### 原始字符串

使用 `[[]]` 声明多行原始字符串

```lua
local c = [[abc
csaada\n
casca
]]

print(c)
```



#### 连接字符串

使用 `..` 连接字符串

```lua
local a = "hello"
local b = 'world'
local c = a..b
```



#### 类型转换

可以在数字与字符串之间转换

```lua
local c = tostring(123)
local n = tonumber(c)
```



#### 长度

使用 `#` 获得字符串长度

```lua
local a = "Hello, world!"
print(#a)
```



#### 下标

使用 `[]` 下标访问字符串元素

```lua
local a = "hello"
print(a[1])
print(a[-1])
```



#### 成员函数

使用 `string` 表中定义的函数

```lua
local s = "Hello, world!"
print(string.byte(s, 1)) -- 获得 H 对应的编码 72
```



### 函数

#### 定义

定义函数

```lua
function f(a, b)
    print(a, b)
    return a + b
end

f(1, 2)
```



#### 返回值

可以同时返回多个值

```lua
function f(a, b)
    return a, b
end

local a, b = f(1, 2)
print(a, b)
```



#### 可变参数

允许使用可变参数，当存在可变参数时，它必须是最后一个参数。可变参数以数组的形式传入，通过 `#` 可以获得参数数量。

```lua
function average(...)
    local sum = 0
    local arg = { ... }
    for i, v in ipairs(arg) do
        sum = sum + v
    end
    return sum / #arg
end

average(1, 2, 3, 4, 5) -- returns 3
```



#### 匿名函数

定义一个匿名函数并赋值给一个局部变量

```lua
function create()
    local sub = function(a, b)
        return a - b
    end

    print(sub(10, 5))
end

create()
```



### 数组/表

#### 数字下标

使用 `{}` 定义数组，可以存放任意类型，包括函数

```lua
local a = {1, "ac", {}, function() return 1 end}
print(a[1])
print(a[2])
print(a[3])
print(a[4]())
print(a[5])
a[5] = 123
```

注意下标从 `1` 开始，如果下标越界，则默认为 `nil`，也可以赋值。



可以将数组作为表进行操作

```lua
local a = {1, "ac", {}, function() return 1 end}
table.insert(a, "bc")   		-- 插入到最后
print(a[5])
table.insert(a, 2, "de")    	-- 插入到 2 位置
print(a[2])
local b = table.remove(a, 3)    -- 删除 3 位置的元素
print(b)
```



#### 特殊下标

使用字符串作为元素

```lua
local a = {
    a = 1,
    b = "hello",
    c = function ()

    end,
    d = 2
}

print(a.a)
print(a["b"])
```

只有符合变量命名规范的元素可以通过 `.` 访问，对于不符合规范的元素，只能用 `[]` 访问。例如

```lua
local a = {
    a = 1,
    [";;"] = "hi"
}

print(a[";;"])
```

可以向表中添加元素

```lua
local a = {
    a = 1,
    [";;"] = "hi"
}

a["app"] = "hello"
print(a.app)
```

注意特殊下标的数组中，元素的排列顺序不定。



#### 全局表

存在特殊的表 `_G`，所有全局变量都保存在这个表中

```lua
A = 19
B = 23
C = A + B

print(_G["A"])
print(_G["B"])
print(_G["C"])
```

甚至 `table.insert` 中 `table` 也是全局变量，因此它也在 `_G` 中；由于 `insert` 是 `table` 的成员，因此也可以访问

```lua
print(_G["table"])
print(_G["table"]["insert"])
```



### 布尔型

布尔型 `true, false` 可以直接使用，通过比较运算符获得

```lua
print(1 > 2)
print(1 < 2)
print(1 >= 2)
print(1 <= 2)
print(1 == 2)
print(1 ~= 2)
```

还可以执行逻辑运算

```lua
A = true
B = false
C = A and B
D = A or B
E = not A
```

在 lua 中只有 `nil, false` 代表假，其它值都为真。



逻辑运算将返回**最后一次运算后的值**而非布尔值，因此

- `and` 返回第一个假值，如果没有假值则返回最后一个真值
- `or` 返回第一个真值，如果没有真值则返回最后一个假值

例如

```lua
local a = nil
local b = 10
local c = 0
local d = false

print(a and b)
print(b and a)
print(a or b)
print(b or a)
print(b and c)
print(c and b)
print(b or c)
print(c or b)
print(a and d)
print(d and a)
print(a or d)
print(d or a)
```

通过逻辑运算的短路，可以实现三路运算符

```lua
local a = nil
local b = 0
print(b > 10 and "yes" or "no")
```

在 `b > 10 and "yes"` 中表达式 `b > 10` 为假，因此 `and` 返回得到 `false or "no"`；而 `false` 为真，因此 `or` 返回 `"no"` 。



### 分支

分支语句

```lua
if 1 > 10 then
    print("1 is greater than 10")
elseif 1 < 10 then
    print("1 is less than 10")
else
    print("1 is equal to 10")
end
```



### 循环

循环语句

```lua
for i = 1, 10, 2 do
    print(i)
end
```

在 `for` 循环中不能更改循环变量

```lua
for i = 1, 10, 2 do
    print(i)
    i = 10	-- 这里 i 是额外创建的变量
end
```



使用 `while` 循环

```lua
local n = 10
while n > 0 do
    print(n)
    n = n - 1
    if n == 5 then
        break
    end
end
```



## 进阶语法

### 多文件调用

#### Dofile

使用 `dofile` 调用其它 lua 文件

```lua
dofile("hello.lua")
```

在调用前定义的全局变量可以在调用文件中访问

```lua
-- main.lua
a = 10
dofile("hello.lua")

-- hello.lua
print(a)
```


#### Require

也可以使用 `require` 调用其它 lua 文件

```lua
-- hello.lua
print("Hello, World!")

-- main.lua
require "hello"
```

在调用前定义的全局变量可以在调用文件中访问

```lua
-- main.lua
a = 10
require "hello"

-- hello.lua
print(a)
```



#### 返回值

使用 `require` 调用的 `lua` 文件可以有返回值

```lua
-- hello.lua
print("Hello, World!")
return "done"

-- main.lua
local r = require("hello")
print(r)
```

这里 `require` 可以看作是一个函数，并且它只会运行一次引入的文件，获得第一次运行的返回值

```lua
-- hello.lua
_G.count = _G.count + 1
print("Hello, World!")
return _G.count

-- main.lua
-- 创建全局变量
_G.count = 1

local r = require("hello")
r = require("hello")
r = require("hello")
r = require("hello")
print(r)
print(_G.count)
```



#### 路径

将会从 `package.path` 中保存的路径中查找 `require` 的文件。可以添加新的路径

```lua
package.path = package.path..";./test/?.lua"
print(package.path)
```



#### 返回表

通常在制作模块时，首先定义一个空的表，然后向表中添加方法，最后返回

```lua
-- hello.lua
local hello = {}

-- 在 hello 表中定义 say 方法
function hello.say()
    print("Hello, world!")
end

return hello
```

然后可以在其它位置获得模块并调用方法

```lua
local hello = require "hello"
hello.say()
```



### 迭代器

#### 基本概念

在 `for` 语句中保存了迭代函数、状态常量、控制变量。迭代器初始化时将状态常量和控制变量返回，作为迭代函数的参数传入，将迭代函数的返回值作为迭代结果。共有两种迭代器

- 无状态迭代器
- 多状态迭代器

无状态迭代器不保留任何状态，避免创建新闭包的开销。例如

```lua
-- 迭代函数
function iter(a, i)
    i = i + 1
    local v = a[i]
    if v then
        return i, v
    end
end

-- 迭代器
function mpair(a)
	-- 迭代函数、状态常量、控制变量
    return iter, a, 0
end

arr = {"Hello", "World", "Lua"}

for i, v in mpair(arr) do
    print(i, v)
end
```

其中状态常量是被迭代的容器，控制变量在每次迭代后增加，当迭代函数遇到 `nil` 时迭代结束。



多状态迭代器需要保存多个状态信息，例如使用闭包

```lua
function walk_it(collection)
    -- index 保存索引信息
    local index = 0
    local count = #collection
    -- 闭包函数
    return function()
        index = index + 1
        if index <= count then
            return collection[index]
        end
    end
end

arr = {"apple", "banana", "orange"}

for fruit in walk_it(arr) do
    print(fruit)
end
```



#### 迭代表

一般的迭代表的方法为

```lua
local t = {1, 2, 3, 4, 5}

for i=1, #t do
    print(i, t[i])
end
```

而 lua 提供了专用的迭代器

```lua
local t = {1, 2, 3, 4, 5}

for i, j in ipairs(t) do
    print(i, j)
end
```

但是它只能迭代数字下标的表。使用 `pairs` 迭代特殊下标的表

```lua
local t = {
    [1] = "hello",
    [2] = "world",
    [3] = "!",
    ["hi"] = "there"
}

for i, j in pairs(t) do
    print(i, j)
end
```

这里 `pairs` 调用了 `next` 函数获得给定下标的下一个元素对

```lua
local t = {
    a = 1,
    b = 2,
    c = 3
}

for k, v in pairs(t) do
    print(k, v)
end

print("----------------")
print(next(t))
print(next(t, "a"))
print(next(t, "b"))
print(next(t, "c"))
```



### 正则

#### 查找

使用 `find` 进行普通查找；使用 `match` 进行正则查找

```lua
local s = "abcd1234avcca"

print(string.find(s, "123"))    -- 查找子串 "123" 在 s 中的位置，返回 nil 表示未找到
print(string.find(s, "123", 5)) -- 从 s 的第 5 个字符开始查找 "123"，返回 6 表示找到
print(string.match(s, "a.*c"))  -- 匹配 s 中以 "a" 开始，以 "c" 结束的子串，返回 "abcd1234avcca"
print(string.match(s, "d%d"))   -- 匹配 s 中以 "d" 开始，第二个字符为数字的子串，返回 "d1"
```

在 lua 中 `%` 表示转义，例如

```lua
string.match(s, "%.")	-- 匹配 .
```

匹配规则

| 模式   | 作用          | 模式    | 作用          |
| :--- | :---------- | :---- | :---------- |
| `.`  | 一个字符        | `%a`  | 字母          |
| `%c` | 控制字符        | `%d`  | 数字          |
| `%g` | 除空白以外的可打印字符 | `%l`  | 小写字母        |
| `%p` | 标点符号        | `%s`  | 空白字符        |
| `%u` | 大写字母        | `%w`  | 字母和数字       |
| `%x` | 16 进制数字符号   | `%`   | 转义字符        |
| `[]` | 匹配其中的一个字符   | `[^]` | 匹配不在其中的一个字符 |
| `+`  | 匹配多次        |       |             |

如果希望匹配到满足模式的所有内容，可以使用

```lua
local s = "abcd1234adsd"

-- 依次向后获得匹配到的内容
for w in string.gmatch(s, "%w+") do
    print(w)
end
```



#### 替换

使用 `gsub` 替换匹配结果

```lua
local s = "abcd1234adsd"
print(string.gsub(s, "%d", "x"))
```



### 元表

#### 元方法

可以为表指定方法。例如

```lua
local t = { a = 1 }
local mt = {
    -- 元方法：重载 + 操作符
    __add = function(a, b)
        return a.a + b
    end
}

-- 设置 mt 为 t 的元表
setmetatable(t, mt)

print(t + 1)
```

为普通的表指定一个包含函数的表作为元表，即可重载操作符。



再例如重载下标操作

```lua
local t = { a = 1 }
local mt = {
    __index = function(t, k)
        return 10
    end,
}
setmetatable(t, mt)

print(t["abc"])	-- 得到 10
```

当 lua 发现 `t` 没有 `abc` 元素时，就会检查元方法 `__index` 进行调用。甚至可以将其设为一个表

```lua
local t = { a = 1 }
local mt = {
    __index = {
        abc = 123,
        def = 456
    }
}
setmetatable(t, mt)

print(t["abc"])	-- 得到 123
```

由于 `t` 没有 `abc` 元素，因此查找到 `__index` 中的 `abc` 对应的值。



#### 隐式参数

如果表中的方法的第一个参数是表自身，则可以将其省略，改为使用 `:` 调用

```lua
local t = {
    a = 1,
    add = function(t, x)
        t.a = t.a + x
    end
}

t.add(t, 1)
t:add(1)
```



#### 实现类

借助元方法和隐式参数，可以实现面向对象编程。例如

```lua
-- Bag 类
Bag = {}

Bagmt = {
    put = function(self, item)
        table.insert(self.items, item)
    end
}
Bagmt["__index"] = Bagmt

function Bag.new()
    local t = {
        -- 用于保存内容
        items = {}
    }
    -- 将 Bagmt 作为 t 的元表
    setmetatable(t, Bagmt)
    return t
end
```

现在通过 `Bag.new()` 可以创建一个 `Bag` 类，这个类包括一个 `items` 和 `Bagmt` 元表。并且可以调用 `Bagmt` 中的方法

```lua
local b = Bag.new()
b:put("123")
```

首先 lua 查找 `b` 的 `put` 字段，没有找到，因此会在 `Bagmt` 元表中查找；接着调用 `put(b, "123")`，于是 `b.items` 被插入新的项。



类似地添加更多方法

```lua
Bag = {}

Bagmt = {
    put = function(self, item)
        table.insert(self.items, item)
    end,
    take = function(self, i)
        return table.remove(self.items, i)
    end,
    list = function(self)
        return table.concat(self.items, ", ")
    end,
    clear = function(self)
        self.items = {}
    end
}
Bagmt["__index"] = Bagmt

function Bag.new()
    local t = {
        -- 用于保存内容
        items = {}
    }
    -- 将 Bagmt 作为 t 的元表
    setmetatable(t, Bagmt)
    return t
end

local b = Bag.new()
b:put("apple")
b:put("banana")
b:put("orange")
print(b:list())  -- apple, banana, orange
print(b:take(2)) -- banana
print(b:list())  -- apple, orange
b:clear()
print(b:list())  -- (empty)
```



#### 实现继承

指定一个新的元表，保存之前元表中的所有方法，同时添加新的方法，即可得到继承类

```lua
ColorBag = {}
ColorBagmt = {
    get_color = function(self)
        print("This bag is", self.color)
    end
}
-- 继承 Bagmt 中的方法
for k, v in pairs(Bagmt) do
    ColorBagmt[k] = v
end
ColorBagmt["__index"] = ColorBagmt

function ColorBag.new(color)
    local t = {
        -- 用于保存内容
        items = {},
        -- 颜色
        color = color
    }
    -- 将 ColorBagmt 作为 t 的元表
    setmetatable(t, ColorBagmt)
    return t
end

colorBag = ColorBag.new("red")
colorBag:get_color() -- This bag is red
```



### 协程

简单的协程示例

```lua
-- 创建协程
local co = coroutine.create(function()
    for i = 1, 10 do
        print(i)
        -- 立即返回，暂停协程
        coroutine.yield()
    end
end)

-- 启动协程
coroutine.resume(co)
coroutine.resume(co)
coroutine.resume(co)
```

协程可以携带返回值，也可以传入值

```lua
-- 创建协程
local co = coroutine.create(function()
    print("Hello, world!")
    -- 返回 true 1 2
    local a, b, c = coroutine.yield(1, 2)
    -- 传入 3 4 5
    print(a, b, c)
end)

print(coroutine.resume(co))     -- true    1       2
coroutine.resume(co, 3, 4, 5)   -- 3       4       5
```

获得协程状态

```lua
local co = coroutine.create(function()
    print("Hello, world!")
    local a, b, c = coroutine.yield(1, 2)
    print(a, b, c)
end)

print(coroutine.resume(co))
print(coroutine.status(co))
```



### 文件读写

#### 简单模式

简单模式通过 `io` 控制

```lua
-- 读文件
file = io.open('hello.lua', 'r')
-- 指定当前读取文件
io.input(file)
for line in io.lines() do
    print(line)
end
io.close(file)

-- 写文件
file = io.open('hello.lua', 'w')
-- 指定当前输出文件
io.output(file)
io.write('Hello, Lua!\n')
io.close(file)
```



#### 完全模式

完全模式通过文件对象自身控制

```lua
-- 读文件
file = io.open('hello.lua', 'r')
for line in file:lines() do
    print(line)
end
file:close()

-- 写文件
file = io.open('hello.lua', 'w')
file:write('Hello, Lua!\n')
file:close()
```



### 二进制数据

根据字节序，使用 `pack` 将字符串保存为大小端格式的二进制数据

```lua
-- 小端
local data = string.pack("<L", 1)
print("len:", #data)
print(data:byte(1))
print(data:byte(2))
print(data:byte(3))
print(data:byte(4))

-- 大端
local data = string.pack(">L", 1)
print("len:", #data)
print(data:byte(1))
print(data:byte(2))
print(data:byte(3))
print(data:byte(4))
```

使用 `unpack` 解析数据

```lua
local r1, r2 = string.unpack("<LL", data..data)
print(r1, r2)
```

其中 `L` 表示 `unsigned long`，更多类型参考官方文档。



## 联合编程

### 基本介绍

Lua 的 C 语言 API 是轻量高效的。Lua 不分配任何全局内存，所有状态存放在 `lua_State` 中。

> 在 `lua_State` 对象附近放置一个互斥锁可以确保任何 lua 实例都线程安全。




### 下载源码

首先下载特定版本的源码，放到固定目录下。

```embed
title: "Lua: version history"
image: "https://www.lua.org/images/logo.gif"
description: "Here is a chronology of the versions of Lua. The evolution of Lua is documented in a paper presented at HOPL III, the Third ACM SIGPLAN History of Programming Languages Conference, in 2007. The source code and documentation for all releases of Lua are available in the download area."
url: "https://www.lua.org/versions.html#5.4"
```

可以建立项目编译源码生成库，但是为了方便调试，我们直接将源码编译到项目中。



### 简单示例
#### 项目框架

创建 cmake 文件

```cmake
cmake_minimum_required(VERSION 3.5...3.29)
project(main)

set(CMAKE_CXX_STANDARD 20)

# 获得 lua 源文件，排除 lua.c 和 luac.c（其中包含了 main 函数）
set(SRC)
file(GLOB LUA_SRC "D:/lib/lua-5.4.7/src/*.c")
foreach(file ${LUA_SRC})
  if(file MATCHES "D:/lib/lua-5.4.7/src/lua[c]?.c")
    continue()
  endif()
  list(APPEND SRC ${file})
endforeach()

include_directories("D:/lib/lua-5.4.7/src")

add_executable(main main.cpp ${SRC})
```

然后创建测试源码

```cpp
extern "C"
{
#include <lauxlib.h>
#include <lua.h>
#include <lualib.h>
}

#include <stdlib.h>

// 自定义内存分配器
static void *l_alloc(void *ud, void *ptr, size_t osize, size_t nsize)
{
    if (nsize == 0)
    {
        free(ptr);
        return NULL;
    }
    else
        return realloc(ptr, nsize);
}

int main()
{
    // 初始化 lua 虚拟机
    lua_State *L = lua_newstate(l_alloc, NULL);

    // 打开基础库
    luaopen_base(L);

    // 加载脚本
    luaL_loadfile(L, "../../main.lua");

    // 执行脚本
    lua_pcall(L, 0, 0, 0);
    return 0;
}
```

需要给定脚本

```lua
print("Hello World!")
```



#### 导入库

如果需要使用 `string, table` 等 `lua` 库，就需要导入对应的库

```cpp
luaopen_string(L);
luaopen_table(L);
```

不过这样可能会遗漏某些库。因此可以将它们全部导入，同时开启错误捕获来查看错误信息

```cpp
extern "C"
{
#include <lauxlib.h>
#include <lua.h>
#include <lualib.h>
}

#include <stdlib.h>

// 自定义内存分配器
static void *l_alloc(void *ud, void *ptr, size_t osize, size_t nsize)
{
    if (nsize == 0)
    {
        free(ptr);
        return NULL;
    }
    else
        return realloc(ptr, nsize);
}

int main()
{
    // 初始化 lua 虚拟机
    lua_State *L = lua_newstate(l_alloc, NULL);

    // 打开基础库
    luaL_openlibs(L); // 一次性打开所有标准库

    // 加载脚本
    if (luaL_loadfile(L, "../../main.lua") || lua_pcall(L, 0, 0, 0))
    {
        // 捕获并打印错误
        fprintf(stderr, "Error: %s\n", lua_tostring(L, -1));
        return 1;
    }

    // 关闭 Lua 状态机
    lua_close(L);
    return 0;
}
```

执行脚本

```lua
print("Hello, world!")
local s = "Hello, Lua!"
local l = string.len(s)
print(l)

local t = {}
table.insert(t, 1)
table.insert(t, 2)
table.insert(t, 3)
print(table.concat(t, " "))
```



### Lua 调用 C++

Lua 与 C++ 交互流程如下

![[image-20240819153748895.png|500]]



#### 函数调用

首先定义一个 lua 函数，然后通过 `lua_register` 注册函数

```cpp
extern "C"
{
#include <lauxlib.h>
#include <lua.h>
#include <lualib.h>
}

#include <stdlib.h>

// 定义一个 lua 函数
int test(lua_State *L)
{
    printf("Hello, world!\n");
    return 0;
}

// 自定义内存分配器
static void *l_alloc(void *ud, void *ptr, size_t osize, size_t nsize)
{
    if (nsize == 0)
    {
        free(ptr);
        return NULL;
    }
    else
        return realloc(ptr, nsize);
}

int main()
{
    // 初始化 lua 虚拟机
    lua_State *L = lua_newstate(l_alloc, NULL);

    // 打开基础库
    luaL_openlibs(L); // 一次性打开所有标准库

    // 注册函数
    lua_register(L, "test", test);

    // 加载脚本
    if (luaL_loadfile(L, "../../main.lua") || lua_pcall(L, 0, 0, 0))
    {
        // 捕获并打印错误
        fprintf(stderr, "Error: %s\n", lua_tostring(L, -1));
        return 1;
    }

    // 关闭 Lua 状态机
    lua_close(L);
    return 0;
}
```

在 `main.lua` 中调用

```lua
test()
```

编译运行 C++ 项目，可以得到函数的输出。



#### 普通参数

当函数传入参数时

```lua
test("lua string", 12)
```

从栈底开始依次获得参数

```cpp
int test(lua_State *L)
{
    // 从栈底开始获得参数
    size_t len;
    const char *name = lua_tolstring(L, 1, &len);
    int age = lua_tonumber(L, 2);

    printf("name: %s, age: %d\n", name, age);
    return 0;
}
```



#### 数组参数

对于数组类型参数

```lua
local arr = { "hello", "world", "lua" }
test(arr)
```

通过下面流程获得

```cpp
int test(lua_State *L)
{
    luaL_checktype(L, 1, LUA_TTABLE);   // 确保栈上的第一个参数是一个表
    lua_Integer len = lua_rawlen(L, 1); // 获取表的长度，不调用 __len 方法
    // lua_Integer len = luaL_len(L, 1);  // 获取表的长度，调用 __len 方法
    for (int i = 1; i <= len; i++)
    {
        lua_rawgeti(L, 1, i);                       // 取出表中的第 i 个元素
        printf("%d: %s\n", i, lua_tostring(L, -1)); // 打印元素的值
    }
    return 0;
}
```

另一种访问方式是

```cpp
int test(lua_State *L)
{
    luaL_checktype(L, 1, LUA_TTABLE);   // 确保栈上的第一个参数是一个表
    lua_Integer len = lua_rawlen(L, 1); // 获取表的长度，不调用 __len 方法
    
    for (int i = 1; i <= len; i++)
    {
        lua_pushnumber(L, i);	// 将 `i` 压入栈顶
        lua_gettable(L, 1);		// 从栈底（1）获得表 `arr`，从栈顶位置弹出索引 `i`，然后将 `arr[i]` 压入栈顶
        size_t len;
        const char *str = lua_tolstring(L, -1, &len);	// 从栈顶（-1）获得值并转换为字符串
        lua_pop(L, 1);									// 从栈顶弹出 1 个值（`arr[i]`），还原栈
		printf("%d: %s\n", i, str);
    }
    return 0;
}
```



#### 键值对

对于保存键值对的表

```lua
local tab = {name = "John", age = 30, city = "New York"}
test(tab)
```

通过下面流程获得

```cpp
int test(lua_State *L)
{
    luaL_checktype(L, 1, LUA_TTABLE); // 确保栈上的第一个参数是一个表
    lua_pushnil(L);                   // 准备一个 nil 值，作为遍历的初始值
    while (lua_next(L, 1))
    {
        // 遍历表中的每一个 key-value 对
        if (lua_isstring(L, -2))
        {
            // 如果 key 是字符串，则打印 key-value 对
            printf("%s - %s\n", lua_tostring(L, -2), lua_tostring(L, -1));
        }
        lua_pop(L, 1);
    }
    return 0;
}
```

其中 `lua_next` 会弹出栈顶的值，然后依次推入 `key, value`，因此需要最后弹出一个值来保持平衡。



如果希望通过键获得值，使用

```cpp
int test(lua_State *L)
{
    luaL_checktype(L, 1, LUA_TTABLE); // 确保栈上的第一个参数是一个表
    lua_getfield(L, 1, "name");       // 获取表中名为 "name" 的值
    printf("Hello, %s!\n", lua_tostring(L, -1));
    return 0;
}
```

其中第二个参数对应表的位置。



#### 类型检查

有两种常用的类型检查

```cpp
// 检查第一个参数为表，如果不是，则中断执行
luaL_checktype(L, 1, LUA_TTABLE);

// 检查第一个参数为表，如果不是，进行处理
if (lua_type(L, 1) != LUA_TTABLE)
    return luaL_argerror(L, 1, "the first argument must be a table");
```



#### 返回值

如果要返回基本类型，首先栈顶推入值，然后指定 `return` 数量

```cpp
int test(lua_State *L)
{
    lua_pushstring(L, "return hello");
    return 1;
}
```

其中 `return 1` 表示弹出 1 个栈顶元素作为返回值。因此

```lua
local r = test()
print(r)
```

能够获得返回的字符串。



要返回表，首先创建表并推入栈顶，然后设置元素

```cpp
int test(lua_State *L)
{
    lua_newtable(L);           // 创建一个空表
    lua_pushstring(L, "name"); // key
    lua_pushstring(L, "John"); // value
    lua_settable(L, -3);       // 弹出 key 和 value 并设置到表中
    lua_pushstring(L, "age");  // key
    lua_pushinteger(L, 25);    // value
    lua_settable(L, -3);       // 弹出 key 和 value 并设置到表中
    return 1;
}
```

在 lua 中可以获得表

```lua
local r = test()
print(r["name"])
print(r["age"])
```



### C++ 调用 Lua

#### 全局变量

在 lua 中设置全局变量

```lua
width = 100
print(name)
```

加载脚本前，可以设置全局变量；加载脚本后，可以获得全局变量

```cpp
extern "C"
{
#include <lauxlib.h>
#include <lua.h>
#include <lualib.h>
}

#include <stdlib.h>

// 自定义内存分配器
static void *l_alloc(void *ud, void *ptr, size_t osize, size_t nsize)
{
    if (nsize == 0)
    {
        free(ptr);
        return NULL;
    }
    else
        return realloc(ptr, nsize);
}

int main()
{
    // 初始化 lua 虚拟机
    lua_State *L = lua_newstate(l_alloc, NULL);

    // 打开基础库
    luaL_openlibs(L); // 一次性打开所有标准库

    // 设置全局变量 name
    lua_pushstring(L, "John");
    lua_setglobal(L, "name");

    // 加载脚本
    if (luaL_loadfile(L, "../../main.lua") || lua_pcall(L, 0, 0, 0))
    {
        // 捕获并打印错误
        fprintf(stderr, "Error: %s\n", lua_tostring(L, -1));
        return 1;
    }

    // 获得全局变量 width 的值
    lua_getglobal(L, "width");
    int width = lua_tointeger(L, -1);
    lua_pop(L, 1);
    printf("width = %d\n", width);

    // 关闭 Lua 状态机
    lua_close(L);
    return 0;
}
```



#### 访问表

对于全局表

```lua
tab = { name = "John", age = 30, city = "New York" }
```

通过下面流程访问

```cpp
// 获得全局表
lua_getglobal(L, "tab");

lua_getfield(L, -1, "name");
printf("name: %s\n", lua_tostring(L, -1));
lua_pop(L, 1); // 弹出 name

lua_getfield(L, -1, "age");
printf("age: %d\n", lua_tointeger(L, -1));
lua_pop(L, 1); // 弹出 age

lua_pop(L, 1); // 弹出 tab
```



#### 传递表

创建表，然后将其传递为全局变量

```lua
lua_newtable(L);           // 创建一个空表

lua_pushstring(L, "name"); // key
lua_pushstring(L, "John"); // value
lua_settable(L, -3);       // 弹出 key 和 value 并设置到表中

lua_pushstring(L, "age");  // key
lua_pushinteger(L, 25);    // value
lua_settable(L, -3);       // 弹出 key 和 value 并设置到表中

lua_setglobal(L, "person"); // 设置全局变量 person
```



#### 函数调用

在 lua 中定义函数

```lua
function event()    
    print("Hello, world!")
end
```

获得函数并调用

```cpp
lua_getglobal(L, "event");
lua_pcall(L, 0, 0, 0);
```

增加错误判断

```cpp
lua_getglobal(L, "event");
if (lua_pcall(L, 0, 0, 0))
{
    fprintf(stderr, "Error: %s\n", lua_tostring(L, -1)); // 从栈顶获得错误信息
    lua_pop(L, 1);                                       // 弹出错误信息
}
printf("Top is %d\n", lua_gettop(L)); // 打印栈顶元素的索引，用于检查栈是否正确弹出
```



对于多参数且有返回值的情况

```lua
function event(x, y, z)    
    print(x, y, z)
    return "event"
end
```

需要传入参数和返回值数

```cpp
lua_getglobal(L, "event");

// 推入 3 个参数
lua_pushstring(L, "hello");
lua_pushnumber(L, 123);
lua_pushboolean(L, 1);

// 调用时指定 3 个参数和一个返回值
lua_pcall(L, 3, 1, 0);

// 栈顶获取返回值
printf("%s\n", lua_tostring(L, -1));
```



#### 错误处理

构造异常处理函数，接受一个异常信息参数

```lua
-- event 函数不存在
function event1(e)    
    print(e)
    return "event"
end

-- 异常处理函数
function ferror(e)
    print("ferror: ", e)
    return "lua change error"
end
```

首先获得异常处理函数，调用 `event` 时传入

```cpp
// 记录 ferror 函数的栈索引
int top = lua_gettop(L);
lua_getglobal(L, "ferror");
top++;

lua_getglobal(L, "event");
lua_pushstring(L, "hello");

// 使用 top 索引处的 ferror 函数作为异常处理函数
if (lua_pcall(L, 1, 1, top))
{
    printf("call error: %s\n", lua_tostring(L, -1));
    lua_pop(L, 1);
}
else
{
    printf("lua return: %s\n", lua_tostring(L, -1));
    lua_pop(L, 1);
}

// 弹出 ferror
lua_pop(L, 1);
```



