# 编程规范

## 代码规范

### 注释位置

在重要文件头部

* 文件名 + 功能说明 + [作者] + [版本] + [版权声明] + [日期]

在用户自定义函数前

* 对函数接口进行说明
* 函数功能 + 入口参数 + 出口参数 + 返回值（包括出错处理）

重要语句块上方

* 对代码功能、原理进行解释说明

重要语句右方

* 定义非通用的变量
* 函数调用
* 较长的、多重嵌套的语句块结束处

在修改的代码行旁边加注释。



### 类的版式

“以数据为中心” 的版式

* private 类型的数据写在前面，public 类型的数据写在后面
* 关注类的内部结构

“以行为为中心” 的版式

* public 类型的数据写在前面，private 类型的数据写在后面
* 关注的是类应该提供什么样的接口

提倡后者，因为用户最关心的是接口。



### 标识符命名规则

尽量与所采用的操作系统或开发工具的风格保持一致。

* 在 Linux 平台
    * 习惯小写加下划线
    * function_name
* Windows 风格
    * 大小写混排的单词组合
    * FunctionName



Windows 命名规则

* 在变量和函数名前加上前缀，用于标识变量的数据类型
* [限定范围的前缀] + [数据类型前缀] + [有意义的英文单词]
    * 限定范围的前缀
        * 静态变量前加 s_，表示 static
        * 全局变量前加 g_，表示 global
        * 类内成员函数加 m_
        * 默认情况为局部变量
    * 数据类型前缀
        * ch 字符变量前缀
        * i 整型变量前缀
        * f 实型变量前缀
        * p 指针变量前缀

缺点是太过繁琐，可以简化命名规则：

* 变量名形式
    * 小写字母开头
    * 名词或形容词 + 名词
* 函数名形式
    * 大写字母开头
    * 动词或动词 + 名词
* 宏和 const 常量全用大写字母，并用下划线分割单词
* 限定范围的前缀和数据类型前缀可要可不要，无特殊意义的循环变量使用单字母变量



### 常量规则

使用直观的常量表示多次出现的数字或字符串

```cpp
const float PI = 3.1415926;
```

* 在 C++ 中用 const 常量完全取代宏常量
* 需要对外公开的常量集中放在一个公共的头文件中，不需要对外公开的常量放在定义文件的头部
* 不能在类声明中初始化 const 数据成员，要在初始化列表中初始化



const 数据成员只在某个对象生存期内是常量，而对类而言是可变的。

* 因为类可以创建多个对象
* 不同对象的 const 数据成员值不同

**应该使用类中的枚举常量实现在整个类中恒定的常量**

```cpp
class A
{
	enum {SIZE1 = 100, SIZE2 = 200};	// 枚举常量
    int arrayA[SIZE1];
    int arrayB[SIZE2];
};
```



### 函数设计原则

函数功能要单一，不要设计多用途的函数；函数的规模要小，尽量控制在 50 行代码以内。

参数规则：

* 参数要书写完整，不要省略参数类型和参数名
* 没有参数时，用 void 填充
* 参数个数尽量控制在 5 个以内
* 参数名要恰当，顺序要合理
* 如果参数是指针，且仅用于输入，则应在类型前加 const

返回值规则：

* 不要省略返回值类型，可声明为 void
* 确保返回值与声明类型一致，不要依赖自动类型转换
* 不能返回指向**栈内存**的指针

函数内部实现的规则：

* 在函数入口处，使用断言 assert 检查参数的合理性
* 尽量少用全局变量，确保函数的单出口和单入口，不得不用时，要严格控制对它的改写，例如几个有关联的函数需要使用全局变量时
    * 全局变量应和访问全局变量的函数放在单独的一个文件中，与其它文件分别编译
    * 并且将该全局变量声明为 static（静态全局变量）
* 尽量少用静态局部变量，以避免函数具有记忆功能



## 程序测试

- 单元测试：对程序中的单个子程序或具有独立功能的代码段进行测试
- 集成测试：在单元测试的基础上，将通过单元模块组装成系统或子系统，再进行测试，重点是检查模块之间的接口是否正确
- 系统测试：针对整个产品系统进行的测试，验证系统是否满足了需求规格的定义，以及软件系统的正确性和性能是否满足要求
- 验收测试：部署软件之前的最后一个测试操作。目的是确保软件准备就绪，向软件购买者展示该系统软件满足用户要求



### assert

应当尽量使用 assert 进行测试，当 assert 接收的表达式错误时，它会通过 `std::err` 输出错误信息，然后调用 abort 终止程序。

```c++
assert(0);

// 输出结果
Assertion failed: 0, file main.cpp, line 10
```

assert 的用法：

* 函数开始处检验传入参数的合法性
* 每个 assert 只检验一个条件
* 不能使用改变环境的语句，因为 assert 只在 DEBUG 时生效，如果在 assert 中改变变量，可能在真正运行时出现问题
* assert 用来避免显而易见的错误，不能用来处理异常错误是不应该出现的，异常是不可避免的。

最简单的使用规律是在函数最开始使用，在中间使用则需要慎重考虑。方法的最开始还没开始一个功能过程，在功能过程执行中出现的问题几乎都是异常。



频繁调用 assert 会极大地影响程序的性能，增加额外的开销。在调试结束后，可以通过宏调用禁用 assert

```cpp
#define NDEBUG
#include <cassert>
```



## 开发文档

### Readme

主要记录

* 日期、创建者、内容
* 每次重大功能的添加、修改

具体格式：

* 日期——TAB——创建者——TAB——内容
* 日期——TAB——修改的文件名——TAB——修改的功能
* 对修改后的功能和原理的说明
* ...



# Effective C++

## 让自己习惯 C++

### 条款 01 —— 视 C++ 为一个语言联邦

应当将 C++ 视为一个由相关语言组成的联邦。

1. C
2. Object-Oriented C++
3. Template C++
4. STL



### 条款 02 —— 尽量以 `const, enum, inline` 替换 `#define`

由于常量定义式通常被放在头文件内，因此有必要将指针也声明为 const 。例如

```cpp
const char* const authorName = "Scott";
```

当然使用 string 对象通常更合适。



对于 class 专属常量

```cpp
class GamePlayer
{
	static const int NumTurns = 5;
	int scores[NumTurns];
};
```

只要不取地址，就无需定义。然而如果需要取该常量的地址，就必须额外提供一个空的定义式

```cpp
const int GamePlayer::NumTurns;
```

不过尝试以后好像取地址也不需要提供定义式。



另外，如果编译器不允许将静态常量作为数组的大小，可以借助枚举类型

```cpp
class GamePlayer
{
	enum { NumTurns = 5 };
	int scores[NumTurns];
}
```

如果不希望别人从外部获得某个整数常量，enum 可以用来实现这个约束。



### 条款 03 —— 尽可能使用 `const`

关键字 const 可以强制执行常量语义。如果出现在星号左边，则表示指向的值是常量；如果出现在星号右边，则表示指针本身是常量。



#### 返回常量

令函数返回一个常量值可以降低因客户错误造成的意外。例如

```cpp
class Rational { ... };
const Rational operator*(const Rational& lhs, const Rational& rhs);
```

如果不返回 const，就有可能出现

```cpp
Rational a, b, c;
(a * b) = c;
```

一个良好的用户自定义类型的特征是它们避免无端地与内置类型不兼容。



#### 成员函数 const

成员函数如果是 const 意味着什么？这有两个流行概念：bitwise constness 和 logical constness 。前者认为成员函数不改变任何成员变量则是 const 。然而这仍然会有漏洞，例如在指针作为成员时

```cpp
class CTextBlock
{
public:
	char& operator[](std::size_t position) const 
	{
		return pText[position];
	}

private:
	char* pText;
};
```

注意到虽然 `[]` 操作符被声明为 const，然而它返回一个内部指针对应值的引用，这就给了修改值的可能

```cpp
const CTextBlock cctb("Hello");
char *pc = &cctb[0];
*pc = 'J';	// cctb 声明为 const 却被修改
```



这种情况导出 logical constness，即 const 成员函数在客户端检测不出时能够修改对象内部的成员。例如需要使用高速缓存 cache 文本区块的长度时

```cpp
class CTextBlock
{
public:
	std::size_t length() const
	{
		if (!lengthIsValid)
		{
			// 这里修改内部成员，因此是错误的
			textLength = std::strlen(pText);
			lengthIsValid = true;
		}
		return textLength;
	}

private:
	char* pText;
	std::size_t textLength;
	bool lengthIsValid;
};
```

这里 `legnth()` 代码不能通过编译，但是能够反映出 logical constness 的基本思想。添加 mutable 修饰可以得到正确的代码

```cpp
class CTextBlock
{
public:
	...
	
private:
	char* pText;
	mutable std::size_t textLength;
	mutable bool lengthIsValid;
};
```



#### 避免重复

如果在 `operator[]` 的常量和非常量版本实现中存在大量的代码（边界检测、日志访问、检测数据完整性），那么就会出现重复。我们可以借助类型转换在非常量版本中调用常量版本

```cpp
class CTextBlock
{
public:
	const char& operator[](std::size_t position) const;
	char& operator[](std::size_t position)
	{
		return const_cast<char&>(
			static_cast<const CTextBlock&>(*this)[position]
		);
	}
};
```

首先为 `*this` 添加常量修饰，然后可以调用 `operator[]` 的常量版本，最后用 `const_cast` 去除常量修饰返回。



### 条款 04 —— 确定对象被使用前已经初始化

在自定义类型初始化时，确保总是在初值列中列出所有成员变量，以免忘记哪些成员变量可以无需初值。尤其注意对于内置类型（整型、浮点等），允许无初值，因此一旦忘记初始化，就可能导致某些不确定行为。



