# 现代 Cpp 

## 版本检测

随着 C++ 标准的不断更新，其中的操作技术和细节也越来越多，我们按照版本顺序介绍几种常用的新特性。



确认编译器是否支持 `C++11` :考查宏 `__cplusplus` 的值。

```C++
int main()
{
    std::cout << __cplusplus << std::endl;
}
```

如果显示

```
201103
```

及以后，则表示该编译器支持 `C++11` 。



## ODR 使用

单一定义规则使用 (One Definition Rule Usage) 是指要求变量、函数、类等实体只能有唯一的定义。例如

```cpp
extern int x; // 声明，但没有定义 

int foo() 
{ 
	sizeof(x); 		// 这里不是 ODR 使用，因为它只检查了 x 的大小
	return x + 1;	// 这里是一个ODR使用，因为它使用了x的值 
}
```

如果一个变量被声明，并且它在被链接的所有文件中都未定义，则 ODR 使用将会报错。



## 消除歧义

### ADL

参数相关查找 ADL（Argument-Dependent Lookup）是一种名称查找机制，它允许在函数调用中根据参数的类型查找函数。例如

```cpp
#include <iostream>

namespace Test
{

class A
{
};

void func(A)
{
}

} // namespace Test

int main()
{
    Test::A obj;
    func(obj);
    return 0;
}

```

这里虽然没有指明 `func` 所在的命名空间，但是根据函数参数，可以自动查找到对应的命名空间。



ADL的查找规则如下：

1. 编译器首先在函数模板所在的命名空间中查找匹配的函数模板
2. 如果在函数模板所在的命名空间中找不到匹配的函数模板，则编译器会到函数调用所在的作用域中查找匹配的函数模板
3. 如果在函数调用所在的作用域中也找不到匹配的函数模板，则编译器会到实参类型所对应的命名空间中查找匹配的函数模板
4. 如果在实参类型所对应的命名空间中也找不到匹配的函数模板，则编译器会报错

ADL 的主要作用是允许函数模板在不同的命名空间中定义，而不需要使用全局命名空间或限定作用域。



根据查找规则，如果有两个命名空间中的同名函数，则会优先调用函数参数对应的命名空间中的函数。例如

```cpp
#include <iostream>

namespace Test
{

class A
{
};

// 此函数被调用
void func(A)
{
    std::cout << "Hello from Test::func(A)" << std::endl;
}

} // namespace Test

namespace Test2
{

void func(Test::A)
{
    std::cout << "Hello from Test2::func(Test::A)" << std::endl;
}

} // namespace Test2

int main()
{
    Test::A obj;
    func(obj);
    return 0;
}
```

再例如

```cpp
namespace Test
{

struct X
{
};

struct Y
{
};

void f(int);
void g(X);

} // namespace Test

namespace Test2
{

void f(int i)
{
	// 调用 Test2::f
    f(i);
}

void g(Test::X x)
{
    g(x);   // 报错，不能区分 Test::g, Test2::g
}

void h(Test::Y y)
{
	// 调用 Test2::h
    h(y);
}

} // namespace Test2
```



在 C++20 中增强了 ADL，能够匹配带有非类型模板参数的函数

```cpp
namespace Test
{

class A
{
};

template <int x> void select(A *){};

} // namespace Test

void f(Test::A *p)
{
    // 在 c++20 之前都会报错
    select<3>(p);
}

int main()
{
    Test::A a;
    f(&a);
    return 0;
}
```



### 待决名

在模板定义中，某些构造在不同的实例化之间都可能不同。例如

```cpp
// B<T> 实例化取决于 T
template <typename T> struct X : B<T>
{
    // T::A 取决于 T
    typename T::A *pa;

    void f(B<T> *pb)
    {
        // B<T>::i 取决于 T
        static int i = B<T>::i;
        pb->j++;
    }
};
```



如果一个类型、函数取决于模板类型，则这个类型、函数就成为待决名。例如

```cpp
template <class T> struct X
{
    void f()
    {
    }
};

template <class T> struct Y : X<T>
{
    void g()
    {
        // this->f() 取决于 X<T>
        this->f();
    }
};
```

其中 `this->f()` 就是一个待决名。待决名的名称查找推迟到实例化时进行，例如

```cpp
int main()
{
	// 待决名构造完成
    Y<int> y;
    y.g();
    return 0;
}
```



而非待决名在模板定义时就进行无限定的名称查找。例如

```cpp
template <class T> struct X
{
    void f()
    {
    }
};

template <class T> struct Y : X<T>
{
    void g()
    {
        // f() 是非待决名，它在定义时就进行名称查找
        f();
    }
};
```

此时这段代码不能编译通过，因为模板参数无限定情形下没有匹配的 `f` 实现。



## 语法糖

### 原始字面量

在使用字符串时对于某些具有特殊含义的字符需要转义才能正常输出，在 C++11 中添加了定义原始字符串的字面量

```cpp
R"xxx(原始字符串)xxx"
```

其中 `()` 两边的字符串可以忽略，但是如果它们存在就必须相同。

```cpp
string str1 = "D:\\hello\\world\\A.tex";		// 使用 \ 转义
string str2 = R"(D:\hello\world\A.tex)";		// 不需要使用，但是要加括号
string str3 = R"abc(D:\hello\world\A.tex)abc";	// 两边字符串会被忽略
```

另一个例子是输出换行

```cpp
string str1 = "Hello\
	World.";
string str2 = R"(Hello
    World)";
```

不必再使用反斜杠表示换行连接。



### for 循环

这个语法是对 `for` 循环语句的优化。往往我们需要遍历一个容器中的所有元素，迭代器的写法较为冗长且不够直观。为了更方便地进行遍历操作，C++11 提出了 `ranged_base for` 写法。

```C++
vector<double> vec;
// 传值
for (auto elem : vec)
{
    // 直接使用 elem 值
}

// 传引用
for (auto &elem : vec)
{
    // 可以修改 elem 值
}
```

在遍历容器时，如果只读数据，使用 `const auto &` 效率会高于 `const auto` ，因为省去了赋值操作

```cpp
// 只读数据
for (const auto &elem : vec)
{
    // ...
}
```

新的语法中冒号后面的内容只会执行一次，然后确定循环迭代的范围，因此我们不再需要考虑边界问题。



### 就地语句

在 C++17 中允许在 if 语句中直接声明局部变量，从而避免了多次遍历时需要重复定义变量的问题。例如下面的例子，如果没有这个特性，就需要在外部定义两个不同名字的迭代器

```cpp
// 将临时变量放到 if 语句内
if (const std::vector<int>::iterator itr = std::find(vec.begin(), vec.end(), 3); itr != vec.end())
{
    *itr = 4;
}

// 不需要再定义 itr2 变量
if (const std::vector<int>::iterator itr = std::find(vec.begin(), vec.end(), 2); itr != vec.end())
{
    *itr = 4;
}
```

这一特性也允许在 `switch, while` 语句中使用。



### 结构化绑定

在 C++17 中允许将返回值通过 `[]` 直接绑定到其中的变量。例如

```cpp
#include <iostream>
#include <tuple>

// 返回 3 个元素的元组
std::tuple<int, double, std::string> f()
{
    return std::make_tuple(1, 2.3, "456");
}

int main()
{
    // 这里直接将 3 个元素分别绑定到 x, y, z
    auto [x, y, z] = f();
    std::cout << x << ", " << y << ", " << z << std::endl;
    return 0;
}
```



结构化绑定还可以应用于循环体

```cpp
#include <iostream>
#include <map>
#include <vector>

template <typename Key, typename Value, typename F> void update(std::map<Key, Value> &m, F foo)
{
    // 遍历元素并调用 foo
    for (auto &&[key, value] : m)
        value = foo(key);
}

int main()
{
    std::map<std::string, long long int> m{{"a", 1}, {"b", 2}, {"c", 3}};
    update(m, [](std::string key) { return std::hash<std::string>{}(key); });

    for (auto &&[key, value] : m)
        std::cout << key << ":" << value << std::endl;
}
```



### 内联变量

在 C++17 中引入内联变量，允许将变量声明为内联，从而避免重复定义的问题。编译器可以选择将变量的值直接嵌入到使用它的地方，而不是每次都读取内存中的值，从而减小函数调用开销。

```cpp
inline int var = 10;
```

虽然内联变量的引入解决了多次定义错误的问题，并且可以提高性能，但并不是所有的变量都适合内联。对于较大的变量或者在多个编译单元中使用的变量，内联可能会导致代码膨胀和性能下降。因此，需要权衡使用内联变量的利弊。



### 字面量

可以对字符串进行重载，按照给定的形式调用对应的函数。支持 4 种字面量

1. 整型字面量：重载时必须使用 `unsigned long long`、`const char *`、模板字面量算符参数；
2. 浮点型字面量：重载时必须使用 `long double`、`const char *`、模板字面量算符；
3. 字符串字面量：必须使用 `(const char *, size_t)` 形式的参数表；
4. 字符字面量：参数只能是 `char`, `wchar_t`, `char16_t`, `char32_t` 这几种类型。



#### 字符串

引入 `std::literals` 来使用标准库中的字面量

```cpp
// using namespace std::literals;
using namespace std::string_literals; // 使用字符串字面量

int main() {
	auto str = "hello"s;
	return 0;
}
```



#### 单位量

利用字面量实现长度单位运算

```cpp
struct length
{
    double value;
    enum unit
    {
        metre,
        kilometre,
        millimetre,
        centimetre,
        inch,
        foot,
        yard,
        mile,
    };

    // 单位因子
    static double factors[];

    explicit length(double v, unit u = metre)
    {
        value = v * factors[u];
    }
};

// 单位因子
double length::factors[] = {1.0, 1000.0, 1e-3, 1e-2, 0.0254, 0.3048, 0.9144, 1609.344};

// 长度加法
length operator+(length lhs, length rhs)
{
    return length(lhs.value + rhs.value);
}

// v 单位为 m
length operator"" _m(long double v)
{
    return length(v, length::metre);
}

// v 单位为 cm
length operator"" _cm(long double v)
{
    return length(v, length::centimetre);
}

int main()
{
    // 获得长度
    length l1 = length(1.0, length::metre);
    length l2 = length(1.0, length::centimetre);
    std::cout << l2.value << std::endl;
    std::cout << (l1 + l2).value << std::endl;

    // 1.0_m + 10.0_cm
    std::cout << (1.0_m + 1.0_cm).value << std::endl;
    // _m, _cm 就是字符串形式的重载
    
    return 0;
}
```



#### 格式化输出

利用字面量实现格式化输出

```cpp
#include <format>
#include <iostream>

struct A
{
    constexpr A(const char* s) : str(s) {}
    const char* str;
};

// c++20 开始允许类作为非类型模板参数
template <A a>
constexpr auto operator""_f()
{
	// "{} * {} = {}\n" 传入 std::format 的第一个参数，将 args 作为格式化参数
    return [=]<typename... T>(T&&... args) { return std::format(a.str, std::forward<T>(args)...); };
}

int main()
{
	// "{} * {} = {}\n"_f 部分调用了上面的函数，返回 lambda 表达式，然后将 (5, 5.6, 5 * 5.6) 作为 lambda 表达式的参数
    std::cout << "{} * {} = {}\n"_f(5, 5.6, 5 * 5.6) << std::endl;
    return 0;
}
```



### 初始化分支

在 C++17 中可以在 `if, switch` 中初始化变量

```cpp
int n = 5;
if (auto x = n; x > 0)
    std::cout << "x is positive" << std::endl;
else if (auto x2 = n; x2 > 0)
    std::cout << "x2 is positive" << std::endl;
else
    std::cout << "x and x2 are not positive" << std::endl;
```

这等价于

```cpp
int n = 5;
{
    int x = n;
    if (x > 0)
        std::operator<<(std::cout, "x is positive").operator<<(std::endl);
    else
    {
        {
            int x2 = n;
            if (x2 > 0)
                std::operator<<(std::cout, "x2 is positive").operator<<(std::endl);
            else
                std::operator<<(std::cout, "x and x2 are not positive").operator<<(std::endl);
        }
    }
}
```

因此在前面分支中的括号中声明的变量在后面的分支中都可见。



借助这一局部作用域，可以实现线程自动解锁

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex m;
bool flag = false;

void f(int n)
{
    if (std::lock_guard l(m); flag)
        std::cout << "flag is true" << std::endl;

	// 离开 if 作用域后立即解锁
}
```



### 三路比较

在 C++20 中引入了三路比较符 `<=>`，它比较两个值，并返回比较结果对应的类型

* `std::strong_ordering`、`std::weak_ordering` 或 `std::partial_ordering`

```cpp
#include <iostream>

int main()
{
    int x = 1, y = 2;
    auto ret = x <=> y;
    if (ret == 0)
        std::cout << "x and y are equal\n";
    else if (ret < 0)
        std::cout << "x is less than y\n";
    else
        std::cout << "x is greater than y\n";
    
    return 0;
}
```

三路比较符的主要用途在于同时实现 `>=,==,<=,<,>` 运算符，例如默认生成比较运算

```cpp
#include <compare>
#include <iostream>

struct Point
{
    int x, y;
    
	// 默认生成比较操作符
    auto operator<=>(const Point &other) const = default;
};

int main()
{
    Point p1{1, 2};
    Point p2{1, 3};

    if (p1 < p2)
        std::cout << "p1 is less than p2\n";
    else if (p1 == p2)
        std::cout << "p1 is equal to p2\n";
    else
        std::cout << "p1 is greater than p2\n";

    return 0;
}
```

也可以指定比较方法

```cpp
struct Test
{
    int x;

    auto operator<=>(const Test &other) const
    {
        if (x == other.x)
            return 1;
        else if (x < other.x)
            return -1;
        else
            return 0;
    }
};
```



如果不需要某个运算符，可以直接将其删除

```cpp
auto operator<=>(const Point &other) const = default;
auto operator>(const Point &other) const = delete;
```



## 数据类型

### 数组

数组本身就是一个类型，数组长度也构成数组类型的一部分

```cpp
std::is_same_v<int[5], int[5]>; // true
std::is_same_v<int[5], int[6]>; // false
```



### nullptr_t

在 C++11 引入了空指针类型，它可以避免与整型等基本类型的隐式转换

```cpp
std::nullptr_t null = nullptr;
```

任何 `nullptr` 也都是 `std::nullptr_t` 类型，并且可以转换为任意指针类型。



只用于空指针，从而解决 `NULL` 二义性的问题。例如在下面的情况中，函数 func 被重载，如果使用 `NULL` 作为参数，会导致歧义

```cpp
void func(int v) 
{
    cout << "func(int v) - " << v << endl;
}

void func(int *v) 
{
    cout << "func(int *v) - " << v << endl;
}
```

因为 `NULL` 作为 0 被传入，无法判断应当调用哪个函数，此时需要用空指针 `nullptr` 代替；但同时，`NULL` 和 `nullptr` 在逻辑表达式中相等 

```cpp
cout << (NULL == nullptr) << endl; // 1
```



### thread_local

在 C++11 引入 `thread_local` 标志，用于为每个线程创建一个变量，从而避免多线程访问冲突。例如

```cpp
#include <iostream>
#include <thread>

// 定义一个线程局部的全局变量
thread_local int threadLocalVar = 0;

void threadFunction(int id) {
    // 设置线程局部变量的值
    threadLocalVar = id;
    std::cout << "Thread " << id << ": threadLocalVar = " << threadLocalVar << std::endl;
}

int main() {
    std::thread t1(threadFunction, 1);
    std::thread t2(threadFunction, 2);

    t1.join();
    t2.join();

    return 0;
}
```

每个线程访问到的是对应线程中的全局变量，不会冲突。



### any

在 C++17 中引入了任意类型 `std::any`，可以储存任何**可拷贝构造**和**可销毁**的类型的对象

```cpp
#include <any>

int main()
{
    std::any a = 1;
    std::cout << a.has_value() << " " << a.type().name() << std::endl;

    a = 3.14;
    std::cout << a.has_value() << " " << a.type().name() << std::endl;
    a = true;
    std::cout << a.has_value() << " " << a.type().name() << std::endl;
    
    // 原地重新构造
    a.emplace<std::string>("hello");
    
    // 重置变量为空
    a.reset();

    return 0;
}
```

可以显式指定变量类型

```cpp
std::any a = std::make_any<double>(3.14);
```

它与直接赋值的区别在于：直接赋值会在储存的过程中调用构造函数，而 `make_any` 接收已经调用构造函数的值。



通过 `in_place_type` 显式指定储存元素的类型

```cpp
// 向 set 中传入元素类型和排序函数
auto sc = [](int x, int y) { return std::abs(x) < std::abs(y); };
std::any a8{std::in_place_type<std::set<int, decltype(sc)>>, {4, 8, -7, -2, 0, 5}, sc};
```



在类型匹配的情况下，可以进行类型转换

```cpp
std::any a = 3.14;
double b = std::any_cast<double>(a);
double *c = std::any_cast<double>(&a);
```

传入地址会转换为指针类型。



可以接收移动构造，但是需要对应的类型同时**具有拷贝构造函数**

```cpp
std::string s("hello, world!");
std::any a = std::move(s);
```



### array

array 实际上就是固定大小的数组，是一种更安全、更容易使用的数组类型，它的好处是提供了现代化的容器算法。例如支持迭代器循环、支持标准库算法。用法总结为

```cpp 
// 需要指定容器长度和类型
std::array<int, 4> arr = {1, 2, 3, 4};

arr.empty(); // 检查容器是否为空
arr.size();  // 返回容纳的元素数

// 迭代器支持
for (auto &i : arr)
{
    // ...
}

// 用 lambda 表达式排序
std::sort(arr.begin(), arr.end(), [](int a, int b) { return b < a; });

// 数组大小参数必须是常量表达式
constexpr int len = 4;
std::array<int, len> arr = {1, 2, 3, 4};

// 非法,不同于 C 风格数组，std::array 不会自动退化成 T*
// int *arr_p = arr;
```

在进行指针操作时，可以通过取地址或 data 成员获取指针

```cpp
void foo(int *p, int len)
{
    return;
}

std::array<int, 4> arr = {1, 2, 3, 4};

foo(&arr[0], arr.size());
foo(arr.data(), arr.size());
```



### variant

在 C++17 引入了 `std::variant` 类型，它是一个类型安全的 union 变量，允许在不同类型之间进行切换。例如

```cpp
// 一般 union
union v {
    int a;
    double b;
    std::string c;
};

// 安全的 union
std::variant<int, double, std::string> v;
```

其安全性来源于编译期检查和自动析构。



#### 成员函数

它提供的基本操作包括

| 操作                       | 说明                                                   |
| -------------------------- | ------------------------------------------------------ |
| `emplace<T>()`             | 为 T 类型分配新的值                                    |
| `emplace<Ind>()`           | 为指定索引的类型分配新的值                             |
| `index()`                  | 当前值对应的类型索引                                   |
| `holds_alternative<T>()`   | 判断对应 T 类型是否存在值                              |
| `swap()`                   | 交换值                                                 |
| `valueless_by_exception()` | 返回是否有异常导致没有值                               |
| `get<T>()`                 | 获得对应 T 类型的值                                    |
| `get<Ind>()`               | 获得对应索引类型的值                                   |
| `get_if<T>()`              | 获得对应 T 类型的值的指针，如果不存在，则返回 nullptr  |
| `get_if<Idx>()`            | 获得对应索引类型的值的指针，如果不存在，则返回 nullptr |
| `visit()`                  | 对当前值执行操作                                       |



例如基本操作

```cpp
int main()
{
    // 安全联合
    std::variant<int, double, std::string> a = 1;
    auto int_value = std::get<int>(a);

    // error 不支持类型转换，目前保存的是 int 类型
    // auto double_value = std::get<double>(a);
    std::cout << std::holds_alternative<int>(a) << " " << std::holds_alternative<double>(a) << "\n";
    std::cout << a.index() << "\n";

    a = 1.2;
    std::cout << std::get<double>(a) << " " << std::get<1>(a) << "\n";

    a = "hello";
    std::cout << std::get<std::string>(a) << " " << std::get<2>(a) << "\n";

    // 获得对应指针
    std::cout << std::get_if<0>(&a) << " " << std::get_if<int>(&a) << "\n";
    std::cout << std::get_if<1>(&a) << " " << std::get_if<double>(&a) << "\n";
    std::cout << std::get_if<2>(&a) << " " << std::get_if<std::string>(&a) << "\n";

    std::variant_alternative<1, decltype(a)>::type f = 1.2; // 获得 a 的 1 号类型
    std::cout << std::variant_size_v<decltype(a)> << "\n";  // 获得 a 的类型数

    // 使用 std::visit 访问不同类型的值
    std::visit(
        [](auto &&arg) {
            using T = std::decay_t<decltype(arg)>;
            if constexpr (std::is_same_v<T, int>)
                std::cout << "int: " << arg << std::endl;
            else if constexpr (std::is_same_v<T, double>)
                std::cout << "double: " << arg << std::endl;
            else if constexpr (std::is_same_v<T, std::string>)
                std::cout << "string: " << arg << std::endl;
        },
        a);

    return 0;
}
```



#### 指定构造

可以通过 `std::in_place_index` 指定索引对应的构造类型

```cpp
int main()
{
    // 直接在 double 类型的位置构造一个对象
    std::variant<int, double, std::string> v{std::in_place_index<1>, 3.14};

    // 使用 index() 获取当前存储的类型的索引
    std::cout << "Current index: " << v.index() << std::endl;

    // 获取当前存储的值
    if (v.index() == 1)
        std::cout << "Value: " << std::get<double>(v) << std::endl;

    return 0;
}
```

这通常用于变长模板参数

```cpp
template <size_t n, typename... T> constexpr std::variant<T...> test(const std::tuple<T...> &tpl)
{
  	// 获得索引对应的变长模板参数中的类型
    return std::variant<T...>{std::in_place_index<n>, std::get<n>(tpl)};
}
```



#### monostate

这是一个空类型，用来表示 `std::variant` 不携带信息。例如

```cpp
std::variant<std::monostate, int, std::string> var;
```

由于构造函数无参，因此默认得到一个空的枚举变量。一般使用 `std::monostate` 表示异常状态。

> 任何情况下，两个 `std::monostate` 都相等。



### tuple

引入 `tuple` 库，即可使用元组。除了直接初始化，还可以通过 `make_tuple, forward_as_tuple` 初始化

```cpp
std::tuple<int, int, std::string> t, t1, t2;
t = std::make_tuple(1, 2, "qwe");
t1 = std::make_tuple(3, 2, "qwe");
t2 = std::forward_as_tuple(3, 2, "qwe");
```



#### 获得元素

通过 `std::get` 来获得对应位置的元素，可以修改元素

```cpp
int a =std::get<0>(t);
std::get<0>(t) = std::get<1>(t);
std::cout << a << std::endl;
std::cout << (std::get<0>(t) > std::get<1>(t) ? "true" : "false") << std::endl;
std::cout << (t1 == t2 ? "true" : "false") << std::endl;
```



通过 `std::tuple_size` 获得长度，通过 `std::tuple_element` 获得对应位置元素的类型

```cpp
typedef std::tuple<int, int, int, std::string> T;
std::cout << std::tuple_size<T>::value << std::endl;
std::cout << std::tuple_size<T>::value << std::endl;
std::tuple_element<1, T>::type a1 = 10;
std::cout << a1 << std::endl;
```



通过 `std::tie` 将元组中的元素拆分给单独变量

```cpp
std::tuple<double, char, std::string> student({1.2, 'A', "Li"});

double gpa;
char grade;
std::string name;

// 元组进行拆包
std::tie(gpa, grade, name) = student;
```



在 C++14 中允许通过类型获取元组中的元素

```cpp
std::tuple<double, char, std::string> student({1.2, 'A', "Li"});

std::get<double>(student);
std::get<char>(student);
std::get<std::string>(student);
```

注意如果存在重复类型，则不能使用这种方法。



#### 参数传递

在 C++17 中增加了 `std::apply`，可用于将元组分离为参数传递。例如

```cpp
#include <iostream>
#include <tuple>

void print_values(int a, float b, const std::string &c)
{
    std::cout << "a: " << a << ", b: " << b << ", c: " << c << std::endl;
}

int main()
{
    std::tuple<int, float, std::string> values(42, 3.14, "Hello");

    // 这里将 values 中的元素拆开传递
    std::apply(print_values, values);
    return 0;
}
```



借助 `std::apply`，可以将元组元素通过变长参数传递

```cpp
std::tuple<int, std::string, float> t1(10, "Test", 3.14);
std::apply([](auto &&...args) { ((std::cout << args << '\n'), ...); }, t1);
```



#### 运行期索引

由于 `get` 需要常量表达式作为索引，这就导致不能在运行期动态指定索引。要实现运行期索引，需要借助 `std::variant` 容纳多类型的变量。例如

```cpp
#include <iostream>
#include <tuple>
#include <variant>

template <size_t n, typename... T> constexpr std::variant<T...> _tuple_index(const std::tuple<T...> &tpl, size_t i)
{
    if constexpr (n >= sizeof...(T))
        throw std::out_of_range("越界.");

    // 获得对应 i == n 位置的元素，保存在 std::variant<T...> 中，因为它可以存不同类型的变量
    if (i == n)
        return std::variant<T...>{std::in_place_index<n>, std::get<n>(tpl)};

    // n + 1，实现常量递增
    return _tuple_index<(n < sizeof...(T) - 1 ? n + 1 : 0)>(tpl, i);
}

// 获得 tpl 中索引为 i 的元素
template <typename... T> constexpr std::variant<T...> tuple_index(const std::tuple<T...> &tpl, size_t i)
{
    // 0 作为模板参数
    return _tuple_index<0>(tpl, i);
}

// 通过 visit 访问 variant 的元素，并执行函数
template <typename T0, typename... Ts> std::ostream &operator<<(std::ostream &s, std::variant<T0, Ts...> const &v)
{
    std::visit([&](auto &&x) { s << x; }, v);
    return s;
}

int main()
{
    std::tuple<double, char, std::string> student({1.2, 'A', "Li"});

    // 可以通过索引获得元素
    int i = 1;
    std::cout << tuple_index(student, i) << std::endl;

    return 0;
}
```

注意到上面的索引函数是常量表达式函数，因此实际上在编译期就完成了计算。



借助运行期索引，可以实现遍历元组

```cpp
template <typename T> auto tuple_len(T &tpl)
{
    // 获得 T 类型元组的长度的值
    return std::tuple_size_v<T>;
}


int main()
{
    std::tuple<double, char, std::string> student({1.2, 'A', "Li"});

    // 可以通过索引获得元素
    for (int i = 0; i != tuple_len(student); i++)
        std::cout << tuple_index(student, i) << std::endl;

    return 0;
}
```



#### 元组合并

使用 `std::tuple_cat` 合并元组

```cpp
auto new_tuple = std::tuple_cat(student, student);
```



## 标准属性

### 基本介绍

C++ 中的属性（Attributes）是一种用于向编译器传递特定信息的机制。这些信息可以用于优化、发出警告、启用或禁用某些检查等。C++11 引入了标准化的属性语法，即双方括号 `[[...]]`，这些属性被用来影响编译器的行为而不会改变程序的语义。



### 常用属性

#### nodiscard

标记一个函数或类型的返回值必须被使用。例如

```cpp
// 类实例化后不能丢弃
class [[nodiscard]] A {

};

// 返回值不能丢弃
[[nodiscard]] int add(int a, int b) {
    return a + b;
}

