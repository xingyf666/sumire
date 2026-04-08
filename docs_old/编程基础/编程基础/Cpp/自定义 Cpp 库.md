# 自定义 Cpp 库

## 多节点树

具有多个叶节点的树结构

```cpp
template <class T, bool Destructible = true> class MN_Tree : public Tree<T, std::less<T>, void, Destructible>
{
  public:
    using Node = details::Tree_Node<T, void>;

    MN_Tree(MN_Tree &&) = delete;

    /**
     * @brief 默认构造
     *
     */
    MN_Tree() : Tree<T, std::less<T>, void, Destructible>()
    {
        // 必须清除子节点
        this->m_nil->nodes.clear();
    }

    /**
     * @brief 插入新值
     *
     * @param[in] value
     * @param[in] parent
     * @return Node*
     */
    Node *insert(const T &value, Node *parent)
    {
        // 创建新节点
        Node *node = new Node;
        node->value = value;
        node->nodes.clear();

        if (this->m_root == this->m_nil)
        {
            this->m_root = node;
            this->m_root->parent() = this->m_nil;
            return this->m_root;
        }

        if (parent == nullptr || parent == this->m_nil)
            parent = this->m_root;

        parent->nodes.push_back(node);
        node->parent() = parent;
        return node;
    }

    /**
     * @brief 节点查找
     *
     * @param[in] value
     * @return Node*
     */
    Node *find(const T &value) const override
    {
        Node *p = this->m_nil;
        auto func = [&](Node *node) {
            if (node->value == value)
                p = node;
        };

        this->preorder_walk(this->m_root, func);
        return p;
    }

    /**
     * @brief 插入值
     *
     * @param[in] value
     * @return Node*
     */
    Node *insert(const T &value) override
    {
        return insert(value, this->m_nil);
    }

    /**
     * @brief 移除值
     *
     * @param[in] value
     * @return true
     * @return false
     */
    bool remove(const T &value) override
    {
        // 找到节点
        Node *p = find(value);
        return remove(p);
    }

    /**
     * @brief 根据节点删除
     *
     * @param[in] p
     * @return true
     * @return false
     */
    bool remove(Node *p) override
    {
        if (p == nullptr || p == this->m_nil)
            return false;

        Node *node = p;
        auto parent = node->parent();
        if (node == this->m_root)
        {
            this->m_root = this->m_nil;

            // 如果还有子节点，就用第一个子节点代替根节点
            if (!node->nodes.empty())
            {
                auto front = node->nodes.front();
                node->nodes.erase(node->nodes.begin());
                this->m_root = front;
                front->parent() = this->m_nil;
                parent = this->m_root;
            }
        }

        // 注意这里 parent 可能是 node 的子节点，无法查找到 node
        auto it = std::find(parent->nodes.begin(), parent->nodes.end(), node);
        if (it != parent->nodes.end())
            parent->nodes.erase(it);

        // 修改连接关系
        for (auto child : node->nodes)
        {
            parent->nodes.push_back(child);
            child->parent() = parent;
        }
        delete node;
        return true;
    }
};

// 节点声明宏
#define DECLARE_TREE(type)                                                                                             \
                                                                                                                       \
  public:                                                                                                              \
    using type##_Tree = xi::MN_Tree<type *, false>;                                                                    \
    using type##_Node = type##_Tree::Node;                                                                             \
                                                                                                                       \
    type##_Node *node() const                                                                                          \
    {                                                                                                                  \
        return m_node.get();                                                                                           \
    }                                                                                                                  \
                                                                                                                       \
    type##_Node *parent() const                                                                                        \
    {                                                                                                                  \
        return m_node->parent();                                                                                       \
    }                                                                                                                  \
                                                                                                                       \
  protected:                                                                                                           \
    std::shared_ptr<type##_Node> m_node;                                                                               \
    std::shared_ptr<type##_Tree> m_tree;

// 初始化宏
#define INIT_TREE(type)                                                                                                \
    m_tree.reset(new type##_Tree);                                                                                     \
    m_node.reset(m_tree->insert(this));
```



## 定时器

### 多线程触发

在创建定时器时，单独创建一个线程触发计时

![](image-20240909215345152.png|800)


```cpp
class ThreadTimer {
  public:
    ThreadTimer(int repeat = -1): m_active(false), m_period(0), m_repeat(repeat) {}
    ~ThreadTimer() { stop(); }

    template <typename F, typename... Args> void start(int period, F&& func, Args&&... args) {
        if(m_active) return;
        m_period = period;
        m_func = std::bind(std::forward<F>(func), std::forward<Args>(args)...);
        m_active.store(true);
        m_thread = std::thread([&]() {
            while(m_active.load()) {
                m_func();
                if(m_repeat > 0) {
                    m_repeat--;
                    if(m_repeat == 0) {
                        m_active.store(false);
                        return;
                    }
                }
                std::this_thread::sleep_for(std::chrono::milliseconds(m_period));
            }
        });
        m_thread.detach();
    }

    void stop() { m_active.store(false); }

  private:
    std::thread m_thread;
    std::atomic<bool> m_active;
    std::function<void()> m_func;
    int m_period;  // 触发间隔 ms
    int m_repeat;  // 重复次数 -1 表示无限循环
};
```



### 时间线触发

使用多线程触发会占用其它线程，如果定时器数量过多就会出现问题。更好的方案是使用时间线触发，将定时器按照下一次触发时间排序，每当定时器触发时，就更新其触发时间，然后重新排序。

![](image-20240909215944155.png|800)

```cpp
class Timer {
    friend class TimerManager;

  public:
    explicit Timer(int repeat = -1): m_period(0), m_repeat(repeat) { m_time = now(); }
    ~Timer() = default;

    template <typename F, typename... Args> void callback(int period, F&& func, Args&&... args) {
        m_period = period;
        m_func = std::bind(std::forward<F>(func), std::forward<Args>(args)...);
    }

    void on_timer() {
        if(!m_func || m_repeat == 0) return;
        // 考虑执行时间
        m_func();
        m_time += m_period;
        if(m_repeat > 0) {
            m_repeat--;
        }
    }

  private:
    // 获得当前时间
    static int64_t now() { return std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::system_clock::now().time_since_epoch()).count(); }

  private:
    std::function<void()> m_func;
    int64_t m_time;  // 触发时间
    int m_period;    // 触发间隔 ms
    int m_repeat;    // 重复次数 -1 表示无限循环
};

class TimerManager {
  public:
    TimerManager() = default;
    ~TimerManager() = default;

    // 注册定时器
    template <typename F, typename... Args> void schedule(int period, F&& func, Args&&... args) {
        Timer timer;
        timer.callback(period, std::forward<F>(func), std::forward<Args>(args)...);
        m_timers.insert(std::make_pair(timer.m_time, timer));
    }

    template <typename F, typename... Args> void schedule(int period, int repeat, F&& func, Args&&... args) {
        Timer timer(repeat);
        timer.callback(period, std::forward<F>(func), std::forward<Args>(args)...);
        m_timers.insert(std::make_pair(timer.m_time, timer));
    }

    void update() {
        if(m_timers.empty()) return;
        int64_t now = Timer::now();
        for(auto it = m_timers.begin(); it != m_timers.end();) {
            // 如果第一个计时器还没到时间，则后面的一定还没到时间，直接返回
            if(it->first > now) return;
            it->second.on_timer();

            // 移除当前迭代器，自动获得下一个迭代器
            auto t = it->second;
            it = m_timers.erase(it);

            // 如果还没执行完，则重新插入
            if(t.m_repeat != 0) {
                auto new_it = m_timers.insert(std::make_pair(t.m_time, t));

                // 如果新插入的位置比当前位置靠前，则更新当前位置
                if(it == m_timers.end() || new_it->first < it->first) {
                    it = new_it;
                }
            }
        }
    }

  private:
    // 使用定时器时间戳排序
    std::multimap<int64_t, Timer> m_timers;
};
```



## 函数封装器

### Lambda

我们可以将函数和函数参数封装到一个结构体中，然后使用这个结构体调用。事实上，所谓 lambda 表达式，实际上就是隐含产生了一个这样的结构体。

```cpp
#include <iostream>

struct func_t {
    void operator()() const { std::cout << "Numbers: " << x << " " << y << std::endl; }

    int& x;
    int& y;
};

template <class Fn> void repeat(const Fn& fn, int n) {
    for(int i = 0; i < n; i++) {
        fn();
    }
}

int main() {
    int x = 10, y = 20;
    func_t f{x, y};

    // 两种等价写法
    repeat(f, 3);
    repeat([&x, &y]() { std::cout << "Numbers: " << x << " " << y << std::endl; }, 3);
    return 0;
}
```

如果将 lambda 表达式改为

```cpp
struct func_t {
    void operator()() const {
        y = 1;
        std::cout << "Numbers: " << x << " " << y << std::endl;
    }

    int& x;
    int y;
};

[&x, y]() {
    y = 1;
    std::cout << "Numbers: " << x << " " << y << std::endl;
}
```

显然会报错，因为等价于上面的结构体 const 函数。



### 实现函数封装

借助变长模板和特化，可以实现 `std::function` 的基本功能

```cpp
#pragma once

#include <memory>
#include <type_traits>
#include <utility>
#include <stdexcept>

// 用于捕获非函数签名，例如 Function<int> 得到 static_assert(false)
template <class FnSig> struct Function {
    static_assert(!std::is_same_v<FnSig, FnSig>, "FnSig must be a function signature");
};

// 特化版本，捕获函数签名
template <class Ret, class... Args> struct Function<Ret(Args...)> {
  private:
    struct FuncBase {
        virtual Ret call(Args...) = 0;
        virtual ~FuncBase() = default;
    };

    // 辅助类，用于保存函数对象，并提供调用接口
    template <class F> struct FuncImpl : FuncBase {
        F f;

        FuncImpl(F f): f(std::move(f)) {}

        virtual Ret call(Args... args) override { return std::invoke(f, std::forward<Args>(args)...); }
    };

    // 保存函数指针
    std::shared_ptr<FuncBase> m_base;

  public:
    // 空构造
    Function() = default;

    // 函数构造，要求传入接受 Args... 返回 Ret 的 F，并且不能传入 Function 本身
    template <class F>
        requires(std::is_invocable_r_v<Ret, F, Args...> && !std::is_same_v<std::decay_t<F>, Function>)
    Function(F f): m_base(std::make_shared<FuncImpl<F>>(std::move(f))) {}

    // 调用函数
    Ret operator()(Args... args) const {
        if(!m_base) [unlikely](unlikely)
            throw std::runtime_error("Function not initialized");
        return m_base->call(std::forward<Args>(args)...);
    }
};
```

使用如下

```cpp
#include <iostream>

#include "func.h"

int add(int a, int b) {
    return a + b;
}

int main() {
    Function<int(int, int)> f = add;
    int result = std::invoke(f, 2, 3);
    std::cout << "2 + 3 = " << result << std::endl;
    return 0;
}
```



## 智能指针

### unique_ptr

模仿标准库构造一个简单的 `unique_ptr` 类型

```cpp
#pragma once

#include <cstdio>
#include <iostream>
#include <utility>

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

	// 删除拷贝构造和赋值
    UniquePtr(const UniquePtr&) = delete;
    UniquePtr& operator=(const UniquePtr&) = delete;

    template <class U, class DU>
        requires std::convertible_to<U*, T*>
    UniquePtr(UniquePtr<U, DU>&& other) {
        m_p = std::exchange(other.m_p, nullptr);
    }

    template <class U, class DU>
        requires std::convertible_to<U*, T*>
    UniquePtr& operator=(UniquePtr<U, DU>&& other) {
        if(this != &other) [likely](likely) {
            if(m_p != nullptr) Deleter{}(m_p);

            // 交换指针，并将 other 的指针置为 nullptr
            m_p = std::exchange(other.m_p, nullptr);
        }
        return *this;
    }

    ~UniquePtr() {
        if(m_p != nullptr) Deleter{}(m_p);
    }

    T* get() { return m_p; }
	Deleter get_deleter() const { return Deleter{}; }
	T* release() { return std::exchange(m_p, nullptr); }
    void reset(T* p = nullptr) {
        if(m_p != nullptr) Deleter{}(m_p);
        m_p = p;
    }

    T& operator*() const { return *m_p; }
    T* operator->() const { return m_p; }

    operator bool() const { return m_p != nullptr; }
};

// 针对数组特化
template <class T, class Deleter> struct UniquePtr<T[], Deleter> : UniquePtr<T, Deleter> {};

// 万能引用参数
template <class T, class... Args> UniquePtr<T> make_unique(Args&&... args) {
    return UniquePtr<T>(new T(std::forward<Args>(args)...));
}
```

使用如下

```cpp
#include <iostream>
#include <vector>

#include "unique.h"

class Animal {
  public:
    virtual ~Animal() = default;
};

class Dog : public Animal {};
class Cat : public Animal {};

int main() {
    std::vector<UniquePtr<Animal>> animals;
    animals.push_back(make_unique<Dog>());
    animals.push_back(make_unique<Cat>());
    return 0;
}
```

> > 在 `vector` 容器扩容时会将原先的元素析构，然后调用拷贝构造在新的内存中创建新对象，因此这里必须删除拷贝构造和赋值，统一使用移动构造和赋值。



### shared_ptr

#### 基本实现

采用引用计数结构，实现 `shared_ptr` 类的基本功能