C++ 有着非常固定的成员初始化次序。基类先于子类初始化，类成员变量按照声明次序初始化。然而，对于多个编译单元（多个目标文件），如果某个编译单元内的某个非局部静态对象的初始化依赖另一个编译单元内的某个非局部静态对象，那么前者初始化时后者可能尚未初始化。决定两者的初始化次序非常困难，但是我们可以通过单例模式常用的手法消除这个问题

```cpp
class A
{
public:
	A& singleton()
	{
		static A a;
		return a;
	}
};

class B
{
public:
	B()
	{
		auto a = A::singleton();
		// do something
	}
};
```

然而，这些内含 static 对象的函数在多线程系统中带有不确定性。任何非常量的静态对象在多线程环境下“等待某事发生”都会有麻烦。处理这个麻烦的一种方法是在单线程启动阶段就手动调用所有返回静态对象引用的函数。



## 构造/析构/赋值运算
### 条款 05 —— 了解 C++ 默默编写并调用哪些函数

如果没有自己声明，编译器会为空类声明一个默认构造函数、一个拷贝构造函数、一个拷贝赋值函数和一个析构函数。如果声明了构造函数，则不会创建默认构造函数；如果拷贝赋值函数不合法或则不会创建拷贝赋值函数。例如

```cpp
class A
{
private:
	std::string& name;
};
```

由于引用不能赋值，因此 A 不会有默认的拷贝赋值函数。另外，如果基类的拷贝赋值函数被声明为 private，则继承类不会生成拷贝赋值函数。



### 条款 06 —— 明确拒绝不想使用的自动生成函数

如果不想要自动生成的函数，例如拒绝拷贝构造函数，可以将其只声明为 private 函数。例如

```cpp
class A
{
private:
	A(const A&);
	A& operator=(const A&);
};
```

注意只是声明不要定义，防止成员函数或友元函数访问。这样当程序调用拷贝构造函数时就会报错。



我们可以将报错从连接期移动到编译期。只要将拷贝构造函数和拷贝赋值函数在一个专门为了阻止拷贝动作而设计的基类中。例如 

```cpp
class Uncopyable
{
protected:
	Uncopyable() {}
	~Uncopyable() {}

private:
	Uncopyable(const Uncopyable&);
	Uncopyable& operator=(const Uncopyable&);
};
```

只需要继承此基类，试图拷贝子类对象时，编译器便会尝试生成拷贝函数，这些函数会尝试调用基类的相应的 private 拷贝函数，从而报错。

> [!note]
> Uncopyable 类的实现和运用非常巧妙，不需要以 public 继承，析构函数不用是虚函数。不过使用这项技术可能导致多重继承，有时会阻止空基类优化。



### 条款 07 —— 为多态基类声明虚析构函数

当继承类对象经由一个基类指针被删除，则该对象的继承部分不会被销毁。任何类只要带有虚函数都几乎确定应该有一个虚析构函数。反之，如果类不含虚函数，通常表示它不意图被用作一个基类。例如

```cpp
class Point
{
public:
	Point(int x, int y);
	~Point();

private:
	int x, y;
};
```

如果一个 int 占用 32 位，则 Point 对象可以塞入一个 64 位寄存器。然而，如果其中包含一个虚函数，对象就需要包含一个虚表指针用于在运行期决定哪个虚函数被调用。

> [!note]
> 所有 STL 容器都不带任何虚函数，标准库的 string 也不带虚函数，因此不要试图继承这些标准容器。



如果希望使用多态，却没有任何可用的抽象函数接口，就可以为基类声明一个纯虚析构函数，这样基类作为抽象类不能被实例化。例如

```cpp
class AWOV
{
public:
	virtual ~AWOV() = 0;
};
```

由于子类析构函数调用时一定还会调用基类析构函数，因此还必须定义析构函数

```cpp
AWOV::~AWOV() {}
```



### 条款 08 —— 别让异常逃离析构函数

C++ 并不禁止析构函数抛出异常，但不建议这样做。考虑如下代码

```cpp
class Widget
{
public:
	...
	~Widget()
	{
		// 假设这里会抛出异常
	}
};

void do_something()
{
	std::vector<Widget> v;
}
```

当向量容器被销毁时，其中的每个对象都应该被销毁。如果其中一个对象析构时抛出异常，那么剩下的对象还是应该被销毁，因此容器继续调用它们的析构函数。但是如果又有一个析构函数抛出异常，在两个异常同时存在的情况下，程序可能结束执行也可能导致不明确行为。在当前情况下，会导致不明确行为。



如果析构函数遇到需要抛出异常的情况时，一种选择是直接结束程序。另一种选择是将可能抛出异常的操作封装为单独的函数调用。例如

```cpp
class File
{
public:
	void close()
	{
		file.close();
		closed = true;
	}

	~File()
	{
		if (!closed)
		{
			try
			{
				file.close();
			}
			catch (...)
			{
				// 如果失败，就记录失败或者吞下异常
			}
		}
	}

private:
	fstream file;
	bool closed;
};
```

这样用户可以在外部调用关闭函数，并处理可能发生的异常。如果用户不主动调用，则由析构函数调用并吞下异常，由此产生的问题责任就丢给了用户。



### 条款 09 —— 绝不在构造和析构过程中调用虚函数

如果在子类的构造或析构中调用重写的虚函数，将会调用基类对应的虚函数。例如在子类构造函数中调用虚函数

```cpp
class A
{
public:
	A()
	{
		// 调用虚函数
		test();
	}
	
	virtual void test() const = 0;
};

void A::test() const
{
	// do something
}

class B : public A
{
public:
	virtual void test() const
	{
		// do something
	}
};
```

接着执行下面的代码

```cpp
B b;
```

显然程序将会报错，因为 B 对象构造时调用 A 对象的构造函数，此时对象的类型是 A 类，因此会调用 A 类中的 test 函数。



如果类有多个构造函数，每个都需要执行某些相同工作，则可以将共同的初始化代码放进一个初始化函数内

```cpp
class A
{
public:
	A()
	{
		init();
	}

	void init()
	{
		// 调用虚函数
		test();
	}

	virtual void test() const = 0;
};
```

然而这里又出现了在 init 中调用虚函数的问题。



那么应该如何在构造时，确保对不同子类调用不同的虚函数呢？一种做法是

```cpp
class A
{
public:
	A(const std::string& info)
	{
		test(info);
	}

	void test(const std::string& info) const;
};

class B : public A
{
public:
	B( parameters ) : A(create_info(parameters))
	{
	}

private:
	static std::string create_info( parameters );
};
```

即通过一个静态函数，将输入参数转换为虚函数所需的参数调用。



### 条款 10 —— 拷贝赋值函数返回引用

赋值可以写成连锁形式

```cpp
int x, y, z;
x = y = z = 15;
```

为了实现连锁赋值，赋值操作符就应该返回指向操作符左侧的实参的引用。



### 条款 11 —— 在拷贝赋值中处理自我赋值

自赋值动作不是那么显然可以辨别出来。例如

```cpp
class A { ... };
a[i] = a[j];
*px = * py;
```

这种并不明显的自我复制，是“别名”（aliasing）带来的结果。再例如来自同一个继承体系的两个对象就可能造成别名

```cpp
class Base { ... };
class Derived : public Base { ... };
void do_somethig(const Base& rb, Derived* pd);
```



自我赋值带来的不安全就在于动态内存分配。例如

```cpp
class Bitmap { ... };
class Widget
{
private:
	Bitmap* pb;
};

Widget& Widget::operator=(const Widget& rhs)
{
	delete pb;
	pb = new Bitmap(*rhs.pb);
	return *this;
}
```

这里首先删除了分配的内存，那么接下来的拷贝操作显然会报错。我们可以首先判断是否相同

```cpp
Widget& Widget::operator=(const Widget& rhs)
{
	if (this == &rhs) return *this;
	
	delete pb;
	pb = new Bitmap(*rhs.pb);
	return *this;
}
```

然而，这看似解决了问题，却还会带来其它问题。如果分配新内存时出现异常，那么赋值后 Widget 指向了被删除的空内存。



更好的方案是简单地交换代码

```cpp
Widget& Widget::operator=(const Widget& rhs)
{
	auto p = pb;
	pb = new Bitmap(*rhs.pb);
	delete p;	
	return *this;
}
```

如果想要提高效率，或许可以把相等判断添加回去，然而需要先考虑自我赋值的频率有多高，是否在每次赋值时都添加一次条件判断来避开自我赋值时多余的内存分配。



确保代码不但异常安全而且自我赋值安全的替代方案是使用交换技术。例如

```cpp
class Widget
{
	void swap(Widget& rhs);
};

Widget& Widget::operator=(const Widget& rhs)
{
	Widget tmp(rhs);
	swap(tmp);
	return *this;
}
```



### 条款 12 —— 复制对象时勿忘每个成分

关键在于使用继承类的拷贝函数时，一定要记得调用基类的拷贝函数。



## 资源管理

### 条款 13 —— 以对象管理资源

如果需要返回裸指针的函数，那么最好将其立即放进管理对象内。这种“以对象管理资源”的观念常被称为“资源取得时机便是初始化时机”（Resource Acquisition Is Initialization；RAII）。



### 条款 14 —— 在资源管理类中小心拷贝行为

典型的资源管理类如

```cpp
class Lock
{
public:
	explicit Lock(Mutex* pm) : mutexPtr(pm)
	{
		lock(mutexPtr);
	}

	~Lock()
	{
		unlock(mutexPtr);
	}

private:
	Mutex *mutexPtr;
};
```

这样就可以通过一个区块来实现线程锁的锁定和释放。我们也可以用智能指针并附带删除器实现

```cpp
class Lock
{
public:
	// 使用 unlock 作为删除器
	explicit Lock(Mutex* pm) : mutexPtr(pm, unlock)
	{
		lock(mutexPtr.get());
	}

private:
	std::shared_ptr<Mutex> mutexPtr;
};
```



如果一个 RAII 对象被复制，通常有如下选择

