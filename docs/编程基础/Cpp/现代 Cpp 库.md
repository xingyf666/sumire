# 现代 Cpp 库

## 源码信息库

在 C++20 中提供了获得源码具体信息的库，例如文件名、行号和函数名。过去需要使用 `__LINE__, __FILE__` 的预定义宏可以被很好地取代。



例如通过默认实参实现获得位置信息

```cpp
#include <iostream>
#include <source_location>
#include <string_view>

void log(std::string_view message, const std::source_location& location = std::source_location::current())
{
    std::cout<< location.file_name() << ":" << location.line() << " " << message << std::endl;
}

int main()
{
	// 将会获得 12 行
    log("Hello, world!");
    return 0;
}
```

由于默认实参在函数调用位置初始化，因此位置信息是函数调用处的位置信息而非函数定义处的位置信息。



## 范围库

### 基本介绍

在对容器数据进行处理时，可能需要修改容器本身，甚至需要创建一个原容器的拷贝，而范围库的引入为过滤和处理容器提供了一种新的范例。例如

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    const std::vector<int> v = {1, 2, 3, 4, 5};
    auto result = std::ranges::take_view(v, 3);
    for (int i : result)
        std::cout<< i << " ";
    return 0;
}
```

将输出

```shell
1 2 3
```

其中 `take_view` 返回前面几个元素，这不会修改容器也不会创建新的容器。等价的写法是

```cpp
auto result = v | std::views::take(3);
```

利用管道运算符和视图适配器获得范围。



### 视图适配器

视图适配器 `std::views` 位于 `std::ranges::views` 中，前者是后者的别名。它们是轻量级的、非拥有的数据结构，可以对序列进行懒操作。



#### 视图方法

视图通常是通过范围适配器（range adaptors）来创建的，这些适配器能够对现有的序列进行过滤、变换等操作，而不会产生额外的存储开销。例如

* `std::views::filter` 过滤序列中的元素
- `std::views::transform` 对序列中的元素进行变换
- `std::views::take` 获取序列的前 N 个元素
- `std::views::drop` 跳过序列的前 N 个元素
- `std::views::reverse` 反转序列

使用范例

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    const std::vector<int> v = {1, 2, 3, 4, 5};
    
    auto result = v | std::views::take(3);
    auto result2 = v | std::views::filter([](int i) { return i % 2 == 0; });
    auto result3 = v | std::views::transform([](int i) { return i * 2; });
    
    for (int i : result)
        std::cout<< i << " ";
    std::cout<< "\n";
    for (int i : result2)
        std::cout<< i << " ";
    std::cout<< "\n";
    for (int i : result3)
        std::cout<< i << " ";
    std::cout<< "\n";
    return 0;
}
```



#### 适配器

将输入的 range 转化成 view

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    std::vector v{1, 2, 3, 4, 5};
    // all 将 v 转化成 view
    for (int i : std::views::all(v) | std::views::reverse)
        std::cout<< i << " ";
    std::cout<< std::endl;

    return 0;
}
```



#### 创建视图

视图是一个对象，它操作一个范围并返回一个修改后的范围。视图不包含自己的数据，不保留底层数据的副本，只根据需要返回底层元素的迭代器。例如

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};

	// 创建视图
    std::ranges::take_view tv{v, 3};
    for (auto i : tv)
        std::cout<< i << " ";
    std::cout<< std::endl;
    return 0;
}
```



#### 管道操作

视图操作将会返回视图类型，因此可以嵌套使用

```cpp
auto result = v | std::views::take(3) | std::views::take(2);
```

可以自己实现管道操作

```cpp
#include <iostream>
#include <vector>

template <class U, class F> std::vector<U> &operator|(std::vector<U> &vec, F f)
{
    for (auto &i : vec)
        f(i);
    return vec;
}

int main()
{
    std::vector v{1, 2, 3, 4, 5};
    v | [](int &i) { i *= 2; } | [](int i) { std::cout<< i << " "; };
    std::cout<< std::endl;
    return 0;
}

```



#### 字符串视图

在 C++17 中引入了 `std::string_view`，拥有和 `const char*` 一样小的传递开销

```cpp
#include <iostream>
#include <ranges>
#include <vector>

void print(auto &&r)
{
    std::cout<< std::views::all(r) << std::endl;
}

int main()
{
    print(std::string_view{"hello"});
    return 0;
}
```



### 范围操作

#### 范围算法

新的算法库重新实现了常用的算法，并在 `std::ranges` 中重新封装。旧版的排序写法为

```cpp
std::sort(v.begin(), v.end());
```

现在可以用范围调用

```cpp
std::ranges::sort(v);
```

配合视图使用

```cpp
std::ranges::sort(std::ranges::views::drop(v, 5));
std::ranges::sort(v | std::views::reverse | std::views::drop(5));
```



#### 迭代器

范围可以获得迭代器

```cpp
#include <iostream>
#include <ranges>
#include <vector>

int main()
{
    std::vector v{3, 1, 4};
    if (std::ranges::find(v, 4) != std::ranges::end(v))
        std::cout<< "Found 4 in the vector\n";
    return 0;
}
```



#### 模板类型

范围作为模板类型

```cpp
#include <iostream>
#include <ranges>
#include <vector>

// std::ranges::input_range 是一个 concept，它要求一个类型可以转换为迭代器，并且可以用范围式的语法来访问其元素
template <std::ranges::input_range R> void print(char id, R &&r)
{
    if (std::ranges::empty(r))
    {
        std::cout<< id << " is empty\n";
        return;
    }

    std::cout<< id << " contains:\n";
    for (auto &&elem : r)
        std::cout<< elem << ' ';
    std::cout<< '\n';
}

int main()
{
    print('v', std::vector{1, 2, 3, 4, 5});
    print('s', "hello world");
    print('c', std::string{"hello world"});
    return 0;
}
```



#### 字符串复制

可以获得范围对应的数据，然后执行对应数据类型的操作。例如

```cpp
#include <iostream>
#include <ranges>

int main()
{
    std::string s = "Hello, world!";
    char a[20];
    std::strcpy(a, std::ranges::data(s));
    std::cout<< a << std::endl;
    return 0;
}
```



### 范围概念

#### 范围迭代器

概念 `std::ranges::range` 要求一个可迭代类型

```cpp
template <class T>
concept range = requires(T& t)
{
	ranges::begin(t);
	ranges::end(t);
}
```



#### 借用范围

概念 `std::ranges::borrowed_range` 要求引用的生命周期小于它引用对象的生命周期，避免悬挂引用。例如

```cpp
#include <iostream>
#include <ranges>
#include <vector>

template <std::ranges::borrowed_range R> void f(R &&r)
{
    std::cout<< "concept borrowed_range" << std::endl;
}

int main()
{
    // 报错，右值生命周期立即结束，不能传入
    // f(std::vector<int>{1, 2, 3});
    std::vector v{1, 2, 3};
    f(v);
    return 0;
}
```

这样确保在函数内部使用的引用变量一定有效。



由于 `std::string_view` 对变量模板 `enable_borrowed_range` 进行特化，它满足 `std::ranges::borrowed_range` 的要求

```cpp
f(std::string_view{"hello"});
```



#### 范围大小

概念 `std::ranges::sized_range` 要求可以用 `size` 函数在常数时间内获得大小。



#### 范围视图

概念 `std::ranges::view` 要求能够在常量时间复杂度内进行移动构造、移动赋值和析构。



#### 其它概念

| 概念                                 | 作用                               |
| :--------------------------------- | :------------------------------- |
| `std::ranges::input_range`         | 满足 `input_iterator`              |
| `std::ranges::output_range`        | 满足 `output_iterator`             |
| `std::ranges::forward_range`       | 满足 `forward_iterator` 向前迭代       |
| `std::ranges::bidirectional_range` | 满足 `bidirectional_iterator` 双向迭代 |
| `std::ranges::random_access_range` | 满足 `random_access_iterator` 随机访问 |
| `std::ranges::contiguous_range`    | 满足 `contiguous_iterator` 连续迭代器   |



### 工厂函数

范围库提供一些工厂函数，用于生成范围。



#### 递增范围

生成指定范围递增的范围。例如

```cpp
auto nums = std::views::iota(1, 10);
```



利用延迟计算的特性，可以快速生成满足要求的结果。例如获得 1000 以内的奇平方数

```cpp
#include <iostream>
#include <ranges>

int main()
{
	// iota(1) 产生从 1 开始的无穷序列
    auto res = std::views::iota(1) | std::views::transform([](auto n) { return n * n; }) |
               std::views::filter([](auto n) { return n % 2 == 1; }) |
               std::views::take_while([](auto n) { return n < 1000; });

    for (auto n : res)
        std::cout<< n << " ";
    std::cout<< std::endl;
    return 0;
}
```



#### 空范围

生成无元素的空范围

```cpp
std::ranges::empty_view<int> e;
```



#### 单元素范围

生成具有指定值的单个元素的只读范围

```cpp
auto s = std::views::single(5);
```



#### 连接范围

将嵌套的范围连接起来

```cpp
#include <iostream>
#include <ranges>
#include <string>
#include <vector>

int main()
{
    std::vector<std::vector<char>> nested_ranges = {
        {'H', 'e', 'l', 'l', 'o', ','}, {' '}, {'W', 'o', 'r', 'l', 'd', '!'}};

    auto joined = nested_ranges | std::views::join;

    for (char c : joined)
        std::cout<< c;
    std::cout<< std::endl;
    return 0;
}

```

利用此方法将二维数组扁平化操作

```cpp
#include <iostream>
#include <ranges>

int main()
{
    int arr[5][5]{};
    // 将 arr 转化为一维范围，然后遍历赋值（此时 arr 已经是一维数组）
    for (int n = 0; auto &i : arr | std::views::join)
        i = ++n;

    // 输出 arr（先转换为一维数组）
    for (auto &i : arr | std::views::join)
        std::cout<< i << " ";
    std::cout<< std::endl;

    // 等价于下面代码
    for (int i = 0; i < 5; ++i)
        for (int j = 0; j < 5; ++j)
            std::cout<< arr[i][j] << " ";
    std::cout<< std::endl;
    return 0;
}
```



### 轻量视图

使用 `<span>` 库提供的轻量级视图，可以保存许多连续内存容器，便于作为非引用的容器操作

```cpp
#include <span>
#include <vector>
#include <iostream>

void print_span(std::span<int> s) {
    for (int value : s) {
        std::cout<< value << ' ';
    }
    std::cout<< '\n';
}

int main() {
    int arr[] = {1, 2, 3, 4, 5};
    std::vector<int> vec = {6, 7, 8, 9, 10};

    // 使用 `std::span` 来引用一个数组
    std::span<int> span1(arr);
    print_span(span1); // 输出: 1 2 3 4 5

    // 使用 `std::span` 来引用一个 `std::vector`
    std::span<int> span2(vec);
    print_span(span2); // 输出: 6 7 8 9 10

    // 使用 `std::span` 引用部分数组
    std::span<int> span3(arr + 1, 3); // 引用从 arr[1] 开始的3个元素
    print_span(span3); // 输出: 2 3 4

    return 0;
}
```



## 线程库

在 C++11 以前，Windows 和 Linux 各自有一套多线程框架。而 C++11 提供了 `<thread>` 库统一了多线程框架的接口调用。



### 基本概念

进程是一个应用程序被操作系统拉起加载到内存之后，从开始执行到执行结束的过程。它是程序的一次执行。线程是进程中的一个实体，被系统独立分配和调度的基本单位。

- 进程本身不能获取 CPU 时间，只有它的线程可以
- 一个进程可以拥有多个线程
- 每个线程共享同样的内存空间，开销比较小
- 每个进程拥有独立的内存空间，开销更大

多线程不仅可以实现并行计算，单核处理器也可以利用多线程实现异步操作。



### 多线程

将一个函数创建为线程，然后 `join` 就可以启用线程

```cpp
#include <iostream>
#include <thread>

void hello()
{
    std::cout<< "hello world" << std::endl;
}

int main()
{
    std::thread t(hello);
    
    // 注意不能重复 join
    t.join();
    
    // 判断是否可 join
	if (t.joinable())
        t.join();
    
    // 获得线程 id
    std::cout<< t.get_id() << std::endl;
    
    return 0;
}
```

创建多个线程时，不能确定哪个线程先开始

```cpp
int main()
{
    std::thread t1(hello);
    std::thread t2(hello);
    t1.join();
    t2.join();
    return 0;
}
```



### 参数调用

可以使用 lambda 表达式作为参数，同时提供需要的参数值

```cpp
auto fun = [](int x) {
    while (x-- > 0)
        std::cout<< x << std::endl;
};

std::thread t1(fun, 10);
t1.join();
```

引用参数调用需要显式转换为引用

```cpp
#include <iostream>
#include <thread>

void f(int &x, int y)
{
    x += y;
}

int main()
{
    int v = 10;
    std::thread t(f, std::ref(v), 5);
    t.join();
    std::cout<< "v: " << v << std::endl;
    return 0;
}
```



也可以使用仿函数作为参数调用

```cpp
class Base
{
  public:
    void operator()(int x)
    {
        while (x-- > 0)
            std::cout<< x << std::endl;
    }
};

int main()
{
    std::thread t(Base(), 10);
    t.join();
    return 0;
}
```



非静态成员函数通过函数指针、成员指针和参数值传入线程

```cpp
class Base
{
  public:
    void fun(int x)
    {
        while (x-- > 0)
            std::cout<< x << std::endl;
    }
};

int main()
{
    Base b;
    std::thread t(&Base::fun, &b, 10);
    t.join();
    return 0;
}
```



静态成员函数直接获得函数地址传入参数

```cpp
class Base
{
  public:
    static void fun(int x)
    {
        while (x-- > 0)
            std::cout<< x << std::endl;
    }
};

int main()
{
    std::thread t(&Base::fun, 10);
    t.join();
    return 0;
}
```



### 线程控制

在子线程创建的时候，子线程将会立即开始执行。而 C++ 标准要求线程对象析构时，必须已经执行 `join` 或 `detach`，否则将会发生未定义行为。这两个函数用于显式声明线程的类型，前者表示在子线程结束前，父线程不会结束；后者表示父线程可以不考虑子线程是否完成。




#### join

使用 `join` 添加的线程附加在父线程中，此时父线程会继续执行，如果执行完毕后，还有子线程未结束，将会阻塞直到子线程退出

```cpp
#include <chrono>
#include <iostream>
#include <thread>

void run(int count)
{
    while (count-- > 0)
        std::cout<< cout<< std::endl;
        
    // 暂停线程
    std::this_thread::sleep_for(std::chrono::seconds(3));
}

int main()
{
    std::thread t1(run, 10);
    std::cout<< "main()" << std::endl;
    t1.join();
    if (t1.joinable())
        t1.join();
    std::cout<< "main() after" << std::endl;
    return 0;
}
```



#### detach

使用 `detach` 添加的线程从父线程中分离，此时父线程不等待子线程执行完毕

```cpp
#include <chrono>
#include <iostream>
#include <thread>

void run(int count)
{
    while (count-- > 0)
        std::cout<< cout<< std::endl;
        
    // 暂停线程
    std::this_thread::sleep_for(std::chrono::seconds(3));
}

int main()
{
    std::thread t1(run, 10);
    std::cout<< "main()" << std::endl;
    t1.detach();
    if (t1.joinable())
        t1.detach();
    std::cout<< "main() after" << std::endl;
    return 0;
}
```



使用 detach 可能会产生悬空引用。例如

```cpp
void f(int& i)
{
    i++;
}

void call()
{
    int n = 10;
    std::thread t(f, std::ref(n));
    t.detach();

    // 由于 detach 不会等待线程结束，则 call 完成后 n 立即销毁
}
```

一旦离开 `call` 的范围，就会造成 `f` 中的引用悬空。



#### joinable

使用 joinable 判断一个 `std::thread` 是否为空，当调用 `join/detach` 后，其中的线程会被清空。



### 线程实例

在创建多线程时，首先要获得硬件支持的最大线程数

```cpp
std::thread::hardware_concurrency();
```

为了避免过多的线程开销，应当让每个线程尽可能处理更多的元素

```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <numeric>
#include <thread>
#include <vector>

// 使得每个线程具有最小数目的元素以避免过多的线程开销
template <typename Iterator, typename T> struct accumulate_block
{
    void operator()(Iterator first, Iterator last, T &result)
    {
        // 累积迭代范围的值
        result = std::accumulate(first, last, result);
    }
};

template <typename Iterator, typename T> T parallel_accumlate(Iterator first, Iterator last, T init)
{
    // 获得迭代器之间的距离
    unsigned long const length = std::distance(first, last);

    if (!length)
        return init;

    // 获得每个线程最少处理的元素数
    unsigned long const min_per_thread = 25;
    unsigned long const max_threads = (length + min_per_thread - 1) / min_per_thread;
    std::cout<< max_threads << std::endl;

    // 获得硬件支持的并发数
    unsigned long const hardware_threads = std::thread::hardware_concurrency();
    std::cout<< hardware_threads << std::endl;

    // 计算线程数和每个线程处理的块的大小
    unsigned long const num_threads = std::min(hardware_threads != 0 ? hardware_threads : 2, max_threads);
    std::cout<< num_threads << std::endl;
    unsigned long const block_size= length / num_threads;
    std::cout<< block_size<< std::endl;

    std::vector<T> results(num_threads);
    std::vector<std::thread> threads(num_threads - 1);

    Iterator block_start = first;
    for (unsigned long i = 0; i < (num_threads - 1); ++i)
    {
        // 对迭代器做位移，得到终止位置
        Iterator block_end = block_start;
        std::advance(block_end, block_size);

        // 创建线程，传入数组元素引用
        threads[i] = std::thread(accumulate_block<Iterator, T>(), block_start, block_end, std::ref(results[i]));
        block_start = block_end;
    }

    // 对迭代范围的线程调用成员函数 join
    accumulate_block<Iterator, T>()(block_start, last, results[num_threads - 1]);
    std::for_each(threads.begin(), threads.end(), std::mem_fn(&std::thread::join));

    // 累加线程的计算结果
    return std::accumulate(results.begin(), results.end(), init);
}

int main()
{
    std::vector<int> v{3, 4, 5, 6};
    int res = 0;
    std::cout<< parallel_accumlate(v.begin(), v.end(), res) << std::endl;
    return 0;
}
```



### 互斥量

在 C++11 中 `<mutex>` 是最基本的互斥量，用于锁定和解锁线程。



#### lock

锁定当前线程来确保当前线程操作的变量被完全锁定，不被其它线程影响

```cpp
#include <iostream>
#include <mutex>
#include <thread>

// 全局变量
int sum = 0;

// 线程锁
std::mutex m;

void *countgold()
{
    int i;
    for (i = 0; i < 10000000; i++)
    {
        m.lock();
        sum += 1;
        m.unlock();
    }
    return NULL;
}

int main()
{
    std::thread t1(countgold);
    std::thread t2(countgold);

    // 等待线程执行完毕
    t1.join();
    t2.join();

    std::cout<< "sum = " << sum << std::endl;
    return 0;
}
```



#### lock_guard

在实际编写代码时，最好不直接调用成员函数，因为这需要在出口位置调用 `unlock()` 并进行异常处理。C++11 提供了 RAII 语法的模板类，确保了异常安全。

```cpp
#include <iostream>
#include <mutex>
#include <thread>

int v = 1;

void critical_section(int change_v)
{
    // 在作用域开头定义互斥
    static std::mutex mtx;
    std::lock_guard<std::mutex> lock(mtx);

    // 执行竞争操作
    v = change_v;

    // 离开此作用域后 mtx 会被释放
}

int main()
{
    std::thread t1(critical_section, 2), t2(critical_section, 3);
    t1.join();
    t2.join();

    std::cout<< v << std::endl;
    return 0;
}
```



#### unique_lock

相对于 `std::lock_guard`，新增的 `std::unique_lock` 更加灵活，允许显式地调用 `lock, unlock`，可以缩小锁的作用范围，提供更高的并发度。

```cpp
#include <iostream>
#include <mutex>
#include <thread>

int v = 1;

void critical_section(int change_v)
{
    static std::mutex mtx;
    std::unique_lock<std::mutex> lock(mtx);
    
    // 执行竞争操作
    v = change_v;
    std::cout<< v << std::endl;
    
    // 解锁
    lock.unlock();

    // 在此期间，任何人都可以抢夺 v 的持有权

    // 开始另一组竞争操作，再次加锁
    lock.lock();
    v += 1;
    std::cout<< v << std::endl;
}

int main()
{
    std::thread t1(critical_section, 2), t2(critical_section, 3);
    t1.join();
    t2.join();
    return 0;
}
```



#### RAII

RAII（Resource Acquisition Is Initialization）是一种编程管理，通过对象的生命周期来管理资源的生命周期，从而避免资源泄露和保证资源的正确释放。借助这种思想实现等待线程完结

```cpp
#include <iostream>
#include <thread>

class thread_guard
{
    std::thread &t;

  public:
    explicit thread_guard(std::thread &t_) : t(t_)
    {
    }

    ~thread_guard()
    {
        if (t.joinable())
            t.join();
    }

    thread_guard(const thread_guard &) = delete;
    thread_guard &operator=(const thread_guard &) = delete;
};

void f(int& i)
{
    i++;
}

void call()
{
    int n = 10;
    std::thread t(f, std::ref(n));
    thread_guard g(t);
}

int main()
{
    call();
    return 0;
}
```

当 `call` 调用完成后，`g` 将开始析构，此时线程被 `join`，从而需要等待线程结束才能完成析构。



#### adopt_lock

当一个互斥量被锁定时，它不能够再次锁定。例如

```cpp
std::mutex mtx;

void f()
{
    mtx.lock();

    // 运行报错
    std::lock_guard<std::mutex> lc(mtx);
}
```

因此无法使用 RAII 管理锁。此时可以使用 `std::adopt_lock` 占位符

```cpp
std::mutex mtx;

void f()
{
    mtx.lock();
    std::lock_guard<std::mutex> lc(mtx, std::adopt_lock);
}
```

事实上 `std::lock_guard` 通过重载构造函数来判断是否需要锁定，当存在 `std::adopt_lock` 占位符时，就会进入不锁定的构造函数。



#### call_once

自 C++11 起在 `<mutex>` 中定义了 `std::call_once`，它在多线程中调用时也只准确执行一次可调用对象。

```cpp
template <class Callable, class.. Args>
void call_once(std::once_flag& flag, Callable&& f, Args&&... args);
```

使用时传入调用标记

```cpp
#include <iostream>
#include <mutex>
#include <thread>

int n = 10;
std::mutex mtx;
std::once_flag once_flag;

void f()
{
    std::lock_guard<std::mutex> lock(mtx);
    n++;
    std::cout<< "n = " << n << std::endl;
}

int main()
{
	// 只会调用一次
    std::call_once(once_flag, f);
    std::call_once(once_flag, f);
    std::call_once(once_flag, f);
    std::call_once(once_flag, f);
    std::call_once(once_flag, f);
    return 0;
}
```

如果调用时出现异常，则不会翻转标记，因此可能调用多次。例如

```cpp
#include <iostream>
#include <mutex>
#include <thread>

int n = 10;
std::mutex mtx;
std::once_flag once_flag;

void f()
{
    std::lock_guard<std::mutex> lock(mtx);
    n++;
    std::cout<< "n = " << n << std::endl;
    throw std::runtime_error("error");
}

int main()
{
    try
    {
        std::call_once(once_flag, f);
    }
    catch (const std::exception &e)
    {
        std::cerr << e.what() << '\n';
    }

    try
    {
        std::call_once(once_flag, f);
    }
    catch (const std::exception &e)
    {
        std::cerr << e.what() << '\n';
    }

    return 0;
}
```



#### shared_mutex

读写锁 `shared_mutex` 用于解决读写问题，它同时只能有一个写者或多个读者，性能要比普通锁更好。例如