```cpp
#pragma once

#include <memory>
#include <utility>

template <typename T> struct SpControlBlock {
    T* m_data;
    size_t m_ref;

    explicit SpControlBlock(T* ptr) noexcept: m_data(ptr), m_ref(1) {}
    SpControlBlock(SpControlBlock&&) = delete;
};

template <typename T> struct SharedPtr {
  private:
    template <typename U> friend class SharedPtr;
    SpControlBlock<T>* m_p;

  public:
    SharedPtr(std::nullptr_t = nullptr) noexcept: m_p(new SpControlBlock<T>{nullptr}) {}
    explicit SharedPtr(T* ptr = nullptr) noexcept: m_p(new SpControlBlock<T>{ptr}) {}

    SharedPtr(const SharedPtr& other): m_p(other.m_p) { m_p->m_ref++; }

    SharedPtr(SharedPtr&& other): m_p(other.m_p) {
        other.m_p->m_data = nullptr;
        other.m_p->m_ref = 0;
    }

    ~SharedPtr() noexcept {
        // 最后一个 shared_ptr 再去删除对象与共享计数
        // ptr_ 不为空且此时共享计数减为 0 的时候，再去删除
        m_p->m_ref--;
        if(m_p->m_ref == 0) {
            if(m_p->m_data != nullptr) delete m_p->m_data;
            delete m_p;
        }
    }

    SharedPtr& operator=(const SharedPtr& other) noexcept {
        if(this != &other) [likely](likely) {
            m_p->m_ref--;
            if(m_p->m_ref == 0) {
                delete m_p->m_data;
                delete m_p;
            }
            m_p = other.m_p;
            m_p->m_ref++;
        }
        return *this;
    }

    SharedPtr& operator=(SharedPtr&& other) noexcept {
        if(this != &other) [likely](likely) {
            m_p->m_ref--;
            if(m_p->m_ref == 0) {
                delete m_p->m_data;
                delete m_p;
            }
            m_p = other.m_p;
            other.m_p->m_data = nullptr;
            other.m_p->m_ref = 0;
        }
        return *this;
    }

    size_t use_count() const { return m_p->m_ref; }

    T* get() { return m_p->m_data; }
    void swap(SharedPtr& rhs) noexcept { std::swap(m_p, rhs.m_p); }

    T& operator*() const noexcept { return *m_p->m_data; }
    T* operator->() const noexcept { return m_p->m_data; }

    operator bool() const noexcept { return m_p->m_data != nullptr; }
};

template <typename T, typename... Args> SharedPtr<T> make_shared(Args&&... args) {
    return SharedPtr<T>(new T(std::forward<Args>(args)...));
}
```



#### 线程安全

考虑到线程安全，将引用计数设为原子变量，同时将对引用计数的操作用一个原子操作封装

```cpp
#pragma once

#include <atomic>
#include <memory>
#include <utility>

template <class T> struct DefaultDeleter {
    void operator()(T* p) const { delete p; }
};

// 针对数组特化
template <class T> struct DefaultDeleter<T[]> {
    void operator()(T* p) const { delete[] p; }
};

class SpControlBlock {
  protected:
    std::atomic<size_t> m_ref;

  public:
    SpControlBlock() noexcept: m_ref(1) {}
    SpControlBlock(SpControlBlock&&) = delete;

    void incref() { m_ref.fetch_add(1, std::memory_order_relaxed); }

    void deref() {
        // 使用 fetch_sub 减少引用计数，它返回减少前的引用计数
        // 使用一个操作 fetch_sub 而不是分离的 m_ref-- 以及 m_ref.load()，避免在两个操作之间出现数据竞争
        if(m_ref.fetch_sub(1, std::memory_order_relaxed) == 1) {
            // 可以删除自己
            delete this;
        }
    }

    size_t ref() const { return m_ref.load(std::memory_order_relaxed); }

    virtual ~SpControlBlock() = default;
};

template <class T, class Deleter> class SpControlBlockImpl : public SpControlBlock {
  private:
    T* m_data;
    Deleter m_deleter;

  public:
    explicit SpControlBlockImpl(T* ptr) noexcept: m_data(ptr) {}
    explicit SpControlBlockImpl(T* ptr, Deleter deleter) noexcept: m_data(ptr), m_deleter(std::move(deleter)) {}

    ~SpControlBlockImpl() noexcept override {
        if(m_data != nullptr) Deleter{}(m_data);
    }
};

template <class T> struct SharedPtr {
  private:
    template <class U> friend class SharedPtr;
    SpControlBlock* m_bk;

  public:
    using element_type = T;

    // 由于 new 可能抛出异常，所以不能 noexcept
    SharedPtr(SpControlBlock* bk): m_bk(bk) { m_bk->incref(); }
    SharedPtr(std::nullptr_t = nullptr): m_bk(new SpControlBlockImpl<T, DefaultDeleter<T>>{nullptr}) {}

    // 显式构造
    template <class U> explicit SharedPtr(U* ptr = nullptr): m_bk(new SpControlBlockImpl<U, DefaultDeleter<U>>{ptr}) {}

    // 显式构造，带有 Deleter
    template <class U, class Deleter> explicit SharedPtr(U* ptr, Deleter deleter): m_bk(new SpControlBlockImpl<U, Deleter>{ptr, std::move(deleter)}) {}

    template <class U> SharedPtr(const SharedPtr<U>& other) noexcept: m_bk(other.m_bk) { m_bk->incref(); }
    template <class U> SharedPtr(SharedPtr<U>&& other): m_bk(other.m_bk) { other.m_bk = new SpControlBlockImpl<U, DefaultDeleter<U>>{nullptr}; }

    ~SharedPtr() noexcept {
        // 最后一个 shared_ptr 再去删除对象与共享计数
        // ptr_ 不为空且此时共享计数减为 0 的时候，再去删除
        m_bk->deref();
    }

    SharedPtr& operator=(const SharedPtr& other) noexcept {
        if(this != &other) [likely](likely) {
            m_bk->deref();
            m_bk = other.m_bk;
            m_bk->incref();
        }
        return *this;
    }

    SharedPtr& operator=(SharedPtr&& other) {
        if(this != &other) [likely](likely) {
            m_bk->deref();
            m_bk = other.m_bk;
            other.m_bk = new SpControlBlockImpl<T, DefaultDeleter<T>>{nullptr};
        }
        return *this;
    }

    size_t use_count() const { return m_bk->ref(); }
    bool unique() const { return use_count() == 1; }

    T* get() { return m_bk->m_data; }
    void swap(SharedPtr& rhs) noexcept { std::swap(m_bk, rhs.m_bk); }

    template <class U> void reset(U* ptr = nullptr) {
        m_bk->deref();
        m_bk = new SpControlBlockImpl<U, DefaultDeleter<U>>{ptr};
    }

    template <class U, class Deleter> void reset(U* ptr, Deleter deleter) {
        m_bk->deref();
        m_bk = new SpControlBlockImpl<U, Deleter>{ptr, std::move(deleter)};
    }

    T& operator*() const noexcept { return *m_bk->m_data; }
    T* operator->() const noexcept { return m_bk->m_data; }

    operator bool() const noexcept { return m_bk->m_data != nullptr; }
};

template <class T, class U> SharedPtr<T> static_pointer_cast(const SharedPtr<U>& ptr) {
    return SharedPtr<T>(static_cast<T*>(ptr.get()));
}

template <class T, class U> SharedPtr<T> dynamic_pointer_cast(const SharedPtr<U>& ptr) {
    T* p = dynamic_cast<T*>(ptr.get());
    return p == nullptr ? SharedPtr<T>() : SharedPtr<T>(p);
}

template <class T, class U> SharedPtr<T> const_pointer_cast(const SharedPtr<U>& ptr) {
    return SharedPtr<T>(const_cast<T*>(ptr.get()));
}

template <class T, class U> SharedPtr<T> reinterpret_pointer_cast(const SharedPtr<U>& ptr) {
    return SharedPtr<T>(reinterpret_cast<T*>(ptr.get()));
}

template <typename T, typename... Args> SharedPtr<T> make_shared(Args&&... args) {
    return SharedPtr<T>(new T(std::forward<Args>(args)...));
}
```

使用如下

```cpp
#include <iostream>
#include <vector>

#include "shared.h"

class Animal {
  public:
    Animal() { std::cout << "Animal constructor called" << std::endl; }
    virtual ~Animal() { std::cout << "Animal destructor called" << std::endl; }
};

class Dog : public Animal {
  public:
    Dog() { std::cout << "Dog constructor called" << std::endl; }
    virtual ~Dog() { std::cout << "Dog destructor called" << std::endl; }
};

class Cat : public Animal {
  public:
    Cat() { std::cout << "Cat constructor called" << std::endl; }
    virtual ~Cat() { std::cout << "Cat destructor called" << std::endl; }
};

int main() {
    std::vector<SharedPtr<Animal>> animals;
    animals.push_back(make_shared<Dog>());
    animals.push_back(make_shared<Cat>());
    animals.push_back(SharedPtr<Animal>(new Dog));
    animals.push_back(SharedPtr<Animal>(new Cat));

    return 0;
}
```



## 反射库

### 基本概念

反射机制允许程序在运行时借助 Reflection API 取得任何类的内部信息，并能直接操作对象的内部属性和方法。有众多场景需要反射

* RPC（远程调用）
* WEB MVC
* 对象序列化

目标是实现

- 类对象反射
- 类成员数据反射
- 类成员函数反射

类结构形如

![](image-20240809113402420.png|800)



### 类对象反射

实现类对象反射的代码如下

```cpp
#pragma once

#include <map>
#include <string>

// 作为基类
class Object
{
  public:
    Object() = default;
    virtual ~Object() = default;
};

// 构造指针
using create_object = Object *(*)();

// 单例模式
template <typename T> class Singleton
{
  public:
    static T &GetInstance()
    {
        static T instance;
        return instance;
    }

  private:
    Singleton() = default;
    ~Singleton() = default;
    Singleton(const Singleton &) = delete;
    Singleton &operator=(const Singleton &) = delete;
};

// 工厂模式
class ClassFactory
{
    friend class Singleton<ClassFactory>;

  public:
    void RegisterClass(const std::string &name, create_object creator)
    {
        m_creators[name] = creator;
    }

    Object *CreateClass(const std::string &name)
    {
        auto it = m_creators.find(name);
        if (it == m_creators.end())
            return nullptr;

        // 调用 create_object 指针指向的函数，返回创建的对象指针
        return it->second();
    }

  private:
    ClassFactory() = default;
    ~ClassFactory() = default;

  private:
    std::map<std::string, create_object> m_creators;
};

// 注册器，利用 RAII 机制，自动注册类，自动销毁
class ClassRegister
{
  public:
    // 注册器调用工厂的注册函数，不需要友元关系
    ClassRegister(const std::string &name, create_object creator)
    {
        Singleton<ClassFactory>::GetInstance().RegisterClass(name, creator);
    }
};

// 宏定义，简化注册过程
#define REGISTER_CLASS(classname)                                                                                      \
    Object *createObject##classname()                                                                                  \
    {                                                                                                                  \
        Object *p = new classname;                                                                                     \
        return p;                                                                                                      \
    }                                                                                                                  \
    ClassRegister classRegister##classname(#classname, createObject##classname)
```

使用方法为

```cpp
#include <iostream>

#include "ClassFactory.h"

class A : public Object
{
  public:
    A()
    {
        std::cout << "A created" << std::endl;
    }
};

// 注册类
REGISTER_CLASS(A);

int main()
{
	// 获得全局唯一类工厂
    ClassFactory &factory = Singleton<ClassFactory>::GetInstance();

    Object *a = factory.CreateClass("A");
    delete a;
    return 0;
}
```



### 类成员数据反射

#### 获得偏移量

对于一个类，可以借助 `0` 地址获得成员数据的偏移量。例如

```cpp
#include <iostream>

class A
{
  public:
    A(const std::string &name, int age) : m_name(name), m_age(age)
    {
    }

  public:
    std::string m_name;
    int m_age;
};

int main()
{
	// 借助 0 地址获得偏移量
    auto offset1 = (size_t) & ((A *)0)->m_name;
    auto offset2 = (size_t) & ((A *)0)->m_age;

    std::cout << "Offset of m_name: " << offset1 << std::endl;
    std::cout << "Offset of m_age: " << offset2 << std::endl;

    A t("Alice", 25);

	// 根据偏移量和类实例地址获得成员数据
    std::string name = *(std::string *)((size_t)(&t) + offset1);
    int age = *(int *)((size_t)(&t) + offset2);

    std::cout << "Name: " << name << std::endl;
    std::cout << "Age: " << age << std::endl;
    return 0;
}
```



可以将这两个操作写成宏

```cpp
#define OFFSET(type, member) ((size_t) & ((type *)0)->member)
```

事实上，在标准库中已经定义了相应的宏

```cpp
offsetof(A, m_name);
```

这种方式的问题是编译可能不通过，因为空值不能强制转换。可以改为使用真实值

```cpp
A t("Alice", 25);
auto offset1 = (size_t)&t.m_name - (size_t)&t;
```



#### 实现效果

使用第二种方式，实现类成员数据反射