1. 禁止赋值
2. 使用引用计数法
3. 复制底部资源
4. 转移底层资源的拥有权



### 条款 15 —— 在资源管理类中提供对原始资源的访问

有时需要获得原始指针，可以定义 get 函数用来返回原始指针。另一种方法是提供隐式转换函数

```cpp
class Font
{
public:
	operator FontHandle() const
	{
		return f;
	}

private:
	FontHandle f;
}
```

让 RAII 类返回原始资源似乎与“封装”发生矛盾，但这并不是什么设计灾难。因为 RAII 类不是为了封装而存在，而是为了确保资源释放能够自动发生。



### 条款 16 —— 采取相同形式成对使用 `new` 和 `delete` 

如果使用 `new` 创建动态数组，那么就要使用 `delete[]` 释放资源。为了避免错误匹配，尽量不要对数组形式使用 `typedef` 操作。



### 条款 17 —— 以独立语句将 `new` 对象放入智能指针

假设存在一个接受智能指针和一个整型的函数

```cpp
void test(std::shared_ptr<Widget> pw, int priority;
```

调用方式为

```cpp
test(std::shared_ptr<Widget>(new Widget), priority());
```

那么 C++ 会以什么次序完成调用呢？这并不确定，因为编译器会对代码执行流程重排，得到尽可能更高的执行效率。假如重排后顺序为

1. 执行 `new Widget`
2. 调用 `priority`
3. 调用 `std::shared_ptr` 构造函数

那么如果该调用导致异常，就可能导致 `new` 得到的指针丢失。要避免这一问题，应该先单独创建智能指针

```cpp
std::shared_ptr<Widget> pw(new Widget);
test(pw, priority());
```

现如今标准库提供了封装构造过程的函数，使用该函数也可以避免这一问题

```cpp
test(std::make_shared<Widget>(), priority());
```



## 设计与声明

### 条款 18 —— 让接口容易被正确使用，不易被误用

#### 外覆类型

首先要考虑可能出现什么样的错误。假设有一个日期类

```cpp
class Date
{
public:
	Date(int month, int day, int year);
}:
```

那么用户可能以错误的顺序输入参数，也可能输入无效的月份或天气。



解决这种问题的方案是导入简单的外覆类型 wrapper types 表示参数类型

```cpp
struct Day
{
	explicit Day(int d) : val(d) {}
	int val;
};

struct Month
{
	explicit Month(int m) : val(m) {}
	int val;
};

struct Year
{
	explicit Year(int y) : val(y) {}
	int val;
};
```

接下来要限制值的范围，例如预先定义所有有效的月份

```cpp
class Month
{
public:
	static Month Jan() { return Month(1); }
	static Month Feb() { return Month(2); };
	...

private:
	explicit Month(int m);
};
```

这种方法使用函数替换对象来表示某个特殊的对象。



#### 类型行为

除非有好的理由，否则应该尽量让类型的行为与内置类型一致。例如已知 int 类型的行为方式，就应该让自定义类型与其有相同表现。例如

```cpp
a * b = c;
```

对乘积赋值并不合法，因此应该让 `operator*` 返回常量引用。

> [!note]
 避免无端与内置类型不兼容，真正的理由是为了提供行为一致的接口。STL 容器的接口通常非常一致，例如每个 STL 容器都有一个 `size` 成员函数，返回容器中对象的数量。



#### 返回智能指针

使用返回裸指针的函数可能出现用户忘记删除指针的情况，因此最好直接返回智能指针。如果删除指针之前需要释放资源，可以绑定一个删除器。如果需要在原始指针之前构建智能指针，可以创建一个空的智能指针，然后赋值

```cpp
std::shared_ptr<A> p(nullptr, release());
p = new A;
```

使用智能指针的一个很好的性质是，它会自动使用每个指针专属的删除器，从而消除另一个潜在的客户错误——“cross-DLL problem“。这个问题发生于对象在 DLL 中被动态创建，却在另一个 DLL 中被销毁。在许多平台上，这类操作会导致运行期错误。而智能指针不会有这份问题，因为它默认的删除器是其所诞生的 DLL 中的删除函数。



### 条款 19 —— 设计类 class 犹如设计类型 type

必须了解面对的问题。

1. 新 type 的对象应该如何被创建和销毁？
2. 对象的初始化和对象的赋值应该有什么差别？
3. 新 type 的对象如果以值传递，意味着什么？
4. 什么是新 type 的“合法值”？
5. 新 type 需要配合某个继承图系（inheritance graph）吗？
6. 新 type 需要什么样的转换？
7. 什么样的操作符和函数对此新 type 而言是合理的？
8. 什么样的自动生成函数应该被删除？
9. 谁能够取得新 type 的成员？
10. 什么是新 type 的“未声明接口（undeclared interface）”？
11. 新 type 有多么一般化？
12. 真的需要一个新 type 吗？



### 条款 20 —— 以传常量引用替换传值

以常量引用作为参数通常效率更高，除了简单的内置类型。以引用传值还可以实现多态，因为引用往往以指针实现出来，意味着真正传递的是指针。某些编译器拒绝把只由一个 double 组成的对象放进缓存器，却允许对单独的 double 这么做。此时就更应该以引用传递。



### 条款 21 —— 必须返回对象时，不要返回其引用

考虑有理数类

```cpp
class Rational
{
public:
	Rational(int numerator = 0, int denominator = 1);

private:
	int n, d;

	// 返回常量，与 int 表现一致
	friend const Rational operator*(const Rational&, const Rational&);
};
```



比较糟糕的方式是返回局部对象的引用

```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
	Rational result(lhs.n * rhs.n, lhs.d * rhs.d);
	return result;
}
```

即使返回堆空间对象，也会产生问题

```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
	Rational* result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
	return *result;
}
```

关键在于连锁乘法

```cpp
Rational x, y, z, w;
w = x * y * z;
```

其中一次乘法动态分配的内存就丢失了。因此不如直接构造新对象

```cpp
const Rational operator*(const Rational& lhs, const Rational& rhs)
{
	Rational result(lhs.n * rhs.n, lhs.d * rhs.d);
	return result;
}
```



### 条款 22 —— 将成员变量声明为 private

#### 语法一致性 

首先是语法一致性。如果 public 接口内的每样东西都是函数，用户就不需要在打算访问类成员时区分是否该使用小括号。



#### 访问控制

通过函数控制成员，可以实现出“不准访问”、“只读访问”、“读写访问”甚至“惟写访问”。例如

```cpp
class AccessLevels
{
public:
	int get_read_only() const { return readOnly; }
	void set_read_write(int value) { readWrite = value; }
	int get_read_write() const { return readWrite; }
	void set_write_only(int value) { writeOnly = value; }

private:
	int noAccess;	// 不能访问
	int readOnly;	// 只读访问
	int readWrite;	// 读写访问
	int writeOnly;	// 惟写访问
};
```



#### 封装

如果通过函数访问成员变量，就可以在日后修改内部成员和函数体，而用户不需要修改函数调用过程。Public 意味着不封装，不封装意味着不可改变，任何对原始代码的修改都会导致对用户代码的修改。Protected 成员变量也类似，对它的修改会导致对所有继承类的破坏。

> [!note]
从封装的角度来看，其实只有两种访问权限：private 和其它。



### 条款 23 —— 以非成员、非友元替换成员函数

假设有这样一个类

```cpp
class WebBrowser
{
public:
	void clear_cache();
	void clear_history();
	void remove_cookies();
};
```

如果用户希望能一次执行所有动作，可以提供一个成员函数

```cpp
class WebBrowser
{
public:
	void clear_everything();
};
```

或者使用非成员函数

```cpp
void clear_browser(WebBrowser& wb)
{
	wb.clear_cache();
	wb.clear_history();
	wb.remove_cookies();
}
```

从面向对象的角度来看，似乎应该使用成员函数的版本。然而，这并不正确。面向对象守则要求数据应该尽可能被封装，而成员函数比非成员函数的封装性更弱，因为它可以访问其它实际上不应该访问的成员变量。

> [!note]
只有当不增加“能够访问类内私有成分”的函数数量时，才是成功的封装。



比较自然的方式，是将辅助函数放在不同头文件的相同命名空间中

```cpp
// webbrowser.h
namespace WebBrowserStuff
{
class WebBrowser { ... };
}

// webbrowserbookmarks.h
namespace WebBrowserStuff
{
	// 与 bookmark 相关的辅助函数
}

// webbrowsercookies.h
namespace WebBrowserStuff
{
	// 与 cookie 相关的辅助函数
}
```



### 条款 24 —— 若所有参数都需要类型转换，则采用非成员函数

以有理数类为例

```cpp
class Rational 
{
public:
  Rational(int n = 0, int d = 1) : n(n), d(d) {}

  int numerator() const { return n; }
  int denominator() const { return d; }

private:
  int n;
  int d;
};
```

使用上面的乘法运算，可以实现有理数之间的乘法以及有理数乘整型，因为整型作为参数可转换为有理数。然而，整型乘有理数却会报错，这时候就定义非成员函数

```cpp
const Rational operator*(const Rational &lhs, const Rational &rhs) 
{
  return Rational(lhs.numerator() * rhs.numerator(),
                  lhs.denominator() * rhs.denominator());
}
```

由于我们并不需要访问其它成员，因此也不需要将其声明为友元。

> [!note]
成员函数的反面是非成员函数而非友元函数。



### 条款 25 —— 考虑写一个不抛异常的 swap 函数

原本 swap 只是 STL 的一部分，而后成为异常安全性编程的支柱，以及用来处理自我赋值可能性的一个常见机制。最简单的 swap 函数如下

```cpp
namespace std {
template <typename T> void swap(T &a, T &b) 
{
  T tmp(a);
  a = b;
  b = tmp;
}
} // namespace std
```