```cpp
#include <iostream>
#include <shared_mutex>
#include <thread>
#include <vector>

std::shared_mutex mutex;
int value = 0;

void reader()
{
    for (int i = 0; i < 5; ++i)
    {
        // 使用这个局部作用域，当线程休眠时，其他线程可以获得锁
        {
        	// 在这里其它 reader 依然可以访问锁
            std::shared_lock<std::shared_mutex> lock(mutex);
            std::cout<< "Reading value: " << value << "\n";
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}

void writer()
{
    for (int i = 0; i < 5; ++i)
    {
        // 使用这个局部作用域，当线程休眠时，其他线程可以获得锁
        {
            std::unique_lock<std::shared_mutex> lock(mutex);
            value = i;
            std::cout<< "Writing value: " << value << "\n";
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(150));
    }
}

int main()
{
    std::vector<std::thread> threads;
    for (int i = 0; i < 3; ++i)
        threads.emplace_back(reader);
    threads.emplace_back(writer);

    for (auto &thread : threads)
        thread.join();
    return 0;
}
```



#### condition_variable

使用 `std::condition_variable` 条件变量来唤醒线程。它等待一个互斥量，直到被唤醒。格式为

```cpp
std::condition_variable cv;

std::unique_lock<std::mutex> lock(mtx);
cv.wait(lock);
```

此时程序将一直阻塞在 wait 处，直到有通知信号

```cpp
cv.notify_one();	// 通知一个阻塞的 wait
cv.notify_all();	// 通知所有阻塞的 wait
```

可以传入一个 lambda 表达式，如果返回值为 true 则 `wait` 直接返回；否则将解锁互斥量，同时阻塞到本行，直到有通知信号。例如

```cpp
#include <iostream>
#include <shared_mutex>
#include <thread>
#include <vector>

std::mutex mutex;
std::condition_variable cv;
bool ready = false;

void print(int id)
{
    std::unique_lock<std::mutex> lock(mutex);
    
    // 等待 ready 信号为 true
    cv.wait(lock, [] { return ready; });
    std::cout<< "Thread " << id << " prints " << std::endl;
}

void go()
{
    std::lock_guard<std::mutex> lock(mutex);
    ready = true;
    cv.notify_all();
}

int main()
{
    std::vector<std::thread> threads;
    for (int i = 0; i < 10; ++i)
        threads.emplace_back(print, i);
    std::cout<< "10 Threads ready to race..." << std::endl;

    // go 执行后，ready 信号会通知所有线程开始打印
    go();

    for (auto &t : threads)
        t.join();
    return 0;
}
```

虽然每个线程输出的 id 不同，但是它们一定在 `10 Threads ready to race...` 之后出现。



例如下面这个生产消费模式

```cpp
#include <chrono>
#include <condition_variable>
#include <iostream>
#include <mutex>
#include <queue>
#include <thread>

int main()
{
    // 保存生产
    std::queue<int> produced_nums;
    std::mutex mtx;

    // 条件变量
    std::condition_variable cv;
    bool notified = false; // 通知信号

    // 生产者
    auto producer = [&]() {
        // 每 0.9 秒生产一次
        for (int i = 0;; i++)
        {
            std::this_thread::sleep_for(std::chrono::milliseconds(900));

            // 锁定线程，produced_nums 只有当前线程可以修改
            std::unique_lock<std::mutex> lock(mtx);
            std::cout<< "producing " << i << std::endl;
            produced_nums.push(i);
            notified = true;

            // 唤醒所有线程
            // 此处也可以使用 notify_one
            cv.notify_all();
        }
    };

    // 消费者
    auto consumer = [&]() {
        while (true)
        {
            std::unique_lock<std::mutex> lock(mtx);

            // 避免虚假唤醒，只要没唤醒就一直等待
            // while (!notified)
            //     cv.wait(lock);
            // 两种写法等价
            cv.wait(lock, [&] { return notified; });

            // 短暂取消锁，使得生产者有机会在消费者消费空前继续生产
            lock.unlock();
            // 现在生产者进行生产

            // 消费者慢于生产者，每 1 秒消费一次
            std::this_thread::sleep_for(std::chrono::milliseconds(1000));

            // 锁定线程，produced_nums 只有当前线程可以修改
            lock.lock();
            while (!produced_nums.empty())
            {
                std::cout<< "consuming " << produced_nums.front() << std::endl;
                produced_nums.pop();
            }

            // 取消唤醒
            notified = false;
        }
    };

    // 分别在不同的线程中运行，一个生产者，两个消费者
    std::thread p(producer);
    std::thread cs[2];
    for (int i = 0; i < 2; ++i)
        cs[i] = std::thread(consumer);

    p.join();
    for (int i = 0; i < 2; ++i)
        cs[i].join();
    return 0;
}
```



### 线程死锁

#### 多线程死锁

如果两个线程锁定的顺序恰好相反，就有可能出现线程死锁：

1. `mtx1` 被线程 1 锁定
2. `mtx2` 被线程 2 锁定
3. 线程 1 访问 `mtx2` 失败，被阻塞
4. 线程 2 访问 `mtx1` 失败，被阻塞

至此两个线程都被阻塞，这就是死锁。

```cpp
void test() {
  std::mutex mtx1;
  std::mutex mtx2;

  std::thread t1([&] {
    for (int i = 0; i < 1000; i++) {
      mtx1.lock();
      mtx2.lock();
      mtx2.unlock();
      mtx1.unlock();
    }
  });

  std::thread t2([&] {
    for (int i = 0; i < 1000; i++) {
      mtx2.lock();
      mtx1.lock();
      mtx1.unlock();
      mtx2.unlock();
    }
  });
}
```

要避免死锁问题

- 避免一个线程同时持有多个锁
- 如果确实需要多个锁，则保证多个线程上锁的顺序一致

或者可以使用 `std::lock` 上锁

```cpp
std::thread t1([&] {
    for (int i = 0; i < 1000; i++) {
      std::lock(mtx1, mtx2);
      mtx1.lock();
      mtx2.lock();
      mtx2.unlock();
      mtx1.unlock();
    }
});
```



#### 同一线程死锁

如果一个线程重复调用 `lock()` 就会造成死锁。例如

```cpp
std::mutex mtx;

void other() {
 mtx.lock();
 // do something
 mtx.unlock();
}

void func() {
  mtx.lock();
  // other 中又锁定一次
  other();
  mtx.unlock();
}
```

应该不要在 `other()` 中上锁，或者改为使用

```cpp
std::recursive_mutex mtx;
```

它会自动判断是否是同一个线程多次上锁，如果是则会增加计数；之后解锁则会减小计数，计数为 0 时解锁。



### 异步
#### async

使用 `std::async` 创建一个期物实现异步操作

```cpp
#include <iostream>
#include <thread>
#include <future>

constexpr int f(int n)
{
    int t = 0;
    for (int i = 0; i < n; ++i)
        t += i;
    return t;
}

int main()
{
    std::future<int> result = std::async(f, 100);
    std::cout<< "Result: " << result.get() << std::endl;
    return 0;
}
```

异步 `std::async` 与线程 `std::thread` 最大的区别是

* `std::thread` 一定会创建线程，如果资源不够则会报错
* `std::async` 不一定创建线程，不会产生错误

可以指定调用方式

```cpp
std::async(std::launch::defferd, f, 100);						// 延迟调用，不会创建线程，而是延迟到 get() 或 wait() 调用时执行
std::async(std::launch::async, f, 100);							// 异步调用，必须创建线程（默认）
std::async(std::launch::async | std::launch::async, f, 100);	// 由系统决定延迟或异步
```

> [!note]
>
> 使用 `std::launch::deffered` 创建延迟调用的函数，类似于函数式异步，可以实现惰性求值。



#### future

期物 `std::future` 提供了访问异步操作结果的途径。在 C++11 之前，假设线程 A 创建一个子线程 B，并希望获得 B 执行后的结果，就需要提前创建一个事件。当 B 执行完成后 `SetEvent` 发送事件，A 在需要结果的位置 `WaitForSingleObject` 等待结果。而 `std::future` 简化了这一流程，例如

```cpp
#include <future>
#include <iostream>
#include <thread>

int main()
{
    // 创建异步任务
    std::future<int> future = std::async(std::launch::async, []() {
        std::this_thread::sleep_for(std::chrono::seconds(3));
        return 42;
    });

    std::cout<< "Waiting for result..." << std::endl;
    std::future_status status;
    do
    {
        // 等待异步任务完成或超时
        status = future.wait_for(std::chrono::seconds(1));
        
        if (status == std::future_status::timeout)
            std::cout<< "Still waiting..." << std::endl;
        else if (status == std::future_status::ready)
            std::cout<< "Result: " << future.get() << std::endl;
        else if (status == std::future_status::deferred)
            std::cout<< "Deferred" << std::endl;
            
    } while (status != std::future_status::ready);
    std::cout<< "Done." << std::endl;
    return 0;
}
```

由于 `std::future` 删除了拷贝构造和赋值，因此不能共享。如果有共享需求，使用 `std::shared_future` 。



#### packaged_task

使用 `std::packaged_task` 封装要执行的函数，用于获得对应的期物，再用此期物创建一个线程，阻塞直到期物完成。例如

```cpp
#include <future>
#include <iostream>
#include <thread>

int main()
{
    // 将一个返回值为 7 的 lambda 表达式封装到 task 中
    // std::packaged_task 的模板参数为要封装函数的类型
    std::packaged_task<int()> task([]() { return 7; });

    // 获得 task 的期物
    std::future<int> result = task.get_future(); // 在一个线程中执行 task
    std::thread(std::move(task)).detach();
    std::cout<< "waiting...";

    // 在此设置屏障，阻塞到期物的完成
    result.wait(); 

    // 输出执行结果
    std::cout<< "done!" << std::endl << "future result is " << result.get() << std::endl;
    return 0;
}
```



#### promise

类模板 `std::promise` 提供储存值或异常的设施，之后由 `std::future` 对象异步获得结果。

> `std::promise` 只应当使用一次。

例如

```cpp
#include <future>
#include <iostream>
#include <thread>

void test(std::promise<int> &prom, int n)
{
    int t = 0;
    for (int i = 0; i < n; i++)
        t += i;
    prom.set_value(t);
}

int main()
{
    std::promise<int> prom;
    std::thread th(test, std::ref(prom), 100);
    th.join();
    std::future<int> fut = prom.get_future();
    std::cout<< "Result: " << fut.get() << std::endl;
    return 0;
}
```

通过 `promise` 传递结果

```cpp
#include <future>
#include <iostream>
#include <thread>
#include <numeric>
#include <vector>

void accumulate(std::vector<int>::iterator begin, std::vector<int>::iterator end, std::promise<int> prom)
{
    int sum = std::accumulate(begin, end, 0);
    prom.set_value(sum);
}

int main()
{
    std::vector<int> v = {1, 2, 3, 4, 5};
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();

    // 通过 promise 保存 accumulate 的结果
    std::thread t(accumulate, v.begin(), v.end(), std::move(prom));

    std::cout<< "result = " << fut.get() << std::endl;
    t.join();
    return 0;
}
```

由于 `promise` 没有拷贝构造，因此使用移动构造传入。



对于空类型的 `promise`，它可以起到恢复线程的作用，主要用于发送信号。例如

```cpp
#include <future>
#include <iostream>
#include <thread>

int main()
{
    std::promise<void> prom;
    std::future<void> fut = prom.get_future();
    std::thread th([&prom]() {
        std::cout<< "Hello from thread" << std::endl;
    });
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout<< "Main" << std::endl;
    th.join();
    return 0;
}
```

由于主线程等待 1 秒，因此输出

```shell
Main
Hello from thread
```

而如果阻塞线程，就需要通过 `set_value` 来解锁

```cpp
#include <future>
#include <iostream>
#include <thread>

int main()
{
    std::promise<void> prom;
    std::future<void> fut = prom.get_future();
    std::thread th([&]() {
        // 子线程阻塞
        fut.wait();
        std::cout<< "Hello from thread" << std::endl;
    });
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout<< "Main" << std::endl;
    th.join();

    // 如果这里没有 set_value，则 fut.wait() 会一直阻塞
    prom.set_value();
    return 0;
}
```

输出结果为

```shell
Hello from thread
Main
```



### 同步

#### latch

在 C++20 提供 `std::latch` 用于一次性同步多个线程。例如

```cpp
#include <barrier>
#include <iostream>
#include <latch>
#include <thread>

using namespace std::chrono_literals;

std::latch work_start(3);

void work() {
    std::cout<< "Working..." << std::endl;
    work_start.count_down();  // 减少计数器
}

int main() {
    for(int i = 0; i < 3; ++i) std::jthread thread{work};
    std::this_thread::sleep_for(3s);
    work_start.wait();  // 等待计数器为 0
    return 0;
}
```

初始构造参数为等待线程数，当计数器归零，则不再等待。可以在线程达到 `latch` 同时阻塞线程

```cpp
#include <barrier>
#include <iostream>
#include <latch>
#include <thread>
#include <vector>

using namespace std::chrono_literals;

std::latch work_start(3);

void work() {
    std::cout<< "Working..." << std::endl;
    work_start.arrive_and_wait();  // 减少计数器，并等待其他线程
}

int main() {
    std::vector<std::jthread> threads;
    for(int i = 0; i < 3; ++i) threads.push_back(std::jthread(work));
    for(auto& t: threads) t.detach();
    std::this_thread::sleep_for(3s);
    return 0;
}
```



#### barrier

在 C++20 提供 `std::barrier` 用于周期性同步线程。例如

```cpp
#include <barrier>
#include <iostream>
#include <latch>
#include <thread>
#include <vector>
#include <atomic>

using namespace std::chrono_literals;

std::barrier work_start(3);
std::atomic_int counter(0);

void work() {
    std::cout<< "Working..." << std::endl;
    counter++;
    std::this_thread::sleep_for(1s);
    work_start.arrive_and_wait();  // 减少计数器，并等待其他线程
}

int main() {
    for(int k = 0; k < 5; k++) {
        std::vector<std::jthread> threads;
        for(int i = 0; i < 3; ++i) threads.push_back(std::jthread(work));
        for(auto& t: threads) t.join();
        std::cout<< counter << std::endl;
    }

    return 0;
}
```

还可以使用 `arrive_and_drop` 不会阻塞线程，还会减少 `barrier` 控制的线程数量。



### 原子操作

#### 原子性

多个线程对同一个数据进行原子操作，会产生结果丢失。例如，当线程 A 执行 `g_value++` 时，如果线程切换时间正好是在线程 A 将值保存到 `g_value` 之前，线程 B 继续执行 `g_vlaue++`，那么当线程 A 再次被切换回来之后，会将原来线程 A 保存的值保存到 `g_value` 上，线程 B 进行的加法操作将被覆盖。因此需要使用原子锁函数解决这一问题。



观察下面的代码。注意到 `a, flag` 被两个并行线程访问，两个线程的执行先后顺序未知，因此程序的行为未定义

```cpp
#include <iostream>
#include <thread>

int main()
{
    int a = 0;
    int flag = 0;

    std::thread t1([&]() {
        while (flag != 1)
            ;

        // 访问 a，这一步不知道什么时候执行
        int b = a;
        std::cout<< "b = " << b << std::endl;
    });

    std::thread t2([&]() {
        // 这两步都不知道什么时候执行
        a = 5;
        flag = 1;
    });

    t1.join();
    t2.join();
    return 0;
}
```



虽然互斥可以解决这一问题，但是它是一个高昂的操作系统级功能。它通常包含两条基本原理：

1. 提供线程间自动的状态转换，即『锁住』这个状态
2. 保障在互斥锁操作期间，所操作变量的内存与临界区外进行隔离

这是一组非常强的同步条件，换句话说当最终编译为 CPU 指令时会表现为非常多的指令。这对于一个仅需原子级操作（没有中间态）的变量太苛刻了。



为此 C++11 提供了原子类型模板 `std::atomic` 用于定义原子操作。例如

```cpp
#include <atomic>
#include <iostream>
#include <thread>

// 原子 int 类型
std::atomic<int> cout= {0};

int main()
{
    // 原子加法
    std::thread t1([]() { count.fetch_add(1); });
    std::thread t2([]() {
        count++;    // 等价于 fetch_add
        cout+= 1;   // 等价于 fetch_add
    });

    t1.join();
    t2.join();
    
    std::cout<< cout<< std::endl;
    return 0;
}
```



并非所有类型都具有原子操作，因为原子操作的可行性取决于 CPU 架构。这就需要检查类型对原子操作的支持度

```cpp
#include <atomic>
#include <iostream>

struct A
{
    float x;
    int y;
    long long z;
};

int main()
{
    std::atomic<A> a;
    std::cout<< std::boolalpha << a.is_lock_free() << std::endl;
    return 0;
}
```



#### 成员函数

使用原子变量的成员函数

- `fetch_add` 返回旧值，增加指定值
- `exchange` 将新值写入，返回旧值
- `compare_exchange_strong` 比较旧值与新值，如果不相等则将原子变量写入新值；相等则将新值写入原子变量。返回是否相等
- `store` 写入值
- `load` 加载值



#### 多步操作

在使用原子变量时，需要警惕出现多次的原子操作，因为在这些原子操作之间仍然可能出现其它线程的干扰。例如

```cpp
std::atomic<int> x;

x += 1;     // 原子
x = x + 1;  // 非原子

x.fetch_add(1);         // 原子
x.store(x.load() + 1);  // 非原子
```

因此应当尽可能将原子操作合并使用。



### 一致性模型

如果将一个变量 v 在多个线程之间的操作设为原子操作，即

> 任何一个线程在操作 v 完成后，其它线程均能**同步**感知到 v 的变化。

这意味着 v 在多线程中表现为顺序执行，没有任何效率上的收益。为了提高效益，就需要**削弱原子操作的同步条件**。



通常会考虑 4 种一致性模型

1. 线程一致性：强一致性或原子一致性。这要求任何一次读操作都能获得最新的写入数据，并且所有操作在全局时间上顺序执行。

    ```cpp
    // T1 依次存入 1 3，时间顺序存入 1 3
    // T2 依次存入 2 4，时间顺序存入 2 4
    // 全局顺序 1 2 3 4
    x.store(1);	// T1 (1)
    x.store(2);	// T2 (1)
    x.store(3);	// T1 (2)
    x.store(4);	// T2 (2)
    x.read();
    ```

    上述操作中，每一步都是严格按顺序执行。

2. 顺序一致性：要求任何一次读操作都能获得最新的写入数据，但是不需要顺序执行。

    ```cpp
    // T1 依次存入 1 3，时间顺序存入 1 3
    // T2 依次存入 2 4，时间顺序存入 2 4
    // 全局顺序 2 1 3 4
    x.store(2);	// T2 (1)
    x.store(1);	// T1 (1)
    x.store(3);	// T1 (2)
    x.store(4);	// T2 (2)
    x.read();
    ```

    依然能获得最新的写入数据，尽管不再按顺序执行。

3. 因果一致性：只需要保证具有因果关系的操作顺序。

    ```cpp
    // T1 a = 1, b = 2, x = 2,
    // T2 x = 1, c = a + b, x= 3
    a = 1;		// T1 (1)
    x = 1;		// T2 (1)
    b = 2;		// T1 (2)
    c = a + b;	// T2 (2)
    x = 3;		// T2 (3)
    x = 2;		// T1 (3)
    ```

    由于 c 依赖 a, b，因此只需要保证 a, b 的赋值在 c 计算之前。

4. 最终一致性：只保证一个操作在未来不知道某个时间节点上可以被观测到。

    ```cpp
    x.store(0);
    // T1 x = 3, x = 4
    // T2 read, read, read, read
    x.store(3);	// T1 (1)
    x.read();	// T2 (1)
    x.read();	// T2 (2)
    x.read();	// T2 (3)
    x.read();	// T2 (4)
    x.store(4);	// T1 (2)
    ```

    上述操作中，T1 写入两次，T2 读取四次，但是只保证 `x = 4` 这个结果在未来某个时间点可以被获得，有可能 T2 读取四次都没有获得这个结果。

上面介绍的 4 种要求依次减弱。

> [!note]
> 在 `x86` 平台上只有最强的内存序。



### 内存顺序

为了追求性能，实现不同强度的一致性，C++11 提供 `std::memory_order` 的不同选项对应这些模型。

> 释放/消费模型中的 `std::memory_order_consume` 由于难以在不同的硬件架构上提供精确高效的实现，从 C++20 开始已被弃用并从 C++ 标准中删除。

默认情况下，采用顺序一致模型 `std::memory_order_seq_cst`，它的同步条件最严格，但是效率最低。



遵从严格的一致性会大幅降低并行性能，因此现代 CPU 普遍不支持严格的顺序一致性。例如

```cpp
k = 3;    // #1
x = a;    // #2 a 没有使用过，在内存中
y = k;    // #3 k 由于刚刚使用过，可能还在寄存器中暂存
```

编译器可能会让 CPU 在等待数据总线返回 a 的值之前，先执行 `y = k` 的赋值，因为 k 在之前的代码刚刚访问过，可能还暂存在某个寄存器中，访问更快速。因此优化后的执行顺序为

```cpp
k = 3;    // #1
y = k;    // #3
x = a;    // #2
```

对于单线程，这不会有影响；但是对于多线程操作，可能会产生问题。



#### 宽松模型

此模型只保证当前操作是原子操作，即不会被其它操作覆盖，但是不保证操作顺序问题。例如下面两个线程

```cpp
#include <atomic>
#include <iostream>
#include <thread>
#include <vector>

int data = 42;
std::atomic_bool data_ready(false);
int disorder = 0;

// 线程 1
void writer_thread() 
{
    data = 10;                                         // #1
    data_ready.store(true, std::memory_order_relaxed); // #2
}

// 线程 2
void reader_thread() 
{
    // #3：对 data_ready 的读操作（循环语句可以保证不被重排）
    while (!data_ready.load(std::memory_order_relaxed))
        ;

    // #4
    if (data != 10)
        disorder++;
}
```

由于**循环的存在，可以保证 #3 一定在 #4 之前**；由于 `std::memory_order_relaxed` 标记，#1, #2 的执行顺序不能保证。因此可能出现两种情况

1. 执行顺序 `#1->#2->#3->#4`，此时 data 先赋值为 10，因此 disorder 自增；
2. 执行顺序 `#2->#3->#4->#1`，此时 data 后赋值，因此 disorder 不变。

这里循环中的 load 操作无论标记为任何值，都能确保在 #4 之前发生。



由于累加不需要考虑顺序，因此使用宽松模型可以提高并行效率。例如

```cpp
std::atomic<int> counter = {0};
std::vector<std::thread> vt;

// fetch_add 的选项设定了模型
for (int i = 0; i < 100; ++i)
    vt.emplace_back([&]() { counter.fetch_add(1, std::memory_order_relaxed); });

for (auto &t : vt)
    t.join();
std::cout<< "current counter:" << counter << std::endl;
```



#### 释放/获取模型

此模型下将限制线程间的操作顺序，在释放操作和获取操作之间规定时间顺序：

* 如果 store 操作被标记为 `std::memory_order_release`，则当前线程此操作**之前**的任何读写操作**不能排在当前 store 的后面**；
* 如果 load 操作被标记为 `std::memory_order_acquire`，则当前线程此操作**之后**的任何读写操作**不能排在当前 load 的前面**；
* 如果 A 线程中的 store 操作被标记为 `std::memory_order_release`，B 线程中相同原子量的 load 操作被标记为 `std::memory_order_acquire`，则 A 线程的这个 store 操作之前的读写操作对 B 线程可见；

我们可以将标记看做一个屏障：

1. 标记 store 为 `std::memory_order_release` 可以确保它前面的读写确实发生在它前面；
2. 标记 load 为 `std::memory_order_acquire` 可以确保它后面的读写确实发生在它后面；

例如下面这个例子

```cpp
std::atomic<std::string *> ptr;
int data;

void producer()
{
    std::string *p = new std::string("Hello"); // #1
    data = 42;                                 // #2
    ptr.store(p, std::memory_order_release);   // #3
}

void consumer()
{
    std::string *p2;

    // #4
    while (!(p2 = ptr.load(std::memory_order_acquire)))
        ;
    
    assert(*p2 == "Hello"); // #5
    assert(data == 42);     // #6
}

int main()
{
    std::thread t1(producer);
    std::thread t2(consumer);
    t1.join();
    t2.join();
}
```

其中 consumer 中的循环语句确保 #4 在 #5, #6 之前发生。而 `std::memory_order_release` 保证 #3 在 #1, #2 之后发生，`std::memory_order_acquire` 确保了 #3 之前的读写操作对 consumer 可见，因此之后的 #5, #6 都可以正常获得 #1, #2 写入操作的值，不会报错。