// warning C4834: 放弃具有 `[[nodiscard]]` 属性的函数的返回值
add(1, 2);
```

如果确实需要忽略返回值，可以使用

```cpp
std::ignore = A();
```



#### maybe_unused

标记一个变量可能不会使用

```cpp
void example([[maybe_unused]] int x) {
    // 即使 x 没有被使用，也不会有警告
}
```



#### fallthrough

标记 switch 语句中的穿透行为

```cpp
switch(value) {
    case 1:
    // 标记 case 1 直接穿透到 case 2
        [[fallthrough]];
    case 2:
        // ...
        break;
    default:
        break;
}
```



#### likely/unlikely

标记一个分支很有可能执行或很不可能执行

```cpp
if ([[likely]] condition) {
    // 更可能执行的代码路径
} else {
    // 不太可能执行的代码路径
}
```



#### noreturn

标记一个函数没有返回值，用于异常处理函数或程序终止函数

```cpp
[[noreturn]] void terminateProgram() { 
	throw std::runtime_error("Program terminated"); 
	// 后面不返回值
}
```



#### `__declspec(noinline)` 

当进行调试测试时，我们可能希望一些函数不被编译器内联优化，以便于我们追踪函数调用栈。此时可以使用此声明

```cpp
__declspec(noinline) void debug() { 
	std::cout << "This function will not be inlined." << std::endl; 
}
```

当一个函数使用了 `__declspec(noinline)`，即使编译器认为将其内联会带来性能提升，它也会强制不对该函数进行内联。



## 命名空间

### 嵌套命名空间

当多个命名空间嵌套时，例如

```cpp
namespace outer {
    namespace inner {
        
    } // namespace inner
} // namespace outer
```

可以写成

```cpp
namespace outer::inner {
    
} // namespace outer
```

从而简化内部空间中类、函数的缩进格式。



### 内联命名空间

内联命名空间中的名称自动放在外部作用于中。例如

```cpp
namespace outer {
    inline namespace inner {
    	void foo();
    } // namespace inner
} // namespace outer
```

它实际上等价于

```cpp
namespace outer {
    void foo();
} // namespace outer
```

当需要调用 foo 函数时，只需要通过 out::foo() 进行调用。



内联命名空间的意义在于兼容接口。例如同一个函数进行了不同的更新迭代

```cpp
namespace Parent
{
    // 原始版本
    namespace V1
    {
        void foo() { std::cout << "foo v1.0" << std::endl; }
    }
	
    // 更新版本
    inline namespace V2
    {
        void foo() { std::cout << "foo v2.0" << std::endl; }
    }
}

int main()
{
    Parent::foo();
}
```

这时候要调用最新版本的 foo 函数，可以直接通过 Parent::foo()；如果需要调用原始的版本，使用

```cpp
Parent::V1::foo();
```

如果更新了第三个版本，只需要改为

```cpp
namespace Parent
{
    // 原始版本
    namespace V1
    {
        void foo() { std::cout << "foo v1.0" << std::endl; }
    }
	
    // 版本 2
    namespace V2
    {
        void foo() { std::cout << "foo v2.0" << std::endl; }
    }
    
    // 版本 3
    inline namespace V3
    {
        void foo() { std::cout << "foo v3.0" << std::endl; }
    }
}

int main()
{
    Parent::foo();
}
```



## 非受限联合体

联合体 union 中的数据会共享同一块内存空间。在 C++11 之前的联合体有如下限制：

1. 不能拥有非 POD 类型的成员
2. 不能拥有静态成员
3. 不能拥有引用成员

新的标准规定**任何非引用类型**都可以成为联合体的数据成员，这就是非受限联合体。



### 静态成员

我们可以定义静态成员变量和函数

```cpp
union Test  
{  
    int age;  
    long id;  
      
    // 静态成员  
    static char c;  
    static int print()  
    {  
        cout << "c value: " << c << endl;  
        return 0;  
    }  
};  
char Test::c;
```

其中**静态成员变量共用一块内存，同一实例的非静态成员变量共用另一块内存**。



### POD 类型

POD 是 Plain Old Data 的缩写，即普通的旧数据，通常用于说明一个类型的属性。使用 POD 类型的好处在于

1. 字节赋值。可以使用 memset 和 memcpy 对 POD 类型初始化和拷贝
2. 提供对 C 内存布局兼容。C++ 程序可以与 C 函数进行相互操作
3. 保证静态初始化安全有效



#### 平凡类型

平凡的类或结构体应该具有如下特点：

1. 具有平凡的默认构造函数和析构函数
    - 不定义任何构造函数或使用 `=defualt` 关键字声明
2. 具有平凡的拷贝构造函数和移动构造函数
    - 同理于上
3. 具有平凡的拷贝赋值运算和移动赋值运算
    - 同理于上
4. 不包含虚类和虚基类



#### 标准布局类型

标准布局类型应该具有如下特点：

1. 所有非静态成员有**相同**的访问权限
2. 在类或者结构体继承时，满足下面情况之一
    - 派生类中有非静态成员，基类中包含静态成员（或基类没有变量）
    - 基类有非静态成员，而派生类没有非静态成员
3. 子类中第一个非静态成员的类型与基类不同
4. 没有虚函数和虚基类
5. 所有非静态数据成员均符合标准布局类型，其基类也符合标准布局类型

例如对于 g++ 编译器

```cpp
struct Parent {};
struct Child : public Parent
{
    Parent p;   // 子类的第一个非静态成员
    int foo;
};

struct Parent {};
struct Child : public Parent
{
    int foo;    // 子类的第一个非静态成员
    Parent p;
};
```

其中第一个子类不是标准布局类型，**因为第一个非静态成员变量和父类类型相同**，而后者就是一个标准布局。



#### 对 POD 类型的判断

可以使用 C++11 提供的 `<type_traits>` 判断数据是否是 POD 类型。使用 `is_trivial` 类模板

```cpp
template <class T> struct std::is_trivial;
```

其成员 value 是 bool 类型，可以判断 T 是否是平凡类型。`is_trivial` 还可以对内置的标准类型数据 int, float 等以及数组类型进行判断

```cpp
#include <iostream>  
#include <type_traits>  
using namespace std;  
​  
class A {};  
class B { B() {} };  
class C : B {};  
class D { virtual void fn() {} };  
class E : virtual public A { };  
​  
int main()   
{  
    cout << std::boolalpha;  
    cout << "is_trivial:" << std::endl;  
    cout << "int: " << is_trivial<int>::value << endl;  
    cout << "A: " << is_trivial<A>::value << endl;  
    cout << "B: " << is_trivial<B>::value << endl;  
    cout << "C: " << is_trivial<C>::value << endl;  
    cout << "D: " << is_trivial<D>::value << endl;  
    cout << "E: " << is_trivial<E>::value << endl;  
    return 0;  
}
```

其中使用 `boolalpha` 来将 bool 值转化为字符串 `true/false` 输出，使用 `noboolalpha` 取消这一转化。



同样可以使用 `is_standard_layout` 判断类型是否符合标准布局

```cpp
template <class T> struct std::is_standard_layout;
```

通过 value 成员进行判断，方法和上面相同。



### 非 POD 类型

在 C++11 中会默认删除一些非受限联合体的默认构造函数。例如它有一个非 POD 成员，该成员有非平凡构造函数，则非受限联合体的默认构造函数会被删除，其它特殊成员函数，如默认拷贝构造函数等也有此规则

```cpp
union Student  
{  
    int id;  
    string name;  
};  
​  
int main()  
{  
    Student s;  
    return 0;  
}
```

此段代码不能编译通过，因为 string 是非 POD 类型，有非平凡构造函数，因此 Student 没有默认构造函数。



#### placement new

解决方案是使用**定位放置 new** 操作。一般情况下，new 申请空间时通过堆分配内存，但是某些时候需要在已分配的特定内存创建对象，这种操作就叫做 `placement new` 即定位放置，格式为

```cpp
ClassName* ptr = new (address)ClassName;
```

例如在指定的地址创建对象指针

```cpp
class Base  
{  
public:  
    int number;  
};  
​  
int n = 100;  
Base *p = new (&n)Base; 
```

预先分配的内存地址而不是内存分配器新分配出来的地址，在你**提供的内存**上就地构建对象，从而实现**不额外分配堆内存、只调用构造函数**的效果。它的好处在于可以解决一些**需要构造参数的类**进行 new 时出现的问题，可以先 new 然后再调用构造函数

```cpp
Vec *U = new Vec[s];  
for (int i = 0; i < s; i++)  
{  
    // 在这里调用构造函数初始化  
    new (U + i) Vec(n, 0);  
}
```

使用定位放置 new 操作，我们甚至可以反复申请同一块内存，避免内存重复创建销毁的开销。



#### 自定义构造函数

通过定位放置 new 操作，可以定义非受限联合体的构造函数

```cpp
union Student  
{  
    Student()  
    {  
        new (&name)string;  
    }  
    ~Student() {}  
      
    int id;  
    string name;  
};  
​  
int main()  
{  
    Student s;  
    s.name = "Li";  
    s.id = 100;  
    return 0;  
}
```

在这里我们定义的构造函数**定位放置**了对象的地址与 `string name` 成员变量相同，这样整个联合体就可以共用 `string name` 的内存。



#### 匿名非受限联合体

可以定义匿名的非受限联合体，一个比较实用的场景就是配合类的定义使用

```cpp
// 假设我们要储存不同人员的不同信息  
// 学生登记学号  
// 本地人和学生都要登记身份证  
// 外地人登记地址和联系方式  
​  
// 外来人口信息的结构体  
struct Foreigner  
{  
    Foreigner(string s, string ph) : addr(s), phone(ph) {}  
    string addr;  
    string phone;  
};  
​  
// 登记人口信息  
class Person  
{  
public:  
    // 使用枚举进行区分  
    enum class Category : char {Student, Local, Foreign};  
    // 根据不同的输入判断人员类型  
    Person(int num) : number(num), type(Category::Student) {}  
    Person(string id) : idNum(id), type(Category::Local) {}  
    Person(string addr, string phone) : foreign(addr, phone), type(Category::Foreign) {}  
    ~Person() {}  
​  
private:  
    Category type;  
    // 使用匿名非受限联合体减少内存空间占用  
    union  
    {  
        int number;  
        string idNum;  
        Foreigner foreign;  
    };  
};
```

在 Person 类中可以直接访问 foreign 等内部数据成员。



## 类型转换

### 数值类型转换

在 C++11 中专门提供了类型转换函数。



#### 数值转换为字符串

在 `<string>` 库中提供了 to_string 方法

```cpp
string to_string (int val);
string to_string (long val);
string to_string (long long val);
string to_string (unsigned val);
string to_string (unsigned long val);
string to_string (unsigned long long val);
string to_string (float val);
string to_string (double val);
string to_string (long double val);
```



#### 字符串转换为数值

对于不同类型的数值提供了不同的函数

```cpp
int       stoi( const std::string& str, std::size_t* pos = 0, int base = 10 );
long      stol( const std::string& str, std::size_t* pos = 0, int base = 10 );
long long stoll( const std::string& str, std::size_t* pos = 0, int base = 10 );

unsigned long      stoul( const std::string& str, std::size_t* pos = 0, int base = 10 );
unsigned long long stoull( const std::string& str, std::size_t* pos = 0, int base = 10 );

float       stof( const std::string& str, std::size_t* pos = 0 );
double      stod( const std::string& str, std::size_t* pos = 0 );
long double stold( const std::string& str, std::size_t* pos = 0 );
```

参数含义

* str 是要转换的字符串
* pos 是传出参数，记录从哪个字符开始无法继续进行解析
* base 若为 0 则自动检测数值进制，若前缀为 0 则为 8 进制，如果前缀为 0x 或 0X 则为 16 进制，否则为 10 进制

只有前面部分字符是数值才能进行转换，如果第一个字符不是数值就会失败。



### 强制类型转换

C++ 语言是一种所谓的“静态语言”，各数据类型之间往往无法直接转换，需要借助一些工具。



#### 静态强制类型转换

一般情况下，通过为类型加上 `()` 来强制转换为该类型

```cpp
Company *company = new Company("Apple", "Iphone");
TechCompany *techcompany = (TechCompany)company;
delete company;
```

上面 company 是 Company 类型的变量，要转换为 TechCompany 类型才能赋值给 techcompany ；只需要删除 company ，因为 company 和 techcompany 指向同一块内存，只需要释放一次。



#### 动态强制类型转换

使用 `const_cast` 可以将 const 属性去除，产生非 const 变量

```cpp
const_cast<MyClass*>(value)
```

这样就改变了 value 的常量性。



使用 `dynamic_cast` 进行安全转换，若转换的类有误则返回 NULL

```cpp
dynamic_cast<MyClass*>(value)
```

用来把一种类型的对象指针安全地强制转换为另一种类型的对象指针，如果 value 的类型不是一个 MyClass 类（或 MyClass 的子类）的指针，这个操作符返回 NULL 。



使用 `reinterpret_cast` 以二进制覆盖转换进行转换

```cpp
reinterpret_cast<T>(value)
```

在不进行任何实质性转换的情况下，把一种类型的指针解释为另一种类型的指针或者把一种整数解释为另一种整数。



而 `static_cast` 为老式强制类型转换操作的替代品

```cpp
static_cast<T>(value)
```



这些方法还有对应于智能指针的转换。例如

```cpp
std::shared_ptr<Base> basePtr = std::make_shared<Derived>();
std::shared_ptr<Derived> derivedPtr = std::dynamic_pointer_cast<Derived>(basePtr);
```

再例如

```
std::shared_ptr<int> intPtr = std::make_shared<int>(100);
std::shared_ptr<float> floatPtr = std::reinterpret_pointer_cast<float>(intPtr);
```



### C 风格转换

当我们在 C++ 中使用 C 风格的类型转换时，编译器将其转换为 C++ 的类型转换，这将造成不安全。例如

```cpp
int *p = nullptr;
int x = static_cast<int>(p);	// 编译错误
int y = (int)p;					// 编译通过
```

这是因为编译器将会依次尝试 `const_cast->static_cast->reinterpret_cast->reinterpret_cast(const_cast)` 于是 `(int)p` 被转换为

```coo
reinterpret_cast<int>(p);
```

从而导致于预期不符的结果。



再例如

```cpp
const int n = 1;
auto t = reinterpret_cast<int*>(&n);
```

将会报错，因为无法转换 `const` 类型的数据。而使用 C 风格写法却能够转换成功

```cpp
auto t = (int*)&n;
```



## 类型推导

### typeid

通过 `typeinfo` 库可以获得一个变量对应的类型字符串，注意返回值与编译器有关，不一定能得到符合格式要求的结果

```cpp
#include <iostream>
#include <typeinfo>

int main(int argc, char *argv[])
{
    auto &typeInfo = typeid(int);
    const std::type_info &typeInfo2 = typeid(20);

    std::cout << "typeInfo.name():" << typeInfo.name() << std::endl;
    std::cout << "typeInfo2.name():" << typeInfo2.name() << std::endl;

    if (typeid(int) == typeid(10))
        std::cout << "==" << std::endl;
    else
        std::cout << "!=" << std::endl;

    return 0;
}
```

对于继承类，返回值是当前变量的类型，与实际的多态类型无关

```cpp
#include <iostream>
#include <typeinfo>

class Base
{
  public:
    Base() = default;
    virtual ~Base() noexcept = default;

    virtual void print()
    {
        std::cout << "Base" << std::endl;
    }
};

class Deriv : public Base
{
  public:
    Deriv() = default;
    ~Deriv() noexcept = default;

    virtual void print()
    {
        std::cout << "Deriv" << std::endl;
    }
};

int main(int argc, char *argv[])
{
    Base *base = new Deriv;
    Deriv *deriv = new Deriv;

    if (typeid(base) == typeid(deriv))
        std::cout << "==" << std::endl;
    else
        std::cout << "!=" << std::endl;

    std::cout << "Base id: " << typeid(base).name() << std::endl;
    std::cout << "Deriv id: " << typeid(deriv).name() << std::endl;

    return 0;
}
```



### auto

使用 auto 声明的变量必须要进行初始化，以让编译器推导出它的实际类型，它属于编译器特性，不影响运行效率

```cpp
auto i = 10; //int
auto str = "c++"; //char
auto p = new Person();
p->run();
```

变量不是指针或引用时，推导结果不保留 const, volatile 关键字，反之则会保留

```cpp
int tmp = 150;
const auto a1 = tmp;
auto a2 = a1;
const auto &a3 = tmp;
auto &a4 = a3;
```

* a1 是 `const int` ，auto 被推导为 `int`
* a2 是 `int` ，auto 被推导为 `int`
* a3 是 `const int&` ，auto 被推导为 `const int` 
* a4 是 `const int&` ，auto 被推导为 `const int`



#### 限制条件

auto 不是万能的，在以下场景中不能进行推导：

1. 不能作为函数参数使用
2. 不能用于类的非静态成员变量的初始化
3. 不能使用 auto 定义数组
4. 不能使用 auto 推导模板参数



#### 简化返回类型

容器迭代器的类型声明往往比较繁琐，使用 auto 可以简便许多

```cpp
vector<int> v;
auto it = v.begin();
```

在使用模板时，很多情况下不知道具体的类型

```cpp
template <class T>
class A1
{
public:
    static int get() { return 1; }
};

template <class T>
class A2
{
public:
    static char get() { return '2'; }
};

template <class T>
void func()
{
    auto val = T::get();
    cout << val << endl;
}

int main()
{
    func<A1>();
    func<A2>();
}
```

由于 func 使用不同的类作为参数，因此不知道要使用哪种类型，就可以用 auto 替代。如果不使用 auto 就不得不再定义一个模板参数，并且还需要在调用函数时显式地声明两个模板参数。



C++11 中，auto 作为返回值应当指定返回类型，如下；C++14 可以不指定

```cpp
auto sum(T1 x, T2 y) -> decltype(x + y)
{
    return x + y;
}
```



#### 作为参数

在 C++20 中，允许 auto 作为传入变量的类型

```cpp
int add(auto x, auto y)
{
    return x + y;
}

auto i = 5; // 被推导为 int
auto j = 6; // 被推导为 int
std::cout << add(i, j) << std::endl;
```



### declval

在 `<utility>` 中定义了 `std::declval` 模板

```cpp
template<class T>
typename std::add_rvalue_reference<T>::type declval() noexcept;
```

它接收一个类型，然后**返回对应类型的右值引用，但是不生成对应类型的实例**。例如在下面的情况中，使用 decltype 获得类型时，就不需要调用 T1, T2 的默认构造函数

```cpp
#include <utility>

template <typename T1, typename T2,
          typename R = typename std::decay_t<decltype(true ? std::declval<T1>() : std::declval<T2>())>>
R GetMax(T1 a, T2 b)
{
    return b < a ? a : b;
}
```

其中一个好处是可以在编译期获取对应的类型。



另一个好处是可以**避免纯虚函数和无默认构造函数的类的类型获取问题**。例如下面 `NonDefault` 类型，无法直接使用 `decltype` 来获得类型，需要通过 `declval` 嵌套获得

```cpp
#include <iostream>
#include <utility>

// 有默认构造
struct Default
{
    int foo() const
    {
        return 1;
    }
};

// 无默认构造
struct NonDefault
{
    NonDefault() = delete;
    int foo() const
    {
        return 1;
    }
};

int main()
{
    decltype(Default().foo()) n1 = 1;
    // decltype(NonDefault().foo()) n2 = n1;
    decltype(std::declval<NonDefault>().foo()) n2 = n1;
    std::cout << "n1 = " << n1 << '\n' << "n2 = " << n2 << '\n';
}
```



### decltype

decltype 并不会实际计算表达式的值，编译器分析表达式并得到它的类型，直接返回变量一切类型：引用、指针、数组。函数调用也算一种表达式，因此不必担心在使用 decltype 时执行了函数。



#### 基本介绍

decltype 与变量相结合，可用于定义特定的变量，例如：

```cpp
int a = 10;
decltype(a) b = 20;

int a[] = {5, 6};
decltype(a) b = {4, 3};
```



decltype 也可以直接运用于表达式，获取表达式结果的类型，但并**不执行表达式**。表达式是一个右值，则返回右值类型

```cpp
int r = 4;
decltype(r + 0) b = 10; // 右值 int
```

表达式是左值，或者被 `()` 包围，则返回对应类型的引用，不能忽略 const, volatile 限定符

```cpp
int *p = &r;
decltype(*p) c = 5;			// 左值 int &
decltype((r)) d = 5;		// () 包围 int &
decltype(r = r + 1) e = 5;	// 左值 int &
```



decltype 运用于函数，通过函数的返回值和形参列表，定义了一种名为**函数类型**的东西，它的作用主要是为了定义函数指针。我们可以用 using 来声明一种类型，在这里是声明一种函数类型

```cpp
using FuncType = int(int &, int);
```

然后可以使用这一类型来定义变量，这里定义的是函数指针，它指向一个同类型的函数

```cpp
int add_to(int &des, int ori);
FuncType *pf = add_to;
```

通过 decltype 获取函数类型，只需要传入一个函数名

```cpp
decltype(add_to) *pf = add_to;
```

也可获取返回值类型，需要传入函数定义

```cpp
decltype(add_to(int, int)) m = 5;
```

需要注意：对于返回右值的函数，除了类类型，其余推导出的类型不会携带 const, volatile 限定符

```cpp
const int func();
decltype(func()) t = 1;
```

此时 t 的类型时 `int` 而非 `const int` 。



#### 返回推导

第一种用法一般用于模板函数的设计，比如下例：

```C++
template <typename T1, typename T2>
auto add(const T1 &x, const T2 &y) -> decltype(x + y)
{
    return x + y;
}
```

此函数显式声明模板的返回值类型为 `decltype(x + y)` 。在后置语法中甚至还可以调用函数

```cpp
int test(int i)
{
    return i;
}

double test(double d)
{
    return d;
}

template <class T>
auto func(T t) -> decltype(test(t))
{
    return test(t);
}
```

先调用函数获取返回值类型，然后进入函数内部。



在 C++14 开始，可以让普通函数具备返回推导，因此可以省去 `decltype` 声明

```cpp
template <class T>
auto func(T t)
{
    return test(t);
}
```



#### 对象类型

第二种用法主要在程序规模较大，模板类型错综复杂时起作用，我们可以用这个写法较为方便的知道某个类模板对象的具体类型。比如：

```C++
template <typename T>
void func(T obj)
{
    typedef typename decltype(obj)::iterator iType;
}
```

其中 iType 表示 T 的迭代器的数据类型。注意这种情况下，如果传入的对象数据类型不是一个容器（没有迭代器），则会出现编译错误。



对于 `lambda` 表达式，更常见的情况是我们有一个 `lambda` 对象，但没有该对象的数据类型（ `lambda` 的数据类型极为复杂），这时 `auto` 和 `decltype` 就能派上用场了。例如：

```C++
auto cmp = [](int a, int b)
{
    return (a > b);
};
set<int, decltype(cmp)> S(cmp); //存储顺序从大到小的有序集合
```



#### 编译排查

一种模板技巧是利用逗号运算符，它返回最后一个变量的值。配合 `decltype` 实现编译时检查，例如

```cpp
template <typename T> auto len(T const &&t) -> decltype((void)(t.size()), T::size_type)
{
    return t.size();
}
```

当 T 没有 `size()` 成员时，编译将会报错，因此 `(void)(t.size())` 实际上起到排查错误的作用。



### decltype(auto)

这是 C++14 开始提供的用法，用于返回类型转发时的类型推导。例如

```cpp
std::string  lookup1();	// 值
std::string& lookup2();	// 右值
```

对于这两个函数的封装，在 C++11 中为

```cpp
std::string look_up_a_string_1() 
{
    return lookup1();
}

std::string& look_up_a_string_2() 
{
    return lookup2();
}
```

而在 C++14 中可以简化为

```cpp
decltype(auto) look_up_a_string_1()
{
    return lookup1();
}

decltype(auto) look_up_a_string_2() 
{
    return lookup2();
}
```

当需要将函数保存为变量时，可以利用 `decltype(auto)` 推导类型

```cpp
int func1() {
	return 10;
}

int &func2(int *x) {
	return *x;
}

int x = 10;

decltype(auto) f1 = func1();
decltype(auto) f2 = func2(&x);
```

如果使用 `auto` 推导，就需要显式指定引用

```cpp
int x = 10;

auto f1 = func1();
auto& f2 = func2(&x);
```



### decay

在 C++14 中引入了类型衰减，用于获得当前类型的简化版本，例如

```cpp
#include <iostream>
#include <type_traits>

template <typename T> void printType()
{
    // 使用 std::decay_t 获取衰减后的类型
    using DecayType = std::decay_t<T>;
    std::cout << "Original type: " << typeid(T).name() << std::endl;
    std::cout << "Decayed type: " << typeid(DecayType).name() << std::endl;
}

// 用于比较 T 的衰减类型与 U 是否相同（注意继承关系）
template <typename T, typename U> struct decay_equiv : std::is_same<typename std::decay<T>::type, U>::type
{
};

int main()
{
    std::cout << std::boolalpha << decay_equiv<int, int>::value << '\n'
              << decay_equiv<int &, int>::value << '\n'
              << decay_equiv<int &&, int>::value << '\n'
              << decay_equiv<const int &, int>::value << '\n'
              << decay_equiv<int[2], int *>::value << '\n'
              << decay_equiv<int(int), int (*)(int)>::value << '\n';

    printType<int>();         // int
    printType<int &>();       // int
    printType<int &&>();      // int
    printType<const int &>(); // int
    printType<int[2]>();      // int *
    printType<int(int)>();    // int(*)(int)

    return 0;
}
```



使用类型衰减，可以在模板函数中获得最原始的类型（取消指针、常量或引用类型），确保模板参数类型正确。例如

```cpp
#include <iostream>
#include <vector>

template <template <typename, typename> class OutContainer = std::vector, typename F, class R>
auto fmap(R &&inputs, F &&f)
{
    // 获得 inputs 传入 f 后的返回值类型的退化类型
    typedef std::decay_t<decltype(f(*inputs.begin()))> result_type;

    // 由于 OutContainer 是 std::vector 模板，因此需要同时提供类型和对应的分配器
    OutContainer<result_type, std::allocator<result_type>> result;

    // 创建新的容器保存结果
    for (auto &&item : inputs)
        result.push_back(f(item));
    return result;
}

// 对每一个数进行加1操作
int add(int x)
{
    return x + 1;
}

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};
    auto result = fmap(v, add);

    for (auto item : result)
        std::cout << item << " ";
    std::cout << std::endl;
    return 0;
}
```



### using

C++11 允许使用 using 代替 typedef ，且更加灵活。例如可以存放数据类型（按照别名形式）

```cpp
using Tx = std::vector<double>;	 // Tx 将作为模板库的别名
```

使用时作为类型

```cpp
Tx vec(5) = {0, 1, 2, 3, 4};
```

还有函数指针类型

```C++
using func = void(*)(int,int);
```

很明显，这里的 `func` 就代表一个函数指针。



#### 模板别名

使用 typedef 虽然可以重定义类型，例如

```cpp
typedef map<int, string> m1;
typedef map<int, int> m2;
typedef map<int, double> m3;
```

但这种固定第一个参数类型的定义非常麻烦，我们自然想到使用模板定义，但是 typedef 并不支持模板别名，需要定义外敷类

```cpp
// 定义外敷类
template <class T>
struct MyMap
{
    typedef map<int, T> type;
};

int main()
{
    // 需要通过这种方式实例化类型
    MyMap<int>::type m;
  	return 0;
};
```

如果使用 using 就可以简化为

```cpp
template <class T>
using mymap = map<int, T>;