```cpp
#pragma once

#include <map>
#include <string>
#include <vector>

// 基类
class Object;

// 构造指针
using create_object = Object *(*)();

// 单例模式
template <typename T> class Singleton
{
  public:
    // 用于构造单例
    static T &GetInstance()
    {
        static T instance;
        return instance;
    }

  private:
    Singleton() = default;
    ~Singleton() = default;
    Singleton(const Singleton &) = delete;
    Singleton &operator=(const Singleton &) = delete;
};

// 类字段信息，保存类字段名称、类型、偏移量
class ClassField
{
  public:
    ClassField() = default;
    ~ClassField() = default;

    ClassField(const std::string &name, const std::string &type, size_t offset)
        : m_name(name), m_type(type), m_offset(offset)
    {
    }

    const std::string &GetName()
    {
        return m_name;
    }

    const std::string &GetType()
    {
        return m_type;
    }

    size_t GetOffset()
    {
        return m_offset;
    }

  private:
    std::string m_name;
    std::string m_type;
    size_t m_offset;
};

// 工厂模式
class ClassFactory
{
    friend class Singleton<ClassFactory>;

  public:
    // 注册类
    void RegisterClass(const std::string &className, create_object creator)
    {
        m_creators[className] = creator;
    }

    // 创建类
    Object *CreateClass(const std::string &className)
    {
        auto it = m_creators.find(className);
        if (it == m_creators.end())
            return nullptr;

        // 调用 create_object 指针指向的函数，返回创建的对象指针
        return it->second();
    }

    // 注册类字段
    void RegisterField(const std::string &className, const std::string &fieldName, const std::string &fieldType,
                       size_t offset)
    {
        m_fields[className].push_back(new ClassField(fieldName, fieldType, offset));
    }

    // 获得类字段数量
    int GetFieldCount(const std::string &className)
    {
        return m_fields[className].size();
    }

    // 获得类字段
    ClassField *GetField(const std::string &className, int index)
    {
        int size = m_fields[className].size();
        if (index < 0 || index >= size)
            return nullptr;
        return m_fields[className][index];
    }

    // 获得类字段
    ClassField *GetField(const std::string &className, const std::string &fieldName)
    {
        for (auto field : m_fields[className])
        {
            if (field->GetName() == fieldName)
                return field;
        }
        return nullptr;
    }

  private:
    ClassFactory() = default;
    ~ClassFactory() = default;

  private:
    std::map<std::string, create_object> m_creators;
    std::map<std::string, std::vector<ClassField *>> m_fields;
};

// 作为基类，具有获得类信息的能力
class Object
{
  public:
    // 空构造
    Object() = default;
    virtual ~Object() = default;

    // 设置类名，在 createObject##className() 中，执行空构造，同时设置类名
    void SetClassName(const std::string &className)
    {
        m_className = className;
    }

    const std::string &GetClassName() const
    {
        return m_className;
    }

    // 借助 ClassFactory 获得类信息
    int GetFieldCount()
    {
        return Singleton<ClassFactory>::GetInstance().GetFieldCount(m_className);
    }

    ClassField *GetField(int index)
    {
        return Singleton<ClassFactory>::GetInstance().GetField(m_className, index);
    }

    ClassField *GetField(const std::string &fieldName)
    {
        return Singleton<ClassFactory>::GetInstance().GetField(m_className, fieldName);
    }

    // 通过偏移值获得/修改字段值
    template <typename T> void GetValue(const std::string &fieldName, T &value)
    {
        ClassField *field = Singleton<ClassFactory>::GetInstance().GetField(m_className, fieldName);
        if (field == nullptr)
            return;
        value = *(T *)((size_t)this + field->GetOffset());
    }

    template <typename T> void SetValue(const std::string &fieldName, const T &value)
    {
        ClassField *field = Singleton<ClassFactory>::GetInstance().GetField(m_className, fieldName);
        if (field == nullptr)
            return;
        *(T *)((size_t)this + field->GetOffset()) = value;
    }

  private:
    std::string m_className;
};

// 注册器，利用 RAII 机制，自动注册类，自动销毁
class ClassRegister
{
  public:
    // 注册器调用工厂的注册函数，不需要友元关系
    ClassRegister(const std::string &name, create_object creator)
    {
        Singleton<ClassFactory>::GetInstance().RegisterClass(name, creator);
    }

    // 注册器调用工厂的注册字段函数，不需要友元关系
    ClassRegister(const std::string &className, const std::string &fieldName, const std::string &fieldType,
                  size_t offset)
    {
        Singleton<ClassFactory>::GetInstance().RegisterField(className, fieldName, fieldType, offset);
    }
};

// 宏定义，简化注册过程
#define REGISTER_CLASS(className)                                                                                      \
    Object *createObject##className()                                                                                  \
    {                                                                                                                  \
        Object *p = new className;                                                                                     \
        p->SetClassName(#className);                                                                                   \
        return p;                                                                                                      \
    }                                                                                                                  \
    ClassRegister register##className(#className, createObject##className);

#define REIGSTER_FIELD(className, fieldName, fieldType)                                                                \
    className className##fieldName;                                                                                    \
    ClassRegister register##className##fieldName(                                                                      \
        #className, #fieldName, #fieldType, (size_t)&className##fieldName.fieldName - (size_t)&className##fieldName);
```

关键点在于

1. 将注册封装在类的构造函数中：不能在外部执行代码，只能借助类实例初始化过程来执行我们希望执行的代码，因此这种封装是必要的。
2. 使用 `createObject##className()` 包装构造过程，在其中设置类名

可以验证效果

```cpp
#include <iostream>

#include "ClassFactory.h"

class A : public Object
{
  public:
    A() : m_name("A"), m_age(10)
    {
    }

  public:
    std::string m_name;
    int m_age;
};

REGISTER_CLASS(A)
REIGSTER_FIELD(A, m_name, std::string)
REIGSTER_FIELD(A, m_age, int)

int main()
{
    ClassFactory &factory = Singleton<ClassFactory>::GetInstance();
    Object *a = factory.CreateClass("A");

    a->SetValue("m_name", std::string("Alice"));
    a->SetValue("m_age", 20);

    std::string name;
    int age;

    a->GetValue("m_name", name);
    a->GetValue("m_age", age);

    std::cout << "Name: " << name << std::endl;
    std::cout << "Age: " << age << std::endl;

    int count = a->GetFieldCount();
    for (int i = 0; i < count; i++)
    {
        std::string fieldName = a->GetField(i)->GetName();
        std::string fieldType = a->GetField(i)->GetType();
        size_t offset = a->GetField(i)->GetOffset();
        std::cout << fieldName << ", " << fieldType << ", " << offset << std::endl;
    }

    return 0;
}
```



### 类成员函数反射

首先我们需要了解类成员指针的保存和调用。每个类成员函数都存在一个隐式参数 `this`，因此将其封装为

```cpp
using a_method = std::function<void(A *)>;
```

我们可以通过它保存类成员指针

```cpp
a_method f = &A::f;
```

为了便于保存，我们通过强制转换获得它的指针，然后通过解引用使用它。最终得到

```cpp
#include <functional>
#include <iostream>

class A
{
  public:
    void f()
    {
        std::cout << "A::f()" << std::endl;
    }
};

using a_method = std::function<void(A *)>;

int main()
{
    a_method f = &A::f;
    uintptr_t func = (uintptr_t)&f;

    // A::f() 存在隐式参数 this，所以需要将 this 指针作为参数传入
    A a;
    (*(a_method *)func)(&a);

    return 0;
}
```



使用这种方法，我们将 `std::function<void(A*)>` 类型的函数保存为 `uintptr_t` 类型的指针，在使用时强制转换

```cpp
#pragma once

#include <map>
#include <string>
#include <vector>

// 基类
class Object;

// 构造指针
using create_object = Object *(*)();

// 单例模式
template <typename T> class Singleton
{
  public:
    // 用于构造单例
    static T &GetInstance()
    {
        static T instance;
        return instance;
    }

  private:
    Singleton() = default;
    ~Singleton() = default;
    Singleton(const Singleton &) = delete;
    Singleton &operator=(const Singleton &) = delete;
};

// 类字段信息，保存类字段名称、类型、偏移量
class ClassField
{
  public:
    ClassField() = default;
    ~ClassField() = default;

    ClassField(const std::string &name, const std::string &type, size_t offset)
        : m_name(name), m_type(type), m_offset(offset)
    {
    }

    const std::string &GetName()
    {
        return m_name;
    }

    const std::string &GetType()
    {
        return m_type;
    }

    size_t GetOffset()
    {
        return m_offset;
    }

  private:
    std::string m_name;
    std::string m_type;
    size_t m_offset;
};

// 类成员函数信息，保存类成员函数名称、参数类型、返回类型、偏移量
class ClassMethod
{
  public:
    ClassMethod() = default;
    ~ClassMethod() = default;

    ClassMethod(const std::string &name, uintptr_t method) : m_name(name), m_method(method)
    {
    }

    const std::string &GetName()
    {
        return m_name;
    }

    uintptr_t GetMethod()
    {
        return m_method;
    }

  private:
    std::string m_name;
    uintptr_t m_method;
};

// 工厂模式
class ClassFactory
{
    friend class Singleton<ClassFactory>;

  public:
    // 注册类
    void RegisterClass(const std::string &className, create_object creator)
    {
        m_creators[className] = creator;
    }

    // 创建类
    Object *CreateClass(const std::string &className)
    {
        auto it = m_creators.find(className);
        if (it == m_creators.end())
            return nullptr;

        // 调用 create_object 指针指向的函数，返回创建的对象指针
        return it->second();
    }

    // 注册类字段
    void RegisterField(const std::string &className, const std::string &fieldName, const std::string &fieldType,
                       size_t offset)
    {
        m_fields[className].push_back(new ClassField(fieldName, fieldType, offset));
    }

    // 获得类字段数量
    int GetFieldCount(const std::string &className)
    {
        return m_fields[className].size();
    }

    // 获得类字段
    ClassField *GetField(const std::string &className, int index)
    {
        int size = m_fields[className].size();
        if (index < 0 || index >= size)
            return nullptr;
        return m_fields[className][index];
    }

    // 获得类字段
    ClassField *GetField(const std::string &className, const std::string &fieldName)
    {
        for (auto field : m_fields[className])
        {
            if (field->GetName() == fieldName)
                return field;
        }
        return nullptr;
    }

    // 注册类方法
    void RegisterMethod(const std::string &className, const std::string &methodName, uintptr_t method)
    {
        m_methods[className].push_back(new ClassMethod(methodName, method));
    }

    // 获得类方法数量
    int GetMethodCount(const std::string &className)
    {
        return m_methods[className].size();
    }

    // 获得类方法
    ClassMethod *GetMethod(const std::string &className, int index)
    {
        int size = m_methods[className].size();
        if (index < 0 || index >= size)
            return nullptr;
        return m_methods[className][index];
    }

    // 获得类方法
    ClassMethod *GetMethod(const std::string &className, const std::string &methodName)
    {
        for (auto method : m_methods[className])
        {
            if (method->GetName() == methodName)
                return method;
        }
        return nullptr;
    }

  private:
    ClassFactory() = default;
    ~ClassFactory() = default;

  private:
    std::map<std::string, create_object> m_creators;
    std::map<std::string, std::vector<ClassField *>> m_fields;
    std::map<std::string, std::vector<ClassMethod *>> m_methods;
};

// 作为基类，具有获得类信息的能力
class Object
{
  public:
    // 空构造
    Object() = default;
    virtual ~Object() = default;

    // 设置类名，在 createObject##className() 中，执行空构造，同时设置类名
    void SetClassName(const std::string &className)
    {
        m_className = className;
    }

    const std::string &GetClassName() const
    {
        return m_className;
    }

    // 借助 ClassFactory 获得类信息
    int GetFieldCount()
    {
        return Singleton<ClassFactory>::GetInstance().GetFieldCount(m_className);
    }

    ClassField *GetField(int index)
    {
        return Singleton<ClassFactory>::GetInstance().GetField(m_className, index);
    }

    ClassField *GetField(const std::string &fieldName)
    {
        return Singleton<ClassFactory>::GetInstance().GetField(m_className, fieldName);
    }

    // 通过偏移值获得/修改字段值
    template <typename T> void GetValue(const std::string &fieldName, T &value)
    {
        ClassField *field = Singleton<ClassFactory>::GetInstance().GetField(m_className, fieldName);
        if (field == nullptr)
            return;
        value = *(T *)((size_t)this + field->GetOffset());
    }

    template <typename T> void SetValue(const std::string &fieldName, const T &value)
    {
        ClassField *field = Singleton<ClassFactory>::GetInstance().GetField(m_className, fieldName);
        if (field == nullptr)
            return;
        *(T *)((size_t)this + field->GetOffset()) = value;
    }

    int GetMethodCount()
    {
        return Singleton<ClassFactory>::GetInstance().GetMethodCount(m_className);
    }

    ClassMethod *GetMethod(int index)
    {
        return Singleton<ClassFactory>::GetInstance().GetMethod(m_className, index);
    }

    ClassMethod *GetMethod(const std::string &methodName)
    {
        return Singleton<ClassFactory>::GetInstance().GetMethod(m_className, methodName);
    }

    // 使用变长参数模板调用类方法
    void CallMethod(const std::string &methodName)
    {
        ClassMethod *method = Singleton<ClassFactory>::GetInstance().GetMethod(m_className, methodName);
        if (method == nullptr)
            return;

        // 借助 decltype(this) 获得 this 指针的类型
        (*((std::function<void(decltype(this))> *)method->GetMethod()))(this);
    }

  private:
    std::string m_className;
};

// 注册器，利用 RAII 机制，自动注册类，自动销毁
class ClassRegister
{
  public:
    // 注册器调用工厂的注册函数，不需要友元关系
    ClassRegister(const std::string &name, create_object creator)
    {
        Singleton<ClassFactory>::GetInstance().RegisterClass(name, creator);
    }

    // 注册器调用工厂的注册字段函数，不需要友元关系
    ClassRegister(const std::string &className, const std::string &fieldName, const std::string &fieldType,
                  size_t offset)
    {
        Singleton<ClassFactory>::GetInstance().RegisterField(className, fieldName, fieldType, offset);
    }

    // 注册器调用工厂的注册方法函数，不需要友元关系
    ClassRegister(const std::string &className, const std::string &methodName, uintptr_t method)
    {
        Singleton<ClassFactory>::GetInstance().RegisterMethod(className, methodName, method);
    }
};

// 宏定义，简化注册过程
#define REGISTER_CLASS(className)                                                                                      \
    Object *createObject##className()                                                                                  \
    {                                                                                                                  \
        Object *p = new className;                                                                                     \
        p->SetClassName(#className);                                                                                   \
        return p;                                                                                                      \
    }                                                                                                                  \
    ClassRegister register##className(#className, createObject##className);

#define REIGSTER_FIELD(className, fieldName, fieldType)                                                                \
    className className##fieldName;                                                                                    \
    ClassRegister register##className##fieldName(                                                                      \
        #className, #fieldName, #fieldType, (size_t)&className##fieldName.fieldName - (size_t)&className##fieldName);

#define REGISTER_METHOD(className, methodName)                                                                         \
    std::function<void(className *)> register##className##methodName##Func = &className::methodName;                   \
    ClassRegister register##className##methodName(#className, #methodName,                                             \
                                                  (uintptr_t) & (register##className##methodName##Func));
```