一种数据类型是“以指针指向一个对象，该对象内含真正的数据”. 这种设计的常见表现形式是所谓的“pimpl 手法”。例如

```cpp
class WidgetImpl
{
public:
	...

private:
	int a, b, c;
	std::vector<double> v;
};

class Widget
{
public:
	Widget(const Widget& rhs);
	Widget& operator=(const Widget& rhs)
	{
		...
		*pImpl = *(rhs.pImpl);
		...
	}

private:
	WidgetImpl* pImpl;
};
```

要实现此类型的 swap 函数，可以首先定义一个内部的 swap 成员

```cpp
class Widget
{
public:
	void swap(Widget& other)
	{
		using std::swap;
		swap(pImpl, other.pImpl);
	}
};

namespace std 
{
template <> void swap<Widget>(Widget &a, Widget &b) 
{ 
	a.swap(b); 
}
} // namespace std
```

这里我们对 STL 的 swap 模板进行全特化。

> [!note]
成员 swap 绝不可抛出异常，因为它原本就应该用来提供强烈的异常安全性保障。



假设 Widget 和 WidgetImpl 都是模板类，尝试使用下面的方案

```cpp
namespace WidgetStuff 
{
template <typename T> void swap(Widget<T> &a, Widget<T> &b) 
{
	a.swap(b); 
}
} // namespace std
```

注意不要尝试偏特化 `swap<Widget<T>>`，函数偏特化不被允许。另外，我们没有在 std 中定义新的模板，这也并不被允许。



假设需要在一个模板函数内部进行交换，就可以使用 `std::swap` 以及可能存在的成员函数 swap，此时编译器会自己选择合适的交换函数。例如

 ```cpp
namespace WidgetStuff 
{
template <typename T>
void do_something(T& obj1, T& obj2)
{
  using std::swap;
  swap(obj1, obj2);
}
}
```

此时首先会查找当前命名空间中是否有合适的 swap 函数，从而匹配到正确的函数。



## 实现

### 条款 26 —— 尽可能延后变量定义式的出现时间

只要定义了一个变量，其类型带有一个构造函数或析构函数，则程序就要承受构造成本和析构成本。并且如果使用默认构造函数然后赋值，其效率低于直接使用拷贝构造函数，因此应该尝试延后变量定义直到能够给它初值参数为止。



对于循环体，有两种写法

```cpp
Widget w;
for (int i = 0; i < n; i++) 
{
  // do something
  w = i;
}

for (int i = 0; i < n; i++) 
{
  Widget w(i);
}
```

它们的成本如下

- 1 个构造函数 + 1 个析构函数 + n 个赋值操作
- n 个构造函数 + n 个析构函数

根据实际开销比较，如果构造和析构的成本大于赋值，选择前者；否则选择后者。



### 条款 27 —— 尽量少做类型转换

类型转换 cast 破坏了类型系统。C 风格的转型动作

```c
(T)expression
T(expression)
```

C++ 提供新的转型

```cpp
const_cast<T>(expression)			// 去除常量
dynamic_cast<T>(expression)			// 安全向下转型
reinterpret_cast<T>(expression)		// 低级转型，取决于编译器
static_cast<T>(expression)			// 强迫隐式转换
```

新式转型更受欢迎，因为很容是被辨识；并且有利于编译器区分不同的转型。



#### 转型对代码的改变

转型看上去好像只是改变了类型，但实际上有可能编译出运行期执行的代码。例如

```cpp
int x, y;
double d = static_cast<double>(x) / y;
```

显然将整型转换为浮点会改变实际的值。



#### 多重继承

考虑下面带有多重继承的代码

```cpp
#include <iostream>

class Base1 
{
public:
  int x;
};

class Base2 
{
public:
  int y;
};

class Derived : public Base1, public Base2 
{
public:
  int z;
};

int main() 
{
  Derived d;
  Base1 *pb1 = &d;
  Base2 *pb2 = &d;
  std::cout << "\&d = " << &d << std::endl;
  std::cout << "pb1 = " << pb1 << std::endl;
  std::cout << "pb2 = " << pb2 << std::endl;
  return 0;
}
```

上面代码看上去只是让基类指针指向子类对象，然而有时 `&d` 与 `pb` 可能并不相同。编译运行得到

```shell
&d = 000000831779FAE8
pb1 = 000000831779FAE8
pb2 = 000000831779FAEC
```

注意到两个基类指针的值不同。这种情况下，程序会将一个偏移量 offset 施加于子类指针上来获得正确的基类指针值。

> [!note]
> 单一对象可能拥有一个以上的地址。

这意味着通常应该避免做出“对象在 C++ 中如何布局”的假设，更不应该以此假设为基础执行转型动作。



#### 静态转型

某些错误的转型会导致似是而非的代码。考虑如下代码

```cpp
#include <iostream>

class Base 
{
public:
  int x = 0;

  void reset() { x = 10; }
};

class Derived : public Base 
{
public:
  int y = 1;

  void reset() 
  {
    Base::reset();							// 在 this 对象上调用基类 reset
    static_cast<Base>(*this).reset();		// 在 this 对象的拷贝上调用基类 reset
    static_cast<Base&>(*this).reset();		// 在 this 对象的引用上调用基类 reset
    y = 20;
  }
};

int main() 
{
  Derived d;
  d.reset();
  std::cout << "d.x = " << d.x << std::endl;
  std::cout << "d.y = " << d.y << std::endl;
  return 0;
}
```

注意到子类调用基类函数的三种方式，其中第二种方式是错误的，因为它是在一个拷贝对象上执行的调用。



#### 动态转型

动态转型 `dynamic_cast` 的许多实现版本执行速度相当慢。例如至少一个很普遍的实现版本基于类名称的字符串比较，因此如果在一个 4 层单继承体系中的某个对象上执行动态转型，就会需要 4 次字符串比较。



通常之所以需要使用动态转型，是因为要对一个基类对象执行子类函数，然而无法判断基类对象是否是正确的子类。有两个一般性做法可以避免这个问题

1. 使用容器并在其中储存直接指向子类对象的指针
2. 在基类中提供虚函数



### 条款 28 —— 避免返回指向对象内部成分的句柄

返回指向对象内部成分的句柄，带来的第一个问题是访问权限的混乱。例如

```cpp
struct Point 
{
  int x, y;
};

class Rect 
{
private:
  std::shared_ptr<Point> m_upperLeft;
  std::shared_ptr<Point> m_lowerRight;

public:
  Point &upper_left() const { return *m_upperLeft; }
  Point &lower_right() const { return *m_lowerRight; }
};
```

注意到常量成员函数返回了内部指针的引用，这就导致外部可以通过常量成员函数访问并修改内部成员。解决方案也很简单，就是添加返回的常量限定

```cpp
const Point &upper_left() const { return *m_upperLeft; }
const Point &lower_right() const { return *m_lowerRight; }
```



另一个问题是可能导致悬空引用。例如

```cpp
Rect *r = new Rect;
const Point *p = &(r->upper_left());
delete r;
```

一旦 Rect 指针被销毁，那么 Point 指针就悬空了。



### 条款 29 —— 为“异常安全”而努力是值得的

假设有一个类表示菜单的背景图案

```cpp
class PrettyMenu 
{
public:
  void change_background(std::size_t size) 
  {
    m_mutex.lock();
    delete[] m_bgImg;
    ++m_imgChanges;
    m_bgImg = new char[size];
    m_mutex.unlock();
  }

private:
  std::mutex m_mutex;
  char *m_bgImg;
  int m_imgChanges;
};
```

为了确保多线程安全，使用线程锁锁定对背景图案资源的申请、删除和记录。然而，从异常安全的角度来看，这个函数相当糟糕。



异常安全有两个条件

1. 不泄露任何资源 —— 一旦 `new` 操作异常，那么线程不会被解锁
2. 不允许数据损坏 —— 一旦 `new` 操作异常，`m_bgImg` 将指向无效的地址

解决第一个问题只需要使用 RAII 技术，因此关键在于第二个问题。



#### 异常安全函数

异常安全函数应当提供以下三个保证之一

- 基本承诺：如果异常被抛出，确保程序内任何事物应当有效。例如一旦 `new` 操作异常，`m_bgImg` 可能指向原先的地址，也可能指向某个默认地址，用户需要调用函数查看图像是什么
- 强烈保证：如果异常被抛出，确保程序状态不改变。例如一旦抛出异常，程序应当返回调用函数之前的状态
- 不抛掷 noexcept 保证：承诺绝不抛出异常



#### 基本承诺

首先我们尝试解决数据损坏，使用智能指针并调整代码顺序来做到

```cpp
class PrettyMenu 
{
public:
  void change_background(std::size_t size) {
    std::unique_lock<std::mutex> lock(m_mutex);
    m_bgImg.reset(new char[size]);
    ++m_imgChanges;
  }

private:
  std::mutex m_mutex;
  std::shared_ptr<char> m_bgImg;
  int m_imgChanges;
};
```

此时如果 `new` 操作异常，则 `reset` 不会执行，`m_bgImg` 指向原先的地址。



#### 强烈保证

考虑更复杂的场景，假设上面的函数使用流式输入

```cpp
class PrettyMenu 
{
public:
  void change_background(std::istream src) {
    std::unique_lock<std::mutex> lock(m_mutex);
    m_bgImg.reset(new Image[src]);
    ++m_imgChanges;
  }

private:
  std::mutex m_mutex;
  std::shared_ptr<Image> m_bgImg;
  int m_imgChanges;
};
```

此时如果 `Image` 构造函数抛出异常，由于它从输入流中读取内容，因此输入流的状态被改变了，无法提供返回调用前状态的保证。

> [!note]
>  一般化的设计策略称为 copy and swap，它首先复制一份要修改的内容，然后修改副本，确保修改过程正确后再重新赋值回去。



### 条款 30 —— 透彻了解 `inlining` 的里里外外