对于同时具有读写操作的 `compare_exchange_strong` 操作，可能需要添加对应读写的两个标记，这就是 `std::memory_order_acq_rel` 标记：

* 如果 load/store 操作被标记为 `std::memory_order_acq_rel`，则当前线程此操作**前后**的任何读写操作**不能排在当前 load/store 的后面**；
* 如果 A 线程中的 load/store 操作被标记为 `std::memory_order_acq_rel`，B 线程中相同原子量的 load 操作被标记为 `std::memory_order_acquire`，则 A 线程的这个 load/store 操作之前的读写操作对 B 线程可见；
* 如果 A 线程中的 load/store 操作被标记为 `std::memory_order_acq_rel`，B 线程中相同原子量的 store 操作被标记为 `std::memory_order_release`，则 B 线程的这个 store 操作之前的读写操作对 A 线程可见；

因此 `std::memory_order_acq_rel` 实际上就是两种标记的结合。考虑下面的例子

```cpp
std::vector<int> data;
std::atomic<int> flag = {0};

void thread_1()
{
    data.push_back(42);                       // #1
    flag.store(1, std::memory_order_release); // #2
}

void thread_2()
{
    int expected = 1;                                                             // #3
    while (!flag.compare_exchange_strong(expected, 2, std::memory_order_acq_rel)) // #4
        expected = 1;
}

void thread_3()
{
    // #5
    while (flag.load(std::memory_order_acquire) < 2)
        ;
    assert(data.at(0) == 42); // #6
}
```

其中 `compare_exchange_strong` 比较 flag 与 expected 的值：

1. 如果相等，则**以原子方式**将 flag 更新为 2，同时返回 true；
2. 如果不等，则**以原子方式**将 flag 的值保存在 expected 中，同时返回 false；

首先 `std::memory_order_release` 确保 #1 在 #2 之前；循环体确保 #5 在 #6 之前，#3 在 #4 之前；而 `std::memory_order_acq_rel` 使得 #4 能看到 #2 之前的 #1 操作；而 `std::memory_order_acquire` 确保 **#5 能看到 #4 之前的操作，自然也包括 #2 之前的 #1 操作**；因此 #6 操作时 `data[0]` 确保存在。



#### 顺序一致模型

此模型下原子操作满足顺序一致性，进而可能对性能产生损耗。对原子变量的访问使用 `std::memory_order_seq_cst` 标记等价于：

* 当前 load 操作被标记为 `std::memory_order_acquire`；
* 当前 store 操作被标记为 `std::memory_order_release`；
* 当前 load/store 操作被标记为 `std::memory_order_acq_rel`；

对所有线程来说，这个标记会对所有使用此标记的原子变量进行同步，使得线程看到的内存操作的顺序都是一样的，这会导致效率降低。例如

```cpp
std::atomic<int> counter = {0};
std::vector<std::thread> vt;

// fetch_add 的选项设定了模型
for (int i = 0; i < 100; ++i)
    vt.emplace_back([&]() { counter.fetch_add(1, std::memory_order_seq_cst); });

for (auto &t : vt)
    t.join();
std::cout<< "current counter:" << counter << std::endl;
```

它与之前的宽松模型一致，但是由于需要保证顺序，因此效率较低。



#### 插入标记

可以通过 `std::atomic_thread_fence` 直接插入标记，而不必执行 load/store 操作

```cpp
std::atomic_thread_fence(std::memory_order_acquire);
```



## 协程库

### 基本概念

协程是可以暂停和恢复的函数，它与线程没有直接关联。在 C++ 中函数定义包含 `co_await, co_yield, co_return` 关键字之一则为协程。

> [!note]
> 带有协程的函数在执行时会立即调用协程的初始化函数 `initial_suspend()`，并根据返回值决定是否暂停协程。



### 简单实例

协程库 `<coroutine>` 提供协程句柄，用于控制协程。例如

```cpp
#include <coroutine>
#include <future>
#include <iostream>
#include <thread>

struct promise {
  struct promise_type;

  // 协程迭代器
  struct iterator {
    // 协程引用，std::coroutine_handle 本身就代表协程
    std::coroutine_handle<promise_type> &h;

    int &operator*() { return h.promise().n; }

    iterator &operator++() {
      // 如果协程尚未完成，则恢复协程
      if (!h.done())
        h.resume();
      return *this;
    }

    // 检查协程是否完成
    bool operator!=(const iterator &) const { return !h.done(); }
  };

  // 承诺类型，协程的返回类型必须拥有此类型
  struct promise_type {
    int n;

    promise get_return_object() {
      return {std::coroutine_handle<promise_type>::from_promise(*this)};
    }

    std::suspend_never initial_suspend() noexcept { return {}; }
    std::suspend_always final_suspend() noexcept { return {}; }
    std::suspend_always yield_value(int r) noexcept {
      n = r;
      return {};
    }

    void return_void() {}
    void unhandled_exception() {}
  };

  iterator begin() { return {_h}; }
  iterator end() { return {_h}; }

  std::coroutine_handle<promise_type> _h;
};

promise iota(int value) {
  std::cout<< "iota(" << value << ")" << std::endl;
  while (value > 0) {
    co_yield value--;
  }
}

int main() {
  for (int x : iota(10))
    std::cout<< x << " ";
  std::cout<< std::endl;
  return 0;
}
```

其中 `std::coroutine_handle` 本身就代表协程，用来返回 `promise` 实例。



### 语法结构

#### co_await

表达式 `co_await` 用于暂停协程直到恢复。协程开始执行时，会调用 `initial_suspend()`，即 `promise` 中的成员

```cpp
// 协程结构中对应的初始化
std::suspend_never initial_suspend() noexcept { return {}; }
```

同时 `co_await` 它的结果，等价于

```cpp
std::suspend_always res = promise.initial_suspend();
co_await res;
```

表达式 `co_await` 本身等价于

```cpp
co_await promise.yield_value
```



返回类型 `std::suspend_never, std::suspend_always` 结构形如

```cpp
_EXPORT_STD struct suspend_never {
  // 返回 true，表示不暂停协程
  _NODISCARD constexpr bool await_ready() const noexcept { return true; }
  constexpr void await_suspend(std::coroutine_handle<> h) const noexcept {}
  constexpr void await_resume() const noexcept {}
}
```

被 `co_await` 的实例要求具有 `await_ready` 方法

- 如果返回 `true`，则协程将继续；
- 如果返回 `false`，则协程将暂停；

例如给出如下结构

```cpp
struct Input {
  // 这里返回 false，则表示会暂停
  bool await_ready() { return false; }
  void await_suspend(std::coroutine_handle<> h) {}
  void await_resume() {}
}

int main() {
  auto lambda = []() -> promise {
    Input t;
    std::cout<< "Begins" << std::endl;
    
    // 由于 t.await_ready() 返回 false，因此此处暂停
    co_await t;
    
    std::cout<< "Ends" << std::endl;
  };

  // 调用表达式
  promise ret = lambda();
  std::cout<< "Hello, world!" << std::endl;

  // 恢复协程，调用最后一个输出
  ret._h.resume();
  return 0;
}
```

输出结果为

```cpp
Begins
Hello, world!
Ends
```



#### co_yield

表达式 `co_yield` 将创建协程，调用 `initial_suspend()`，并返回暂停的协程。



首先我们将 `promise` 结构改写，利用 `std::coroutine_handle<promise>` 继承构建一个返回类型 `coroutine`，它具有 `promise_type` 类型，因此符合协程要求

```cpp
struct promise;

// promise 作为模板参数
struct coroutine : std::coroutine_handle<promise> {
  // 承诺类型，协程的返回类型必须拥有此类型
  using promise_type = ::promise;
};

struct promise {
  int n;

  // 此处 coroutine 表示协程，它返回 promise 实例
  coroutine get_return_object() { return {coroutine::from_promise(*this)}; }

  std::suspend_never initial_suspend() noexcept { return {}; }
  std::suspend_always final_suspend() noexcept { return {}; }
  std::suspend_always yield_value(int r) noexcept {
    n = r;
    return {};
  }

  void return_void() {}
  void unhandled_exception() {}
};
```

> [!note]
> 这里 `promise` 就是 `promise_type`，只是设置了别名。因此我们实际上省略了之前的 `promise` 结构。

然后我们给出一个满足 `co_await` 要求的结构

```cpp
template <typename T> struct Future {
  T n;

  // 返回 false，则协程暂停
  bool await_ready() { return false; }

  void await_suspend(std::coroutine_handle<> handle) {
    // 创建异步任务
    std::thread th([this, handle]() {
      // t == 5
      int t = n;
      for (int i = 1; i < t; i++)
        n *= i;

      // 任务完成后重新启动协程
      if (!handle.done())
        handle.resume();
    });
    th.detach();
  }

  T await_resume() { return n; }
};
```

现在可以重载 `co_await` 操作符

```cpp
// 重载 co_await 操作符
template <typename T> inline auto operator co_await(const Future<T> &f) {
  // 将 f 中保存的值返回
  std::cout<< "co_await " << f.n << std::endl;
  return f;
}

coroutine f() {
  Future fu{5};
  
  // 在这里协程暂停，并进入 await_suspend 方法，执行 5 * 4*3*2*1
  int v = co_await fu;
  // co_await 返回 f，同时调用 await_resume 返回异步结果 120
  
  std::cout<< "v = " << v << std::endl;
  
  // co_yield 创建一个 promise 实例，并调用 yield_value(int r) 保存值
  // 然后协程将会暂停
  co_yield 20;
}

int main() {
  // ret 获得一个协程句柄
  auto ret = f();
  std::cout<< "main" << std::endl;
  std::this_thread::sleep_for(std::chrono::seconds(1));
  
  // 恢复协程，co_yield 20 执行结束，ret 协程完成，获得 20
  ret.resume();
  
  std::cout<< std::boolalpha << ret.done() << std::endl;
  
  // 获得协程结果
  std::cout<< "co_yield value = " << ret.promise().n << std::endl;
  return 0;
}
```

> [!note]
> 使用 `co_await fu` 暂停协程后，进入 `await_suspend` 执行异步任务后，被 `handle.resume()` 恢复；而 `co_yield 20` 暂停协程后，被 `ret.resume()` 恢复。



对上述例子进行改写：使用 `std::async` 替代 `std::thread, detach()` 这种不确定结束时间的形式。首先添加标准库中的期物

```cpp
struct promise {
  std::future<T> future;
  // ...
};
```

然后通过 `std::async` 创建异步任务

```cpp
template <typename T> struct Future {
  // ...
  void await_suspend(std::coroutine_handle<> handle) {
    // 改为通过 async 创建异步任务
    handle.promise().future = std::async([this, handle]() {
      int t = n;
      for (int i = 1; i < t; i++)
        n *= i;

      // 任务完成后重新启动协程
      if (!handle.done())
        handle.resume();
        return n;
    });
  }
  // ...
};
```

现在在 `main` 函数中直接等待期物结束即可

```cpp
int main() {
  auto ret = f();
  std::cout<< "main" << std::endl;
  
  // 等待期物结束
  ret.promise().future.wait();
  ret.resume();
  
  std::cout<< std::boolalpha << ret.done() << std::endl;
  std::cout<< "co_yield value = " << ret.promise().n << std::endl;
  return 0;
}
```



#### co_return

表达式 `co_return` 用于完成协程执行并返回一个值。

```cpp
#include <coroutine>
#include <future>
#include <iostream>
#include <thread>

struct promise;

struct coroutine : std::coroutine_handle<promise> {
  // 承诺类型，协程的返回类型必须拥有此类型
  using promise_type = ::promise;
};

struct promise {
  int result = 0;

  coroutine get_return_object() { return {coroutine::from_promise(*this)}; }

  std::suspend_never initial_suspend() noexcept { return {}; }
  std::suspend_always final_suspend() noexcept { return {}; }

  // 保存 co_return 的返回值
  void return_value(int v) {
    result = v;
    std::cout<< "return_value " << v << std::endl;
  }
  void unhandled_exception() {}
};

coroutine f() {
  std::cout<< "f" << std::endl;

  // 初始化协程，不会暂停，调用 return_value 后返回
  co_return 20;
}

int main() {
  auto ret = f();
  std::cout<< "main" << std::endl;
  std::cout<< std::boolalpha << ret.done() << std::endl;
  std::cout<< ret.promise().result << std::endl;
  return 0;
}
```

由于 `initial_suspend()` 返回 `std::suspend_never`，因此进入 `f()` 时协程不会暂停；反之，如果改为

```cpp
std::suspend_always initial_suspend() noexcept { return {}; }
```

则 `f()` 将在调用时立即暂停。



#### 悬空引用

协程开始时将所有函数形参复制到协程状态中，按值传递的形参被移动或复制，按引用传递的参数保持为引用。因此如果在被引用对象销毁后恢复协程，将会导致悬空引用。例如

```cpp
#include <coroutine>
#include <future>
#include <iostream>
#include <thread>

struct promise;

struct coroutine : std::coroutine_handle<promise> {
  // 承诺类型，协程的返回类型必须拥有此类型
  using promise_type = ::promise;
};

struct promise {
  int result = 0;

  coroutine get_return_object() { return {coroutine::from_promise(*this)}; }

  // 这里改为使用 std::suspend_always，因此协程初始化将会暂停
  std::suspend_always initial_suspend() noexcept { return {}; }
  std::suspend_always final_suspend() noexcept { return {}; }

  // 保存 co_return 的返回值
  void return_void() {}
  void unhandled_exception() {}
};

void f() {
  // 立即调用，由于 initial_suspend 返回 std::suspend_always，因此协程立即暂停
  coroutine h = [i = 10]() -> coroutine {
    std::cout<< "coroutine " << i << " started" << std::endl;
    co_return;
  }();

  // 现在 lambda 已被销毁
  std::cout<< "Hello, world!" << std::endl;

  std::cout<< std::boolalpha << h.done() << std::endl;

  // 协程恢复后，执行输出语句，然而右值引用捕获 i 已经销毁，因此会导致未定义行为
  h.resume();
  h.destroy();
}

void g() {
  // 立即调用，由于 initial_suspend 返回 std::suspend_always，因此协程立即暂停
  coroutine h = [](int i) -> coroutine {
    std::cout<< "coroutine " << i << " started" << std::endl;
    co_return;
  }(10);

  // 现在 lambda 已被销毁
  std::cout<< "Hello, world!" << std::endl;

  std::cout<< std::boolalpha << h.done() << std::endl;

  // 协程恢复后，执行输出语句，作为值传递的参数被复制到 coroutine 中，因此能够正常运行
  h.resume();
  h.destroy();
}

int main() {
  f();
  g();
  return 0;
}
```



### 协程迭代

被 `co_yield` 暂停的协程可以通过迭代器自增来恢复，从而实现协程迭代。回到简单实例给出的代码，注意到其中的迭代器结构

```cpp
iterator &operator++() {
  // 如果协程尚未完成，则恢复协程
  if (!h.done())
    h.resume();
  return *this;
}

// 解引用会获得保存的值
int &operator*() { return h.promise().n; }
```

利用这个迭代器自增，当执行 `++` 操作时，会恢复协程并返回自身；而解引用会获得保存的值。因此可以执行下面的循环

```cpp
promise iota(int value) {
  std::cout<< "iota(" << value << ")" << std::endl;
  while (value > 0) {
    co_yield value--;
  }
}

int main() {
  // promise 的迭代器不断自增，实际上是在不断恢复协程
  auto coro = iota(10);
  for (auto it = coro.begin(); it!= coro.end(); ++it)
    std::cout<< *it << " ";
  std::cout<< std::endl;
  return 0;
}
```

在 `while` 循环中，每次执行到 `co_yield` 时都会暂停并返回 `promise`，其中保存 `value` 值。而迭代器解引用获得保存的值输出。



利用现代 C++ 的写法，可以简化为

```cpp
for (int x : iota(10))
  std::cout<< x << " ";
std::cout<< std::endl;
```



## 随机数库

### 标准生成器

相较于使用 `RAND_MAX` 取余生成随机数，现代的方法是使用 random 库中的随机数生成器类，包括 `mt19937, default_random_engine` 等；以及常用的分布函数生成器。

```c++
#include <iostream>
#include <random>
#include <time.h>

int main()
{
    std::mt19937 generator((unsigned int)time(nullptr));    // 使用 time(nullptr) 作为随机数生成器的种子
    std::uniform_real_distribution<double> uniform(0, 1.0); // 在 [0,1] 区间内的均匀分布

    int N = 100;
    for (int i = 0; i < N; i++)
    {
        cout<< uniform(generator) << endl;
    }

    std::normal_distribution<double> normal(0.0, 1.0); 	  // 期望为 0.0，标准差为 1.0 的正态分布
    for (int i = 0; i < N; i++)
    {
        cout<< normal(generator) << endl;
    }

    return 0;
}
```

需要注意的是，如果使用类封装生成器，并在函数中调用，则应当使用静态本地变量来定义生成类

```c++
// 封装了随机数生成器
class Random {...};

// 产生随机数
double rand()
{
    // 静态定义，确保每次调用使用同一个生成器
    static Random gen;
    double r = gen.random();
    return r;
}
```

之所以这样是为了防止每次使用的生成器的种子不同，导致产生的不是想要的随机数。



#### 封装随机数

随机数生成算法

```cpp
#pragma once

#include <cmath>
#include <functional>

using Func = std::function<double(double)>;

class Random
{
public:
    Random(int seed, int N = 48, int p = 19) : seed_(seed), xn_(1)
    {
        N_ = 1 << N;
        a_ = 1;
        for (int i = 0; i < p; i++)
            a_ *= 5;
    }

    // [0,1] 上的均匀分布
    double uniform()
    {
        xn_ = (a_ * xn_) % N_;
        return 1.0 * xn_ / N_;
    }

    // 根据逆分布函数产生随机数
    double rand(Func f) { return f(uniform()); }

private:
    const int seed_;
    long long N_, a_;
    int xn_;
};
```



#### 随机种子

对于相同的种子，生成器 `std::mt19937` 总会得到相同的结果。即使使用时间戳，得到的随机种子以秒为精度，因此不够随机。更好的方法是使用

```cpp
std::random_device rd;
std::mt19937 gen(rd());
```

其中 `std::random_device` 从硬件层面获得随机数，足够随机，但效率较低，因此一般只用于生成随机种子。



#### 多线程

为了应对多个线程同时访问的情况，标准库使用的随机生成器中的随机种子是 `thread_local` 类型，类似于

```cpp
#include <random>
#include <thread>

int rand() {
    static thread_local int seed;
    // 生成随机数
    return seed;
}

int main() {
    std::thread t1([](){
        rand();
    });
    std::thread t2([](){
        rand();
    });
    return 0;
}
```

这会导致多个线程可能生成相同的随机序列。解决方案是使用多个不同的生成器

```cpp
std::mt19937 gen1(rd());
std::mt19937 gen2(rd());
```



#### 随机分布

标准库提供了多种随机分布

| 分布                          | 含义     |
| :-------------------------- | :----- |
| `uniform_int_distribution`  | 均匀整数分布 |
| `uniform_real_distribution` | 均匀实数分布 |
| `gamma_distribution`        | 伽马分布   |
| `exponential_distribution`  | 指数分布   |
| `binomial_distribution`     | 二项分布   |
| `poisson_distribution`      | 泊松分布   |
| `normal_distribution`       | 正态分布   |
| `cauchy_distribution`       | 柯西分布   |



### 标准库生成

#### 随机拾取

利用随机分布，可以便捷地拾取容器中的值。例如

```cpp
std::random_device rd;
xorshift32_state rng{rd()};
std::vector<std::string> choices = {"apple", "banana", "orange", "grape", "pear"};

// 随机生成 size_t 作为索引
std::uniform_int_distribution<size_t> uni(0, choices.size() - 1);
printf("Random choice: %s\n", choices[uni(rng)].c_str());
printf("Random choice: %s\n", choices[uni(rng)].c_str());
printf("Random choice: %s\n", choices[uni(rng)].c_str());
printf("Random choice: %s\n", choices[uni(rng)].c_str());
```

注意生成范围不包括最大值。



#### 容器填充

使用算法库中的 `std::generate` 配合随机生成算法填充容器

```cpp
std::random_device rd;

std::vector<int> v(100);
std::generate(v.begin(), v.end(), [rng = xorshift32_state{std::random_device{}()}, uni = std::uniform_int_distribution<int>(0, 100)]() mutable { return uni(rng); });

for(int i: v) {
    printf("%d ", i);
}
printf("\n");
```

类似地使用 `std::generate_n` 用于向容器推入元素

```cpp
std::random_device rd;

std::vector<int> v;
v.reserve(100);
std::generate_n(std::back_inserter(v), 100, [rng = xorshift32_state{std::random_device{}()}, uni = std::uniform_int_distribution<int>(0, 100)]() mutable { return uni(rng); });

for(int i: v) {
    printf("%d ", i);
}
printf("\n");
```



#### 随机洗牌

使用 `std::shuffle` 算法对容器范围随机洗牌，需要提供一个随机生成器

```cpp
std::shuffle(v.begin(), v.end(), std::mt19937{rd()});
```



### 随机算法

#### Xorshift

使用 `std::mt19937` 并不高效，可以使用其它随机生成算法。例如 Xorshift 算法

```embed
title: "Xorshift - Wikipedia"
image: "https://upload.wikimedia.org/wikipedia/commons/e/ee/Xorshift.png"
description: "From Wikipedia, the free encyclopedia"
url: "https://en.wikipedia.org/wiki/Xorshift"
```

从维基百科上获得算法代码，进行封装后，作为生成器

```cpp
#include <random>
#include <thread>

struct xorshift32_state {
    uint32_t a;

    /* The state must be initialized to non-zero */
    uint32_t operator()() {
        /* Algorithm "xor" from p. 4 of Marsaglia, "Xorshift RNGs" */
        uint32_t x = a;
        x ^= x << 13;
        x ^= x >> 17;
        x ^= x << 5;
        return a = x;
    }
};

int main() {
    xorshift32_state rng{1};
    std::uniform_int_distribution<int> uni(0, 100);
    printf("Random numbers: %d\n", uni(rng));
    return 0;
}
```

此时编译将会报错，因为缺少 `std::uniform_int_distribution` 需要的成员函数。补全成员函数

```cpp
#include <random>
#include <thread>

struct xorshift32_state {
    uint32_t a;

    using value_type = uint32_t;

    /* The state must be initialized to non-zero */
    uint32_t operator()() noexcept {
        /* Algorithm "xor" from p. 4 of Marsaglia, "Xorshift RNGs" */
        uint32_t x = a;
        x ^= x << 13;
        x ^= x >> 17;
        x ^= x << 5;
        return a = x;
    }

    // 注意 xorshift32 最小值不能为 0
    static constexpr uint32_t min() noexcept { return 1; }
    static constexpr uint32_t max() noexcept { return std::numeric_limits<float>::max(); }
};

int main() {
    xorshift32_state rng{1};
    std::uniform_int_distribution<int> uni(0, 100);
    printf("Random numbers: %d\n", uni(rng));
    printf("Random numbers: %d\n", uni(rng));
    printf("Random numbers: %d\n", uni(rng));
    printf("Random numbers: %d\n", uni(rng));
    return 0;
}
```

要让每次生成的序列不同，使用 `std::random_deivce` 初始化。



#### Wang's hash

Xorshift 算法需要前一次生成的结果作为新的种子，因此前后生成的结果具有较高的相关性。更好的算法是使用 Wang's hash 算法，它能够在每次用完后直接丢弃。例如

```cpp
// xorshift
xorshift32_state rng{1};
for(int i = 0; i < 10; i++) {
    printf("Random numbers: %d\n", uni(rng));
}

// wangshash
for(int i = 0; i < 10; i++) {
    wangshash rng{i};
    printf("Random numbers: %d\n", uni(rng));
}
```

这有利于并行，因为每个循环处理不同的生成器对象，不会有线程存取冲突。