上述代码只能注册无显式参数、返回值为空的方法，如果需要注册更多类型方法，需要进一步完善。



## 序列化库

### 基本概念

序列化 (Serialization) 是将对象的状态信息转换为可以存储或传输的形式的过程。在序列化期间，对象将其状态写入到临时或持久性存储区。以后可以从存储区中读取或反序列化对象的状态，重新创建该对象。



#### 序列化的方式

1. 文本格式：JSON, XML
2. 二进制格式：protobuf



#### 二进制序列化

将数据结构或对象转换成二进制串，这种方式数据小、传输快、序列化/反序列化速度快。




### 基本类型

#### 类型编码

| 字段类型     | 字段长度（字节） | 底层编码                              |
| :------- | :------- | :-------------------------------- |
| `bool`   | 2        | `Type(1) + Value(1)`              |
| `char`   | 2        | `Type(1) + Value(1)`              |
| `int32`  | 5        | `Type(1) + Value(4)`              |
| `int64`  | 9        | `Type(1) + Value(8)`              |
| `float`  | 5        | `Type(1) + Value(4)`              |
| `double` | 9        | `Type(1) + Value(8)`              |
| `string` | 不定       | `Type(1) + Length(5) + Value(变长)` |



#### 类型定义

允许序列化/反序列化的类型

```cpp
enum DataType
{
    BOOL,
    CHAR,
    INT32,
    INT64,
    FLOAT,
    DOUBLE,
    STRING,
    VECTOR,
    LIST,
    MAP,
    CUSTOM
};
```




要对基本类型序列化，我们构造一个数据类，通过 `char` 向量保存数据，给出保存 `char` 类型数据的基本方法

```cpp
#pragma once

#include <cstring>
#include <iostream>
#include <string>
#include <vector>

// 数据流
class DataStream
{
  public:
    // 可序列化类型
    enum DataType
    {
        BOOL,
        CHAR,
        INT32,
        INT64,
        FLOAT,
        DOUBLE,
        STRING,
        VECTOR,
        LIST,
        MAP,
        CUSTOM
    };

    DataStream() = default;
    ~DataStream() = default;

    void Write(const char *data, int len)
    {
        // 扩容
        Reserve(len);

        int size = m_buf.size();
        m_buf.resize(size + len); // 重新设置大小，进行初始化，否则无法写入数据
        std::memcpy(&m_buf[size], data, len);
    }

  private:
    // 重新分配容量
    void Reserve(int len)
    {
        int size = m_buf.size();
        int cap = m_buf.capacity();

        if (size + len > cap)
        {
            // 扩容
            while (size + len > cap)
            {
                if (cap == 0)
                    cap = 1;
                else
                    cap *= 2;
            }
            m_buf.reserve(cap);
        }
    }

  private:
    int m_pos = 0;           // 读取位置
    std::vector<char> m_buf; // 序列化缓冲区
};
```



#### 写入

然后可以针对这些类型写入，例如对于 `bool` 类型

```cpp
void Write(bool value)
{
    char type = DataType::BOOL;
    Write((char *)&type, sizeof(char));
    Write((char *)&value, sizeof(char));
}
```

对于字符串类型

```cpp
void Write(const char *value)
{
    char type = DataType::STRING;
    Write((char *)&type, sizeof(char));

    // 写入字符串长度
    int len = strlen(value);
    Write(len);

    // 写入字符串内容
    Write(value, len);
}

void Write(const std::string &value)
{
    // 直接使用 C 字符串写入法
    Write(value.c_str());
}
```

我们通过一个 `Print()` 方法输出整个数据流

```cpp
void Print() const
{
    int size = m_buf.size();
    std::cout << "DataStream size: " << size << std::endl;

    int i = 0;
    while (i < size)
    {
        switch ((DataType)m_buf[i])
        {
        case DataType::BOOL: {
            if ((int)m_buf[++i] == 0)
                std::cout << "false";
            else
                std::cout << "true";
            std::cout << std::endl;
            ++i;
            break;
        }
        case DataType::CHAR: {
            std::cout << (char)m_buf[++i] << std::endl;
            ++i;
            break;
        }
        case DataType::INT32: {
            std::cout << *(int32_t *)&m_buf[++i] << std::endl;
            i += sizeof(int32_t);
            break;
        }
        case DataType::INT64: {
            std::cout << *(int64_t *)&m_buf[++i] << std::endl;
            i += sizeof(int64_t);
            break;
        }
        case DataType::FLOAT: {
            std::cout << *(float *)&m_buf[++i] << std::endl;
            i += sizeof(float);
            break;
        }
        case DataType::DOUBLE: {
            std::cout << *(double *)&m_buf[++i] << std::endl;
            i += sizeof(double);
            break;
        }
        case DataType::STRING: {
            if ((DataType)m_buf[++i] == DataType::INT32)
            {
                int len = *(int *)&m_buf[++i];
                i += sizeof(int);
                std::cout << std::string(&m_buf[i], len) << std::endl;
                i += len;
            }
            else
                // 字符串编码错误
                throw std::logic_error("Parse string error");
            break;
        }
        }
    }
}
```



#### 读取

读取数据

```cpp
bool Read(char &value)
{
    if (m_buf.at(m_pos) != DataType::CHAR)
        return false;
    value = m_buf.at(++m_pos);
    ++m_pos;
    return true;
}

bool Read(int32_t &value)
{
    if (m_buf.at(m_pos) != DataType::INT32)
        return false;
    value = *(int32_t *)&m_buf.at(++m_pos);
    m_pos += sizeof(int32_t);
    return true;
}
```

对于字符串

```cpp
bool Read(std::string &value)
{
    if (m_buf.at(m_pos) != DataType::STRING)
        return false;

    DataType type = (DataType)m_buf.at(++m_pos);
    if (type == DataType::INT32)
    {
        int len = *(int *)&m_buf.at(++m_pos);
        m_pos += sizeof(int);
        value = std::string(&m_buf.at(m_pos), len);
        m_pos += len;
    }
    else
        // 字符串编码错误
        throw std::logic_error("Parse string error");

    return true;
}
```



#### 流式操作

利用局部宏展开来简化代码，并利用 SFINAE 机制实现模板流式操作

```cpp
#pragma once

#include <cstring>
#include <iostream>
#include <string>
#include <vector>

// 数据流
class DataStream
{
  public:
    // 可序列化类型
    enum DataType
    {
        BOOL,
        CHAR,
        INT32,
        INT64,
        FLOAT,
        DOUBLE,
        STRING,
        VECTOR,
        LIST,
        MAP,
        CUSTOM
    };

    DataStream() = default;
    ~DataStream() = default;

    void Print() const
    {
        int size = m_buf.size();
        std::cout << "DataStream size: " << size << std::endl;

        int i = 0;
        while (i < size)
        {
            switch ((DataType)m_buf[i])
            {
            case DataType::BOOL: {
                if ((int)m_buf[++i] == 0)
                    std::cout << "false";
                else
                    std::cout << "true";
                std::cout << std::endl;
                ++i;
                break;
            }
            case DataType::CHAR: {
                std::cout << (char)m_buf[++i] << std::endl;
                ++i;
                break;
            }
            case DataType::INT32: {
                std::cout << *(int32_t *)&m_buf[++i] << std::endl;
                i += sizeof(int32_t);
                break;
            }
            case DataType::INT64: {
                std::cout << *(int64_t *)&m_buf[++i] << std::endl;
                i += sizeof(int64_t);
                break;
            }
            case DataType::FLOAT: {
                std::cout << *(float *)&m_buf[++i] << std::endl;
                i += sizeof(float);
                break;
            }
            case DataType::DOUBLE: {
                std::cout << *(double *)&m_buf[++i] << std::endl;
                i += sizeof(double);
                break;
            }
            case DataType::STRING: {
                if ((DataType)m_buf[++i] == DataType::INT32)
                {
                    int len = *(int *)&m_buf[++i];
                    i += sizeof(int);
                    std::cout << std::string(&m_buf[i], len) << std::endl;
                    i += len;
                }
                else
                    // 字符串编码错误
                    throw std::logic_error("Parse string error");
                break;
            }
            }
        }
    }

    // 流式输入
    template <typename T, typename U = std::enable_if_t<std::is_fundamental_v<T> || std::is_same_v<T, std::string> ||
                                                        std::is_convertible_v<T, const char *>>>
    DataStream &operator<<(const T &value)
    {
        Write(value);
        return *this;
    }

    // 流式输出
    template <typename T, typename U = std::enable_if_t<std::is_fundamental_v<T> || std::is_same_v<T, std::string> ||
                                                        std::is_convertible_v<T, const char *>>>
    DataStream &operator>>(T &value)
    {
        Read(value);
        return *this;
    }

    bool Read(std::string &value)
    {
        if (m_buf.at(m_pos) != DataType::STRING)
            return false;

        DataType type = (DataType)m_buf.at(++m_pos);
        if (type == DataType::INT32)
        {
            int len = *(int *)&m_buf.at(++m_pos);
            m_pos += sizeof(int);
            value = std::string(&m_buf.at(m_pos), len);
            m_pos += len;
        }
        else
            // 字符串编码错误
            throw std::logic_error("Parse string error");

        return true;
    }

#ifndef DATASTREAM_READ
#define DATASTREAM_READ(T, U, value)                                                                                   \
    bool Read(T &value)                                                                                                \
    {                                                                                                                  \
        if (m_buf.at(m_pos) != DataType::U)                                                                            \
            return false;                                                                                              \
        value = *(T *)&m_buf.at(++m_pos);                                                                              \
        m_pos += sizeof(T);                                                                                            \
        return true;                                                                                                   \
    }

    DATASTREAM_READ(bool, BOOL, value)
    DATASTREAM_READ(char, CHAR, value)
    DATASTREAM_READ(int32_t, INT32, value)
    DATASTREAM_READ(int64_t, INT64, value)
    DATASTREAM_READ(float, FLOAT, value)
    DATASTREAM_READ(double, DOUBLE, value)
#endif

    void Write(const char *data, int len)
    {
        // 扩容
        Reserve(len);

        int size = m_buf.size();
        m_buf.resize(size + len); // 重新设置大小，进行初始化，否则无法写入数据
        std::memcpy(&m_buf[size], data, len);
    }

    void Write(const char *value)
    {
        char type = DataType::STRING;
        Write((char *)&type, sizeof(char));

        // 写入字符串长度
        int len = strlen(value);
        Write(len);

        // 写入字符串内容
        Write(value, len);
    }

    void Write(const std::string &value)
    {
        // 直接使用 C 字符串写入法
        Write(value.c_str());
    }

#ifndef DATASTREAM_WRITE
#define DATASTREAM_WRITE(T, U, value)                                                                                  \
    void Write(T value)                                                                                                \
    {                                                                                                                  \
        char type = DataType::U;                                                                                       \
        Write((char *)&type, sizeof(char));                                                                            \
        Write((char *)&value, sizeof(T));                                                                              \
    }

    DATASTREAM_WRITE(bool, BOOL, value)
    DATASTREAM_WRITE(char, CHAR, value)
    DATASTREAM_WRITE(int32_t, INT32, value)
    DATASTREAM_WRITE(int64_t, INT64, value)
    DATASTREAM_WRITE(float, FLOAT, value)
    DATASTREAM_WRITE(double, DOUBLE, value)
#endif

  private:
    // 重新分配容量
    void Reserve(int len)
    {
        int size = m_buf.size();
        int cap = m_buf.capacity();

        if (size + len > cap)
        {
            // 扩容
            while (size + len > cap)
            {
                if (cap == 0)
                    cap = 1;
                else
                    cap *= 2;
            }
            m_buf.reserve(cap);
        }
    }

  private:
    int m_pos = 0;           // 读取位置
    std::vector<char> m_buf; // 序列化缓冲区
};
```



