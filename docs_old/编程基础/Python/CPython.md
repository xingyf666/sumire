# CPython

## 基本介绍

CPython 将 C 的 API 暴露给其它软件，方便做 Python 不方便做的事。C-API 是做二进制扩展的关键。C 扩展的特点是，当它们不需要回调到解释器运行时，它们的操作可以释放掉 GIL 。



二进制扩展

- 加速器模块：可完全独立的模块，为提高性能而存在
- 包装器模块：暴露 C 接口给 Python 代码
- 底层入口模块：操作系统级别的 API 模块



### C++ 调用 Python
#### 简单示例

创建 `CMakelists.txt` 文件

```cmake
cmake_minimum_required(VERSION 3.27)

project(main)

set(CMAKE_CXX_STANDARD 20)

include_directories("D:/Python310/include")
link_libraries("D:/Python310/libs/python310.lib")

add_executable(main main.cpp)
```

然后指定环境变量、初始化，就可以调用 API

```cpp
#include <Python.h>
#include <iostream>

int main()
{
    // 指定 Python 环境变量
    _putenv("PYTHONHOME=D:/Python310/");
    _putenv("PYTHONPATH=D:/Python310/libs/");

    Py_Initialize();
    PyRun_SimpleString("print('Hello, Python!')");
    Py_Finalize();

    return 0;
}
```

如果在 Debug 下编译，则需要修改 cmake 文件

```cpp
link_libraries("D:/Python310/libs/python310_d.lib")
```

同时将 `python310.lib` 复制一份，重命名。



我们可以输出一个 python 模块的所有属性

```cpp
#include <Python.h>
#include <iostream>

int main()
{
	// 指定 Python 环境变量
    _putenv("PYTHONHOME=D:/Python310/");
    _putenv("PYTHONPATH=D:/Python310/libs/");

    // 初始化 Python 解释器
    Py_Initialize();

    // 导入想查看的 Python 模块
    PyObject *pModule = PyImport_ImportModule("math");
    if (pModule == nullptr)
    {
        PyErr_Print();
        return 1;
    }

    // 获取模块的属性列表
    PyObject *pDir = PyObject_Dir(pModule);
    if (pDir == nullptr)
    {
        PyErr_Print();
        return 1;
    }

    // 确保返回的是一个 Python 列表
    if (PyList_Check(pDir))
    {
        Py_ssize_t num_attributes = PyList_Size(pDir);
        for (Py_ssize_t i = 0; i < num_attributes; ++i)
        {
            PyObject *pAttrName = PyList_GetItem(pDir, i);
            if (PyUnicode_Check(pAttrName))
            {
                // 将属性名称从 Python 对象转换为 C 字符串
                const char *attr_name = PyUnicode_AsUTF8(pAttrName);
                if (attr_name)
                    std::cout << attr_name << std::endl;
            }
        }
    }

    // 关闭 Python 解释器
    Py_Finalize();

    return 0;
}
```



#### 无参函数

在一个 `test1.py` 文件中定义

```python
def say():
	print("Hello, world!")
```

然后在 C++ 中调用

```cpp
#include <Python.h>
#include <iostream>

int main()
{
    // 指定 Python 环境变量
    _putenv("PYTHONHOME=D:/Python310/");
    _putenv("PYTHONPATH=D:/Python310/libs/");

    Py_Initialize();

    // 初始化文件路径，可以找到 py 文件
    PyRun_SimpleString("import sys");
    PyRun_SimpleString("sys.path.append('.')");

    // 导入 test.py
    PyObject* pModule = PyImport_ImportModule("test1");

	// 获得属性（函数）
    PyObject* func = PyObject_GetAttrString(pModule, "say");

	// 检查函数，调用函数
    if (func && PyCallable_Check(func))
        PyObject_CallObject(func, nullptr);

    Py_Finalize();

    return 0;
}
```

注意使用的模块名不要与其它可能的模块重名，例如 `test.py` 可能是一个预定义的模块，如果这里创建 `test.py` 就可能无法导入正确的文件。