![[现代 Cpp 库.assets/wang's hash.pdf#height=400]]



## 内存池

### 内存检验

#### 内存分配

使用测试代码验证内存分配开销

```cpp
#include <chrono>
#include <iostream>
#include <list>
#include <tuple>
#include <variant>
#include <vector>


// 高精度计时器
typedef std::chrono::high_resolution_clock __clock;
typedef std::chrono::milliseconds __milliseconds;
typedef std::chrono::microseconds __microseconds;
typedef std::chrono::seconds __seconds;

#define TIMER_START(NAME) __clock::time_point __TIMER##NAME##_START = __clock::now();
#define TIMER_END(NAME)                                                                                           \
    __clock::time_point __TIMER##NAME##_END = __clock::now();                                                     \
    {                                                                                                             \
        auto d = std::chrono::duration_cast<__microseconds>(__TIMER##NAME##_END - __TIMER##NAME##_START).count(); \
        std::cout<< "Timer " << #NAME << ": " << d << "us\n";                                                    \
    }

int main() {
    {
        std::vector<char> a;
        TIMER_START(vector);
        for(int i = 0; i < 65536; i++) {
            a.push_back(42);
        }
        TIMER_END(vector)
    }
    {
        std::list<char> b;
        TIMER_START(list);
        for(int i = 0; i < 65536; i++) {
            b.push_back(42);
        }
        TIMER_END(list)
    }
    return 0;
}
```

运行得到

```shell
Timer vector: 1802us
Timer list: 8446us
```



这其中 `list` 由于需要频繁分配节点，主要开销在于 ` new/malloc ` 新的内存空间需要进入操作系统内核。每个标准容器都有一个分配器模板参数

```cpp
std::vector<char, std::allocator<char>> a;

// 使用分配器分配内存
std::allocator<char> a;
char *p = a.allocate(4);
a.deallocate(p, 4);
```



如果改为从固定的 buffer 中获得就能显著提高运行效率

```cpp
// list 每次分配 24 字节，包括 prev, next ,char，其中 char 对齐到指针大小，因此占 8 字节
static char buf[65536 * 24];
static size_t watermark = 0;

// 自定义分配器
template <class T> struct custom_allocator {
    using value_type = T;

    custom_allocator() = default;
	template <class U> constexpr custom_allocator(const custom_allocator<U>& other) noexcept {}

	// 每次分配指定字节，移动标记
    T* allocate(size_t n) {
        // 24 Bytes per allocation
        // std::cout<< "allocate " << n * sizeof(T) << " bytes\n";
        char* p = buf + watermark;
        watermark += n * sizeof(T);
        return (T*)p;
    }

	// 不释放（由于 vector 在扩容时会释放内存，为了防止 watermark 出错，这里不做操作）
    void deallocate(T* p, size_t n) {  
        // do nothing
    }

    constexpr bool operator==(const custom_allocator& other) const noexcept { return this == &other; }
};

int main() {
    {
        watermark = 0;
        std::vector<char, custom_allocator<char>> a;
        TIMER_START(vector);
        for(int i = 0; i < 65535; i++) {
            a.push_back(42);
        }
        TIMER_END(vector)
    }
    {
        watermark = 0;
        std::list<char, custom_allocator<char>> b;
        TIMER_START(list);
        for(int i = 0; i < 65535; i++) {
            b.push_back(42);
        }
        TIMER_END(list)
    }
    return 0;
}

```

运行得到

```shell
Timer vector: 2180us
Timer list: 3301us
```

现在 `list` 分配内存的速度显著提升。



#### 资源管理

创建一个资源管理结构，在分配器中只保留资源指针，防止分配器拷贝的开销。

```cpp
static char g_buf[65536 * 24];

struct memory_source {
    size_t watermark = 0;
    char* buf = g_buf;

    char* allocate(size_t n, size_t align) {
        // 对齐字节数（向上取整再乘对齐字节数）
        watermark = (watermark + align - 1) / align * align;
        char* p = buf + watermark;
        watermark += n;
        return p;
    }
};

template <class T> struct custom_allocator {
    using value_type = T;
    memory_source* source = nullptr;

    custom_allocator() = default;
    custom_allocator(memory_source* s): source(s) {}
    template <class U> constexpr custom_allocator(const custom_allocator<U>& other) noexcept: source(other.source) {}

    T* allocate(size_t n) {
        // alignof(T) 得到 T 的对齐字节数
        char* p = source->allocate(n * sizeof(T), alignof(T));
        return (T*)p;
    }

    void deallocate(T* p, size_t n) {
        // do nothing
    }

    constexpr bool operator==(const custom_allocator& other) const noexcept { return source == other.source; }
};

int main() {
    memory_source source;
    {
        source.watermark = 0;
        std::vector<char, custom_allocator<char>> a{custom_allocator<char>{&source}};
        TIMER_START(vector);
        for(int i = 0; i < 65535; i++) {
            a.push_back(42);
        }
        TIMER_END(vector)
    }
    {
        source.watermark = 0;
        std::list<char, custom_allocator<char>> b{custom_allocator<char>{&source}};
        TIMER_START(list);
        for(int i = 0; i < 65535; i++) {
            b.push_back(42);
        }
        TIMER_END(list)
    }
    return 0;
}
```



### 多态内存资源
#### 分配器

标准库提供了内存资源分配器

```cpp
#include <memory_resource>

// 全局内存池
std::pmr::synchronized_pool_resource pool;

static char g_buf[65536 * 2];

int main() {
    {
        // 设置内存资源初始字节数，指定全局内存池，当内存不足时，会自动分配内存
        std::pmr::monotonic_buffer_resource source{1000, &pool};
        std::vector<char, std::pmr::polymorphic_allocator<char>> a{&source};
        TIMER_START(vector);
        for(int i = 0; i < 65535; i++) {
            a.push_back(42);
        }
        TIMER_END(vector)
    }
    {
        // 使用 g_buf 作为内存池，指定全局内存池，当内存不足时，会自动分配内存
        std::pmr::monotonic_buffer_resource source{&g_buf, sizeof(g_buf), &pool};
        std::list<char, std::pmr::polymorphic_allocator<char>> b{&source};
        TIMER_START(list);
        for(int i = 0; i < 65535; i++) {
            b.push_back(42);
        }
        TIMER_END(list)
    }
    return 0;
}
```

其中容器的模板参数较为复杂，可以使用预定义的别名

```cpp
std::vector<char, std::pmr::polymorphic_allocator<char>> a{&source};
std::list<char, std::pmr::polymorphic_allocator<char>> b{&source};

// 别名
std::pmr::vector<char> a{&source}; 
std::pmr::list<char> b{&source};
```

这样我们只需要指定内存资源类型。



#### 标准资源

默认情况下使用 `new/delete` 管理内存资源

```cpp
std::pmr::list<char> b{std::pmr::get_default_resource()};
std::pmr::list<char> b{std::pmr::new_delete_resource()};	// get_default_resource()
```

可以修改默认使用的内存资源

```cpp
std::pmr::monotonic_buffer_resource source{1000, &pool};
std::pmr::set_default_resource(&source);
```

使用完成后需要将其设置回默认的内存资源，防止 `source` 析构后内存分配出现问题。



#### 增长缓存资源

使用 `monotonic_buffer_resource` 预先分配大块的缓存，当有内存使用需要时，则从中划分出内存，因此相较于每次都单独申请内存更为高效。当分配的缓存用完时，则会申请更大的内存空间；可以指定申请的内存资源

```cpp
std::pmr::synchronized_pool_resource pool;
std::pmr::monotonic_buffer_resource source{1000, &pool};
```

如果不指定，则会使用默认的内存资源

```cpp
std::pmr::monotonic_buffer_resource source{1000, std::pmr::get_default_resource()};
```

由 `monotonic_buffer_resource` 分配的内存不会被释放和回收，直到整个对象被销毁。因此它适用于临时分配大量内存的场景。



#### 空内存资源

使用 `null_memory_resource` 表示没有内存资源，当有内存需要时抛出 `std::bad_alloc` 错误。例如

```cpp
std::pmr::monotonic_buffer_resource source{1000, std::pmr::null_memory_resource()};
```

用于限制进一步申请更多内存资源。



#### 内存池资源

分为线程同步和无线程同步的内存池 `synchronized_pool_resource/unsynchronized_pool_resource`，它们在自动分配大块内存的同时，允许内存回收，并且前者提供线程安全的内存管理。



#### 资源效率

不同的内存资源分配效率不同，按照顺序从低到高为

```shell
windows malloc < synchronized_pool_resource < glibc malloc < unsynchronized_pool_resource < monotonic_buffer_resource
```



## format

格式化库 `<format>` 用于格式化输出，提供类似于 Python 中的格式化字符串的功能

```cpp
std::string message = std::format("Name: {}, Age: {}, Height: {:.2f} ft", name, age, height);
std::cout<< message << std::endl;
```

每个变量值会依次替换 `{}` 位置。提供特殊参数

- `{}` 默认替换类型
- `{:.2f}` 浮点类型，指定 2 位精度
- `{:#x}` 输出为 16 进制
- `{:10}` 指定最小宽度为 10，默认右对齐
- `{:<10}` 左对齐并指定最小宽度
- `{:>10}` 右对齐并指定最小宽度
- `{:^10}` 居中对齐并指定最小宽度
- `{:0>10}` 右对齐，左侧填充 0 直到达到最小宽度

新的版本还可能提供直接将变量名填写在 `{}` 中的格式。



## ratio

自 C++11 起，C++ 标准库提供了一个接口允许你具体指定编译期分数，并允许对它们执行编译期运算。这就是 ratio 类

```cpp
namespace std {
    template <intmax_t N, intmax_t D = 1>
    class ratio {
        public:
        	typedef ratio<num,den> type;
        	static constexpr intmax_t num;	// 分子
        	static constexpr intmax_t den;	// 分母，默认为 1
    };
}
```



通过 ratio 类，可以定义分数，**会自动进行约分**。通过 `::num,::den` 分别访问分子和分母

```cpp
#include <ratio>

int main()
{
    typedef ratio<5, 3> FiveThirds;
    std::cout<< FiveThirds::num << "/" << FiveThirds::den << std::endl;
 
    typedef ratio<25, 15> AlsoFiveThirds;
    std::cout<< AlsoFiveThirds::num << "/" << AlsoFiveThirds::den << std::endl;
 
    typedef ratio<42,42> one;
    std::cout<< one::num << "/" << one::den << std::endl;
 
    typedef ratio<0> zero;        //分母默认为1
    std::cout<< zero::num << "/" << zero::den << std::endl;
 
    typedef ratio<7, -3> Neg1;    //-号会被移动到分子身上
    std::cout<< Neg1::num << "/" << Neg1::den << std::endl;
 
    typedef ratio<-7, 3> Neg2;    //分子的-号保持不变
    std::cout<< Neg2::num << "/" << Neg2::den << std::endl;
 
    typedef ratio<-7, -3> Neg3;   //分子分母的-号都会消失
    std::cout<< Neg3::num << "/" << Neg3::den << std::endl;
}
```



并且 ratio 提供基本运算

| 运算                | 作用     |
| ------------------- | -------- |
| ratio_add           | 和       |
| ratio_subtract      | 差       |
| ratio_multiply      | 积       |
| ratio_divide        | 商       |
| ratio_equal         | 等于     |
| ratio_not_equal     | 不等于   |
| ratio_less          | 小于     |
| ratio_less_equal    | 小于等于 |
| ratio_greater       | 大于     |
| ratio_greater_equal | 大于等于 |

比较操作返回 trait 类型，通过 `::value` 获得布尔值，例如

```cpp
std::cout<< boolalpha;
std::cout<< std::ratio_equal<std::ratio<1, 3>, std::ratio<1, 3>>::value << std::endl;
std::cout<< std::ratio_equal<std::ratio<2, 3>, std::ratio<1, 3>>::value << std::endl;
std::cout<< std::ratio_greater<std::ratio<2, 3>, std::ratio<1, 3>>::value << std::endl;
```



在 ratio 库中预定义了一些常用单位，例如

![[20200412092819731.png]]

这使得我们可以快速得到纳秒单位

```cpp
// LL 表示 long long 类型
typedef std::ratio<1, 1000000000LL> test;
std::cout<< test::num << "/" << test::den << std::endl;
 
typedef std::nano _nano;
std::cout<< _nano::num << "/" << _nano::den << std::endl;
```

上述定义的两个类型是等价的。



## chrono

chrono 是 C++11 中的时间库，提供计时，时钟等功能。



### 精度

时钟节拍（时间精度）可通过 ratio 库定义。例如

* `ratio<60, 1>` 分钟 minutes
* `ratio<1, 1>` 秒 seconds
* `ratio<1, 1000>` 毫秒 milliseconds

chrono 借助时间精度来定义周期，例如

```cpp
#include <iostream>  
#include <chrono>  

using namespace std;  

int main()  
{  
    cout<< "millisecond : ";  
    // std::chrono::milliseconds::period 是一个 ratio 类型
    cout<< std::chrono::milliseconds::period::num << "/" 
         << std::chrono::milliseconds::period::den << "s" <<endl;  
    system("pause");  
    return 0;  
} 
```



### 时间段

类型 `std::chrono::duration` 表示一段时间，定义为

```cpp
template <class Rep, class Period = ratio<1>>
class duration;
```

其中 Rep 表示一种数值类型，比如 int float double；Period 是单位精度，比如 second milisecond 。



通过成员函数 count() 返回当前对象有多少个单位时间

```cpp
#include <iostream>  
#include <chrono>  

int main()  
{  
    // 定义 1000 毫秒，也就是 1 秒，count() 返回 1000
    std::chrono::milliseconds mscond(1000);
    std::cout<< mscond.count() << " milliseconds.\n";  

    std::cout<< mscond.count() * std::chrono::milliseconds::period::num / std::chrono::milliseconds::period::den;  
    std::cout<< " seconds.\n";  
    system("pause");  
    return 0;  
} 
```

借助 `std::chrono::duration_cast` 进行时间类型的转换，例如毫秒转秒

```cpp
std::chrono::milliseconds ms(54802);
std::chrono::seconds s = std::chrono::duration_cast<std::chrono::seconds>(ms);
```

这里的结果是截断的，不进行舍入。



### 时间点

类型 `std::chrono::time_point` 表示一个具体时间，定义为

```cpp
template <class Clock, class Duration = typename Clock::duration>
class time_point;
```

其中 Clock 指定所要使用的时钟，Duration 指定计时单位。每个时间点都有一个时间戳，即时间原点。chrono 库中采用的是 Unix 的时间戳 1970年1月1日 00:00，所以 time_point 也就是距离时间戳的时间长度。



通过 `time_since_epoch()` 获得当前时间点距离时间戳的时间长度

```cpp
#include <iostream>  
#include <chrono>  
#include <ctime>  

using namespace std;  

int main()  
{  
    // 距离时间戳 2 秒，tp.time_since_epoch().count() 返回 2
    chrono::time_point<chrono::system_clock, chrono::seconds> tp(chrono::seconds(2));  
    cout<< "to epoch : " << tp.time_since_epoch().count() << "s" <<endl;  
    
    // 转化为 ctime，打印输出时间点  
    time_t tt = chrono::system_clock::to_time_t(tp);  
    char a[50];  
    ctime_s(a, sizeof(a), &tt);  
    cout<< a;  
    system("pause");  
    return 0;  
}  
```



### 时钟

时钟代表当前的系统时间。chrono 中有三种时钟：system_clock，steady_clock 和 high_resolution_clock。每一个 clock 类中都有确定的 `time_point, duration, Rep, Period` 类型。



由于时钟自动可调，这种调节可能会破坏节拍频率的均匀分布，因此 system_clock 是不稳定的。C++ 标准库提供一个稳定时钟

* `std::chrono::steady_clock`
* `std::chrono::high_resolution_clock` 是标准库中提供的具有最小节拍周期（因此具有最高的精度的时钟）

我们可以比较这 3 种时钟的测量精度：分别为 100 纳秒、1 纳秒和 1 纳秒。

```cpp
#include <iostream>  
#include <chrono>  

using namespace std;  

int main()  
{  
    cout<< "system clock: ";  
    cout<< chrono::system_clock::period::num << "/" << chrono::system_clock::period::den << "s" << endl;  
    cout<< "steady clock: ";  
    cout<< chrono::steady_clock::period::num << "/" << chrono::steady_clock::period::den << "s" << endl;  
    cout<< "high resolution clock: ";  
    cout<< chrono::high_resolution_clock::period::num << "/" << chrono::high_resolution_clock::period::den << "s" << endl;  
    system("pause");  
    return 0;  
}
```



通过成员函数 `now()` 获取当前系统时间，`time_point_cast()` 用于转换时间精度，例如

```cpp
#include <iostream>  
#include <chrono>  
#include <ctime>  

using namespace std;  

int main()  
{  
    // 定义毫秒级别的时钟类型  
    typedef chrono::time_point<chrono::system_clock, chrono::milliseconds> microClock_type;  
    // 获取当前时间点，windows system_clock 是 100 纳秒级别的，所以要转换  
    microClock_type tp = chrono::time_point_cast<chrono::milliseconds>(chrono::system_clock::now());  
    // 转换为 ctime 用于打印显示时间  
    time_t tt = chrono::system_clock::to_time_t(tp);  
    char _time[50];  
    ctime_s(_time,sizeof(_time),&tt);  
    cout<< "now time is : " << _time;  
    // 计算距离 1970-1-1,00:00 的时间长度，因为当前时间点定义的精度为毫秒，所以输出的是毫秒  
    cout<< "to 1970-1-1,00:00  " << tp.time_since_epoch().count() << "ms" << endl;  
    system("pause");  
    return 0;  
}  
```



### 计时

利用 now 获得时间点，然后作差计算用时

```cpp
using std::chrono::high_resolution_clock = __clock;
using std::chrono::milliseconds = __milliseconds;

// 获得开始时间
std::chrono::time_point<__clock> start = __clock::now();

// 执行代码

// 获得终止时间
std::chrono::time_point<__clock> end = __clock::now();

// 计算时间差（毫秒）
int d = std::chrono::duration_cast<__milliseconds>(end - start).count();
```



## optional

在 C++17 中引入了 `std::optional`，可以更安全地获得返回值。



### 使用场景

当一个函数可能返回值也可能不返回值时，过去常用的方法是使用一个标记布尔变量。例如

```cpp
int get_int(int x, bool& success) {
    if(x > 0) {
        success = true;
        return x;
    } else {
        success = false;
        return -1;
    }
}

bool success;
int x = get_int(10, success);
if (success) {
	// do something
}
```

但是这种方式在传参时非常麻烦，并且需要额外定义标记变量。可以改为返回指针，但是又会造成内存管理问题。



在引入 `optional` 后，可以用它包装返回值，包装后可以隐式转换为 bool 值用于判断返回是否为空

```cpp
std::optional<int> get_optional_int(int x) {
    if(x > 0) {
        return x;
    } else {
        // 返回空值
        return std::nullopt;
    }
}

auto x = get_optional_int(10);
if (x) {
	// do something
}
```



### 成员函数

#### 创建对象

如果需要手动创建 `optional` 对象，使用

```cpp
auto y = std::make_optional<int>(10);
auto null = std::make_optional<int>();
```



#### 获得值

被包装的值可以通过 `.value()` 获得，或者通过 `*value` 获得；通过 `.has_value()` 显式获得是否为空。例如

```cpp
auto x = get_optional_int(10);
if (x.has_value()) {
	std::cout<< x.value() << std::endl;
}
```

如果想要指定为空时的返回值，可以使用 `.value_or()` 指定

```cpp
// 当 x 为空时得到 -1
std::cout<< x.value_or(-1) << std::endl;
```



#### 类型转换

不存在 `optional` 到被封装值的类型的隐式转换，但是可以将值赋给 `optional` 对象

```cpp
std::optional<int> op;
op = 10;
int x = op;	// 报错
```



## regex

在 C++11 中引入了正则表达式，需要导入库 `regex` 。



### 语法规则
#### 中括号

使用 `[expr]` 匹配**包含**在表达式内的字符或者排序规则表达式；`[^expr]` 匹配**不包含**在表达式内的字符或者排序规则表达式

* 单个字符：如 `[A]` 匹配 A
* `char1-char2` 形式的字符域：如 `[A-F]` 匹配大写 A 到 F 中的任何一字母
* `[:name:]` 形式字符类：匹配对应的字符类，如下

|  字符类名  |                    说明                    |
| :--------: | :----------------------------------------: |
| alnum 或 w |         字母（不区分大小写）和数字         |
|   alpha    |            字母（不区分大小写）            |
|   blank    |                空格或制表符                |
|   cntrl    |              文件格式转义字符              |
| digit 或 d |                    数字                    |
|   graph    |   字母（不区分大小写） 、数字、英文标点    |
|   lower    |                  小写字母                  |
|   upper    |                  大写字母                  |
|   print    | 字母（不区分大小写）、数字、英文标点和空格 |
|   punct    |                  英文标点                  |
| space 或 s |                    空格                    |
|   xdigit   |      表示十六进制字符（abcdefABCDEF）      |



#### 匹配字符

| 符号 | 作用               |
| ---- | ------------------ |
| `^`  | 匹配目标序列的开头 |
| `$`  | 匹配目标序列的末尾 |
| `.`  | 通配符             |



#### 转义序列

通过 `\` 搭配其它字符表达转义效果：

* `\\, \f, \n, \r, \t, \v` 分别匹配反斜杠、换页符、回车符、水平制表符和垂直制表符
* 8 进制数转义： `\ooo`，如 `\101` 匹配字符 A
* 16 进制转义： `\xhh`，如 `\x41` 匹配字符 A
* Unicode 转义： `\uhhhh`， 如 `\u0041` 匹配字符 A
* DSW 字符转义

| 转义序列 |  等效命名类   |    默认命名类    |
| :------: | :-----------: | :--------------: |
|    \d    |  `[ [:d:] ]`  |  `[[:digit:]]`   |
|    \D    | `[^[: D : ]]` | `[^[: digit :]]` |
|    \s    |  `[[: s :]]`  | `[[: space :]]`  |
|    \S    | `[^[: S :]]`  | `[^[: space :]]` |
|    \w    |  `[[: w :]]`  |  `[a-zA-Z0-9]`   |
|    \W    | `[^[: W :]]`  |  `[^a-zA-Z0-9]`  |



#### 限定符

指定要匹配的表达式 E 出现的次数

|   量词   |                      含义                       |
| :------: | :---------------------------------------------: |
|   `E?`   | 匹配 0 次或者 1 次 E（表达式），等价于 `E{0,1}` |
|   `E+`   |           多次匹配 E，等价于 `E{1,}`            |
|   `E*`   |       匹配 0 次或者多次 E，等价于 `E{0,}`       |
|  `E{n}`  |          匹配 n 次 E，等价于 `E{n,n}`           |
| `E{n,}`  |                 至少匹配 n 次 E                 |
| `E{,n}`  |                 至多匹配 n 次 E                 |
| `E{n,m}` |          至少匹配 n 次，至多匹配 m 次           |



#### 断言

|   断言    |       含义        |
| :-----: | :-------------: |
|  `\b`   |     一个单词的边界     |
|  `\B`   |    一个非单词的边界     |
| `(?=E)` | 表达式后面是 E 则匹配成功  |
| `(?!E)` | 表达式后面不是 E 则匹配成功 |
| `(?:E)` |                 |



#### 分组

可以使用 `()` 对表达式进行分组

```cpp
"([0-9]-)+abc(-[0-9])+"
```

其中括号有两组，使用括号的目的是为了让后面的 `+` 表示对括号中的内容多次匹配。



### 构造表达式

regex 提供 `basic_regex` 模板类，可以针对 char 和 wchar_t 类型进行特化

```cpp
typedef basic_regex<char> regex;
typedef basic_regex<wchar_t> wregex;
```

一般的构造函数形式为

```cpp
basic_regex(std::string str, flag_type f = default);
```

可以调整默认参数 flag_type

* icase 匹配时不区分大小写
* nosubs 不保存匹配的子表达式
* optimize 执行速度优于构造速度



例如匹配一个 http 链接的正则表达式定义为

```cpp
std::string pattern = "http(s)?://([\\w-]+\\.)+[\\w-]+(/[\\w- ./?%&=]*)?";
std::regex r(pattern);
```

我们解释一下上面的正则表达式：

* `http(s)?` 其中 (s)? 使用了 E? 语法，表示匹配 0 次或 1 次
* `://` 直接匹配
* `([\\w-]+\\.)+` 括号表示这是一整个表达式，最后 `+` 表示多次匹配
    * `[\\w-]+` 匹配字母和 `-` ，最后 `+` 表示多次匹配
    * `\\.` 使用转义表示匹配 `.`
* `[\\w-]+` 匹配字母和 `-` ，最后 `+` 表示多次匹配
* `(/[\\w- ./?%&=]*)?` 括号表示这是一整个表达式，最后 `?` 表示匹配 0 次或 1 次
    * `/` 直接匹配这个符号
    * `[\\w- ./?%&=]*` 匹配各种字符，最后 `*` 表示匹配 0 次或者多次



#### match_results

用于存放匹配结果的容器，特例化为不同的类型

```cpp
typedef match_results<const char*> cmatch;
typedef match_results<const wchar_t*> wcmatch;
typedef match_results<string::const_iterator> smatch;
typedef match_results<wstring::const_iterator> wsmatch;
```

最常用的是 smatch 类型，用于存放 string 的迭代器。

|  成员函数  |           作用           |
| :--------: | :----------------------: |
|    size   |      返回匹配的次数      |
|    str     | 将匹配结果作为字符串返回 |
|   length   |      返回匹配的长度      |
|   beign    |        头部迭代器        |
|    end     |        尾部迭代器        |
|  position  |      返回匹配的位置      |
| operator[] |     返回对应下标的值     |

我们可以通过分组来访问匹配的每一部分

```cpp
#include <iostream>
#include <regex>
#include <string>

using namespace std;

int main()
{
    std::string pattern = "([0-9])+(abc)([0-9])+";
    std::regex r(pattern);

    string str = "123abc456";
    std::smatch result;
    cout<< regex_match(str, result, r) << endl;
    cout<< result[0] << endl;
    cout<< result[1] << endl;
    cout<< result[2] << endl;
    cout<< result[3] << endl;
    
    return 0;
}
```

上面正则表达式分成了 3 组，匹配后的结果是 result ，此时可以通过 `[0]` 访问整个匹配结果，通过 `[1],[2],[3]` 分别访问匹配结果的每一组字符串。 也可以通过 str 进行访问

```cpp
result.str();	// 访问整个匹配结果的字符串
result.str(1);	// 访问第 1 组
result.str(2);	// 访问第 2 组
result.str(3);	// 访问第 3 组
```

注意如果匹配 `"([0-9])+abc([0-9])+"` 就只有 2 组，中间 `abc` 不会作为组来访问。



#### sub_match

这是保存匹配结果子串的容器，上面 `result[0]` 返回的就是这一类型，它同样可以特例化为不同类型

```cpp
typedef sub_match<const char*> csub_match;
typedef sub_match<const wchar_t*> wcsub_match;
typedef sub_match<string::const_iterator> ssub_match;
typedef sub_match<wstring::const_iterator> wssub_match;
```

sub_match 继承子 pair 类型，具有相似的成员变量

|  变量   |         含义         |
| :-----: | :------------------: |
| matched |    此匹配是否成功    |
|  first  | 此匹配序列开头的位置 |
| second  | 此匹配序列结尾的位置 |

其中 first 和 second 返回的是原字符序列的迭代器。

| 成员函数 |     作用     |
| :------: | :----------: |
|   str    | 转化为字符串 |
|  length  |   匹配长度   |



#### regex_match

判断检测**整个字符序列**是否与正则表达式匹配

```cpp
bool regex_match(string, match_results, rule, flag_type = default);
bool regex_match(string, rule, flag_type = default);
```

例如

```cpp
#include <iostream>
#include <regex>
#include <string>

using namespace std;

int main()
{
    std::string pattern = "[0-9]";
    std::regex r(pattern);

    string str = "123abc456";
    cout<< regex_match(str, r) << endl;

    return 0;
}
```

它将返回 0 ，而 regex_search 会返回 1 。



#### regex_search

查找字符序列中**是否包含**与正则表达式匹配的结果，找到第一个之后就会返回结果并停止查找

```cpp
bool regex_search(beginIt, endIt, match_results, rule, flag_type = default);
bool regex_search(string, match_results, rule, flag_type = default);
```

参数：

* match_results 是 `std::smatch` 类型的容器，用于存放匹配的结果
* rule 是正则表达式
* flag_type 是匹配方式，可用参数如下

![[image-202404242117.png]]

例如上面的匹配 http 链接的方法

```cpp
#include <iostream>
#include <regex>
#include <string>

using namespace std;

int main()
{
    std::string pattern = "http(s)?://([\\w-]+\\.)+[\\w-]+(/[\\w-./?%&=]*)?";
    std::regex r(pattern);

    string str = "http://www.baidu.com/s?ie=utf-8&f";
    std::smatch result;
    cout<< regex_search(str, result, r) << endl;
    cout<< result.str() << endl;

    return 0;
}
```

如果想要不断匹配，可以利用返回值

```cpp
while (regex_search(beginIt, endIt, result, r))
{
	// ...
	beginIt = result[0].second;	// 更新开始匹配的位置
}
```



#### regex_iterator

由于 regex_search 只能匹配一次，因此要找到所有匹配序列，就需要使用迭代器。与 match_results 类似，regex_iterator 也是模板对象，它特例化为几种不同类型

```cpp
typedef regex_iterator<const char*> cregex_iterator;
typedef regex_iterator<const wchar_t*> wcregex_iterator;
typedef regex_iterator<string::const_iterator> sregex_iterator;
typedef regex_iterator<wstring::const_iterator> wsregex_iterator;
```

迭代器实际上是每一步都执行 regex_search ，然后将迭代器指向匹配位置。例如

```cpp
#include <iostream>
#include <regex>
#include <string>

using namespace std;

int main()
{
    std::string pattern = "[0-9]+a";
    std::regex r(pattern);

    string str = "123abc456a";
    std::smatch result;
    
	// 注意这里同时定义了 it 和 end ，后者表示尾部迭代器
    for (std::sregex_iterator it(str.begin(), str.end(), r), end; it != end; ++it)
    {
        cout<< it->str() << endl;
    }

    return 0;
}
```

将会得到 `123a` 和 `456a` 作为输出。



#### regex_replace

将**所有**匹配正则表达式的字符序列替换为指定序列，返回替换后的结果

```cpp
string regex_replace(string, rule, newStr, flag_type = default);
```

此函数传入值，因此不会改变外部字符序列。

```cpp
#include <iostream>
#include <regex>
#include <string>

using namespace std;

int main()
{
    std::string pattern = "[0-9]+a";
    std::regex r(pattern);

    string str = "123abc456a";
    std::smatch result;
    cout<< regex_replace(str, r, "Hello") << endl;
    cout<< regex_replace(str, r, "") << endl;

    return 0;
}
```

分别得到 `HellobcHello` 和 `bc` 作为输出。



# 第三方库
## SQLite

[SQLite](https://www.sqlite.org/index.html) 是一个轻量级的开源数据库，源代码完全公开不受版权限制，实现了自给自足的、无服务器、零配置的 SQL 数据库引擎。 

```embed
title: "SQLite Download Page"
image: "https://www.sqlite.org/images/sqlite370_banner.gif"
description: ""
url: "https://www.sqlite.org/download.html"
```



### 安装和库文件

如果要调用 C/C++ 接口，需要下载源码包。下载两个种类的压缩包，分别是 Windows 和 Linux 下的源代码

![image-20220125174742007](image-20220125174742007.png)



下载完成后，对于 Windows 平台，需要将源代码编译生成动态库 dll ，这里我们借助 gcc 命令

```shell
gcc -shared sqlite3.c -o sqlite3.dll
```

这样就生成了对应的动态库 sqlite 3. Dll ，然后将头文件和动态库复制到需要的路径下即可。



对于 ubuntu 平台，需要编译安装源代码库，按照如下流程

```shell
tar xvzf sqlite-autoconf-3071502.tar.gz
cd sqlite-autoconf-3071502
./configure --prefix=/usr/local
make
make install
```



### C/C++ 接口

以下是重要的 C&C++ / SQLite 接口程序，可以满足您在 C/C++ 程序中使用 SQLite 数据库的需求。



该例程打开一个指向 SQLite 数据库文件的连接，返回一个用于其他 SQLite 程序的数据库连接对象，第二个参数是 sqlite 3 对象指针数组

```cpp
sqlite3_open(const char *filename, sqlite3 **ppDb)；;
```

如果 filename 参数是 NULL 或  `':memory:'` ，那么 sqlite 3_open () 将会在 RAM 中创建一个内存数据库，这只会在 session 的有效时间内持续。如果文件名 filename 不为 NULL，那么 sqlite 3_open () 将使用这个参数值尝试打开数据库文件。如果该名称的文件不存在，sqlite 3_open () 将创建一个新的命名为该名称的数据库文件并打开。



该例程提供了一个执行 SQL 命令的快捷方式，SQL 命令由 sql 参数提供，可以由多个 SQL 命令组成。

```cpp
sqlite3_exec(sqlite3*, const char *sql, sqlite_callback, void *data, char **errmsg);
```

在这里，第一个参数 `sqlite3` 是打开的数据库对象，`sqlite_callback` 是一个回调，data 作为其第一个参数，errmsg 将被返回用来获取程序生成的任何错误。程序解析并执行由 **sql** 参数所给的每个命令，直到字符串结束或者遇到错误为止。



该例程关闭之前调用 sqlite 3_open () 打开的数据库连接。

```cpp
sqlite3_close(sqlite3*);
```

所有与连接相关的语句都应在连接关闭之前完成。如果还有查询没有完成，`sqlite3_close()` 将返回 `SQLITE_BUSY` 禁止关闭的错误消息。



### 连接数据库

下面的 C 代码段显示了如何连接到一个现有的数据库。如果数据库不存在，那么它就会被创建，最后将返回一个数据库对象。

```cpp
#include <stdio.h>
#include <sqlite3.h>

int main(int argc, char* argv[])
{
   sqlite3 *db;
   char *zErrMsg = 0;
   int rc;

   rc = sqlite3_open("test.db", &db);

   if( rc )
   {
      fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
      exit(0);
   }
    else
   {
      fprintf(stderr, "Opened database successfully\n");
   }
   sqlite3_close(db);
}
```

现在，让我们来编译和运行上面的程序，在当前目录中创建我们的数据库 `test.db`。您可以根据需要改变路径。

```shell
$gcc test.c -l sqlite3
$./a.out
Opened database successfully
```

如果要使用 C++ 源代码，可以按照下列所示编译代码：

```shell
$g++ test.c -l sqlite3
```

在这里，把我们的程序链接上 sqlite 3 库，以便向 C 程序提供必要的函数。这将在您的目录下创建一个数据库文件 test. Db，您将得到如下结果：

```shell
-rwxr-xr-x. 1 root root 7383 May  8 02:06 a.out
-rw-r--r--. 1 root root  323 May  8 02:05 test.c
-rw-r--r--. 1 root root    0 May  8 02:06 test.db
```



### 创建表

下面的 C 代码段将用于在先前创建的数据库中创建一个表：

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <sqlite3.h> 

static int callback(void *NotUsed, int argc, char **argv, char **azColName)
{
   int i;
   for(i=0; i<argc; i++)
   {
      printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
   }
   printf("\n");
   return 0;
}

int main(int argc, char* argv[])
{
   sqlite3 *db;
   char *zErrMsg = 0;
   int  rc;
   char *sql;

   /* Open database */
   rc = sqlite3_open("test.db", &db);
   if( rc )
   {
      fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
      exit(0);
   }
    else
   {
      fprintf(stdout, "Opened database successfully\n");
   }

   /* Create SQL statement */
   sql = "CREATE TABLE COMPANY("  \
         "ID INT PRIMARY KEY     NOT NULL," \
         "NAME           TEXT    NOT NULL," \
         "AGE            INT     NOT NULL," \
         "ADDRESS        CHAR(50)," \
         "SALARY         REAL );";

   /* Execute SQL statement */
   rc = sqlite3_exec(db, sql, callback, 0, &zErrMsg);
   if( rc != SQLITE_OK )
   {
   fprintf(stderr, "SQL error: %s\n", zErrMsg);
      sqlite3_free(zErrMsg);
   }
    else
   {
      fprintf(stdout, "Table created successfully\n");
   }
   sqlite3_close(db);
   return 0;
}
```

上述程序编译和执行时，它会在 test. Db 文件中创建 COMPANY 表，最终文件列表如下所示：

```shell
-rwxr-xr-x. 1 root root 9567 May  8 02:31 a.out
-rw-r--r--. 1 root root 1207 May  8 02:31 test.c
-rw-r--r--. 1 root root 3072 May  8 02:31 test.db
```



### INSERT 操作

下面的 C 代码段显示了如何在上面创建的 COMPANY 表中创建记录：

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <sqlite3.h>

static int callback(void *NotUsed, int argc, char **argv, char **azColName){
   int i;
   for(i=0; i<argc; i++)
   {
      printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
   }
   printf("\n");
   return 0;
}

int main(int argc, char* argv[])
{
   sqlite3 *db;
   char *zErrMsg = 0;
   int rc;
   char *sql;

   /* Open database */
   rc = sqlite3_open("test.db", &db);
   if( rc )
   {
      fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
      exit(0);
   }
    else
   {
      fprintf(stderr, "Opened database successfully\n");
   }

   /* Create SQL statement */
   sql = "INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY) "  \
         "VALUES (1, 'Paul', 32, 'California', 20000.00 ); " \
         "INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY) "  \
         "VALUES (2, 'Allen', 25, 'Texas', 15000.00 ); "     \
         "INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY)" \
         "VALUES (3, 'Teddy', 23, 'Norway', 20000.00 );" \
         "INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY)" \
         "VALUES (4, 'Mark', 25, 'Rich-Mond ', 65000.00 );";

   /* Execute SQL statement */
   rc = sqlite3_exec(db, sql, callback, 0, &zErrMsg);
   if( rc != SQLITE_OK )
   {
      fprintf(stderr, "SQL error: %s\n", zErrMsg);
      sqlite3_free(zErrMsg);
   }
    else
   {
      fprintf(stdout, "Records created successfully\n");
   }
   sqlite3_close(db);
   return 0;
}
```

上述程序编译和执行时，它会在 COMPANY 表中创建给定记录，并会显示以下两行：

```shell
Opened database successfully
Records created successfully
```



### SELECT 操作

在我们开始讲解获取记录的实例之前，让我们先了解下回调函数的一些细节，这将在我们的实例使用到。这个回调提供了一个从 SELECT 语句获得结果的方式。它声明如下：

```cpp
typedef int (*sqlite3_callback)(
    void*,    /* Data provided in the 4th argument of sqlite3_exec() */
    int,      /* The number of columns in row */
    char**,   /* An array of strings representing fields in the row */
    char**    /* An array of strings representing column names */
);
```

如果上面的回调在 sqlite_exec () 程序中作为第三个参数，那么 SQLite 将为 SQL 参数内执行的每个 SELECT 语句中处理的每个记录调用这个回调函数。

下面的 C 代码段显示了如何从前面创建的 COMPANY 表中获取并显示记录：

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <sqlite3.h>

static int callback(void *data, int argc, char **argv, char **azColName)
{
   int i;
   fprintf(stderr, "%s: ", (const char*)data);
   for(i=0; i<argc; i++)
   {
      printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
   }
   printf("\n");
   return 0;
}

int main(int argc, char* argv[])
{
   sqlite3 *db;
   char *zErrMsg = 0;
   int rc;
   char *sql;
   const char* data = "Callback function called";

   /* Open database */
   rc = sqlite3_open("test.db", &db);
   if( rc )
   {
      fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
      exit(0);
   }else
   {
      fprintf(stderr, "Opened database successfully\n");
   }

   /* Create SQL statement */
   sql = "SELECT * from COMPANY";

   /* Execute SQL statement */
   rc = sqlite3_exec(db, sql, callback, (void*)data, &zErrMsg);
   if( rc != SQLITE_OK )
   {
      fprintf(stderr, "SQL error: %s\n", zErrMsg);
      sqlite3_free(zErrMsg);
   }else
   {
      fprintf(stdout, "Operation done successfully\n");
   }
   sqlite3_close(db);
   return 0;
}
```

上述程序编译和执行时，它会产生以下结果：

```shell
Opened database successfully
Callback function called: ID = 1
NAME = Paul
AGE = 32
ADDRESS = California
SALARY = 20000.0

Callback function called: ID = 2
NAME = Allen
AGE = 25
ADDRESS = Texas
SALARY = 15000.0

Callback function called: ID = 3
NAME = Teddy
AGE = 23
ADDRESS = Norway
SALARY = 20000.0

Callback function called: ID = 4
NAME = Mark
AGE = 25
ADDRESS = Rich-Mond
SALARY = 65000.0

Operation done successfully
```



### UPDATE 操作

下面的 C 代码段显示了如何使用 UPDATE 语句来更新任何记录，然后从 COMPANY 表中获取并显示更新的记录：

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <sqlite3.h> 

static int callback(void *data, int argc, char **argv, char **azColName)
{
   int i;
   fprintf(stderr, "%s: ", (const char*)data);
   for(i=0; i<argc; i++)
   {
      printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
   }
   printf("\n");
   return 0;
}

int main(int argc, char* argv[])
{
   sqlite3 *db;
   char *zErrMsg = 0;
   int rc;
   char *sql;
   const char* data = "Callback function called";

   /* Open database */
   rc = sqlite3_open("test.db", &db);
   if( rc )
   {
      fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
      exit(0);
   }
   else
   {
      fprintf(stderr, "Opened database successfully\n");
   }

   /* Create merged SQL statement */
   sql = "UPDATE COMPANY set SALARY = 25000.00 where ID=1; " \
         "SELECT * from COMPANY";

   /* Execute SQL statement */
   rc = sqlite3_exec(db, sql, callback, (void*)data, &zErrMsg);
   if( rc != SQLITE_OK )
   {
      fprintf(stderr, "SQL error: %s\n", zErrMsg);
      sqlite3_free(zErrMsg);
   }
   else
   {
      fprintf(stdout, "Operation done successfully\n");
   }
   sqlite3_close(db);
   return 0;
}
```

上述程序编译和执行时，它会产生以下结果：

```shell
Opened database successfully
Callback function called: ID = 1
NAME = Paul
AGE = 32
ADDRESS = California
SALARY = 25000.0

Callback function called: ID = 2
NAME = Allen
AGE = 25
ADDRESS = Texas
SALARY = 15000.0

Callback function called: ID = 3
NAME = Teddy
AGE = 23
ADDRESS = Norway
SALARY = 20000.0

Callback function called: ID = 4
NAME = Mark
AGE = 25
ADDRESS = Rich-Mond
SALARY = 65000.0

Operation done successfully
```



### DELETE 操作

下面的 C 代码段显示了如何使用 DELETE 语句删除任何记录，然后从 COMPANY 表中获取并显示剩余的记录：

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <sqlite3.h> 

static int callback(void *data, int argc, char **argv, char **azColName)
{
   int i;
   fprintf(stderr, "%s: ", (const char*)data);
   for(i=0; i<argc; i++)
   {
      printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
   }
   printf("\n");
   return 0;
}

int main(int argc, char* argv[])
{
   sqlite3 *db;
   char *zErrMsg = 0;
   int rc;
   char *sql;
   const char* data = "Callback function called";

   /* Open database */
   rc = sqlite3_open("test.db", &db);
   if( rc )
   {
      fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
      exit(0);
   }
   else
   {
      fprintf(stderr, "Opened database successfully\n");
   }

   /* Create merged SQL statement */
   sql = "DELETE from COMPANY where ID=2; " \
         "SELECT * from COMPANY";

   /* Execute SQL statement */
   rc = sqlite3_exec(db, sql, callback, (void*)data, &zErrMsg);
   if( rc != SQLITE_OK )
   {
      fprintf(stderr, "SQL error: %s\n", zErrMsg);
      sqlite3_free(zErrMsg);
   }
   else
   {
      fprintf(stdout, "Operation done successfully\n");
   }
   sqlite3_close(db);
   return 0;
}
```

上述程序编译和执行时，它会产生以下结果：

```shell
Opened database successfully
Callback function called: ID = 1
NAME = Paul
AGE = 32
ADDRESS = California
SALARY = 20000.0

Callback function called: ID = 3
NAME = Teddy
AGE = 23
ADDRESS = Norway
SALARY = 20000.0

Callback function called: ID = 4
NAME = Mark
AGE = 25
ADDRESS = Rich-Mond
SALARY = 65000.0

Operation done successfully
```





## Catch 2

在 [catch](https://github.com/catchorg/Catch2/releases) 的文档指出，对于 C++ 单元测试框架，目前已经有 Google Test, Boost. Test, CppUnit, Cute, 以及其它的一些，那么 catch 有什么优势呢，文档主要列举了以下这些优势：

* 简单易用：只需要下载 catch. Hpp ,包含到你的工程就可以了
* 不依赖外部库：只要你可以编译 C++11 ，有 C++ 的标准库就可以了
* 测试 case 可以分割为 sections ：每个 setcion 都是独立的运行单元
* 提供了 BDD 式的测试模式：可以使用 Given-When-Then section 来做 BDD 测试
* 只用一个核心的 assertion 宏来做比较。用标准的 C++ 运算符来做比较，但是可以分解表达式，记录表达式等号左侧和右侧的值
* 可以用任何形式的字符串给测试命名，不用担心名字是否合法



其它一些关键特性有：

* 可以给 test case 打 tag ，因而可以很容易的只跑某个 tag 组的 test cases
* 输出通过 reporter 对象完成，支持基本的文本和 XML 格式输出测试结果，也支持自定义 reporter
* 支持 JUnit xml 输出，这样可以和第三方工具整合，如 CI 服务器等
* 提供默认的 main () 函数，用户也可以使用自己的 main () 函数
* 提供一个命令行解析工具，用户在使用自己的 main () 函数的时候可以添加命令行参数
* catch 软件可以做自测试
* 提供可选的 assertion 宏，可以报告错误但不终止 test case
* 通过内置的 Approx () 语法支持可控制精度的浮点数比较
* 通过 Matchers 支持各种形式的比较，也支持自定义的比较方法



### 简单易用

Catch 是一个 header-only 的开源库，这意味着你只需要把一个头文件放到系统或者你的工程的某个目录，编译的时候指向它就可以了。

下面用一个 Catch 文档中的小例子说明如何使用 Catch ，假设你写了一个求阶乘的函数：

```cpp
int Factorial( int number ) 
{
   return number <= 1 ? number : Factorial( number - 1 ) * number; 
}
```

为了简单起见，将被测函数和要测试代码放在一个文件中，你只需要在这个函数之前加入两行：

```cpp
#define CATCH_CONFIG_MAIN
#include <catch.hpp>
```


第一行的作用是由 catch 提供一个 main 函数，第二行的作用是包含测试所需要的头文件，假设最后的文件为 catchTest. Cpp ，假设相关的文件安装到了 /usr/local/include 下，下面这样编译就可以了：

```shell
g++ -std=c++11 -o catchTest catchTest.cpp -I/usr/local/include/
```

后面的头文件路径可以不要。



运行一下，结果为：

```shell
===============================================================================
No tests ran

```

那么如何加入一个 test case 呢，很简单：

```cpp
TEST_CASE() 
{
    REQUIRE(Factorial(2) == 2);
}
```

当然你也可以为你的 TEST_CASE 起名字，或者加标签：

```cpp
TEST_CASE("Test with number big than 0", "[tag1]") 
{
    REQUIRE(Factorial(2) == 2);
}
```


其中 "Test with number big than 0" 是 test case 的名字，全局必须唯一； "tag 1" 是标签名，需要放在 `[]` 内部，一个 test case 可以有多个标签，多个 test case 可以使用相同的标签。 REQUIRE 是一个 assert 宏，用来判断是否相等。



### 命令行选项

以上面的这个简单程序为例，可以通过下面-？来查询命令行选项参数：

```shell
./catchTest -?
```

你会得到选项参数

```shell
Catch v2.13.8
usage:
  test [<test name|pattern|tags> ... ] options

where options are:
  -?, -h, --help                            display usage information
  -l, --list-tests                          list all/matching test cases
  -t, --list-tags                           list all/matching tags
  -s, --success                             include successful tests in
                                            output
  -b, --break                               break into debugger on failure
  -e, --nothrow                             skip exception tests
  -i, --invisibles                          show invisibles (tabs, newlines)
  -o, --out <filename>                      output filename
  -r, --reporter <name>                     reporter to use (defaults to
                                            console)
  -n, --name <name>                         suite name
  -a, --abort                               abort at first failure
  -x, --abortx <no. failures>               abort after x failures
  -w, --warn <warning name>                 enable warnings
  -d, --durations <yes|no>                  show test durations
  -D, --min-duration <seconds>              show test durations for tests
                                            taking at least the given number
                                            of seconds
  -f, --input-file <filename>               load test names to run from a
                                            file
  -#, --filenames-as-tags                   adds a tag for the filename
  -c, --section <section name>              specify section to run
  -v, --verbosity <quiet|normal|high>       set output verbosity
  --list-test-names-only                    list all/matching test cases
                                            names only
  --list-reporters                          list all reporters
  --order <decl|lex|rand>                   test case order (defaults to
                                            decl)
  --rng-seed <'time'|number>                set a specific seed for random
                                            numbers
  --use-colour <yes|no>                     should output be colourised
  --libidentify                             report name and version according
                                            to libidentify standard
  --wait-for-keypress <never|start|exit     waits for a keypress before
  |both>                                    exiting
  --benchmark-samples <samples>             number of samples to collect
                                            (default: 100)
  --benchmark-resamples <resamples>         number of resamples for the
                                            bootstrap (default: 100000)
  --benchmark-confidence-interval           confidence interval for the
  <confidence interval>                     bootstrap (between 0 and 1,
                                            default: 0.95)
  --benchmark-no-analysis                   perform only measurements; do not
                                            perform any analysis
  --benchmark-warmup-time                   amount of time in milliseconds
  <benchmarkWarmupTime>                     spent on warming up each test
                                            (default: 100)

For more detailed usage please see the project docs
```


一些比较常见的命令行选项的使用如下：

显示 test case 总体情况：

```shell
./catchTest -l
```

得到结果：

```shell
All available test cases:
  Anonymous test case 1
1 test case
```



显示所有的标签（tags）:

```shell
./catchTest -t
```

得到结果：

```shell
All available tags:
0 tags
```



运行某个 tag 下的所有 test cases ：

```shell
./catchTest [tag1]
```



运行某个名字的 test case ：

```shell
./catchTest "Test with number big than 0"
```



### Sections

一般的测试框架都采用基于类的 test fixture , 通常需要定义 setup () 和 teardown () 函数（或者在构造/析构函数中做类似的事情）。 Catch 不仅全面支持 test fixture 模式，还提供了一种 section 机制：每个 section 都是独立运行单元，比如下面：

```c++
TEST_CASE( "vectors can be sized and resized", "[vector]" ) 
{
	// 共享部分代码
    std::vector<int> v( 5 );

    REQUIRE( v.size() == 5 );
    REQUIRE( v.capacity() >= 5 );

    // 依次执行这些 SECTION
    SECTION( "resizing bigger changes sizeand capacity" ) 
    {
        v.resize( 10 );

        REQUIRE( v.size() == 10 );
        REQUIRE( v.capacity() >= 10 );
    }
    SECTION( "resizing smaller changes sizebut not capacity" ) 
    {
        v.resize( 0 );

        REQUIRE( v.size() == 0 );
        REQUIRE( v.capacity() >= 5 );
    }
    SECTION( "reserving bigger changes capacity but not size" ) 
    {
        v.reserve( 10 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 10 );
    }
    SECTION( "reserving smaller does not change sizeor capacity" ) 
    {
        v.reserve( 0 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 5 );
    }
}
```



### BDD-style

将 test case 和 section 重命名就可以得到一种 BDD 模式的测试，TDD（测试驱动开发）和 BDD（行为驱动开发）是两种比较流行的开发模式。下面是一个例子：

```cpp
SCENARIO( "vectors can be sized and resized", "[vector]" ) 
{
    // 给定条件
    GIVEN( "A vector with some items" ) 
    {
        std::vector<int> v( 5 );

        REQUIRE( v.size() == 5 );
        REQUIRE( v.capacity() >= 5 );

        // 当满足情况
        WHEN( "the sizeis increased" ) 
        {
            v.resize( 10 );

            // 就得到结果
            THEN( "the sizeand capacity change" ) 
            {
                REQUIRE( v.size() == 10 );
                REQUIRE( v.capacity() >= 10 );
            }
        }
        WHEN( "the sizeis reduced" ) 
        {
            v.resize( 0 );

            THEN( "the sizechanges but not capacity" ) 
            {
                REQUIRE( v.size() == 0 );
                REQUIRE( v.capacity() >= 5 );
            }
        }
        WHEN( "more capacity is reserved" ) 
        {
            v.reserve( 10 );

            THEN( "the capacity changes but not the size" ) 
            {
                REQUIRE( v.size() == 5 );
                REQUIRE( v.capacity() >= 10 );
            }
        }
        WHEN( "less capacity is reserved" ) 
        {
            v.reserve( 0 );

            THEN( "neither sizenor capacity are changed" ) 
            {
                REQUIRE( v.size() == 5 );
                REQUIRE( v.capacity() >= 5 );
            }
        }
    }
}
```

其中的 GIVEN WHEN THEN 都是用于测试结果的解释。



添加参数： success 输出成功结果， reporter 指定输出格式为 compact 简洁的

```shell
./test --reporter compact --success
```

输出结果：

```shell
test.cpp:11: passed: v.size() == 5 for: 5 == 5
test.cpp:12: passed: v.capacity() >= 5 for: 5 >= 5
test.cpp:20: passed: v.size() == 10 for: 10 == 10
test.cpp:21: passed: v.capacity() >= 10 for: 10 >= 10
test.cpp:11: passed: v.size() == 5 for: 5 == 5
test.cpp:12: passed: v.capacity() >= 5 for: 5 >= 5
test.cpp:30: passed: v.size() == 0 for: 0 == 0
test.cpp:31: passed: v.capacity() >= 5 for: 5 >= 5
test.cpp:11: passed: v.size() == 5 for: 5 == 5
test.cpp:12: passed: v.capacity() >= 5 for: 5 >= 5
test.cpp:40: passed: v.size() == 5 for: 5 == 5
test.cpp:41: passed: v.capacity() >= 10 for: 10 >= 10
test.cpp:11: passed: v.size() == 5 for: 5 == 5
test.cpp:12: passed: v.capacity() >= 5 for: 5 >= 5
test.cpp:50: passed: v.size() == 5 for: 5 == 5
test.cpp:51: passed: v.capacity() >= 5 for: 5 >= 5
Passed 1 test case with 16 assertions.
```



如果直接输出成功结果

```shell
./test --success
```

输出结果：

```shell
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
test is a Catch v2.13.8 host application.
Run with -? for options

-------------------------------------------------------------------------------
Scenario: vectors can be sized and resized
      Given: A vector with some items
-------------------------------------------------------------------------------
test.cpp:7
...............................................................................

test.cpp:11: PASSED:
  REQUIRE( v.size() == 5 )
with expansion:
  5 == 5

test.cpp:12: PASSED:
  REQUIRE( v.capacity() >= 5 )
with expansion:
  5 >= 5

-------------------------------------------------------------------------------
Scenario: vectors can be sized and resized
      Given: A vector with some items
       When: the sizeis increased
       Then: the sizeand capacity change
-------------------------------------------------------------------------------
test.cpp:18
...............................................................................

test.cpp:20: PASSED:
  REQUIRE( v.size() == 10 )
with expansion:
  10 == 10

test.cpp:21: PASSED:
  REQUIRE( v.capacity() >= 10 )
with expansion:
  10 >= 10

-------------------------------------------------------------------------------
Scenario: vectors can be sized and resized
      Given: A vector with some items
-------------------------------------------------------------------------------
test.cpp:7
...............................................................................

test.cpp:11: PASSED:
  REQUIRE( v.size() == 5 )
with expansion:
  5 == 5

test.cpp:12: PASSED:
  REQUIRE( v.capacity() >= 5 )
with expansion:
  5 >= 5

-------------------------------------------------------------------------------
Scenario: vectors can be sized and resized
      Given: A vector with some items
       When: the sizeis reduced
       Then: the sizechanges but not capacity
-------------------------------------------------------------------------------
test.cpp:28
...............................................................................

test.cpp:30: PASSED:
  REQUIRE( v.size() == 0 )
with expansion:
  0 == 0

test.cpp:31: PASSED:
  REQUIRE( v.capacity() >= 5 )
with expansion:
  5 >= 5

-------------------------------------------------------------------------------
Scenario: vectors can be sized and resized
      Given: A vector with some items
-------------------------------------------------------------------------------
test.cpp:7
...............................................................................

test.cpp:11: PASSED:
  REQUIRE( v.size() == 5 )
with expansion:
  5 == 5

test.cpp:12: PASSED:
  REQUIRE( v.capacity() >= 5 )
with expansion:
  5 >= 5

-------------------------------------------------------------------------------
Scenario: vectors can be sized and resized
      Given: A vector with some items
       When: more capacity is reserved
       Then: the capacity changes but not the size
-------------------------------------------------------------------------------
test.cpp:38
...............................................................................

test.cpp:40: PASSED:
  REQUIRE( v.size() == 5 )
with expansion:
  5 == 5

test.cpp:41: PASSED:
  REQUIRE( v.capacity() >= 10 )
with expansion:
  10 >= 10

-------------------------------------------------------------------------------
Scenario: vectors can be sized and resized
      Given: A vector with some items
-------------------------------------------------------------------------------
test.cpp:7
...............................................................................

test.cpp:11: PASSED:
  REQUIRE( v.size() == 5 )
with expansion:
  5 == 5

test.cpp:12: PASSED:
  REQUIRE( v.capacity() >= 5 )
with expansion:
  5 >= 5

-------------------------------------------------------------------------------
Scenario: vectors can be sized and resized
      Given: A vector with some items
       When: less capacity is reserved
       Then: neither sizenor capacity are changed
-------------------------------------------------------------------------------
test.cpp:48
...............................................................................

test.cpp:50: PASSED:
  REQUIRE( v.size() == 5 )
with expansion:
  5 == 5

test.cpp:51: PASSED:
  REQUIRE( v.capacity() >= 5 )
with expansion:
  5 >= 5

===============================================================================
All tests passed (16 assertions in 1 test case)
```



也可以使用下面的两个宏来使用链式的 WHEN 和 THEN

```cpp
AND_WHEN( something )
AND_THEN( something )
AND_GIVEN( something )
```

其使用方式与上面相同



### Assertion Macros

上面其实已经提到了一些 assertion 宏，像 REQUIRE 等，这里全面的介绍下。常用的有 REQUIRE 系列和 CHECK 系列：

```cpp
REQUIRE( expression )
CHECK( expression )
```

REQUIRE 宏在 expression 为 false 时将会终止当前 test case ,  CHECK 在 expression 为 false 时会给出警告信息但当前 test case 继续往下执行。



对应的还有：

```cpp
REQUIRE_FALSE( expression )
CHECK_FALSE( expression )
```

REQUIRE_FALSE 宏在 expression 为 true 时将会终止当前 test case ,  CHECK_FALSE 在 expression 为 true 时会给出警告信息但当前 test case 继续往下执行。



这里有一点要指出的是，对于：

```cpp
CHECK(a == 1 && b == 2)
CHECK( a == 2 || b == 1 )
```

这样的语句，由于宏展开的原因， catch 不能通过编译，需要在表达式的两端加上括号：

```cpp
CHECK((a == 1 && b == 2))
CHECK((a == 2 || b == 1 ))
```



我们在最初求阶乘的例子上进行测试，代码如下：

```cpp
TEST_CASE("Test with number big than 0", "[tag1]")
{
    REQUIRE(Factorial(2) == 2);
    // REQUIRE((Factorial(3) == 6) && (Factorial(4) == 24)); cannot compile
    CHECK(Factorial(0) == 1);
    REQUIRE((Factorial(3) == 6 && Factorial(4) == 24));
}
```

编译运行得到：

```shell
~/work/test/catch$ ./test -r compact -s
test.cpp:11: passed: Factorial(2) == 2 for: 2 == 2
test.cpp:13: failed: Factorial(0) == 1 for: 0 == 1
test.cpp:14: passed: (Factorial(3) == 6 && Factorial(4) == 24) for: true
Failed 1 test case, failed 1 assertion.
```

注意到，这里 `Factorial(0) == 1` 使用的是 CHECK ，由于 `0！= 1` ，故该测试失败，但因为我们用的是 CHECK ，故这里只是给出了警告信息，没有终止 test case 。



### Floating point comparisons

Catch 2 支持比较全面的浮点数比较，可能是作者在银行工作，这个测试框架也是针对作者写的银行业务的代码，这些代码对数值比较的要求较多。具体的说 catch 2 浮点数比较采用类 Approx ,  Approx 采用数值初始化，同时支持以下三种属性：

Epsilon：相当于误差相对于原值的百分比值，比如 epsilon=0.01 ，则意味着与其比较的值在该 Approx 值的 %1 的范围均视为相等。

默认值为 `std::numeric_limits::epsilon()*100` 

```cpp
Approx target = Approx(100).epsilon(0.01);
100.0 == target; // Obviously true
200.0 == target; // Obviously still false
100.5 == target; // True, because we set target to allow up to 1% difference
```

Margin： epsilon 是一个相对的百分比值， margin 是一个绝对值，其默认值为 0 。比如：

```cpp
Approx target = Approx(100).margin(5);
100.0 == target; // Obviously true
200.0 == target; // Obviously still false
104.0 == target; // True, because we set target to allow absolute difference of at most 5
```

scale：有时比较的两边采用不同的量级，采用 scale 后， Approx 允许的误差为： `(Approx::scale + Approx::value) * epsilon` ，其默认值为 0 。比如：

```cpp
Approx target = Approx(100).scale(100).epsilon(0.01);
100.0 == target; //  true
101.5 == target; //  true
200.0 == target; // false
100.5 == target; // True, because we set target to allow up to 1% difference, and final target value is 100	

Approx target1 = Approx(100).scale(100).margin(5);
100.0 == target1; // true
200.0 == target1; // false
104.0 == target1; // True, because we set target to allow absolute difference of at most 5
```



查看 Catch 2 的源码，可以找到 Approx 的实现如下：

```cpp
bool Approx::equalityComparisonImpl(const double other) const 
{
    // First try with fixed margin, then compute margin based on epsilon, scale and Approx's value
    // Thanks to Richard Harris for his help refining the scaled margin value
    return marginComparison(m_value, other, m_margin)
            || marginComparison(m_value, other, m_epsilon * (m_scale + std::fabs(std::isinf(m_value)? 0 : m_value)));
}  

// Performs equivalent check of std::fabs(lhs - rhs) <= margin
// But without the subtraction to allow for INFINITY in comparison
bool marginComparison(double lhs, double rhs, double margin) 
{
    return (lhs + margin >= rhs) && (rhs + margin >= lhs);
}
```

因此我们不需要显式的设置 epsilon , scale , 或者 margin ，一般情况下使用它们的默认值就可以了：

```cpp
REQUIRE( 1 == Approx( 2.1 ) );
```

Catch 2 也为用户定义了一个替代字符 _a ，这样使用时不用每次都写 Approx , 只需要在前部包含命名空间就可以了：

```cpp
using namespace Catch::literals;
REQUIRE( 1 == 2.1_a );
```



### Exceptions

下面两个宏用来测试 expression 中没有异常抛出，满足条件则 assertion 为 true

```cpp
REQUIRE_NOTHROW( expression )
CHECK_NOTHROW( expression )
```



下面两个宏用来测试 expression 中有异常抛出，满足条件则 assertion 为 true

```cpp
REQUIRE_THROWS( expression )
CHECK_THROWS( expression )
```



下面两个宏用来测试 expression 中有某种类型的异常抛出，满足条件则 assertion 为 true

```cpp
REQUIRE_THROWS_AS( expression, exception type )
CHECK_THROWS_AS( expression, exception type )
```



下面两个宏用来测试 expression 中有异常名包含某个 string 的异常或符合某种 string matcher 的异常抛出，满足条件则 assertion 为 true 

```cpp
REQUIRE_THROWS_WITH( expression, string or string matcher )
CHECK_THROWS_WITH( expression, string or string matcher )
```

例如：

```cpp
REQUIRE_THROWS_WITH( openThePodBayDoors(), Contains( "afraid" ) && Contains( "can't do that" ) );
REQUIRE_THROWS_WITH( dismantleHal(), "My mind is going" );
```



下面两个宏用来测试 expression 中有某种类型且符合某种 string matcher 的异常抛出，满足条件则 assertion 为 true

```cpp
REQUIRE_THROWS_MATCHES( expression, exception type, matcher for given exception type )
CHECK_THROWS_MATCHES( expression, exception type, matcher for given exception type )
```



### Matchers

Matchers 顾名思义就是某个 string 或 int 或其它类型是否和 Matcher 所定义的条件 match 。



#### String matchers

内置的 string matchers 有 StartsWith, EndsWith, Contains, Equals 和 Matches，前四个是常见的 string 或 substring 的比较。

例如，验证某个 string 是否以某个 string 结束：

```cpp
using Catch::Matchers::EndsWith; // or Catch::EndsWith
std::string str = getStringFromSomewhere();
REQUIRE_THAT( str, EndsWith( "as a service" ) )
```



EndsWith 也支持是否大小写采用大小写匹配：

```cpp
REQUIRE_THAT( str, EndsWith( "as a service", Catch::CaseSensitive::No ) ); 
```

也支持多个 Match 串联：

```cpp
REQUIRE_THAT( str, 
    EndsWith( "as a service" ) || 
    (StartsWith( "Big data" ) && !Contains( "web scale" ) ) );
```



最后一个 Matches 是 matchECMAScript 类型的正则表达式，例如：

```cpp
using Catch::Matchers::Matches;
REQUIRE_THAT(std::string("this string contains 'abc' as a substring"),
                         Matches("this string CONTAINS 'abc' as a substring", Catch::CaseSensitive::No));
```



#### Vector matchers

Vector 类型的 match 有 Contains ,  VectorContains 和 Equals 。 VectorContains 判断某个 vector 内部是否有某个元素， Contains 判断某个 vector 是否包含另外一个 vector 。

下面是来自 selftest 的一个例子：

```cpp
TEST_CASE("Vector matchers", "[matchers][vector]") {
    using Catch::Matchers::VectorContains;
    using Catch::Matchers::Contains;
    using Catch::Matchers::UnorderedEquals;
    using Catch::Matchers::Equals;
    std::vector<int> v;
    v.push_back(1);
    v.push_back(2);
    v.push_back(3);
    std::vector<int> v2;
    v2.push_back(1);
    v2.push_back(2);

    std::vector<int> empty;

    SECTION("Contains (element)") 
    {
        CHECK_THAT(v, VectorContains(1));
        CHECK_THAT(v, VectorContains(2));
    }
    SECTION("Contains (vector)") 
    {
        CHECK_THAT(v, Contains(v2));
        v2.push_back(3); // now exactly matches
        CHECK_THAT(v, Contains(v2));

        CHECK_THAT(v, Contains(empty));
        CHECK_THAT(empty, Contains(empty));
    }
    SECTION("Contains (element), composed") 
    {
        CHECK_THAT(v, VectorContains(1) && VectorContains(2));
    }

    SECTION("Equals") 
    {

        // Same vector
        CHECK_THAT(v, Equals(v));

        CHECK_THAT(empty, Equals(empty));

        // Different vector with same elements
        v2.push_back(3);
        CHECK_THAT(v, Equals(v2));
    }
    SECTION("UnorderedEquals") 
    {
        CHECK_THAT(v, UnorderedEquals(v));
        CHECK_THAT(empty, UnorderedEquals(empty));

        auto permuted = v;
        std::next_permutation(begin(permuted), end(permuted));
        REQUIRE_THAT(permuted, UnorderedEquals(v));

        std::reverse(begin(permuted), end(permuted));
        REQUIRE_THAT(permuted, UnorderedEquals(v));
    }
}
```



#### Floating point matchers

浮点数类型的 matchers 有两种： WithinULP 和 WithinAbs ， WithinAbs 比较两个浮点数是差的绝对值是否小于某个值； WithinULP 做 ULP 类型的检查。

下面是一个例子：

```cpp
TEST_CASE("Floating point matchers: float", "[matchers][ULP]") 
{
    using Catch::Matchers::WithinAbs;
    using Catch::Matchers::WithinULP;
    SECTION("Margin") 
    {
        REQUIRE_THAT(1.f, WithinAbs(1.f, 0));
        REQUIRE_THAT(0.f, WithinAbs(1.f, 1));

        REQUIRE_THAT(0.f, !WithinAbs(1.f, 0.99f));
        REQUIRE_THAT(0.f, !WithinAbs(1.f, 0.99f));

        REQUIRE_THAT(0.f, WithinAbs(-0.f, 0));
        REQUIRE_THAT(NAN, !WithinAbs(NAN, 0));

        REQUIRE_THAT(11.f, !WithinAbs(10.f, 0.5f));
        REQUIRE_THAT(10.f, !WithinAbs(11.f, 0.5f));
        REQUIRE_THAT(-10.f, WithinAbs(-10.f, 0.5f));
        REQUIRE_THAT(-10.f, WithinAbs(-9.6f, 0.5f));
    }
    SECTION("ULPs") 
    {
        REQUIRE_THAT(1.f, WithinULP(1.f, 0));
        REQUIRE_THAT(1.1f, !WithinULP(1.f, 0));
            REQUIRE_THAT(1.f, WithinULP(1.f, 0));
        REQUIRE_THAT(-0.f, WithinULP(0.f, 0));

        REQUIRE_THAT(NAN, !WithinULP(NAN, 123));
    }
    SECTION("Composed") 
    {
        REQUIRE_THAT(1.f, WithinAbs(1.f, 0.5) || WithinULP(1.f, 1));
        REQUIRE_THAT(1.f, WithinAbs(2.f, 0.5) || WithinULP(1.f, 0));

        REQUIRE_THAT(NAN, !(WithinAbs(NAN, 100) || WithinULP(NAN, 123)));
    }
}
```



#### Custom matchers

通过继承 MatchBase , 可以自定义用户的 matcher ,  Catch:: MatcherBase 这里的 T 是用户想要做 match 的数据类型。同时还需要重写 match 和 describe 两个函数。

如下是一个判断某个数是否在某个范围类的自定义 matcher ：

```cpp
// The matcher class
class IntRange : public Catch::MatcherBase<int> 
{
    int m_begin, m_end;
public:
    IntRange( int begin, int end ) : m_begin( begin ), m_end( end ) {}
    // Performs the test for this matcher
    virtual bool match( int const& i ) const override 
    {
        return i >= m_begin && i <= m_end;
    }

    // Produces a string describing what this matcher does. It should
    // include any provided data (the begin/ end in this case) and
    // be written as if it were stating a fact (in the output it will be
    // preceded by the value under test).
    virtual std::string describe() const 
    {
        std::ostringstream ss;
        ss << "is between " << m_begin << " and " << m_end;
        return ss.str();
    }
};

// The builder function
inline IntRange IsBetween( int begin, int end ) 
{
    return IntRange( begin, end );
}

// ...

// Usage
TEST_CASE("Integers are within a range")
{
    CHECK_THAT( 3, IsBetween( 1, 10 ) );
    CHECK_THAT( 100, IsBetween( 1, 10 ) );
}
```



## googletest

Googletest 是谷歌开源测试框架，在 GitHub 上下载最新版本

```embed
title: "GitHub - google/googletest: GoogleTest - Google Testing and Mocking Framework"
image: "https://opengraph.githubassets.com/ceaae46aee6e847814c0c4bf18d6d8f41924b80247b0553c01f993cdd1ab82ef/google/googletest"
description: "GoogleTest - Google Testing and Mocking Framework. Contribute to google/googletest development by creating an accouton GitHub."
url: "https://github.com/google/googletest"
```

如果要在 MinGW 下进行测试，执行

```shell
$ cmake -DCMAKE_CXX_STANDARD=17 -G "MinGW Makefiles" ..
$ make
```

注意确保 MinGW 使用 sjlj 版本，否则会有 thread 报错。



如果在 MSVC 下进行测试，可以生成时勾选 `BUILD_SHARED_LIBS` 来生成动态库

```shell
cmake -DBUILD_SHARED_LIBS=ON
```

指定安装目录后，生成 `INSTALL` 项目。



### 简单样例

#### Gtest

考虑一个单文件项目

```cpp
#include <gtest/gtest.h>

TEST(testCase, testHello)
{
}
```

配置 cmake

```cmake
cmake_minimum_required(VERSION 3.10)

project(main)

add_executable(main main.cpp)

set(GTest_DIR "E:/Desktop/github/googletest/install/lib/cmake/GTest")
find_package(GTest REQUIRED)

target_link_libraries(main GTest::GTest)
target_link_libraries(main GTest::gtest_main)
```

分别链接 `GTest` 和内置的 `gtest_main` 模块。



#### Gmock

Gmock 用于模拟接口调用。例如

```cpp
// User.h

#pragma once

class User
{
public:
    virtual int getAge() = 0;
};
```

可以创建模拟方法

```cpp
// MockUser.h

#pragma once

#include "User.h"
#include <gmock/gmock.h>

class MockUser : public User
{
public:
    /**
     * @file MockUser.h
     * @author xingyifan
     * @date 2023-09-24 22:13
     * 
     * @description: 生成模拟方法
     */
    MOCK_METHOD(int, getAge, (), (override));
};
```

然后在 `test1.cpp` 中调用

```cpp
#include <iostream>
#include <gtest/gtest.h>

#include "MockUser.h"

TEST(testCase, testUser)
{
    MockUser user;
    EXPECT_CALL(user, getAge)
        .Times(::testing::AtLeast(3))
        .WillOnce(::testing::Return(200))
        .WillOnce(::testing::Return(300))
        .WillRepeatedly(::testing::Return(500));

    std::cout<< user.getAge() << std::endl;
    std::cout<< user.getAge() << std::endl;
    std::cout<< user.getAge() << std::endl;
}

int main(int argc, char **argv)
{
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

这里 Times 指定一个方法至少要调用 3 次；WillOnce 表示调用一次的返回值；WillRepeatedly 表示之后调用的返回值。如果后面调用 getAge () 方法少于 3 次就会报错。



### 基本概念

GTest 测试需要进行断言，它们检查条件是否为真。断言的结果可以是

* 成功 success
* 非致命失败 nonfatal failure
* 致命失败 fatal failure

如果发生致命故障，则中止当前功能；否则程序将正常继续。测试套件 (Test Suite) 包含一个或多个测试。应该将测试分组到反映测试代码结构的测试套件中。当测试套件中的多个测试需要共享公共对象和子例程时，可以将它们放入测试装置 (Test Fixture) 类中。一个测试程序可以包含多个测试套件。



#### 断言

GTest 提供了一组断言进行测试，可以在 [Assertions Reference](https://google.github.io/googletest/reference/assertions.html) 中查阅更多断言的形式。断言有两类

* ASSERT_* 断言在失败时产生致命错误，同时放弃执行函数
* EXPECT_* 断言在失败时产生非致命错误，将会继续执行

由于 ASSERT_* 失败后会立即返回，因此可能产生内存泄漏，需要注意。



断言失败会输出预定义的信息。也可以自定义错误信息，通过 `<<` 操作符指定，例如

```cpp
ASSERT_EQ(x.size(), y.size()) << "Vectors x and y are of unequal length";

for (int i = 0; i < x.size(); ++i)
	EXPECT_EQ(x[i], y[i]) << "Vectors x and y differ at index " << i;
```

其中 `EQ` 比较两个值是否相等。任何可以输入到 `ostream` 中的内容都可以作为信息。



#### 测试

测试样例使用格式

```cpp
TEST(TestSuiteName, TestName) {
  ... test body ...
}
```

分别指定测试套件名和测试名，应该符合 C++ 命名规范，并且不能有下划线。测试套件名用来对测试样例进行分组，因此具有相似逻辑的测试应该具有相同的测试套件名。



#### 测试装置

当我们需要对**同一组数据**进行多种不同操作时，就可以使用测试装置。例如给定一个类

```cpp
#pragma once

class Array
{
public:
	Array() : _arr(new int[5])
	{
		for (int i = 0; i < 5; i++)
			_arr[i] = 0;
	}
	~Array() { delete[]_arr; }

	int& operator[](int i) { return _arr[i]; }
	int operator[](int i) const { return _arr[i]; }

protected:
	int* _arr;
};

```

然后定义一个测试类，它需要继承 `::testing::Test` 类

```cpp
#pragma once

#include <gtest/gtest.h>
#include "Array.h"

class ArrayTest : public ::testing::Test
{
protected:
    // 初始化函数，其中初始化要测试的数据
	void SetUp() override
	{
		a0[0] = 10;
		a1[1] = 12;
		a2[2] = 22;
	}
	
    // 析构函数，用来销毁测试数据，这里不需要
	//void TearDown() override {}
	
    // 要测试的数据
	Array a0;
	Array a1;
	Array a2;
};
```

然后通过 `TEST_F` 构建测试

```cpp
#include <gtest/gtest.h>
#include "ArrayTest.h"

TEST_F(ArrayTest, Test1)
{
	EXPECT_EQ(a0[0], 10);
}

TEST_F(ArrayTest, Test2)
{
	EXPECT_NE(a1[1], 10);
}

TEST_F(ArrayTest, Test3)
{
	EXPECT_EQ(a2[2], 22);
}

```

注意**测试套件名就是测试类名**。每个 `TEST_F` 都会通过 `SetUp` 创建测试数据，完成测试后通过 `TearDown` 销毁数据。不同的 `TEST_F` 之间不会相互影响。



#### 跳过测试

使用 `GTEST_SKIP()` 跳过部分测试，同时指定跳过原因

```c++
TEST_F(ArrayTest, Test3)
{
	GTEST_SKIP() << "跳过原因" << std::endl;
	EXPECT_EQ(a2[2], 22);
}
```



#### Main 函数

一般来说不需要自定义 main 函数，而是链接预定义的函数

```cmake
target_link_libraries(mytest GTest::gtest_main)
```

当然，也可以选择自己写程序入口

```cpp
int main(int argc, char **argv)
{
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```



#### Quick Start

首先创建 `test.cpp` 文件

```cpp
#include <gtest/gtest.h>

// Demonstrate some basic assertions.
TEST(HelloTest, BasicAssertions) {
  // Expect two strings not to be equal.
  EXPECT_STRNE("hello", "world");
  // Expect equality.
  EXPECT_EQ(7 * 6 + 1, 42);
}
```

然后设置 cmake 文件：通过 FetchContent 直接从 GitHub 上获得 googletest，就不需要在本地进行引用和链接

```cmake
cmake_minimum_required(VERSION 3.14)
project(MAIN)

# GoogleTest requires at least C++14
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 获取 GoogleTest
include(FetchContent)
FetchContent_Declare(
    googletest
    URL https://github.com/google/googletest/archive/03597a01ee50ed33e9dfd640b249b4be3799d395.zip
)

# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

# 启用测试
enable_testing()

# 编译测试源码
add_executable(mytest test.cpp)

# 连接 GTest 预定义的 main 文件
target_link_libraries(mytest GTest::gtest_main)

include(GoogleTest)

# 创建 gtest
gtest_discover_tests(mytest)
```

注意目标名称不能是 test，否则会出错。使用 GTest 预定义的 main 函数，就不需要自定义。通过 gtest 函数直接创建测试目标。



### 并行测试

GTest-parallel 是一个将 Google-Test 中的每个测试并行化执行的脚本。

```embed
title: "GitHub - google/gtest-parallel: Run Google Test suites in parallel."
image: "https://opengraph.githubassets.com/374795d0cd91187bdac2b408c7ac90829645ecfed32e136ab0ca79bf2824c1d8/google/gtest-parallel"
description: "Run Google Test suites in parallel. Contribute to google/gtest-parallel development by creating an accouton GitHub."
url: "https://github.com/google/gtest-parallel"
```

使用方式如下

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



## googlebenchmark

Googlebenchmark 是谷歌开源性能测试框架，在 GitHub 上下载最新版本

```embed
title: "GitHub - google/benchmark: A microbenchmark support library"
image: "https://opengraph.githubassets.com/137d2f80a877f91af4b20006aa22a4ec2f7a5c70d96c64ed74ab38a6cdaf2b80/google/benchmark"
description: "A microbenchmark support library. Contribute to google/benchmark development by creating an accouton GitHub."
url: "https://github.com/google/benchmark"
```

在 MSVC 下进行测试，生成时勾选 `BUILD_SHARED_LIBS` 来生成动态库

```shell
cmake -DBUILD_SHARED_LIBS=ON
```

指定安装目录后，生成 `INSTALL` 项目。



### 简单示例

通过对状态迭代器循环来大量调用测试函数得到平均用时。例如

```cpp
#include <benchmark/benchmark.h>

// 另一个 benchmark
static void BM_StringCreation(benchmark::State& state) {
  for (auto _ : state)
    std::string empty_string;
}
// 注册函数为一个 benchmark
BENCHMARK(BM_StringCreation);

// 另一个 benchmark
static void BM_StringCopy(benchmark::State& state) {
  std::string x = "hello";
  for (auto _ : state)
    std::string copy(x);
}
// 注册函数为一个 benchmark
BENCHMARK(BM_StringCopy);

// 入口函数
BENCHMARK_MAIN();
```

配置 cmake

```cmake
cmake_minimum_required(VERSION 3.10)

project(main)

add_executable(main main.cpp)

set(benchmark_DIR "D:/lib/googlebenchmark-1.9.0/Debug/lib/cmake/benchmark")
find_package(benchmark REQUIRED)

target_link_libraries(main benchmark::benchmark)
```



### 初始化

在某些情况下，我们可能需要在测试之前进行统一的初始化，这时候就需要自定义测试类，在类中提供初始化和资源释放函数。例如

```cpp
class Template_Sample : public benchmark::Fixture {
	int level = 0;

protected:
	void SetUp(benchmark::State& state) override { 
		// 初始化过程
	}

	void TearDown(benchmark::State& state) override { 
		// 释放资源
	}
};

// 定义测试流程
BENCHMARK_DEFINE_F(Template_Sample, test_code)(benchmark::State& state) {
	for (auto _ : state) {
		// 测试代码
	}
}

// 注册函数，设置返回的时间单位
BENCHMARK_REGISTER_F(Template_Sample, test_code) -> Unit(benchmark::kMillisecond);
```



## Json
### jsoncpp

Jsoncpp 是一个较常用的开源库，我们分别介绍其在 Linux 和 Window 下的安装。

```embed
title: "GitHub - open-source-parsers/jsoncpp: A C++ library for interacting with JSON."
image: "https://opengraph.githubassets.com/00c023c754c37a798c6bf7b8af1a2ce71b530fb15ce6fea576ee562abb8ac86a/open-source-parsers/jsoncpp"
description: "A C++ library for interacting with JSON. Contribute to open-source-parsers/jsoncpp development by creating an accouton GitHub."
url: "https://github.com/open-source-parsers/jsoncpp"
```

在 Linux 下只需要在终端输入

```cpp
sudo apt-get install libjsoncpp-dev
```

就可以直接安装 jsoncpp 解析库。



在 Windows 下，需要先下载源代码进行编译。下载好源代码之后，使用 CMake 进行编译，先设置好源代码位置和输出路径，然后点击左下角 Configure 按钮，一般不需要改动选项，继续操作得到下面的界面

![[基本 Cpp.assets/image-20220313235413635.png]]

等待配置完成，得到下面界面后点击 Generate 生成项目

![[基本 Cpp.assets/image-20220313235630620.png]]

最后得到文件夹中的 VS 项目文件

![[基本 Cpp.assets/image-20220313235801685.png]]

打开 jsoncpp. Json 进入项目，在左侧项目列表中选择 json基本 Cpp 项目

![[基本 Cpp.assets/image-20220314000035842.png]]

右击项目选择 “生成” ，就可以看到输出结果显示生成成功。



观察 .lib 和 .dll 文件的路径，将它们提取出来

![[基本 Cpp.assets/image-20220314000348412.png]]

这就得到了想要的库文件。



### JSON for modern C++

这是一个用于解析 json 文件的开源库，是一个德国大牛 nlohmann 写的，该版本的 json 有以下特点：

* 直观的语法
* 整个代码由一个头文件组成 json. Hpp ，没有子项目，没有依赖关系，没有复杂的构建系统，使用起来非常方便
* 使用 c++11 标准编写
* 使用 json 像使用 STL 容器一样
* STL 和 json 容器之间可以相互转换
* 只支持 utf-8 编码，因此中文可能乱码

进入 single_only 的文件夹，找到 .hpp 文件下载即可。

```embed
title: "GitHub - nlohmann/json: JSON for Modern C++"
image: "https://repository-images.githubusercontent.com/11171548/a403c600-b5f0-11e9-8db8-2d5e6ec2ac98"
description: "JSON for Modern C++. Contribute to nlohmann/json development by creating an accouton GitHub."
url: "https://github.com/nlohmann/json"
```

引用头文件即可直接使用

```cpp
#include "json.hpp"
using json = nlohmann::json;
```



#### 创建对象

通过输入输出流直接写入和读取 json 数据

```cpp
int main()
{
    json j;
    cin >> j;
    cout<< j;

    return 0;
}
```

输入 json 数据

```json
{
    "Domain": "Omega",
    "BoundaryType": "Dirichlet",
    "BoundaryConditions": {
        "left": [
            1,
            0
        ],
        "top": [
            1,
            0
        ],
        "right": [
            1,
            0
        ],
        "bottom": [
            1,
            0
        ]
    }
}
```

输出序列化结果

```shell
{"BoundaryConditions":{"bottom":[1,0],"left":[1,0],"right":[1,0],"top":[1,0]},"BoundaryType":"Dirichlet","Domain":"Omega"}
```



我们可以直接通过键值对的方式赋值

```cpp
int main()
{
    json j;
    j["Domain"] = "Omega";
    j["value"] = {1, "null", 3};
    j["BoundaryConditions"]["top"] = 5;

    cout<< j << endl;

    return 0;
}
```

输出序列化结果

```json
{"BoundaryConditions":{"top":5},"Domain":"Omega","value":[1,"null",3]}
```



#### 数组

一般我们不需要说明参数的类型，但有时如果需要明确一些边缘情况，就可以特别说明

```cpp
json aa = json::array();   // 创建空数组
```



数组可以直接通过下标访问，并且对于嵌套数组，可以用多重下标访问

```cpp
int main()
{
    json j;
    j["array"] = {{1, {2, 3}}, "x", "y"};

    cout<< j << endl;
    cout<< j["array"][0][0] << endl;
    cout<< j["array"][0][1] << endl;
    cout<< j["array"][0][1][0] << endl;
    cout<< j["array"][1] << endl;
    cout<< j["array"][2] << endl;

    return 0;
}
```

Json 结构

```json
{"array":[[1,[2,3]],"x","y"]}
```



可以通过 size获得数组长度，实现对对象的遍历

```cpp
for (int i = 0; i < (int)j.size(); i++)
{
    cout<< j[i] << endl;
}
```



#### 对象

```cpp
json oo = json({});        // 创建空对象
json oo1 = json::object(); // 创建空对象
```



对象通过键值来访问，上面已经使用 array 键值进行了操作

```cpp
int main()
{
    json j;
    j["array"] = {{{"a", 3}}, "x", "y"};

    cout<< j << endl;
    cout<< j["array"][0]["a"] << endl;
    cout<< j["array"][1] << endl;
    cout<< j["array"][2] << endl;

    return 0;
}

```

Json 结构

```json
{"array":[{"a":3},"x","y"]}
```

注意上面没有用键值对来生成对象元素，而是**通过双重 `{}` 来表示对应关系**

```json
{{"a",3}}
```

这表示 a 键对应值 3 ，区别于单层 `{}` 表示数组。



任何一个对象都可以对一个 json 变量赋值

```cpp
int main()
{
    json j = {{"array", {{"a", 2}}}};
    cout<< j << endl;
    json j1 = j["array"];
    cout<< j1 << endl;

    return 0;
}
```

因此我们可以取出一个 json 中的对象赋值给另一个 json 变量。



当对象数量过多时，可以通过迭代器进行遍历获取，例如

```cpp
for (nlohmann::json::iterator it = m_data.begin(); it != m_data.end(); ++it)
{
    // 获得 key 字符串
    std::string id = it.key();
    // it.value() 获得数据
}
```

需要注意的是，不知道为什么，json 对象名为 type 时，不能通过迭代器读取。例如

```json
{
    "type": "A",
    "val": "B"
}
```

迭代器会从 val 开始读取，无法获得 type 名称。



#### 类型检测

有时我们需要确认 json 保存的数据的类型，可以使用

```cpp
is_null();
is_array();
is_object();
```

等一系列方法进行判断。尤其我们需要确认访问的内容是不是存在，因此 `is_null` 较为常用。



#### 序列化

输出 json 数据时需要进行序列化，将字符串进行输出，得到标准的 json 格式。一般有两种输出方式：

* 通过标准输出格式

```cpp
cout<< setw (4) << j << endl;
```

这里 setw 算子被重载，用于设置缩进宽度。



* 使用 dump 函数

```cpp
string s = j.dump ();
cout<< j.dump (4) << endl;
```

Dump 函数返回序列化结果，得到字符串格式；如果提供缩进参数，就返回对应的 json 格式字符串。



#### 反序列化

获取 json 数据需要对数据流进行反序列化，有 4 种形式：

* 通过标准输入

```cpp
Cin >> j;
```



* 通过 `_json` 后缀

```cpp
Json j = "{\"name\":\"Li Hua\", \"value\": 10}"_json;
cout<< j << endl;
```

通过 `_json` 后缀来解析字符串进行储存，注意后缀必须紧接字符串。



* 使用 `json::parse()` 函数

```cpp
Json j = json::parse ("{\"name\":\"Li Hua\", \"value\": 10}");
cout<< j << endl;
```



* 迭代器

可以从迭代器范围读取 json; 也就是说，可以从其**内容存储为连续字节序列**的迭代器访问的任何容器，例如 `std::vector` 访问

```cpp
vector<uint8_t> v = {'a', 'b', 'c', 'd'};
json j = json::parse(v.begin (), v.end ());
```

这种做法似乎存在一些问题，暂时不用。



#### 字符串

我们可以直接将对象对应的字符串转换为 `std::string` 格式，例如对于 json 格式

```cpp
{
    "name": "abc"
}
```

我们可以用

```cpp
Json j;
fstream fp ("test. Txt", ios::in);
fp >> j;
std::string str = json["name"];
```

转换为标准字符串。也可以利用序列化 dump 获得字符串变量

```cpp
std::string str = j["name"].dump();
Str = str.Substr (1, str.size()-2);
```

得到的结果有双引号，因此取子字符串即可。



反过来只要字符串保存的格式符合 json 格式，也可以把字符串转换为 json 数据保存

```cpp
J["test"] = nlohmann::json::parse (text);
```

关于如何检查字符串格式是否正确，可以使用开源库 JSON_checker，它可以在 utf-8 编码下进行格式检查。



#### 文件读取

只需要通过标准的文件输入输出流就可以直接读取和写入 json 文件

```cpp
int main ()
{
    fstream fp ("inputs.json", ios::in);
    Json j;
    fp >> j;
    cout<< j << endl;

    return 0;
}
```

 

### JSON_checker

[JSON_checker](http://www.json.org/JSON_checker/) 是一个 Pushdown 自动机，它可以非常快速地确定 JSON 文本在语法上是否正确。它可以用于过滤系统的输入，或者验证系统的输出在语法上是否正确。它可以用于生成非常快速的 JSON 解析器。



一个标准的检查 json 格式的函数如下

```cpp
Bool checkValid (std::string json)
{
    // 首先清除不需要的字符，中文、tab、换行、空格
    std::string res = "";
    for (int i = 0; i < int (json.size()); i++)
    {
        // 如果字符高位为 1 则有中文字符
        If (! (json[i] & 0 x 80) && '	' != json[i] && '\n' != json[i] && ' ' != json[i])
            Res += json[i];
    }

    // 然后检查 json 格式
    JSON_checker jc = new_JSON_checker (20);
    for (int i = 0; i < int (res.size()); i++)
    {
        If (! JSON_checker_char (jc, res.at(i)))
        {
            QDebug () << "JSON_checker_char: syntax error";
            return false;
        }
    }
    If (! JSON_checker_done (jc))
    {
        QDebug () << "JSON_checker_char: syntax error";
        return false;
    }
    return true;
}
```




## protobuf

这是一种 google 的数据交换格式，独立于语言，独立于平台。它是一种二进制格式，因此比 `xml, json` 进行数据交换快。它可以用于分布式应用之间的数据通信或异构环境下的数据交换。



### 安装配置

在 Linux 下安装 protobuf，下载 `3.21.12` 版本，因为之后的版本安装会较为复杂。

```embed
title: "Releases · protocolbuffers/protobuf"
image: "https://opengraph.githubassets.com/7a47225b7868efb0dc62d2b9fe0f36ca9b8a6155559ee9e3898f76a9d6341fca/protocolbuffers/protobuf"
description: "Protocol Buffers - Google’s data interchange format - protocolbuffers/protobuf"
url: "https://github.com/protocolbuffers/protobuf/releases"
```

下载 `cpp` 版本的源码后，依次执行

```cpp
tar -zxvf protobuf-cpp-3.21.12.tar.gz
cd protobuf-cpp-3.21.12
mkdir build
cd build
../configure
make
sudo make install
```

完成后执行

```shell
protoc --version
```

将会报错提示找不到动态库。首先找到动态库

```shell
sudo find /usr/local/ -name libprotoc.so
sudo emacs /etc/ld.so.conf
```

修改配置文件添加 `/usr/local/lib/` 路径，执行配置

```shell
sudo ldconfig
```

现在可以正常执行。



### 使用流程

给出需要序列化的数据格式

```cpp
struct Person
{
	int id;
	string name;
	string sex;
	int age;
};
```

在 `.proto` 文件中转换为对应的格式

```protobuf
// 指定版本
syntax = "proto3";

message Person
{
	int32 id = 1;
	bytes name = 2;
	bytes sex = 3;
	int32 age = 4;
}
```

然后使用 `protoc` 命令将 `.proto` 文件转换为 C++ 文件

```shell
protoc ./test.proto --cpp_out=./
```

就得到 `test.pb.h` 和 `test.pb.cc` 两个文件，其中实现了 `Person` 类，并指定了获得/修改属性的方法。



观察头文件，可以看到属性枚举

```cpp
enum : int {
	kNameFieldNumber = 2,
	kSexFieldNumber = 3,
	kIdFieldNumber = 1,
	kAgeFieldNumber = 4,
};
```

每个枚举值就是前面指定的序号。在后面实现了相关的方法，例如

```cpp
void clear_name();
const std::string& name() const;

template <typename ArgT0 = const std::string&, typanme... Args>
void set_name(ArgT0&& arg0, ArgT... args);

std::string* mutable_name();
```

然后引用头文件并使用即可。



### 序列化

Profobuf 自动生成的消息结构提供序列化 API

```cpp
// 序列化为字符串
bool SerializeToString(std::string* output) const;
bool SerializeToArray(void* data, int size) const;

// 序列化到磁盘
bool SerializeToOstream(std::ostream* output) const;
bool SerializeToFileDescripter(int file_descriptor) const;
```

使用时形如

```cpp
Person p;
p.set_id(10);
p.set_age(32);
p.set_sex("man");
p.set_name("lucy");

std::string output;
p.SerializeToString(&output);
```



### 反序列化

同理也存在从字符串解析为消息结构的 API

```cpp
bool ParseFromString(cons std::string& data);
bool ParseFromArray(const void* data, int size);

bool ParseFromIstream(std::istream* input);
bool ParseFromFileDescriptor(int file_descriptor);
```

使用时形如

```cpp
Person p;
p.ParseFromString(input);

int id = p.id();
int age = p.age();
std::string sex = p.sex();
std::string name = p.name();
```



### 复合类型

如果指定嵌套的结构

```protobuf
syntax = "proto3";

message Address
{
	int32 num = 1;
	bytes addr = 2;
}

message Person
{
	int32 id = 1;
	bytes name = 2;
	bytes sex = 3;
	int32 age = 4;
	Address addr = 5;
}
```

其中 `Address` 和 `Person` 中的一般成员变量类似于前；对于 `Address` 类型的成员，生成

```cpp
bool clear_addr();
const ::Address& addr() cosnt;
::Address* mutable_addr();
```

使用时形如

```cpp
Address addr;
addr.set_addr("Beijing");
addr.set_num("10");

Person p;
p.mutable_addr()->set_addr("Beijing");
p.mutable_addr()->set_num("10");
```



### 数组类型

如果需要传送数组类型，使用 `repeat` 指定动态数组

```protobuf
syntax = "proto3";

message Address
{
	int32 num = 1;
	bytes addr = 2;
}

message Person
{
	int32 id = 1;
	repeat bytes name = 2;
	bytes sex = 3;
	int32 age = 4;
	Address addr = 5;
}
```

将会为 `name` 生成

```cpp
int name_size() const;

void clear_name();
const std::string& name(int index) const;
std::string* mutable_name(int index);
void set_name(int index, const std::string& value);
void add_name(const std::string& value);
```

允许修改和加长数组。



### 枚举

可以指定枚举类型，第一个枚举值必须指定为 0，否则无法编译通过

```protobuf
syntax = "proto3";

enum Color
{
	Red = 0;
	Green = 5;
	Yellow;
	Blue = 9;
}

message Address
{
	int32 num = 1;
	bytes addr = 2;
}

message Person
{
	int32 id = 1;
	repeat bytes name = 2;
	bytes sex = 3;
	int32 age = 4;
	Address addr = 5;
	Color color = 6;
}
```

将会为 `color` 生成

```cpp
void clear_color();
::Color color() const;
void set_color(::Color value);
```



### 多个 proto

将部分结构移动到其它 proto 文件中，然后进行组合

```protobuf
// Address.proto
syntax = "proto3";

message Address
{
	int32 num = 1;
	bytes addr = 2;
}

// Person.proto
syntax = "proto3";

// 导入其它 proto
import "Address.proto";

enum Color
{
	Red = 0;
	Green = 5;
	Yellow;
	Blue = 9;
}

message Person
{
	int32 id = 1;
	repeat bytes name = 2;
	bytes sex = 3;
	int32 age = 4;
	Address addr = 5;
	Color color = 6;
}
```

执行命令

```shell
protoc ./Person.proto --cpp_out=./
protoc ./Address.proto --cpp_out=./
```

将得到两个头文件和源文件。



### 命名空间

```protobuf
// Address.proto
syntax = "proto3";

// 指定命名空间
package First;

message Address
{
	int32 num = 1;
	bytes addr = 2;
}

// Person.proto
syntax = "proto3";

// 导入其它 proto
import "Address.proto";

// 指定命名空间
package Second;

enum Color
{
	Red = 0;
	Green = 5;
	Yellow;
	Blue = 9;
}

message Person
{
	int32 id = 1;
	repeat bytes name = 2;
	bytes sex = 3;
	int32 age = 4;
	First.Address addr = 5;
	Color color = 6;
}
```

在消息体中使用其它命名空间中的消息体时，只需要指定对应的命名空间即可。最终生成的代码将得到对应的 C++ 命名空间。



## cxxopts

cxxopts 是轻量级 C++ 命令行选项解析器。

```embed
title: "GitHub - jarro2783/cxxopts: Lightweight C++ command line option parser"
image: "https://opengraph.githubassets.com/5ee6c7c094bda6cfb776724f9c9286a36cc3542dd9d866727eb4e7e768226b46/jarro2783/cxxopts"
description: "Lightweight C++ command line option parser. Contribute to jarro2783/cxxopts development by creating an accouton GitHub."
url: "https://github.com/jarro2783/cxxopts"
```

它解析形如下列的命令行选项

```shell
--long
--long=argument
--long argument
-a
-ab
-abc argument
```

其中 `ab` 没有参数，`c` 需要一个参数。



### 基本用法

#### 创建选项

引入单独的头文件，创建选项对象

```cpp
cxxopts::Options options("MyProgram", "One line description of MyProgram");
```

使用 `add_option` 添加选项，如果不指定第三个选项，则默认为布尔类型

```cpp
options.add_options()
("d,debug", "Enable debugging") 												// bool 类型
("i,integer", "Int param", cxxopts::value<int>())								// int 类型
("f,file", "File name", cxxopts::value<std::string>())							// std::string 类型
("v,verbose", "Verbose output", cxxopts::value<bool>()->default_value("false")) // 布尔类型，指定默认值为 false
;
```



#### 解析选项

然后可以解析命令行参数

```cpp
auto result = options.parse(argc, argv);
```

将会获得一个 `map` 对象，从中获得不同选项的出现次数。如果出现对应的选项，则可以获得

```cpp
std::string file;
if(result.count("file")) 
	file = result["file"].as<std::string>();
```

可以设置允许无法识别的选项，这些选项将被跳过而不报错，通过 `unmathced` 获得这些选项

```cpp
options.allow_unrecognised_options();
result.unmatched()
```



#### 位置参数

可以指定隐式的参数位置，例如对于多个连续的输入参数，可以指定依次解析的参数类型。例如

```cpp
options.add_options()
  ("script", "The script file to execute", cxxopts::value<std::string>())
  ("server", "The server to execute on", cxxopts::value<std::string>())
  ("filenames", "The filename(s) to process", cxxopts::value<std::vector<std::string>>());	// 允许解析为 vector

options.parse_positional({"script", "server", "filenames"});

options.parse(argc, argv);
```

输入 `my_script.py my_server.com file1.txt file2.txt file3.txt` 将被依次解析为

```sh
my_script.py --> script
my_server.com --> server
file1.txt file2.txt file3.txt --> filenames
```

此外，`--` 之后的任何内容都将被解析为位置参数。



#### 默认值和隐式值

默认值是不指定选项时，选项的默认值；隐式是指定选项但不指定参数时，选项的默认值。

```cpp
cxxopts::value<std::string>()->default_value("value")
cxxopts::value<std::string>()->implicit_value("implicit")
```

默认值不会被计入选项的 `count` 数量。



#### 布尔值

布尔类型的选项默认隐式值为 `true`，在命令行中指定它的值不起作用。例如

```cpp
"d,debug", "Enable debugging", cxxopts::value<bool>()->default_value("true")
```

则 `-d true, -d false` 都将得到 `true`；不指定 `-d` 时也会得到 `true` 。因此一般设置

```cpp
"d,debug", "Enable debugging", cxxopts::value<bool>()->default_value("false")
```

这样指定 `-d` 时为 `true`，不指定时为 `false` 。



#### vector

当指定 `vector` 类型的参数时，默认使用 `,` 分隔

```cpp
--my_list=1,-2.1,3,4.5
```

可以在引入头文件之前定义 `CXXOPTS_VECTOR_DELIMITER` 宏来指定分隔符

```cpp
#define CXXOPTS_VECTOR_DELIMITER ':'
#include "cxxopts.hpp"
```

从而接收参数形式为

```shell
--my_list=1:-2.1:3:4.5
```



如果同一个选项被指定多次

```cpp
--use train --use bus --use ferry
```

可以通过 `vector` 类型接收

```cpp
options.add_options()
("use", "Usable means of transport", cxxopts::value<std::vector<std::string>>())
```



### 使用示例

```cpp
#include "cxxopts.hpp"

#include <iostream>

int main(int argc, char** argv)
{
    cxxopts::Options options("test", "A brief description");

    options.add_options()
        ("b,bar", "Param bar", cxxopts::value<std::string>())
        ("d,debug", "Enable debugging", cxxopts::value<bool>()->default_value("false"))
        ("f,foo", "Param foo", cxxopts::value<int>()->default_value("10"))
        ("h,help", "Print usage")
    ;

    auto result = options.parse(argc, argv);

    if (result.count("help"))
    {
      std::cout<< options.help() << std::endl;
      exit(0);
    }
    bool debug = result["debug"].as<bool>();
    std::string bar;
    if (result.count("bar"))
      bar = result["bar"].as<std::string>();
    int foo = result["foo"].as<int>();

    std::cout<< "Debug: " << debug << std::endl;
    std::cout<< "Bar: " << bar << std::endl;
    std::cout<< "Foo: " << foo << std::endl;

    return 0;
}
```



## gflags

Google 开源的命令行参数解析工具

```embed
title: "GitHub - gflags/gflags: The gflags package contains a C++ library that implements commandline flags processing. It includes built-in support for standard types such as string and the ability to define flags in the source file in which they are used. Online documentation available at:"
image: "https://opengraph.githubassets.com/a244a2e76e81a0522318b616286e883d2e8ee95a8342dff69626c82272394e7b/gflags/gflags"
description: "The gflags package contains a C++ library that implements commandline flags processing. It includes built-in support for standard types such as string and the ability to define flags in the source…"
url: "https://github.com/gflags/gflags"
```

下载源码后，使用 cmake 构建安装。

> [!note] 
> 注意源码中有一个 BUILD 文件，因此不能创建同名的 build 文件夹。



## glog

Google 开发的 C++ 日志库，设计目标是提供一种高效、功能丰富的日志记录解决方案。它以功能强大和较为完善的日志系统而闻名。

```embed
title: "GitHub - google/glog: C++ implementation of the Google logging module"
image: "https://opengraph.githubassets.com/95a94a97cb5b3de9283367e2b9a74e24191db7fd7ba8b2a0ddfdefa3b0493121/google/glog"
description: "C++ implementation of the Google logging module. Contribute to google/glog development by creating an accouton GitHub."
url: "https://github.com/google/glog"
```

下载源码，使用 cmake 构建安装，需要提供 gflags 库。



首先给出测试样例

```cpp
#include <glog/logging.h>

int main(int argc, char* argv[]) {
    google::InitGoogleLogging(argv[0]);

    LOG(INFO) << "This is an info message.";
    LOG(WARNING) << "This is a warning message.";
    LOG(ERROR) << "This is an error message.";

    google::ShutdownGoogleLogging();
    return 0;
}
```

配置 cmake 项目

```cmake
cmake_minimum_required(VERSION 3.10)
project(main)

set(gflags_DIR "D:/lib/gflags/Debug/lib/cmake/gflags")
find_package(gflags REQUIRED)

set(glog_DIR "D:/lib/glog/Debug/lib/cmake/glog")
find_package(glog REQUIRED)

add_executable(main main.cpp)

target_link_libraries(main PUBLIC glog::glog)
target_link_libraries(main PUBLIC ${GFLAGS_LIBRARIES})
```

需要将所有动态库都移动到生成目录下。如果运行报错，可以手动双击运行，查看缺少哪些动态库。



## spdlog

一个高性能、仅头文件的 C++ 日志库，设计目标是提供极简的接口和快速的日志处理能力。

```embed
title: "GitHub - gabime/spdlog: Fast C++ logging library."
image: "https://opengraph.githubassets.com/14cb5fb8aa1c4f1d430abdef9ec575b257962e9aa29e15245e0fc5ba2a41ef82/gabime/spdlog"
description: "Fast C++ logging library. Contribute to gabime/spdlog development by creating an accouton GitHub."
url: "https://github.com/gabime/spdlog"
```




首先给出测试样例

```cpp
#include <spdlog/spdlog.h>

int main() 
{
    spdlog::info("Welcome to spdlog!");
    spdlog::error("Some error message with arg: {}", 1);
    
    spdlog::warn("Easy padding in numbers like {:08d}", 12);
    spdlog::critical("Support for int: {0:d};  hex: {0:x};  oct: {0:o}; bin: {0:b}", 42);
    spdlog::info("Support for floats {:03.2f}", 1.23456);
    spdlog::info("Positional args are {1} {0}..", "too", "supported");
    spdlog::info("{:<30}", "left aligned");
    
    spdlog::set_level(spdlog::level::debug); // Set global log level to debug
    spdlog::debug("This message should be displayed..");    
    
    // change log pattern
    spdlog::set_pattern("[%H:%M:%S %z] [%n] [%^---%L---%$] [thread %t] %v");
    
    // Compile time log levels
    // Note that this does not change the current log level, it will only
    // remove (depending on SPDLOG_ACTIVE_LEVEL) the call on the release code.
    SPDLOG_TRACE("Some trace message with param {}", 42);
    SPDLOG_DEBUG("Some debug message");
}
```

配置 cmake 项目

```cmake
cmake_minimum_required(VERSION 3.10)
project(main)

add_executable(main main.cpp)

target_include_directories(main PUBLIC "D:/lib/spdlog/include")
```



如果要将日志写入文件，只需要定义基本日志对象

```cpp
#include <iostream>
#include <spdlog/sinks/basic_file_sink.h>

int main()
{
    try
    {
        auto logger = spdlog::basic_logger_mt("basic_logger", "logs/basic-log.txt");
        logger->info("Welcome to spdlog!");
    }
    catch (const spdlog::spdlog_ex &ex)
    {
        std::cout<< "Log init failed: " << ex.what() << std::endl;
    }
    return 0;
}
```



## alglib

ALGLIB 是一个跨平台的数值分析和数据处理库。它支持五种编程语言（C++、C#、Java、Python、Delphi）和多种操作系统。ALGLIB 功能包括：

- 数据分析（分类/回归、统计）
- 优化和非线性求解器
- 插值和线性/非线性最小二乘拟合
- 线性代数（直接算法、EVD/SVD）、直接和迭代线性求解器
- 快速傅里叶变换和许多其他算法

可以直接下载免费版的源码

```embed
title: "ALGLIB - C++/C#/Java numerical analysis library"
image: "https://www.alglib.net/img/site-logo-v4.png"
description: "ALGLIB is a cross-platform numerical analysis and data processing library. It supports five programming languages (C++, C#, Java, Python, Delphi) and several operating systems (Windows and POSIX, including Linux). ALGLIB features include:"
url: "https://www.alglib.net/"
```

根据 cpp 版本代码的使用手册

```embed
title: "manual.cpp.html"
image: "https://www.alglib.net/favicon.ico"
description: "ALGLIB is a cross-platform numerical analysis and data mining library. It supports several programming languages (C++, C#, Java, Delphi, VB.NET, Python) and several operating systems (Windows, *nix family)."
url: "https://www.alglib.net/translator/man/manual.cpp.html#int_main"
```



## matplotlib-cpp

这是最简单的 C++ 绘图库。它的构建类似于 Matlab 和 matplotlib 使用的绘图 API。

```embed
title: "GitHub - lava/matplotlib-cpp: Extremely simple yet powerful header-only C++ plotting library built on the popular matplotlib"
image: "https://opengraph.githubassets.com/adc78cbaf6fd11efba631f65141f57b9d994a6c6da6feb71ed7b6445904b0db4/lava/matplotlib-cpp"
description: "Extremely simple yet powerful header-only C++ plotting library built on the popular matplotlib - lava/matplotlib-cpp"
url: "https://github.com/lava/matplotlib-cpp"
```



### 简单示例

下载源码后，将单独的头文件复制到项目目录下就可以使用

```cpp
// 不使用 numpy，不然还需要提供 numpy 头文件
#define WITHOUT_NUMPY
#include "matplotlibcpp.h"

namespace plt = matplotlibcpp;

int main() {
    plt::plot({1,3,2,4});
    plt::show();
}
```

需要链接 Python 库，并且要 pip 安装 matplotlib，然后通过 cmake 链接

```cmake
cmake_minimum_required(VERSION 3.10)
project(main)

add_executable(main main.cpp)

target_include_directories(main PUBLIC "D:/Python310/include")
target_link_libraries(main PUBLIC "D:/Python310/libs/python310.lib")
```

注意在 Release 下编译运行，否则会有链接错误。



### 接口风格

使用与 matlab 中相同的绘制接口

```cpp
#define WITHOUT_NUMPY
#include "matplotlibcpp.h"
#include <cmath>

namespace plt = matplotlibcpp;

constexpr double M_PI = 3.14159265358979323846;

int main()
{
    // Prepare data.
    int n = 5000;
    std::vector<double> x(n), y(n), z(n), w(n, 2);
    for (int i = 0; i < n; ++i)
    {
        x.at(i) = i * i;
        y.at(i) = sin(2 * M_PI * i / 360.0);
        z.at(i) = log(i);
    }

    // Set the sizeof output image to 1200x780 pixels
    plt::figure_size(1200, 780);
    // Plot line from given x and y data. Color is selected automatically.
    plt::plot(x, y);
    // Plot a red dashed line from given x and y data.
    plt::plot(x, w, "r--");
    // Plot a line whose name will show up as "log(x)" in the legend.
    plt::named_plot("log(x)", x, z);
    // Set x-axis to interval [0,1000000]
    plt::xlim(0, 1000 * 1000);
    // Add graph title
    plt::title("Sample figure");
    // Enable legend.
    plt::legend();
    // Save the image (file format is determined by the extension)
    plt::save("./basic.png");
}
```

绘制结果如图所示

![[image-20240930022626385.png|800]]



### 现代风格

可以使用 C++11 风格的 lambda 表达式绘制

```cpp
#define WITHOUT_NUMPY
#include "matplotlibcpp.h"
#include <cmath>
#include <vector>

namespace plt = matplotlibcpp;

constexpr double M_PI = 3.14159265358979323846;

int main()
{
    // Prepare data.
    int n = 5000; // number of data points
    std::vector<double> x(n), y(n);
    for (int i = 0; i < n; ++i)
    {
        double t = 2 * M_PI * i / n;
        x.at(i) = 16 * sin(t) * sin(t) * sin(t);
        y.at(i) = 13 * cos(t) - 5 * cos(2 * t) - 2 * cos(3 * t) - cos(4 * t);
    }

    // plot() takes an arbitrary number of (x,y,format)-triples.
    // x must be iterable (that is, anything providing begin(x) and end(x)),
    // y must either be callable (providing operator() const) or iterable.
    plt::plot(
        x, y, "r-", x, [](double d) { return 12.5 + abs(sin(d)); }, "k-");

    // show plots
    plt::show();
}
```

绘制结果如图所示

![[image-20240930022816926.png]]



## Boost

Boost 库是为 C++ 语言标准库提供扩展的 C++ 程序库总称。

```embed
title: "Version 1.82.0"
image: "https://www.boost.org/gfx/space.png"
description: "April 14th, 2023 03:08 GMT"
url: "https://www.boost.org/users/history/version_1_82_0.html"
```

安装流程如下：

* 在 boost 库的主目录下，找到 `bootstrap.bat` 文件双击，生成 `b2.exe`
* 双击 `b2.exe` 运行，等待程序编译完成，大约需要十几分钟到两个小时

将会在主目录下生成 `bin.v2` 和 stage 两个文件夹，其中前者是中间文件，直接删除；后者存放 dll 和 lib 文件。



可以根据情况添加参数

```shell
.\b 2 install --prefix="D:\lib\Boost" variant=release
```

重要参数包括

* 使用 stage (默认) 或 install 参数，前者只生成库，后者会安装到系统中，包含头文件和 CMake 文件夹
    * --stagedir 在 stage 时指定路径
    * --prefix 在 install 时指定路径
* toolset 指定工具集，可选 minGW 和 msvc，对于 vs 2017 选择 msvc-14.1
* link 指定生成静态库或动态库，一般设为 static 静态
* runtime-link 链接时运行库，标记如何链接 C++ 运行库
* architecture 表示 CPU 架构，可设为 `x86` 或 arm
* address-model 地址长度，32 表示编译 32 位库文件，64 表示编译 64 位库文件
* threading 指定编译方式。多线程设为 multi，单线程设为 single
* variant 编译 debug/release 版本



大约十几分钟可以安装成功，然后创建 `main.cpp` 文件

```cpp
#include <boost/version.hpp>
#include <boost/config.hpp>
#include <iostream>

Int main ()
{
    std::cout<< BOOST_VERSION << std::endl;
    std::cout<< BOOST_LIB_VERSION << std::endl;
    std::cout<< BOOST_COMPILER << std::endl;
    std::cout<< BOOST_STDLIB << std::endl;
    
    return 0;
}
```

配置 cmake 文件

```cmake
Cmake_minimum_required (VERSION 3.18)
Project (MAIN)

# 设置搜索目录
Set (Boost_DIR "D:/lib/Boost/lib/cmake/Boost-1.82.0")

# 导入包
Find_package (Boost REQUIRED)
If (Boost_FOUND)
    Set (Boost_LIBRARY_DIRS "D:/lib/Boost/lib")
    Message (Boost_INCLUDE_DIRS "${Boost_INCLUDE_DIRS}")
    Message (Boost_LIBRARY_DIRS "${Boost_LIBRARY_DIRS}")
Endif (Boost_FOUND)

# 引入头文件目录
Include_directories (${Boost_INCLUDE_DIRS})

Add_executable (main main. Cpp)

# 引入库文件
Target_link_libraries (main ${Boost_LIBRARY_DIRS})
```



## dbg-macro

开源的调试库，可以方便地进行标准数据和容器的输出。

```embed
title: "GitHub - sharkdp/dbg-macro: A dbg(…) macro for C++"
image: "https://opengraph.githubassets.com/20542b25f1e8eae0f2b3fcf3811954f89e61c752f21a683e0a9e0e195f0fff41/sharkdp/dbg-macro"
description: "A dbg(…) macro for C++. Contribute to sharkdp/dbg-macro development by creating an accouton GitHub."
url: "https://github.com/sharkdp/dbg-macro"
```

支持彩色、行号、变量类型的输出

```cpp
#include <dbg.h>
#include <cstdint>
#include <vector>

// You can use "dbg (..)" in expressions:
Int 32_t factorial (int 32_t n) {
  If (dbg (n <= 1)) {
    return dbg (1);
  } else {
    return dbg (n * factorial (n - 1));
  }
}

Int 32_t main () {
  std::string message = "hello";
  Dbg (message);  // [example. Cpp: 15 (main)] message = "hello" (std::string)

  Const int 32_t a = 2;
  Const int 32_t b = dbg (3 * a) + 1;  // [example. Cpp: 18 (main)] 3 * a = 6 (int 32_t)

  std::vector<int32_t> numbers{b, 13, 42};
  dbg (numbers);  // [example. Cpp: 21 (main)] numbers = {7, 13, 42} (std::vector<int32_t>)

  Dbg ("this line is executed");  // [example. Cpp: 23 (main)] this line is executed

  Factorial (4);

  return 0;
}
```

可进行类型输出

```cpp
template <typename T>
Void my_function_template () {
  using MyDependentType = typename std::remove_reference<T>::type&&;
  dbg (dbg::type<MyDependentType>());
}
```



## cling

Cling 是一个交互式 C++ 解释器，构建在 Clang 和 LLVM 编译器基础结构之上。Cling 实现了读取-评估-打印循环 （REPL） 概念，以便利用快速应用程序开发。作为 LLVM 和 Clang 的一个小扩展，解释器重用了它们的优势，例如广受赞誉的简洁和富有表现力的编译器诊断。



需要先 clone llvm 项目进行构建

```sh
git clone https://github.com/root-project/llvm-project.git
Cd llvm-project
Git checkout cling-latest
Cd ../
git clone <cling>
Mkdir cling-build && cd cling-build
Cmake -DLLVM_EXTERNAL_PROJECTS=cling -DLLVM_EXTERNAL_CLING_SOURCE_DIR=../cling/ -DLLVM_ENABLE_PROJECTS="clang" -DLLVM_TARGETS_TO_BUILD="host; NVPTX" ../llvm-project/llvm
```

此段构建流程中 `host; NVPTX` 是项目 README 中的目标修正后的结果。由于 LLVM 更新，导致目标名称从 `nvptx` 变为 `NVPTX`，因此需要修正大小写。



由于 LLVM 项目规模极大（约十几个 G），因此无法一次编译完成，可能出现编译器堆内存不足的情况。因此按照顺序编译 Release 项目

* libcling
* clingInterpreter
* cling
* libclang

通过 cling 项目生成可执行文件；通过 libclang 项目生成 clang 库文件。将生成的库文件复制到项目 lib 目录下，然后可以使用 `cling.exe` 程序。 