### 复合类型

#### 基本元素

对于具有基本元素的复合类型，例如 `vector, list`，可以直接推广

```cpp
// 写入
#ifndef DATASTREAM_WRITE_ITERABLE
#define DATASTREAM_WRITE_ITERABLE(C, U, value)                                                                         \
    template <typename T> void Write(const std::C<T> &value)                                                           \
    {                                                                                                                  \
        char type = DataType::U;                                                                                       \
        Write((char *)&type, sizeof(char));                                                                            \
        Write((int)value.size());                                                                                      \
        for (auto &&v : value)                                                                                         \
            Write(v);                                                                                                  \
    }

    DATASTREAM_WRITE_ITERABLE(vector, VECTOR, value)
    DATASTREAM_WRITE_ITERABLE(list, LIST, value)
    DATASTREAM_WRITE_ITERABLE(set, SET, value)
#endif

// 读取
#ifndef DATASTREAM_READ_ITERABLE
#define DATASTREAM_READ_ITERABLE(C, U, value)                                                                          \
    template <typename T> bool Read(std::C<T> &value)                                                                  \
    {                                                                                                                  \
        if (m_buf.at(m_pos) != DataType::U)                                                                            \
            return false;                                                                                              \
        m_pos++;                                                                                                       \
        int size;                                                                                                      \
        Read(size);                                                                                                    \
        value.clear();                                                                                                 \
        for (int i = 0; i < size; ++i)                                                                                 \
        {                                                                                                              \
            T v;                                                                                                       \
            Read(v);                                                                                                   \
            value.push_back(v);                                                                                        \
        }                                                                                                              \
        return true;                                                                                                   \
    }

    DATASTREAM_READ_ITERABLE(vector, VECTOR, value)
    DATASTREAM_READ_ITERABLE(list, LIST, value)
#endif
```

由于 `set` 需要使用 `insert` 插入元素，`map` 有一对元素，因此需要分开实现。

> 注意在读写复合类型时，可以利用基本类型的读写，但是**类型枚举必须强制转换为字符进行读写**，这是为了确保类型变量前面不会有这个类型变量本身的类型变量，导致干扰读写。



#### 流式操作

利用模板匹配实现不同类型的流式操作

```cpp
// 流式输入
template <typename T,
          typename U = std::enable_if_t<std::is_fundamental_v<T> || std::is_convertible_v<T, const char *>>>
DataStream &operator<<(const T &value)
{
    Write(value);
    return *this;
}

// 复合类型流式输入
template <typename T, typename U = typename T::value_type,
          typename V = std::enable_if_t<std::is_fundamental_v<U> || std::is_same_v<U, std::string> ||
                                        std::is_convertible_v<U, const char *>>>
DataStream &operator<<(const T &value)
{
    Write(value);
    return *this;
}

// map 流式输入
template <
    typename T, typename U,
    typename V = std::enable_if_t<
        (std::is_fundamental_v<T> || std::is_same_v<T, std::string> || std::is_convertible_v<T, const char *>)&&(
            std::is_fundamental_v<U> || std::is_same_v<U, std::string> || std::is_convertible_v<U, const char *>)>>
DataStream &operator<<(const std::map<T, U> &value)
{
    Write(value);
    return *this;
}

// 流式输出
template <typename T,
          typename U = std::enable_if_t<std::is_fundamental_v<T> || std::is_convertible_v<T, const char *>>>
DataStream &operator>>(T &value)
{
    Read(value);
    return *this;
}

// 复合类型流式输出
template <typename T, typename U = typename T::value_type,
          typename V = std::enable_if_t<std::is_fundamental_v<U> || std::is_same_v<U, std::string> ||
                                        std::is_convertible_v<U, const char *>>>
DataStream &operator>>(T &value)
{
    Read(value);
    return *this;
}

// map 流式输出
template <
    typename T, typename U,
    typename V = std::enable_if_t<
        (std::is_fundamental_v<T> || std::is_same_v<T, std::string> || std::is_convertible_v<T, const char *>)&&(
            std::is_fundamental_v<U> || std::is_same_v<U, std::string> || std::is_convertible_v<U, const char *>)>>
DataStream &operator>>(std::map<T, U> &value)
{
    Read(value);
    return *this;
}
```



### 自定义类型

为了实现自定义类型的序列化，创建接口类

```cpp
// 序列化类型
class Serializable
{
  public:
    virtual void Serialize(DataStream &stream) const = 0;
    virtual void Deserialize(DataStream &stream) = 0;
};
```

然后在 `DataStream` 中调用接口

```cpp
void Write(const Serializable &value)
{
    value.Serialize(*this);
}

bool Read(Serializable &value)
{
    return value.Deserialize(*this);
}
```



#### 实现接口

创建继承 `Serializable` 的类，实现接口

```cpp
class A : public Serializable
{
  public:
    A() : x(0), y("")
    {
    }

    A(int x, const std::string &y) : x(x), y(y)
    {
    }

    void Serialize(DataStream &ds) const override
    {
        // 类型必须通过 Write(const char *, int) 写入，不能通过 ds << type 写入
        char type = DataStream::DataType::CUSTOM;
        ds.Write((char *)&type, sizeof(char));
        ds << x << y;
    }

    bool Deserialize(DataStream &ds) override
    {
        // 类型也只能通过 Read(char *, int) 读取，不能通过 ds >> type 读取
        char type;
        ds.Read((char *)&type, sizeof(char));
        if (type != DataStream::DataType::CUSTOM)
            return false;
        ds >> x >> y;
        return true;
    }

    void Print() const
    {
        std::cout << "A: x=" << x << ", y=" << y << std::endl;
    }

  private:
    int x;
    std::string y;
};
```

这里我们实现了字符串直接读取

```cpp
bool Read(char *value, int len)
{
    // 注意 Read 函数不检查数据类型
    std::memcpy(value, &m_buf.at(m_pos), len);
    m_pos += len;
    return true;
}
```



#### 可变参数

要对具有可变数量的成员的类进行序列化，就需要借助可变参数模板

```cpp
// 可变读取
template <typename T, typename... Args> void WriteArgs(const T &value, const Args &...args)
{
    Write(value);
    WriteArgs(args...);
}

void WriteArgs()
{
}

// 可变写入
template <typename T, typename... Args> bool ReadArgs(T &value, Args &...args)
{
    return Read(value) && ReadArgs(args...);
}

bool ReadArgs()
{
    return true;
}
```



#### 序列化宏

为了方便起见，我们使用宏来简化定义

```cpp
#define SERIALIZE(...)                                                                                                 \
void Serialize(DataStream &stream) const override                                                                  \
{                                                                                                                  \
    char type = DataStream::CUSTOM;                                                                                \
    stream.Write((char *)&type, sizeof(char));                                                                     \
    stream.WriteArgs(__VA_ARGS__);                                                                                 \
}                                                                                                                  \
bool Deserialize(DataStream &stream) override                                                                      \
{                                                                                                                  \
    char type;                                                                                                     \
    stream.Read(&type, sizeof(char));                                                                              \
    if (type != DataStream::CUSTOM)                                                                                \
        return false;                                                                                              \
    return stream.ReadArgs(__VA_ARGS__);                                                                           \
}
```

现在可以将类简化为

```cpp
class A : public Serializable
{
  public:
    A() : x(0), y("")
    {
    }

    A(int x, const std::string &y) : x(x), y(y)
    {
    }

    void Print() const
    {
        std::cout << "A: x=" << x << ", y=" << y << std::endl;
    }

    // 使用宏定义序列化
    SERIALIZE(x, y)

  private:
    int x;
    std::string y;
};
```



最终实现代码为