int main()
{
    mymap<int> m;
  	return 0;  
};
```



#### 隐藏

隐藏是指派生类的函数屏蔽了与其同名的基类函数，规则如下：

1. 派生类的函数与基类的函数同名，但是参数不同。此时，不论有无 virtual 关键字，基类的函数将被隐藏（不是重载）
2. 派生类的函数与基类的函数同名，并且参数也相同，但是基类函数没有 virtual 关键字。此时，基类的函数被隐藏（不是覆盖）

使用了 using 关键字，就可以避免 1 的情况，使得父类同名函数在子类中得以重载，不被隐藏。



应用在类中，让父类同名函数在子类中以重载方式使用。我们之前提到，如果子类有与父类相同的函数，需要显式地用作用域操作符来进行调用，例如

```cpp
class Animal
{
public:
	void speak() 
    {
        cout << "Animal::speak()" << endl;
    }    
};

class Cat : public Animal
{
public:
    void speak() 
    {
        Animal::speak(); // 直接调用基类方法
        cout << "Cat::speak()" << endl;
    }
}
```

但通过 using 可以使子类实例以重载形式调用父类同名方法，但需要注意，父类中不能有该函数 private 类型的版本

```cpp
class Base
{
public:
	void menfcn()
	{
		cout << "Base function" << endl;
	}
 
	void menfcn(int n)
	{
		cout << "Base function with int" << endl;
	}
// 不能有该函数的私有版本
// private: 
//	 void menfcn(std::string _name) {}
};
 
class Derived : private Base
{
public:
	using Base::menfcn;
    
	int menfcn(int num)
	{
		cout << "Derived function with int : "<< num << endl;
		return num;
	}
};
```

这样就可以直接调用父类方法

```cpp
Derived d;
d.menfcn();	// 这是父类的方法
```

在构造函数一节中，我们也有探讨这一方法。



### common_type

#### 公共类型

假设需要比较两个值的最小值并返回，但是两个值的类型不同，要如何定义返回值？这就可以借助 `std::commom_type` 解决。下面定义一个 min 函数，返回 `T1, T2` 两个类型中的公共类型

```cpp
template<typename T1, typename T2>
typename std::common_type<T1, T2>::type min(const T1& x, const T2& y);
```

例如 `std::common_type<int, long>::type` 返回 `int`，而 `std::common_type<std::string, const char*>::type` 返回 `std::string` 。



标准库 `std::common_type` 内部结构为

```cpp
template <typename T1, typename T2>
struct common_type<T1, T2> {
	// 利用 ? : 获得公共类型
    typedef decltype(true ? declval<T1>() : declval<T2>()) type;
}
```

其中 `std::declval` 根据传入类型返回一个值，因此无论如何都会返回 `T1` 对应的类型。这需要保证 `T1, T2` 确实具有公共类型，例如整型的公共类型为 `int`，也就是说任何 `int, long` 传入都会返回 `int` 类型。



#### 变长参数类型

要实现多个类型的公共类型，可以利用模板递归。例如

```cpp
#include <iostream>

// 两个类型参数的通用类型
template <typename T0, typename T1> struct common_type_two {
    // 只适用于可加对象
    // using type = decltype(std::declval<T0>() + std::declval<T1>());
    using type = decltype(true ? std::declval<T0>() : std::declval<T1>());
};

// 作为顶层模板
template <class... Ts> struct common_type {};

// 偏特化，作为递归终止条件
template <class T0> struct common_type<T0> {
    using type = T0;
};

// 偏特化，提供递归条件
template <class T0, class T1, class... Ts> struct common_type<T0, T1, Ts...> {
    using type = typename common_type_two<T0, typename common_type<T1, Ts...>::type>::type;
};

int main() {
    typename common_type<int, double, float>::type x;
    return 0;
}
```



#### 函数式方法

可以使用函数方法实现相同的效果

```cpp
template <class T0, class... Ts>
constexpr auto get_common_type(T0 t0, Ts... ts) {
    if constexpr (sizeof...(ts) == 0) {
        return t0;
    } else {
        return decltype(true? t0 : get_common_type(ts...))();
    }
}

int main() {
    using what = decltype(get_common_type(1, 2.0, 3.14));
    std::cout << what() << std::endl;
    return 0;
}
```

但是这种写法会使用类的拷贝或移动，对于删除了拷贝构造的类就会出错

```cpp
struct Animal {};

struct Cat : Animal {
	// 同时会删除拷贝构造
    Cat(Cat&&) = delete;
};
```

解决方案是引入一个辅助结构，它只用来保存 `T` 类型，内部为空，可随意拷贝和移动

```cpp
#include <iostream>

template <class T> struct dummy {
	// 返回临时对象的辅助函数
    static consteval T declval() { return std::declval<T>(); }
};

template <class T0, class... Ts> constexpr auto get_common_type(T0 t0, Ts... ts) {
    if constexpr(sizeof...(ts) == 0) {
        return t0;
    } else {
    	// 返回 dummy 封装的结构
        return dummy<decltype(true ? t0.declval() : get_common_type(ts...).declval())>();
    }
}

struct Animal {};

struct Cat : Animal {
    Cat(Cat&&) = delete;
};

int main() {
    using what = decltype(get_common_type(dummy<Cat>(), dummy<Animal>()));
    return 0;
}
```



#### 识别容器

使用变长参数识别的 `commom_type` 结构，可以构造识别 `tuple` 类型中元素的公共类型的结构

```cpp
template <class Tup>
struct tuple_apply_common_type {

};

// 为 tuple 特化，利用 std::tuple<Ts...> 捕获 tuple，并分离出元素 Ts...
template <class ...Ts>
struct tuple_apply_common_type<std::tuple<Ts...>> {
	// 获得 Ts... 的公共类型
    using type = typename common_type<Ts...>::type;
};

int main() {
    using Tup = std::tuple<int, double, char>;
    using what = tuple_apply_common_type<Tup>::type;	// double
    return 0;
}
```

如果希望代码适配 `variant`，添加

```cpp
template <class ...Ts>
struct tuple_apply_common_type<std::variant<Ts...>> {
    using type = typename common_type<Ts...>::type;
};
```



注意到构造的类成员两边都是 `type`，因此可以直接继承

```cpp
template <class... Ts> struct tuple_apply_common_type<std::tuple<Ts...>> : common_type<Ts...> {};
template <class... Ts> struct tuple_apply_common_type<std::variant<Ts...>> : common_type<Ts...> {};
```



### invoke

借助 `invoke` 对函数调用情况进行分类。例如 `std::invoke_result_t` 获得函数调用后的返回类型，可以在递归调用中添加返回值来打断调用

```cpp
// 编译期循环
template <size_t Beg, size_t End, class F> void static_for(F f) {
    if constexpr(Beg < End) {
        // 绑定 Beg 到 i，类型为 size_t
        std::integral_constant<size_t, Beg> i;

        // 首先判断 f 能否用 i 作为参数，获得返回值类型，并判断返回值是否为 void
        if constexpr(std::is_void_v<std::invoke_result_t<F, decltype(i)>>)
            f(i);
        else if(f(i))
            return;

        static_for<Beg + 1, End>(f);
    }
}

static_for<0, 3>([&](auto i) {
    if(i.value == var.index()) {
        std::cout << "Index: " << std::get<i.value>(var) << std::endl;
        return true;
    }
    return false;
});
```

使用 `std::is_invocable_v` 判断函数是否具有指定输入参数类型

```cpp
template <size_t Beg, size_t End, class F> void static_for(F f) {
    if constexpr(Beg < End) {
        // 绑定 Beg 到 i，类型为 size_t
        std::integral_constant<size_t, Beg> i;

        // 控制循环
        struct Breaker {
            bool* broken = false;
            constexpr void static_break() const { *broken = true; }
        };

        // 检查 F 能否接收 i, Breaker
        if constexpr(std::is_invocable_v<F, decltype(i), Breaker>) {
            bool broken = false;
            f(i, Breaker{&broken});

            // 如果 f 调用了 Breaker::static_break，则退出循环
            if(broken) return;
        } else {
            // 如果 f 不能接收 i, Breaker，则直接调用 f
            f(i);
        }

        static_for<Beg + 1, End>(f);
    }
}

std::variant<int, double, char, bool> var = 1.0;
static_for<0, 3>([&](auto i, auto ctrl) {
    if(i.value == var.index()) {
        std::cout << "Index: " << std::get<i.value>(var) << std::endl;
        return ctrl.static_break();
    }
    return false;
});
```

> [!note]
> 在 C++20 中移除了 `std::result_of, std::result_of_t` 模板，被 `std::invoke_result, std::invoke_result_t` 系列模板替代。



### Type Trait

Type Trait 在 C++11 中被大幅度扩展，可以根据元素类型执行不同的操作。例如

```cpp
#include <type_traits>
 
template <typename T>
void foo(const T& val)
{
    if (std::is_pointer<T>::value) {
        std::cout << "foo() called for a pointer" << std::endl;
    }
    else {
        std::cout << "foo() called for a value" << std::endl;
    }
}

int main()
{
    int num;
    int *p = &num;
    
    foo(num);
    foo(p);
}
```

这里 `std::is_pointer` 属于一个 traits 类：

* 如果传入的参数类型为指针类型，其返回 `std::true_type`；
* 如果传入的参数类型不是指针类型，其返回 `std::false_type`；

通过 `::value` 获得对应的 `true` 或 `false` 值。这两个类型是 `std::integral_constant` 的特化

```cpp
namespace std {
    template <typename T, T val>
    struct integral_constant {
        static constexpr T value = val;
        typedef T value_type;
        typedef integral_constant<T, v> type;
        constexpr operator value_type() {
            return value;
        }
    };
    typedef integral_constant<bool, true> true_type;
    typedef integral_constant<bool, false> false_type;
}
```



#### 区分指针

根据上面的样例，我们可以给出一个根据传入元素是否为指针来执行不同输出操作的代码

```cpp
template <typename T>
void foo(const T& val)
{
    std::cout << (std::is_pointer<T>::value ? *val : val) << std::endl;
}
```

虽然在逻辑上成立，然而这段代码不能通过编译，因为在 val 不是指针时，`*val` 操作没有意义，编译器不能预先判断类型。所以需要进行改良

```cpp
#include <type_traits>

template<typename T>
void foo_impl(const T& val, std::true_type)
{
    std::cout << "foo() called for a pointer:" << *val << std::endl;
}

template<typename T>
void foo_impl(const T& val, std::false_type)
{
    std::cout << "foo() called for a value:" << val << std::endl;
}
 
template<typename T>
void foo(const T& val)
{
    // 将关于指针区分的代码分离到两个函数中
    foo_impl(val, std::is_pointer<T>());
}
```

这就是 `std::true_type` 和 `std::false_type` 的作用：**它们都是类型，所以可以作为重载函数的不同参数类型**。



#### 类型检测

Type Trait 提供了大量类型判断模板，例如

![[20200411151744849.png]]

还可以针对类的特性判断

![[20200411152328944.png]]



#### 判断关系

用来检测传入类型之间的关系

![[2020041115385278.png]]

基本数据类型（例如 int ）可能是左值或右值，不能直接赋值，因此只要 `std::is_assignable` 的第一个类型不是类，就一定返回 false_type 类型。不过指定了左值引用的类型可以赋值

```cpp
is_assignable<int, int>::value;		// false_type
is_assignable<int&, int>::value;	// true_type
is_assignable<int&&, int>::value;	// false_type
```

再例如通过 `is_same` 判断两个类型是否相同

```cpp
std::is_same<int, int>::value 		// 返回 true_type
std::is_same<int, double>::value 	// 返回 false_type
```

在 C++17 中引入了 `is_same_v` 用于直接获得类型值

```cpp
std::is_same_v<int, int>		// 返回 true_type
std::is_same_v<int, double> 	// 返回 false_type
```



#### 引用修饰

以下操作允许修改传入类型

![[2020041115531190.png]]

add_value_reference 把将右值引用转换为左值引用，但是 add_rvalue_reference 不能把左值引用转换为右值引用，需要先移除引用，然后转换为右值引用

```cpp
add_rvalue_reference<remove_reference<T>::type>::type;
```



#### 其它修饰

下面这些模板允许对类型进行更复杂的操作

![[20200411155558199.png]]

例如 rank 可以返回数组类型的维数，extent 获得数组的宽度，例如

```cpp
rank<int>::value;		// 0
rank<int[]>::value;		// 1
rank<int[5]>::value;	// 1
rank<int[][7]>::value;	// 2

extent<int>::value;			// 0
extent<int[]>::value;		// 0
extent<int[5]>::value;		// 5
extent<int[][7]>::value;	// 0
```



#### 萃取技术

萃取的目的是推导模板类型后提取类型。对于一个迭代器，它要应用于不同的容器，就需要针对不同的类型得到不同的返回值，这就需要推导返回值。一种实现方式如下

```cpp
template <class T> struct MyIter
{
    // 内嵌类型
    typedef T value_type;
    T *ptr;
    
    MyIter(T *p = 0) : ptr(p)
    {
    }
    
    // 取引用
    T &operator*() const
    {
        return *ptr;
    }
};

// 通过内嵌类型 value_type 推导返回值
template <class I> typename I::value_type func(const I& ite)
{
    return *ite;
}
```



我们需要从迭代器中萃取出对象的类型信息。然而，如果 T 是一个非 class 类型，例如指针和引用，就无法通过内嵌类型推导，解决方案是通过偏特化。典型的萃取器如下，它可以从迭代器、指针和指针常量中萃取出相应的类型

```cpp
// 自定义迭代器
template <class T> struct MyIter
{
    // 内嵌类型，用于萃取（有些暂时设为 T）
    typedef T value_type;
    typedef T iterator_category;
    typedef size_t difference_type;
    typedef T* pointer;
    typedef T& reference;
    
    MyIter(T *p = 0) : ptr(p)
    {
    }
    
    // 取引用
    T &operator*() const
    {
        return *ptr;
    }

    T *ptr;
};

// 主模板，用于萃取迭代器所指对象的类型
template <class Iterator> struct iterator_traits
{
    // 迭代器类型, STL 提供五种迭代器
    typedef typename Iterator::iterator_category iterator_category;

    // 迭代器所指对象的类型
    // 如果想与STL算法兼容, 那么在类内需要提供 value_type 定义
    typedef typename Iterator::value_type value_type;

    // 用于处理两个迭代器间距离的类型
    typedef typename Iterator::difference_type difference_type;

    // 直接指向对象的原生指针类型
    typedef typename Iterator::pointer pointer;

    // 对象的引用类型
    typedef typename Iterator::reference reference;
};

// 针对指针提供特化版本
template <class T> struct iterator_traits<T *>
{
    typedef random_access_iterator_tag iterator_category;
    typedef T value_type;
    typedef ptrdiff_t difference_type;
    typedef T *pointer;
    typedef T &reference;
};

// 针对指向常对象的指针提供特化
template <class T> struct iterator_traits<const T *>
{
    typedef random_access_iterator_tag iterator_category;
    typedef T value_type;
    typedef ptrdiff_t difference_type;
    typedef const T *pointer;
    typedef const T &reference;
};

// 统计容器里 value 的个数
template <class T, class I> typename iterator_traits<I>::difference_type count(I first, I last, const T &value)
{
    // 如果 I = MyIter<T>，这里 iterator_traits<I>::value_type 得到 I::value_type，即 value_type == T
    // 如果 I = T*，则直接得到 value_type == T（偏特化）
    // 如果 I = const T*，则直接得到 value_type == T（偏特化）
    typename iterator_traits<I>::value_type type;

    // ... 遍历计数
    typename iterator_traits<I>::difference_type n = 0;

    return n;
}
```



在 STL 中，针对多种基础类型都提供了对应的特化萃取器

```cpp
__STL_TEMPLATE_NULL struct __type_traits<bool> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

#endif /* __STL_NO_BOOL */

__STL_TEMPLATE_NULL struct __type_traits<char> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<signed char> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<unsigned char> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

#ifdef __STL_HAS_WCHAR_T

__STL_TEMPLATE_NULL struct __type_traits<wchar_t> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

#endif /* __STL_HAS_WCHAR_T */

__STL_TEMPLATE_NULL struct __type_traits<short> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<unsigned short> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<int> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<unsigned int> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<long> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<unsigned long> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};
```



## 智能指针
### auto_ptr

auto_ptr 表示智能指针类型，智能指针会自动销毁，不需要调用 delete 进行删除。除此之外，我们可以像使用普通指针一样使用它：

```cpp
auto_ptr<Person> p(new Person());
p->run(); // 直接调用对象的函数
```

上面智能指针 p 指向了堆空间的 Person 对象。



智能指针本质是一个新的对象封装了指针，我们可以尝试自定义智能指针

```cpp
template <typename T> class auto_ptr
{
  public:
    explicit auto_ptr(T *ptr = nullptr) noexcept : ptr_(ptr)
    {
    }

    ~auto_ptr() noexcept
    {
        delete ptr_;
    }

    // 重载取值操作
    T &operator*() const noexcept
    {
        return *ptr_;
    }

    // 重载 -> 操作
    T *operator->() const noexcept
    {
        return ptr_;
    }

    // 隐式转换为 bool 值
    operator bool() const noexcept
    {
        return ptr_;
    }

    // 获得内部指针
    T *get() const noexcept
    {
        return ptr_;
    }

    // 拷贝构造，被复制放释放原来指针的所有权，交给复制方
    auto_ptr(auto_ptr &other) noexcept
    {
        ptr_ = other.release();
    }

    // 拷贝并交换
    auto_ptr &operator=(auto_ptr &rhs) noexcept
    {
        // auto_ptr tmp(rhs.release());
        // tmp.swap(*this);
        auto_ptr(rhs.release()).swap(*this);
        return *this;
    }

    // 原来的指针释放所有权
    T *release() noexcept
    {
        T *ptr = ptr_;
        ptr_ = nullptr;
        return ptr;
    }

    // 转移指针所有权
    void swap(auto_ptr &rhs) noexcept
    {
        using std::swap;
        swap(ptr_, rhs.ptr_);
    }

  private:
    T *ptr_;
};

template <typename T> void swap(auto_ptr<T> &lhs, auto_ptr<T> &rhs) noexcept
{
    lhs.swap(rhs);
}
```

这基本就是最初的智能指针的核心实现。然而，它有一个巨大的隐患：如果一个智能指针进行赋值操作，它就丢失了。例如

```cpp
auto_ptr<int> aup(new int(5));
auto_ptr<int> aup2;
aup2 = aup;	// 此时 aup 失去了指针所有权
```

因此这一类型在 C++17 被删除了。 



最简单高效的解决方案，就是直接禁止智能指针的拷贝构造和直接赋值

```cpp
template <class T> class scoped_ptr
{
  public:
    explicit scoped_ptr(T *ptr = 0) noexcept : ptr_(ptr)
    {
    }

    ~scoped_ptr() noexcept
    {
        delete ptr_;
    }

    T &operator*() const noexcept
    {
        return *ptr_;
    }

    T *operator->() const noexcept
    {
        return ptr_;
    }

    T *get() const noexcept
    {
        return ptr_;
    }

    void swap(scoped_ptr &rhs) noexcept
    {
        using std::swap;
        swap(ptr_, rhs.ptr_);
    }

    // 重置指针
    void reset(T *p = 0) noexcept
    {
        scoped_ptr(p).swap(*this);
    }

  private:
    T *ptr_;

    // 隐藏拷贝和赋值
    scoped_ptr(scoped_ptr const &);
    scoped_ptr &operator=(scoped_ptr const &);
};

template <typename T> void swap(scoped_ptr<T> &lhs, scoped_ptr<T> &rhs) noexcept
{
    lhs.swap(rhs);
}
```



### unique_ptr

#### 自定义

在 `auto_ptr` 的基础上可以进一步得到唯一指针。例如

```cpp
// 删除器
template <class T> struct DefaultDeleter {
    void operator()(T* p) const { delete p; }
};

// 针对数组特化
template <class T> struct DefaultDeleter<T[]> {
    void operator()(T* p) const { delete[] p; }
};


template <class T, class Deleter = DefaultDeleter<T>> struct UniquePtr {
  private:
    T* m_p;

    // 友元声明，允许 UniquePtr<U> 转换为 UniquePtr<T>
    template <class U, class DU> friend struct UniquePtr;

  public:
    UniquePtr(std::nullptr_t = nullptr) noexcept: m_p(nullptr) {}
    explicit UniquePtr(T* p = nullptr) noexcept: m_p(p) {}

    UniquePtr(const UniquePtr&) = delete;
    UniquePtr& operator=(const UniquePtr&) = delete;

	// 移动构造
    template <class U, class DU>
        requires std::convertible_to<U*, T*>
    UniquePtr(UniquePtr<U, DU>&& other) {
        m_p = std::exchange(other.m_p, nullptr);
    }

	// 移动赋值
    template <class U, class DU>
        requires std::convertible_to<U*, T*>
    UniquePtr& operator=(UniquePtr<U, DU>&& other) {
        if(this != &other) [[likely]] {
            if(m_p != nullptr) Deleter{}(m_p);

            // 交换指针，并将 other 的指针置为 nullptr
            m_p = std::exchange(other.m_p, nullptr);
        }
        return *this;
    }
    
    ~UniquePtr() {
        if(m_p != nullptr) {
            Deleter{}(m_p);
        }
    }

    T* get() { return m_p; }

    T* release() { return std::exchange(m_p, nullptr); }

    void reset(T* p = nullptr) {
        if(m_p != nullptr) Deleter{}(m_p);
        m_p = p;
    }
	
	// 转移指针所有权
	void swap(unique_ptr &rhs)
	{
	    std::swap(ptr_, rhs.ptr_);
	}

    T& operator*() const { return *m_p; }

    T* operator->() const { return m_p; }

    operator bool() const { return m_p != nullptr; }
};

// 针对数组特化
template <class T, class Deleter> struct UniquePtr<T[], Deleter> : UniquePtr<T, Deleter> {};

// 构造指针
template <class T, class... Args> inline UniquePtr<T> make_unique(Args&&... args) {
    return UniquePtr<T>(new T(std::forward<Args>(args)...));
}
```

此时无法通过拷贝构造或赋值，所有类似操作只能通过移动进行，相较于 `auto_ptr` 的好处在于它显式声明了所属权的转移。



#### 标准定义

标准库中的 `unique_ptr` 使用如下

```cpp
std::unique_ptr<int> func()
{
    return std::unique_ptr<int>(new int(250));
}

int main()
{
    std::unique_ptr<int> ptr1 = func();
    std::unique_ptr<int> ptr2 = std::move(ptr1);
    return 0;
}
```

使用 `reset` 解除对内存的管理，也可以用于初始化 `unique_ptr`

```cpp
int main()
{
    std::unique_ptr<int> ptr1(new int(10));
    std::unique_ptr<int> ptr2 = std::move(ptr1);

    ptr1.reset();
    ptr2.reset(new int(250));	// 重新指定内存

    return 0;
}
```

通过 `get` 获取原始内存

```cpp
pointer get() const noexcept;
```



#### 删除器

在 `unique_ptr` 中指定删除器必须确定删除器类型

```cpp
int main() 
{
    using func_ptr = void (*)(int*);
    unique_ptr<int, func_ptr> ptr1(new int(10), [](int* p) { delete p; });

    return 0;
}
```

其中 `<int, func_ptr>` 就是在指定删除函数的类型。

> [!note] 类型擦除
> 使用 `unique_ptr` 创建的指针在删除时默认会按照模板类型删除，这就导致如果将子类指针保存在基类 `unique_ptr` 中，有可能发生内存泄漏。



#### make_unique

在 C++11 中忘记提供了 `make_unique` 方法，但是可以自行实现

```cpp
template <typename T, typename... Args> std::unique_ptr<T> make_unique(Args &&...args)
{
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}
```



### shared_ptr

shared_ptr 是一个模板类，可以使用构造函数、辅助函数以及 `reset` 方法进行初始化。通过成员函数 `use_count` 获取管理这块内存的智能指针数量

```cpp
long use_count() const noexcept;
```



#### 实现思路

一种实现思路是构造一个专门用于存放引用计数的类型

```cpp
class shared_count
{
  public:
    shared_count() : count_(1)
    {
    }

    // 增加计数
    void add_count()
    {
        ++count_;
    }

    // 减少计数
    long reduce_count()
    {
        return --count_;
    }

    // 获取当前计数
    long get_count() const
    {
        return count_;
    }

  private:
    long count_;
};
```

然后让每个智能指针都保存一份引用计数

```cpp
template <typename T> class shared_ptr
{
  public:
    // 声明所有其它模板实例均为友元
    template <typename U> friend class shared_ptr;

    explicit shared_ptr(T *ptr = nullptr) noexcept : ptr_(ptr)
    {
        // 创建新的引用计数
        if (ptr)
            shared_count_ = new shared_count();
    }

    ~shared_ptr() noexcept
    {
        // 最后一个 shared_ptr 再去删除对象与共享计数
        // ptr_ 不为空且此时共享计数减为 0 的时候，再去删除
        if (ptr_ && !shared_count_->reduce_count())
        {
            delete ptr_;
            delete shared_count_;
        }
    }

    // 拷贝构造，增加被拷贝的引用计数
    template <typename U> shared_ptr(const shared_ptr<U> &other) noexcept
    {
        ptr_ = other.ptr_;
        if (ptr_)
        {
            other.shared_count_->add_count();
            shared_count_ = other.shared_count_;
        }
    }

    // 移动构造，直接移动引用计数
    template <typename U> shared_ptr(shared_ptr<U> &&other) noexcept
    {
        ptr_ = other.ptr_;
        if (ptr_)
        {
            shared_count_ = other.shared_count_;
            other.shared_count_ = nullptr;
        }
    }

    // 始终只有一个对象有管理这块空间的权限
    shared_ptr &operator=(shared_ptr rhs) noexcept
    {
        rhs.swap(*this);
        return *this;
    }

    void swap(shared_ptr &rhs) noexcept
    {
        using std::swap;
        swap(ptr_, rhs.ptr_);
        swap(shared_count_, rhs.shared_count_);
    }

    // 获得引用计数
    long use_count() const noexcept
    {
        if (ptr_)
            return shared_count_->get_count();
        else
            return 0;
    }

  private:
    T *ptr_;
    shared_count *shared_count_;
};

template <typename T> void swap(shared_ptr<T> &lhs, shared_ptr<T> &rhs) noexcept
{
    lhs.swap(rhs);
}
```

值得注意的是，上面将拷贝构造和移动构造均声明为模板，因此会有默认的拷贝构造和移动构造，需要补上。

> 注意到不同智能指针保存的引用计数可能不同，但是没有问题。由于新的智能指针的引用计数总是大于旧的智能指针中的计数，这就确保了只要还存在多于一个智能指针，则所有智能指针的引用计数一定大于 1，所以不会销毁指针。



#### 初始化

通过构造函数初始化

```cpp
shared_ptr<int> ptr1(new int(250));		// 指向 int 变量
shared_ptr<char> ptr2(new char[12]);	// 指向字符数组
shared_ptr<int> ptr3;				   	// 空指针
shared_ptr<int> ptr4(nullptr);		    // 初始化为空
```

后两个智能指针不管理任何内存，因此 `use_count` 返回 0；需要注意，**不要使用一个普通指针初始化多个智能指针**。



通过拷贝和移动构造函数初始化

```cpp
shared_ptr<int> ptr1(new int(250));
// 使用拷贝构造函数初始化，count + 1
shared_ptr<int> ptr2 = ptr1;
// shared_ptr<int> ptr2(ptr1);
// 使用移动构造函数初始化，count 不变
shared_ptr<int> ptr3 = std::move(ptr1);
// shared_ptr<int> ptr3(std::move(ptr1));
```

使用移动构造只会转移内存所有权，管理内存的对象不会增加。



通过 `std::make_shared` 初始化

```cpp
template <class T, class... Args>
shared_ptr<T> make_shared(Args&&... args);
```

其中 args 就是要初始化的数据

```cpp
class Test
{
public:
    int a;
    Test() : a(0) {}
    Test(int a) : a(a) {}
};