虽然 inline 通过将函数直接替换为函数本体，免去了函数调用的开销，然而这会增加目标码 object code 的大小。在一台内存有限的机器上，过度热衷于 inline 会导致程序体积太大。即使拥有虚内存，inline 造成的代码膨胀也会导致额外的换页行为 paging，降低指令高速缓存装置的命中率。

- 内联函数通常一定被放置于头文件中，因为大多数构建环境在编译过程中内联，而为了将函数调用替换为函数本体，编译器必须知道函数的内容
- 模板函数也通常放置于头文件中，但是它与内联函数无关
- 虚函数不能是内联函数，因为程序需要等待运行期才知道要调用的函数
- 定义在类中的函数是隐式内联的，然而编译器会根据情况决定是否内联



### 条款 31 —— 将文件间的编译依存关系降至最低

假设对类的 private 成分做了修改，那么编译器将会重新编译所有相关文件。问题在于 C++ 没有把“将接口从实现中分离”做得很好。类定义包括了成员变量的定义，例如

```cpp
#include <string>
#include "date.h"
#include "address.h"

class Person 
{
public:
  Person(const std::string &name, const Date &birthday, const Address &addr);

  std::string name() const;
  Date birthday() const;
  Address address() const;

private:
  std::string m_name;
  Date m_birthday;
  Address m_address;
};
```

当前文件依赖两个额外的头文件，形成一种编译依存关系 compilation dependency 。



或许我们可以使用前置声明。然而前置声明的主要困难是，编译器必须在编译期知道对象的大小。考虑

```cpp
int main()
{
	int x;
	Person p;
}
```

当编译器看到变量定义，必须知道需要分配多少栈内存，要知道 Person 的大小，就需要询问定义式。这使得我们不能在不改变 Person 定义式的情况下，修改其成员变量。



#### 指向实现

这种问题在 Java 中不存在，因为它在定义对象时，只分配一个指针。例如

```cpp
int main()
{
	int x;
	Person* p;
}
```

事实上，借助 pimpl idiom (pointer to implementation) 设计，我们可以实现类似的效果。例如

```cpp
// Person.h

class PersonImpl;
class Date;
class Address;

class Person 
{
public:
  Person(const std::string &name, const Date &birthday, const Address &addr);

  std::string name() const;
  Date birthday() const;
  Address address() const;

private:
  std::shared_ptr<PersonImpl> m_impl;
};
```

然后在另一个文件中，例如源文件中定义

```cpp
// Person.cpp

class PersonImpl
{
public:
	std::string m_name;
	Date m_birthday;
	Address m_address;
	
	std::string name() const { return m_name; }
	Date birthday() const { return m_birthday; }
	Address address() const { return m_address; }
};

Person::Person(const std::string &name, const Date &birthday, const Address &addr)
	: m_impl(new PersonImpl(name, birthday, addr))
{
}

std::string Person::name() const { return m_impl->name(); }
Date Person::birthday() const { return m_impl->birthday(); }
Address Person::address() const { return m_impl->address(); }
```

这样当用户使用 Person 类时，任何对 PersonImpl 的修改都不会造成 Person 相关的代码重新编译。



这种分离的关键策略在于

- 如果使用对象引用或对象指针可以完成任务，就不要使用对象
- 尽量以类声明替换类定义（声明一个包含类的函数时，即使用到传值参数，也不需要类的定义）

例如可以如此声明

```cpp
class Date;
Date today();
void clear_appointments(Date d);
```

虽然要调用这些函数时，还是需要包含 `Date` 的定义，然而关键在于用户可能只调用某些函数，此时就没有必要引入定义式，从而节省了编译开销。
 


#### 抽象基类

使用抽象接口配合对象工厂来生成不同子类实例，同时还可以避免修改不同子类时重新编译基类相关的代码。例如

```cpp
// Person.h

class Person {
public:
  virtual ~Person();
  virtual std::string name() const = 0;
  virtual Date birthday() const = 0;
  virtual Address address() const = 0;

  static std::shared_ptr<Person>
  create(const std::string &name, const Date &birthday, const Address &address);
};

// RealPerson.h

class RealPerson : public Person {
private:
  std::string m_name;
  Date m_birthday;
  Address m_address;

public:
  RealPerson(const std::string &name, const Date &birthday,
             const Address &address)
      : m_name(name), m_birthday(birthday), m_address(address) {}

  ~RealPerson() override {}
  std::string name() const override { return m_name; }
  Date birthday() const override { return m_birthday; }
  Address address() const override { return m_address; }
};

// Person.cpp
std::shared_ptr<Person> Person::create(const std::string &name,
                                       const Date &birthday,
                                       const Address &address) {
  return std::make_shared<RealPerson>(name, birthday, address);
}
```

用户只需要使用 `Person.h` 头文件，然后通过 `create` 方法获得子类实例的指针。



## 继承与面向对象设计

### 条款 32 —— 确定 `public` 继承塑模出 `is-a` 关系

以 C++ 进行面向对象编程，最重要的规则是：public inheritance（公开继承）意味 "is-a" 的关系。这意味着所有能够在基类执行的操作，也必须能够在子类中执行。



考虑如下代码

```cpp
class Person { ... };
class Student : public Person { ... };
```

这表明“学生”是一种“人”，即“人”可以执行的事情，“学生”也可以执行；反之，“学生”可以执行的事情，“人”不一定能够执行。



然而，并非所有情况都是如此。例如企鹅是一种鸟

```cpp
class Bird
{
public:
	virtual void fly();
};

class Penguin : public Bird
{
	...
};
```

鸟通常是会飞的，然而企鹅却不会飞。一种处理方式是添加一个新的中间类

```cpp
class Bird
{
};

class FlyingBird : public Bird
{
public:
	virtual void fly();
};

class Penguin : public FlyingBird
{
	...
};
```

即便如此，可能某些系统中其实并不需要区分会飞鸟和不会飞的鸟，因此需要根据系统需求来判断是否有必要考虑这种区分。



另一种处理方式是在飞行时报错

```cpp
class Penguin : public Bird
{
public:
	virtual void fly() { throw; }
};
```

这种错误只会在运行期被检测出来，违反了条款 18 —— 好的接口可以防止无效的代码通过编译。



### 条款 33 —— 避免遮掩继承而来的名称

内层作用域中的名称会遮掩外层作用域中的名称。考虑下面代码

```cpp
class Base {
private:
  int x;

public:
  virtual void mf1() = 0;
  virtual void mf1(int) {}
  virtual void mf2() {}
  void mf3() {}
  void mf3(double) {}
};

class Derived : public Base {
public:
  virtual void mf1() {}
  void mf3() {}
  void mf4() {}
};

int main() {
  Derived d;
  int x = 0;
  d.mf1();
  d.mf1(x); // 报错，此函数被子类遮蔽
  d.mf2();
  d.mf3();
  d.mf3(x); // 报错，此函数被子类遮蔽

  return 0;
}
```

注意到任何同名的函数都会被子类遮蔽。如果希望子类使用基类中参数类型不同的同名函数，通过 `using` 声明

```cpp
class Derived : public Base {
public:
  using Base::mf1;	// 使用 Base::mf1(int)
  using Base::mf3;	// 使用 Basie::mf3(double)
  virtual void mf1() {}
  void mf3() {}
  void mf4() {}
};
```

注意只能使用可以视为重载的基类函数，而例如 `Base::mf3()` 就不能使用。



有时我们会使用 private 继承，而 `using` 声明会使得所有同名函数都在子类中可见。如果只希望继承其中一个函数，可以使用转交函数 forwarding function

```cpp
class Derived : private Base {
public:
  virtual void mf1() { Base::mf1(10); }
  void mf3() {}
  void mf4() {}
};
```



### 条款 34 —— 区分接口继承和实现继承

#### 接口继承

接口继承即纯虚函数，子类只继承基类的函数接口，并且必须重写基类的函数接口。纯虚函数可以提供定义

```cpp
class Shape {
public:
  virtual void draw() const = 0;
};

// 定义纯虚函数
void Shape::draw() const { std::cout << "Shape::draw()" << std::endl; }

class Rectangle : public Shape {
public:
  void draw() const override { std::cout << "Rectangle::draw()" << std::endl; }
};

int main() {
  Rectangle r;
  Shape *sh = &r;
  sh->draw();        // 调用 Rectangle::draw()
  sh->Shape::draw(); // 调用 Shape::draw()
  return 0;
}
```



#### 实现继承

非纯虚函数用于实现继承，但是子类不必重写基类函数。这可能造成问题：例如新的子类在设计中可能忘记重写基类函数

```cpp
class AirPlane {
public:
  virtual void fly(const string &dest) {
    std::cout << "fly to China" << std::endl;
  }
};

class ModelA : public AirPlane {
public:
  void fly(const string &dest) override {
    std::cout << "fly to America" << std::endl;
  }
};

class ModelB : public AirPlane {
public:
};
```

这里子类 `ModelB` 直接使用了默认的飞行方式，这种默认继承可能不是我们想要的。但是我们又希望提供一个默认的继承，只不过能够提醒我们不要忘记重写继承，这时可以使用纯虚函数并额外提供一个默认函数，由子类调用

```cpp
class AirPlane {
public:
  virtual void fly(const string &dest) = 0;

protected:
  void default_fly() { std::cout << "fly to China" << std::endl; }
};

class ModelA : public AirPlane {
public:
  void fly(const string &dest) override {
    std::cout << "fly to America" << std::endl;
  }
};

class ModelB : public AirPlane {
public:
  void fly(const string &dest) override { default_fly(); }
};
```



使用纯虚函数定义可以更好地解决这种问题