```cpp
#pragma once

#include <cstring>
#include <iostream>
#include <list>
#include <map>
#include <set>
#include <string>
#include <utility>
#include <vector>

class DataStream;

// 序列化类型
class Serializable
{
  public:
    virtual void Serialize(DataStream &stream) const = 0;
    virtual bool Deserialize(DataStream &stream) = 0;
};

// 序列化宏
#define SERIALIZE(...)                                                                                                 \
    void Serialize(DataStream &stream) const override                                                                  \
    {                                                                                                                  \
        char type = DataStream::CUSTOM;                                                                                \
        stream.Write((char *)&type, sizeof(char));                                                                     \
        stream.WriteArgs(__VA_ARGS__);                                                                                 \
    }                                                                                                                  \
    bool Deserialize(DataStream &stream) override                                                                      \
    {                                                                                                                  \
        char type;                                                                                                     \
        stream.Read(&type, sizeof(char));                                                                              \
        if (type != DataStream::CUSTOM)                                                                                \
            return false;                                                                                              \
        return stream.ReadArgs(__VA_ARGS__);                                                                           \
    }

// 数据流
class DataStream
{
  public:
    // 可序列化类型
    enum DataType
    {
        BOOL,
        CHAR,
        INT32,
        INT64,
        FLOAT,
        DOUBLE,
        STRING,
        VECTOR,
        LIST,
        MAP,
        SET,
        CUSTOM
    };

    // 内存端序
    enum ByteOrder
    {
        BIG_ENDIAN,
        LITTLE_ENDIAN
    };

    DataStream() : m_pos(0)
    {
        // 检测内存端序
        CheckByteOrder();
    }

    ~DataStream() = default;

    void Print() const
    {
        int size = m_buf.size();
        std::cout << "DataStream size: " << size << std::endl;

        int i = 0;
        while (i < size)
        {
            switch ((DataType)m_buf[i])
            {
            case DataType::BOOL: {
                if ((int)m_buf[++i] == 0)
                    std::cout << "false";
                else
                    std::cout << "true";
                std::cout << std::endl;
                ++i;
                break;
            }
            case DataType::CHAR: {
                std::cout << (char)m_buf[++i] << std::endl;
                ++i;
                break;
            }
            case DataType::INT32: {
                std::cout << *(int32_t *)&m_buf[++i] << std::endl;
                i += sizeof(int32_t);
                break;
            }
            case DataType::INT64: {
                std::cout << *(int64_t *)&m_buf[++i] << std::endl;
                i += sizeof(int64_t);
                break;
            }
            case DataType::FLOAT: {
                std::cout << *(float *)&m_buf[++i] << std::endl;
                i += sizeof(float);
                break;
            }
            case DataType::DOUBLE: {
                std::cout << *(double *)&m_buf[++i] << std::endl;
                i += sizeof(double);
                break;
            }
            case DataType::STRING: {
                if ((DataType)m_buf[++i] == DataType::INT32)
                {
                    int len = *(int *)&m_buf[++i];
                    i += sizeof(int);
                    std::cout << std::string(&m_buf[i], len) << std::endl;
                    i += len;
                }
                else
                    // 字符串编码错误
                    throw std::logic_error("Parse string error");
                break;
            }
            default: {
                std::cout << "Unknown type" << std::endl;
                ++i;
                break;
            }
            }
        }
    }

    // 流式输入
    template <typename T,
              typename U = std::enable_if_t<std::is_fundamental_v<T> || std::is_convertible_v<T, const char *> ||
                                            std::is_base_of_v<Serializable, T>>>
    DataStream &operator<<(const T &value)
    {
        Write(value);
        return *this;
    }

    // 复合类型流式输入
    template <typename T, typename U = typename T::value_type,
              typename V = std::enable_if_t<std::is_fundamental_v<U> || std::is_same_v<U, std::string> ||
                                            std::is_convertible_v<U, const char *>>>
    DataStream &operator<<(const T &value)
    {
        Write(value);
        return *this;
    }

    // map 流式输入
    template <
        typename T, typename U,
        typename V = std::enable_if_t<
            (std::is_fundamental_v<T> || std::is_same_v<T, std::string> || std::is_convertible_v<T, const char *>)&&(
                std::is_fundamental_v<U> || std::is_same_v<U, std::string> || std::is_convertible_v<U, const char *>)>>
    DataStream &operator<<(const std::map<T, U> &value)
    {
        Write(value);
        return *this;
    }

    // 流式输出
    template <typename T,
              typename U = std::enable_if_t<std::is_fundamental_v<T> || std::is_convertible_v<T, const char *> ||
                                            std::is_base_of_v<Serializable, T>>>
    DataStream &operator>>(T &value)
    {
        Read(value);
        return *this;
    }

    // 复合类型流式输出
    template <typename T, typename U = typename T::value_type,
              typename V = std::enable_if_t<std::is_fundamental_v<U> || std::is_same_v<U, std::string> ||
                                            std::is_convertible_v<U, const char *>>>
    DataStream &operator>>(T &value)
    {
        Read(value);
        return *this;
    }

    // map 流式输出
    template <
        typename T, typename U,
        typename V = std::enable_if_t<
            (std::is_fundamental_v<T> || std::is_same_v<T, std::string> || std::is_convertible_v<T, const char *>)&&(
                std::is_fundamental_v<U> || std::is_same_v<U, std::string> || std::is_convertible_v<U, const char *>)>>
    DataStream &operator>>(std::map<T, U> &value)
    {
        Read(value);
        return *this;
    }

    bool Read(char *value, int len)
    {
        // 注意 Read 函数不检查数据类型
        std::memcpy(value, &m_buf.at(m_pos), len);
        m_pos += len;
        return true;
    }

    bool Read(std::string &value)
    {
        if (m_buf.at(m_pos) != DataType::STRING)
            return false;

        DataType type = (DataType)m_buf.at(++m_pos);
        if (type == DataType::INT32)
        {
            int len = *(int *)&m_buf.at(++m_pos);
            m_pos += sizeof(int);
            value = std::string(&m_buf.at(m_pos), len);
            m_pos += len;
        }
        else
            // 字符串编码错误
            throw std::logic_error("Parse string error");

        return true;
    }

    template <typename T> bool Read(std::set<T> &value)
    {
        if (m_buf.at(m_pos) != DataType::SET)
            return false;

        m_pos++;
        int size;
        Read(size);
        for (int i = 0; i < size; ++i)
        {
            T v;
            Read(v);
            value.insert(v);
        }
        return true;
    }

    template <typename T, typename U> bool Read(std::map<T, U> &value)
    {
        if (m_buf.at(m_pos) != DataType::MAP)
            return false;

        m_pos++;
        int size;
        Read(size);
        for (int i = 0; i < size; ++i)
        {
            T key;
            Read(key);
            U val;
            Read(val);
            value[key] = val;
        }
        return true;
    }

    template <typename T, typename... Args> bool ReadArgs(T &value, Args &...args)
    {
        return Read(value) && ReadArgs(args...);
    }

    bool ReadArgs()
    {
        return true;
    }

    bool Read(Serializable &value)
    {
        return value.Deserialize(*this);
    }

#ifndef DATASTREAM_READ
#define DATASTREAM_READ(T, U, value)                                                                                   \
    bool Read(T &value)                                                                                                \
    {                                                                                                                  \
        if (m_buf.at(m_pos) != DataType::U)                                                                            \
            return false;                                                                                              \
        value = *(T *)&m_buf.at(++m_pos);                                                                              \
        m_pos += sizeof(T);                                                                                            \
        return true;                                                                                                   \
    }

    DATASTREAM_READ(bool, BOOL, value)
    DATASTREAM_READ(char, CHAR, value)
    DATASTREAM_READ(int32_t, INT32, value)
    DATASTREAM_READ(int64_t, INT64, value)
    DATASTREAM_READ(float, FLOAT, value)
    DATASTREAM_READ(double, DOUBLE, value)
#endif

#ifndef DATASTREAM_READ_ITERABLE
#define DATASTREAM_READ_ITERABLE(C, U, value)                                                                          \
    template <typename T> bool Read(std::C<T> &value)                                                                  \
    {                                                                                                                  \
        if (m_buf.at(m_pos) != DataType::U)                                                                            \
            return false;                                                                                              \
        m_pos++;                                                                                                       \
        int size;                                                                                                      \
        Read(size);                                                                                                    \
        value.clear();                                                                                                 \
        for (int i = 0; i < size; ++i)                                                                                 \
        {                                                                                                              \
            T v;                                                                                                       \
            Read(v);                                                                                                   \
            value.push_back(v);                                                                                        \
        }                                                                                                              \
        return true;                                                                                                   \
    }

    DATASTREAM_READ_ITERABLE(vector, VECTOR, value)
    DATASTREAM_READ_ITERABLE(list, LIST, value)
#endif

    void Write(const char *data, int len)
    {
        // 扩容
        Reserve(len);

        int size = m_buf.size();
        m_buf.resize(size + len); // 重新设置大小，进行初始化，否则无法写入数据
        std::memcpy(&m_buf[size], data, len);
    }

    void Write(const char *value)
    {
        char type = DataType::STRING;
        Write((char *)&type, sizeof(char));

        // 写入字符串长度
        int len = strlen(value);
        Write(len);

        // 写入字符串内容
        Write(value, len);
    }

    void Write(const std::string &value)
    {
        // 直接使用 C 字符串写入法
        Write(value.c_str());
    }

    template <typename T, typename U> void Write(const std::map<T, U> &value)
    {
        char type = DataType::MAP;
        Write((char *)&type, sizeof(char));

        // 写入 map 大小
        Write((int)value.size());

        // 写入 map 内容
        for (auto &&pair : value)
        {
            Write(pair.first);
            Write(pair.second);
        }
    }

    template <typename T, typename... Args> void WriteArgs(const T &value, const Args &...args)
    {
        Write(value);
        WriteArgs(args...);
    }

    void WriteArgs()
    {
    }

    void Write(const Serializable &value)
    {
        value.Serialize(*this);
    }

#ifndef DATASTREAM_WRITE
#define DATASTREAM_WRITE(T, U, value)                                                                                  \
    void Write(T value)                                                                                                \
    {                                                                                                                  \
        char type = DataType::U;                                                                                       \
        Write((char *)&type, sizeof(char));                                                                            \
        Write((char *)&value, sizeof(T));                                                                              \
    }

    DATASTREAM_WRITE(bool, BOOL, value)
    DATASTREAM_WRITE(char, CHAR, value)
    DATASTREAM_WRITE(int32_t, INT32, value)
    DATASTREAM_WRITE(int64_t, INT64, value)
    DATASTREAM_WRITE(float, FLOAT, value)
    DATASTREAM_WRITE(double, DOUBLE, value)
#endif

#ifndef DATASTREAM_WRITE_ITERABLE
#define DATASTREAM_WRITE_ITERABLE(C, U, value)                                                                         \
    template <typename T> void Write(const std::C<T> &value)                                                           \
    {                                                                                                                  \
        char type = DataType::U;                                                                                       \
        Write((char *)&type, sizeof(char));                                                                            \
        Write((int)value.size());                                                                                      \
        for (auto &&v : value)                                                                                         \
            Write(v);                                                                                                  \
    }

    DATASTREAM_WRITE_ITERABLE(vector, VECTOR, value)
    DATASTREAM_WRITE_ITERABLE(list, LIST, value)
    DATASTREAM_WRITE_ITERABLE(set, SET, value)
#endif

  private:
    // 重新分配容量
    void Reserve(int len)
    {
        int size = m_buf.size();
        int cap = m_buf.capacity();

        if (size + len > cap)
        {
            // 扩容
            while (size + len > cap)
            {
                if (cap == 0)
                    cap = 1;
                else
                    cap *= 2;
            }
            m_buf.reserve(cap);
        }
    }

    // 检测内存端序
    ByteOrder CheckByteOrder()
    {
        int n = 0x12345678;
        char str[4];

        memcpy(str, &n, sizeof(int));

        // 大端将高位字节放在低地址，因此 12 放在 0 的位置
        if (str[0] == 0x12)
            m_byteOrder = BIG_ENDIAN;
        else
            m_byteOrder = LITTLE_ENDIAN;

        return m_byteOrder;
    }

  private:
    int m_pos;               // 读取位置
    std::vector<char> m_buf; // 序列化缓冲区
    ByteOrder m_byteOrder;   // 内存端序
};
```



### 大小端

在不同计算机中，内存的布局方式不同。例如存在大端和小端，表示字节

| 012 FF 420     | 小端存储                       | 大端存储                       |
| :----------- | :------------------------- | :------------------------- |
| 891464 F 6     | 数据低位在内存低位<br>数据高位在内存高位<br> | 数据高位在内存低位<br>数据低位在内存高位<br> |
| `[012FF420]` | F 6                         | 89                         |
| `[012FF421]` | 64                         | 14                         |
| `[012FF422]` | 14                         | 64                         |
| `[012FF423]` | 89                         | F 6                         |

使用系统宏 `__BYTE_ORDER` 检测，它在 `<endian.h>` 头文件中，但是这个宏不一定存在；更好的做法是通过字节储存顺序判断

```cpp
int n = 0x12345678;
char str[4];

memcpy(str, &n, sizeof(int));

// 大端将高位字节放在低地址，因此 12 放在 0 的位置
if (str[0] == 0x12)
    printf("BIG-ENDIAN\n");
else
    printf("LITTLE-ENDIAN\n");
```

对于大端系统，需要执行反转操作

```cpp
char *first = (char *)&value;
char *last = first + sizeof(value);
std::reverse(first, last);
```

在读写操作中将所有值反转（类型不需要反转，因为只有一个字节）。



## 对象池

### 基本概念

对象池一开始申请大内存空间，然后把大内存分配成小内存空间，当需要使用时直接分配使用，不再向系统申请内存空间，也不直接释放内存空间，使用完后放回池。


### 对象池框架

我们首先给出一个测试框架

```cpp
#pragma once

#include <stdexcept>

// 内存分配策略基类
template <typename T> class Allocator
{
  public:
    virtual T *Allocate() = 0;
    virtual void Deallocate(T *p) = 0;
};

// 堆内存分配器
template <typename T> class MallocAllocator : public Allocator<T>
{
  public:
    virtual T *Allocate() override
    {
        return reinterpret_cast<T *>(::malloc(sizeof(T)));
    }

    virtual void Deallocate(T *p) override
    {
        ::free(p);
    }
};

// 对象池
template <typename T, typename AllocatorT> class ObjectPool
{
  public:
    ObjectPool() = default;
    ~ObjectPool() = default;

    void *Allocate(size_t n)
    {
        if (sizeof(T) != n)
            throw std::bad_alloc();
        return m_allocator.Allocate();
    }

    void Deallocate(void *p)
    {
        m_allocator.Deallocate(static_cast<T *>(p));
    }

  private:
    AllocatorT m_allocator;
};
```

在需要使用对象池的类型中指定对象池

```cpp
class A
{
    using ObjectPool = ObjectPool<A, MallocAllocator<A>>;
    static ObjectPool pool;
  public:
    A()
    {
        std::cout << "A::A()" << std::endl;
    }

    ~A()
    {
        std::cout << "A::~A()" << std::endl;
    }

    void *operator new(size_t n)
    {
        std::cout << "A::operator new(size_t n)" << std::endl;
        return pool.Allocate(n);
    }

    void operator delete(void *p)
    {
        std::cout << "A::operator delete(void *p)" << std::endl;
        pool.Deallocate(p);
    }
};

A::ObjectPool A::pool;
```



### 分配策略

#### 数组策略

数组策略构造一个定长内存数据和一个使用标记数组。

* 分配内存时，遍历数组找到尚未使用的内存进行分配，同时将其标记为已使用
* 释放内存时，计算出内存到数组头部的距离，将对应位置的内存标记为未使用

![](image-20240810163524820.png|800)

具体实现如下

```cpp
// 数组策略分配器
template <typename T, int N> class ArrayAllocater : public Allocator<T>
{
  public:
    ArrayAllocater()
    {
        for (int i = 0; i < N; ++i)
            m_used[i] = false;
    }

    ~ArrayAllocater() = default;

    virtual T *Allocate() override
    {
        for (int i = 0; i < N; ++i)
        {
            if (!m_used[i])
            {
                m_used[i] = true;
                return reinterpret_cast<T *>(&m_buffer[i * sizeof(T)]);
            }
        }
        throw std::bad_alloc();
    }

    virtual void Deallocate(T *p) override
    {
        auto i = ((unsigned char *)p - m_buffer) / sizeof(T);
        if (i >= N || i < 0)
            throw std::bad_alloc();
        m_used[i] = false;
    }

  private:
    unsigned char m_buffer[N * sizeof(T)];
    bool m_used[N];
};
```



#### 堆策略

数组策略在分配内存时需要遍历，因此具有 $O(n)$ 复杂度。利用最大堆算法，可以实现 $O(\log n)$ 复杂度。初始化建堆

* 分配内存时，将堆顶的 `Entry` 移动到堆尾，并将剩余元素重新建堆
* 释放内存时，将内存归还给堆尾后面的元素，然后将这个元素视为堆的一部分重新建堆

![](image-20240810171500214.png|800)

具体实现如下

```cpp
// 堆策略分配器
template <typename T, int N> class HeapAllocater : public Allocator<T>
{
    enum State
    {
        FREE = 1, // 未使用
        USED = 0, // 已使用
    };

    struct Entry
    {
        State state; // 状态
        T *p;        // 对象指针

        bool operator<(const Entry &other) const
        {
            return state < other.state;
        }
    };

  public:
    HeapAllocater() : m_available(N)
    {
        // 初始标记为可用
        for (int i = 0; i < N; ++i)
        {
            m_entry[i].state = FREE;
            m_entry[i].p = reinterpret_cast<T *>(&m_buffer[i * sizeof(T)]);
        }

        // 调用生成最大堆
        std::make_heap(m_entry, m_entry + N);
    }

    ~HeapAllocater() = default;

    virtual T *Allocate() override
    {
        if (m_available <= 0)
            throw std::bad_alloc();

        // pop_heap 将最大元素（堆顶）移动到堆尾，并将剩余元素建堆
        Entry e = m_entry[0];
        std::pop_heap(m_entry, m_entry + m_available);
        m_available--;

        // 标记为已使用，同时将指针置空
        m_entry[m_available].state = USED;
        m_entry[m_available].p = nullptr;

        return e.p;
    }

    virtual void Deallocate(T *p) override
    {
        if (p == nullptr || m_available >= N)
            return;

        m_entry[m_available].state = FREE;
        m_entry[m_available].p = reinterpret_cast<T *>(p);
        m_available++;

        // 将末尾元素上浮到合适位置
        std::push_heap(m_entry, m_entry + m_available);
    }

  private:
    unsigned char m_buffer[N * sizeof(T)];
    Entry m_entry[N]; // 堆数组
    int m_available;  // 可用对象数量
};
```

