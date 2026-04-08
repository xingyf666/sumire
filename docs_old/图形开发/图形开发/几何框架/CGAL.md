# CGAL

## 基本介绍

### 安装编译

在官方文档中找到 [GitHub 链接](https://github.com/CGAL/cgal/releases)下载 `CGAL-5.6.zip` 和 GMP and MPFR libraries, for Windows 64bits 两个文件。将后者的 auxiliary/gmp 文件夹复制到前者的 auxiliary 文件夹下即可。



确保 Boost 库已安装，现在就可以创建测试文件。在主目录下创建

```cmake
cmake_minimum_required(VERSION 3.18)
project(MAIN)

# 增加子目录 src，以及目标文件夹 bin
add_subdirectory(src bin)
```

然后在 src 目录下创建

```cmake
# 确定 Boost 库和 CGAL 库路径
set(Boost_DIR "D:/lib/Boost/lib/cmake/Boost-1.82.0")
set(CGAL_DIR "D:/lib/CGAL-5.6")

# 查找 CGAL 库，会自动链接 Boost 库
find_package(CGAL REQUIRED)

add_executable(main main.cpp)

# 链接 CGAL 库，添加动态库路径到调试环境
target_link_libraries(main CGAL::CGAL)
set_target_properties(main PROPERTIES VS_DEBUGGER_ENVIRONMENT "PATH=D:/lib/CGAL-5.6/auxiliary/gmp/lib;%PATH%")
```

提供 src 目录下的 `main.cpp` 文件

```cpp
#include <iostream>
#include <CGAL/Exact_predicates_exact_constructions_kernel.h>

typedef CGAL::Exact_predicates_exact_constructions_kernel Kernel;
typedef Kernel::Point_3 CGPoint3;

using namespace std;

int main()
{
	double x = 2.21231224, y = 3.321443645, z = 4.12335465;
	CGPoint3 pnt(x, y, z);
	cout << pnt << endl;

	double x1 = CGAL::to_double(pnt.x());
	cout << x1 << endl;

	return 0;
}
```

最后通过 cmake 编译 VS 项目即可。



### 点和线段

所有头文件都位于 CGAL 目录下，我们先定义基本类型 kernel，通过实例化模板得到点和线段类型，并进行简单的操作。

```cpp
#include <iostream>
#include <CGAL/Simple_cartesian.h>

// 定义基本类型为双精度浮点
typedef CGAL::Simple_cartesian<double> Kernel;

// 利用基本类型得到点和线段类型
typedef Kernel::Point_2 Point_2;
typedef Kernel::Segment_2 Segment_2;

int main()
{
    // 构造平面点
    Point_2 p(1, 1), q(10, 10);
    std::cout << "p = " << p << std::endl;
    std::cout << "q = " << q.x() << " " << q.y() << std::endl;

    // 计算平方距离
    std::cout << "sqdist(p,q) = " << CGAL::squared_distance(p, q) << std::endl;

    // 构造线段
    Segment_2 s(p, q);
    Point_2 m(5, 9);

    std::cout << "m = " << m << std::endl;

    // 点到线段的距离
    std::cout << "sqdist(Segment_2(p,q), m) = "  << CGAL::squared_distance(s, m) << std::endl;
    std::cout << "p, q, and m ";

    // 判定 p, q, m 之间的关系：共线、逆时针、顺时针
    switch (CGAL::orientation(p, q, m)) {
    case CGAL::COLLINEAR:
        std::cout << "are collinear\n";
        break;
    case CGAL::LEFT_TURN:
        std::cout << "make a left turn\n";
        break;
    case CGAL::RIGHT_TURN:
        std::cout << "make a right turn\n";
        break;
    }

    // 计算中点
    std::cout << " midpoint(p,q) = " << CGAL::midpoint(p, q) << std::endl;

    return 0;
}
```



### 舍入误差

正如所有浮点计算过程一样，在判断点是否共线时会有浮点误差导致结果与预期不符。例如下面三种情况都应该得到共线，但只有最后一种整型输入能够得到正确的结果。

```cpp
#include <iostream>
#include <CGAL/Simple_cartesian.h>

typedef CGAL::Simple_cartesian<double> Kernel;
typedef Kernel::Point_2 Point_2;

int main()
{
    {
        Point_2 p(0, 0.3), q(1, 0.6), r(2, 0.9);
        std::cout << (CGAL::collinear(p, q, r) ? "collinear\n" : "not collinear\n");
    }
    {
        Point_2 p(0, 1.0 / 3.0), q(1, 2.0 / 3.0), r(2, 1);
        std::cout << (CGAL::collinear(p, q, r) ? "collinear\n" : "not collinear\n");
    }
    {
        Point_2 p(0, 0), q(1, 1), r(2, 2);
        std::cout << (CGAL::collinear(p, q, r) ? "collinear\n" : "not collinear\n");
    }
    return 0;
}
```



这时候我们建议使用 Exact 构造精确谓词和精确 Kernel 进行判断。然而下面三种共线情况只有后两种能得到正确的结果。

```cpp
#include <iostream>
// exact + exact
#include <CGAL/Exact_predicates_exact_constructions_kernel.h>
#include <sstream>

typedef CGAL::Exact_predicates_exact_constructions_kernel Kernel;
typedef Kernel::Point_2 Point_2;

int main()
{
    Point_2 p(0, 0.3), q, r(2, 0.9);
    {
        q = Point_2(1, 0.6);
        std::cout << (CGAL::collinear(p, q, r) ? "collinear\n" : "not collinear\n");
    }
    {
        std::istringstream input("0 0.3   1 0.6   2 0.9");
        input >> p >> q >> r;
        std::cout << (CGAL::collinear(p, q, r) ? "collinear\n" : "not collinear\n");
    }
    {
        q = CGAL::midpoint(p, r);
        std::cout << (CGAL::collinear(p, q, r) ? "collinear\n" : "not collinear\n");
    }
    return 0;
}
```

这是因为文本输入后变为浮点型再进行构造，得到的结果就产生了误差；然而通过文本读取直接构造就能得到完全精确的值，通过 midpoint 算法得到的结果也是完全精确的。



### 凸包算法

由于凸包算法只比较输入点的坐标和方向测试，**不会产生新的点**，使用**精确谓词但不进行精确构造**也是可以的。下面的 2 维凸包算法返回 result 数组中最后一个凸包上的点后面一位的地址，因此它与 result 作差就得到凸包上点的个数。

```cpp
#include <iostream>
// exact + inexact
#include <CGAL/Exact_predicates_inexact_constructions_kernel.h>
#include <CGAL/convex_hull_2.h>

typedef CGAL::Exact_predicates_inexact_constructions_kernel K;
typedef K::Point_2 Point_2;

int main()
{
    Point_2 points[5] = { Point_2(0,0), Point_2(10,0), Point_2(10,10), Point_2(6,5), Point_2(4,1) };
    Point_2 result[5];
    Point_2* ptr = CGAL::convex_hull_2(points, points + 5, result);
    std::cout << ptr - result << " points on the convex hull:" << std::endl;
    for (int i = 0; i < ptr - result; i++) {
        std::cout << result[i] << std::endl;
    }
    return 0;
}
```



也可以使用 STL 容器执行算法，先输入向量的迭代器范围，最后要输入一个 output iterator，这里 result 由于是空容器，因此不能简单地传入头部迭代器，而是使用辅助函数 `std::back_inserter` 生成的输出迭代器，它会自动执行 push_back 操作来扩充容器。

```cpp
#include <CGAL/Exact_predicates_inexact_constructions_kernel.h>
#include <CGAL/convex_hull_2.h>
#include <vector>

typedef CGAL::Exact_predicates_inexact_constructions_kernel K;
typedef K::Point_2 Point_2;
typedef std::vector<Point_2> Points;

int main()
{
	Points points, result;
	points.push_back(Point_2(0, 0));
	points.push_back(Point_2(10, 0));
	points.push_back(Point_2(10, 10));
	points.push_back(Point_2(6, 5));
	points.push_back(Point_2(4, 1));

	CGAL::convex_hull_2(points.begin(), points.end(), std::back_inserter(result));
	std::cout << result.size() << " points on the convex hull" << std::endl;
	return 0;
}
```



### 内核和特性类

考虑前面使用的凸包算法以及类似的其它凸包算法，可以发现它们都有两个版本。其中一个就是前面提到的传入迭代器的版本，另一个需要传入特征类 Traits

```cpp
template<class InputIterator , class OutputIterator , class Traits >
OutputIterator
convex_hull_2(InputIterator first,
              InputIterator beyond,
              OutputIterator result,
              const Traits & ch_traits)
```

这是泛型编程的体现：Traits 中定义了用户所使用类型的一些特征，让函数支持任意点类型，也可支持多种凸包算法。



以 Graham/Andraw Scan 算法为例，它将点从左到右排序，然后增量地添加点到凸包序列中。要实现这一算法，需要

* 点的类型
* 排列点的方法
* 判定三个点的方向

为此需要提供 4 个类型

- `Traits::Point_2` 点的类型
- `Traits::Less_xy_2` 排列点的方法
- `Traits::Left_turn_2` 方向测试
- `Traits::Equal_2`

一种选择是通过模板传入这些方法

```cpp
template <class InputIterator, class OutputIterator, class Point_2, class Less_xy_2, class Left_turn_2, class Equal_2>
OutputIterator
ch_graham_andrew( InputIterator  first,
                  InputIterator  beyond,
                  OutputIterator result);
```

但为了避免参数太多、太长，则将这些参数都定义在一个特征类中，即 Traits 类型中（CGAL 每个 Kernel 中都有定义好的 Traits 类型）。在 [ConvexHullTraits_2](https://doc.cgal.org/latest/Convex_hull_2/classConvexHullTraits__2.html) 中给出了相应的概念，其中的 `Convex_hull_traits_2` 就可以用在这里

```cpp
CGAL::Convex_hull_traits_2()
    
#include <iostream>
#include <iterator>
#include <CGAL/Exact_predicates_inexact_constructions_kernel.h>
#include <CGAL/Convex_hull_traits_2.h>
#include <CGAL/convex_hull_2.h>
    
typedef CGAL::Exact_predicates_inexact_constructions_kernel K3;
typedef CGAL::Convex_hull_traits_2<K3> K;
typedef K::Point_2 Point_2;

int main()
{
  std::istream_iterator< Point_2 >  input_begin( std::cin );
  std::istream_iterator< Point_2 >  input_end;
  std::ostream_iterator< Point_2 >  output( std::cout, "\n" );
  CGAL::convex_hull_2( input_begin, input_end, output, K() );
  return 0;
}
```



### 概念和模型

一个概念 concept 是一个类型上的一系列要求，即具有嵌套类型、成员函数等。概念的一个模型 model 则是实现概念要求的类。例如

```cpp
template <typename T>
T
duplicate(T t)
{
  return t;
}
```

这就是一个概念，如果要用一个类 C 来实现它，那么 C 至少要具有复制构造器，单例类就不满足要求。



## [半边数据结构](https://doc.cgal.org/latest/HalfedgeDS/index.html)