```cpp
class AirPlane {
public:
  virtual void fly(const string &dest) = 0;
};

// 实现纯虚函数
void AirPlane::fly(const string &dest) {
  std::cout << "fly to China" << std::endl;
}

class ModelA : public AirPlane {
public:
  void fly(const string &dest) override {
    std::cout << "fly to America" << std::endl;
  }
};

class ModelB : public AirPlane {
public:
  // 子类直接调用基类纯虚函数
  void fly(const string &dest) override { AirPlane::fly(dest); }
};
```



### 条款 35 —— 考虑虚函数以外的其它选择

假设我们需要为一个游戏人物提供获得健康度的函数，提供一个虚函数是最直接的选择

```cpp
class GameCharacter {
public:
  virtual int health() const { return 100 - age; }

private:
  int age;
};
```

但是我们也可以考虑其它替代方案。



#### 模板模式

一种流派认为所有虚函数都应该私有。可以保留健康度函数为非虚函数，它调用私有的虚函数计算接口

```cpp
class GameCharacter {
public:
  int health() const { return compute_helath(); }

private:
  virtual int compute_helath() const { return 100 - age; }

private:
  int age;
};
```

这种让用户通过公共非虚函数间接调用私有虚函数的方法，称为 non-virtual interface (NVI) 手法。它的好处在于可以在调用虚接口前后提供额外的操作空间，例如

```cpp
int health() const { 
	// do something
	int h = compute_helath();
	// do something
	return h; 
}
```

我们将 `health` 函数称为虚函数的外覆器 wrapper，可以提供事前线程锁定、输出日志，事后解除锁定等操作。如果直接使用虚函数，就需要在每个子类虚函数中定义这些操作，而外覆器直接在基类中统一提供，简化了操作。



#### 策略模式

我们可以将操作封装为一个函数，作为参数传入来调用，也就是一种策略模式。例如

```cpp
class GameCharacter {
public:
  void setHealthFunc(std::function<int(const GameCharacter *)> func) {
    healthFunc = func;
  }

  int health() const { return healthFunc(this); }

private:
  int age;
  std::function<int(const GameCharacter *)> healthFunc;
};
```



### 条款 36 —— 绝不重新定义继承来的非虚函数

非虚函数都是静态绑定，在继承体系中无法应用多态。



### 条款 37 —— 绝不重新定义继承来的默认参数值

虚函数是动态绑定，然而默认参数值却是静态绑定，在继承体系中容易造成混乱。例如

```cpp
class Shape {
public:
  enum Color { RED, GREEN, BLUE };

  virtual void draw(Color c = RED) const {
    std::cout << "Drawing a shape in color: " << c << std::endl;
  }
};

class Rectangle : public Shape {
public:
  void draw(Color c = BLUE) const override {
    std::cout << "Drawing a rectangle in color " << c << std::endl;
  }
};

int main() {
  Rectangle r;
  Shape *sh = &r;
  sh->draw();
  return 0;
}
```

由于默认参数值静态绑定，因此 `sh->draw()` 会使用基类的默认参数值。

>[!note]
>为什么 C++ 要将默认参数值设为静态绑定？因为编译期决定要比运行期决定更快且更简单。



### 条款 38 —— 通过复合塑模出 `has-a` 或“根据某物实现出”

复合 composition 表示一种类型包含另一种类型的对象，区别于继承表示的“是一种”关系。假设想要实现一个 `Set` 类型，确保更低的空间复杂度，那么一种选择是使用标准库的 `list` 类型作为基础。如果直接让 `Set<T>` 继承 `list<T>`，问题在于 `list<T>` 并不是一个 `Set`，它可以有重复的元素，这不符合"is-a"关系。因此更好的方式是将 `list<T>` 放在 `Set<T>` 内部，使用复合实现。



### 条款 39 —— 明智而审慎地使用 private 继承

#### 私有继承的含义

在 private 继承中，基类成员在子类中是私有的，所以外部无法访问。这意味着子类不能被转换为基类使用，例如

```cpp
class Person {};
class Student : public Person {};

void eat(const Person &p) { std::cout << "Person eating" << std::endl; }
void study(const Student &s) { std::cout << "Student studying" << std::endl; }

int main() {
  Student s;
  eat(s);	// 调用失败

  return 0;
}
```

使用 private 继承意味着只继承实现部分（定义）而不继承接口部分（不能调用），因此仅表示子类通过基类实现得到。



#### 复合还是私有继承

注意到 private 继承似乎与复合表达相同的含义，那么何时使用复合、何时使用 private 继承？答案是几乎任何时候都应该使用复合。假设需要为一个窗口类提供计时器，使用 private 继承的方案是

```cpp
class Timer {
public:
  virtual void on_timeout() = 0;
  // ...
};

class Widget : private Timer {
public:
  void on_timeout() override {
    std::cout << "Widget timeout" << std::endl;
  }
  // ...
};
```

在窗口类中重写计时函数实现相应的功能，使用私有继承是为了防止外部调用此函数，毕竟窗口类本身似乎不应该具有计时功能。



但是这带来的问题就是：新的窗口子类可能重写这个计时函数。使用复合就不会有这类问题

```cpp
class Timer {
public:
  virtual void on_timeout() = 0;
  // ...
};

class Widget {
private:
  class WidgetTimer : public Timer {
    void on_timeout() override {
      std::cout << "Widget timeout" << std::endl;
    }
  };

private:
  WidgetTimer timer;

public:
  // ..
};
```

由于 `WidgetTimer` 作为私有类型，窗口子类不可能访问这个成员类型。并且复合方法允许我们将 `WidgetTimer` 定义在另一个源文件中，在这里只保留一个声明，从而减少编译依赖。

>[!note]
>由于 C++ 引入了 `final` 关键字，因此也可以通过为 `on_timeout` 提供 `final` 修饰来阻止子类重写。



#### 空间最优化

有一种空间最优化的激进情形可能会考虑使用 private 继承。假设需要在一个类中使用另一个空类

```cpp
class Adder {
public:
  int compute(int a, int b) { return a + b; }
};

class Something {
private:
  int x;
  Adder adder;

public:
  Something(int x) : x(x) {}
  int add(int y) { return adder.compute(x, y); }
};
```

由于技术原因，C++ 要求空类至少占一个字节，因此 `Adder` 将会占用空间；考虑到字节对齐，`Something` 需要占用 8 字节空间。而使用 private 继承就可以触发 EBO 机制（空基类优化）

```cpp
class Adder {
public:
  int compute(int a, int b) { return a + b; }
};

class Something : private Adder {
private:
  int x;

public:
  Something(int x) : x(x) {}
  int add(int y) { return compute(x, y); }
};
```

此时 `Something` 只占用 4 字节。



### 条款 40 —— 明智而审慎地使用多重继承

多重继承通常容易造成问题：例如继承接口重名还有经典的钻石继承。对于多继承，要避免钻石继承中顶层类成员被继承多份的问题，public 继承应该总是 virtual，即虚继承。例如

```cpp
class File {};
class InputFile : virtual public File {};
class OutputFile : virtual public File {};
class IOFile : public InputFile
				public OutputFile
				{};
```

不过考虑到虚继承的额外开销，以及子类总是要承担对基类的初始化问题，有两点原则

1. 非必要不使用虚继承
2. 如果必须使用虚继承，尽可能避免在基类中存放数据

在此基础上，在接口类可以通过另一个类便捷地实现时，可以使用多重继承。例如给定接口类

```cpp
class IPerson {
public:
  virtual ~IPerson();
  virtual std::string name() const = 0;
};
```

我们希望实现一个子类，并且已经有一个类提供了类似的功能

```cpp
class PersonInfo {
public:
  explicit PersonInfo(const char *name) { std::strcpy(this->name, name); }
  virtual ~PersonInfo() {}
  
  virtual const char *the_name() const {
    static char buf[255];
    std::strcpy(buf, before());
    std::strcat(buf, name);
    std::strcat(buf, after());
    return buf;
  }

private:
  virtual const char *before() const { return "["; }
  virtual const char *after() const { return "]"; }

private:
  char name[255];
};
```

这个类能够保存人名，并在人名前后添加修饰符后返回。这种情况下，就可以选择多继承

```cpp
class CPerson : public IPerson, private PersonInfo {
public:
  explicit CPerson(const char *name) : PersonInfo(name) {}
  std::string name() const override { return the_name(); }

  // 取消前后修饰符
  const char *before() const override { return ""; }
  const char *after() const override { return ""; }
};
```

接口类可以选择 public 继承，实现类选择 private 继承（塑模“实现出”关系）。



## 模板与泛型编程

### 条款 41 —— 了解隐式接口和编译期多态

考虑下面的多态代码

```cpp
class Widget {
public:
  virtual std::size_t size() const { return 0; }
  virtual void normalize() {}
};

void do_something(Widget &w) {
  if (w.size() > 10) {
    w.normalize();
  }
}
```

其中 `w` 可能表现出运行期多态。而对于模板编程，可以转换为

```cpp
template <typename T> void do_something_else(T &w) {
  if (w.size() > 10) {
    w.normalize();
  }
}
```

这就是所谓的编译期多态，它在编译期确定传入类型是否具有 `size(), normalize()` 接口。



### 条款 42 —— 了解 `typenmae` 的双重意义

在声明模板类型时，使用 `typename` 和 `class` 是等价的。然而，如果涉及到模板类型的内部类型，就应当使用 `typename` 修饰。例如

```cpp
template <typename C> class A {
  C::const_iterator *ptr;
};
```

这种写法会产生歧义：如果 `C::const_iterator` 是静态变量，那么 `*` 可能会被理解为乘号。因此需要添加修饰

```cpp
template <typename C> class A {
  typename C::const_iterator *ptr;
};
```



#### 返回嵌套

另一种情况是嵌套类型作为返回类型

```cpp
template <typename T> struct A {
  struct B {};
  B b;
  B *re();
};

template <typename T> A<T>::B *A<T>::re() { return &b; }
```

这里 `A<T>::B *` 返回类型也需要添加修饰。



#### 参数嵌套

再例如嵌套类型作为参数类型时