shared_ptr<int> ptr1 = make_shared<int>(520);
shared_ptr<Test> ptr2 = make_shared<Test>();
shared_ptr<Test> ptr3 = make_shared<Test>(520);
```



通过 reset 方法初始化

```cpp
void reset() noexcept;

template< class Y >
void reset( Y* ptr );

template< class Y, class Deleter >
void reset( Y* ptr, Deleter d );

template< class Y, class Deleter, class Alloc >
void reset( Y* ptr, Deleter d, Alloc alloc );
```

其中 ptr 是要取得所有权的对象指针，d 是要失去所有权的对象指针，alloc 是内部储存所用的分配器

```cpp
shared_ptr<int> ptr1 = make_shared<int>(520);
shared_ptr<int> ptr2 = ptr1;
shared_ptr<int> ptr3 = ptr1;
ptr3.reset();
```

最后一步会清除 ptr3，内存引用计数减少 1 。



#### 原始指针

对应基础数据类型来说，通过操作智能指针和操作其管理的内存效果是一样的。但是如果共享智能指针管理的是一个对象，就需要取出原始内存的地址操作

```cpp
T* get() const noexcept;
```

通过 get 直接返回原始指针。



#### 删除器

当只能指针管理的内存对应的引用计数变为 0 时，此内存会被析构

```cpp
// 申请数组指针
shared_ptr<Person[]> p(new Person[5]);

// 指向同一个对象的指针
shared_ptr<Person> p0;
{
    shared_ptr<Person> p(new Person);
    shared_ptr<Person> p1 = p;
    shared_ptr<Person> p2 = p1;
    p0 = p;
}

// 到这里，p p1 p2都已销毁，但p0没有销毁，其指向的对象还存在
shared_ptr<Person> p3 = p0;

p1.use_count(); // 返回有多少个指向这一对象的指针
```

我们也可以自己指定删除动作，通过回调函数完成

```cpp
void deleteIntPtr(int* p)
{
    delete p;
    cout << "内存已析构" << endl;
}

int main()
{
    shared_ptr<int> ptr(new int(250), deleteIntPtr);
    return 0;
}
```

也可以写成 lambda 表达式

```cpp
int main()
{
    shared_ptr<int> ptr(new int(250), [](int*  p) {delete p;});
    return 0;
}
```



需要注意，由于 `shared_ptr` 默认删除器不支持对数组对象的删除，因此**在管理动态数组时必须指定删除器**

```cpp
int main()
{
    shared_ptr<int> ptr(new int[10], [](int* p) {delete[]p; });
    return 0;
}
```

C++ 也提供了默认用于删除数组的默认删除器

```cpp
int main()
{
    shared_ptr<int> ptr(new int[10], default_delete<int[]>());
    return 0;
}
```



#### 返回智能指针

如果在类中返回自身的 `share_ptr`，如下

```cpp
struct Test
{
    shared_ptr<Test> getThis()
    {
        return shared_ptr<Test>(this);
    }
};

int main()
{
    shared_ptr<Test> sp1(new Test);
    shared_ptr<Test> sp2 = sp1->getThis();
    return 0;
}
```

在执行上面代码时会出现问题，因为两个智能指针虽然指向同一块内存，但是由于 sp 2 不是通过 sp 1 初始化得到的实例对象，离开作用域后 this 将被两个智能指针各自析构。



我们可以通过继承 `std::enable_shared_from_this<T>` 类来解决

```cpp
#include <iostream>
#include <memory>
using namespace std;

struct Test : public enable_shared_from_this<Test>
{
    shared_ptr<Test> getThis()
    {
        // 这个函数是基类的成员函数，用于返回 this 的共享指针
        return shared_from_this();
    }
    ~Test()
    {
        cout << "class Test is disstruct ..." << endl;
    }
};

int main()
{
    shared_ptr<Test> sp1(new Test);
    shared_ptr<Test> sp2 = sp1->getThis();
    return 0;
}
```

其中 `enable_shared_from_this` 使用了 `weak_ptr` 在 `shared_from_this` 内部检测 `this` 对象，并通过 `lock` 方法返回 `shared_ptr` 对象。



### weak_ptr

#### 使用场景

shared_ptr 根据强引用的个数来判断是否应该销毁，当出现互相指向的智能指针，强引用个数至少为 1，从而无法销毁对象

```cpp
class Person
{
public:
	shared_ptr<Car> m_car;
}

class Car
{
public:
	shared_ptr<Person> m_person;
}

shared_ptr<Car> car(new Car);
shared_ptr<Person> person(new Person);

// 两个指针的成员指针分别指向对方
car->m_person = person;
person->m_car = car;
```

为了解决这一问题，可以使用 `weak_ptr` ，它不会增加强引用次数，因此可用于解决 `shared_ptr` 的循环引用问题。

![[image-20240405174701640.png]]

![[image-20240405174820845.png]]



弱引用智能指针不管理 `shared_ptr` 内部的指针，没有重载操作符 `*` 和 `->` ，因为它不共享指针，不能操作资源，主要作用是作为旁观者监视 `shared_ptr` 中的资源是否存在

```cpp
// 默认构造函数
constexpr weak_ptr() noexcept;

// 拷贝构造
weak_ptr (const weak_ptr& x) noexcept;
template <class U> weak_ptr (const weak_ptr<U>& x) noexcept;

// 通过 shared_ptr 对象构造
template <class U> weak_ptr (const shared_ptr<U>& x) noexcept;
```

我们不仅可以声明 `weak_ptr`，还可以通过 `shared_ptr` 构造和隐式转化为 `weak_ptr`

```cpp
weak_ptr<int> wp1(new int(250));
shared_ptr<int> wp2(new int(20));
weak_ptr<int> wp3 = wp2;	// 隐式转换
```



#### 成员函数

通过 `use_count` 获取当前检测数据的引用计数

```cpp
// 函数返回所监测的资源的引用计数
long int use_count() const noexcept;
```

虽然不使用引用计数，但是可以通过 `expired` 方法判断观测的资源是否被释放

```cpp
bool expired() const noexcept;
```

通过 `lock` 方法获取管理所监测资源的 `shared_ptr` 对象

```cpp
shared_ptr<element_type> lock() const noexcept;
```

通过 reset 方法清空对象，使其不监测任何资源。



## Lambda 表达式

Lambda 表达式是 functor 的一种简便写法，意在构造一个简单的函数对象（ lambda 表达式要求函数体是 inline function ）。

> Lambda 表达式默认为 constexpr 类型。

其定义式如下：

```cpp
[capture list] (params list) mutable exception -> return type { function body }
```

参数解释：

|     参数      |                         作用                         |
| :-----------: | :--------------------------------------------------: |
| capture list  |       捕获外部变量列表，默认只捕获变量当时的值       |
|  params list  | 形参列表（可省略），不能使用默认参数，不能省略参数名 |
|    mutable    |           是否可以修改捕获的变量（可省略）           |
|   exception   |                是否抛出异常（可省略）                |
|  return type  |                 返回值类型（可省略）                 |
| function body |                        函数体                        |

Lambda 表达式没有默认构造，也没有赋值操作。



看下面一段 lambda 表达式：

```C++
int main()
{
    int x = 0;
    auto [x]() mutable
    {
        x++;
        return x;
    }
}
```

这一段 lambda 的作用和下面的 functor 类似：

```c++
class Functor
{
private:
    int x;

public:
    int operator()()
    {
        x++;
        return x;
    };
};
```

比较下面三段 lambda 表达式代码：

```C++
auto lambda1 = [x]() mutable
{
    cout << x << endl;
    return ++x;
};
auto lambda2 = [&x]()
{
    cout << x << endl;
    return ++x;
};
auto lambda3 = [x]()
{
    cout << x << endl;
    return ++x;
};
```

`lambda3` 无法通过编译，因为函数体内改动了捕获值 `x` ，却没有 `mutable` 标记，从而引发了编译错误。



为了便于理解，我们展示几种基本用法

```cpp
int main() {
	// 定义一个 lambda 表达式
    [] 
    {
        cout << "Hello" << endl;
    };
    
    // 调用 lambda 表达式
    ([] {
        cout << "Hello" << endl;
    })();
    
    // 存放 lambda 表达式
    void (*p)() = [] 
    {
        cout << "Hello" << endl;
    };

	// 调用存放的函数
	(*p)();
    
    //使用auto变量来省略指针类型
    auto q = [] (int a, int b) -> int 
    {
        return a + b;
    };
}
```



### 捕获参数

#### 值捕获

lambda 表达式可以捕获外部参数，默认的值捕获仅仅获得当时参数的值

```cpp
int x = 5, y = 15;

auto p = [x, y] 
{ 
    cout << x + y << endl;
};
```

更一般的，可以简单地添加参数来实现对所有外部参数的捕获，称为匿名捕获。例如这是捕获所有外部参数的值

```cpp
auto p = [=] 
{
    cout << x + y << endl;
};
```



#### 地址捕获

地址捕获可以随时获得和修改参数的值

```cpp
auto p = [&x , &y] 
{
    x = 5, y = 10;
    cout << x + y << endl;
};
```

也可以捕获所有外部参数的引用

```cpp
auto p = [&] 
{
    cout << x + y << endl;
};
```



#### 混合捕获

当然，有时候需要特别指定某些参数需要捕获引用，有些参数只捕获值

```cpp
// x 地址捕获，y 值捕获
auto p = [&x, y] 
{
    return x;
};

// x 值捕获，其余地址捕获
auto p = [&, x] 
{
    return x;
};

// x 地址捕获，其余值捕获
auto p = [=, &x] 
{
    return x;
};
```



#### 右值捕获

在 C++14 中，允许捕获的成员进行初始化，这就允许了右值捕获

```cpp
#include <iostream>
#include <memory>  // std::make_unique
#include <utility> // std::move

void lambda_expression_capture()
{
    // 注意这是一个 unique 指针，正常来说不能被值捕获
    auto important = std::make_unique<int>(1);
    
    // 转移为右值捕获
    auto add = [v1 = 1, v2 = std::move(important)](int x, int y) -> int { return x + y + v1 + (*v2); };
    std::cout << add(3, 4) << std::endl;
}
```



#### auto 捕获

在 C++14 中允许用 auto 作为参数传入，此时参数类型根据实参推导。事实上，这相当于将 lambda 表达式模板化

```cpp
class A
{
  public:
    std::string name()
    {
        return "I am A.";
    }
};

class B
{
  public:
    std::string name()
    {
        return "I am B.";
    }
};

int main()
{
    A a;
    B b;
	
    // auto 相当于模板，可以匹配任意类型
    auto f = [](auto &t) -> decltype(t.name()) { return t.name(); };
    std::cout << f(a) << std::endl;
    std::cout << f(b) << std::endl;
    return 0;
}
```



#### mutable

通过 mutable 选项可以修改捕获参数在函数内部的值，不改变外部的值

```cpp
auto p = [x] () mutable 
{ 
    x++;
    return x;
};
```



#### 非捕获

非局部变量或以常量表达式初始化的引用不需要捕获就可以使用。例如

```cpp
static int a = 42;
const int N = 10;
auto p = [=] { ++a; int arr[N]{}; };
std::cout << sizeof p << '\n';
p();
```

此时 `a,N` 不被 lambda 表达式捕获，但是仍然可以使用对应的值。在 `p()` 执行后，`a` 为 43；并且由于 `p` 没有捕获值，因此它是一个空类，只占用 1 字节。



如果一个变量没有 ODR 使用，则它不会被捕获。例如

```cpp
float x;
float& r = x;
auto p = [=] {};
std::cout << sizeof p << '\n';
```

此时 `p` 仍然是空类，因为前面的两个变量都没有 ODR 使用。



### 异常声明

可以给 lambda 表达式添加 `noexcept` 修饰

```cpp
auto p = [] () noexcept {};
```



### 转换函数指针

不带有捕获的 lambda 表达式可以转换为函数指针

```cpp
using func_type = void (*)(int);
func_type fp = [](int) {};
```

带有捕获的 lambda 表达式需要通过 function 包装器封装

```cpp
function<void(int)> fp = function<void(int)>([&](int) {});
```



### 泛型 lambda

C++14 新特性允许参数为 auto 类型

```cpp
auto p = [] (auto a, auto b) -> int 
{
    return a + b;
};
```

使用这种形式，主要是为了配合模板使用。例如

```cpp
template <typename T>
auto multiply(T x, T y) -> decltype(x * y)
{
	return x * y;
}
```

在使用模板参数的情况下，不清楚返回值的具体类型，就需要通过 `-> decltype(x * y)` 来获得返回类型。



### 模板形参

C++20 允许显式提供模板形参

```cpp
 auto q = []<typename T>(T a, T b) -> T 
 { 
 	return a + b; 
 };
```



### 内部结构

使用 [cpp 编译展开](https://cppinsights.io/)工具可以看到 lambda 表达式的内部构造方法。实际上 lambda 表达式是一个无名非联合非聚合类型

```cpp
auto p = [] {};
auto q = [num = 0] {};
std::cout << sizeof p << '\n';
std::cout << sizeof q << '\n';
```

这里 `p` 是一个空类，因此只占用 1 字节；而 `q` 包含一个私有成员 `num`，占用 4 字节。



考虑下面的结构体

```cpp
struct X : decltype([] { std::cout << "Hello, world!\n"; }) {};
```

它继承一个 lambda 表达式，这个表达式是一个空类，因此这个结构体也是空结构体。我们可以使用这个结构体

```cpp
X x;
x();
```

此时 `x()` 调用的是 `operator()` 成员，也就是前面的 lambda 表达式。



默认情况下，lambda 表达式的 `operator()` 被 `const` 修饰，这就是为什么在 lambda 表达式内部不能修改捕获值

```cpp
int main()
{
  int n = 10;
  auto p = [n] () {};
}
```

上述代码编译展开为

```cpp
int main()
{
  int n = 10;
    
  class __lambda_6_12
  {
    public: 
    inline /*constexpr */ void operator()() const
    {
    }
    
    private: 
    int n;
    
    public:
    __lambda_6_12(int & _n)
    : n{_n}
    {}
    
  };
  
  __lambda_6_12 p = __lambda_6_12{n};
  return 0;
}
```

注意到 `n` 被转换为一个类的私有成员，并且 `operator()` 被 `const` 修饰。



然而，只需要添加 `mutable` 修饰，就可以修改捕获值

```cpp
int main()
{
  int n = 10;
  auto p = [n] () mutable {};
}
```

上述代码编译展开为

```cpp
int main()
{
  int n = 10;
    
  class __lambda_6_12
  {
    public: 
    inline /*constexpr */ void operator()()
    {
    }
    
    private: 
    int n;
    
    public:
    __lambda_6_12(int & _n)
    : n{_n}
    {}
    
  };
  
  __lambda_6_12 p = __lambda_6_12{n};
  return 0;
}
```

此时可以在 lambda 表达式内部修改 `n` 的值。



## constexpr

一般使用的 const 关键字表示变量只读，但并不意味着变量值不变

```cpp
int a = 10;
const int &b = a;
```

这里 b 是常量引用，不能修改 b ，但是可以修改 a ，从而 b 的值也会随之改变，



### 常量表达式

constexpr 用于修饰常量表达式。常量表达式是有多个常量组成并且在**编译过程中就得到计算结果的表达式**，这节约了程序每次运行时的开销。建议**凡是表达只读语义的场景使用 const ，而常量语义的场景都使用 constexpr** 。在定义常量时，两者等价，都可以在程序的编译阶段计算出结果

```cpp
const int m = f();			// 不是常量表达式，m 在运行时才会获得
const int i = 520;			// 常量表达式
const int j = i + 1;		// 常量表达式

constexpr int i = 520;		// 常量表达式
constexpr int j = i + 1;	// 常量表达式
```

对于 C++ 内置类型数据，可以直接用 constexpr 修饰，但是自定义数据类型则不行

```cpp
// 无效修饰
constexpr struct Test
{
    int id;
    int num;
};
```

如果要定义常量，应该这样写

```cpp
struct Test
{
    int id;
    int num;
};

int main()
{
    constexpr Test t{1, 2};
    return 0;
}
```



### 常量表达式函数

用 constexpr 修饰函数返回值的函数称为常量表达式函数。



#### 修饰函数

constexpr 不能修改任意函数的返回值，应当满足以下条件：

1. 函数有返回值，且 return 的表达式是常量表达式
2. 函数使用前有对应的定义语句，即**函数定义要在函数调用之前**
3. 整个函数中**不能出现常量表达式以外的语句**，除了 `using, typedef, static_assert, return` 语句

以上规则对修饰普通函数和类成员函数都有效。例如

```cpp
// 先定义函数，函数中没有非常量表达式
constexpr int func()
{
    using mytype = int;
    constexpr mytype a = 100;
    constexpr mytype b = 15;
    return a + b - 100;
}

int main()
{
    constexpr int num = func();
    return 0;
}
```

再例如

```cpp
constexpr int foo(int i)
{
    return i + 10;
}