#### 有参函数

给出有参函数

```python
def add(a, b):
	c = a + b
	print("Sum is ", c)
	return c
```

参数需要通过 Python 元组传入

```cpp
#include <Python.h>
#include <iostream>

int main()
{
    // 指定 Python 环境变量
    _putenv("PYTHONHOME=D:/Python310/");
    _putenv("PYTHONPATH=D:/Python310/libs/");

    Py_Initialize();

    // 初始化文件路径，可以找到 py 文件
    PyRun_SimpleString("import sys");
    PyRun_SimpleString("sys.path.append('.')");

    // 导入 test.py
    PyObject *pModule = PyImport_ImportModule("test1");
    PyObject *func = PyObject_GetAttrString(pModule, "add");
    if (func && PyCallable_Check(func))
    {
        // 创建元组
        PyObject *pArgs = PyTuple_New(2);

		// 传入 int 类型 ("i") 参数
        PyTuple_SetItem(pArgs, 0, Py_BuildValue("i", 1));
        PyTuple_SetItem(pArgs, 1, Py_BuildValue("i", 2));

        PyObject *pRet = PyObject_CallObject(func, pArgs);

        // 解析返回值为 int 类型
        int result;
        PyArg_Parse(pRet, "i", &result);
        std::cout << "1 + 2 = " << result << std::endl;
    }

    Py_Finalize();

    return 0;
}
```



#### 对象

给定类及其属性

```cpp
class Person:
def __init__(self, name, age):
    self.name = name
    self.age = age

def foo(self):
    print("Hello, my name is " + self.name + " and I am " + str(self.age) + " years old.")
```

需要首先构造类实例，然后调用方法（也可以获得实例属性进行调用）

```cpp
#include <Python.h>
#include <iostream>

int main()
{
    // 指定 Python 环境变量
    _putenv("PYTHONHOME=D:/Python310/");
    _putenv("PYTHONPATH=D:/Python310/libs/");

    Py_Initialize();

    // 初始化文件路径，可以找到 py 文件
    PyRun_SimpleString("import sys");
    PyRun_SimpleString("sys.path.append('.')");

    // 导入 test.py
    PyObject *pModule = PyImport_ImportModule("test1");
    PyObject *cls = PyObject_GetAttrString(pModule, "Person");
    if (cls)
    {
        // 创建元组
        PyObject *pArgs = PyTuple_New(2);

        // 传入 string, int 类型参数
        PyTuple_SetItem(pArgs, 0, Py_BuildValue("s", "Alice"));
        PyTuple_SetItem(pArgs, 1, Py_BuildValue("i", 18));

        // 调用构造函数创建对象
        PyObject *obj = PyEval_CallObject(cls, pArgs);

        // 调用对象方法
        PyObject *pRet = PyObject_CallMethod(obj, "foo", nullptr);
    }

    Py_Finalize();

    return 0;
}
```



#### 引用计数

CPython 默认调用在调试模式下生成的 `_d` 库，其中提供 `Py_DECREF` 宏，减少引用计数，从而释放内存。例如

```cpp
PyObject *obj = PyEval_CallObject(cls, pArgs);
Py_DECREF(obj)
```

如果不执行此操作，则 Python 不会回收对应的内存，可能造成内存泄露。



## Ctypes

### 基本功能

#### 简单示例

创建 `CMakelists.txt` 文件

```cmake
cmake_minimum_required(VERSION 3.27)

project(test LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 20)

add_library(test SHARED main.cpp)
```

测试函数

```cpp
#include <stdio.h>

extern "C" __declspec(dllexport) void TestCtypes()
{
    printf("Hello from Ctypes!\n");
}
```

将编译生成的 dll 文件复制到 python 项目下，执行

```python
from ctypes import *

test = cdll.LoadLibrary("./test.dll")
test.TestCtypes()
```

就可以完成调用。