```cpp
template <typename C> void f(const C &container, typename C::iterator iter) {}
```



#### 基类嵌套

假设继承某个模板类中的嵌套类型，语法要求继承和初始化列表中不用使用 `typename` 修饰。例如

```cpp
template <typename T> class Base {
public:
  struct Nested {
    int x;
  };
};

// public 后面不用 typename
template <typename T> class Derived : public Base<T>::Nested {
public:
  // Base<T>::Nested(x) 前不用 typename
  explicit Derived(int x) : Base<T>::Nested(x) { 
    typename Base<T>::Nested tmp; 
  }
};
```



#### Typedef

使用 `typedef, using` 定义别名时，也需要添加修饰

```cpp
// 这里其实不用 typename 也行
template <typename T> void work(T t) {
  typedef typename std::iterator_traits<T>::value_type value_type;
  value_type v(*t);
}

template <typename T>
using value_type = typename std::iterator_traits<T>::value_type;
```



### 条款 43 —— 学习处理模板化基类内的名称

>[!note]
>似乎我所使用的编译器不会出现这里提到的问题。

考虑下面的代码，我们为不同的 `Company` 提供 `MsgSender` 模板基类，然后在外面又包装一层模板日志类。在日志类中调用 `MsgSender` 中的函数，这时候由于继承的是模板类，不能确定 `MsgSender` 中是否具有 `send_clear` 成员，因此编译不能通过。

```cpp
class CompanyA {
public:
  void send_clear_text(const std::string &str) {
    std::cout << "Sending clear text message to CompanyA: " << str << std::endl;
  }
};

class CompanyB {
public:
  void send_clear_text(const std::string &str) {
    std::cout << "Sending clear text message to CompanyB: " << str << std::endl;
  }
};

template <typename Company> class MsgSender {
public:
  void send_clear(const std::string &str) {
    Company c;
    c.send_clear_text(str);
  }
};

// 全特化，没有 send_clear
template <> class MsgSender<CompanyA> {};

template <typename Company> class LoggingMsgSender : public MsgSender<Company> {
public:
  void send_clear_msg(const std::string &str) {
    // logging
    send_clear(str);
    // logging
  }
};
```



要解决这一问题，一种方案是调用模板基类函数时使用指针

```cpp
this->send_clear(str);
```

第二种是使用 `using` 声明

```cpp
template <typename Company> class LoggingMsgSender : public MsgSender<Company> {
public:
  // 告诉编译器假设基类中有 send_clear
  using MsgSender<Company>::send_clear;
  
  void send_clear_msg(const std::string &str) {
    // logging
    send_clear(str);
    // logging
  }
};
```

第三种是显示提供基类名称

```cpp
MsgSender<Company>::send_clear(str);
```

但这种方案有很大风险，如果 `send_clear` 是一个虚函数，这种显式调用会取消多态特性。



### 条款 44 —— 将与参数无关的代码抽离模板

模板原本用于节省时间和避免代码重复。然而，有时候可能会生成不必要的重复代码。例如

```cpp
template <typename T, std::size_t N> class SquareMatrix {
public:
  void invert() {}
};
```

每当我们对不同 `T,N` 定义不同 ` SquareMatrix<T, N>` 变量时，用于求逆的函数都会多生成一份，尽管每份调用过程是完全相同的。



首先可以考虑将 `N` 分离出去，先构造一个包含 `invert` 的基类

```cpp
template <typename T> class SquareMatrixBase {
protected:
  // 它接收尺寸，这样子类实现中可以忽略尺寸
  void invert(std::size_t n) {}
};

// 子类通过基类实现，只继承方法不继承接口，因此使用 private 继承
template <typename T, std::size_t N>
class SquareMatrix : private SquareMatrixBase<T> {
  // 避免覆盖基类函数
  using SquareMatrixBase<T>::invert;

public:
  void invert() { this->invert(N); }
};
```

然后需要考虑子类如何访问数据：矩阵规模可以在编译期决定，并且位于子类；然而需要注意基类 `invert` 调用又需要获得矩阵数据，所有还要再基类中提供访问数据的方法

```cpp
template <typename T> class SquareMatrixBase {
protected:
  SquareMatrixBase(std::size_t n, T *p) : m_pData(p), m_size(n) {}
  void invert(std::size_t n) {}

private:
  T *m_pData;
  std::size_t m_size;
};

template <typename T, std::size_t N>
class SquareMatrix : private SquareMatrixBase<T> {
  using SquareMatrixBase<T>::invert;

public:
  SquareMatrix() : SquareMatrixBase<T>(N, m_data) {}
  void invert() { this->invert(N); }

private:
  T m_data[N * N];
};
```

当然也可以使用动态内存。



### 条款 45 —— 运用成员函数模板接受所有兼容类型

假设需要将一种智能指针隐式转换为另一种智能指针。例如

```cpp
template <typename T> class SmartPointer {
public:
  SmartPointer() : ptr(nullptr) {}

private:
  T *ptr;
};

class Base {};
class Derived : public Base {};

SmartPointer<Derived> p1;
SmartPointer<Base> p2(p1);
```

虽然 `Derived*` 可以直接转换为 `Base*`，然而一旦作为模板类型，就不能进行转换。这时候就需要设置成员函数模板

```cpp
template <typename T> class SmartPointer {
public:
  SmartPointer() : ptr(nullptr) {}

  template <typename U>
  SmartPointer(const SmartPointer<U> &other) : ptr(other.get()) {}

  T *get() const { return ptr; }

private:
  T *ptr;
};
```




### 条款 46 —— 需要类型转换时请为模板定义非成员函数

我们将条款 24 中的有理数变为模板类，对应的乘法操作也变为模板类

```cpp
template <typename T> class Rational {
public:
  Rational(T n = 0, T d = 1) : n(n), d(d) {}

  T numerator() const { return n; }
  T denominator() const { return d; }

private:
  T n;
  T d;
};

template <typename T>
const Rational<T> operator*(const Rational<T> &lhs, const Rational<T> &rhs) {
  return Rational<T>(lhs.numerator() * rhs.numerator(),
                     lhs.denominator() * rhs.denominator());
}
```

现在对整型执行乘法将会报错

```cpp
  Rational<int> r(1, 2);
  Rational<int> result = r * 2;
```

原先没有使用模板时，编译器可以将 `2` 隐式转换为 `Rational`，然而添加模板后，不能判断实现哪个模板乘法，转换为哪个模板类 `Rational<T>` 。



我们可以将乘法函数放进模板类中，这样 `r * 2` 就会调用对应这个模板类的乘法函数

```cpp
template <typename T> class Rational {
public:
  Rational(T n = 0, T d = 1) : n(n), d(d) {}

  T numerator() const { return n; }
  T denominator() const { return d; }

  friend const Rational operator*(const Rational &lhs, const Rational &rhs) {
    return Rational(lhs.numerator() * rhs.numerator(),
                    lhs.denominator() * rhs.denominator());
  }

private:
  T n;
  T d;
};
```

>[!note]
>由于定义在类中的函数都是内联函数，友元函数很复杂，就可能会增加类中函数的体积。一种做法就是将乘法实现放在外面，然后在类内 `operator*` 中调用乘法实现，这样内联后只会增加一行代码。



### 条款 47 —— 请使用 traits classes 表现类型信息

STL 主要由“用以表现容器、迭代器和算法”的模板构成，但也提供工具性模板。例如

```cpp
template <typename IterT, typename DistT> void advance(IterT &iter, DistT d);
```

它提供对迭代器的步进/退操作。回顾一下几种迭代器类型

- Input 迭代器。只能向前读一次，一次一步
- Output 迭代器。只能向前写一次，一次一步
- Forward 迭代器。只能向前读写，一次一步
- Bidirectional 迭代器。可以向前或向后，一次一步
- Random access 迭代器。可以向前或向后，任意跳步

对于几种迭代器，标准库都提供了标记结构 tag struct

```cpp
struct input_iterator_tag {};
struct output_iterator_tag {};
struct forward_iterator_tag : public input_iterator_tag {};
struct bidirectional_iterator_tag : public forward_iterator_tag {};
struct random_access_iterator_tag : public bidirectional_iterator_tag {};
```

假设我们要自己实现 `advance`，就需要检查输入迭代器的类型来决定执行的操作。标准库技术会提供一个模板 `iterator_traits` 来提取迭代器类型

```cpp
template <typename IterT> struct iterator_traits {
  using iterator_category = typename IterT::iterator_category;
};

// 针对内置指针偏特化，提取迭代器为 Random
template <typename T> struct iterator_traits<T *> {
  using iterator_category = random_access_iterator_tag;
};
```

标准容器中的迭代器会包含一个内置类型 `iterator_category`

```cpp
template <typename T> class vector {
public:
  class iterator {
  public:
    using iterator_category = random_access_iterator_tag;
  };
};
```

于是在 `advance` 内部可以通过 `iterator_traits` 获得迭代器类型

```cpp
template <typename IterT, typename DistT> void advance(IterT &iter, DistT d) {
  iterator_traits<IterT>::iterator_category cat;
  // 执行类型判断 ...
}
```

在古老的版本中，借助函数重载可以对不同类型执行不同操作

```cpp
template <typename IterT, typename DistT> void advance(IterT &iter, DistT d) {
  iterator_traits<IterT>::iterator_category cat;
  do_something(iter, d, cat);
}

template <typename IterT, typename DistT>
void do_something(IterT &iter, DistT d, forward_iterator_tag) {}

template <typename IterT, typename DistT>
void do_something(IterT &iter, DistT d, random_access_iterator_tag) {}
```



### 条款 48 —— 认识模板元编程

Template metaprogramming (TMP) 模板元编程是编写 template-based C++ 程序并执行于编译期的过程。TMP 已被证明是个“图灵完备”机器。



## 定制 `new` 和 `delete`