当我们分配内存后，`Entry` 中的指针就悬空了，直到某个内存（不一定是之前分配的内存）被归还时，`Entry` 中的指针指向这个内存。



#### 栈策略

栈策略能够实现 $O(1)$ 复杂度的内存分配。利用栈机制，将内存逐渐移动到栈上标记并进行分配。

- 分配内存时，如果内存栈中有内存，就将栈顶内存弹出分配；如果没有，就分配 `allocated` 位置的内存
- 释放内存时，将内存推入内存栈顶保存

![](image-20240810172437363.png|800)

具体实现如下

```cpp
// 栈策略分配器
template <typename T, int N> class StackAllocater : public Allocator<T>
{
  public:
    StackAllocater() : m_allocated(0), m_available(0)
    {
    }

    ~StackAllocater() = default;

    virtual T *Allocate() override
    {
        if (m_available <= 0 && m_allocated >= N)
            throw std::bad_alloc();

        if (m_available > 0)
            return m_stack[--m_available];
        else
        {
            auto p = m_buffer + m_allocated * sizeof(T);
            m_allocated++;
            return reinterpret_cast<T *>(p);
        }
    }

    virtual void Deallocate(T *p) override
    {
        m_stack[m_available++] = p;
    }

  private:
    unsigned char m_buffer[N * sizeof(T)];
    std::array<T *, N> m_stack; // 栈数组
    int m_allocated;            // 未使用对象位置
    int m_available;            // 栈中可用对象数量
};
```



#### 区块策略

上述策略中，即使是最好的栈策略，虽然能够实现 $O(1)$ 复杂度，但是长度固定，不够灵活。区块策略能够动态地管理内存

- 每个对象 `T` 所分配的内存被强制视为 `Chunk` 结构内存，这个结构保存指向下一块内存的指针
- 分配内存时，将会分配一整个 `Block`，每个 `Block` 中有固定数量的 `Chunk` 内存。程序首先分配这些 `Chunk`，通过 `next` 指针移动到下一个未分配的 `Chunk`，将其强制转换为 `T` 内存后返回。只有当 `Block` 内存用完时，才会分配新的 `Block` 内存
- 释放内存时，将传入的 `T` 内存强制转换为 `Chunk` 保存

![](image-20240810173134914.png|800)

具体实现如下

```cpp
// 区块策略分配器
template <typename T, int ChunksPerBlock> class BlockAllocater : public Allocator<T>
{
  public:
    BlockAllocater() : m_head(nullptr)
    {
    }

    ~BlockAllocater() = default;

    virtual T *Allocate() override
    {
        if (m_head == nullptr)
        {
            if (sizeof(T) < sizeof(T *))
            {
                std::cerr << "Object size is too small to be allocated by block allocator." << std::endl;
                throw std::bad_alloc();
            }
            m_head = AllocateBlock(sizeof(T));
        }

        Chunk *p = m_head;
        m_head = p->next;
        return reinterpret_cast<T *>(p);
    }

    virtual void Deallocate(T *p) override
    {
        auto chunk = reinterpret_cast<Chunk *>(p);
        chunk->next = m_head;
        m_head = chunk;
    }

  private:
    struct Chunk
    {
        Chunk *next;
    };

    Chunk *AllocateBlock(size_t chunkSize)
    {
        size_t blockSize = chunkSize * ChunksPerBlock;
        auto head = reinterpret_cast<Chunk *>(::malloc(blockSize));
        auto chunk = head;
        for (int i = 0; i < ChunksPerBlock - 1; ++i)
        {
            chunk->next = reinterpret_cast<Chunk *>(reinterpret_cast<unsigned char *>(chunk) + chunkSize);
            chunk = chunk->next;
        }
    }

  private:
    Chunk *m_head;
};
```

注意到所谓 `T` 和 `Chunk` 其实指的是相同的一块内存，只是 C++ 允许我们将同一块内存视为不同的数据类型进行处理。由于 `Chunk` 包含一个类型指针，它占用 `4/8` 个字节（由操作系统决定），因此当分配内存时，如果 ` T ` 大小大于 `T*`，则分配的内存不够 `Chunk` 使用，此时应该报错。



### 策略总结

我们总结上述提出的策略

- 数组策略：大小固定，分配效率低（$O(n)$）
- 堆策略：大小固定，分配效率一般（$O(\log n)$）
- 栈策略：大小固定，分配效率高（$O(1)$）
- 区块策略：动态扩容，分配效率高（$O(1)$）



## 线程池

### 基本概念

进程之间切换代价高，线程之间切换代价小（进程会分配资源，而线程只有自己的寄存器，没有分配资源）。多线程可以解决 CPU 和 IO 速度不匹配的问题，多线程更适合在 IO 切换频繁的场景。充分利用多核 CPU 资源、提高程序的并发效率。



线程池需要一个管理线程和多个工作线程。将多个任务通过队列保存，管理线程为每个任务分配线程数，如果任务数远多于工作线程数，则会创建新的工作线程；否则销毁多余的空闲线程。

![](image-20240810233642921.png|500)



### 实现效果

实现异步线程池

```cpp
#pragma once

#include <atomic>
#include <condition_variable>
#include <functional>
#include <future>
#include <iostream>
#include <map>
#include <memory>
#include <mutex>
#include <queue>
#include <thread>
#include <vector>

// 线程池
class ThreadPool
{
  public:
    ThreadPool(int minThread = 4, int maxThread = std::thread::hardware_concurrency())
        : m_minThread(minThread), m_maxThread(maxThread), m_curThread(minThread), m_idleThread(minThread), m_stop(false)
    {
        // 创建管理线程
        m_manager = new std::thread(&ThreadPool::Manager, this);

        // 创建工作线程，保存 id - worker 对
        for (int i = 0; i < m_minThread; i++)
        {
            auto worker = std::thread(&ThreadPool::Worker, this);
            std::cout << "Worker thread " << worker.get_id() << " is created." << std::endl;
            m_workers.insert(std::make_pair(worker.get_id(), std::move(worker)));
        }
    }

    ~ThreadPool()
    {
        m_stop.store(true);
        m_condition.notify_all();

        // 锁定线程创建过程，保证 m_workers 不会在销毁 workers 时改变
        {
            std::lock_guard<std::mutex> lock(m_managerMutex);
            for (auto &worker : m_workers)
            {
                // 阻塞等待线程退出
                if (worker.second.joinable())
                {
                    worker.second.join();
                    std::cout << "Worker thread " << worker.first << " is destroyed." << std::endl;
                }
            }

            if (m_manager->joinable())
            {
                m_manager->join();
                std::cout << "Manager thread is destroyed." << std::endl;
            }
            delete m_manager;
        }
    }

    // 添加任务
    void AddTask(std::function<void(void)> task)
    {
        // 只锁定队列互斥锁，保证队列安全
        {
            std::lock_guard<std::mutex> lock(m_queueMutex);
            m_tasks.emplace(task);
        }

        // 唤醒一个空闲线程
        m_condition.notify_one();
    }

    // 返回可调用对象 F 返回值类型的 future 对象
    template <typename F, typename... Args>
    std::future<typename std::result_of_t<F(Args...)>> AddTask(F &&f, Args &&...args)
    {
        using returnType = typename std::result_of_t<F(Args...)>;

        // 使用智能指针包装任务，这样传入 m_tasks 时，不会拷贝任务
        auto task = std::make_shared<std::packaged_task<returnType()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...));

        // 获取 future 对象
        std::future<returnType> future = task->get_future();

        // 添加到任务队列（换了一种锁定方式）
        m_queueMutex.lock();
        m_tasks.emplace([task]() { (*task)(); });
        m_queueMutex.unlock();

        // 唤醒一个空闲线程来执行这个任务
        m_condition.notify_one();
        return future;
    }

  private:
    void Manager()
    {
        // 定时检查线程状态
        while (!m_stop.load())
        {
            std::this_thread::sleep_for(std::chrono::seconds(3));
            int idle = m_idleThread.load();
            int cur = m_curThread.load();

            // 线程太多
            if (idle > cur / 2 && cur > m_minThread)
            {
                // 销毁两个线程，首先唤醒所有空闲线程
                m_exitThread.store(2);
                m_condition.notify_all();

                // 锁定释放资源过程
                std::lock_guard<std::mutex> lock(m_exitMutex);

                // 释放资源
                for (auto id : m_exitIds)
                {
                    auto ifound = m_workers.find(id);

                    // 找到需要退出的线程，执行 join
                    // 由于线程已经退出，所以 join 不会阻塞，但是会释放线程资源
                    if (ifound != m_workers.end())
                    {
                        ifound->second.join();
                        m_workers.erase(ifound);
                        std::cout << "Worker thread " << id << " is destroyed." << std::endl;
                    }
                }
                m_exitIds.clear();
            }
            // 线程太少
            else if (idle == 0 && cur < m_maxThread)
            {
                // 锁定线程创建过程，保证 m_workers 不会在销毁 workers 时改变
                std::lock_guard<std::mutex> lock(m_managerMutex);

                // 创建一个线程
                auto worker = std::thread(&ThreadPool::Worker, this);
                std::cout << "Worker thread " << worker.get_id() << " is created by Manager." << std::endl;
                m_workers.insert(std::make_pair(worker.get_id(), std::move(worker)));
                m_curThread++;
                m_idleThread++;
            }
        }
    }

    void Worker()
    {
        while (!m_stop.load())
        {
            std::function<void(void)> task = nullptr;

            // 锁定取出任务过程
            {
                std::unique_lock<std::mutex> lock(m_queueMutex);

                // 等待任务队列非空
                while (m_tasks.empty() && !m_stop.load())
                {
                    // 等待通知（需要传入 unique_lock 对象）
                    m_condition.wait(lock);

                    // 需要退出的空闲线程
                    if (m_exitThread.load() > 0)
                    {
                        m_curThread--;
                        m_idleThread--;
                        m_exitThread--;
                        std::cout << "Worker thread " << std::this_thread::get_id() << " exits." << std::endl;

                        // 锁定退出过程
                        std::lock_guard<std::mutex> lock(m_exitMutex);
                        m_exitIds.emplace_back(std::this_thread::get_id());
                        return;
                    }
                }

                // 取出任务
                if (!m_tasks.empty())
                {
                    std::cout << "Worker thread " << std::this_thread::get_id() << " gets task." << std::endl;
                    task = std::move(m_tasks.front());
                    m_tasks.pop();
                }
            }

            // 执行任务
            if (task)
            {
                m_idleThread--;
                task();
                m_idleThread++;
            }
        }
    }

  private:
    std::thread *m_manager;                           // 管理线程
    std::map<std::thread::id, std::thread> m_workers; // 工作线程
    std::vector<std::thread::id> m_exitIds;           // 已经退出的工作线程 id
    std::atomic<int> m_minThread;                     // 最小线程数
    std::atomic<int> m_maxThread;                     // 最大线程数
    std::atomic<int> m_curThread;                     // 当前线程数
    std::atomic<int> m_idleThread;                    // 空闲线程数
    std::atomic<int> m_exitThread;                    // 需要退出的线程数
    std::atomic<bool> m_stop;                         // 停止标志
    std::queue<std::function<void(void)>> m_tasks;    // 任务队列
    std::mutex m_queueMutex;                          // 任务队列互斥锁
    std::mutex m_exitMutex;                           // 退出线程互斥锁
    std::mutex m_managerMutex;                        // 管理线程互斥锁
    std::condition_variable m_condition;              // 条件变量
};
```

测试代码

```cpp
#include <iostream>

#include "Task.h"

void calc1(int x, int y)
{
    int z = x + y;
    std::cout << "Result: " << z << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(200));
}

int calc2(int x, int y)
{
    int z = x + y;
    std::this_thread::sleep_for(std::chrono::milliseconds(200));
    return z;
}

int main()
{
    ThreadPool pool(2);
    for (int i = 0; i < 10; i++)
    {
        // 绑定参数，从而满足 std::function<void(void)>
        pool.AddTask(std::bind(calc1, i, i * 2));
    }

    std::vector<std::future<int>> results;
    for (int i = 0; i < 10; i++)
        results.push_back(pool.AddTask(calc2, i, i * 2));

    for (auto &result : results)
        std::cout << "Result: " << result.get() << std::endl;
    // 这里每个 get() 将会依次阻塞，直到所有任务完成

    return 0;
}

```



## Json

### API

可以很容易实现一个 Json 格式类