int main()
{
    // 由于 foo(2) 作为常量，可以在编译阶段得到 12，因此可以使用
    std::array<int, foo(2)> arr;

    int i = 10;
    foo(i); // Call is Ok
	
    // 由于 i 的值在运行时才确定，因此 foo(i) 不能作为模板参数使用
    std::array<int, foo(i)> arr1;
}
```

而如果将 `i` 也改为常量表达式，就可以将 `foo(i)` 作为模板参数

```cpp
constexpr int i = 10;
std::array<int, foo(i)> arr1;
```

这里实际上也可以使用 `const` 类型。



在 C++14 之后，可以在常量表达式函数中使用局部变量、循环和分支

```cpp
constexpr int fibonacci(const int n)
{
    if (n == 1)
        return 1;
    if (n == 2)
        return 1;
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```



#### 修饰模板函数

虽然 constexpr 可以修饰函数模板，但是由于类型不确定，如果函数实例化结果不满足上面的要求，则 constexpr 会被自动忽略

```cpp
#include <iostream>

using namespace std;

struct Person
{
    const char *name;
    int age;
};

template <class T>
constexpr T display(T t)
{
    return t;
}

int main()
{
    // p 不是常量，此时 display 的 constexpr 无效
    Person p{"Li", 12};
    Person ret = display(p);

    // 常量表达式函数
    constexpr int ret = display(50);

    // 常量对象，常量表达式函数
    constexpr Person p1{"Liu", 13};
    constexpr Person ret1 = display(p1);

    return 0;
}
```



#### 修饰构造函数

如果想直接得到一个常量对象，则可以用 constexpr 修饰构造函数。构造函数的函数体必须为空，并且必须使用初始化列表对各个成员赋值

```cpp
#include <iostream>

using namespace std;

class Person
{
public:
    const char *name;
    int age;

public:
    constexpr Person(const char *name, int age) : name(name), age(age) {}
};

int main()
{
    constexpr Person p("Li", 15);
    return 0;
}
```



#### 修饰成员函数

通过修饰成员函数，可以获得对应的常量表达式

```cpp
#include <iostream>

class test3
{
  public:
    int value;

    constexpr int getvalue() const
    {
        return (value);
    }

    constexpr test3(int Value) : value(Value)
    {
    }
};

int main()
{
    constexpr test3 x(100);
    
    // 这里得到常量表达式，因此可以通过编译
    int array[x.getvalue()];
}
```



### 常量条件句

#### 区分返回类型

在 C++17 中可以将 `constexpr` 引入 if 条件语句。例如

```cpp
#include <iostream>

template <typename T> auto print_type_info(const T &t)
{
    if constexpr (std::is_integral<T>::value)
        return t + 1;
    else
        return t + 0.001;
}

int main()
{
    std::cout << print_type_info(5) << std::endl;
    std::cout << print_type_info(3.14) << std::endl;
}
```

这使得编译器可以在编译阶段完成条件计算，得到下面的等价代码

```cpp
int print_type_info(const int &t)
{
    return t + 1;
}

double print_type_info(const double &t)
{
    return t + 0.001;
}

int main()
{
    std::cout << print_type_info(5) << std::endl;
    std::cout << print_type_info(3.14) << std::endl;
}
```



#### 区分返回值

使用常量条件句，配合 type trait 实现根据是否有返回值分别实例化。例如

```cpp
#include <iostream>

template <typename F> auto func(F f) {
    if constexpr (std::is_void_v<F>) {
        f(); 
    } else {
        return f();
    }
}

int main() {
    func([] { std::cout << "Hello, world!" << std::endl; });
    func([] { return 42; });
    return 0;
}
```



#### 匹配成员

配合 requires 子句，根据模板是否具有对应的成员函数分别实例化。例如

```cpp
struct Dog {
    void bark() { std::cout << "Woof!" << std::endl; }
};

struct Cat {
    void meow() { std::cout << "Meow!" << std::endl; }
};

struct Human {
    void speak(int message) { std::cout << message << std::endl; }
};

template <typename T> auto func(T t) {
    if constexpr(requires { t.bark(); }) {
        t.bark();
    } else if constexpr(requires { t.meow(); }) {
        t.meow();
    } else if constexpr(requires { t.speak(std::declval<int>()); }) {
        t.speak(42);
    } else {
        std::cout << "I don't know what to do with this object." << std::endl;
    }
}
```



对于 C++20 以前的版本，可以自定义检测。利用默认模板参数和偏特化实现

```cpp
template <typename T, typename U = void> struct has_bark {
    static constexpr bool value = false;
};

// 注意偏特化写在 has_bark 的 <> 中
template <typename T> struct has_bark<T, std::void_t<decltype(std::declval<T>().bark())>> {
    static constexpr bool value = true;
};

template <typename T> auto func(T t) {
    if constexpr(has_bark<T>::value) {
        t.bark();
    } else {
        std::cout << "I don't know what to do with this object." << std::endl;
    }
}
```

其中 `std::void_t` 只判断模板表达式能否编译通过。



### 编译期求值

在 C++20 引入 `consteval` 修饰符，表示一个函数必须在编译期完成求值。例如

```cpp
constexpr int add(int x, int y) {
    return x + y;
}

consteval int sub(int x, int y) {
    return x - y;
}

int main() {
    constexpr int ca = add(1, 2);
    int a = add(1, 2);
    
    constexpr int cb = sub(1, 2);
    int b = sub(1, 2);
    return 0;
}
```

与 `constexpr` 不同，如果在运行期调用则会报错。



## 引用元素

通常所有容器都会建立元素拷贝，返回的也是元素的拷贝。STL 只支持 value 语义，不支持 reference 语义。使用 reference 可以在多个不同的容器中管理同一份对象。



### 智能指针引用

我们使用 `shared_ptr` 管理指针对象，首先定义一个类和输出任意容器中的类信息的模板函数

```cpp
class Item {
private:
    std::string name;
    float price;
public:
    Item(const std::string& n, float p = 0) :name(n), price(p) {}
    
    std::string getName()const { return name; }
    void setName(const std::string& n) { name = n; }

    float getPrice()const { return price; }
    void setPrice(float p) { price = p; }
};

template <typename Coll>
void printItems(const std::string& msg, const Coll& coll)
{
    std::cout << msg << std::endl;
    for (const auto& elem : coll)
    {
        std::cout << "   " << elem->getName() << ":" << elem->getPrice() << std::endl;
    }
}
```

测试程序

```cpp
int main()
{
    // 定义两种容器来保存智能指针
    typedef std::shared_ptr<Item> ItemPtr;
    std::set<ItemPtr> allItems;
    std::deque<ItemPtr> bestsellers;
    
    bestsellers = { ItemPtr(new Item("Kong Yize",20.10)),
        ItemPtr(new Item("A Midsummer Night's Dream",14.99)),
        ItemPtr(new Item("The Maltese Falcon",9.88))
    };
    allItems = { ItemPtr(new Item("Water",0.44)),
        ItemPtr(new Item("Pizza",2.22)) 
    };
    allItems.insert(bestsellers.begin(), bestsellers.end());

    // 输出容器中的元素
    printItems("bestsellers", bestsellers);
    printItems("allItems", allItems);
    std::cout << std::endl;

    // 将 bestsellers 中的价格翻倍
    std::for_each(bestsellers.begin(), bestsellers.end(), 
        [](std::shared_ptr<Item>& elem) {elem->setPrice(elem->getPrice() * 2);}
    );
    // 将 bestsellers[1] 的元素更改为 allItems 内名为 “Pizza” 的元素
    // 找到元素的智能指针，取值后赋值
    bestsellers[1] = *(
        find_if(allItems.begin(), allItems.end(), [](std::shared_ptr<Item> elem) {return elem->getName() == "Pizza"; })
     );
    // 将 bestsellers[0] 对象的 price 设置为 44.77
    bestsellers[0]->setPrice(44.77);

    // 输出容器中的元素
    printItems("bestsellers", bestsellers);
    printItems("allItems", allItems);
}
```

使用智能指针会使事情变得复杂，例如容器无法使用 `find` 算法，因为智能指针比较不会比较其内部储存的指针，所以需要用 `find_if` 替代。



### Reference 封装

在 `<functional>` 中定义了模板类型 `std::reference_wrapper` 将传值改为传引用。以及操作

* `std::ref` 将类型 `T` 隐式转换为 `T&`；
* `std::cref` 将类型 `T` 隐式转换为 `const T&`；

例如下面将 num2 用 `std::ref` 封装传入函数，则相当于将 num2 的引用传入，函数调用后 num2 值增加。

```cpp
template<typename T>
void foo(T val) { val++; }

int main()
{
    int num1 = 1, num2 = 1;
    foo(num1);
    std::cout << "num1:" << num1 << std::endl;

    foo(std::ref(num2)); // 改为 T& 调用
    std::cout << "num2:" << num2 << std::endl;
}
```

在 C++ 标准库中大量使用了这一特性。例如

* `make_pair()` 用此特性创建引用元素的 pair
* `make_tuple()` 用此特性创建引用元素的 tuple
* `Binder` 用此特性绑定引用
* `Thread` 用此特性以引用形式传递实参

使用 `reference_wrapper` 可以将元素引用存放在容器中。例如

```cpp
int main()
{
    // 定义引用封装元素的向量容器
    std::vector<std::reference_wrapper<Item>> books;
    
    // 存入 f 的引用
    Item f("Faust", 12.99);
    books.push_back(f);
	
    // 输出所有元素信息：通过 .get() 方法获得引用元素
    for (const auto& book : books) {
        std::cout << book.get().getName() << ": " << book.get().getPrice() << std::endl;
    }

    // 修改 f 也会改变容器中的值
    f.setPrice(9.99);
    std::cout << books[0].get().getPrice() << std::endl;

    // 可以看到 vector 内元素也改变了
    for (const auto& book : books) {
        std::cout << book.get().getName() << ": " << book.get().getPrice() << std::endl;
    }
}
```

这种做法的优点是不需要指针语法，并且可以使用 `find` 查找元素，但是需要注意被引用元素的生命周期问题。



## 右值及右值引用

### 基本介绍

右值引用标记为 `&&` ，我们首先要知道什么是左值和右值

* 左值是指储存在内存中、有明确的储存地址（可取地址）的数据
* 右值是指可以提供数据值的数据（不可取地址）

一般来说，可以认为左值就是应当出现在操作符左边的值，右值则是应当出现在操作符右边的值。

```cpp
int a = b;
```

这里 a 是左值， b 是“右值”。这意味着左值是持续存在的，而右值**只需要暂时存在**。上面的 b 也可以是持续存在的左值，只不过它在这里作为”右值“使用。于是，可以得到左右值的特点：

* 左值：能对表达式**取地址**、或**具名对象/变量**，一般指表达式结束后依然存在的持久对象
* 右值：**不能对表达式取地址**，或**匿名对象**，一般指表达式结束就不再存在的临时对象

由于右值引用本身是引用，因此**右值引用本身是左值**。



#### 纯右值

纯右值（**prvalue**，PureRvalue）：**按值返回的临时对象**、**运算表达式产生的临时对象**、**原始字面量**和 **lambda 表达式**等。



字面量（10, true）通常为纯右值，除了字符串字面量（"hello"）为左值。例如

```cpp
// 纯右值
int &&value = 520;

// 定义左值引用变量 str，长度为 6
const char(&str)[6] = "01234";
std::cout << std::is_lvalue_reference<decltype("hello")>::value << std::endl;
```



数组可以被隐式转换为对应的指针类型，例如

```cpp
// 右值引用
const char *p = "01234";
const char* &&pr = "01234";
const char* &prr = "01234"; // error
```



#### 将亡值

将亡值（**xvalue**，eXpiring value）：生命期即将结束的值，**一般是跟右值引用相关的表达式**，这样表达式通常是将要被移动的对象，是**被显式标记为右值的对象**。



例如函数的返回值。传统上，返回对象会被复制一份，赋值给接收变量，然而这可能造成额外的开销。在 C++11 之后，下面返回左值的代码将会被隐式转换为右值，进而后续赋值给 t2 的操作会进行移动，从而避免了赋值的开销

```cpp
class Test
{
public:
    Test() {}
    Test(const Test &a) {}
};

Test getRObj()
{
    // 返回将亡值，编译器优化进行移动
    return Test();
}

Test getLObj()
{
    // 返回左值
    Test t;
    return t;
}

int main()
{
    // 获得右值引用，右值的生命周期延长
    Test &&t1 = getRObj();
    
    // 返回的左值进行隐式转换为右值，进行移动
    Test t2 = getLObj();
    
    // 万能引用，正确
    const Test &t = getRObj();

    return 0;
}
```

可以看到，由于 getRObj 返回右值，只能使用右值引用；而 `const &` 常量左值引用是一个万能引用，它可以接受左值、右值、常量左值和常量右值。



非常量引用不允许绑定到非左值，是因为存在逻辑错误。例如

```cpp
void increase(int &v)
{
    v++;
}

void foo()
{
    double s = 1;
    increase(s);
}
```

由于 double 不能被 int 引用，因此会产生一个右值来保存 s 的值，但是对临时值的修改不会体现在 s 上，它没有被修改，导致函数错误。

> 常量引用可以绑定到非左值，是因为 Fortran 需要。



#### 右值参数

如果右值引用作为函数参数传入，则具名函数参数成为左值

> 这就是因为右值引用作为引用，本身是左值。

```cpp
void func(int&& a)
{
    // 此时 v 是左值类型
    decltype(a) v = 100;
}
```



#### 返回右值

对于返回**确定右值**的函数，如果外部变量接收这个右值，则编译器会自动进行优化。例如

```cpp
class A
{
  public:
    A()
    {
        std::cout << "Create A\n";
    }

    A(const A &)
    {
        std::cout << "Copy A\n";
    }

    A(A &&)
    {
        std::cout << "Move A\n";
    }
};

// 编译器可以优化 返回值移动出去
A getA_unnamed()
{
    return A();
}

// 编译器可以优化 返回值移动出去
A getA_named() 
{
    A a;
    return a;
}

// 编译器无法优化
A getA_duang()
{
    A a1;
    A a2;
	
    // 随机返回
    if (rand() > 42)
        return a1;
    else
        return a2;
}

// 编译器无法优化
A getA_move()
{
    A a;
    return std::move(a);
}

int main()
{
    // 返回的右值直接变为外部变量 a，不需要额外调用任何构造函数。
    A a = getA_unnamed();
    A aa = getA_named();
    
    // 不进行优化
    A ar = getA_duang();
    A am = getA_move();
    return 0;
}
```



### && 的特性

并非所有情况下 `&&` 都代表右值引用。如果模板参数使用 `T&&` 或者自动推导使用 `auto&&` ，此时 `&&` 被称作未定的引用类型。另外需要注意 `const T&&` 表示右值引用而不是未定引用类型。



#### 右值参数

在传统 C++ 中，不能对一个引用继续引用。而右值引用的出现放宽了这一点，遵循如下规则

| 形参类型 | 实参类型 | 推导形参类型 |
| -------- | -------- | ------------ |
| `T&`     | 左引用   | `T&`         |
| `T&`     | 右引用   | `T&`         |
| `T&&`    | 左引用   | `T&`         |
| `T&&`    | 右引用   | `T&&`        |

也就是说，除非传入的实参是右值引用，并且形参类型为右值引用，否则形参都被推导为左值引用。



在模板函数中使用

```cpp
template <class T>
void f(T&& param);
void f1(const T&& param);

f(10);	// 传入右值 T&& 表示右值引用
int x = 10;
f(x);	// 传入左值 T&& 表示左值引用
f1(x);	// const T&& 是右值引用，不需要推导
```



#### 引用坍缩

使用 `auto&&` 自动推导类型 

```cpp
int x = 520, y = 1314;
auto&& v1 = x;		// 左值引用
auto&& v2 = 250;	// 右值引用

// 等价于 int&& v3 = y ，错误
decltype(x)&& v3 = y
```



由于上面代码中存在未定引用类型，当它作为参数时，有可能被右值或左值引用初始化，在进行类型推导时右值引用 `&&` 会发生变化，这种变化称为**引用坍缩**：

* 通过右值推导 `T&&` 或 `auto&&` 得到右值引用
* 通过非右值（右值引用、左值、左值引用、常量右值引用、常量左值引用）推导 `T&&` 或 `auto&&` 得到左值引用

```cpp
int&& a1 = 5;		// a1 右值引用，但 a1 本身为左值
auto&& bb = a1;		// 右值引用推导 bb 为左值引用

int a2 = 5;
int &a3 = a2;		// a3 左值引用，本身为左值
auto&& cc = a3;		// 左值引用推导 cc 为左值引用
auto&& cc1 = a2;	// 左值推导 cc1 为左值引用

const int& s1 = 100;	// 常量左值引用
const int&& s2 = 100;	// 常量右值引用
auto&& dd = s1;			// 常量左值引用推导 dd 为常量左值引用
auto&& ee = s2;			// 常量右值引用推导 ee 为常量左值引用

const auto&& x = 5;		// 右值引用，不需要推导
```



在使用循环时，通过 `auto&&` 进行自动推导是最安全的选择。因为无论循环元素类型是右值引用还是左值引用，与 `&&` 结合坍缩后都相当于完美转发

```cpp
std::vector<int> v = {1, 2, 4};
for (auto &&a : v)
    std::cout << a << std::endl;
```



### 转移和完美转发

#### move

在 C++11 添加了右值引用，并且不能使用左值初始化右值引用，如果想用左值初始化右值引用，需要借助 `std::move` 方法，它可以将左值转换为右值。使用这个函数并不能移动任何东西，而是和构造函数一样都具有移动语义，将对象的状态或者所有权从一个对象转移到另一个对象，只是转移，没有内存拷贝。



事实上 `std::move` 基本等同于一个类型转换 `static_cast<T&&>(lvalue)` ，使用方法如下

```cpp
class Test
{
public:
    Test() {}
};

int main()
{
    Test t;
    Test&& v1 = t;	// 错误
    Test&& v2 = move(t);
    return 0;
}
```

使用左值初始化右值是错误的，因此需要使用 move 进行转移。



如果一个临时容器很大，并且需要将整个容器赋值给另一个容器，可以使用

```cpp
list<string> ls;
ls.push_back("a"); 
// ...
list<string> ls1 = ls;	// 使用拷贝，效率很低
list<string> ls2 = move(ls);
```

使用 move 几乎没有任何代价，只是转移了资源所有权。如果一个对象内部有较大的堆内存或者动态数组时，使用 move 可以高效地实现数据所有权的转移。另外，我们也可以给类编写相应的移动构造函数和具有移动语义的赋值函数

```cpp
class T
{
public:
    T(T&& another) {}
	T&& operator=(T&& rhs) {}
};
```



#### forward

右值引用类型独立于值，一个右值引用作为函数参数的形参时，**在函数内部转发该参数给内部其它函数时，它就变成一个左值**。如果需要按照参数原来的类型转发到另一个函数，就可以使用 `std::forward<T>` 函数

* 当 T 是左值引用，转换为 T 类型的左值
* 当 T 不是左值引用，转化为 T 类型的右值

```cpp
#include <iostream>

using namespace std;

template <class T>
void printValue(T &t)
{
    cout << "l-value: " << t << endl;
}

template <class T>
void printValue(T &&t)
{
    cout << "r-value: " << t << endl;
}

template <class T>
void testForward(T &&v)
{
    printValue(v);
    printValue(move(v));
    printValue(forward<T>(v));
    cout << endl;
}

int main()
{
    // 形参为未定引用类型，实参为右值，推导为右值引用 v
    testForward(100);	
    // printValue(v); 已命名的右值 v 视为左值传入
    // printValue(move(v)); v 被转换为右值传入
    // printValue(forward<T>(v)); 模板参数为右值引用，转换为右值传入
    
    int num = 10;
    
    // 模板类型左值引用，num 转为左值，推导 v 为左值引用
    testForward(forward<int&>(num));
    // printValue(v); 左值
    // printValue(move(v)); 左值 v 被转换为右值传入
    // printValue(forward<T>(v)); 模板参数为左值引用，传入左值

    // 模板类型右值引用，num 转为右值，推导 v 为右值引用
    testForward(forward<int&&>(num));
    // printValue(v); 已命名的右值 v 视为左值传入
    // printValue(move(v)); 左值 v 被转换为右值传入
    // printValue(forward<T>(v)); 模板参数为右值引用，转换为右值传入

    return 0;
}
```

我们得到

```shell
l-value: 100    
r-value: 100    
r-value: 100    
                
l-value: 10     
r-value: 10     
l-value: 10     
                
l-value: 10     
r-value: 10     
r-value: 10     
```



## 初始化列表

### 统一初始化

在 C98 中普通数组和可以直接进行内存拷贝的对象可以使用列表初始化

```cpp
int arr[] = {1, 2, 3};

struct Person
{
	int id;
    double salary;
} Li{1, 3000};
```

在 C++11 中，列表初始化变得更加灵活了

```cpp
#include <iostream>
using namespace std;

class Test
{
public:
    Test(int) {}

private:
    Test(const Test &);
};

int main(void)
{
    Test t1(520);
    // 错误，520 会隐式转换为匿名对象 Test(520) ，调用拷贝构造函数
    Test t2 = 520;
    Test t3 = {520};
    Test t4{520};
    int a1 = {1314};
    int a2{1314};
    int arr1[] = {1, 2, 3};
    int arr2[]{1, 2, 3};
    return 0;
}
```

其中 t4, a2, arr2 都是新型的初始化方式。我们还可以对 new 操作符创建的对象进行初始化

```cpp
int *p = new int{520};
double b = double{3.14};
int *array = new int[3]{1,2,3};
```

列表初始化还可以用于函数返回值

```cpp
class Test
{
public:
    Test(int, double) {}
};

Test func()
{
    return {1, 3.14};
}
```

其中返回的列表直接用于初始化 Test 对象。



#### 聚合体

对于不同的自定义类型，会有不同的列表初始化结果

```cpp
struct T1
{
    int x;
    int y;
} a = {1, 2};

struct T2
{
    int x;
    int y;
    T2(int, int) : x(10), y(10) {}
} b = {1, 2};
```

这里 a 会通过初始化列表构造为 `x=1, y=2` ，而 b 会被构造函数初始化为 `x=10, y=10` 。



只有当使用列表初始化的对象是一个聚合体，才会使用列表数据初始化。聚合体一般有：

* 普通数组
* 满足以下条件的类（class, struct, union）：
    * 无自定义构造函数
    * 无私有或保护的非静态数据成员（另外：静态变量不能使用列表初始化）
    * 无基类
    * 无虚函数
    * 类中不能有使用 `{}` 或 `=` 直接初始化的非静态成员（在 C++14 中可以）

```cpp
class A
{
public:
    int x;
    int y = 5;	// 直接初始化，因此不能使用列表初始化
};
```



#### 非聚合体

如果要对非聚合体使用列表初始化，需要自定义构造函数，在构造函数中使用初始化列表对类成员变量初始化

```cpp
class A
{
public:
    int x;
    double y;
    // 使用初始化列表
    A(int a, int b, int c) : x(a), y(b), z(c) {}
private:
    int z;	// 存在私有变量，不能直接使用初始化列表
};

int main()
{
    A t{1, 2.5, 3};
}
```

其中 t 调用使用初始化列表的构造函数初始化。



最后，使用非聚合体类型作为成员函数的类也可以是聚合体

```cpp
class T1
{
public:
    int x;
    double y;
private:
    int z;
};

class T2
{
public:
    T1 t1;
    int x1;
    double y1;
};

int main()
{
    T2 t2{{}, 12, 3.14};
    return 0;
}
```

其中 T1 是非聚合体，然而 T2 依然是聚合体，可以使用列表初始化，注意这里使用 `{}` 初始化 t1 ，相当于调用无参构造函数。



### initializer_list

#### 基本用法

initializer_list 是 C++11 提供的一种新类型，其定义于头文件 `<initializer_list>` 中，其类型对象是一个访问**只读**类型对象数组的轻量代理对象，用花括号初始化器列表列表初始化一个对象，其中对应构造函数接受一个 `std::initializer_list` 参数，以花括号初始化器列表为赋值的右运算数，或函数调用参数。例如

```cpp
void test(std::initializer_list<int> &l)
{
	return;
}

int main()
{
    test({1, 2, 3});
    return 0;
}
```

initializer_list 对象只能被整体初始化或者赋值，不过由于其内部仅仅储存了初始化列表中元素的引用，因此效率很高。



成员函数如下

| 成员函数 |               作用               |
| :------: | :------------------------------: |
|   size   | 返回 initializer_list 中元素数目 |
|  begin   |       返回指向首元素的指针       |
|   end    |  返回指向末尾元素后一位置的指针  |

我们可以利用该类型实现输入数组作为参数的初始化，然后通过 size 函数获得数组的长度。



#### 通用初始化

任何对象都是可以用 `initializer_list` 初始化的，哪怕其构造函数没有以 `initializer_list` 为参数的形式。例如：

```C++
template <typename T>
class A
{
public:
    A(T r = 0, T i = 0) : r(r), i(i) {}
};
```

这个类内没有接受 `initializer_list<>` 为参数的构造函数，但依旧可以用 `initializer_list` 构造这个类的对象

```cpp
A<int> a{3, 4};		// initializer_list 初始化
A<int> a2(3, 4);
```

在这个例子中，对象 a 初始化时，接收的参数 `{3,4}` 类型为 `initializer_list<int>` ，编译器先将其转化为 `array<int,2>` ，然后取出该对象的两个元素，用作构造函数的两个整型参数。但如果同时有接受 `initializer_list<T>` 的构造函数，则不会调用该构造函数。



作为构造函数的参数有一个重要的好处在于，它可以不提前指定参数的个数，这一点在实际操作中非常有用。比如下面关于`Vec`对象的初始化：

```C++
template <typename T, int Dim>
class Vec
{
private:
    T arr[Dim];

public:
    Vec(initializer_list<T> L)
    {
        auto j = L.begin();
        for (int d = 0; d < Dim; ++d)
            arr[d] = *j++;
    }
};
```



#### 类型转换

值得一提的是， `initializer_list<T>` 的对象可以自动转化为其包含的若干个 `T` 的对象，反过来则不行。

```C++
Vec<double, 2> V{3.0, 4.0};   // 正确，输入一个 initializer_list
// Vec<double,2> v1(3.0,4.0); // 错误，(3.0,4.0) 不能自动转换为 {3.0,4.0}.
```

这就是使用 `{}` 初始化的好处之一，能够避免精度丢失

```cpp
int(3.14f)	// 通过
int{3.14f}	// 报错
```



#### 数组初始化

同时，还优化了数组的初始化

```cpp
int array[]{11, 22, 33, 44}
```

可以直接在数组变量后添加初始化列表。



## 类与对象

### 默认函数

编译器自定义的成员函数即为默认函数：

* 无参构造函数
* 拷贝构造函数
* 移动构造函数
* 拷贝赋值函数
* 移动赋值函数
* 析构函数

一旦程序员实现了这些自定义版本，就不会再生成默认版本。最典型的是声明可带参数的构造函数，此时就不会有默认的无参构造函数。



#### default

我们可以直接指定默认函数

````cpp
class Base
{
public:
    Base() = default;
    Base(const Base& obj) = default;
    Base(Base&& obj) = default;
    Base& operator= (const Base& obj) = default;
    Base& operator= (Base&& obj) = default;
    ~Base() = default;
};
````

也可以在类的外部指定

```cpp
class Base
{
public:
    Base();
    ~Base();
};

Base::Base() = default;
Base::~Base() = default;
```



#### delete

delete 的语义与 default 恰恰相反，即**不生成**默认的函数。如果我们不希望用户调用某个成员函数就可以使用

```cpp
class A
{
public:
    // 禁用拷贝构造
    A(const A&) = delete;
    void print() = delete;
};
```

其中关于 print 的禁用没有意义，更常见的是如下的情况

```cpp
class A
{
public:
    void print(int n) 
    {
        cout << n << endl;
    }
    void print(char c) = delete;
};
```

这种写法防止通过字符隐式转换来调用 `print(int n)` 函数。



也可以在一个派生类中**要求基类中某个函数不得被调用**

```cpp
class A
{
public:
    void print()
    {
        cout << "This is A." << endl;
    }
};

class B : public A
{
public:
    void print() = delete;
};
```

此时，继承子类中的 print 函数不存在。



如果我们希望禁止一个类实例在堆上构造，可以将对应的构造方法删除

```cpp
struct Test 
{
    void *operator new(std::size_t) = delete;
    void *operator new[](std::size_t) = delete;
}
```

不仅如此，还可以删除全局函数

```cpp
void f(int) {}
void f(double) = delete;
```

这将会禁止 `f(1.2)` 形式的隐式调用。



### 构造函数

#### 生成规则

六个特殊的成员函数其生成规则如下：

- 默认构造函数，在用户没有声明自定义的构造函数的时候并且编译期需要的时候生成
- 析构函数，在 C++11 中默认是 noexcept
- 拷贝构造函数，用户自定义**移动操作会导致不生成默认的拷贝构造函数**
- 拷贝赋值操作符，用户自定义**移动操作会导致不生成默认的拷贝赋值操作**
- 移动构造函数和移动赋值操作符，仅仅在没有用户自定义的拷贝操作，移动操作和析构操作的时候才会生成

**当类中含有特殊成员函数变为模板特殊成员函数时不满足上述生成规则**。



#### 委托构造函数

委托构造函数允许使用同一个类中的一个构造函数调用其它构造函数

```cpp
class A
{
public:
    int a;
    
public:
    A(int a) : a(a) {}
    // 委托前面的构造函数进行初始化
    A(int a, int b) : A(a)
    {
        cout << b << endl;
    }
};
```

需要注意，不要在委托构造函数中产生循环调用，以及如果在其中初始化了某个成员变量，就不能再使用初始化列表初始化。



#### 继承构造函数

在继承类中的构造函数可以直接使用父类的构造函数，例如

```cpp
class Base
{
public:
    int m_i;
    double m_j;
    string m_k;
    
public:
    Base(int i) : m_i(i) {}
    Base(int i, double j) : m_i(i), m_j(j) {}
    Base(int i, double j, string k) : m_i(i), m_j(j), m_k(k) {}
};

class Child : public Base
{
public:
    // 一般形式
    Child(int i) : Base(i) {}
    Child(int i, double j) : Base(i, j) {}
    Child(int i, double j, string k) : Base(i, j, k) {}
};
```

使用继承构造函数 `using 类名::构造函数名` 来声明使用基类的构造函数

```cpp
class Child : public Base
{
public:
    // 直接使用基类构造函数
    using Base::Base;
};
```

另外，如果在子类中隐藏了父类同名函数，也可以通过 using 来使用父类函数

```cpp
class Base
{
public:
    void func(int i)
    {
        cout << i << endl;
    }
    void func(int j, double k)
    {
        cout << j << " " << k << endl;
    }
};

class Child : public Base
{
public:
    // 一般此函数会隐藏父类的 func 函数
    void func()
    {
        cout << "This is child." << endl;
    }
    // 声明可以使用基类 func 函数
    using Base::func;
};

int main()
{
    Child c;
    c.func();
    c.func(1);
    c.func(2, 3.14);
    return 0;
}
```



#### 移动构造函数

在 C++ 中进行对象赋值操作时，有时需要进行深拷贝，这时开销就很大。可以使用右值引用进行优化，例如

```cpp
#include <iostream>

using namespace std;

class Test
{
public:
    Test() : m_num(new int(100)) {}
    Test(const Test &a) : m_num(new int(*a.m_num)) {}
    ~Test() { delete m_num; }

public:
    int *m_num;
};

Test getObj()
{
    return Test();
}

int main()
{
    Test t = getObj();

    return 0;
}
```

注意这里调用 getObj 时发生了深拷贝，在其中创建的临时对象的内存在申请后就立即释放，造成了资源浪费。右值引用具有**移动语义**，可以将资源通过浅拷贝从一个对象转移到另一个对象，例如

```cpp
#include <iostream>

using namespace std;

class Test
{
public:
    Test() : m_num(new int(100)) {}
    Test(const Test &a) : m_num(new int(*a.m_num)) {}
    
    // 添加移动构造函数
    Test(Test &&a) : m_num(a.m_num)
    {
        // 转移指针后立即取消原先的指针
        a.m_num = nullptr;
    }
    ~Test() { delete m_num; }

public:
    int *m_num;
};

Test getObj()
{
    return Test();
}

int main()
{
    Test t = getObj();

    return 0;
}
```

其中 getObj 的返回值是一个将亡值，进行赋值操作时，调用了移动构造函数。移动构造使用的右值引用会将临时对象中的堆内存地址的所有权转移给对象 t ，因此可以继续使用。

> 在提供移动构造函数时，也会提供常量左值引用的拷贝构造函数，保证移动不成还能使用深拷贝；另外，如果存在移动构造函数，那么就不需要定义移动复制函数，后者会被前者取代。



### 修饰类继承

#### final

使用 final 可以限制某个类不能被继承，或者某个虚函数不能被重写。如果使用 final 修饰函数，则只能修饰虚函数，并且要把 final 放在类或函数的后面。



使用 final 修饰函数表示不能重写

```cpp
class Base
{
public:
    virtual void test() {}
};

class Child : public Base
{
public:
    void test() final {}
};
```

此时 Child 类中 test 函数不能被子类重写。



使用 final 修饰的类不能被继承

```cpp
class Base
{
public:
    virtual void test() {}
};

class Child final: public Base
{
public:
    void test() {}
};
```

此时 Child 类不能有子类。



#### override

override 表明子类中会重写父类的虚函数

```cpp
class Base
{
public:
    virtual void test() {}
};

class Child final: public Base
{
public:
    void test() override {}		// 合法重写
    void test(int) override {}	// 非法重写
};
```

这样就能让编译器检查两个函数是否确实是重写关系，降低了出错的概率。



### 可调用对象

可调用对象一般有如下几种：

* 函数指针
* 具有 `operator()` 成员函数的类
* 可被转换为函数指针的类
* 类成员函数指针或类成员指针



例如可以隐式转换为函数的类

```cpp
#include <iostream>
#include <string>
using namespace std;

// 定义函数指针的类型
using func_ptr = void(*)(int, string);

struct Test
{
	static void print(int a, string b)
    {
        cout << "name: " << b << ", age: " << a << endl;
    }
    
    // 定义隐式变换函数进行转换
    operator func_ptr()
    {
        return print;
    }
};

int main()
{
    Test t;
    t(15， "Li");
    return 0;
}
```



再例如类成员函数及其指针

```cpp
#include <iostream>
#include <string>
using namespace std;

struct Test
{
    void print(int a, string b)
    {
        cout << "name: " << b << ", age: " << a << endl;
    }
    int m_num;
};

int main(void)
{
    // 定义类成员函数指针指向类成员函数
    void (Test::*func_ptr)(int, string) = &Test::print;
    
    // 类成员指针指向类成员变量
    int Test::*obj_ptr = &Test::m_num;

    Test t;
    // 通过类成员函数指针调用类成员函数
    (t.*func_ptr)(19, "Li");
    
    // 通过类成员指针初始化类成员变量
    t.*obj_ptr = 1;
    cout << "number is: " << t.m_num << endl;

    return 0;
}
```



#### function

 可调用对象的包装器 `std::function` 是一个类模板，可以容纳**除了类成员（函数）指针**以外的所有可调用对象。



需要头文件 `<functional>` ，语法为

```cpp
std::function<return_type(arguments)> diy_name = obj;
```

使用它包装普通函数、静态成员函数和仿函数

```cpp
int add(int a, int b)
{
    return a + b;
}

struct T1
{
    static int test(int a, int b)
    {
        return a * b;
    }
};

struct T2
{
    int operator()(int a, int b)
    {
        return a - b;
    }
};

T2 t;
function<int(int, int)> f1 = add;
function<int(int, int)> f2 = T1::test;
function<int(int, int)> f3 = t;
f1(1, 2);
f2(1, 2);
f3(1, 2);
```



因为回调函数通过函数指针实现，可以使用对象包装器取代函数指针

```cpp
class A
{
public:
    // 传入包装器对象
    A(const function<void()>& f) : callback(f) {}
    
    // 调用函数指针
    void notify()
    {
        callback();
    }
    
private:
    function<void()> callback;
};

class B
{
public:
    void operator()()
    {
        cout << "func is called." << endl;
    }
};

int main()
{
    B b;
    A a(b);		// 传入包装器初始化
    a.notify();	// 调用回调函数
    return 0;
}
```



#### mem_fn

不仅可以将一般函数和 lambda 表达式进行封装，还可以封装成员函数。例如

```cpp
#include <functional>
#include <iostream>

class MyClass
{
  public:
    void print(int x)
    {
        std::cout << "Value: " << x << std::endl;
    }
};

int main()
{
    MyClass obj;
    auto func = std::mem_fn(&MyClass::print);

    // 调用成员函数
    func(obj, 42);

    return 0;
}
```



#### bind

使用 `std::bind` 将可调用对象与其参数一起进行绑定。绑定后的结果可以使用 ` std::function ` 进行保存，并延迟调用到任何需要的时候。语法格式如下：

```cpp
auto f = std::bind(可调用对象地址, 绑定的参数/占位符);
```

通过绑定可以将 n 元可调用对象转换为 n-1 元可调用对象

```cpp
#include <functional>
#include <iostream>

void output(int x, int y)
{
    std::cout << x << " " << y << std::endl;
}

int main(void)
{
    std::bind(output, 1, 2)();                       // output 绑定参数 1 2 ，因此不需要参数了
    std::bind(output, std::placeholders::_1, 2)(10); // output 绑定占位符和 2 ，只需要一个参数
    std::bind(output, 2, std::placeholders::_1)(10); // output 绑定 2 和占位符 ，只需要一个参数

    // error, 调用时没有第二个参数
    std::bind(output, 2, std::placeholders::_2)(10);

    // 调用时第一个参数 10 被吞掉了，没有被使用
    std::bind(output, 2, std::placeholders::_2)(10, 20);

    // 绑定两个占位符，需要两个参数调用
    std::bind(output, std::placeholders::_1, std::placeholders::_2)(10, 20);
    std::bind(output, std::placeholders::_2, std::placeholders::_1)(10, 20);

    return 0;
}
```

其中 `placeholders::_1` 表示使用传入的第一个参数替换此占位符，同理 `placeholders::_n` 依次表示用传入的第 n 个参数替换占位符。



原本 `std::function` 不能保装类成员函数指针或类成员指针，但通过 bind 就可以配合实现，基本语法为

```cpp
auto f = std::bind(类函数/成员地址, 类实例对象地址, 绑定的参数/占位符);
```

需要注意第一个参数是直接对类的成员函数取地址，例如

```cpp
#include <iostream>
#include <functional>

class Test
{
public:
    void output(int x, int y)
    {
        std::cout << "x: " << x << ", y: " << y << std::endl;
    }
    int m_number = 100;
};

int main(void)
{
    Test t;

    // 绑定类成员函数，这里直接取 Test 类的函数的地址，与类实例地址绑定
    std::function<void(int, int)> f1 = std::bind(&Test::output, &t, std::placeholders::_1, std::placeholders::_2);
    
    // 绑定类成员变量(公共)，直接取 Test 类的成员变量的地址，与类实例地址绑定
    std::function<int&(void)> f2 = std::bind(&Test::m_number, &t);

    // 调用
    f1(520, 1314);
    f2() = 2333;
    std::cout << "t.m_number: " << t.m_number << std::endl;

    return 0;
}
```

其中 `f2` 在绑定时返回值类型是类成员 `m_number` 的引用，没有占位符或参数表明它没有参数要求。最后 `f2()` 返回 `int&` ，因此可以直接修改类成员 `m_number` 的值。



#### invoke

使用 `std::invoke` 来调用函数

```cpp
void foo(int x) {
    std::cout << "foo: " << x << std::endl;
}

struct Functor {
    void operator()(int x) const { std::cout << "called: " << x << std::endl; }
};

std::invoke(foo, 30);
std::invoke(functor, 40);
```

它真正的用途在于处理类成员指针

```cpp
struct A {
    void foo(int x) const { std::cout << "foo: " << x << std::endl; }

    int x = 42;
};

int main() {
    A a;

	// 通过类函数指针调用 (a.*Fn)(10);
    auto Fn = &A::foo;
    std::invoke(Fn, a, 20);

	// 通过类成员指针获得成员 a.*x;
    auto x = &A::x;
    std::cout << std::invoke(x, a) << std::endl;

    return 0;
}
```

还可以用来自动处理引用包装器

```cpp
void test(int& x) {
    ++x;
}

int main() {
    int x = 5;
    std::reference_wrapper<int> ref(x);

    std::invoke(test, ref);
    return 0;
}
```



### 取地址

在 C++11 中引入 `std::addressof` 方法，它获得变量的实际地址。例如

```cpp
#include <iostream>
#include <memory>

struct Test
{
    int data;
};

int main()
{
    Test obj;
    Test *p1 = std::addressof(obj);

    std::cout << "Address of obj: " << p1 << std::endl;
    return 0;
}
```

在使用泛型编程时，取地址算符 `&` 可能会被重载，而 `std::addressof` 可以绕开重载，直接获得地址。



### 多参数 `[]`

在 C++11 中引入了以初始化列表为参数的 `[]` 重载

```cpp
#include <iostream>

struct S
{
    template <class T> auto operator[](std::initializer_list<T> il)
    {
        for (auto &&i : il)
            std::cout << i << " ";
        std::cout << std::endl;
    }
};

int main()
{
    S s;
    s[{1, 2, 3, 4, 5}];
    return 0;
}
```

在 C++23 中允许变长模板参数的 `[]` 重载

```cpp
#include <iostream>

struct S
{
    template <class... Args> auto operator[](Args... args)
    {
        ((std::cout << args << " "), ...);
        std::cout << std::endl;
    }
};

int main()
{
    S s;
    s[1, 2, 3, 4, 5];
    return 0;
}
```



### 引用修饰

在 C++11 中引入了引用限定符，允许在成员函数后添加引用修饰

```cpp
#include <iostream>

struct S
{
    int x;

    void f(int x) &
    {
        x = this->x;
        std::cout << "f()&" << std::endl;
    }

    void f(int x) &&
    {
        x = this->x;
        std::cout << "f()&&" << std::endl;
    }

    void f(int x) const &
    {
        std::cout << "f() const&" << std::endl;
    }
    
    void f(int x) const &&
    {
        std::cout << "f() const&&" << std::endl;
    }
};

int main()
{
    S s;
    s.f(1);   // calls f() &
    S().f(1); // calls f() &&

    const S cs;
    cs.f(1);        // calls f() const&
    const S().f(1); // calls f() const&&

    return 0;
}
```



### 类内初始化

C++17 允许在类内部直接初始化成员变量。例如

```cpp
class A
{
public:
    int a = 10;
    int b = 20;
    const double t = 15;
    static const char c = 'a';
    
    static int d = 10;	// 错误
}
```

但需要注意的是，一般的 static 成员变量依然需要在类外初始化。



### 改进的友元

在 C++11 中声明一个类为另一个类的友元时，不再需要使用 class 关键字，并且还可以使用类的别名

```cpp
class Tom;
using Jerry = Tom;

class Jack
{
    friend class Tom;	// c98 语法
    friend Tom;			// c11 语法
};

class Nick
{
    friend Jerry;		// c11 语法
};

class Tom
{
public:
    Jack jObj;
    Nick nObj;
};
```

这样就可以在 Tom 类中直接访问 Jack 和 Nick 的成员。



虽然这种改进看起来不大，但是在应用上我们现在可以为类模板声明友元，这在 C98 中是无法做到的

```cpp
class Tom;

template <class T>
class Person
{
    friend T;
};
```

这时我们在模板实例化时才确定谁是友元类型。



一个实用的场景是对模板类进行测试

```cpp
template <class T>
class Circle
{
public:
    friend T;
    Circle(double r) : radius(r) {}
    
private:
    double radius;
};

// 校验类
class Verify
{
public:
    // 检查 c 的半径是否满足要求
    void verifyCircle(double r, Circle<Verify>& c)
    {
        if (r >= c.radius)
        {
            cout << "Yes" << endl;
        }
        else
        {
            cout << "No" << endl;
        }
    }
};

int main()
{
    Verify v;
    // 此时 circle 的友元时 Verify
    Circle<Verify> circle(30);
    v.verifyCircle(60, circle);
    return 0;
}
```

这样通过模板来引入校验类，就省去了额外定义访问函数的麻烦，还可以将检测代码进行封装。



### 拷贝与交换

自定义智能指针，需要交换内部的指针变量

```cpp
#include <iostream>
#include <cstdlib>

namespace A
{

template <typename T> class smart_ptr
{
  public:
    smart_ptr() noexcept : ptr_(new T())
    {
    }

    smart_ptr(const T &ptr) noexcept : ptr_(new T(ptr))
    {
    }

    smart_ptr(smart_ptr &rhs) noexcept
    {
        // 释放所有权
        ptr_ = rhs.release();
    }

    // noexcept == throw() 保证不抛出异常
    void swap(smart_ptr &rhs) noexcept
    {
        // 使用标准 swap 交换内部指针
        using std::swap;
        swap(ptr_, rhs.ptr_);
    }

    T *release() noexcept
    {
        T *ptr = ptr_;
        ptr_ = nullptr;
        return ptr;
    }

    T *get() const noexcept
    {
        return ptr_;
    }

  private:
    T *ptr_;
};

// 针对自定义类型提供交换函数，注意放在命名空间中防止与 std::swap 重复
template <typename T> void swap(A::smart_ptr<T> &lhs, A::smart_ptr<T> &rhs) noexcept
{
    lhs.swap(rhs);
}

} // namespace A
```



现在考虑智能指针的拷贝问题。最简单的方式是防止自赋值的写法

```cpp
smart_ptr &operator=(const smart_ptr &rhs)
{
    if (*this != rhs)
    {
        delete ptr_;
        ptr_ = new T(rhs.ptr_); // 当 new 发生异常，此时 ptr_ 指向的而是一块被删除区域,而不是被赋值对象的区域
        return *this;
    }
    return *this;
}
```

但是这并不安全，当 `new` 出现错误时，则对 `ptr_` 的修改会出错。



较好的解决方案是通过交换赋值，有两种写法

```cpp
// 改为传入值
smart_ptr &operator=(smart_ptr rhs) noexcept
{
    swap(rhs);
    return *this;
}

// 复制然后交换
smart_ptr &operator=(smart_ptr &rhs) noexcept
{
    smart_ptr tmp(rhs);
    swap(tmp);
    return *this;
}
```



### 指向实现的指针

使用 plmpl 技术将类的实现放到源文件中，这种做法有几个好处：

1. 二进制兼容性：开发库时，可以在不破坏与客户端的二进制兼容性的情况下向隐藏声明中添加/修改字段。 
2. 数据隐藏：用于隐藏实现库公共接口的其他库/实现技术。 
3. 编译时间减少了，因为向隐藏类型添加/删除字段和/或方法时，仅需要重建源文件。



考虑下面这个例子

```cpp
class X
{
  private:
    C c;
    D d;
};
```

使用 plmpl 技术将其写成

```cpp
// 头文件
class X
{
  private:
    struct XImpl;
    XImpl *pImpl;
};

// 源文件
struct X::XImpl
{
    C c;
    D d;
};
```

这样一来，结构 XImpl 的实现将在编译为库文件后隐藏起来，无法查看其中的成员。



### 枚举类

枚举是要定义一个类型，并且可以穷举其中的个体使用。但是它存在一些缺陷，在 C++ 中具有名字的 enum 类型的名字和其成员的名字都是全局可见的。这与 namespace, class/struct 及 union 需要通过 `name::member` 形式访问格格不入，当我们的定义名重复时，就会产生问题

```cpp
// error 变量重定义
enum A {red, yellow, green};
enum B {red, ride, rick};
```

并且由于枚举被设计为常量数值的别名，因此它的成员总是可以隐式地转换为整型，但有时我们并不希望出现这种转换。



C++11 引入了枚举类或强类型枚举，通过在 enum 后添加 class 来声明

```cpp
enum class Colors {red, green, yellow};
```

强类型枚举成员的作用域，起自其声明之处，终止枚举定义结束之处；并且**不能被隐式转换为整型**。



强类型枚举可以显式指定底层类型，可以是除 wchar_t 以外的任何整型

```cpp
enum class A : char
{
	a = 'a',
	b = 'b',
	c
};

// 和上面没有任何区别
enum struct A : char
{
	a = 'a',
	b = 'b',
	c
};

// 每一变量都占用 2 字节 16 比特。
enum class A : int16_t
{
	a,
	b,
	c
};
```

上面的枚举变量确实是 char 类型，其占用的内存只有 1 字节，但是使用时会转换为对应的 ASCII 码。为了配合新的枚举类型，原有的枚举类型也进行了扩展，可以像上面一样指定底层类型。



## 模板

### 修饰模板

#### extern

在传统 C++ 中，每当编译器遇到完整的模板就会实例化，这可能导致重复的实例化，编译时间增加。在 C++11 中引入了外部模板，可以手动指定是否在当前文件中进行实例化

```cpp
template class std::vector<bool>;          // 将 vector 模板类强行实例化
extern template class std::vector<double>; // 不要在当前编译文件中实例化模板
```



#### typename

当涉及到模板参数的内置类型时，都需要 typename 修饰来提醒编译器这是一个类型。例如

```cpp
template <typename T> void func(T t) {
    typename T::value_type* x;	// value_type 是一个类型而不是静态成员
    typename std::decay<decltype(t[0])>::type* y;
}
```



#### constexpr

在 C++14 中允许变量模板

```cpp
#include <iostream>

template <class T>
constexpr T v{};

int main()
{
    std::cout << v<int> << std::endl;
    return 0;
}
```



### 默认模板参数

在 C++98 中可以对模板类使用默认的模板参数

```cpp
template <class T = int>
class A
{
public:
    void print() {}
};
```

但不支持函数的默认模板参数。在 C++11 中添加了这一功能

```cpp
template <class T = int>
void test(T t)
{
	cout << t << endl;
}
```

需要注意：

* 模板类即使全都有默认参数，也**需要显式声明模板参数**，而模板函数不需要
* 模板函数的**默认参数不需要写在最后一个**，这会给我们提高灵活性



当默认模板参数和模板参数自动推导同时使用时，按照如下优先顺序：

1. 如果可以推导出参数类型，则使用推导出的参数类型
2. 如果不能推导，但是有默认参数类型，则使用默认参数类型

```cpp
#include <iostream>

using namespace std;

template <class T = int, class R>
T func(R arg)
{
    cout << "func: " << arg << endl;
    return arg;
}

int main()
{
    cout << func(3.14) << endl;
    cout << func<int>(3.14) << endl;
    cout << func<double>(3.14) << endl;
    cout << func<int, double>(3.14) << endl;
    cout << func<double, int>(3.14) << endl;

    return 0;
}
```

输出结果如下：

```shell
func: 3.14
3
func: 3.14
3
func: 3.14
3.14
func: 3.14
3
func: 3
3
```

由于我们将默认模板参数放在第 1 个，而第二个模板参数可以通过实参类型自动推导，因此不需要显式地写出任何模板参数。



### 变长模板参数

#### 模板递归

变长模板参数语法为

```C++
template <typename... Types>
function()
{
    ...
}
```

适用于多个模板参数，这里的“多个”指参数类型和个数都没有事先确定。



使用方法如下，可以指定可变参数类型，并且传入可变参数

> 变长模板参数只能放在普通模板参数的后面。

```C++
void print()
{
    // 作为递归的结束
}

// typename... 表示可变参数类型
template <typename T, typename... types>
void print(T a, types... args)
{
    std::cout << a << std::endl;
    
    // arg... 表示可变参数
    print(args...);
}
```

这是利用递归实现可变模板参数的方式。



如果要在函数中知道可变模板参数的具体个数，可以用 `sizeof` 函数。具体写法：

```C++
sizeof...(args)
```

值得一提的是，函数

```C++
template <typename... types>
void print(types... args)
{
	// ...
}
```

可以和上面所述的 `print()` 函数共存，且**上面所述的 `print()` 是这个函数的特化**。



#### 模板展开

在 C++17 中增加了对变长模板的展开，允许通过常量条件句循环展开。例如

```cpp
template <typename T0, typename... T> void printf(T0 t0, T... t)
{
    std::cout << t0 << std::endl;
    
    // 借助常量表达式展开
    if constexpr (sizeof...(t) > 0)
        printf(t...);
}
```



#### 模板特化

模板特化是否是特化要根据实际类或函数使用的模板数来决定，模板类型多的定义不一定就是特化。例如下面获得 `tuple` 大小的结构

```cpp
// 顶层模板
template <class T>
struct tuple_size {

};

// 特化 T = std::tuple<>
template <>
struct tuple_size<std::tuple<>>
{
    static constexpr size_t value = 0;
};

// 特化 T = std::tuple<T0, Ts...>
template <class T0, class... Ts>
struct tuple_size<std::tuple<T0, Ts...>>
{
    static constexpr size_t value = 1 + tuple_size<std::tuple<Ts...>>::value;
};
```

即使 `template <class T0, class... Ts>` 使用了数量更多的变长模板，由于它们作为 `tuple` 的模板参数使用，而 `tuple` 单独作为当前结构体的模板，因此是特化。



#### 初始化列表展开

可以借助初始化列表完成模板展开，例如

```cpp
template <typename T, typename... Ts> auto printf3(T value, Ts... args)
{
    std::cout << value << std::endl;
    (void)std::initializer_list<T>{([&args] { std::cout << args << std::endl; }(), value)...};
}
```

其中列表元素为

```cpp
( [&args] { std::cout << args << std::endl; }(), value ) == value
```

首先执行前面的 lambda 表达式，输出变长参数，然后由于逗号表达式，整个表达式结果为 value，保存在初始化列表中。

> void 强制转换来防止编译器报错。



#### 折叠表达式

利用模板展开的特性，C++17 允许对表达式进行折叠。折叠表达式的实例化按以下方式展开成表达式

* 一元右折叠 `(expr op ...)->(expr1 op (... op (expr_n_1 op expr_n)))`
* 一元左折叠 `(... op expr)->(((expr1 op expr2) op ...) op expr_n)`
* 二元右折叠 `(expr op ... op Expr)->(expr1 op (... op (expr_n_1 op (expr_n op Expr))))`
* 二元左折叠 `(Expr op ... op expr)->((((Expr op expr1) op expr2) op ...) op expr_n)`

这里右边展开式中的 `...` 仅表示省略。



###### 一元折叠

一元折叠例如

```cpp
template <typename... T> auto average(T... t)
{
    // t + 剩余部分递归展开调用
    return (t + ...) / sizeof...(t);
}

int main()
{
    std::cout << average(1, 2, 3, 4, 5, 6, 7, 8, 9, 10) << std::endl;
}
```

再例如

```cpp
#include <iostream>

template <typename... Args> void print(Args... args)
{
    ((std::cout << args << " "), ...);
}

int main()
{
    print("Hello", "World", 123);
    return 0;
}
```

其中 `,` 作为运算符指导展开。



###### 递归求值

折叠表达式还可以递归求值

```cpp
#include <iostream>

template <int... args> constexpr int v = (args + ...);
template <int... args> constexpr int v2 = (args - ...);
template <int... args> constexpr int v3 = (... - args);

int main()
{
	std::cout << v<1, 2, 3> << std::endl;	// (1+(2+3))
	std::cout << v2<1, 2, 3> << std::endl;	// (1-(2-3))
   	std::cout << v3<1, 2, 3> << std::endl;	// ((1-2)-3)
    return 0;
}
```

注意区分左折叠和右折叠。



###### 数组展开

还可以展开为数组

```cpp
#include <iostream>

template <int... vals, class F> auto func(F f)
{
    // func 递归展开
    int _[] = {(f(vals), 0)...};
    for (int i = 0; i < sizeof...(vals); ++i)
        std::cout << _[i] << std::endl;
}

int main()
{
    func<1, 2, 3>([](int x) { std::cout << x << std::endl; });
    return 0;
}
```

其中 `(f(vals), 0)` 中的 `0` 目的仅仅是作为表达式的最终结果，展开为

```cpp
{(f(1), 0), (f(2), 0), (f(3), 0)} // {0, 0, 0}
```



包展开符号 `...` 优先级较低，因此允许多次使用包参数

```cpp
#include <iostream>

template <class... Args> auto f(Args... args)
{
	// 使用两次 args
    static auto list = {args * args...};
    return list;
}

int main()
{
    auto ret = f(1, 2, 3, 4, 5);
    for (auto i : ret)
        std::cout << i << " ";
    std::cout << std::endl;
    return 0;
}
```

甚至可以访问数组元素后展开

```cpp
#include <iostream>

template <class... Args> void print(Args... args)
{
    ((std::cout << args << ' '), ...);
}

template <class Arr, class... T> auto f(Arr arr, T... idx)
{
    // 数组元素展开
    print(arr[idx]...);
}

int main()
{
    int arr[] = {1, 2, 3, 4, 5};
    f(arr, 0, 1, 2, 3, 4);
    return 0;
}
```



###### 二元折叠

二元折叠例如

```cpp
#include <iostream>

template<class... Args>
void print(Args... args)
{
	(std::cout << ... << args) << std::endl;
}

int main()
{
    print("Hello", "World", 123);
    return 0;
}
```



#### 递归继承

可以递归继承多个类

```cpp
#include <iostream>

template <class... T> struct overload : T...
{
    // 继承所有父类的 operator()
    using T::operator()...;
};

int main()
{
    // 继承 3 个 lambda 表达式
    auto c = overload{[](int x, int y) { std::cout << "int, int\n"; },
                      [](double x, double y) { std::cout << "double, double\n"; },
                      [](auto x, auto y) { std::cout << "auto, auto\n"; }};

    c(1, 2);
    c(3.14, 2.71);
    c("hello", "world");
    return 0;
}
```



#### 模板实例

###### tuple

例如 `std::tuple` 实现的功能是将若干不同类型的对象集中在同一个数据结构中，其具体构造和实现则是依赖**递归继承**，用到了可变模板的性质。

```C++
// 带可变参数的元组
template <typename... Values> class mytuple;

// 特化：没有参数的元组
template <> class mytuple<>
{
};

template <typename Head, typename... Tail> class mytuple<Head, Tail...> : private mytuple<Tail...>
{
    // 定义继承父类，长度比当前 tuple 小
    typedef mytuple<Tail...> inherited;

  public:
    // 类似于前，它是递归出口
    mytuple()
    {
    }

    // 递归定义可变模板
    mytuple(Head v, Tail... vtail) : m_head(v), inherited(vtail...)
    {
    }
    
    Head head()
    {
        return m_head;
    }
    
    // 返回包含尾部元素的元组
    // 继承类内存分配按照父类在前，子类成员变量在后的顺序
    // 因此 *this 返回的对象初始化 inherited 对象恰好就得到尾部元素的元组
    inherited &tail()
    {
        return *this;
    }

  protected:
    Head m_head;
};
```

这里的默认构造函数是**递归出口**，带有可变参数的构造函数则为递归定义。



下面是一段测试程序：

```C++
int main()
{
    mytuple<int, float, std::string> t(41, 6.3, "nico");
    int x = t.head();
    std::cout << x << std::endl;
    float b = t.tail().head();
    std::cout << b << std::endl;
}
```

值得一提的是，由于该继承形式是**私有继承**，我们不能直接通过对象 `t` 去访问其之后的成员。函数 `tail()` 返回的是除去第一个参数后的 `tuple` 。



对于上述定义的 tuple 类型，可以验证

```cpp
mytuple<int, float, string> t(41, 6.3, "nico");
```

这是一个 `4 + 4 + 32` 字节的实例。



我们还可以修改实现为

```cpp
template <typename... Values> class mytuple;

template <> class mytuple<>
{
};

template <typename Head, typename... Tail> class mytuple<Head, Tail...>
{
    typedef mytuple<Tail...> composited;

  protected:
    Head m_head;
    composited m_tai;

  public:
    // 作为递归出口
    mytuple()
    {
    }

    mytuple(Head h, Tail... tail) : m_head(h), m_tai(tail...)
    {
    }

    // 等价于 Head head()  { return m_head; }
    auto head() -> decltype(m_head)
    {
        return m_head;
    }

    // 获得除去 head 以外的 tuple 部分大小
    int get()
    {
        return sizeof(composited);
    }

    // 获得除去 head 以外的 tuple 部分
    composited &tail()
    {
        return m_tai;
    }
};
```

与之前不同的是，我们还保存了除了 head 以外的 tuple 部分直接作为成员。在此基础上，下面的实例

```cpp
mytuple<int, float, string> t(41, 6.3, "nico");
```

由于 `string` 以 8 字节对齐，出现在 `composited` 中，会让其余成员也按照 8 字节对齐。因此实例将分解为以下部分：

1. 第一层 `int + composited`，其中 `int` 占 4 字节，对齐后为 8 字节；
2. 第二层 `float + composited`，其中 `float` 占 4 字节，对齐后为 8 字节；
3. 第三层 `string + composited`，其中 `string` 占 32 字节，已经对齐；然而 `composited` 为空，这会直接调用全特化版本，占 1 字节，对齐后为 8 字节；

最终得到 `8 + 8 + 32 + 8` 字节 。



###### tuple_concat

利用变长模板连接两个元组

```cpp
template <class T1, class T2> struct tuple_concat {};

template <class... T1s, class... T2s> struct tuple_concat<std::tuple<T1s...>, std::tuple<T2s...>> {
    using type = std::tuple<T1s..., T2s...>;
};
```



###### tuple_push_front

类似地，可以实现头部推入元素类型

```cpp
template <class T1, class T2> struct tuple_push_front {};

template <class T1, class... T2s> struct tuple_push_front<T1, std::tuple<T2s...>> {
    using type = std::tuple<T1, T2s...>;
};
```



###### tuple_element

获得 `tuple` 指定位置的元素类型

```cpp
template <size_t N, class T> struct tuple_element {};

template <class T0, class... Ts> struct tuple_element<0, std::tuple<T0, Ts...>> {
    using type = T0;
};

template <size_t N, class T0, class... Ts> struct tuple_element<N, std::tuple<T0, Ts...>> {
    using type = typename tuple_element<N - 1, std::tuple<Ts...>>::type;
};
```



###### tuple_is_integral

利用折叠表达式判断元组中的元素是否都是整型

```cpp
template <class... Ts> struct tuple_is_integral {};

template <class ...Ts> struct tuple_is_integral<std::tuple<Ts...>> {
    static constexpr bool value = (true && ... && std::is_integral_v<Ts>);
};
```



###### tuple_map

借助 `std::index_sequence` 实现编译期循环，并构造元组映射函数，将元组中每个元素分别通过 F 作用后放到新的元组中返回。例如

```cpp
template <size_t... Is, class F> void static_for_tuple(F f, std::index_sequence<Is...>) {
    (f(std::integral_constant<size_t, Is>{}), ...);
}

template <size_t N, class F> void static_for_tuple(F f) {
    static_for_tuple(f, std::make_index_sequence<N>{});
	// std::make_index_sequence<3> -> std::index_sequence<0, 1, 2>    
}

template <class F, class ...Ts>
auto tuple_map(F f, std::tuple<Ts ...> tup)
{
    // 构建 F 调用后返回类型构成的元组
    std::tuple<std::invoke_result_t<F, Ts>...> res;
    static_for_tuple<sizeof...(Ts)>([&](auto i) {
        // 分别调用 F 并将结果放入 res 中
        std::get<i.value>(res) = f(std::get<i.value>(tup));
    });
    return res; 
}

std::tuple<int, double, char> tup{1, 2.0, 'a'};
auto res = tuple_map([] (auto x) { return x + 1; }, tup);
```

直接使用 `std::index_sequence` 也可以构造

```cpp
template <class F, class Tup, size_t... Is> auto tuple_map_impl(F f, Tup tup, std::index_sequence<Is...>) {
	// 利用折叠表达式展开，每个元素都通过 F 调用一次
    return std::tuple<std::invoke_result_t<F, std::tuple_element_t<Is, Tup>>...>{f(std::get<Is>(tup))...};
}

template <class F, class Tup> auto tuple_map(F f, Tup tup) {
    return tuple_map_impl(f, tup, std::make_index_sequence<std::tuple_size_v<Tup>>{});
}
```



###### tuple_apply

实现运行时多参数函数调用

```cpp
template <class F, class Tup, size_t... Is> auto tuple_apply_impl(F f, Tup tup, std::index_sequence<Is...>) {
	// 注意这里 ... 在括号里，是将所有元素展开作为 f 的输入
    return f(std::get<Is>(tup)...);
}

template <class F, class Tup> auto tuple_apply(F f, Tup tup) {
    return tuple_apply_impl(f, tup, std::make_index_sequence<std::tuple_size_v<Tup>>{});
}

// 对元组内容求和
std::tuple<int, double, float> tup{1, 2.0, 3.14f};
auto resval = tuple_apply([](auto... x) { return (x + ...); }, tup);
std::cout << "resval: " << resval << std::endl;
```



###### print

```C++
template <class Tuple, size_t N>
struct print_imp
{
    static void print(ostream &out, const Tuple &t)
    {
        print_imp<Tuple, N - 1>::print(out, t);
        out << ", " << get<N - 1>(t);
    }
};

template <class Tuple>
struct print_imp<Tuple, 1>
{
    static void print(ostream &out, const Tuple &t)
    {
        out << get<0>(t);
    }
};

template <class... Args>
ostream &operator<<(ostream &out, const tuple<Args...> &t)
{
    out << '(';
    print_imp<decltype(t), sizeof...(Args)>::print(out, t);
    return out << ')';
}
```

这个实现需要用到 `sizeof...` 函数，这个函数返回一个参数包内的参数个数。



通过流式操作符输出标准 tuple 类型

```cpp
#include <bitset>
#include <iostream>
#include <tuple>

template <int IDX, int MAX, typename... Args> struct print_tuple
{
    static void print(std::ostream &os, const std::tuple<Args...> &t)
    {
        // std::get 获得 tuple 中对应 IDX 索引的元素
        os << std::get<IDX>(t) << (IDX + 1 == MAX ? "" : ",");
        print_tuple<IDX + 1, MAX, Args...>::print(os, t);
    }
};

// 偏特化
template <int MAX, typename... Args> struct print_tuple<MAX, MAX, Args...>
{
    static void print(std::ostream &os, const std::tuple<Args...> &t)
    {
    }
};

template <typename... Args> std::ostream &operator<<(std::ostream &os, const std::tuple<Args...> &t)
{
    os << "[";
    print_tuple<0, sizeof...(Args), Args...>::print(os, t);
    return os << "]";
}

int main()
{
    std::cout << std::make_tuple(7.5, std::string("hello"), std::bitset<16>(377), 47) << std::endl;
}
```



###### printf

在有了可变模板参数的工具之后，我们可以改写一下 `C` 风格的 `printf` 函数。写法如下：

```C++
void myprintf(const char *str)
{
    // 逐项输出字符
    while (*str)
        std::cout << *str++;
    
    std::cout << std::endl;
}

template <typename T, typename... types>
void myprintf(const char *str, T arg1, types... args)
{
    while (*str)
    {
        if (*str == '%' && *(++str) != '%')
        {
            std::cout << arg1;
            myprintf(++str, args...);
            return;
        }
        std::cout << *str++;
    }
}
```

本质上和先前介绍的 `print` 函数比较相似，都是利用递归写法来适应多参数的输出。



###### maximum

这个函数的作用是在若干个参数中找到最大的那个并返回。

```C++
template <typename T>
T maximum(T x)
{
    return x;
}

template <typename T, typename... types>
T maximum(T x, types... args)
{
    return std::max(x, maximum(args...));
}
```



###### hash

利用递归模板，我们可以计算一系列变量整体的 hash 值

```cpp
#include <cstring>
#include <iostream>

// 通用 hash 模板
template <typename T> inline void hash_combine(size_t &seed, const T &val)
{
    // std::hash<T>() 定义了一个 hash 表，通过仿函数作用于 val
    seed = std::hash<T>()(val) + 0x9e3779b9 + (seed << 6) + (seed >> 2);
}

// 计算 hash 值的出口函数
template <typename T> inline void hash_val(size_t &seed, const T &val)
{
    hash_combine(seed, val);
}

// 递归计算 hash 值
template <typename T, typename... Types> inline void hash_val(size_t &seed, const T &val, const Types &...args)
{
    // 用 val 算出新的 seed，然后计算剩余参数的 hash 值
    hash_combine(seed, val);
    hash_val(seed, args...);
}

// 递归计算后，seed 就是最终的 hash 值
template <typename... Types> inline size_t hash_val(const Types &...args)
{
    size_t seed = 0;
    hash_val(seed, args...);
    return seed;
}

int main()
{
    int a = 10;
    float b = 1.5;
    char c = 'b';
    std::string d = "hello";
    std::cout << hash_val(a, b, c, d) << std::endl;

    return 0;
}
```



### 模板模板参数

#### 隐式模板类

模板模板参数是指其中一个模板参数是某个模板类。这样做的好处是，创建模板类的对象时，只需要明确参数的模板类名。例如：

```C++
// SmartPtr 是模板类
template <typename T, template <typename T> class SmartPtr>
class XCls
{
private:
	SmartPtr<T> sp;

public:
	XCls() : sp(new T){};
};
```

调用时，不需要显式给定第二个参数的数据类型，只需要明确其为哪种类型的智能指针即可。如：

```C++
int main()
{
    // 指定 T = string，成员 sp 是 shared_ptr<string> 类型
	XCls<string, shared_ptr> p;
}
```

如果需要类似于

```C++
int main()
{
	XCls<string, shared_ptr<string>> p;
}
```

这样的形式才能构造，那么这样的语法就并非 *template template parameter* 。



#### 隐式模板类型

模板模板参数可以不指定模板参数需要的模板类型，例如

```cpp
// 通过 <typename, typename> 指定 OutContainer 接收两个模板参数
template <template <typename, typename> class OutContainer = std::vector, typename F, class R>
auto fmap(R &&inputs, F &&f)
{
    // 获得 inputs 传入 f 后的返回值类型的退化类型
    typedef std::decay_t<decltype(f(*inputs.begin()))> result_type;

    // 由于 OutContainer 是 std::vector 模板，因此需要同时提供类型和对应的分配器
    OutContainer<result_type, std::allocator<result_type>> result;

    // 创建新的容器保存结果
    for (auto &&item : inputs)
        result.push_back(f(item));
    return result;
}
```



#### 萃取类型

考虑在[[现代 Cpp#公共类型]]中提出的模板结构

```cpp
template <class Tup>
struct tuple_apply_common_type {

};

// 为 tuple 特化，利用 std::tuple<Ts...> 捕获 tuple，并分离出元素 Ts...
template <class ...Ts>
struct tuple_apply_common_type<std::tuple<Ts...>> {
	// 获得 Ts... 的公共类型
    using type = typename common_type<Ts...>::type;
};
```

借助模板模板参数，可以将其进一步抽象，把 `common_type` 抽象为一个模板类传入

```cpp
template <template <class... Ts> class Tmpl, class Tup> struct tuple_apply {};

template <template <class... Ts> class Tmpl, class... Ts> struct tuple_apply<Tmpl, std::tuple<Ts...>> {
    using type = Tmpl<Ts...>;
};

template <template <class... Ts> class Tmpl, class... Ts> struct tuple_apply<Tmpl, std::variant<Ts...>> {
    using type = Tmpl<Ts...>;
};
```

于是可以这样获得公共类型

```cpp
using Tup = std::tuple<int, double, char>;
using Var = std::variant<int, double, char>;
using what = tuple_apply<common_type, Tup>::type;
using what2 = tuple_apply<common_type, Var>::type;
```

利用抽象性，现在可以传入 `tuple, variant` 来构建容器类型

```cpp
using tup = tuple_apply<std::tuple, Tup>::type;		// std::tuple<int, double, char>;
using var = tuple_apply<std::variant, Var>::type;	// std::variant<int, double, char>;
```



#### 重绑定

模板模板参数在构建时非常繁琐，于是引入了重绑定机制

```cpp
// 类型包装器
struct tuple_type_wrapper {
    template <class... Ts> struct rebind {
        using type =  std::tuple<Ts...>;
    };
};

struct variant_type_wrapper {
    template <class... Ts> struct rebind {
        using type =  std::variant<Ts...>;
    };
};

template <class Tmpl, class Tup> struct tuple_apply {};

template <class Tmpl, class... Ts> struct tuple_apply<Tmpl, std::tuple<Ts...>> {
	// 将 Tmpl 作为模板，绑定到类型参数 Ts...
    using type = typename Tmpl::template rebind<Ts...>::type;
};

template <class Tmpl, class... Ts> struct tuple_apply<Tmpl, std::variant<Ts...>> {
    using type = typename Tmpl::template rebind<Ts...>::type;
};

int main() {
    using Tup = std::tuple<int, double, char>;
    using Var = std::variant<int, double, char>;
    using tup = tuple_apply<tuple_type_wrapper, Tup>::type;		// std::tuple<int, double, char>;
    using var = tuple_apply<variant_type_wrapper, Var>::type;	// std::variant<int, double, char>;
    return 0;
}
```



利用重绑定机制，将元组中的类型绑定到容器类型，然后作为新元组的类型

```cpp
// 元组映射器
template <class Tmpl, class Tup> struct tuple_map {};

template <class Tmpl, class... Ts> struct tuple_map<Tmpl, std::tuple<Ts...>> {
    // 将 Ts 中的每个元素映射到 Tmpl 的类型中，作为新的元组的元素
    using type = std::tuple<typename Tmpl::template rebind<Ts>::type...>;
};

template <size_t N> struct array_wrapper {
    template <class T> struct rebind {
        using type = std::array<T, N>;
    };
};

struct vector_wrapper {
    template <class T> struct rebind {
        using type = std::vector<T>;
    };
};

int main() {
    using Tup = std::tuple<int, double, char>;
    using Var = std::variant<int, double, char>;

	// std::tuple<std::array<int, 3>, std::array<double, 3>, std::array<char, 3>>
    using arr = tuple_map<array_wrapper<3>, Tup>::type;

	// std::tuple<std::vector<int>, std::vector<double>, std::vector<char>>
    using vec = tuple_map<vector_wrapper, Tup>::type;
    return 0;
}
```



### 非类型模板推导

#### 简化模板

在 C++17 中允许通过 auto 作为非类型模板参数类型，从而简化模板声明

```cpp
// 一般的非类型模板参数
template <typename T, T value> void foo()
{
    std::cout << value << std::endl;
    return;
}

// C++17 中可以省去 T 类型
template <auto value> void foo()
{
    std::cout << value << std::endl;
    return;
}

int main()
{
    foo<int, 10>(); // 需要显式指定参数类型
    foo<10>();      // value 被推导为 int 类型
}
```



简化类型

```cpp
// 普通模板
template <class T> void f(T);
void g(auto);

// 带 conecpt 的模板
template <std::integral T> void f(T);
void g(std::integral auto);

// 变长参数模板
template <std::integral... T> void f(T...);
void g(std::integral auto...);

template <std::integral T, std::integral U> void f(const T *, U &);
void g(const std::integral auto *, std::integral auto &);

template <class T, std::integral U, std::integral V> void f(T, U, V);
template <class T, std::integral U> void g(T, U, std::integral auto);
```

简化函数模板还可以特化

```cpp
// 简化函数模板
template <class T, std::integral U> void g(T, U, std::integral auto);

// 特化版本
template <> void g(int, int, int);
// 等价写法 template <int, int, int> void g(int, int, int);
```



#### 推导指引

对于较为复杂的类型，需要用户指定推导指引。例如

```cpp
#include <iostream>

template <typename T>
struct Test 
{
    Test(T s) : s(s) {}
    T s;
};

// 所有类型 T 都可以隐式转换为 int
template <typename T>
Test(T) -> Test<int>;

int main()
{
    Test t1(10);
    Test t2(20.5);
    
	// t1,t2 均被推导为 Test<int>
    std::cout << t1.s << std::endl;
    std::cout << t2.s << std::endl;
    
    return 0;
}

```

更有用的写法是

```cpp
#include <iostream>
#include <string>

template <typename T>
struct Test 
{
    Test(T s) : s(s) {}
    T s;
};

// const char* 推导为 std::string
template <typename T>
Test(const char*) -> Test<std::string>;

int main()
{
    Test t1(10);
    Test t2("hello");

    std::cout << t1.s << std::endl;
    std::cout << t2.s << std::endl;
    
    return 0;
}
```



#### 变长推导指引

利用变长模板参数实现变长推导指引

```cpp
#include <iostream>
#include <vector>

template<typename T, size_t size>
struct array 
{
    T data[size];
};

template<class Tu, class... Tv>
array(Tu, Tv...) -> array<std::enable_if_t<(std::is_same_v<Tu, Tv> &&...), Tu>, 1 + sizeof...(Tv)>;

int main()
{
	// 可以不写模板类型，通过 {} 直接推导
    array arr{1, 2, 3, 4, 5};
    for (auto& i : arr.data)
        std::cout << i << " ";
    std::cout << std::endl;
    return 0;
}
```

标准库中的 `std::vector` 也实现了变长推导指引。

> 注意 `array` 没有实现任何构造函数，单纯利用变长推导指引就实现了初始化。



再例如

```cpp
#include <iostream>

template <class... Args>
struct X 
{
	// 不是模板函数，因此不是引用折叠
    X(Args&&... args)
    {
		((std::cout << "X(Args&&... args)" << typeid(args).name()), ...);
        std::cout << std::endl;
    }
};

// 推导指引
template <class... Args>
X(Args... args) -> X<Args...>;

int main()
{
    X x1{"hello", 42, 3.14};

	int a = 10;
    X x2{a};	// 报错，因为 a 不能作为右值传入 Args&&

    int arr[10]{}, arr2[10]{};
    double arr3[10]{};
    X x3{arr, arr2, arr3};
    return 0;
}
```

如果换一种写法就不需要推导指引

```cpp
#include <iostream>

template <class... Args> struct X
{
    X(const Args&...args)
    {
        ((std::cout << "X(Args&&... args)" << typeid(args).name()), ...);
        std::cout << std::endl;
    }
};

// 不需要推导指引
// template <class... Args> X(Args... args) -> X<Args...>;

int main()
{
    X x1{"hello", 42, 3.14};

    int a = 10;
    X x2{a};

    int arr[10]{}, arr2[10]{};
    double arr3[10]{};
    X x3{arr, arr2, arr3};
    return 0;
}
```

这是因为 `"hello", arr, arr2, arr3` 默认被推导为数组类型，只有添加推导指引将其推导为指针类型才能作为右值传入；而使用 `const Args&` 则可以直接将数组类型作为左值传入。



### 编译时运算

借助模板参数，可以实现在编译阶段就完成某些计算操作，减少运行开销。



#### 条件分配

编译器会匹配符合 `<T, std::true_type>` 的模板，注意这里 Enable 是一个类型而不是变量。

> 比较巧妙的是利用默认模板类型以及模板类型推导。如果显式指定 `SafeDivide<float, std::true_type>` 就会直接匹配原始模板；而省去显式指定后，就会由编译器进行类型推导，优先匹配到特化版本。

```cpp
#include <iostream>
#include <type_traits>

// 原始模板，类型为 <T, std::true_type>
template <typename T, typename Enabled = std::true_type> struct SafeDivide
{
    static T Do(T lhs, T rhs)
    {
        return T();
    }
};

// 针对浮点特化，要求 std::is_floating_point<T>::type 是 std::true_type 类型
template <typename T> struct SafeDivide<T, typename std::is_floating_point<T>::type>
{
    static T Do(T lhs, T rhs)
    {
        return lhs / rhs;
    }
};

// 针对整型特化，要求 std::is_integral<T>::type 是 std::true_type 类型
template <typename T> struct SafeDivide<T, typename std::is_integral<T>::type>
{
    static T Do(T lhs, T rhs)
    {
        return rhs == 0 ? 0 : lhs / rhs;
    }
};

int main()
{
    float fa = 1.0f, fb = 2.0f;
    int ia = 4, ib = 2;
    std::cout << SafeDivide<float>::Do(fa, fb) << std::endl;
    std::cout <<SafeDivide<int>::Do(ia, ib) << std::endl;
    return 0;
}
```



#### 阶乘

```cpp
#include <iostream>

using namespace std;

template <int n> struct factorial
{
    static_assert(n >= 0, "Arg must be non-negative");
    static const int value = n * factorial<n - 1>::value;
};

// 出口实现
template <> struct factorial<0>
{
    static const int value = 1;
};

int main()
{
    printf("%d\n", factorial<10>::value);
    return 0;
}
```



#### 条件及加减

```cpp
#include <iostream>

// 通过模板类型实现 if 语句
template <bool cond, typename Then, typename Else> struct IF;

template <typename Then, typename Else> struct IF<true, Then, Else>
{
    typedef Then result;
};

template <typename Then, typename Else> struct IF<false, Then, Else>
{
    typedef Else result;
};

// 判断奇数与偶数，如果为偶则返回 true_type，为奇范围 false_type
template <int N> struct isEven
{
    static const auto RES = IF<N & 1 == 0, std::true_type, std::false_type>::result::value;
};

// 加法结构
template <int nums1, int nums2> struct Add_
{
    static const int value = nums1 + nums2;
};

// 减法结构
template <int nums1, int nums2> struct Sub_
{
    static const int value = nums1 - nums2;
};

// 加减结构（cond = true 则调用加法，否则调用减法
template <bool cond, int nums1, int nums2> struct addSub
{
    static const auto RES = IF<cond, Add_<nums1, nums2>, Sub_<nums1, nums2>>::result::value;
};

int main()
{
    std::cout << isEven<10>::RES << std::endl;
    std::cout << addSub<true, 10, 2>::RES << std::endl;
}
```



#### 循环求和

```cpp
#include <iostream>

// 带条件的循环结构
template <bool condition, typename Body> struct WhileLoop;

// true 时执行下一个循环 cond_value next_type
template <typename Body> struct WhileLoop<true, Body>
{
    // 每次循环都获得下一个循环的类型
    typedef typename WhileLoop<Body::cond_value, typename Body::next_type>::type type;
};

// false 时返回结果
template <typename Body> struct WhileLoop<false, Body>
{
    typedef typename Body::res_type type;
};

// 循环体
template <typename Body> using While_t = WhileLoop<Body::cond_value, Body>;

// 循环体
template <typename Body> struct While
{
    typedef typename WhileLoop<Body::cond_value, Body>::type type;
};
// 上述两种写法等价

namespace my
{

// 常量整型结构
template <class T, T v> struct integral_constant
{
    // 保存值
    static const T value = v;

    // 保存结构类型和值类型
    typedef T value_type;
    typedef integral_constant<T, v> type;
};

} // namespace my

// 求和循环
template <int result, int n> struct SumLoop
{
    // 循环的条件
    static const bool cond_value = n != 0;

    // 循环后的结果
    static const int res_value = result;

    // 循环时的状态
    typedef my::integral_constant<int, res_value> res_type;

    // 循环执行一次后的状态，结果 + n，循环次数 n - 1
    typedef SumLoop<result + n, n - 1> next_type;
};

// 出口结构
template <int n> struct Sum
{
    typedef SumLoop<0, n> type;
};

// 出口结构
template <int n> using Sum_t = SumLoop<0, n>;
// 上述两种写法等价

int main()
{
    std::cout << While<Sum<6>::type>::type::value << std::endl;
    std::cout << While_t<Sum_t<6>>::type::value << std::endl;
    return 0;
}
```



#### 循环遍历

构造编译器循环来遍历元组，使用 `std::integral_constant` 将模板参数包装为编译期变量，从而可以在编译期使用

```cpp
template <size_t Beg, size_t End, class F> void static_for(F f) {
    if constexpr(Beg < End) {
        // 绑定 Beg 到 i，类型为 size_t
        std::integral_constant<size_t, Beg> i;
        f(i);
        static_for<Beg + 1, End>(f);
    }
}

// 遍历元组
Tup tup = {1, 2.0, 'a'};
static_for<0, 3>([&](auto i) { std::cout << std::get<i.value>(tup) << " "; });
std::cout << std::endl;
```

其中 `auto` 作为 lambda 表达式的模板参数，因此 `i.value` 可以作为模板参数。



编译期 for 循环可以用于替代 `std::visit` 来访问如 `std::variant` 变量中的元素

```cpp
std::variant<int, double, char> var = 10.0;
std::visit([&](auto&& arg) { std::cout << arg << std::endl; }, var);

static_for<0, 3>([&](auto i) { 
	if (i.value == var.index()) { 
		std::cout << "Index: " << std::get<i.value>(var) << std::endl; 
	} 
});
```



利用 `std::make_index_sequence` 提供更高效的调用

```cpp
template <size_t... Is, class F> void static_for(F f, std::index_sequence<Is...>) {
    (f(std::integral_constant<size_t, Is>{}), ...);
}

template <size_t N, class F> void static_for(F f) {
    static_for_tuple(f, std::make_index_sequence<N>{});
    // std::make_index_sequence<3> -> std::index_sequence<0, 1, 2>
}

static_for<3>([&](auto i) { 
	if (i.value == var.index()) { 
		std::cout << "Index: " << std::get<i.value>(var) << std::endl; 
	} 
});
```



#### 反射

借助编译器宏实现编译期反射。首先使用编译器宏

```cpp
// 获得函数签名
template <class T, T N> const char* get_enum_name_static() {
#if defined(_MSC_VER)
    return __FUNCSIG__;
#elif defined(__GNUC__) || defined(__clang__)
    return __PRETTY_FUNCTION__;
#else
#    error "Unsupported compiler!"
#endif
}
```

其中 `__FUNCSIG__` 在 MSVC 中定义，而 `__PRETTY_FUNCTION__` 在 GNU, clang 等编译器中定义。



实现编译期循环展开，其中 `func` 用于比较枚举值

```cpp
// 编译期循环展开
template <int Beg, int End, class F, typename std::enable_if_t<Beg == End, int> = 0> void static_for(const F& func) {
}

template <int Beg, int End, class F, typename std::enable_if_t<Beg != End, int> = 0> void static_for(const F& func) {
    // 使用结构内部的枚举，用于构造编译期变量
    struct int_constant {
        enum { value = Beg };
    };
    func(int_constant());
    static_for<Beg + 1, End>(func);
}
```

在外部定义字符串变量，将其传入 lambda 表达式内。这里 `i` 是 `int_constant` 类型，其中的变量是编译期常量，因此可以作为模板参数。

```cpp
// 从枚举获得名称
template <int Beg = 0, int End = 64, class T> std::string get_enum_name(T n) {
    // 保存 get_enum_name_static 模板函数的信息
    std::string s;

    // 循环展开模板。Lambda 表达式中 auto i 相当于 T i
    static_for<Beg, End + 1>([&](auto i) {
        // 只有当 n 与整型转换后对应的枚举值相同时，获得对应模板函数的信息
        if(n == (T)i.value) s = get_enum_name_static<T, (T)i.value>();
    });

    // 如果没找到对应的整型，说明不存在对应枚举，直接返回数字字符串
    if(s.empty()) return std::to_string((int)n);

#if defined(_MSC_VER)
    auto pos = s.find(",");
    pos += 1;
    auto pos2 = s.find_first_of('>', pos);
#else
    auto pos = s.find("N = ");
    pos += 4;
    auto pos2 = s.find_first_of(";]", pos);
#endif

    // 裁剪函数信息，只保留枚举部分；如果还有 ::，将其也裁剪掉（例如 Color::Yellow -> Yellow）
    s = s.substr(pos, pos2 - pos);
    auto pos3 = s.find("::");
    if(pos3 != s.npos) s = s.substr(pos3 + 2);
    return s;
}
```

反过来可以实现从字符串获得枚举

```cpp
// 从名称获得枚举
template <class T, int Beg = 0, int End = 64> T enum_from_name(const std::string& s) {
    for(int i = Beg; i < End; i++) {
        // 将枚举变为字符串比较
        if(s == get_enum_name((T)i)) {
            return (T)i;
        }
    }
    throw std::out_of_range("Out of range");
}
```

使用方法为

```cpp
enum class Color { Red, Yellow, Green };

std::cout << get_enum_name(Color::Yellow) << std::endl;
std::cout << (int)enum_from_name<Color>("Yellow") << std::endl;
```



## 异常处理

### static_assert

通常使用的断言 assert 会在程序运行时检测表达式，如果不满足表达式就会终止程序

```cpp
assert(expression);
```

但有时我们希望在程序运行之前得知某些条件，例如想知道当前是 32 位还是 64 位平台。



静态断言 `static_assert` 在编译时就能进行检查，使用时**不需要引用头文件**。它接收两个参数：

1. 断言表达式，通常需要返回 bool 值
2. 警告信息，通常是一段字符串，在违反断言时提示该信息

```cpp
#include <iostream>                                         
using namespace std;
  
int main()
{
    static_assert(sizeof(long) == 4, "错误, 不是32位平台...");
    cout << "64bit Linux 指针大小: " << sizeof(char*) << endl;
    cout << "64bit Linux long 大小: " << sizeof(long) <<endl;
  
    return 0;
}
```

由于静态断言的表达式是在编译阶段进行检测，所以在它的表达式中不能出现变量，也就是说它**必须是常量表达式**。



### noexcept

在使用异常处理时，我们可以定义

```cpp
void test() throw(int, char, char*)
{
	// ...
}
```

其中 `throw(int, char, char*)` 表示可能抛出的异常，但是这一特性很少使用，因此在 C++11 中被弃用，而表示函数不会抛出异常的 `throw()` 也被 noexcept 异常声明取代。



noexcept 表示修饰的函数不会抛出异常，并且如果其修饰的函数出现异常，则编译器可以选择直接调用 std::terminate 函数终止程序运行

```cpp
void test() noexcept
{
	// ...
}
```

使用 `noexcept` 可以提高编译器的优化机会，因为它可以确保在函数调用时不必进行异常处理路径的准备：

- 在移动构造函数和移动赋值运算符中使用 `noexcept` 可以向编译器提供优化信息，让它知道这些操作不会抛出异常。
- 这对于标准容器和算法的性能至关重要，因为它们在元素移动时可以避免不必要的异常检查。

noexcept 可以接受一个常量表达式作为参数

```cpp
void test() noexcept(const expression);
```

常量表达式的结果会被转换成一个 bool 类型的值：

* true 表示函数不会抛出异常
* false 表示可能抛出异常
* **不带常量表达式的 noexcept 相当于 `noexcept(true)`** ，即不会抛出异常

在 C++17 后，`noexcept` 成为函数类型的一部分

```cpp
#include <iostream>

void f1() noexcept {}
void f2() {}

int main() {
  std::cout << std::boolalpha << "f1() noexcept: " << noexcept(f1())
            << std::endl;
  std::cout << std::boolalpha << "f2() noexcept: " << noexcept(f2())
            << std::endl;
  return 0;
}
```

通过 `noexcept` 获得函数是否是 `noexcept` 函数。



## 内存管理

### 延迟构造

使用 [placement new](#placement new) 技术，可以在申请内存后，延迟调用构造函数。例如

```cpp
Vec *U = new Vec[s];
for (int i = 0; i < s; i++)
{
	// 在这里调用构造函数初始化
	new (U + i) Vec(n, 0);
}
```



### 分配器

分配器是 C++11 引入的用于控制内存批量申请和销毁的类型

```cpp
template <class T> class allocator;
```

例如

```cpp
#include <iostream>
#include <memory>

int main()
{
    // 整型分配器
    std::allocator<int> myAllocator;

    // 分配 5 个整型内存
    int *arr = myAllocator.allocate(5);
	arr[3] = 10;
    
    // 构造最后一个元素
    myAllocator.construct(arr, 100);

    // 销毁 5 个内存
    myAllocator.deallocate(arr, 5);

    return 0;
}
```



还可以用于销毁对象，例如

```cpp
#include <iostream>
#include <memory>
#include <string>

int main()
{
    // 对象分配器
    std::allocator<std::string> myAllocator;

    // 申请内存
    std::string *str = myAllocator.allocate(3);

    // 构造对象
    myAllocator.construct(str, "Geeks");
    myAllocator.construct(str + 1, "for");
    myAllocator.construct(str + 2, "Geeks");

    // 销毁对象
    myAllocator.destroy(str);
    myAllocator.destroy(str + 1);
    myAllocator.destroy(str + 2);

    // 销毁内存
    myAllocator.deallocate(str, 3);
}
```

这意味着已分配的内存实际上可以反复使用，只要没有销毁内存，就可以在销毁对象后重新构造。



### 内存对齐

C++11 引入新的关键字 alignof 和 alignas 控制内存对齐

```cpp
#include <iostream>

struct Storage
{
    char a;
    int b;
    double c;
    long long d;
};

// 按照最大的占用内存长度对齐，这里是 long long 或 double
struct alignas(std::max_align_t) AlignasStorage
{
    char a;
    int b;
    double c;
    long long d;
};

int main()
{
    // 获得内存对齐值
    std::cout << alignof(Storage) << std::endl;
    std::cout << alignof(AlignasStorage) << std::endl;
    return 0;
}
```



## SFINAE

### 基本介绍

所谓 SFINAE 是 ”**S**ubstitution **F**ailure **I**s **N**ot **A**n **E**rror 替换失败不是错误“的简写。也就是说，如果替换产生了无效的代码，编译器不应该报错，而是尝试其他可用的重载。例如

```cpp
template <typename T> void f(const T &t, typename T::iterator *it = nullptr)
{
}

void f(...)
{
}

f(1); // 调用 void f(...) { }
```

由于 int 没有迭代器，因此第一个模板的替换完全错误，但是**编译器会寻找其它可用的重载函数**，所以可以正常执行。



再例如下面这个例子

```cpp
#include <iostream>

struct X
{
    typedef int type;
};

struct Y
{
    typedef int type2;
};

template <typename T> void foo(typename T::type);
template <typename T> void foo(typename T::type2);
template <typename T> void foo(T);

int main()
{
    foo<X>(5);		// 匹配第一个版本
    foo<Y>(15);		// 匹配第二个版本
    foo<int>(20);	// 匹配第三个版本
    return 0;
}
```

只要存在可以匹配的重载，就不会报错。



值得注意的是，要遵循“函数体之外的替换才是安全的”原则。反例如下

```cpp
template <typename T> void f(T t)
{
    t.hhhh();
}

void f(...)
{
}

f(1); // 调用 void f(T t)
```

此时编译器将会报错，因为替换内容出现在函数体内，而 int 没有 `hhhh()` 方法。



### 类型替换

假设我们希望根据传入的模板类型区分调用的成员函数。考虑如下的代码

```cpp
struct ICounter
{
    virtual void increase() = 0;
};

struct Counter
{
    void increase(){};
};
```

我们想实现如下的功能

```cpp
template <typename T> void increase(T &obj)
{
    if (T == int)
        obj++;
    if (T == Counter)
        obj.increase();
}
```

可以借助 `enable_if` 和 SFINAE 特性

```cpp
// 第二个参数匹配为 T* = nullptr
template <typename T> void increase(T &obj, std::enable_if<std::is_base_of<ICounter, T>::value>::type * = nullptr)
{
    obj.increase();
}

// 第二个参数匹配为 T* = nullptr
template <typename T> void increase(T &obj, std::enable_if<std::is_integral<T>::value>::type * = nullptr)
{
    obj++;
}
```

这里 `enable_if` 接收 `<true, T>`，然后返回 `T` 类型。利用替换原则，只有当 ICounter 是 T 的基类时，才会匹配第一个版本；当 T 是 int 时，才会匹配第二个版本。



在 C++11 中，正式提供了表达式 SFINAE，这时候就可以直接在编译期调用方法实现匹配

```cpp
// 现在可以获得表达式类型了
template <typename T> void increase(T &obj, std::decay_t<decltype(obj.increase())>* = nullptr)
{
    obj.increase();
}

template <typename T> void increase(T &obj, std::decay_t<decltype(obj++)> * = nullptr)
{
    obj++;
}
```



### 万能引用

右值引用 `&&` 作为参数时，只能匹配右值引用。然而我们希望它可以匹配任意输入类型，这就是

```cpp
template <typename T> void foo(T&& t);
```

这里利用模板实现了万能引用：由于引用折叠，`T& &&` 将会折叠为左值引用，因此它可以接收任意输入类型。



加入我们希望针对浮点型输入进行特化，直观的写法是

```cpp
void foo(float&& t);
```

然而它只能匹配右值引用。我们利用 SFINAE 来实现匹配 float 的任意引用输入

```cpp
template <typename T> void foo(T&& t, typename std::enable_if<std::is_same<std::decay_t<T>, float>::value>::type* = nullptr);
```



### 判断类型

利用 SFINAE 机制，容易实现 `std::is_same_v` 方法

```cpp
#include <iostream>

// 默认情况下，value 为 false
template <typename T, typename T2>
struct is_same {
    const static bool value = false;
};

// 如果 T 和 T2 相同，则 value 为 true
template <typename T>
struct is_same<T, T> {
    const static bool value = true;
};

// 变量模板
template <typename T, typename T2>
bool is_same_v = is_same<T, T2>::value;

int main()
{
    std::cout << std::boolalpha << is_same_v<int, int> << std::endl;
    std::cout << std::boolalpha << is_same_v<int, double> << std::endl;
    return 0;
}
```



### 序列化

SFINAE 的重要用途就是序列化和反射。对于 Python 来说，可以借助反射实现下面的序列化操作

```python
class A(object):
    # 重载 str，可转换为字符串
    def __str__(self):
        return "I am a A"


class B(object):
    # 自定义序列化函数
    def serialize(self):
        return "I am a B"


class C(object):
    def __init__(self):
        # 定义序列化成员
        self.serialize = 0

    def __str__(self):
        return "I am a C"


# 序列化函数
def serialize(obj):
    # 检查是否有 'serialize' 成员（不保证是方法，有可能是变量）
    if hasattr(obj, "serialize"):
        # 如果 'serialize' 是一个方法，就直接调用
        if hasattr(obj.serialize, "__call__"):
            return obj.serialize()

    # 否则直接转换为字符串
    return str(obj)


a = A()
b = B()
c = C()

print(serialize(a))  # output: I am a A.
print(serialize(b))  # output: I am a B.
print(serialize(c))  # output: I am a C.
```



#### C98 实现

在 C98 中已经可以利用 `sizeof` 来实现判断可序列化的操作

```cpp
// 判断 T 是否可序列化
template <class T> struct hasSerialize
{
    // 用于编译期比较，yes 1 字节，no 2 字节
    typedef char yes[1];
    typedef yes no[2];
    
    // 用于具体判断
    template <typename U, U u> struct reallyHas;
   
    // 测试函数，分别用于接收普通成员函数和常量成员函数（带 const 的成员函数）
    // 如果 C::serialize 存在，则会进入这两个方法之一
    template <typename C> static yes &test(reallyHas<std::string (C::*)(), &C::serialize> *)
    {
    }
    template <typename C> static yes &test(reallyHas<std::string (C::*)() const, &C::serialize> *)
    {
    }

    // 此函数需要空模板，因为检测时需要调用 test<T>(0)
    template <typename> static no &test(...)
    {
    }

    // 判断返回类型的大小是否等于 yes 的大小
    // 0 可转换为对应类型的指针
    static const bool value = sizeof(test<T>(0)) == sizeof(yes);
};

std::cout << hasSerialize<A>::value << std::endl;
std::cout << hasSerialize<B>::value << std::endl;
std::cout << hasSerialize<C>::value << std::endl;
```

我们可以将判断函数再次封装，利用模板类型推导省去显式指定类型

```cpp
// 空结构
struct A
{
};

// 外部序列化
std::string to_string(const A &)
{
    return "I am a A!";
}

// 可序列化结构
struct B
{
    std::string serialize() const
    {
        return "I am a B!";
    }
};

// 具有“错误”序列化成员
struct C
{
    std::string serialize;
};

// 外部序列化
std::string to_string(const C &)
{
    return "I am a C";
}

// 继承类
struct D : A
{
    std::string serialize() const
    {
        return "I am a D!";
    }
};

// 将判断进行封装
template <class T> bool testHasSerialize(const T &)
{
    return hasSerialize<T>::value;
}

D d;
A &a = d;	// 实际类型为 A，因此判断为不可序列化
std::cout << testHasSerialize(d) << std::endl; // Output 1.
std::cout << testHasSerialize(a) << std::endl; // Output 0.
```

注意当前的判断只考虑传入类型，而不考虑实际类型。



接着我们尝试实现序列化函数。似乎可以这样写

```cpp
template <class T> std::string serialize(const T &obj)
{
    // 根据是否可序列化调用不同的方法
    if (hasSerialize<T>::value)
        // 如果 obj 没有 serialize() 方法，则编译报错
        return obj.serialize();
    else
        return to_string(obj);
}
```

然而，对于没有序列化函数的类，上述函数将会报错。



要解决这个问题，我们需要将其拆分为两个函数，利用模板替换特性进行匹配。首先需要完成 `enable_if` 模板（在 Type Trait 中有定义）

```cpp
// 原始模板，接收两个参数，什么也不做
template <bool B, class T = void> struct enable_if
{
};

// 特化版本，提供一个 type 类型
template <class T> struct enable_if<true, T>
{
    typedef T type;
};
```

利用它进行匹配

```cpp
// 如果 hasSerialize<T>::value 为 true，则匹配此函数得到 enable_if<true, std::string>，返回 type 为 std::string
template <class T> typename enable_if<hasSerialize<T>::value, std::string>::type serialize(const T &obj)
{
    // 存在序列化方法
    return obj.serialize();
}

// 如果 hasSerialize<T>::value 为 false，则匹配此函数（注意 !hasSerialize<T>::value 前面有叹号取反）
template <class T> typename enable_if<!hasSerialize<T>::value, std::string>::type serialize(const T &obj)
{
    return to_string(obj);
}

A a;
B b;
C c;

std::cout << serialize(a) << std::endl;
std::cout << serialize(b) << std::endl;
std::cout << serialize(c) << std::endl;
```

这里利用了一个很巧妙的手法，当 `hasSerialize<T>::value` 为 `false` 时，由于 SFINAE 原则，匹配第一个函数失败，但是可以继续匹配第二个函数。



#### C11 实现

利用 `decltype` 和 `constexpr` 可以更简单地实现

```cpp
template <class T> struct hasSerialize
{
    // 通过 std::declval<C>() 获得 C 的右值，调用序列化函数
    // 利用逗号运算符，表达式结果为 bool，作为返回值类型
    template <typename C> static constexpr decltype(std::declval<C>().serialize(), bool()) test(int)
    {
        return true;
    }

    // 用于接收不具有序列化函数的类型
    template <typename C> static constexpr bool test(...)
    {
        return false;
    }

    // 现在可以定义常量静态成员，在编译期获得返回值
    static constexpr bool value = test<T>(int());
};
```

注意到现在所有成员都是常量表达式，所以在编译期就完成操作。



由于 `std::true_type, std::false_type` 可以被继承，从而可以实现一个更巧妙的版本

```cpp
// 原始模板，一般类型将匹配此结构
template <typename T, typename = std::string> struct hasSerialize : std::false_type
{
};

// 特化版本，具有 serialize() 的类型匹配此结构
template <typename T> struct hasSerialize<T, decltype(std::declval<T>().serialize())> : std::true_type
{
};
```

利用模板的偏特化来区分版本，分别继承不同的类型。



### 有效成员

利用 C++14 的新特性，可以将对序列化函数的判断推广到任意成员函数。首先利用 auto 可作为返回值和 lambda 表达式参数的特性，实现 `is_valid` 模板

```cpp
// 单纯记录类型
template <typename UnnamedType> struct container
{
};

template <typename UnnamedType> constexpr auto is_valid(const UnnamedType &t)
{
    // 调用默认构造，返回一个容器，模板类型是传入类型
    return container<UnnamedType>();
}

// 利用 lambda 表达式封装了 t 的一个成员 serialize()，传入 is_valid
// 现在 test 记录了这个 lambda 表达式
auto test = is_valid([](const auto &t) -> decltype(t.serialize()) {});
```



我们可以进一步扩充整个类型

```cpp
// 进一步扩充
template <typename UnnamedType> struct container
{
  private:
    // std::declval<UnnamedType>() 获得一个 UnnamedType 类型的对象
    // 如果 UnnamedType 是可调用的对象，则匹配此模板函数，执行 UnnamedType()( std::declval<Param>() )
    template <typename Param>
    constexpr auto testValidity(int) -> decltype(std::declval<UnnamedType>()(std::declval<Param>()), std::true_type())
    {
        return std::true_type();
    }

    // 用于接收 UnnamedType 不是可调用对象的情况
    template <typename Param> constexpr std::false_type testValidity(...)
    {
        return std::false_type();
    }

  public:
    // 重载使得 container 成为可调用对象。传入的 Param 是一个参数，作为 UnnamedType 调用的参数
    template <typename Param> constexpr auto operator()(const Param &p)
    {
        // 返回类型为 std::true_type 或 std::false_type
        return testValidity<Param>(int());
    }
};

// is_valid 模板，传入一个 UnnamedType 类型，返回 container 是一个可调用对象
template <typename UnnamedType> constexpr auto is_valid(const UnnamedType &t)
{
    return container<UnnamedType>();
}

// 传入的是可调用对象，现在 hasSerialize 是一个可调用对象，它接收的“参数的类型”将会传递给 testValidity 中的 Param 模板参数
// 注意 UnnamedType 的类型是这个 lambda 表达式的类型
auto hasSerialize = is_valid([](auto &&x) -> decltype(x.serialize()) {});
```

始终需要注意，**UnnamedType 的类型是一个 lambda 表达式**，它用来调用 `serialize()` 方法；而 Param 是被检验是否存在 `serialize()` 方法的类型。



我们得到一个通用模板，还可用于检测任意函数的存在性。例如

```cpp
auto hasSomeMethod = is_valid([](auto &&x) -> decltype(x.someMethod()) {});
```

这里 `hasSomeMethod` 就可以用来检测一个类型是否具有 `someMethod()` 方法。



现在使用 `hasSerialize` 可以实现序列化函数

```cpp
// serialize 接受一个参数 obj，它作为 hasSerialize 的传入参数
// Param 的类型是 T，在 testValidity 调用时，UnnamedType 类型是 [](auto &&x) -> decltype(x.serialize()) {}
// 它将会以 obj 作为参数，调用 obj 的 serialize() 方法。最后 decltype 获得对应的类型
template <class T>
auto serialize(T &obj) -> typename std::enable_if<decltype(hasSerialize(obj))::value, std::string>::type
{
    return obj.serialize();
}

// 同理 ! 取反，用于匹配没有序列化方法的类型
template <class T>
auto serialize(T &obj) -> typename std::enable_if<!decltype(hasSerialize(obj))::value, std::string>::type
{
    return to_string(obj);
}
```

当前的 `hasSerialize` 目的就是调用 obj 的 `serialize()` 方法，完成模板匹配。匹配到正确的模板函数后再进行序列化返回。



最后，利用变长模板参数，可以实现一次检测多个类型的模板

```cpp
template <typename UnnamedType> struct container
{
  private:
    // 改为变长模板参数
    template <typename... Params>
    constexpr auto test_validity(int /* unused */)
        -> decltype(std::declval<UnnamedType>()(std::declval<Params>()...), std::true_type())
    {
        return std::true_type();
    }

    // 改为变长模板参数
    template <typename... Params> constexpr std::false_type test_validity(...)
    {
        return std::false_type();
    }

  public:
    // 改为变长模板参数
    template <typename... Params> constexpr auto operator()(Params &&...)
    {
        return test_validity<Params...>(int());
    }
};

// 由于 is_valid 只需要接受一个 lambda 表达式，因此不变
template <typename UnnamedType> constexpr auto is_valid(UnnamedType &&t)
{
    return container<UnnamedType>();
}

// 现在可以接收两个类型的参数
auto test = is_valid([](auto &&a, auto &&b) -> decltype(a.serialize(), b.serialize()) {});
```



### 判断类

利用 SFINAE 技术，可以实现一个判断类的模板

```cpp
#include <iostream>

// 判断 T 是否是一个类的模板
template <typename T> class IsClassT
{
  private:
    typedef char One;
    typedef struct
    {
        char a[2];
    } Two;
    
    // 如果 C 是类，则会调用此函数
    // int C::* 是类成员指针，即使 C 没有整型成员，也可以声明这个类型，只是不能使用
    template <typename C> static One test(int C::*);

    // 如果 C 不是类，就会调用此函数
    // ... 接收任意参数
    template <typename C> static Two test(...);

  public:
    enum
    {
        // 获得返回值大小，One 对应 1 字节，Two 对应 2 字节
        Yes = sizeof(IsClassT<T>::test<T>(0)) == 1
    };

    enum
    {
        No = !Yes
    };
};

// 空结构
struct A
{
};

int main()
{
    // 0 不能转换为 int int::* 因为 int 不是类，所以它不能有成员指针
    std::cout << IsClassT<int>::Yes << std::endl;
    std::cout << IsClassT<A>::Yes << std::endl;

    return 0;
}
```

在上面的实现中，因为 int 不存在成员指针，0 不能转换为 `int int::*`，则对其中一个 test 的调用完全错误。但是这一替换不会报错，因为编译器会寻找其它可用的重载函数。



### 任意类型输出

实现任意类型输出

```cpp
// output_container.h
#pragma once

#include <ostream>     // std::ostream
#include <type_traits> // std::false_type/true_type/decay_t/is_same_v
#include <utility>     // std::declval/pair

// 如果生成此结构，说明不是 pair
template <typename T> struct is_pair : std::false_type
{
};

// 对 is_pair 的模板特化 std::pair<T, U>，如果生成此结构，说明传入的是 pair
template <typename T, typename U> struct is_pair<std::pair<T, U>> : std::true_type
{
};

// 将 T 作为模板，如果它是 pair，则会进入特化版本；否则进入非特化版本。注意内联变量只在 C++17 中有效
template <typename T> inline constexpr bool is_pair_v = is_pair<T>::value;
//template <typename T>  constexpr bool is_pair_v = is_pair<T>::value;

// 用于判断 T 是否具有 stream 输出
template <typename T> struct has_output_function
{
	// std::declval 返回 std::ostream & 的右值引用，如果 U 具有 stream 输出，则会产生此函数，返回 std::true_type
	template <class U> static auto output(U* ptr) -> decltype(std::declval<std::ostream&>() << *ptr, std::true_type());

	// 否则会产生此函数，返回 std::false_type
	template <class U> static std::false_type output(...);

	// 调用 output 通过返回值判断类型
	static constexpr bool value = decltype(output<T>(nullptr))::value;
};

// 将 T 作为模板传入，检测是否有 output 函数。注意内联变量只在 C++17 中有效
template <typename T> inline constexpr bool has_output_function_v = has_output_function<T>::value;
//template <typename T> constexpr bool has_output_function_v = has_output_function<T>::value;

// 为 pair 设定的输出函数
template <typename T, typename U> std::ostream& operator<<(std::ostream& os, const std::pair<T, U>& pr);

// 对于容器 Cont，如果它具有 key_type 键值对，并且它的值为 pair 类型的容器，就会使用此输出函数
template <typename T, typename Cont>
auto output_element(std::ostream& os, const T& element, const Cont&, const std::true_type)
-> decltype(std::declval<typename Cont::key_type>(), os);

// 针对其它所有容器
template <typename T, typename Cont>
auto output_element(std::ostream& os, const T& element, const Cont&, ...) -> decltype(os);

// 如果 T 没有输出函数，就会定义此函数
template <typename T, typename = std::enable_if_t<!has_output_function_v<T>>>
auto operator<<(std::ostream& os, const T& container) -> decltype(container.begin(), container.end(), os)
{
	using std::decay_t;
	using std::is_same_v;

	// 判断容器元素是否是 char，只要不是，就输出 {
	using element_type = decay_t<decltype(*container.begin())>;
	constexpr bool is_char_v = is_same_v<element_type, char>;
	if constexpr (!is_char_v)
		os << "{ ";

	if (!container.empty())
	{
		auto end = container.end();
		bool on_first_element = true;
		for (auto it = container.begin(); it != end; ++it)
		{
			if constexpr (is_char_v)
			{
				if (*it == '\0')
					break;
			}
			if constexpr (!is_char_v)
			{
				if (!on_first_element)
					os << ", ";
				else
					on_first_element = false;
			}

			// 根据类型是否为 char 来调用输出模板函数
			output_element(os, *it, container, is_pair<element_type>());
		}
	}

	if constexpr (!is_char_v)
		os << " }";
	return os;
}

// 针对元素为键值对的类型的输出实现
template <typename T, typename Cont>
auto output_element(std::ostream& os, const T& element, const Cont&, const std::true_type)
-> decltype(std::declval<typename Cont::key_type>(), os)
{
	// element.second 可能还是一个 pair，则调用下面的 pair 流式输出
	os << element.first << " => " << element.second;
	return os;
}

// 针对其它类型的输出实现
template <typename T, typename Cont>
auto output_element(std::ostream& os, const T& element, const Cont&, ...) -> decltype(os)
{
	os << element;
	return os;
}

// 对 pair 实现流式输出，如果 pr.second 还是 pair，则会递归调用
template <typename T, typename U> std::ostream& operator<<(std::ostream& os, const std::pair<T, U>& pr)
{
	os << '(' << pr.first << ", " << pr.second << ')';
	return os;
}
```

配置要求 `C++17` 以上

```cmake
cmake_minimum_required(VERSION 3.28)
project(main)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(main main.cpp output_container.h)
```



## 约束与概念

### 基本介绍

在 C++20 中引入了约束 constraint，类模板、函数模板以及非模板函数（通常是类模板的成员），可以关联到约束。约束指定了对模板实参的要求，可以被用于选择最恰当的函数重载和模板特化。



### 概念

概念是要求的具名集合，必须在命名空间作用域中出现。定义形式为

```cpp
template <class T>
concept Test = std::is_same_v<T, int>;
```

右侧为约束表达式。



#### 占位类型说明符

使用时用其修饰类型

```cpp
#include <iostream>

template <class T>
concept Test = std::is_same_v<T, int>;

void f(Test auto x)
{
    std::cout << "x is an int: " << x << std::endl;
}

int main()
{
    f(1);   // OK
    f(1.2); // Error: T is int, but x is double
    return 0;
}
```

概念不能被显式实例化、显式特化或部分特化，即不能修改概念的原始约束。



#### 标识表达式

概念可以在标识表达式中命名。该标识表达式的值在满足约束表达式时为 true，否则为 false 。例如

```cpp
#include <iostream>

template <class T>
concept Test = std::is_same_v<T, int>;

int main()
{
    // 输出 true
    std::cout << std::boolalpha << Test<int> << std::endl;
    return 0;
}
```



#### 类型模板形参

修饰类型模板形参

```cpp
#include <iostream>

template <class T>
concept Test = std::is_same_v<T, int>;

// 修饰 T
template <Test T> void foo()
{
    std::cout << "foo<int>() called" << std::endl;
}

int main()
{
    foo<int>();    // calls foo<int>()
    foo<double>(); // error: no matching function for call to 'foo'
    return 0;
}
```



#### 复合要求

结合要求 requires 使用

```cpp
#include <iostream>

template <class T>
concept Test = std::is_same_v<T, int>;

// 要求 a + b 必须返回 Test 约束的类型 int
template <class T>
concept Test2 = requires(T a, T b) {
    {
        a + b
    } -> Test;
};

struct X
{
    // X 必须实现 Test2 约束的 operator+，返回 Test 约束的类型 int
    auto operator+(X const &other) const
    {
        return 42;
    }
};

void f(Test2 auto a)
{
}

int main()
{
    f(1);
    f(X());
    return 0;
}
```



### 要求

Requires 表达式产生一个描述约束的 bool 类型的纯右值表达式。



#### 简单要求

简单要求是一个任何不以关键字 requires 开始的表达式语句。它断言该表达式是有效的。表达式是一个不求值的操作数，只检查语言的正确性。例如

```cpp
#include <iostream>

template <class T>
concept Addable = requires(T a, T b) {
    a + b; // 仅要求 a + b 合法
};

void f(Addable auto a, Addable auto b)
{
    std::cout << a + b << std::endl;
}

int main()
{
    f(5, 6);     // OK
    f("a", "b"); // 编译错误，类型不满足 Addable
    return 0;
}
```



#### 类型要求

类型要求以关键字 `typename` 接一个可选限定的类型名称。该要求是指命名的类型是有效的。例如

```cpp
#include <iostream>

template <class T> using Ref = T &;

template <class T>
concept C = requires {
    typename T::inner; // 要求嵌套类型 inner
    typename Ref<T>;   // 要求引用类型 Ref
};

struct X
{
    struct inner
    {
    };
};

int main()
{
    std::cout << std::boolalpha << "C<X> = " << C<X> << std::endl;
    return 0;
}
```



#### 复合要求

复合要求指定命名表达式的属性。例如

```cpp
#include <iostream>

template <typename T>
concept C = requires(T t) {
    {
        t + 1
    } -> std::same_as<int>;
};

int main()
{
    std::cout << std::boolalpha << "C<int> = " << C<int> << std::endl;       // true
    std::cout << std::boolalpha << "C<double> = " << C<double> << std::endl; // false
    return 0;
}
```



可以内部嵌套 requires 表达式

```cpp
#include <iostream>

template <typename T>
concept C = requires(T t) {
    std::same_as<T, int>;  			// 要求 std::same_as<T, int> 有效
    requires std::same_as<T, int>;  // 要求 std::same_as<T, int> 为真
};

int main()
{
    std::cout << std::boolalpha << "C<int> = " << C<int> << std::endl;       // true
    std::cout << std::boolalpha << "C<double> = " << C<double> << std::endl; // false
    return 0;
}
```



#### 变长要求

可以结合变长模板参数给出变长要求，表达式在 `...` 处依次展开

```cpp
template <class T0, class... Ts>
    requires(true && ... && std::is_convertible_v<Ts, T0>)
std::array<T0, sizeof...(Ts) + 1> func(T0 t0, Ts... ts) {
    // 要求 Ts 都可以转换为 T0 类型
    return {t0, ts...};
}
```

其中 `true` 用于变长参数为空的情况，此时得到

```cpp
template <class T0>
    requires(true)
std::array<T0, 1> func(T0 t0) {
	return {t0};
}
```



#### 子句

可以将 requires 作为子句使用。例如

```cpp
#include <iostream>

// 写法 1
template <typename T> requires std::is_same_v<T, int>
void f(T t)
{
    std::cout << "f(int) called" << std::endl;
}

// 写法 2
template <typename T>
void f(T t)
    requires std::is_same_v<T, int>
{
    std::cout << "f(int) called" << std::endl;
}

int main()
{
    f(1);   // OK
    f(2.0); // Error: no matching function for call to 'f'
    return 0;
}
```

配合 concept 使用

```cpp
template <typename T>
concept Addable = requires(T a, T b) { a + b; };

template <typename T>
    requires Addable<T>
void f(T t)
{
}
```



子句中当然可以引入约束表达式

```cpp
template <typename T>
    requires (requires(T a) {
        a + a;
        requires std::is_integral_v<T>;
    })
void f(T t)
{
}
```



#### 对比旧版

使用 C++20 之前的等价写法

```cpp
template <typename T>
    requires std::is_integral_v<T>
void f1(T t)
{
}

template <typename T, typename = std::enable_if_t<std::is_integral_v<T>>>
void f2(T t)
{
}
```



### 约束

约束是逻辑操作和操作数的序列，它指定了对模板实参的要求。它们可以在 requires 表达式中出现，也可以直接作为概念的主体。



#### 合取

两个约束的合取是通过在约束表达式中使用 `&&` 运算符构成的

```cpp
template <class T>
concept Integral = std::is_integral_v<T>;

template <class T>
concept SignedIntegral = Integral<T> && std::is_signed_v<T>;

template <class T>
concept UnsignedIntegral = Integral<T> && std::is_unsigned_v<T>;
```

约束合取依然具有短路特性：如果左侧约束不满足，则不会对右侧约束模板替换。



#### 析取

两个约束的析取是通过在约束表达式中使用 `||` 运算符构成的

```cpp
template <class T = void> requires EqualityComparable<T> || Same<T, void> struct equal_to;
```

约束析取依然具有短路特性：如果左侧约束满足，则不会对右侧约束模板替换。



#### 不可分割约束

不可分割约束由一个表达式和从中出现的模板形参到模板实参的映射组成，这种映射称为形参映射。