### 条款 49 —— 了解 `new-handler` 的行为

当程序无法分配内存时，将会抛出异常。用户可以指定错误处理函数，程序会先调用该函数

```cpp
void out_of_mem() {
  std::cerr << "Out of memory" << std::endl;
  std::abort();
}

int main() {
  // 设置当 new 异常时调用的函数，该函数会返回旧的 new-handler 函数
  std::set_new_handler(out_of_mem);
  int *big = new int[10000000000000L];
  delete[] big;
  return 0;
}
```

当 `operator new` 无法满足内存申请时，将会不断调用 `new-handler` 函数，直到找到足够内存。



考虑为一个类设计 `new-handler` 函数。首先利用 RAII 技术来保存 `new-handler` 对象

```cpp
class NewHandlerHolder {
public:
  explicit NewHandlerHolder(std::new_handler nh) : handler(nh) {}
  ~NewHandlerHolder() { std::set_new_handler(0); }

private:
  std::new_handler handler;

  NewHandlerHolder(NewHandlerHolder &&) = delete;
};
```

当离开作用域后自动释放 `new-handler` 对象。然后定义 `set_new_handler` 和 `new[]` 方法

```cpp
class Widget {
public:
  // 设置并记录当前类的 new-handler，返回旧的 new-handler
  static std::new_handler set_new_handler(std::new_handler handler) {
    std::swap(currentHandler, handler);
    return handler;
  }

  // 单独的 new 输入参数 size 为 Widget 的字节数
  static void *operator new(std::size_t size) {
    std::cout << "Allocating " << size << " bytes" << std::endl;
    return ::operator new(size);
  }

  // 绑定当前类的 new-handler，new 完成后自动释放 new-handler
  static void *operator new[](std::size_t size) {
    NewHandlerHolder holder(std::set_new_handler(currentHandler));
    return ::operator new[](size);
  }

private:
  static std::new_handler currentHandler;
};

std::new_handler Widget::currentHandler = 0;
```

使用流程为

```cpp
void out_of_mem() {
  std::cerr << "Out of memory" << std::endl;
  std::abort();
}

int main() {
  Widget::set_new_handler(out_of_mem);
  Widget *w = new Widget[10000000000000L];
  delete[] w;
  return 0;
}
```



更方便的写法是设计一个模板 RAII 类同时提供 `set_new_handler, operator new[]` 方法

```cpp
template <typename T> class NewHandlerSupport {
public:
  static std::new_handler set_new_handler(std::new_handler handler) {
    std::swap(currentHandler, handler);
    return handler;
  }

  static void *operator new(std::size_t size) {
    std::cout << "Allocating " << size << " bytes" << std::endl;
    return ::operator new(size);
  }

  static void *operator new[](std::size_t size) {
    NewHandlerHolder holder(std::set_new_handler(currentHandler));
    return ::operator new[](size);
  }

private:
  static std::new_handler currentHandler;
};

// 模板 T 的作用是对每种 T 构造不同的静态变量
template <typename T> std::new_handler NewHandlerSupport<T>::currentHandler = 0;

class Widget : public NewHandlerSupport<Widget> {
public:
  // ...
};
```

需要支持 `new-handler` 操作的类直接继承对应的模板类。

>[!note]
>将 `Widget` 作为模板类型的做法是一个称为 curiously recurring template pattern (CRTP) 的技术。它要求模板为子类专门生成一个类，这里用处是产生专门的 `currentHandler` 变量对应子类，防止不同类型的静态变量相互覆盖。



### 条款 50 —— 了解 `new` 和 `delete` 的合理替换时机

通常在如下情形可能需要替换编译器提供的 `new, delete` 操作

- 检测运用错误。例如 `new` 指针被 `delete` 两次，为了检测错误，可能为 `new` 操作分配额外的内存空间用于标记，当出现多次删除时标记会被移除，检查标记是否存在就可以判断是否多次删除。
- 强化性能。频繁内存分配或许会造成碎片化内存以及申请和释放的开销，自定义 `new, delete` 可能统一分配内存和释放内存。
- 收集内存分配数据。



### 条款 51 —— 编写 `new` 和 `delete` 时需固守常规

#### `new`

典型的 `new` 函数应该形如

```cpp
void *operator new(size_t size) {
  if (size == 0)
    size = 1;

  while (true) {
    // 分配内存
    void *ptr = malloc(size);
    if (ptr)
      return ptr;

    // 分配失败，获得当前的 new-handler
    std::new_handler handler = std::set_new_handler(0);
    std::set_new_handler(handler);

	// 调用 new-handler 函数
    if (handler)
      (*handler)();
    else
      throw std::bad_alloc();
  }
}
```

其中每当分配失败，都会循环调用 `new-handler` 函数直到正确分配或者不存在 `new-handler` 函数。因此需要注意

- 自定义的 `new-handler` 中必须执行某些操作确保分配失败后可以退出循环
- 注意在类内定义 `new` 时，接收的 `size` 参数只与当前类有关，一旦继承可能出错



#### `delete`

唯一需要注意的就是保证“删除 `null` 指针永远安全”。形如

```cpp
void operator delete(void *rawMem) {
  if (rawMem == nullptr)
    return;

  free(rawMem);
}
```



### 条款 52 —— 配对 `placement new` 和 `placement delete`

考虑一个简单的 `new` 操作

```cpp
Widget* p = new Widget;
```

共有 `operator new, Widget()` 两个函数调用。假如 `new` 调用成功而 `Widget()` 调用失败，那么程序需要释放掉分配的内存防止内存泄漏。然而用户显然不能释放掉该内存，因为 `p` 没有赋值，所以由 C++ 处理释放过程。



通常 C++ 会自动匹配 `new, delete`，但有一种情况例外，就是使用 `placement new` 分配的情况。考虑带有专属 `new` 操作的类

```cpp
class Widget {
public:
  static void *operator new(std::size_t size, std::ostream &log) {}
};
```

其中 `new` 操作除了 `size` 外还提供一个额外参数用于输出日志，称为 `placement new` 。标准库中常见的 `placement new` 形如

```cpp
void *operator new(std::size_t, void*);
```

它允许指定一个内存位置执行 `new` 操作。例如

```cpp
// 在 &n 位置 new
int n = 100;  
Base *p = new (&n)Base; 

Vec *U = new Vec[s];  
for (int i = 0; i < s; i++)  
{  
    // 在 U + i 处调用 new 及构造函数初始化  
    new (U + i) Vec(n, 0);  
}
```



对于带有 `placement new` 的类，也必须提供对应的 `placement delete` 操作

```cpp
class Widget {
public:
  static void *operator new(std::size_t size, std::ostream &log) {}
  static void operator delete(void *p, std::ostream &log) {}
};

// placement new
Widget* p = new (std::err) Widget;
```

这样当构造失败时，就能够调用对应的 `delete` 方法。如果正常分配，正常删除

```cpp
Widget* p = new (std::err) Widget;
delete p;
```

此时 `delete` 调用正常的 `delete` 方法而非 `placement new` 方法。

>[!note]
>注意为类定义 `placement new` 时还需要定义标准的 `new`，防止正常的 `new` 无法调用。



# Effective Modern C++

## 型别推导

### 条款 1 —— 理解模板型别推导

#### 模板实参

考虑下面几种形式的模板函数

```cpp
template <class T> void f1(T t) {}
template <class T> void f2(const T &t) {}
template <class T> void f3(T &t) {}
template <class T> void f4(T &&t) {}
```

在传入如下不同类型参数时

```cpp
1;	// 传 int&&
int x = 1;
const int& cx = x;
int &rx = x;
```

考虑模板类型 `T` 的推导：

- 对于 `f1` 按值传入，因此所有参数传入时 `T -> int`；
- 对于 `f2` 传常量引用，无法传入 `int&&`；其余均有 `T -> int`；
- 对于 `f3` 传引用，无法传入 `int&&`；传入 `x, rx` 时 `T -> int`，传入 `cx` 时 `T -> const int`；
- 对于 `f4` 传右值引用，传入 `1` 时 `T -> int`；传入 `x, rx` 时 `T -> int&`；传入 `cx` 时 `T -> const int&`；

需要注意按值传递时，传入类型自身的常量属性会被消除。例如

```cpp
int x = 10;
const int *const p = &x; 
```

当 `p` 作为值传入时，类型只会剩下 `const int *`，作用于指针自身的常量属性被去除，然后可以指向其它整型地址。



#### 数组实参

通常数组作为参数传入时会被退化为指针。例如

```cpp
template <class T> f(T param);

const char name[] = "Li";
f(name);	// T 被推导为 const char *
```

然而如果模板函数是引用参数，数组会被推导为真正的类型

```cpp
template <class T> f(T &param);	// T 被推导为

const char name[] = "Li";
f(name);	// T 被推导为 const char [2]
```

数组类型由其元素类型和长度决定。利用这一特性，可以在编译期计算数组长度

```cpp
template <class T, std::size_t N>
constexpr std::size_t array_size(T (&)[N]) noexcept {
  return N;
}

int keys[] = [1, 2, 3];
int vals[array_size(keys)];
```

使用 `constexpr` 保证返回值可以在编译期使用；使用 `noexcept` 生成更高效的目标码。



#### 函数实参

当函数作为参数传入时，也会退化为函数指针或引用。例如

```cpp
void some(int, double);

template <class T>
void f1(T param);

template <class T>
void f2(T& param);
```

通过 `f1, f2` 传入 `some` 分别推导 `T` 为 `void(*)(int, double), void(&)(int, double)` 。



### 条款 2 —— 理解 `auto` 型别推导



### 条款 3 —— 理解 `decltype`



### 条款 4 —— 掌握查看型别推导结果的方法



## Auto

### 条款 5 —— 优先选用 `auto`，而非显式型别声明



### 条款 6 —— 当 `auto` 推导的型别不符合要求时，使用带显式型别的初始化