```cpp
#pragma once

#include <map>
#include <stdexcept>
#include <string>
#include <vector>

class Json
{
  public:
    enum Type
    {
        json_null,
        json_bool,
        json_int,
        json_double,
        json_string,
        json_array,
        json_object
    };

    Json() : m_type(json_null)
    {
    }

    Json(bool value) : m_type(json_bool)
    {
        m_value.m_bool = value;
    }

    Json(int value) : m_type(json_int)
    {
        m_value.m_int = value;
    }

    Json(double value) : m_type(json_double)
    {
        m_value.m_double = value;
    }

    Json(const char *value) : m_type(json_string)
    {
        m_value.m_string = new std::string(value);
    }

    Json(const std::string &value) : m_type(json_string)
    {
        m_value.m_string = new std::string(value);
    }

    Json(Type type) : m_type(type)
    {
        switch (type)
        {
        case json_null:
            break;
        case json_bool:
            m_value.m_bool = false;
            break;
        case json_int:
            m_value.m_int = 0;
            break;
        case json_double:
            m_value.m_double = 0.0;
            break;
        case json_string:
            m_value.m_string = new std::string();
            break;
        case json_array:
            m_value.m_array = new std::vector<Json>();
            break;
        case json_object:
            m_value.m_object = new std::map<std::string, Json>();
            break;
        }
    }

    Json(const Json &other)
    {
        copy(other);
    }

    void operator=(const Json &other)
    {
        // 先清理再赋值
        clear();
        copy(other);
    }

    void copy(const Json &other)
    {
        m_type = other.m_type;
        switch (m_type)
        {
        case json_null:
            break;
        case json_bool:
            m_value.m_bool = other.m_value.m_bool;
            break;
        case json_int:
            m_value.m_int = other.m_value.m_int;
            break;
        case json_double:
            m_value.m_double = other.m_value.m_double;
            break;
        case json_string:
            m_value.m_string = other.m_value.m_string;
            break;
        case json_array:
            m_value.m_array = other.m_value.m_array;
            break;
        case json_object:
            m_value.m_object = other.m_value.m_object;
            break;
        }
    }

    void clear()
    {
        switch (m_type)
        {
        case json_null:
            break;
        case json_bool:
            m_value.m_bool = false;
            break;
        case json_int:
            m_value.m_int = 0;
            break;
        case json_double:
            m_value.m_double = 0.0;
            break;
        case json_string:
            delete m_value.m_string;
            m_value.m_string = new std::string();
            break;
        case json_array:
            for (auto &item : *m_value.m_array)
                item.clear();
            delete m_value.m_array;
            m_value.m_array = new std::vector<Json>();
            break;
        case json_object:
            for (auto &item : *m_value.m_object)
                item.second.clear();
            delete m_value.m_object;
            m_value.m_object = new std::map<std::string, Json>();
            break;
        }
        m_type = json_null;
    }

    bool operator==(const Json &other) const
    {
        if (m_type != other.m_type)
            return false;
        switch (m_type)
        {
        case json_null:
            return true;
        case json_bool:
            return m_value.m_bool == other.m_value.m_bool;
        case json_int:
            return m_value.m_int == other.m_value.m_int;
        case json_double:
            return m_value.m_double == other.m_value.m_double;
        case json_string:
            return *m_value.m_string == *other.m_value.m_string;
        case json_array:
            return m_value.m_array == other.m_value.m_array;
        case json_object:
            return m_value.m_object == other.m_value.m_object;
        }
        return false;
    }

    bool operator!=(const Json &other) const
    {
        return !(*this == other);
    }

    operator bool() const
    {
        if (m_type != json_bool)
            throw std::logic_error("Type error: not a boolean");
        return m_value.m_bool;
    }

    operator int() const
    {
        if (m_type != json_int)
            throw std::logic_error("Type error: not an integer");
        return m_value.m_int;
    }

    operator double() const
    {
        if (m_type != json_double)
            throw std::logic_error("Type error: not a double");
        return m_value.m_double;
    }

    operator std::string() const
    {
        if (m_type != json_string)
            throw std::logic_error("Type error: not a string");
        return *m_value.m_string;
    }

    operator std::vector<Json>() const
    {
        if (m_type != json_array)
            throw std::logic_error("Type error: not an array");
        return *m_value.m_array;
    }

    operator std::map<std::string, Json>() const
    {
        if (m_type != json_object)
            throw std::logic_error("Type error: not an object");
        return *m_value.m_object;
    }

    Json &operator[](int index)
    {
        if (m_type != json_array)
        {
            clear();
            m_type = json_array;
            m_value.m_array = new std::vector<Json>();
        }
        if (index < 0)
            throw std::out_of_range("Index out of range");
        int size = m_value.m_array->size();
        if (index >= size)
            m_value.m_array->resize(index + 1);
        return (*m_value.m_array)[index];
    }

    Json &operator[](const char *key)
    {
        if (m_type != json_object)
        {
            clear();
            m_type = json_object;
            m_value.m_object = new std::map<std::string, Json>();
        }
        return (*m_value.m_object)[key];
    }

    Json &operator[](const std::string &key)
    {
        return (*this)[key.c_str()];
    }

    std::vector<Json>::iterator begin()
    {
        if (m_type != json_array)
            throw std::logic_error("Type error: not an array");
        return m_value.m_array->begin();
    }

    std::vector<Json>::iterator end()
    {
        if (m_type != json_array)
            throw std::logic_error("Type error: not an array");
        return m_value.m_array->end();
    }

    bool isNull() const
    {
        return m_type == json_null;
    }

    bool isBool() const
    {
        return m_type == json_bool;
    }

    bool isInt() const
    {
        return m_type == json_int;
    }

    bool isDouble() const
    {
        return m_type == json_double;
    }

    bool isString() const
    {
        return m_type == json_string;
    }

    bool isArray() const
    {
        return m_type == json_array;
    }

    bool isObject() const
    {
        return m_type == json_object;
    }

    bool asBool() const
    {
        if (m_type != json_bool)
            throw std::logic_error("Type error: not a boolean");
        return m_value.m_bool;
    }

    int asInt() const
    {
        if (m_type != json_int)
            throw std::logic_error("Type error: not an integer");
        return m_value.m_int;
    }

    double asDouble() const
    {
        if (m_type != json_double)
            throw std::logic_error("Type error: not a double");
        return m_value.m_double;
    }

    std::string asString() const
    {
        if (m_type != json_string)
            throw std::logic_error("Type error: not a string");
        return *m_value.m_string;
    }

    std::vector<Json> asArray() const
    {
        if (m_type != json_array)
            throw std::logic_error("Type error: not an array");
        return *m_value.m_array;
    }

    std::map<std::string, Json> asObject() const
    {
        if (m_type != json_object)
            throw std::logic_error("Type error: not an object");
        return *m_value.m_object;
    }

    bool contains(int index) const
    {
        if (m_type != json_array)
            return false;
        return index >= 0 && index < m_value.m_array->size();
    }

    bool contains(const char *key) const
    {
        if (m_type != json_object)
            return false;
        return m_value.m_object->find(key) != m_value.m_object->end();
    }

    bool contains(const std::string &key) const
    {
        return contains(key.c_str());
    }

    void remove(int index)
    {
        if (contains(index))
            m_value.m_array->erase(m_value.m_array->begin() + index);
    }

    void remove(const char *key)
    {
        if (m_type != json_object)
            return;
        m_value.m_object->erase(key);
    }

    void remove(const std::string &key)
    {
        remove(key.c_str());
    }

    void append(const Json &other)
    {
        if (m_type != json_array)
        {
            clear();
            m_type = json_array;
            m_value.m_array = new std::vector<Json>();
        }
        m_value.m_array->push_back(other);
    }

    std::string dump() const
    {
        switch (m_type)
        {
        case json_null:
            return "null";
        case json_bool:
            return m_value.m_bool ? "true" : "false";
        case json_int:
            return std::to_string(m_value.m_int);
        case json_double:
            return std::to_string(m_value.m_double);
        case json_string:
            return "\"" + *m_value.m_string + "\"";
        case json_array: {
            std::string result = "[";
            for (size_t i = 0; i < m_value.m_array->size(); ++i)
            {
                if (i > 0)
                    result += ", ";
                result += (*m_value.m_array)[i].dump();
            }
            result += "]";
            return result;
        }
        case json_object: {
            std::string result = "{";
            for (auto it = m_value.m_object->begin(); it != m_value.m_object->end(); ++it)
            {
                if (it != m_value.m_object->begin())
                    result += ", ";
                result += "\"" + it->first + "\": " + it->second.dump();
            }
            result += "}";
            return result;
        }
        }
        return "";
    }

    ~Json()
    {
        // 注意内存释放问题，有可能出现重复释放
        clear();
    }

  private:
    union Value {
        bool m_bool;
        int m_int;
        double m_double;

        // 由于 string 和 vector 的构造函数和析构函数需要传入参数，因此不能直接使用指针
        std::string *m_string;
        std::vector<Json> *m_array;
        std::map<std::string, Json> *m_object;
    };

    Type m_type;
    Value m_value;
};
```



### Parse

读取字符串，解析为 JSON 格式的解析器

```cpp
#pragma once

#include "Json.h"

class Parse
{
  public:
    Parse() : m_str(""), m_idx(0)
    {
    }

    void load(const std::string &str)
    {
        m_str = str;
        m_idx = 0;
    }

    Json parse()
    {
        skipWhitespace();
        char c = m_str[m_idx];
        switch (c)
        {
        case 'n':
            return parseNull();
        case 't':
            return parseBoolean();
        case 'f':
            return parseBoolean();
        case '0':
        case '1':
        case '2':
        case '3':
        case '4':
        case '5':
        case '6':
        case '7':
        case '8':
        case '9':
        case '-':
            return parseNumber();
        case '\"':
            return parseString();
        case '[':
            return parseArray();
        case '{':
            return parseObject();
        case '\0':
            throw std::logic_error("Parse error: unexpected end of input");
        default:
            throw std::logic_error("Invalid JSON");
        }
        return Json();
    }

  private:
    void skipWhitespace()
    {
        while (m_str[m_idx] == ' ' || m_str[m_idx] == '\t' || m_str[m_idx] == '\n' || m_str[m_idx] == '\r')
            m_idx++;
    }

    Json parseNull()
    {
        if (m_str.compare(m_idx, 4, "null") == 0)
        {
            m_idx += 4;
            return Json();
        }
        throw std::logic_error("Parse error: null expected");
        return Json();
    }

    Json parseBoolean()
    {
        if (m_str.compare(m_idx, 4, "true") == 0)
        {
            m_idx += 4;
            return true;
        }
        if (m_str.compare(m_idx, 5, "false") == 0)
        {
            m_idx += 5;
            return false;
        }
        throw std::logic_error("Parse error: boolean expected");
        return Json();
    }

    Json parseNumber()
    {
        int pos = m_idx;
        if (m_str[m_idx] == '-')
            m_idx++;

        if (m_str[m_idx] < '0' || m_str[m_idx] > '9')
            throw std::logic_error("Parse error: number expected");
        while (m_str[m_idx] >= '0' && m_str[m_idx] <= '9')
            m_idx++;

        if (m_str[m_idx] != '.')
        {
            // 利用 atoi 会转换直到遇到非数字字符的特性，直接解析后面所有字符
            int i = std::atoi(m_str.c_str() + pos);
            return Json(i);
        }

        m_idx++;

        if (m_str[m_idx] < '0' || m_str[m_idx] > '9')
            throw std::logic_error("Parse error: number expected");
        while (m_str[m_idx] >= '0' && m_str[m_idx] <= '9')
            m_idx++;

        double d = std::atof(m_str.c_str() + pos);
        return Json(d);
    }

    Json parseString()
    {
        std::string str;
        while (m_str[++m_idx] != '\"')
        {
            if (m_str[m_idx] == '\\')
            {
                switch (m_str[++m_idx])
                {
                case '\"':
                    str += '\"';
                    break;
                case '\\':
                    str += '\\';
                    break;
                case '/':
                    str += '/';
                    break;
                case 'b':
                    str += '\b';
                    break;
                case 'f':
                    str += '\f';
                    break;
                case 'n':
                    str += '\n';
                    break;
                case 'r':
                    str += '\r';
                    break;
                case 't':
                    str += '\t';
                    break;
                case 'u':
                    // unicode
                    break;
                default:
                    throw std::logic_error("Parse error: invalid escape character");
                }
            }
            else
                str += m_str[m_idx];
        }
        m_idx++;
        return Json(str);
    }

    Json parseArray()
    {
        Json arr(Json::json_array);
        while (m_str[++m_idx] != ']')
        {
            // 直接递归解析
            arr.append(parse());
            skipWhitespace();
            if (m_str[m_idx] == ']')
                break;
            if (m_str[m_idx] != ',')
                throw std::logic_error("Parse error: comma expected");
        }
        m_idx++;
        return arr;
    }

    Json parseObject()
    {
        Json obj(Json::json_object);
        while (m_str[++m_idx] != '}')
        {
            // 解析 key
            std::string key = parse().asString();
            skipWhitespace();

            // 解析 value
            if (m_str[m_idx++] != ':')
                throw std::logic_error("Parse error: colon expected");

            // 添加到对象中
            obj[key] = parse();
            skipWhitespace();
            if (m_str[m_idx] == '}')
                break;
            if (m_str[m_idx] != ',')
                throw std::logic_error("Parse error: comma expected");
        }
        m_idx++;
        return obj;
    }

  private:
    std::string m_str; // 原始字符串
    int m_idx;         // 下一个解析位置
};
```



### 效果检验

在随便一个网站后台找到一个 JSON 文件进行测试

```cpp
#include "Parse.h"

#include <iostream>
#include <fstream>
#include <sstream>

int main()
{
    std::ifstream file("../../test.json");
    std::stringstream buffer;
    buffer << file.rdbuf();
    std::string str = buffer.str();
    file.close();

    std::cout << "Input JSON: " << str << std::endl;

    Parse p;
    p.load(str);

    std::ofstream out("../../output.json");
    Json j = p.parse();
    out << j.dump();
    out.close();
    return 0;
}
```