> >由于 ctypes 在内部调用 `LoadLibrary` 显式链接动态库，不需要 `lib` 文件 [基本 Cpp#链接动态库](基本 Cpp#链接动态库) 。



#### 参数转换

为函数添加不同基本类型的参数

```cpp
extern "C" __declspec(dllexport) void TestCtypes(int x, float y, bool z)
{
    printf("%d %f %d\n", x, y, z);
}
```

只需要使用 `c_` 开头的转换函数就可以实现参数类型转换

```python
test.TestCtypes(101, c_float(3.14), c_bool(True))
```

其中 `int` 类型可以自动转换。



#### 字符串类型

也可以传入不同类型的字符串

```cpp
extern "C" __declspec(dllexport) void TestCtypesString(const char_t *str, int x)
{
    printf("%s, %d\n", str, x);
}

extern "C" __declspec(dllexport) void TestCtypesWString(const wchar_t *str, int x)
{
    printf("%ls, %d\n", str, x);
}
```

注意 C++ 中的宽字符串对应 python 中的字符串，而窄字符串需要通过 `create_string_buffer` 额外创建

```python
str = create_string_buffer(b"Hello, World!")
test.TestCtypesString(str, len(str))

str = "Hello, World!"
test.TestCtypesWString(str, len(str))
```



如果需要在函数内部修改字符串，需要传入足够长度的 buffer

```python
buf = create_string_buffer(100)
test.TestCtypesBuffer(buf, 100)
print(buf.value)
```

对应的 C++ 函数为

```cpp
extern "C" __declspec(dllexport) void TestCtypesBuffer(char *str, int x)
{
    str[0] = 'A';
    str[1] = 'B';
    str[2] = 'C';
    str[3] = '\0';
}
```



#### 调用 C 语言库

对于 VS 标准库，通过 `cdll.msvcrt` 调用标准库

```python
libc = cdll.msvcrt
libc.printf(b"Hello, world!\n")
```

甚至还可以调用 `Win32` 标准库

```python
re = windll.user32.MessageBoxA(0, "窗口内容".encode("gbk"), "请选择".encode("gbk"), 1)
if re == 1:
    windll.user32.MessageBoxW(0, "你选择了确定", "消息框", 0)
else:
    windll.user32.MessageBoxW(0, "你选择了取消", "消息框", 0)
```

注意其中 `A` 后缀对应的窗口函数会根据字符串类型匹配调用函数，而 `W` 后缀对应的窗口函数是宽字符串函数，因此前者需要通过 `encode` 转换编码，后者不需要。



#### 获取返回值

默认返回值均为 `int`，但是也可以修改类型。例如

```cpp
extern "C" __declspec(dllexport) int TestCtypesReturnInt()
{
    return 100;
}

extern "C" __declspec(dllexport) char *TestCtypesReturnString()
{
    return "Hello, World!";
}

extern "C" __declspec(dllexport) wchar_t *TestCtypesReturnWString()
{
    return L"Hello, World!";
}
```

分别调用

```python
test = cdll.LoadLibrary("./test.dll")
print(test.TestCtypesReturnInt())

test.TestCtypesReturnString.restype = c_char_p
print(test.TestCtypesReturnString())

test.TestCtypesReturnWString.restype = c_wchar_p
print(test.TestCtypesReturnWString())
```



#### 指针类型

以指针为参数和返回值

```cpp
extern "C" __declspec(dllexport) int *TestCtypesPointer(float *f)
{
    static int re = 1001;
    *f = 1.1f;
    return &re;
}
```

分别指定参数类型和返回类型

```python
# 参数类型为元组
test.TestCtypesPointer.argtypes = (POINTER(c_float),)
test.TestCtypesPointer.restype = POINTER(c_int)

f = c_float(3.14)
re = test.TestCtypesPointer(f)
print(f.value, re.contents.value)
```

其中 `re.contents` 就是指针指向的内容。



#### 数组类型

在 ctypes 中，数组长度是固定的，需要通过数组元素类型和长度来构造数组类型

```python
arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# 构造长度为 10 的数组类型
ArrType = c_int * len(arr)
carr = ArrType(*arr)

test.TestCtypesArray.argtypes = (ArrType, c_int)
test.TestCtypesArray(carr, len(arr))
```

对应的 C++ 函数为

```cpp
extern "C" __declspec(dllexport) void TestCtypesArray(int *arr, int size)
{
    for (int i = 0; i < size; i++)
        printf("%d ", arr[i]);
}
```



#### 结构体类型

定义结构体并作为函数参数和返回值

```cpp
struct Pos
{
    int x, y;
};

extern "C" __declspec(dllexport) Pos TestCtypesStruct(Pos pos, Pos *ppos, Pos *posArr, int size)
{
    printf("x = %d, y = %d\n", pos.x, pos.y);
    printf("x = %d, y = %d\n", ppos->x, ppos->y);
    for (int i = 0; i < size; i++)
        printf("x = %d, y = %d\n", posArr[i].x, posArr[i].y);
    return pos;
}
```

在 python 中定义相同结构的类，并作为参数传入，最后获得返回值

```python
class Pos(Structure):
    _fields_ = [("x", c_int), ("y", c_int)]

pos1 = Pos(10, 20)
pos2 = Pos(30, 40)
pos3 = [Pos(50, 60), Pos(70, 80)]
PosType = Pos * len(pos3)

test.TestCtypesStruct.argtypes = (Pos, POINTER(Pos), PosType, c_int)
test.TestCtypesStruct.restype = Pos
pos4 = test.TestCtypesStruct(pos1, byref(pos2), PosType(*pos3), len(pos3))
print(f"pos4: x={pos4.x}, y={pos4.y}")
```

使用 `byref` 获得变量地址/指针。



#### 回调函数

调用 C 语言标准库中的快速排序算法，要求传入一个比较函数作为回调函数。使用 `CFUNCTYPE` 定义比较函数类型，其中第一个参数是返回类型，后面的参数是函数参数类型

```cpp
lib = cdll.msvcrt

CMP_FUNC = CFUNCTYPE(c_int, POINTER(c_int), POINTER(c_int))

# 注意参数是指针，在比较时需要获得指向的内容
def cmp(a, b):
    return a.contents.value - b.contents.value

# 定义比较函数
cmp_func = CMP_FUNC(cmp)

data = (c_int * 5)(5, 2, 9, 1, 3)
lib.qsort(data, len(data), sizeof(c_int), cmp_func)
for i in data:
    print(i, end=' ')
```



可以自定义回调函数并在 C 语言中调用

```cpp
typedef void (*Callback)(int);

extern "C" __declspec(dllexport) void TestCtypesCallback(int *arr, int size, Callback callback)
{
    for (int i = 0; i < size; i++)
        callback(arr[i]);
}
```

同时在 python 中定义回调函数类型，作为参数传入

```python
FUNC = CFUNCTYPE(None, c_int)

def callback(n):
    print(n)

arr = [1, 2, 3, 4, 5]
ArrayType = c_int * len(arr)
carr = ArrayType(*arr)

test.TestCtypesCallback.argtypes = (ArrayType, c_int, FUNC)
test.TestCtypesCallback(carr, len(arr), FUNC(callback))
```



### 三维引擎

#### 鬼火引擎

这是一个比较古老的三维引擎库，比较适合用来做 demo 。



#### 创建窗口

首先提供引擎封装类

```cpp
#include <iostream>
#include <irrlicht.h>

struct PyIrrlicht
{
  private:
    PyIrrlicht() = default;
    irr::IrrlichtDevice *device = nullptr;

  public:
    static PyIrrlicht *Instance()
    {
        static PyIrrlicht instance;
        return &instance;
    }

    bool InitDevice(int width, int height)
    {
        std::cout << "InitDevice(" << width << ", " << height << ")" << std::endl;
        device = irr::createDevice(irr::video::EDT_OPENGL, irr::core::dimension2d<irr::u32>(width, height));
        if (!device)
        {
            std::cerr << "Failed to create Irrlicht device" << std::endl;
            return false;
        }

        if (!device->getSceneManager()->addCameraSceneNodeFPS())
        {
            std::cerr << "Failed to add FPS camera" << std::endl;
            return false;
        }
        std::cout << "Device initialized" << std::endl;
        return true;
    }
};

extern "C" __declspec(dllexport) bool InitDevice(int width, int height)
{
    return PyIrrlicht::Instance()->InitDevice(width, height);
}
```

对应的 cmake 文件为

```cmake
cmake_minimum_required(VERSION 3.10)

project(test LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 11)

add_library(${PROJECT_NAME} SHARED main.cpp)

target_include_directories(${PROJECT_NAME} PRIVATE "D:/lib/irrlicht-1.8.5/include")
target_link_libraries(${PROJECT_NAME} PRIVATE "D:/lib/irrlicht-1.8.5/lib/Win64-visualStudio/irrlicht.lib")

set(BINARY_DIR ${PROJECT_BINARY_DIR})
set(OUTPUT_DIR
    "$<$<CONFIG:Debug>:"
    "${BINARY_DIR}/Debug"
    ">"
    "$<$<CONFIG:Release>:"
    "${BINARY_DIR}/Release"
    ">"
    "$<$<CONFIG:RelWithDebInfo>:"
    "${BINARY_DIR}/RelWithDebInfo"
    ">")
string(REPLACE ";" " " OUTPUT_DIR ${OUTPUT_DIR})

set(DEPS_DLL "D:/lib/irrlicht-1.8.5/bin/Win64-visualStudio/irrlicht.dll")

add_custom_command(
  TARGET ${PROJECT_NAME}
  PRE_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy ${DEPS_DLL} ${OUTPUT_DIR})
```

编译后，将动态库复制到 python 项目下，然后执行

```python
from ctypes import *

test = cdll.LoadLibrary("./test.dll")
test.InitDevice(800, 600)
```



## Pybind

### 编译配置

下载源码后，构建编译项目，指定安装目录后，执行 `INSTALL` 项目，将得到的 `include, share` 目录放到固定位置。

![](image-20240813211851510.png)

这里我们使用 `python3.11` 进行编译安装，因此后续也要使用对应的版本编译代码。



### 构建方法

使用 cmake 构建项目。Pybind 提供一个 cmake 函数

```cmake
pybind11_add_module(<name> [MODULE | SHARED] [EXCLUDE_FROM_ALL]
	[NO_EXTRAS] [THIN_LTO] [OPT_SIZE] source1 [source2 ...])
```

它类似于 `add_library`，添加一个库目标，从指定的源文件生成。使用 `SHARED` 创建传统的动态库；默认使用 `MODULE` 确保专用模块创建。



新建 cmake 项目

```cmake
cmake_minimum_required(VERSION 3.27)

project(main)

set(CMAKE_CXX_STANDARD 20)
set(PYBIND11_FINDPYTHON ON)

# 设置查找目录
set(pybind11_DIR "D:/lib/pybind11/share/cmake/pybind11")
find_package(pybind11 CONFIG REQUIRED)

# 连接头文件和库
include_directories("D:/Python311/include" "D:/lib/pybind11/include")
link_libraries("D:/Python311/libs/python311.lib")

# 添加模块
pybind11_add_module(main main.cpp)
```

编译源文件得到 `.pyd` 库文件

```cpp
#include <pybind11/pybind11.h>

int add(int i, int j)
{
    return i + j;
}

// 模块名 example，模块接口 m
PYBIND11_MODULE(example, m)
{
	// 模块说明
    m.doc() = "pybind11 example plugin";

	// 定义并绑定函数
    m.def("add", &add, "A function that adds two numbers");
}
```

将生成的 `main.cp311-win_arm64.pyd` 重命名为 `example.pyd`，移动到指定目录下。然后在 `python3.11` 环境下可以使用

```python
import example

a = example.add(1, 2)
print(a)
```



### 函数绑定
#### 绑定参数

可以为每个参数绑定一个变量名

```cpp
PYBIND11_MODULE(example, m)
{
    m.doc() = "pybind11 example plugin"; // optional module docstring

    // 绑定参数名称
    m.def("add", &add, "A function which adds two numbers",
      pybind11::arg("i"), pybind11::arg("j"));
}
```

然后调用时可以指定参数名

```python
example.add(i=1, j=2)
```



#### 默认参数

如果函数存在默认参数

```cpp
int add(int i = 1, int j = 2) 
{
    return i + j;
}
```

也可以指定

```python
m.def("add", &add, "A function which adds two numbers",
	  pybind11::arg("i") = 1, pybind11::arg("j") = 2);
```



#### 导出变量

通过 `attr` 导出变量和对应的值。如果要导出对象，首先通过 `cast` 转换，然后再通过 `attr` 导出。

```cpp
PYBIND11_MODULE(example, m)
{
    m.attr("the_answer") = 42;
    pybind11::object world = pybind11::cast("World");
    m.attr("what") = world;
}
```



### 类绑定
#### 结构体

使用 `class_` 模板来绑定类及其成员和成员函数

```cpp
#include <pybind11/pybind11.h>

struct Pet
{
    Pet(const std::string &name) : name(name)
    {
    }

    void SetName(const std::string &name_)
    {
        name = name_;
    }

    const std::string &GetName() const
    {
        return name;
    }

    std::string name;
};

PYBIND11_MODULE(example, m)
{
    pybind11::class_<Pet>(m, "Pet")
        .def(pybind11::init<const std::string &>())
        .def("SetName", &Pet::SetName)
        .def("GetName", &Pet::GetName)
        .def("__repr__", [](const Pet &a) { return "<example.Pet named '" + a.name + "'>"; });
}
```

其中 `__repr__` 方法绑定到一个 lambda 表达式，用于在 `print` 时使用

```python
import example

a = example.Pet("cat")
print(a.GetName())
a.SetName("dog")
print(a.GetName())
print(a)
```



#### 访问成员

现在不能访问 `name` 成员，如果需要获得 `name`，使用 `def_readwrite` 绑定

```cpp
PYBIND11_MODULE(example, m)
{
    pybind11::class_<Pet>(m, "Pet")
        .def(pybind11::init<const std::string &>())
        .def("SetName", &Pet::SetName)
        .def("GetName", &Pet::GetName)
        .def_readwrite("name", &Pet::name);
}
```

如果有常量字段，则使用 `def_readonly` 绑定。



如果一个成员是私有的，只能通过接口访问，则使用 `def_property` 绑定

```cpp
#include <pybind11/pybind11.h>

struct Pet
{
    Pet(const std::string &name) : name(name)
    {
    }

    void SetName(const std::string &name_)
    {
        name = name_;
    }

    const std::string &GetName() const
    {
        return name;
    }

private:
    std::string name;
};

PYBIND11_MODULE(example, m)
{
    pybind11::class_<Pet>(m, "Pet")
        .def(pybind11::init<const std::string &>())
        .def("SetName", &Pet::SetName)
        .def("GetName", &Pet::GetName)
        .def_property("name", &Pet::GetName, &Pet::SetName);
}
```

对于常量私有成员，使用 `def_property_readonly` 绑定。

> 类似的还有 `def_property_static, def_readonly_static, def_property_static, def_property_readonly_static` 用于静态属性。



#### 动态属性

Python 中的类具有动态属性，即可以添加不存在的属性

```python
class A:
	name = "li"

p = A()
p.age = 10	# 加入一个不存在的属性
```

而通过 pybind 导出的 C++ 类则没有这一特性，因此效率更高。如果需要使用动态属性，则在绑定时标记

```cpp
pybind11::class_<Pet>(m, "Pet", pybind11::dynamic_attr());
```



#### 继承关系

在模板中依次指定类，从而声明继承关系

```cpp
#include <pybind11/pybind11.h>

struct Pet
{
    Pet(const std::string &name) : name(name)
    {
    }

    std::string name;
};

struct Dog : Pet
{
    Dog(const std::string &name) : Pet(name)
    {
    }

    std::string bark() const
    {
        return "Woof!";
    }
};

PYBIND11_MODULE(example, m)
{
    pybind11::class_<Pet>(m, "Pet").def(pybind11::init<const std::string &>()).def_readwrite("name", &Pet::name);
    pybind11::class_<Dog, Pet>(m, "Dog").def(pybind11::init<const std::string &>()).def("bark", &Dog::bark);
}
```

然后可以直接使用

```python
import example

a = example.Pet("Li")
b = example.Dog("Wang")
print(b.bark())
```



#### 多态

常规的类继承不提供多态性质。例如定义函数

```cpp
m.def("pet_store", []() { return std::unique_ptr<Pet>(new Dog("Molly")); });
```

然后在 Python 中使用

```python
p = example.pet_store()
type(p)		# Pet
p.bark()	# 报错
```

由于函数返回 `Pet` 类指针，即使变量实际为 `Dog` 指针，也会被识别为 `Pet` 类型。



只有当存在至少一个虚函数时，才被认为是多态，并且会被 pybind 自动识别

```cpp
#include <pybind11/pybind11.h>

struct Pet
{
    virtual ~Pet() = default;
};

struct Dog : Pet
{
    std::string bark() const
    {
        return "woof!";
    }
};

PYBIND11_MODULE(example, m)
{
    pybind11::class_<Pet>(m, "Pet");
    pybind11::class_<Dog, Pet>(m, "Dog").def(pybind11::init<>()).def("bark", &Dog::bark);

    // 返回子类指针，能够被识别
    m.def("pet_store2", []() { return std::unique_ptr<Pet>(new Dog); });
}
```



#### 重载

如果存在多个重载函数，通过函数指针区分

```cpp
#include <pybind11/pybind11.h>

struct Pet
{
    Pet(const std::string &name, int age) : name(name), age(age)
    {
    }

    void Set(int age_)
    {
        age = age_;
    }

    void Set(const std::string &name_)
    {
        name = name_;
    }

    std::string name;
    int age;
};

PYBIND11_MODULE(example, m)
{
    pybind11::class_<Pet>(m, "Pet")
        .def(pybind11::init<const std::string &, int>())
        .def("set", static_cast<void (Pet::*)(int)>(&Pet::Set), "Set the pet's age")
        .def("set", static_cast<void (Pet::*)(const std::string &)>(&Pet::Set), "Set the pet's name");
}
```

如果使用 C++14，则可以简化的语法

```cpp
pybind11::class_<Pet>(m, "Pet")
	.def("set", pybind11::overload_cast<int>(&Pet::Set), "Set the pet's age")
	.def("set", pybind11::overload_cast<const std::string &>(&Pet::Set), "Set the pet's name");
```

这样只需要通过输入参数类型区分即可。



对于通过 `const` 修饰符重载的含糊，使用 `pybind11::const_` 标记

```cpp
#include <pybind11/pybind11.h>

struct Widget
{
    int foo(int x);
    int foo(int x) const;
};

PYBIND11_MODULE(example, m)
{
    pybind11::class_<Widget>(m, "Widget")
        .def("foo_mutable", pybind11::overload_cast<int>(&Widget::foo))
        .def("foo_const", pybind11::overload_cast<int>(&Widget::foo), pybind11::const_);
}
```



#### 枚举和内部类型

如果存在内部类型，只需要通过 `::` 获得内部类型然后使用前面的方法绑定

```cpp
#include <pybind11/pybind11.h>

struct Pet
{
    enum Kind
    {
        Dog = 0,
        Cat
    };

    struct Attributes
    {
        float age = 0;
    };

    Pet(const std::string &name, Kind type) : name(name), type(type)
    {
    }

    std::string name;
    Kind type;
    Attributes attr;
};

PYBIND11_MODULE(example, m)
{
    // 绑定类
    pybind11::class_<Pet> pet(m, "Pet");

    // 绑定构造函数和成员
    pet.def(pybind11::init<const std::string &, Pet::Kind>())
        .def_readwrite("name", &Pet::name)
        .def_readwrite("type", &Pet::type)
        .def_readwrite("attr", &Pet::attr);

    // 绑定枚举类型，通过 export_values() 导出到父作用域
    pybind11::enum_<Pet::Kind>(pet, "Kind").value("Dog", Pet::Kind::Dog).value("Cat", Pet::Kind::Cat).export_values();

    // 绑定嵌套类
    pybind11::class_<Pet::Attributes>(pet, "Attributes")
        .def(pybind11::init<>())
        .def_readwrite("age", &Pet::Attributes::age);
}
```

其中枚举类型可以直接转化为字符串，也能转换为整型

```python
import example

p = example.Pet("Fido", example.Pet.Cat)
print(p.type)
print(int(p.type))
```



### 嵌入解释器

可以将 python 解释器嵌入到 C++ 程序中。在 cmake 中配置

```cmake
cmake_minimum_required(VERSION 3.5...3.29)
project(example)

set(CMAKE_CXX_STANDARD 20)
set(PYBIND11_FINDPYTHON ON)

# 设置查找目录
set(pybind11_DIR "D:/lib/pybind11/share/cmake/pybind11")
find_package(pybind11 CONFIG REQUIRED)

add_executable(example main.cpp)
target_link_libraries(example PRIVATE pybind11::embed)
```

然后就可以在 C++ 中调用 Python API

```cpp
#include <pybind11/embed.h>

int main()
{
	// 初始化解释器
    pybind11::scoped_interpreter guard{};

	// 调用 API
    pybind11::print("Hello, world!");
    return 0;
}
```



#### 执行 Python 代码

可以直接将程序代码作为字符串嵌入执行

```cpp
#include <pybind11/embed.h>

int main()
{
    pybind11::scoped_interpreter guard{};
    pybind11::exec(R"(
        kwargs = dict(name="World", number=42)
        message = "Hello, {name}! The answer is {number}".format(**kwargs)
        print(message)
    )");
    return 0;
}
```

或者使用 API 调用

```cpp
#include <pybind11/embed.h>

// 导入字面量支持
using namespace pybind11::literals;

int main()
{
    pybind11::scoped_interpreter guard{};
    auto kwargs = pybind11::dict("name"_a = "World", "number"_a = 42);
    auto message = "Hello, {name}! The answer is {number}"_s.format(**kwargs);
    pybind11::print(message);
    return 0;
}
```

也可以结合使用

```cpp
#include <iostream>
#include <pybind11/embed.h>

// 导入字面量支持
using namespace pybind11::literals;

int main()
{
    pybind11::scoped_interpreter guard{};
    auto locals = pybind11::dict("name"_a = "World", "number"_a = 42);
    pybind11::exec(R"(
        message = "Hello, {name}! The answer is {number}".format(**locals())
    )",
                   pybind11::globals(), locals);

    auto message = locals["message"].cast<std::string>();
    std::cout << message << std::endl;
    return 0;
}
```



#### 导入模块

可以导入 Python 库

```cpp
// 导入系统模块
pybind11::module_ sys = pybind11::module_::import("sys");
pybind11::print(sys.attr("path"));

// 导入 calc 模块
py::module_ calc = py::module_::import("calc");
py::object result = calc.attr("add")(1, 2);
int n = result.cast<int>();
assert(n == 3);
```

如果模块被修改，可以使用 `module_::reload` 重新加载。



#### 嵌入式模块

可以使用 `PYBIND11_EMBEDDED_MODULE` 宏嵌入二进制模块

```cpp
#include <pybind11/embed.h>

PYBIND11_EMBEDDED_MODULE(fast_calc, m) 
{
    // m 是 py::module_ 类型
    m.def("add", [](int i, int j) {
        return i + j;
    });
}

int main() {
    pybind11::scoped_interpreter guard{};

	// 直接导入嵌入的模块
    auto fast_calc = pybind11::module_::import("fast_calc");
    auto result = fast_calc.attr("add")(1, 2).cast<int>();
    assert(result == 3);
}
```

与之前导出 pyd 库时只能指定同一个模块的情况不同，这种方式可以嵌入多个不同的模块。
