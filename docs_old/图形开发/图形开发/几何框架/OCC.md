# OCC 架构

## 基本介绍

### 安装编译

#### 安装版本

在 [OpenCascade 官网](https://dev.opencascade.org/)找到并下载 `OCC 7.4.0` 版本

![](image-20230412134826913.png) 

建议直接下载安装程序进行安装，获取全套的 OCC 以及组件

![](image-20230518150915257.png)



#### 源码编译

下载源码配置编译，需要指定 `TCL/Tk` 库。下载两个库的源码，进入 win 文件夹，首先修改 `buildall.vc.bat` 文件

```bat
@REM 修改使用的 bat 文件
@REM call "C:\Program Files\Microsoft Developer Studio\vc98\bin\vcvars32.bat"
call "D:\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvarsall.bat" amd64

@REM 添加 install 参数
@REM nmake -nologo -f makefile.vc release htmlhelp OPTS=%OPTS% %1
nmake -nologo -f makefile.vc release htmlhelp install OPTS=%OPTS% %1

@REM 设置安装路径
if "%INSTALLDIR%" == "" set INSTALLDIR=E:\Desktop\Tcl
if "%INSTALLDIR%" == "" set INSTALLDIR=E:\Desktop\Tk
```

然后修改 `rules.vc` 中的前缀

```vc
// SUFX	    = tsgx
SUFX	    = sgx
```

在 `Tk` 的 `buildall.vc.bat` 文件中还需要指定 `TCLDIR` 变量。最后执行两个批处理脚本，就可以在相应的路径下找到编译结果。



现在用 cmake 配置 occ 中的 `TCL/Tk` 路径位置

![](image-20240815101303322.png)



### 基本包

#### 基本几何类

gp 包中定义各种简单的几何类型，例如

* gp_Pnt 三维点
* gp_Dir 三维方向
* gp_Ax1 轴



#### 标准数值类型

在 Standard 包中定义了对一般数据类型的重命名封装。



#### 构建几何类

Geom 包中定义了各种复杂的几何数据结构，由 gp 包中的数据结构构建而成，不包含算法。



#### 几何形状构建包

GC_Make 包提供由 gp 包构建的数据结构，但是它包含构建算法，用于快速生成 Geom 数据结构。例如

```cpp
Handle(Geom_TrimmedCurve) arcOfcircle = GC_MakeArcCircle(p1, p2, p3);	// 构造经过给定点的圆弧
Handle(Geom_TrimmedCurve) segment = GC_MakeSegment(p1, p2);				// 构造两点确定的线段
```



#### 拓扑数据结构

TopoDS 包中的类由多个 Geom 类构成，有共同的基类 TopoDS_Shape，也仅仅是数据结构，不包含算法。

|         图形         |      OCC 类      |          描述          |
| :------------------: | :--------------: | :--------------------: |
|    Vertex（顶点）    |  TopoDS_Vertex   |      几何体上的点      |
|      Edge（边）      |   TopoDS_Edge    |   曲线和有边界的向量   |
|     Wire（网格）     |   TopoDS_Wire    |   顶点连接的一系列边   |
|      Face（面）      |   TopoDS_Face    |  闭合网格组成的边界面  |
|     Shell（壳）      |   TopoDS_Shell   |   通过边连接的一组面   |
|     Solid（体）      |   TopoDS_Solid   | 由壳组成的有界三维空间 |
| CompSolid（复合体）  | TopoDS_CompSolid |   通过面连接的一组体   |
| Compound（复合对象） | TopoDS_Compound  |   各种图形构成的集合   |



#### 拓扑结构构建包

BRepBuilderAPI 包提供从 Geom 对象到 TopoDS 对象的构建过程。例如

```cpp
Handle(Geom_TrimmedCurve) segment = GC_MakeSegment(p1, p2);	// 构造两点确定的线段
TopoDS_Edge edge = BRepBuilderAPI_MakeEdge(segment);		// 将线段转换为拓扑边
```



#### 实体构建包

BRepPrimAPI 包提供将 TopoDS 对象构建为实体的方法，这里的实体也是 TopoDS 对象。例如

```cpp
TopoDS_Shape box = BRepPrimAPI_MakeBox(2, 2, 2);	// 2*2*2 的盒子
```



#### 拓扑解析包

TopExp 包提供对 TopoDS 对象的分解方法。例如

```cpp
TopoDS_Shape S;
TopExp_Explorer Ex;
    
// 使用形状 S 和枚举类型 TopAbs_FACE 初始化，表示查找所有面
for (Ex.Init(S,TopAbs_FACE); Ex.More(); Ex.Next())
	TopoDS_Face face = Ex.Current();
```



#### 几何算法包

BRepAlgoAPI 包提供专门用于 Shape 对象的布尔运算（求交 common、并 fuse、差 cut）。例如

```cpp
TopoDS_Shape box = BRepPrimAPI_MakeBox(length, width, height);
TopoDS_Shape neck = BRepPrimAPI_MakeCylinder(axis, radius, height);
TopoDS_Shape body = BRepAlgoAPI_Fuse(box, neck);
```



#### 拓扑转换包

BRep_Tool 包提供从 TopoDS 到 Geom 对象转换方法。例如

```cpp
// 转换为几何曲面
TopoDS_Face face;
Handle(Geom_Surface) surface = BRep_Tool::Surface(face);

// 转换为平面之前需要类型检测
if ( surface->IsKind(STANDARD_TYPE(Geom_Plane)) )
	Handle(Geom_Plane) plane = Handle(Geom_Plane)::DownCast(surface);
```



### 可视化示例

OCC 有两种可视化方案：一种是 OCCT 自带的 AIS 模块；另一种则是通过 VIS 模块链接数据到 VTK 可视化库上使用。后者更专业，功能更多，但是将 BRep 模型转为 mesh 模型数据则需要手动定义。



#### 配置文件结构

├─build
├─lib (存放自己引用的库)
├─output
└─src (存放源文件和头文件)

在主目录下设置 cmake 文件

```cmake
cmake_minimum_required(VERSION 3.18)
project(MAIN)

# 增加子目录 src，以及目标文件夹 bin
add_subdirectory(src bin)
```



在 src 目录下设置 cmake 文件

```cmake
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/output)
file(GLOB_RECURSE SRC CONFIGURE_DEPENDS *.cpp)
file(GLOB_RECURSE INC CONFIGURE_DEPENDS ../*.h)

include_directories(../src)
link_directories(../lib)

# 通过 find 获得 OCC
set(OPENCASCADE_DIR D:/lib/OpenCASCADE-7.4.0-vc14-64/opencascade-7.4.0/cmake)
find_package(OPENCASCADE REQUIRED)

# 设置 VTK 库的路径
set(VTK_PATH D:/lib/OpenCASCADE-7.4.0-vc14-64/vtk-6.1.0-vc14-64)

# 引入 OCC 和 VTK 头文件路径
include_directories(${OpenCASCADE_INCLUDE_DIR})
include_directories(${VTK_PATH}/include/vtk-6.1)

# 引入 OCC 和 VTK 库文件路径
link_directories(${OpenCASCADE_LIBRARY_DIR})
link_directories(${VTK_PATH}/lib)

# 同时将 .cpp 和 .h 文件都作为依赖项
add_executable(main ${SRC} ${INC})

# 链接 OCC 静态库
foreach(LIB ${OpenCASCADE_LIBRARIES})
    target_link_libraries(main ${LIB}.lib)
endforeach(LIB ${OpenCASCADE_LIBRARIES})

# 链接 VTK 的全部静态库
file(GLOB_RECURSE VTK_LIB CONFIGURE_DEPENDS ${VTK_PATH}/lib/*.lib)
target_link_libraries(main ${VTK_LIB})

# 增加动态库路径
set(DLL_PATH1 "D:/lib/OpenCASCADE-7.4.0-vc14-64/opencascade-7.4.0/win64/vc14/bin")
set(DLL_PATH2 "D:/lib/OpenCASCADE-7.4.0-vc14-64/vtk-6.1.0-vc14-64/bin")
set(DLL_PATH3 "D:/lib/OpenCASCADE-7.4.0-vc14-64/freetype-2.5.5-vc14-64/bin")
set(DLL_PATH4 "D:/lib/OpenCASCADE-7.4.0-vc14-64/freeimage-3.17.0-vc14-64/bin")
set(DLL_PATH5 "D:/lib/OpenCASCADE-7.4.0-vc14-64/ffmpeg-3.3.4-64/bin")
set(DLL_PATH6 "D:/lib/OpenCASCADE-7.4.0-vc14-64/tbb_2017.0.100/bin/intel64/vc14")
set(DLL_PATH "PATH=${DLL_PATH1}\;${DLL_PATH2}\;${DLL_PATH3}\;${DLL_PATH4}\;${DLL_PATH5}\;${DLL_PATH6}\;%PATH%")

set_target_properties(main PROPERTIES VS_DEBUGGER_ENVIRONMENT ${DLL_PATH})
```



然后我们写一个显示立方体的简单示例程序

```cpp
#include <BRepPrimAPI_MakeBox.hxx>
#include <IVtkTools_ShapeDataSource.hxx>

#include <vtkNew.h>
#include <vtkAutoInit.h>
#include <vtkRenderer.h>
#include <VtkRenderWindow.h>
#include <vtkRenderWindowInteractor.h>
#include <vtkInteractorStyleTrackballCamera.h>
#include <vtkPolyDataMapper.h>

VTK_MODULE_INIT(vtkRenderingOpenGL)
VTK_MODULE_INIT(vtkInteractionStyle)

int main()
{
	vtkNew<vtkRenderWindow> renderWindow;			// 创建 vtk 窗口
	vtkNew<vtkRenderer> render;						// 创建 vtk 渲染器
	renderWindow->AddRenderer(render.GetPointer()); // 在窗口中加入渲染器

	vtkNew<vtkRenderWindowInteractor> iren;			  // 创建 vtk 交互器
	vtkNew<vtkInteractorStyleTrackballCamera> istyle; // 创建 vtk 相机交互器样式

	iren->SetRenderWindow(renderWindow.GetPointer()); // 设置渲染窗口
	iren->SetInteractorStyle(istyle.GetPointer());	  // 设置交互器样式

	// 创建显示数据
	BRepPrimAPI_MakeBox box(2, 2, 2);		 // 2*2*2 的盒子
	const TopoDS_Shape &shape = box.Shape(); // 获得形状 shape

	vtkNew<IVtkTools_ShapeDataSource> occSource;   // 创建一个可以被 vtk 使用的 occ 数据源
	occSource->SetShape(new IVtkOCC_Shape(shape)); // 将 shape 添加到数据源中

	vtkNew<vtkPolyDataMapper> mapper; // 创建一个 vtk 数据类型

	mapper->SetInputConnection(occSource->GetOutputPort()); // 创建一个管道，将 occ 数据导入到 vtk 数据中

	vtkNew<vtkActor> actor;				   // 创建一个 vtk actor
	actor->SetMapper(mapper.GetPointer()); // 将 vtk 数据交给 actor
	render->AddActor(actor.GetPointer());  // 在渲染器中加入 vtk actor

	// 初始化显示
	iren->Initialize(); // 初始化交互器
	iren->Start();		// 开始运行交互器

	return 0;
}
```



#### 调整编码

需要注意我们应该将文件保存为 UTF-8 with BOM 格式，否则在 VS 编辑器下编译可能出现“未声明的标识符”的错误。

![](image-20230519110144386.png)



#### 代码结构

在 OCC 中每一个类都对应一个头文件，如果需要使用哪一个类，就先引入这个类名的头文件。例如上面代码中

属于 VTK 的类

* vtkNew 是智能指针类型，用于安全地保存指针数据
* vtkRenderWindow 渲染窗口类
* vtkRenderer 渲染器
* vtkRenderWindowInteractor 交互器
* vtkInteractorStyleTrackballCamera 相机交互器
* vtkPolyDataMapper
* vtkActor

属于 OCC 的类

* BRepPrimAPI_MakeBox
* IVtkTools_ShapeDataSource



#### 渲染管线

由于我们使用 OCC 的数据结构，它本身与 VTK 的数据类型不兼容，需要将 OCC 数据与 VTK 数据连通起来。

* 将 OCC 的 TopoDS_Shape 用 IVtkOCC_Shape 包装，然后转换为 IVtkTools_ShapeDataSource 数据
* 创建 vtkPolyDataMapper 使用 SetInputConnection 连接 IVtkTools_ShapeDataSource 数据
* 将 mapper 传入 actor
* 再将 actor 传入 render 渲染





## 基本类型

### 介绍

#### 根类

根类是基本数据类型，包括：

* 布尔、字符、整型或实数；
* 安全处理动态创建的对象（Standard_Transient）；
* 可配置的优化内存管理；
* 扩展的运行时类型机制 RTTI 有助于创建复杂程序，管理异常；

在 Standard 和 MMgt 包中实现。



#### 字符串

使用 ASCII 或 Unicode 类型，通过句柄控制，在 TCollection 包中实现。



#### 集合

集合 Collections 处理动态调整大小的数据。它定义结构和算法，允许保存各种对象，这些对象不需要具有共同的父类。通过特定类型的元素实例化后使用，在 TCollection 和 NCollection 包中实现。



#### 标准对象集合

在 TColStd 包中提供了 TCollection 包中常用的泛型类实例。



#### 通用数学算法

OCC 提供了向量和矩阵类以及相关的数学算法和基础操作（加乘、转置、取逆等）。还包括基本几何类型

* 点、向量、直线、圆和二次曲线、平面、初等曲面；
* 形状的空间位置或相对坐标系的位置；
* 几何变换：平移、旋转、对称、伸缩、复合变换；

提供常用算法的 C++ 实现

* 求解线性方程组；
* 寻找函数最小值；
* 非线性方程组求根；
* 求解特征值和特征向量；



#### 物理量

提供日期、时间信息，还包括基本物理量：长度、面积、体积、质量、密度等。



#### 应用服务

提供基础类的底层服务，用于自定义创建应用程序。包括

* 单位转换工具；
* 表达式的基本解释器，便于创建用户脚本（见 ExprIntrp 包）；
* 处理配置资源文件的工具（见 Resource 包）和定制消息文件（见 Message 包）；
* 进度提示和用于中断接口，提供底层算法与用户的交互；



### 库的组织

OCCT 库由多个模块组成，第一个模块提供基础服务，被其它模块使用，称为 Foundation 类。每个模块由多个工具包构成，每个工具包通常是一个动态库文件，由多个包构建。



包将许多类链接起来。例如几何包具有点、线和圆类。包可以包含枚举、异常和函数。包定义的类名是它的前缀，例如 Geom_Circle；每个包可以描述如下数据类型：

* 枚举；
* 对象类；
* 异常类；
* 指向其它类的指针；

![](Screenshot_20230525_150328.jpg|500)

方法要么是**函数 functions**要么是**过程 procedures**，前者返回一个对象，后者只通过传递参数进行通讯。在这两种情况下，当传输的对象是由句柄操作的实例时，将会传递其**标识符 identifier** 。有三类方法：

* **对象构造器** 创建类的实例；
* **实例方法** 作用于它拥有的实例；
* **类方法** 不作用于单独的实例，只作用于类本身；



### 数据类型

OCC 中的数据类型分为两类：

* 由句柄（或引用）控制的数据类型；
* 通过值控制的数据类型；

![](Screenshot_20230525_151227.jpg|500)

数据类型通过类实现。类不仅定义了数据表示和作用于实例的方法，还说明实例如何被控制。

* 由值控制的类型的变量包含它自身的实例，例如布尔值、字符、整型、实数等；
* 由句柄控制的类型的变量包含**实例的引用**；

一个不包含任何实例引用的句柄称为**null**，要引用一个对象，使用

```cpp
Handle(myClass) m = new myClass;
```

在 OCC 中 Handle 用于安全地处理动态内存。



**原始类型 Primitive Types** 是将语言中的类型重定义，都是**由值控制**。这些类型在 Standard 包中描述。C++ 中的基本类型如下

![](Screenshot_20230525_152120.jpg|500)

由值控制的类型有如下三类：

* 原始类型；
* 枚举类型；
* 没有继承 Standard_Transient 类的类型；

由值操作的类型比由句柄操作的类型更直接，可以执行更快，但是不能独立存储在文件中。



在设计对象时，要如何决定使用值还是句柄来操作对象呢？可以参考如下几点：

* 如果该对象可能具有很长的生命周期，并且希望存在多个引用，则最好使用句柄操作。对象的内存分配在堆上。指向该内存的句柄是一个轻对象，可以在参数中快速传递，避免大对象复制的代价；
* 如果对象生命周期有限，例如在单个算法中使用，则按照值操作对象，不考虑其大小。因为该对象在栈上分配，避免了对 new 和 delete 的隐式调用；
* 如果一个对象只在应用程序的整个生命周期中创建一次，但将存在于整个生命周期中，则由句柄操作，或者定义为全局变量的值；



### 句柄编程

#### 定义句柄

OCCT 中的句柄是一个智能指针。**所有可被句柄操作的类，都必须继承自 Standard_Transient 类**。它提供引用计数器，被后代类继承，关联的 `Handle()` 类使用该计数器跟踪指向对象实例的多个句柄。从 Transient 派生的类对象使用 new 在动态内存中分配，并由句柄操作。句柄定义为模板类 `opencascade::handle<>` 提供预处理宏 `Handle()` 来命名句柄：

```cpp
Handle(Geom_Line) aLine;	// Handle(Geom_Line) 扩展为 opencascade::handle<Geom_Line>
```

此外对于标准 OCCT 类，还为句柄定义了额外的名称，通过前缀 `Handle_` 描述。例如上面代码可以写成

```cpp
Handle_Geom_Line aLine;
```



对于 Transient 类的任何操作通过句柄进行。例如

```cpp
Handle(Geom_Point) p1, p2;
```

此时句柄为 null，不指向任何对象。

* 可以通过 `IsNull()` 方法判断一个句柄是否是空的；
* 使用 `Nullify()` 取消句柄； 

注意局部操作推荐使用值。



#### 类型管理

OCC 提供在运行时获得继承数据类型的方法，因此可以随时检查给定类的类型。要启用这一特性，类声明应当包括 OCCT RTTI 声明。头文件 `Standard_Type.hxx` 提供两个变体预处理宏：

* 内联变体，通过一行代码定义 RTTI 方法

    ```cpp
    #include <Geom_Surface.hxx>
    
    class Appli_ExtSurface : public Geom_Surface
    {
    public:
    	DEFINE_STANDARD_RTTIEXT(Appli_ExtSurface, Geom_Surface)
    };
    ```

    

* 外部变体，在声明时（头文件）定义宏，然后在实现（源码）中定义宏

    ```cpp
    // Appli_ExtSurface.h 
    #include <Geom_Surface.hxx>
    
    class Appli_ExtSurface : public Geom_Surface
    {
    public:
    	DEFINE_STANDARD_RTTIEXT(Appli_ExtSurface, Geom_Surface)
    };
    
    // Appli_ExtSurface.cpp 
    #include <Appli_ExtSurface.hxx>
    
    IMPLEMENT_STANDARD_RTTIEXT(Appli_ExtSurface, Geom_Surface)
    ```

这些宏定义了方法 `DynamicType()`，用于返回描述类的描述符——这个类的 Standard_Type 单例句柄。这个描述符储存类的名称及其父类的描述符。可以获得类的名称作为参数

```cpp
// 判断 aCurve 是否是 Geom_Line 的子类
if (aCurve->IsKind(STANDARD_TYPE(Geom_Line)))
{
}
```

注意，虽然内联版本易于使用，但是对于广泛使用的类，内联方法的多次实例化会导致依赖库代码膨胀。



句柄声明中使用的类型是对象的静态类型，即编译器看到的类型。句柄可以引用该静态类型的子类实例化的对象，因此对象的动态类型（实际类型）可以是在句柄声明中出现的类型的子类。例如 Point 类的子类 CartesianPoint 可以按照如下方式引用

```cpp
Handle(Geom_Point) p1;
Handle(Geom_CartesianPoint) p2;
p2 = new Geom_CartesianPoint;
p1 = p2;	// 即可以让 Geom_Point 的句柄指向 Geom_CartesianPoint 实例
```

同时可以将父类句柄显式转化为子类句柄

```cpp
Handle(Geom_CartesianPoint) p3;
p3 = Handle(Geom_CartesianPoint)::DownCast(p1);	// 将 Geom_Point 句柄转化为 Geom_CartesianPoint 句柄

// 判断转化是否成功
if (p3.IsNull())
    // 转化失败
```

当转化失败，即类之间没有继承关系，则得到的句柄为空。



如果有两个继承自 Standard_Transient 的类 A 和 B，则可以有如下语法

```cpp
Handle(A) a = new A;
Handle(B) b = new B;

// 将它们放在序列中
SequenceOfTransient s;
s.Append(a);
s.Append(b);

// 获得序列中的元素
Handle(Standard_Transient) t = s.Value(1);	// 这一步让父类指针指向子类对象，相当于提升了类
a = Handle(A)::DownCast(t);					// 通过显式转化使得子类句柄可以获得父类句柄
```



#### 使用句柄

当定义了指向实例的句柄后，可以像使用指针一样使用句柄。例如

```cpp
// 用父类句柄管理具体的子类实例
Handle(Standard_Transient) p = new Geom_CartesianPoint(0, 0, 0);

// 当 p 的动态类型（实际类型）是 Geom_CartesianPoint，就将其转换为 Geom_CartesianPoint 句柄使用
if ( p->DynamicType() == STANDARD_TYPE(Geom_CartesianPoint) )
	Handle(Geom_CartesianPoint) cp = Handle(Geom_CartesianPoint)::DownCast(p);
```

如果一个空的句柄访问对象的成员或方法，就会抛出 `NullObject` 异常。



成员函数通过 `->` 调用，类静态成员函数通过 `::` 调用。例如

```cpp
Standard_Integer n;
n = Geom_BezierCurve::MaxDegree();
```



### 集合 Collection

Collection 组件包含处理动态调整数据规模的类，比如数组、列表和映射。集合类类似于 C++ 的 STL，任何数据都可以使用它们进行保存，并可以使用集合类中定义的算法。

* TColStd 包提供了标准对象组件的集合，即针对标准对象提供了实例化；
* 数组 Arrays 用于快速访问对象，但是大小固定；
* 序列 Sequences 大小可变，但是访问时间比数组长，具有访问算法；
* 映射 Maps 是动态的结构，其大小适应插入项的数量，并且具有最快的访问速度；
* 列表 Lists 类似于序列，但是访问算法不同；

序列和映射有特定的迭代器。通用的集合例如

* TCollection_Array1/TCollection_Array2 固定大小的一、二维数组；
* TCollection_HArray1/TCollection_HArray2  固定大小的一、二维数组，区别在于它存放**句柄**；

同理还有其它类型的集合实例。**需要注意 OCC 中的数组集合都是从 1 开始索引！！！**



## 模型数据

### 几何工具

几何工具提供如下服务：

* 通过插值和逼近构造形状；
* 直接构造形状；
* 曲线曲面到 B 样条曲线曲面的转换；
* 计算 2D/3D 曲线上点的坐标；
* 计算形状之间的极值；



#### 插值和逼近

在 GProp 包中的 PEquation 类可以分析点云，验证它们是否在指定精度下重合、共线或共面。如果是，则会计算它们的平均点、线、面。否则，会计算最小包围盒。



包 Geom2dAPI 和 GeomAPI 提供了近似和插值的简单算法：

* 插值

    * Geom2dAPI 提供的 Interpolate 类构建经过约束点的 2D B 样条曲线。还可以给定约束点的参数值和切向；
    * GeomAPI 提供的 Interpolate 类构建经过约束点的 3D B 样条曲线。还可以给定约束点的参数值和切向；

```cpp
GeomAPI_Interpolate Interp(Points);
Interp.Perform();
Handle(Geom_BSplineCurve) C = Interp.Curve();
```

* 逼近

    * Geom2dAPI 提供的 PointsToBSpline 类构建近似 2D B 样条曲线。必须定义最低、最高阶数、连续性和公差。生成曲线将为 $C^2$ 连续曲线，除非在曲线通过的点上定义了相切约束，此时将只有 $C^1$ 连续；
    * GeomAPI 提供的 PointsToBSpline 类构建近似 3D B 样条曲线。必须定义最低、最高阶数、连续性和公差。生成曲线将为 $C^2$ 连续曲线，除非在曲线通过的点上定义了相切约束，此时将只有 $C^1$ 连续；

```cpp
GeomAPI_PointsToBSpline Approx(Points, DegMin, DegMax, Continuity, Tol);
Approx.Perform();
Handle(Geom_BSplineCurve) K = Approx.Curve();
```

* 曲面逼近

    * GeomAPI 提供的 PointsToBSplineSurface 类构建近似 3D B 样条曲面，将逼近或插值给定点； 



在 AppDef 和 AppParCurves 中提供了低级函数，允许提供逼近的更多控制，包括

* 定制有起点和终点的强制切线；
* 近似一组平行曲线，产生相同的参数化；
* 平滑化近似，得到平坦的曲线；

还可以找到函数计算

* 给定点集的包围盒；
* 一组共面、共线或重合点的平均平面、线或点；

AppDef 包允许对多点约束使用平行的 Bezier 或 B 样条曲线逼近。并提供如下服务：

* 点约束数组的定义：类 MultiLine 允许定义给定数量的多点约束，来构建多条线通过的有序多点约束；

![](Screenshot_20230525_192906.jpg|500)

* 定义一组点的约束：类 MultiPointConstraint 允许定义多点约束，并计算点的近似曲线；
* 类 Compute 允许计算一组点的近似 Bezier 曲线；
* 类 BSplineCompute 允许计算一组点的近似 B 样条曲线；
* 变分准则：类 TheVariational 允许使用最小二乘法和变分准则调整近似曲线；



#### 直接构造

在 gce, GC 和 GCE2d 包中定义的直接构造方法提供了构建基本几何实体的简化算法，例如直线、圆和曲线。它们补充了 gp, Geom 和 Geom2d 包提供的引用定义。



在 gp 包中根据点和半径构造圆，需要在创建圆之前构建 AxisAx2d；但如果使用 gce 包，并以 Ox 为轴，就可以直接通过点和半径创建圆。例如

```cpp
// 构建空间中的点
gp_Pnt P1 (0.,0.,0.);
gp_Pnt P2 (0.,10.,0.);
gp_Pnt P3 (10.,0.,0.);

// 使用 gce 算法构造经过点的圆
gce_MakeCirc MC (P1,P2,P3);
if (MC.IsDone()) 
{
	const gp_Circ& C = MC.Value();
}
```



在 gp 包中的每个类，例如 Circ, Circ2d, Mirror, Mirror2d 等，都在 gce 包中有对应的 MakeCirc, MakeCirc2d 等算法。创建完成后，先检查是否出现异常

```cpp
gp_Pnt2d Point1,Point2;
...
// 用两个点创建直线
gce_MakeLin2d L = gce_MakeLin2d(Point1,Point2);
if (L.Status() == gce_Done() )
{
	gp_Lin2d l = L.Value();
}
```

用两个点创建连线，如果它们距离很近，则 `Status()` 会返回枚举变量 `gce_ConfusedPoint`；当然，如果能够确定没有异常，则可以直接生成

```cpp
gp_Lin2d l = gce_MakeLin2d(Point1,Point2);
```



在 GC 和 GCE2d 包中提供了从 Geom, Geom2d 包中生成实体的实现算法。它们实现了和 gce 包中相同的算法，还包括裁剪曲面和曲线的算法。和 gce 包相同，每个 Geom 中的 Circle, Ellipse 等类都在 GC 包中有对应的 MakeCircle, MakeEllipse 等算法。



#### 转换 B 样条

与 B 样条之间的转换操作由三个包实现

* Convert 包提供如下曲线的转换

    * gp 包中基于初等 2D 曲线（线、圆、二次曲线）的有界曲线；
    * gp 包中基于初等曲面（圆柱、锥面、球面或环面）的有界曲面；
    * 由控制点定义的一系列相邻的 2D 或 3D 贝塞尔曲线；

    这些算法计算用于生成样条曲线或曲面所需的数据。这些基本数据（阶数、周期特征、控制点和权重）可以直接在算法中使用，或者通过调用 Geom2d_BSplineCurve, Geom_BSplineCurve, Geom_BSplineSufrace 提供的构造函数生成。

* Geom2dConvert 包；
* GeomConvert 包；



#### 曲线上的点

这部分组件包含提供 API 的高级函数，用于计算 2D 或 3D 曲线上的点的复杂算法。以下特征点位于 3D 空间中的参数曲线上

* 曲线上等距分布的点；
* 沿曲线相同弦长分布的点；
* 与曲线上另一个点相距给定距离的点;

GCPnts 包提供计算如下点的方法

* AbscissaPoint 计算与曲线上另一个点相距给定距离的点;
* UniformAbscissa 计算曲线上给定横坐标的一组点；
* UniformDeflection 计算曲线上一组点，使得它们构成的多边形与曲线之间的偏移距离最大；

要使用这些算法，需要先将 Geom 对象进行转换

```cpp
// 构造几何曲线，用 GeomAdaptor_Curve 封装
Handle(Geom_Curve) mycurve = ... ;
GeomAdaptor_Curve C (mycurve) ;

// 创建 UniformDeflection 算法
GCPnts_UniformDeflection myAlgo () ;
Standard_Real Deflection = ... ;

// 初始化算法
myAlgo.Initialize ( C , Deflection ) ;
if ( myAlgo.IsDone() )
{
    // 如果计算成功，获得生成点的个数
    Standard_Integer nbr = myAlgo.NbPoints() ;
    Standard_Real param ;
    
    // 通过索引获得点对应的参数值
    for ( Standard_Integer i = 1 ; i <= nbr ; i++ )
    {
        param = myAlgo.Parameter (i) ;
        ...
    }
}
```



#### 计算极值

使用 GeomAPI 和 Geom2dAPI 包计算点、曲线、曲面之间的最小距离。例如

* `GeomAPI_ProjectPointOnCurve` 类计算点和曲线之间的所有极值。**极值就是两点连线与曲线正交时连线的长度**；
* `GeomAPI_ProjectPointOnSurface` 类计算点和曲面之间的所有极值。**极值就是两点连线与曲面正交时连线的长度**；



### 2D 几何

Geom2d 包定义了 2D 空间中的几何物体。例如

* 点、向量和曲线的描述；
* 在平面上使用坐标系统进行定位；
* 应用平移、旋转、对称、缩放变换或组合变换；

对象通过**引用**进行控制，如果需要一组对象实例而不是单个对象实例，应该使用 TColGeom2d 包，它提供了Geom2d 包中常用曲线实例的一维数组和序列。



Geom2d 曲线的关键特征是**参数化**。每个类都提供了函数处理曲线的参数方程，特别是计算曲线上参数 u 处的多阶导数向量。参数化的使得所有 Geom2d 曲线都有定向。Geom2d 与 gp 包的关系是

* 参数化和方向将 Geom2d 曲线和 gp 包提供的等价曲线区分开来；
* 可以将 Geom2d 曲线转换为 gp 对象，反之也有可能；
* Geom2d 提供更复杂的曲线：Bezier, B-Spline, Trimmed, Offset 曲线；



Geom2d 对象通过继承形成多级结构进行组织。例如

* 椭圆 Geom2d_Ellipse 即圆锥曲线，继承自 Geom2d_Conic 抽象类；
* Bezier 曲线 Geom2d_BezierCurve 有界，继承自 Geom2d_BoundedCurve 抽象类；

它们有共同的 Geom2d_Curve 抽象类。曲线，点、向量都继承自 Geom2d_Geometry 抽象类。

![](class_geom2d___geometry__inherit__graph.png)



Geom2d 包使用 gp 包的服务来

* 实现基本的代数计算和基本解析几何；
* 描述可应用于 Geom2d 对象的几何变换；
* 描述 Geom2d 对象的基本数据结构；

然而，Geom2d 包本质上提供数据结构而非算法。需要参考 GCE2d 包来查找更先进的 Geom2d 对象构建算法。



### 3D 几何

Geom 包定义了 3D 空间中的几何物体，并包含所有基本的几何变换，如恒等、旋转、平移、镜像、缩放、组合等。以及依赖于几何对象定义的特殊功能，例如在 B 样条曲线上添加控制点。



Geom 曲线和曲面也具有**参数化**特征，因此它们自然定向，区别于 gp 包中的等价曲线和曲面。

* Geom 中的对象也可以转换为 gp 对象，反之也有可能；
* Geom 提供更复杂的曲线和曲面类型：Bezier, B-Spline, Trimmed, Offset 曲线和曲面；

对象通过**引用**进行控制，如果需要一组对象实例而不是单个对象实例，应该使用 TColGeom 包，它提供了Geom 包中常用曲线实例的一维、二维数组和序列。



Geom 对象也通过继承形成多级结构进行组织，并使用 gp 包的服务来

* 实现基本的代数计算和基本解析几何；
* 描述可应用于 Geom 对象的几何变换；
* 描述 Geom 对象的基本数据结构；

然而，Geom 包本质上提供数据结构而非算法。需要参考 GC 包来查找更先进的 Geom 对象构建算法。



![](class_geom___geometry__inherit__graph.png)



### 形状属性

#### 形状局部属性

BRepLProp 包提供了形状组件的局部属性算法，用于计算 BRep 模型中的边和面的局部属性。例如

* 对于边，计算参数 u 对应的点、切向、法向、曲率；
* 对于面，计算参数 (u, v) 对应的点、切向、法向、曲率；

被解析的边和面通过 BRepAdaptor 曲线和曲面（在“曲线上的点”中介绍过）来描述。



#### 曲线曲面局部属性

对于 Geom 曲线或曲面的局部属性，通过

* Geom2dLProp 包：计算 2D 曲线参数值对应的切向、法向；
* GeomLProp 包：计算 3D 曲线、曲面的局部属性；
* LProp 包：提供描述 2D 曲线上特定点的枚举类型；



#### 适配器

通用的 OCC 算法可以应用于多种类型的曲线或曲面。这些算法需要通过接口获得分析曲线或曲面所需的服务，从而对任何类型的曲线或曲面获得单个 API 。这些接口就称为**适配器**。例如

* Adaptor3d_Curve 是一个抽象类，它为任何 3D 曲线提供所需的算法服务；

GeomAdaptor 包提供以下接口：

* Geom 曲线；
* Geom 曲面上的曲线；
* Geom 曲面

BRepAdaptor 包提供以下接口：

* Face
* Edge

使用时只需要先创建 Geom 对象，然后将其传入适配器，就可以调用算法。



### 拓扑结构

OCCT 拓扑结构允许访问和处理对象数据，而无需处理它们的 2D 或 3D 表示。OCCT 几何体通过参数化空间中的坐标描述对象的数据结构。拓扑定义简单几何实体之间的关系，通过这种方式将简单实体装配成为复杂的形状。



抽象拓扑数据结构描述一个基本实体：形状。它可以分解为如下拓扑组件

* Vertex —— 0 维形状，对应几何中的点；
* Edge —— 对应曲线的形状，被顶点控制边界；
* Wire —— 通过顶点连接的边的序列；
* Face —— 由封闭 wire 作为边界的曲面；
* Shell —— 多个面的集合，通过它们的边界 wire 上的 Edge 相连；
* Solid —— 由 Shell 作为边界的 3D 空间；
* Compound Solid —— Solid 的集合；

具有 3D 基础几何体的面也可以是一组连接的三角形近似的曲面。可以不定义曲面，而是保留由三角形表示的面，得到纯多面体的模型。

![](Screenshot_20230528_225915.jpg|500)



OCCT 中描述拓扑数据结构的包有

* TopAbs 包提供拓扑驱动的程序的通用资源。例如拓扑形状、方向和状态的枚举类型，和管理这些类型的方法；
* TopLoc 包提供处理 3D 局部坐标系的资源：Datum3D 和 Location 。前者描述了一个基本坐标系；后者包含一系列基本坐标系；
* TopoDS 包描述用于建模和构建纯拓扑数据结构的类；

用来控制抽象拓扑结构的包有

* TopTools 包提供用于拓扑数据结构的基本工具；
* TopExp 包提供对 TopoDS 包中的数据结构的搜索和控制类；
* BRepTools 包提供搜索、控制、读写 BRep 数据结构的类；



#### 控制形状

TopoDS 包提供对拓扑数据结构的如下描述

* 对无方向和位置的抽象形状的引用；
* 通过工具类访问数据结构；

形状模型是可共享的数据结构，它可以被其它形状使用（一条边可以被实体的多个面使用）。当简单引用不够时，可以添加

* 方向：说明边界中如何使用引用的形状（通过 TopAbs 描述）；
* 局部参照坐标：允许引用形状的位置不同于其定义的位置（通过 TopLoc 描述）；

TopoDS_Shape 类描述了对形状的引用。它包含一个对底层抽象形状的引用、一个方向和一个局部参照坐标。**这个类通过值控制，因此不能共享**。

![](class_topo_d_s___shape__inherit__graph.png)



从 TopoDS_Shape 继承的类没有构造函数，我们需要通过类型验证判断对象的类型

```cpp
#include <TopoDS_Vertex.hxx>
#include <TopoDS_Edge.hxx>
#include <TopoDS_Shape.hxx>
#include <TopoDS.hxx>

// 某个处理边的方法
void ProcessEdge(const TopoDS_Edge&);

void Process(const TopoDS_Shape& aShape) 
{
    // 检查形状的类型
    if (aShape.ShapeType() == TopAbs_VERTEX) 
    {
        // 如果是顶点，就通过 Vertex 函数转换为顶点
        TopoDS_Vertex V;
        V = TopoDS::Vertex(aShape); 				// 可以正确转换
        TopoDS_Vertex V2 = aShape; 					// 不能正确转换
        TopoDS_Vertex V3 = TopoDS::Vertex(aShape); 	// 可以正确转换
    }
    else if (aShape.ShapeType() == TopAbs_EDGE)
    {
        // 不能直接作为边传入
        ProcessEdge(aShape) ;
        
        // 要转换为边类型再传入
        ProcessEdge(TopoDS::Edge(aShape)) ;
    }
    else {
		...
    }
}
```



#### 拓扑分解

TopExp 包提供查找给定类型形状的子对象的方法，例如查找实体的所有面。构建 TopExp_Explorer 类

```cpp
void test() 
{
    TopoDS_Shape S;
    TopExp_Explorer Ex;
    
    // 使用形状 S 和枚举类型 TopAbs_FACE 初始化，表示查找所有面
    for (Ex.Init(S,TopAbs_FACE); Ex.More(); Ex.Next())
    	TopoDS_Face face = Ex.Current();
}
```

查找不在边上的所有顶点

```cpp
for (Ex.Init(S,TopAbs_VERTEX,TopAbs_EDGE); ...)
```

查找所有在 SHELL 中的面，然后查找不在 SHELL 中的面

```cpp
void test() 
{
    TopExp_Explorer Ex1, Ex2;
    TopoDS_Shape S;
    
    // 先查找所有 SHELL
    for (Ex1.Init(S,TopAbs_SHELL);Ex1.More(); Ex1.Next())
    {
        // 访问 SHELL 中的面
        for (Ex2.Init(Ex1.Current(),TopAbs_FACE);Ex2.More();Ex2.Next())
            TopoDS_Face face = Ex2.Current();
    }
    
    // 查找不在 SHELL 中的面
    for(Ex1.Init(S,TopAbs_FACE,TopAbs_SHELL);Ex1.More(); Ex1.Next())
    	TopoDS_Face face = Ex1.Current();
}
```



#### 形状序列

TopTools 包包含 TopoDS 数据结构的存储工具。例如

* `TopTools_Array1OfShape, HArray1OfShape` 是 `TCollection_Array1` 和 `TCollection_HArray1` 的实例化；
* `TopTools_SequenceOfShape` 是 `TCollection_Sequence` 的实例化；



#### 读写形状

使用 BRepTools 和 BinTools 实现将形状读写到文件中。其中 BRepTools 包提供 ASCII 储存格式，BinTools 包提供二进制格式。例如

```cpp
TopoDS_Shape aShape;
if (BRepTools::Read (aShape, "source_file.txt"))
	BinTools::Write (aShape, "result_file.bin");
```

通过 BRepTools 读取形状，BinTools 写入到二进制文件。



## 模型算法

### 投影算法

投影算法可以实现 2D 和 3D 空间点到曲线上的投影，以及 3D 曲线到曲面上的投影。



#### 点投影到曲线

Geom2dAPI_ProjectPointOnCurve 计算点 `gp_Pnt2d` 到几何曲线 `Geom2d_Curve` 上的所有正交投影。例如

```cpp
gp_Pnt2d P;
Handle(Geom2d_BezierCurve) C = new Geom2d_BezierCurve(args);
Geom2dAPI_ProjectPointOnCurve Projector (P, C);
```

可以添加参数约束区间

```cpp
Geom2dAPI_ProjectPointOnCurve Projector (P, C, U1, U2);
```

然后调用相应的函数获得投影结果

```cpp
Standard_Integer NumSolutions = Projector.NbPoints();	// 投影点数量
gp_Pnt2d Pn = Projector.Point(Index);					// 通过索引获得投影点
Standard_Real U = Projector.Parameter(Index);			// 通过索引获得投影点参数
Standard_Real D = Projector.Distance(Index);			// 通过索引获得投影距离
```

可以直接获得最近投影点的信息

```cpp
gp_Pnt2d P1 = Projector.NearestPoint();					// 获得最近投影点
Standard_Real U = Projector.LowerDistanceParameter();	// 获得最近投影参数
Standard_Real D = Projector.LowerDistance();			// 获得最近投影距离
```

对于 3D 点到曲线的投影过程也是完全相同的。

![](Screenshot_20230529_142328.jpg|500)



OCC 通过对操作符的重载，实现了更简单的获得解的方式。例如

```cpp
Standard_Real D = Geom2dAPI_ProjectPointOnCurve (P,C);		// 获得最近投影距离
Standard_Integer N = Geom2dAPI_ProjectPointOnCurve (P,C);	// 获得投影点数
gp_Pnt2d P1 = Geom2dAPI_ProjectPointOnCurve (P,C);			// 获得最近投影点
```

使用它们可以更快地获得最近投影信息。



#### 点投影到曲面

GeomAPI_ProjectPointOnSurf 计算点 `gp_Pnt` 到几何曲面 `Geom_Surface` 上的所有正交投影。例如

```cpp
gp_Pnt P;
Handle (Geom_Surface) S = new Geom_BezierSurface(args);
GeomAPI_ProjectPointOnSurf Proj (P, S);
```

可以增加参数约束区域

```cpp
GeomAPI_ProjectPointOnSurf Proj (P, S, U1, U2, V1, V2);
```

其余操作与点投影到曲线完全相同。

![](Screenshot_20230529_143328.jpg|500)



#### 曲线维数切换

使用 `To2d()` 方法将位于 `gp_Pln` 平面上的 `Geom_Curve` 转为 2D 曲线；使用 `To3d()` 方法将位于 `gp_Pln` 平面上的 `Geom2D_Curve` 转为 3D 曲线。例如

```cpp
Handle(Geom2d_Curve) C2d = GeomAPI::To2d(C3d, Pln);
Handle(Geom_Curve) C3d = GeomAPI::To3d(C2d, Pln);
```



### 拓扑工具

拓扑工具可以通过基本拓扑形状构建更复杂的模型、计算相对位置关系等。例如使用一系列边构建 wire，然后获得面

```cpp
TopoDS_Shape anEdges = ...;					// 输入一系列边的形状
Standard_Real anAngTol = 1.e-8; 			// 允许的角度误差
Standard_Boolean bShared = Standard_False;	// 是否共享边

// 计算转换的 wire
TopoDS_Shape aWires;
Standard_Integer iErr = BOPAlgo_Tools::EdgesToWires(anEdges, aWires, bShared, anAngTol);

// 错误检查
if (iErr) 
    ...

// 计算转换的面
TopoDS_Shape aFaces;
Standard_Boolean bDone = BOPAlgo_Tools::WiresToFaces(aWires, aFaces, anAngTol);

// 错误检查
if (!bDone) 
    ...
```



### 拓扑 API

拓扑 API 提供不同类的不同构造方法，它保留了不同的工具来构建对象，并提供类型转换方法。包括

* BRepAlgoAPI
* BRepBuilderAPI
* BRepFilletAPI
* BRepFeat
* BRepOffsetAPI
* BRepPrimAPI

例如可以通过 BRepBuilderAPI 构建边

```cpp
gp_Pnt P1(10,0,0), P2(20,0,0);
BRepBuilderAPI_MakeEdge ME(P1,P2);

// 检查是否成功
if (!ME.IsDone())
    ...

// 直接获得形状
TopoDS_Edge E = ME;
```

由于 BRepBuilderAPI 使用了强制转换函数，因此可以隐式地直接转换为 TopoDS 对象。



BRepBuilderAPI_MakeEdge 提供了有价值的信息，例如可以获得构建边的顶点形状

```cpp
gp_Pnt P1(10,0,0), P2(20,0,0);
BRepBuilderAPI_MakeEdge ME(P1,P2);

TopoDS_Edge E = ME;
TopoDS_Vertex V1 = ME.Vertex1();
TopoDS_Vertex V2 = ME.Vertex2();
```



拓扑 API 具有历史记录功能，可以保存形状修改的过程。包括顶点、边、面的生成、删除、修改操作都会被记录。使用函数

* `Standard_Boolean IsDeleted(const TopoDS_Shape& theS);` 检查形状是否被删除；
* `const TopTools_ListOfShape& Modifified(const TopoDS_Shape& theS);` 获得从给定形状修改的形状；
* `const TopTools_ListOfShape& Generated(const TopoDS_Shape& theS)` 获得从给定形状生成的形状；

当形状进行**并、交、差**等操作时，这些操作历史可以通过这些函数获得。



### 标准拓扑对象

可以创建的标准拓扑对象包括 Vertices, Edges, Faces, Wires, Polygonal wires, Shells, Solids 等。有两个基类用于它们的创建和修改

* BRepBuilderAPI_MakeShape 是所有 BRepBuilderAPI 类的基类，用于构建形状；
* BRepBuilderAPI_ModifyShape 是所有修改形状类的基类，继承 BRepBuilderAPI_MakeShape 类，实现跟踪所有子形状修改历史的方法；

例如创建顶点

```cpp
gp_Pnt P(0,0,10);
TopoDS_Vertex V = BRepBuilderAPI_MakeVertex(P);
```



#### Edge

构建边时，需要指定顶点来判断定向

```cpp
Handle(Geom_Curve) C = ...;			// 构成边的曲线
TopoDS_Vertex V1 = ...,V2 = ...;	// 曲线的端点
Standard_Real p1 = ..., p2 = ..;	// 端点的参数值
TopoDS_Edge E = BRepBuilderAPI_MakeEdge(C,V1,V2,p1,p2);
```

其中 V1 是起点，V2 是终点，曲线按照定向延伸。

![](Screenshot_20230529_151547.jpg|500)



生成边的参数分别要满足

* 曲线：不能是空句柄；如果是裁剪曲线，会使用原本的基础曲线；
* 顶点：不能是空形状。如果要是空的，则对应的参数必须是无穷（p1 为 RealFirst()，p2 为 RealLast()）；
* 参数：必须满足 `C->FirstParameter() <= p1 < p2 <= C->LastParameter()`；如果不是递增，则会自动交换参数；端点的空间位置与参数在曲线上对应的点之间的距离要小于默认精度；

![](Screenshot_20230529_152220.jpg|500)



为了简化构造，BRepBuilderAPI_MakeEdge 提供了简化版本的构造方法

```cpp
Handle(Geom_Curve) C = ...;
TopoDS_Vertex V1 = ...,V2 = ...;
Standard_Real p1 = ..., p2 = ..;
gp_Pnt P1 = ..., P2 = ...;

// 空的边
TopoDS_Edge E;

E = BRepBuilderAPI_MakeEdge(C,V1,V2);			// 将顶点投影到曲线上构造
E = BRepBuilderAPI_MakeEdge(C,P1,P2,p1,p2);		// 通过点和参数构造
E = BRepBuilderAPI_MakeEdge(C,P1,P2);			// 通过点投影到曲线上构造
E = BRepBuilderAPI_MakeEdge(C,p1,p2);			// 通过参数值构造
E = BRepBuilderAPI_MakeEdge(C);					// 使用整条曲线构造
E = BRepBuilderAPI_MakeEdge(V1,V2);				// 顶点构造线段
E = BRepBuilderAPI_MakeEdge(P1,P2);				// 点构造线段
```



我们也可以使用 gp 包中的对象代替 Geom 对象类创建拓扑形状。例如创建圆弧和直线边连接的 wire 模型

```cpp
#include <BRepBuilderAPI_MakeEdge.hxx>
#include <TopoDS_Shape.hxx>
#include <gp_Circ.hxx>
#include <gp.hxx>
#include <TopoDS_Wire.hxx>
#include <TopTools_Array1OfShape.hxx>
#include <BRepBuilderAPI_MakeWire.hxx>

// 通过 圆心(x,y) 半径 R 角度 ang 创建拓扑边和两个端点
void MakeArc(
    Standard_Real x, Standard_Real y,
    Standard_Real R, Standard_Real ang,
    TopoDS_Shape& E, TopoDS_Shape& V1, TopoDS_Shape& V2
)
{
    // 构建坐标系，然后沿着向量平移
    gp_Ax2 Origin = gp::XOY();
    gp_Vec Offset(x, y, 0.);
    Origin.Translate(Offset);
    
    // 使用 gp_Circ 对象构造圆弧边
    BRepBuilderAPI_MakeEdge ME(gp_Circ(Origin,R), ang, ang+PI/2);
    E = ME;
    V1 = ME.Vertex1();
    V2 = ME.Vertex2();
}

TopoDS_Wire MakeFilletedRectangle(
    const Standard_Real H,
    const Standard_Real L,
    const Standard_Real R)
{
    // 形状数组
    TopTools_Array1OfShape theEdges(1,8);
    TopTools_Array1OfShape theVertices(1,8);

	// 构建圆弧边和顶点，存放到数组中
    Standard_Real x = L/2 - R, y = H/2 - R;
    MakeArc(x,-y,R,3.*PI/2.,theEdges(2),theVertices(2),theVertices(3));
    MakeArc(x,y,R,0.,theEdges(4),theVertices(4),theVertices(5));
    MakeArc(-x,y,R,PI/2.,theEdges(6),theVertices(6),theVertices(7));
    MakeArc(-x,-y,R,PI,theEdges(8),theVertices(8),theVertices(1));
    
    // 创建直线边和顶点
    for (Standard_Integer i = 1; i <= 7; i += 2)
    {
        theEdges(i) = BRepBuilderAPI_MakeEdge(
            TopoDS::Vertex(theVertices(i)),
            TopoDS::Vertex(theVertices(i+1))
        );
    }
    
    // 创建 wire，通过 Add 方法构建
    BRepBuilderAPI_MakeWire MW;
    for (i = 1; i <= 8; i++)
        MW.Add(TopoDS::Edge(theEdges(i)));
    
    return MW.Wire();
}
```

生成的 wire 模型如图所示

![](Screenshot_20230529_212528.jpg|525)



#### Polygon

BRepBuilderAPI_MakePolygon 类用于通过顶点或点建立多边的 wire 模型。这里点会自动地转换为 BRepBuilderAPI_MakeEdge 类中使用的顶点，因此能够快速建立多边形。例如

```cpp
#include <TopoDS_Wire.hxx>
#include <BRepBuilderAPI_MakePolygon.hxx>
#include <TColgp_Array1OfPnt.hxx>

TopoDS_Wire ClosedPolygon(const TColgp_Array1OfPnt& Points)
{
    BRepBuilderAPI_MakePolygon MP;
    
    // 依次添加点
    for(Standard_Integer i=Points.Lower();i=Points.Upper();i++)
    	MP.Add(Points(i));
    
    // 封闭模型，产生多边形
    MP.Close();
    return MP;
}
```

如果不调用 `Close()` 方法，会得到开放的多边形。



可以在定义时指定是否开放

```cpp
TopoDS_Wire W = BRepBuilderAPI_MakePolygon(V1,V2,V3,Standard_True);		// 封闭三角形
TopoDS_Wire W = BRepBuilderAPI_MakePolygon(P1,P2,P3,P4);				// 开放四边形
```

随时可以通过 `Edge(), FirstVertex(), LastVertex()` 获得 wire 中的边和顶点信息。如果两个顶点在相同位置，则不会创建边。



#### Face

BRepBuilderAPI_MakeFace 类通过曲面和 wire 创建面。一个面可以通过曲面和 4 个参数确定，如果没有提供参数，则使用曲面的自然边界。一个 wire 最多通过 4 条边和顶点创建。例如

```cpp
Handle(Geom_Surface) S = ...;

// 指定曲面和参数范围
Standard_Real umin,umax,vmin,vmax;
TopoDS_Face F = BRepBuilderAPI_MakeFace(S,umin,umax,vmin,vmax);
```



可以使用 gp 包中定义的曲面来创建面，并且可以在面上添加 wire

```cpp
gp_Cylinder C = ..;
TopoDS_Wire W = ...;

// 使用 gp_Cylinder 创建面，然后添加 wire
BRepBuilderAPI_MakeFace MF(C);
MF.Add(W);

// 也可以写在一起构建
BRepBuilderAPI_MakeFace MF(C,W);

// 获得拓扑面
TopoDS_Face F = MF;
```

一个面可以任意添加**不相交**的 wire，并且它们只定义表面上的一个区域。面上的边必须有参数曲面描述。如果面上 wire 边界没有参数形式，会通过投影计算。



平面可以通过定义了平面的 wire 直接构建。例如

```cpp
#include <TopoDS_Face.hxx>
#include <TColgp_Array1OfPnt.hxx>
#include <BRepBuilderAPI_MakePolygon.hxx>
#include <BRepBuilderAPI_MakeFace.hxx>

TopoDS_Face PolygonalFace(const TColgp_Array1OfPnt& thePnts)
{
    BRepBuilderAPI_MakePolygon MP;
    
    // 通过顶点构造 Polygon
    for(Standard_Integer i=thePnts.Lower();i<=thePnts.Upper(); i++)
    	MP.Add(thePnts(i));
    
    // 获得封闭 Polygon
    MP.Close();
    
    // 通过 Polygon 构成的 wire 创建平面
    TopoDS_Face F = BRepBuilderAPI_MakeFace(MP.Wire());
    return F;
}
```



MakeFace 的最后一个用处是**复制一个已存在的面**然后添加 wire，例如

```cpp
TopoDS_Face F = ...; // a face
TopoDS_Wire W = ...; // a wire
F = BRepBuilderAPI_MakeFace(F,W);
```



#### Wire

Wire 不是从几何构建，而是一种复合形状。BRepBuilderAPI_MakeWire 通过一系列边创建 Wire，例如

```cpp
TopoDS_Wire W = BRepBuilderAPI_MakeWire(E1,E2,E3,E4);
```

BRepBuilderAPI_MakeWire 会自动判断新的边是否与之前的边共用了顶点，如果是，就会将其连接到 wire；否则，算法将搜索位于相同位置的边的顶点或 wire 顶点。如果存在这样的顶点，就会用该顶点替换掉边上的顶点来实现连接；如果不存在，就说明 wire 发生断裂，将会报错。



对于更多边的 wire 使用 `Add()` 方法添加

```cpp
// 一列边，通过形状数组存放
TopTools_Array1OfShapes theEdges;

// 添加边，注意这里通过 TopoDS::Edge 来将 Shape 转换为 Edge
BRepBuilderAPI_MakeWire MW;
for (Standard_Integer i = theEdge.Lower();i <= theEdges.Upper(); i++)
	MW.Add(TopoDS::Edge(theEdges(i));

TopoDS_Wire W = MW;
```

甚至可以向 wire 中添加 wire

```cpp
#include <TopoDS_Wire.hxx>
#include <BRepBuilderAPI_MakeWire.hxx>

TopoDS_Wire MergeWires (const TopoDS_Wire& W1,const TopoDS_Wire& W2)
{
    BRepBuilderAPI_MakeWire MW(W1);
    MW.Add(W2);
    return MW;
}
```



### 修改物体

BRepBuilderAPI_Transform 类用于对形状施加变换。例如

```cpp
TopoDS_Shape myShape1 = ...;
TopoDS_Shape myShape2 = ...;

// 通过 gp_Trsf 设置变换方法：绕 gp_Ax1 给出的轴，旋转指定角度
gp_Trsf T;
T.SetRotation(gp_Ax1(gp_Pnt(0.,0.,0.),gp_Vec(0.,0.,1.)),2.*PI/5.);

// 通过 gp_Trsf 构造变换算法
BRepBuilderAPI_Transformation theTrsf(T);

// 传入形状 1，并对它施加变换。该变换只修改位置，变换后形状完全相同
theTrsf.Perform(myShape1);
TopoDS_Shape myNewShape1 = theTrsf.Shape();
    
// 传入形状 2，将复制一份形状，然后施加变换
theTrsf.Perform(myShape2,Standard_True);
TopoDS_Shape myNewShape2 = theTrsf.Shape();
```

如果想要复制形状，使用

```cpp
TopoDS_Solid myCopy = BRepBuilderAPI_Copy(mySolid);
```



### 基元

BRepPrimAPI 包提供构造基元（Boxes, Cones, Cylinders, Prisms）的 API ，这些基元可用于创建子形状。



#### Box

BRepPrimAPI_MakeBox 创建一个平行六面体盒子，得到 Shell 或 Solid 。有几种构造方法

![](Screenshot_20230529_221537.jpg|500)



#### Rotation

BRepPrimAPI_MakeOneAxis 用于构造旋转体。通过将曲线绕轴旋转得到旋转基元，可以是 Solid, Shell 或 Face 。需要注意我们并不直接使用这个类，而是使用其子类，它提供了改进的结构。下图展示了 MakeOneAxis 类的构造参数

![](Screenshot_20230529_221847.jpg|500)



#### Cylinder

BRepPrimAPI_MakeCylinder 通过默认坐标系或给定的 `gp_Ax2` 坐标系构建圆柱，例如

```cpp
Standard_Real X = 20, Y = 10, Z = 15, R = 10, DY = 30;

// 构造坐标系
gp_Ax2 axes = gp::ZOX();
axes.Translate(gp_Vec(X,Y,Z));

// 创建圆柱面
TopoDS_Face F = BRepPrimAPI_MakeCylinder(axes,R,DY,PI/2.);
```

![](Screenshot_20230529_222055.jpg|500)



#### Cone

BRepPrimAPI_MakeCone 创建圆台基元。其构造方法与 Cylinder 类似，例如

```cpp
Standard_Real R1 = 30, R2 = 10, H = 15;
TopoDS_Solid S = BRepPrimAPI_MakeCone(R1,R2,H);
```

可以创建 Shell 或 Solid 。

![](Screenshot_20230529_223009.jpg|500)



#### Sphere

BRepPrimAPI_MakeSphere 创建球基元。我们可以创建球面的一部分，例如

```cpp
Standard_Real R = 30, ang = PI/2, a1 = -PI/2.3, a2 = PI/4;
TopoDS_Solid S1 = BRepPrimAPI_MakeSphere(R);
TopoDS_Solid S2 = BRepPrimAPI_MakeSphere(R,ang);
TopoDS_Solid S3 = BRepPrimAPI_MakeSphere(R,a1,a2);
TopoDS_Solid S4 = BRepPrimAPI_MakeSphere(R,a1,a2,ang);
```

可以创建 Shell 或 Solid 。

![](Screenshot_20230529_223144.jpg|500)



#### Torus

BRepPrimAPI_MakeTorus 创建环面基元。例如

```cpp
Standard_Real R1 = 30, R2 = 10, ang = PI, a1 = 0, a2 = PI/2;
TopoDS_Shell S1 = BRepPrimAPI_MakeTorus(R1,R2);
TopoDS_Shell S2 = BRepPrimAPI_MakeTorus(R1,R2,ang);
TopoDS_Shell S3 = BRepPrimAPI_MakeTorus(R1,R2,a1,a2);
TopoDS_Shell S4 = BRepPrimAPI_MakeTorus(R1,R2,a1,a2,ang);
```

![](Screenshot_20230529_223400.jpg|500)



#### Sweeping

扫描 Sweeping 是通过指定路径扫描轮廓获得的对象。路径 Path 通常是一条曲线或直线，沿着路径可以通过

* 顶点生成边；
* 边生成面；
* Wire 生成 Shell；
* Face 生成 Solid；
* Shell 生成复合 Solid；

![](Screenshot_20230529_224052.jpg|500)



BRepPrimAPI_MakeSweep 类由对应的三个扫描子类

* BRepPrimAPI_MakePrism 线扫描

```cpp
TopoDS_Face F = ..;		// 基础面
gp_Dir direc(0,0,1);	// 扫描方向
Standard_Real l = 10;	// 扫描长度

// 构建有限长度的扫描向量
gp_Vec v = direc;
v *= l;

TopoDS_Solid P1 = BRepPrimAPI_MakePrism(F,v);						// 有限扫描
TopoDS_Solid P2 = BRepPrimAPI_MakePrism(F,direc);					// 无限扫描
TopoDS_Solid P3 = BRepPrimAPI_MakePrism(F,direc,Standard_False);	// 半无限扫描
```

![](Screenshot_20230529_224426.jpg|500)

* BRepPrimAPI_MakeRevol 旋转扫描

```cpp
TopoDS_Face F = ...;						// 基础面
gp_Ax1 axis(gp_Pnt(0,0,0),gp_Dir(0,0,1));	// 转轴
Standard_Real ang = PI/3;					// 旋转角

TopoDS_Solid R1 = BRepPrimAPI_MakeRevol(F,axis);		// 完全旋转
TopoDS_Solid R2 = BRepPrimAPI_MakeRevol(F,axis,ang);	// 角度旋转
```

![](Screenshot_20230529_224611.jpg|500)

* BRepPrimAPI_MakePipe 一般扫描



### 布尔运算

布尔运算用来组合两组形状来创建新形状。包括并 Fuse，交 Common，差 Cut 三种运算。

![](Screenshot_20230529_225046.jpg|500)

拓扑操作是创建真实工业零件最便捷的方法。因为大多数工业零件由几个简单的元件组成，通过可以很容易单独创建，最后通过布尔运算将它们进行组合。



## STEP

STEP 是一种常用的交互数据格式，被多种软件所使用。



### 读取 STEP

被识别的 STEP 表示实体类型有

* advanced_brep_shape_representation
* faceted_brep_shape_representation
* manifold_surface_shape_representation
* geometrically_bounded_wireframe_shape_representation
* geometrically_bounded_surface_shape_representation
* hybrid representations (shape_representation containing models of different type)

可被翻译的 STEP 拓扑实体包括

* vertices
* edges
* loops
* faces
* shells
* solids

可被翻译的 STEP 几何实体包括

* points
* vectors
* directions
* curves
* surfaces



读取 STEP 文件的流程如下

```cpp
STEPControl_Reader reader;
IFSelect_ReturnStatus stat = reader.ReadFile("filename.stp");

// 使用指定模式读取
reader.PrintCheckLoad(failsonly,mode);
```

其中如果 failsonly 参数为 true，则只显示失败消息，否则显示所有消息；mode 参数设置面向信息或面向实体的分析方法，可选参数

* IFSelect_ItemsByEntity 对每个 STEP 实体给出所有消息的序列
* IFSelect_CountByItem 每条消息给出 STEP 实体的数量和类型
* IFSelect_ListByItem 每条消息给出 STEP 实体的数量、类型和排序



#### 翻译参数

可以设置翻译参数控制翻译方式，存在多种控制参数。以读取精度参数为例

* read.precision.mode 控制选择哪种读取模式
* read.precision.val 控制读取精度

我们可以获得和修改读取模式

```cpp
Standard_Integer ic = Interface_Static::IVal("read.precision.mode");	// 获得 read.precision.mode
if(!Interface_Static::SetIVal("read.precision.mode",1))					// 设置 read.precision.mode 为 1
	// error
```

其中 mode 0 表示按照文件中数据的精度读取；mode 1 表示按照 read.precision.val 给出的精度读取。在 mode 1 下设置

```cpp
Standard_Real rp = Interface_Static::RVal("read.precision.val");	// 获得 read.precision.val
if(!Interface_Static::SetRVal("read.precision.val",0.01))			// 设置 read.precision.val 为 0.01
	// error
```

当然，读取精度不会超出文件中的数据精度。



#### 读取内容

可以根据需要选择要翻译的内容，通过根实体或在 STEP 文件中的序号来选择内容。使用方法

* `Standard_Boolean ok = reader.TransferRoot(rank)` – 翻译通过序号给出的根实体
* `Standard_Boolean ok = reader.TransferOne(rank)` – 翻译通过序号给出的实体
* `Standard_Integer num = reader.TransferList(list)` – 翻译一列实体，返回成功的数量
* `Standard_Integer NbRoots = reader.NbRootsForTransfer()` 获得可翻译的根实体数量
* `Standard_Integer num = reader.TransferRoots()` 翻译所有根实体，返回成功的数量

当执行翻译操作 `TransferRoot() TransferOne() TransferList()` 后，将会得到一列形状。每次操作的结果会累计，因此最好在相邻两次操作之间清除之前的结果

```cpp
reader.ClearShapes();
```

使用 `TransferRoots()` 进行翻译的话，会自动清除之前的结果。



每一步翻译可以检查翻译信息

```cpp
reader.PrintCheckTransfer(failsonly,mode);
```

它检查最后一步翻译的各种信息。



要获得翻译内容，使用

* `Standard_Integer num = reader.NbShapes()` – 获得结果中的形状数量
* `TopoDS_Shape shape = reader.Shape(rank)` – 根据排列序号获得形状，序号不大于 `NbShapes()`
* `TopoDS_Shape shape = reader.Shape()` – 获得第一个翻译结果
* `TopoDS_Shape shape = reader.OneShape()` – 用**一个形状**保存所有翻译结果



#### 读取示例

```cpp
#include <STEPControl_Reader.hxx>
#include <TopoDS_Shape.hxx>
#include <BRepTools.hxx>

Standard_Integer main()
{
    STEPControl_Reader reader;
    reader.ReadFile("MyFile.stp");
    
    // 获得可翻译的根实体数量
    Standard_Integer NbRoots = reader.NbRootsForTransfer();
    
    // 翻译所有实体，获得成功的数量
    Standard_Integer NbTrans = reader.TransferRoots();
    
    // 将翻译结果存放到一个形状中
    TopoDS_Shape result = reader.OneShape();
}
```



### 写入 STEP

使用写入对象进行写入操作

```cpp
STEPControl_Writer writer;
```



#### 翻译参数

可以设置翻译参数控制翻译方式，存在多种控制参数。以写入精度参数为例

* write.precision.mode 控制选择哪种写入模式
* write.precision.val 控制写入精度

我们可以获得和修改写入模式

```cpp
Standard_Integer ic = Interface_Static::IVal("write.precision.mode");
if(!Interface_Static::SetIVal("write.precision.mode",1))
	// error
```

其中 mode

* -1 设为一个 OCC 形状的最小容差；
* 0 设为一个 OCC 形状的平均容差；
* 1 设为一个 OCC 形状的最大容差；
* 2 根据 write.precision.val 的值；

类似于读取参数，设置

```cpp
Standard_Real rp = Interface_Static::RVal("write.precision.val");
if(!Interface_Static::SetRVal("write.precision.val",0.01))
	// error
```



#### 写入内容

枚举类型 STEPControl_StepModelType 用来定义翻译模型。可选

* `STEPControl_AsIs` 根据 OCC 形状类型自动选择生成表示，可转换任意 OCC 形状；
* `STEPControl_ManifoldSolidBrep` 生成实体为 manifold_solid_brep 或 brep_with_voids
* `STEPControl_FacetedBrep` 生成实体为 faceted_brep 或 faceted_brep_and_brep_with_voids 
* `STEPControl_ShellBasedSurfaceModel` 生成实体为 shell_based_surface_model
* `STEPControl_GeometricCurveSet` 生成实体为 geometric_curve_set

转换流程为

```cpp
STEPControl_StepModelTope mode = STEPControl_ManifoldSolidBrep;
IFSelect_ReturnStatus stat = writer.Transfer(shape,mode);

// 最后写入文件
IFSelect_ReturnStatus stat = writer.Write("filename.stp");
```



#### 写入示例

```cpp
#include <STEPControl.hxx>
#include <STEPControl_Writer.hxx>
#include <TopoDS_Shape.hxx>
#include <BRepTools.hxx>
#include <BRep_Builder.hxx>

Standard_Integer main()
{
	TopoDS_Solid source;
    STEPControl_Writer writer;
    
    // 翻译为 manifold_solid_brep 实体
    writer.Transfer(source, STEPControl_ManifoldSolidBrep);
    
    // 写入文件
    writer.Write("filename.stp");
}
```





## AIS

应用程序交互服务（AIS）提供了在应用程序 GUI 查看器和用于管理选择和演示的包之间创建链接的方法，这使得在3D中对这些功能的管理更加直观，从而更加透明。

![](Screenshot_20230525_194310.jpg|500)



### 基本概念

#### 表示

在 OCC 中，表示服务与数据分离，后者由应用算法生成。这种分离使得我们可以修改几何或拓扑算法以及产生的对象，而无需修改可视化服务。



物体表示涉及如下几部分：

* 可表示的物体 AIS_InteractiveObject
* 观察者
* 交互式上下文 AIS_InteractiveContext

表示涉及 AIS, PrsMgr, StdPrs 和 V3d 包

* AIS 提供交互式对象；
* PrsMgr 提供低级服务接口，只在不使用 AIS 服务时使用；
* StdPrs 对特定几何体提供现成的标准表示算法；
* Prs3d 提供通用表示算法：线框、阴影、隐藏线消除等。控制表示对象的属性，如颜色、线条类型、厚度等；
* V3d 提供 3D 可视化服务；



基本的表示流程为

```cpp
// 创建可视化对象，利用该对象创建交互上下文
Handle(V3d_Viewer) theViewer;
Handle(AIS_InteractiveContext) aContext = new AIS_InteractiveContext (theViewer);

// 创建一个可表示的物体，获得拓扑形状
BRepPrimAPI_MakeWedge aWedgeMaker (theWedgeDX, theWedgeDY, theWedgeDZ, theWedgeLtx);
TopoDS_Solid aShape = aWedgeMaker.Solid();

// 利用形状创建 AIS 表示对象，通过交互上下文显示
Handle(AIS_Shape) aShapePrs = new AIS_Shape (aShape);
aContext->Display (aShapePrs, AIS_Shaded, 0, true);
```

其中 Display() 方法将调用 AIS 表示对象的 Compute 算法获得表示数据，然后传送给可视化对象。

![](Screenshot_20230527_130126.jpg|500)

由于 AIS_InteractiveObject  是**抽象类**，在实际应用中，**使用继承的 AIS_Shape 类对显示对象进行操作**。



#### 选择

标准 OCCT 选择算法分为动态和静态两部分。动态选择会在鼠标移动到对象上时，自动高亮显示对象；静态选择允许选择特定的对象进行进一步处理。有三类选择：

* 点选：允许高亮显示单个物体或其部分；
* 框选：通过鼠标移动框选物体；
* 多边形选择：用户自定义选择非自交多边形选取物体；

对于 OCCT 选择算法，所有可选择对象表示为一组敏感区域，称为**敏感实体 sensitive entities** 。当鼠标在视图中移动，会对每个对象的敏感实体进行碰撞检测。任何物体都会被分割为敏感实体集合用来检测。

![](Screenshot_20230527_130719.jpg|500)

每个敏感实体都储存所有者的引用，并且所有者可以存储额外信息，例如敏感实体的拓扑形状。突出显示的颜色和方法等。



可选择对象存储所有已创建的选择模式和敏感实体的信息。被计算的实体被排列在一个选择中，然后添加到这个物体的选择列表中。在永久删除对象之前，不会从列表中删除任何选择。可以设置选择机制 `AIS_Shape::SelectionMode()`：

* 0 选择整个物体 `AIS_Shape`；
* 1 选择顶点 `TopAbs_VERTEX`；
* 2 选择边 `TopAbs_EDGE`；
* 3 选择 wires `TopAbs_WIRE`；
* 4 选择面 `TopAbs_FACE`；
* 5 选择 shells `TopAbs_SHELL`；
* 6 选择组成固体 `TopAbs_SOLID`；

![](Screenshot_20230527_131513.jpg|500)



### 应用交互式服务

#### AIS_Shape

交互对象 `Interactive Object` 是虚拟实体，可以被表示和选取。一个交互对象可以具有一定数量的特定图形属性，例如可视化模式、颜色和材质。它可以从自己的绘图设备中 `Prs3d_Drawer` 中获得所需的图形属性。

* Highlight Mode：在动态监测时，交互式上下文输出的表示和默认表示相同。因此对于 `AIS_Shape` 需要使用 `SetHilightMode` 和 `UnsetHilightMode` 指定高亮显示模式，该模式独立于一般的活动表示；

可能需要过滤要选择的实体，可以通过 `Filter` 实体细化检测上下文。

![](Screenshot_20230527_145803.jpg|500)

在 AIS 视图中被可视化和选择的实体是对象。它们将模型的底层几何形状与 AIS 中的图形表示链接起来。每个 `AIS_Shape` 对象都具有一个子对象 `myChildren` 的列表，任何针对当前对象的变换都会应用到子对象上。可以调整子对象关系

* `AddChild()`；
* `RemoveChild()`；

继承自 `PrsMgr_PresentableObject` 的类都具有此特性。



#### Prs3d_Drawer

图形属性管理器，即 `Prs3d_Drawer` 类存放特定交互式对象和由交互式上下文控制的图形属性。绘图时从自带的 `Prs3d_Drawer` 中获得图形属性；如果不存在对应的 `Prs3d_Drawer`，则从交互式上下文的 `Prs3d_Drawer` 中获得图形属性。

* 每个交互式对象都可以有自己的 `Prs3d_Drawer`；
* 默认采用交互式上下文的 `Prs3d_Drawer`；
* 在 `AIS_InteractiveObject` 抽象类中，包括颜色、线条粗细、材料和透明度在内的标准属性可以修改。存在对应的虚函数设定对这些属性的操作，继承自它的交互对象类中重定义这些函数可以改变类的行为；

只需重定义 `Prs3d_Drawer` 类中的虚函数，就可以改变 `AIS_TextLabel` 和 `AIS_Shape` 对象的属性。

![](Screenshot_20230527_155328.jpg|500)



#### AIS_InteractiveContext

应用程序交互服务允许以简单透明的方式在视图中管理演示和动态选择，其中心实体是交互式上下文 `Interactive Context`，它与主视图连接。默认情况下，交互式上下文将每个对象作为整体拾取，但用户可以激活对部分的选取。

* Display Mode：使用 `SetDisplayMode` 和 `UnsetDisplayMode` 可以指定演示模式；

注意**如果一个交互对象已经被放入交互式上下文，则对该对象的调整只能通过上下文函数实现**。例如

```cpp
// 使用交互式上下文设置对象显示模式
Handle(AIS_Shape) aShapePrs = new AIS_Shape (theShape);
myIntContext->Display(aShapePrs, AIS_Shaded, 0, false, aShapePrs->AcceptShapeDecomposition());
myIntContext->SetColor(aShapePrs, Quantity_NOC_RED);

// 先修改对象显示模式，然后直接载入到交互式上下文
Handle(AIS_Shape) aShapePrs = new AIS_Shape (theShape);
aShapePrs->SetColor (Quantity_NOC_RED);
aShapePrs->SetDisplayMode (AIS_Shaded);
myIntContext->Display (aShapePrs);
```



交互式上下文也具有 `Prs3d_Drawer` 来存放其控制对象的属性。可以调整显示和选择行为

* 默认 `Drawer`：包含交互对象可以使用的所有颜色和线条信息，它们没有自己的属性；
* 默认可视化模式：默认 mode 0；
* 鼠标移动时的实体高亮色：默认 Quantity_NOC_CYAN1；
* 预选择颜色：默认 Quantity_NOC_GREEN；
* 选择颜色（点击被检测的物体）：默认 Quantity_NOC_GRAY80；

例如

```cpp
Ctx->Display (obj1, false);				// obj1 的显示属性不更新
Ctx->Display (obj2, true);				// obj2 的显示属性会更新
Ctx->SetDisplayMode (obj1, 3, false);	// 设置 obj1 的显示模式为 3，步更新属性
Ctx->SetDisplayMode (2, true);			// 更新上下文的显示模式
```

这里 obj1 的显示属性设为不更新 `false`，因此它不受到最后对上下文显示模式设置的影响

* obj1 保持 mode 3 显示；
* obj2 更新为 mode 2 显示；

注意这里只是为了方便演示，在实际设置显示模式时，应当使用 AIS_Display 的枚举变量而非整型。



AIS_Shape 是最常用的交互对象。它提供了 API 管理形状组成元素的选择操作（顶点、边、面的选择）。

* `SelectionMode()` 方法返回指定形状类型的选择模式；

当交互式上下文的 `Display()` 方法没有设置模式，就会使用交互对象默认的模式。可以激活对象的默认选择模式

* `Activate()` 激活指定的模式；
* `Deactivate()` 去激活指定的模式；

可以激活多个模式。可激活的模式列表通过 `ActivatedModes()` 方法获得。



#### SelectMgr_Filter

要定义动态检测的环境，可以使用标准过滤器或自定义。过滤器的基类是 `SelectMgr_Filter`，它会访问敏感单元的所有者检查它是否具有所需要的性质，如果没有就会被去除。标准过滤器有

* `StdSelect_EdgeFilter` 过滤边（线、圆）；
* `StdSelect_FaceFilter` 过滤面（平面、柱面、球面）；
* `StdSelect_ShapeTypeFilter` 过滤指定类型的形状；
* `AIS_TypeFilter` 指定类型的交互对象；
* `AIS_SignatureFilter` 指定类型和签名的交互对象；
* `AIS_AttributeFilter` 指定属性的交互对象；

这些类通过同名的头文件引用。还可以

* 通过实现 `IsOK()` 方法创建自定义过滤器；
* 通过 `AND, OR` 过滤器来组合多个过滤器；



要使用过滤器，只需要将其添加到交互式上下文中

* `AddFilter()` 添加指定的过滤器；
* `RemoveFilter()` 移除指定的过滤器；
* `RemoveFilters()` 移除列出的过滤器；
* `Filters()` 获得过滤器列表；

例如增加面的过滤器

```cpp
// 着色可视化模式，无特定模式，可分解为子形状
const TopoDS_Shape theShape;
Handle(AIS_Shape) aShapePrs = new AIS_Shape (theShape);
myContext->Display (aShapePrs, AIS_Shaded, -1, true, true);

// 获得选择面的模式的记号，通过这个记号激活对应的模式
const int aSubShapeSelMode = AIS_Shape::SelectionMode (TopAbs_Face);
myContext->Activate (aShapePrs, aSubShapeSelMode);

// 创建旋转面和平面的过滤器，然后添加到上下文中
Handle(StdSelect_FaceFilter) aFil1 = new StdSelect_FaceFilter (StdSelect_Revol);
Handle(StdSelect_FaceFilter) aFil2 = new StdSelect_FaceFilter (StdSelect_Plane);
myContext->AddFilter (aFil1);
myContext->AddFilter (aFil2);

// 检测视图中指定的位置，将只选择旋转面或平面
myContext->MoveTo (thePixelX, thePixelY, myView, true);
```



#### 获得选择形状

动态检测和选择通过直接调用函数的方式实现

* `MoveTo()` 将鼠标位置传递给交互式上下文选择器；
* `Select()` 将之前检测到物体添加到拾取列表中，如果该物体已经在列表中，则将其从列表中删除；
* `ShiftSelect()` 如果最后一次移动检测到的物体没有被选中，则添加到拾取列表中，否则撤回；

检测和选择实体的高亮显示由交互式上下文自动管理。



可以查询上下文

* `HasDeteceted()` 检查是否有被检测的实体；
* `DetectedOwner()` 返回当前高亮的被检测实体；

 使用 `Select()` 和 `ShiftSelect()` 函数后，就可以检查拾取列表

* `InitSelected()` 初始化一个迭代器；
* `MoreSelected()` 检查一个迭代器是否有效；
* `NextSelected()` 移动到下一个迭代器；
* `SelectedOwner()` 返回当前迭代器位置的实体；

所有者对象 `SelectMgr_EntityOwner` 是检查被选择实体的关键对象，通过 `DetectedOwner()` 和 `SelectedOwner()` 获得。交互对象自身可以通过 `SelectMgr_EntityOwner` 的 `Selectable()` 获得，同时根据交互对象的类型标识其子部分。在 `AIS_Shape` 的情况下，子形状通过 `StdSelect_BRepOwner` 的 `Shape()` 方法获得。例如

```cpp
// 通过迭代器遍历选取列表
for (myAISCtx->InitSelected(); myAISCtx->MoreSelected(); myAISCtx->NextSelected())
{
    // 获得当前迭代器位置的所有者对象
    Handle(SelectMgr_EntityOwner) anOwner = myAISCtx->SelectedOwner();
    
    // 获得所有者对象持有的交互对象
    Handle(AIS_InteractiveObject) anObj = Handle(AIS_InteractiveObject)::DownCast (anOwner->Selectable());
    
    // 将 SelectMgr_EntityOwner 对象下降为 StdSelect_BRepOwner 对象
    if (Handle(StdSelect_BRepOwner) aBRepOwner = Handle(StdSelect_BRepOwner)::DownCast (anOwner))
    {
        // 然后通过 StdSelect_BRepOwner 对象获得能够使用拾取的形状
        TopoDS_Shape aShape = aBRepOwner->Shape();
    }
}
```

注意 AIS_Shape 和 TopoDS_Shape 的关系，前者是 AIS 的交互对象，后者是 OCC 的形状数据结构。



#### 显示视图坐标轴

在初始化创建 `V3d_Viewer` 时，可以通过下面两行代码在左下角显示一个视图坐标轴

```cpp
// 通过 V3d_Viewer 创建视口
m_hView = m_hViewer->CreateView();
m_hView->ZBufferTriedronSetup(Quantity_NOC_RED, Quantity_NOC_GREEN, Quantity_NOC_BLUE, 0.8, 0.05, 12);
m_hView->TriedronDisplay(Aspect_TOTP_LEFT_LOWER, Quantity_NOC_WHITE, 0.2, V3d_ZBUFFER);
```



### MFC 文档结构

下面展示使用 AIS 结合 MFC 进行可视化的经典示例，显示效果如下

![](image-20230525144013332.png)



#### 项目文件结构

└─src
    └─OCCFrame

在 OCCFrame 中定义 MFC 的各种类。在主目录下配置 cmake 文件

```cmake
cmake_minimum_required(VERSION 3.18)

# 增加子目录 src
add_subdirectory(src)
```

在 src 目录下配置 cmake 文件

```cmake
cmake_minimum_required(VERSION 3.18)

project(MAIN)

# 增加 AFX 动态库
add_definitions(-D_AFXDLL)

# 设置 MFC 库类型
set(CMAKE_MFC_FLAG 2)

# 当前目录下的主源文件
file(GLOB G_SRC CONFIGURE_DEPENDS *.cpp)
file(GLOB G_INC CONFIGURE_DEPENDS *.h)
file(GLOB G_RC CONFIGURE_DEPENDS *.rc)

# OCCFrame 下的源文件
file(GLOB OCCFRAME_SRC CONFIGURE_DEPENDS OCCFrame/*.cpp)
file(GLOB OCCFRAME_INC CONFIGURE_DEPENDS OCCFrame/*.h)

# 分组为 OCCFrame
source_group(OCCFrame FILES ${OCCFRAME_SRC} ${OCCFRAME_INC})

# 同时将 .cpp 和 .h 文件都作为依赖项
add_executable(${PROJECT_NAME} 
    ${G_SRC} ${G_INC} ${G_RC}
    ${OCCFRAME_INC} ${OCCFRAME_SRC}
    )

set(OPENCASCADE_DIR "D:/lib/opencascade-7.7.0/cmake")
find_package(OPENCASCADE REQUIRED)

target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/src)
target_include_directories(${PROJECT_NAME} PRIVATE ${OpenCASCADE_INCLUDE_DIR})
target_link_libraries(${PROJECT_NAME} PRIVATE ${OpenCASCADE_LIBRARY_DIR}d/*.lib)

set_target_properties(${PROJECT_NAME} PROPERTIES WIN32_EXECUTABLE ON)

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

# 将依赖的动态库复制到输出目录 DEPS_DLL
file(GLOB DEPS_DLL CONFIGURE_DEPENDS ${OpenCASCADE_BINARY_DIR}d/*.dll)

add_custom_command(
  TARGET ${PROJECT_NAME}
  POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy ${DEPS_DLL} ${OUTPUT_DIR})

```



#### OCCDoc

创建文档类，用于管理可视化资源

```cpp
#pragma once

#include <afxext.h>
#include <afxwin.h>

// 渲染头文件
#include <Prs3d_LineAspect.hxx>
#include <Prs3d_ShadingAspect.hxx>

// 图形交互
#include <AIS_InteractiveContext.hxx>
#include <OpenGL_GraphicDriver.hxx>
#include <V3d_Viewer.hxx>

// 光照头文件
#include <V3d_AmbientLight.hxx>
#include <V3d_DirectionalLight.hxx>

// 形状头文件
#include <AIS_Shape.hxx>
#include <TopTools_SequenceOfShape.hxx>
#include <TopoDS_Shape.hxx>

// 自定义文档类
class OCCDoc : public CDocument {
  DECLARE_DYNCREATE(OCCDoc)
protected:
  Handle(V3d_Viewer) m_hViewer;                 // 定义 V3d_Viewer 句柄
  Handle(AIS_InteractiveContext) m_hAisContext; // 定义交互上下文

  TopTools_SequenceOfShape m_shapes; // 存放拓扑形状

public:
  Handle(V3d_Viewer) GetViewer();              // 用于获得 V3d_Viewer
  Handle(AIS_InteractiveContext) GetContext(); // 用于获得上下文
  int GetShapeNum();                           // 获得形状数量
  const TopoDS_Shape &GetShape(int i);         // 获得指定索引的形状

  void UpdateInteractiveContext();          // 刷新上下文数据
  void AddShape(const TopoDS_Shape &shape); // 用于载入形状
  void AdjustSelectionStyle(const Handle(AIS_InteractiveContext) &
                            context); // 修改选择风格

  virtual BOOL OnNewDocument(); // 每次创建新文档都会调用
};
```

对应地实现这些方法

```cpp
#include "OCCDoc.h"

IMPLEMENT_DYNCREATE(OCCDoc, CDocument)

Handle(V3d_Viewer) OCCDoc::GetViewer() { return m_hViewer; }

Handle(AIS_InteractiveContext) OCCDoc::GetContext() { return m_hAisContext; }

int OCCDoc::GetShapeNum() { return m_shapes.Size(); }

const TopoDS_Shape &OCCDoc::GetShape(int i) { return m_shapes.Value(i); }

void OCCDoc::UpdateInteractiveContext() {
  // 移除所有物体
  m_hAisContext->RemoveAll(Standard_True);

  // 遍历每个形状绘制，并激活选择风格
  for (auto sh : m_shapes) {
    Handle(AIS_Shape) shape = new AIS_Shape(sh);
    m_hAisContext->Display(shape, Standard_True);
    m_hAisContext->SetDisplayMode(shape, AIS_Shaded, true);
  }

  // 修改选择风格
  AdjustSelectionStyle(m_hAisContext);

  // 激活选择
  m_hAisContext->Activate(4, true); // faces
  m_hAisContext->Activate(2, true); // edges

  // 刷新和这个文档类对象关联的所有视图窗口
  this->UpdateAllViews(NULL);
}

void OCCDoc::AddShape(const TopoDS_Shape &shape) {
  // 推入形状，然后立即更新
  m_shapes.Append(shape);
  UpdateInteractiveContext();
}

void OCCDoc::AdjustSelectionStyle(const Handle(AIS_InteractiveContext) &
                                  context) {
  // 设置渲染信息以及选中物体的高亮信息
  Handle(Prs3d_Drawer) selDrawer = new Prs3d_Drawer;

  selDrawer->SetLink(context->DefaultDrawer());
  selDrawer->SetFaceBoundaryDraw(true);
  selDrawer->SetDisplayMode(1);
  selDrawer->SetTransparency(0.5f);
  selDrawer->SetZLayer(Graphic3d_ZLayerId_Topmost);
  selDrawer->SetColor(Quantity_NOC_GOLD);
  selDrawer->SetBasicFillAreaAspect(new Graphic3d_AspectFillArea3d());

  // 调整填充区域颜色材质等信息
  const Handle(Graphic3d_AspectFillArea3d) &fillArea =
      selDrawer->BasicFillAreaAspect();

  fillArea->SetInteriorColor(Quantity_NOC_GOLD);
  fillArea->SetBackInteriorColor(Quantity_NOC_GOLD);

  fillArea->ChangeFrontMaterial().SetMaterialName(Graphic3d_NOM_NEON_GNC);
  fillArea->ChangeFrontMaterial().SetTransparency(0.4f);
  fillArea->ChangeBackMaterial().SetMaterialName(Graphic3d_NOM_NEON_GNC);
  fillArea->ChangeBackMaterial().SetTransparency(0.4f);

  selDrawer->UnFreeBoundaryAspect()->SetWidth(1.0);

  // 更新 AIS context
  context->SetHighlightStyle(Prs3d_TypeOfHighlight_LocalSelected, selDrawer);
}

BOOL OCCDoc::OnNewDocument() {
  if (!CDocument::OnNewDocument())
    return FALSE;

  // 创建 OpenGL 引擎
  Handle(Aspect_DisplayConnection) theDisp = new Aspect_DisplayConnection();
  Handle(OpenGl_GraphicDriver) H_OpenGL_GraphicDriver =
      new OpenGl_GraphicDriver(theDisp);

  // 创建视口
  m_hViewer = new V3d_Viewer(H_OpenGL_GraphicDriver);

  // 光照
  Handle(V3d_DirectionalLight) LightDir = new V3d_DirectionalLight(
      V3d_Zneg, Quantity_Color(Quantity_NOC_GRAY97), 1);
  Handle(V3d_AmbientLight) LightAmb = new V3d_AmbientLight();

  // 设置光照方向
  LightDir->SetDirection(1.0, -2.0, -10.0);

  // 添加光照
  m_hViewer->AddLight(LightDir);
  m_hViewer->AddLight(LightAmb);
  m_hViewer->SetLightOn(LightDir);
  m_hViewer->SetLightOn(LightAmb);

  // 创建交互上下文
  m_hAisContext = new AIS_InteractiveContext(m_hViewer);

  // 配置全局设置，渲染边界
  const Handle(Prs3d_Drawer) &contextDrawer = m_hAisContext->DefaultDrawer();
  if (!contextDrawer.IsNull()) {
    const Handle(Prs3d_ShadingAspect) &SA = contextDrawer->ShadingAspect();
    const Handle(Graphic3d_AspectFillArea3d) &FA = SA->Aspect();
    contextDrawer->SetFaceBoundaryDraw(true);
    FA->SetEdgeOff();

    // 设置最大绘制线路
    contextDrawer->SetMaximalParameterValue(1000);
  }

  // 更新视口
  UpdateInteractiveContext();

  return TRUE;
}
```



#### OCCView

创建视口，用于显示可视化资源

```cpp
#pragma once

#include "OCCDoc.h"

// 视口和窗口
#include <V3d_View.hxx>
#include <WNT_Window.hxx>

// 自定义视图窗口类
class OCCView : public CView {
  DECLARE_DYNCREATE(OCCView)
  DECLARE_MESSAGE_MAP()
protected:
  enum class Action3d {
    Action3d_Nothing,
    Action3d_Panning, // 平移
    Action3d_Zooming, // 缩放
    Action3d_Rotation // 旋转
  };

  Action3d m_mode; // 记录操作模式
  CPoint m_pos;    // 记录鼠标平移坐标

  Handle(V3d_View) m_hView; // 视口句柄

public:
  OCCDoc *GetDocument() const;

public:
  virtual void OnInitialUpdate();
  virtual void OnDraw(CDC *pDC);

  afx_msg BOOL OnEraseBkgnd(CDC *pDC);
  afx_msg void OnSize(UINT nType, int cx, int cy);
  afx_msg void OnSizing(UINT fwSide, LPRECT pRect);
  afx_msg void OnLButtonDown(UINT nFlags, CPoint point);
  afx_msg BOOL OnMouseWheel(UINT nFlags, short zDelta, CPoint pt);
  afx_msg void OnMouseMove(UINT nFlags, CPoint point);
  afx_msg void OnLButtonUp(UINT nFlags, CPoint point);
};
```

实现初始化、绘制等操作

```cpp
#include "OCCView.h"

IMPLEMENT_DYNCREATE(OCCView, CView)

BEGIN_MESSAGE_MAP(OCCView, CView)
ON_WM_ERASEBKGND()
ON_WM_SIZE()
ON_WM_SIZING()
ON_WM_LBUTTONDOWN()
ON_WM_MOUSEWHEEL()
ON_WM_MOUSEMOVE()
ON_WM_LBUTTONUP()
END_MESSAGE_MAP()

OCCDoc *OCCView::GetDocument() const {
  ASSERT(m_pDocument->IsKindOf(RUNTIME_CLASS(OCCDoc)));
  return (OCCDoc *)m_pDocument;
}

void OCCView::OnInitialUpdate() {
  CView::OnInitialUpdate();

  // 初始化变量
  m_mode = Action3d::Action3d_Nothing;
  m_pos = {0, 0};

  // 通过 V3d_Viewer 创建视口
  m_hView = GetDocument()->GetViewer()->CreateView();
  m_hView->SetImmediateUpdate(false);

  // 通过当前窗口句柄创建 WNT_Window，并将视口连接到 WNT_Window
  Handle(WNT_Window) H_WNT_Window = new WNT_Window(this->GetSafeHwnd());
  m_hView->SetWindow(H_WNT_Window);

  // 窗口映射
  if (H_WNT_Window->IsMapped())
    H_WNT_Window->Map();

  // 设置视口
  m_hView->MustBeResized();
  m_hView->SetShadingModel(V3d_PHONG);

  // 配置渲染参数，例如反走样、阴影等
  Graphic3d_RenderingParams &RenderParams = m_hView->ChangeRenderingParams();
  RenderParams.IsAntialiasingEnabled = true;
  RenderParams.NbMsaaSamples = 8;
  RenderParams.IsShadowEnabled = false;
  RenderParams.CollectedStats = Graphic3d_RenderingParams::PerfCounters_NONE;
}

void OCCView::OnDraw(CDC *pDC) {
  OCCDoc *pDoc = GetDocument();
  ASSERT_VALID(pDoc);
  if (!pDoc)
    return;

  // 重绘视口
  if (m_hView.IsNull() == FALSE)
    m_hView->Redraw();
}

BOOL OCCView::OnEraseBkgnd(CDC *pDC) {
  // 禁用重绘背景，防止闪烁
  return TRUE;
}

void OCCView::OnSize(UINT nType, int cx, int cy) {
  CView::OnSize(nType, cx, cy);

  // 调整视口形状
  if (m_hView.IsNull() == FALSE)
    m_hView->MustBeResized();
}

void OCCView::OnSizing(UINT fwSide, LPRECT pRect) {
  CView::OnSizing(fwSide, pRect);

  switch (m_hView->RenderingParams().StereoMode) {
  case Graphic3d_StereoMode_RowInterlaced:
  case Graphic3d_StereoMode_ColumnInterlaced:
  case Graphic3d_StereoMode_ChessBoard: {
    // 跟踪窗口反向移动
    m_hView->MustBeResized();
    m_hView->Update();
    break;
  }
  default:
    break;
  }
}

void OCCView::OnLButtonDown(UINT nFlags, CPoint point) {
  Handle(AIS_InteractiveContext) ctx = GetDocument()->GetContext();
  ctx->Select(Standard_True);

  if (ctx->Selection()->IsEmpty()) {
    // 如果未选中，则进入旋转模式
    m_mode = Action3d::Action3d_Rotation;
    m_hView->StartRotation(point.x, point.y);
  } else {
    // 若选中，则进入平移模式
    m_mode = Action3d::Action3d_Panning;
    m_pos = point;
  }

  CView::OnLButtonDown(nFlags, point);
}

BOOL OCCView::OnMouseWheel(UINT nFlags, short zDelta, CPoint pt) {
  // 缩放视口
  if (zDelta > 0) {
    double scar = 1.2;
    if (m_hView->Scale() <= 10000)
      m_hView->SetZoom(scar);
  } else {
    double scar = 0.8;
    if (m_hView->Scale() > 0.0001)
      m_hView->SetZoom(scar);
  }

  // 立即重绘
  InvalidateRect(NULL, TRUE);

  return CView::OnMouseWheel(nFlags, zDelta, pt);
}

void OCCView::OnMouseMove(UINT nFlags, CPoint point) {
  switch (m_mode) {
  case OCCView::Action3d::Action3d_Nothing: {
    // 判断是否选中
    Handle(AIS_InteractiveContext) ctx = GetDocument()->GetContext();
    ctx->MoveTo(point.x, point.y, m_hView, Standard_True);

    break;
  }
  case OCCView::Action3d::Action3d_Panning: {
    // 平移视角
    m_hView->Pan(point.x - m_pos.x, -point.y + m_pos.y);
    m_pos = point;
    InvalidateRect(NULL, TRUE);

    break;
  }
  case OCCView::Action3d::Action3d_Rotation: {
    // 旋转视角
    m_hView->Rotation(point.x, point.y);
    InvalidateRect(NULL, TRUE);

    break;
  }
  }

  CView::OnMouseMove(nFlags, point);
}

void OCCView::OnLButtonUp(UINT nFlags, CPoint point) {
  // 松开鼠标退出模式
  m_mode = Action3d::Action3d_Nothing;
  InvalidateRect(NULL, TRUE);

  CView::OnLButtonUp(nFlags, point);
}
```



#### OCCFrameWnd

创建框架类，作为文档显示的框架

```cpp
#pragma once

#include <afxext.h>
#include <afxwin.h>

#include <STEPControl_Reader.hxx>
#include <STEPControl_Writer.hxx>

#include "OCCView.h"
#include "resource.h"

// 自定义框架类
class OCCFrameWnd : public CFrameWnd {
  DECLARE_DYNCREATE(OCCFrameWnd)
  DECLARE_MESSAGE_MAP()
public:
  OCCDoc *GetDocument() const; // 获得 OCCDoc 文档指针

public:
  afx_msg int OnCreate(LPCREATESTRUCT lpCreateStruct);
  afx_msg void OnDestroy();
  afx_msg void OnOpen();
  afx_msg void OnSave();
};
```

实现动态类运行机制

```cpp
#include "OCCFrameWnd.h"

IMPLEMENT_DYNCREATE(OCCFrameWnd, CFrameWnd)

BEGIN_MESSAGE_MAP(OCCFrameWnd, CFrameWnd)
ON_WM_CREATE()
ON_WM_DESTROY()
ON_COMMAND(ID_OPEN, &OCCFrameWnd::OnOpen)
ON_COMMAND(ID_SAVE, &OCCFrameWnd::OnSave)
END_MESSAGE_MAP()

OCCDoc *OCCFrameWnd::GetDocument() const {
  CView *view = GetActiveView();
  ASSERT(view->IsKindOf(RUNTIME_CLASS(OCCView)));
  return (OCCDoc *)view->GetDocument();
}

int OCCFrameWnd::OnCreate(LPCREATESTRUCT lpCreateStruct) {
  if (CFrameWnd::OnCreate(lpCreateStruct) == -1)
    return -1;

  // 修改窗口信息
  SetWindowPos(NULL, 400, 100, 800, 800, SWP_SHOWWINDOW);

#ifndef NDEBUG
  // 创建支持显示中文的控制台
  AllocConsole();
  freopen("CONOUT$", "w", stdout);
#endif

  return 0;
}

void OCCFrameWnd::OnDestroy() {
  CFrameWnd::OnDestroy();

#ifndef NDEBUG
  // 释放控制台
  FreeConsole();
#endif
}

// 宽字符串转窄字符串
std::string ws2s(const std::wstring &ws) {
  // 创建临时内存并清空
  char *dest = new char[ws.size() + 1];
  memset(dest, 0, ws.size() + 1);

  // 转换函数
  size_t i;
  wcstombs_s(&i, dest, ws.size() + 1, ws.c_str(), ws.size());

  // 获得结果并释放内存
  std::string result = dest;
  delete[] dest;
  return result;
}

void OCCFrameWnd::OnOpen() {
  // 设置筛选器，允许打开 stp 文件
  TCHAR szFilter[] = _T("STEP Files (*.stp)|*.stp");
  CFileDialog cfdlg(TRUE, 0, 0, OFN_HIDEREADONLY | OFN_OVERWRITEPROMPT,
                    szFilter);

  // DoModal 返回 IDOK 或 IDCANCEL
  if (cfdlg.DoModal() == IDOK) {
    // 获取文件路径名和扩展名
    CString name = cfdlg.GetPathName();
    std::string str = ws2s(name.GetBuffer());

    // 创建读取器
    STEPControl_Reader reader;
    reader.ReadFile(Standard_CString(str.c_str()));

    // 检查读取信息
    reader.PrintCheckLoad(Standard_True, IFSelect_ItemsByEntity);

    // 获得可翻译的根实体数量，然后翻译所有实体，获得成功的数量
    Standard_Integer NbRoots = reader.NbRootsForTransfer();
    Standard_Integer NbTrans = reader.TransferRoots();

    // 检查翻译信息
    reader.PrintCheckTransfer(Standard_True, IFSelect_ItemsByEntity);

    // 将获得的形状添加到 Doc 中
    OCCDoc *doc = GetDocument();
    for (int i = 1; i <= reader.NbShapes(); i++)
      doc->AddShape(reader.Shape(i));

    // 刷新 doc 中的交互式上下文
    doc->UpdateInteractiveContext();
  }
}

void OCCFrameWnd::OnSave() {
  // 设置筛选器，允许保存为 .stp 格式文件
  TCHAR szFilter[] = TEXT("STEP Files (*.stp)|");
  CFileDialog cfdlg(FALSE, 0, 0, OFN_HIDEREADONLY | OFN_OVERWRITEPROMPT,
                    szFilter);

  if (cfdlg.DoModal() == IDOK) {
    // 获取路径，添加后缀
    CString name = cfdlg.GetPathName() + ".stp";
    std::string str = ws2s(name.GetBuffer());

    // 创建写入器
    STEPControl_Writer writer;

    // 翻译所有形状
    OCCDoc *doc = GetDocument();
    for (int i = 1; i <= doc->GetShapeNum(); i++)
      writer.Transfer(doc->GetShape(i), STEPControl_AsIs);

    // 写入文件
    writer.Write(Standard_CString(str.c_str()));

    MessageBox(TEXT("保存成功！"));
  }
}
```

注意在 open 和 save 过程中**进行宽字符串到窄字符串**的转换，否则在 Release 模式下会报错。



#### resource

在 VS 中创建菜单资源，新建“文件-打开”和“文件-保存”，将其 ID 修改为 `ID_OPEN, ID_SAVE` 得到的 ` resoure.h ` 文件如下

```cpp
//{{NO_DEPENDENCIES}}
// Microsoft Visual C++ 生成的包含文件。
// 供 MAIN.rc 使用
//
#define IDR_MENU1                       101
#define ID_OPEN                         40003
#define ID_SAVE                         40004

// Next default values for new objects
// 
#ifdef APSTUDIO_INVOKED
#ifndef APSTUDIO_READONLY_SYMBOLS
#define _APS_NEXT_RESOURCE_VALUE        102
#define _APS_NEXT_COMMAND_VALUE         40005
#define _APS_NEXT_CONTROL_VALUE         1001
#define _APS_NEXT_SYMED_VALUE           101
#endif
#endif

```

以及 `main.rc` 文件

```cpp
// Microsoft Visual C++ generated resource script.
//
#include "resource.h"

#define APSTUDIO_READONLY_SYMBOLS
/////////////////////////////////////////////////////////////////////////////
//
// Generated from the TEXTINCLUDE 2 resource.
//
#include "winres.h"

/////////////////////////////////////////////////////////////////////////////
#undef APSTUDIO_READONLY_SYMBOLS

/////////////////////////////////////////////////////////////////////////////
// 中文(简体，中国) resources

#if !defined(AFX_RESOURCE_DLL) || defined(AFX_TARG_CHS)
LANGUAGE LANG_CHINESE, SUBLANG_CHINESE_SIMPLIFIED

#ifdef APSTUDIO_INVOKED
/////////////////////////////////////////////////////////////////////////////
//
// TEXTINCLUDE
//

1 TEXTINCLUDE 
BEGIN
    "resource.h\0"
END

2 TEXTINCLUDE 
BEGIN
    "#include ""winres.h""\r\n"
    "\0"
END

3 TEXTINCLUDE 
BEGIN
    "\r\n"
    "\0"
END

#endif    // APSTUDIO_INVOKED


/////////////////////////////////////////////////////////////////////////////
//
// Menu
//

IDR_MENU1 MENU
BEGIN
    POPUP "文件"
    BEGIN
        MENUITEM "打开",                          ID_OPEN
        MENUITEM "保存",                          ID_SAVE
    END
END

#endif    // 中文(简体，中国) resources
/////////////////////////////////////////////////////////////////////////////



#ifndef APSTUDIO_INVOKED
/////////////////////////////////////////////////////////////////////////////
//
// Generated from the TEXTINCLUDE 3 resource.
//


/////////////////////////////////////////////////////////////////////////////
#endif    // not APSTUDIO_INVOKED


```



#### main

创建运行程序，加载前面这些类

```cpp
#include "OCCFrame/OCCFrameWnd.h"

// 自定义应用程序类
class OCCWinApp : public CWinApp {
public:
  virtual BOOL InitInstance() {
    // 创建模板
    CSingleDocTemplate *pTemplate = new CSingleDocTemplate(
        IDR_MENU1, RUNTIME_CLASS(OCCDoc), RUNTIME_CLASS(OCCFrameWnd),
        RUNTIME_CLASS(OCCView));

    // 添加模板
    AddDocTemplate(pTemplate);

    // 创建新的窗口
    OnFileNew();
    m_pMainWnd->ShowWindow(SW_SHOW);
    m_pMainWnd->SetWindowText(TEXT("OCC Demo"));
    m_pMainWnd->UpdateWindow();
    return TRUE;
  }
};

// 应用程序对象的全局变量
OCCWinApp theApp;
```



### Qt 文档结构

#### 项目文件结构

├─data
├─include
│  └─occmesh
│      ├─frame
└─src
    ├─frame

在主目录下创建

```cmake
cmake_minimum_required(VERSION 3.27)

project(
  main
  VERSION 1.0.0
  LANGUAGES CXX)

# 添加测试数据路径
add_definitions(-DTEST_DATA_PATH="${CMAKE_SOURCE_DIR}/data")

add_subdirectory(src)
```

在 src 目录下创建

```cmake
cmake_minimum_required(VERSION 3.27)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(PYBIND11_FINDPYTHON ON)

project(
  occmesh
  VERSION 1.0.0
  LANGUAGES CXX)

add_compile_options(/bigobj)

# 配置 Qt6
set(CMAKE_PREFIX_PATH "D:/Qt/Qt6.6/6.6.0/msvc2019_64/lib/cmake")
find_package(Qt6 REQUIRED COMPONENTS Widgets)
qt_standard_project_setup()

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

# 为 UI 提供头文件路径
include_directories(${CMAKE_SOURCE_DIR}/include/${PROJECT_NAME}/frame)

# 构建目录
set(BINARY_DIR ${PROJECT_BINARY_DIR})

# 包含子目录
set(SOURCE)
set(SOURCE_DIR "frame")

foreach(dir ${SOURCE_DIR})
  file(GLOB SRC CONFIGURE_DEPENDS ${dir}/*.cpp)
  file(GLOB INC CONFIGURE_DEPENDS ${CMAKE_SOURCE_DIR}/include/${PROJECT_NAME}/${dir}/*.h)
  file(GLOB RC CONFIGURE_DEPENDS ${dir}/*.qrc ${dir}/*.ui)

  source_group(${dir} FILES ${SRC} ${INC} ${RC})
  set(SOURCE ${SOURCE} ${SRC} ${INC} ${RC})
endforeach()

qt_add_executable(${PROJECT_NAME} main.cpp ${SOURCE})

# 通过 find 获得 OCC
set(OPENCASCADE_DIR "D:/lib/opencascade-7.7.0/cmake")
find_package(OPENCASCADE REQUIRED)

# Eigen 库
set(EIGEN3_DIR "D:/lib/Eigen3/share/eigen3/cmake")
find_package(EIGEN3 REQUIRED)

# 链接头文件
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/include)
target_include_directories(${PROJECT_NAME} PRIVATE ${OpenCASCADE_INCLUDE_DIR})
target_include_directories(${PROJECT_NAME} PRIVATE ${EIGEN3_INCLUDE_DIR})

# 链接库文件
target_link_libraries(${PROJECT_NAME} PRIVATE Qt6::Widgets)
target_link_libraries(${PROJECT_NAME} PRIVATE ${OpenCASCADE_LIBRARY_DIR}d/*.lib)

# 设为 Win32 应用
set_target_properties(${PROJECT_NAME} PROPERTIES WIN32_EXECUTABLE ON)

# 指定非 Debug 模式文件输出路径
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

# 将依赖的动态库复制到输出目录 DEPS_DLL
file(GLOB DEPS_DLL CONFIGURE_DEPENDS ${OpenCASCADE_BINARY_DIR}d/*.dll)

add_custom_command(
  TARGET ${PROJECT_NAME}
  POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy ${DEPS_DLL} ${OUTPUT_DIR})

```



#### MainWindow

这里直接使用项目的主窗口进行加载。首先是头文件

```cpp
#pragma once

#include <QFileDialog>
#include <QInputDialog>
#include <QMessageBox>
#include <QMouseEvent>
#include <QResizeEvent>
#include <QTimer>
#include <QtWidgets/QMainWindow>

// 交互头文件
#include <AIS_PointCloud.hxx>
#include <AIS_Shape.hxx>
#include <AIS_Triangulation.hxx>

// 形状头文件
#include <TopoDS_Shape.hxx>

// 序列头文件
#include <TColGeom_HArray1OfCurve.hxx>
#include <TColGeom_HArray1OfSurface.hxx>
#include <TColStd_HSequenceOfTransient.hxx>
#include <TColgp_HArray1OfPnt.hxx>
#include <TColgp_HSequenceOfPnt.hxx>
#include <TopTools_HSequenceOfShape.hxx>

namespace Ui
{
class MainWindow;
}

class MainWindow : public QMainWindow
{
    Q_OBJECT

  public:
    friend class OCCWidget;

    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

    void AddPointCloud(const std::vector<gp_Pnt> &points); // 添加点云
    void AddCurve(Handle(Geom_Curve) curve, double first,
                  double last);                                      // 添加曲线
    void AddSurface(Handle(Geom_Surface) surface);                   // 添加曲面
    void AddShape(const TopoDS_Shape &shape);                        // 用于载入形状
    void AddTriangulation(Handle(Poly_Triangulation) triangulation); // 用于载入三角化

    void ReadSTP(const char *path);
    void WriteSTP(const char *path);
    void ReadIGS(const char *path);
    void WriteIGS(const char *path);
    void ReadBRep(const char *path);
    void WriteBRep(const char *path);
    void ReadCloud(const char *path);
    void WriteCloud(const char *path);

    virtual void resizeEvent(QResizeEvent *event);

  public slots:
    void initializeChildren();

    void openFile();
    void openCloud();
    void openDirectory();
    void saveFile();
    void saveCloud();
    void saveSpecific();
    void deleteFile();

    void coverFPIA();

  private:
    Ui::MainWindow *ui;

    std::vector<TopoDS_Shape> m_selected;                     // 存放被选中的拓扑
    std::vector<TopoDS_Shape> m_shapes;                       // 存放拓扑形状
    std::vector<Handle(AIS_PointCloud)> m_clouds;             // 存放点云
    std::vector<Handle(Poly_Triangulation)> m_triangulations; // 存放三角化
};
```

以及对应的实现

```cpp
#include <occmesh/frame/MainWindow.h>
#include <occmesh/frame/OCCWidget.h>

#include "ui_MainWindow.h"

MainWindow::MainWindow(QWidget *parent) : QMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    this->setWindowTitle("MainWindow");

    // 逐层级启用鼠标追踪
    this->setMouseTracking(true);
    ui->centralWidget->setMouseTracking(true);

    // 初始化连接
    QObject::connect(ui->actionObject, SIGNAL(triggered(bool)), this, SLOT(openFile()));
    QObject::connect(ui->actionCloud, SIGNAL(triggered(bool)), this, SLOT(openCloud()));
    QObject::connect(ui->actionDir, SIGNAL(triggered(bool)), this, SLOT(openDirectory()));
    QObject::connect(ui->actionSaveObj, SIGNAL(triggered(bool)), this, SLOT(saveFile()));
    QObject::connect(ui->actionSaveCloud, SIGNAL(triggered(bool)), this, SLOT(saveCloud()));
    QObject::connect(ui->actionSpecific, SIGNAL(triggered(bool)), this, SLOT(saveSpecific()));
    QObject::connect(ui->actionDelete, SIGNAL(triggered(bool)), this, SLOT(deleteFile()));

    QObject::connect(ui->actionFPIA, SIGNAL(triggered(bool)), this, SLOT(coverFPIA()));

    // 构造完成后调用
    QTimer::singleShot(0, this, &MainWindow::initializeChildren);
}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::AddPointCloud(const std::vector<gp_Pnt> &points)
{
    Handle(TColgp_HArray1OfPnt) cloud = new TColgp_HArray1OfPnt(1, points.size());
    for (int i = 1; i <= cloud->Size(); i++)
        cloud->SetValue(i, points[i - 1]);
    Handle(AIS_PointCloud) c(new AIS_PointCloud);
    c->SetPoints(cloud);
    m_clouds.push_back(c);
}

void MainWindow::AddCurve(Handle(Geom_Curve) curve, double first, double last)
{
    TopoDS_Shape edge = BRepBuilderAPI_MakeEdge(curve, first, last);
    this->AddShape(edge);
}

void MainWindow::AddSurface(Handle(Geom_Surface) surface)
{
    TopoDS_Shape face = BRepBuilderAPI_MakeFace(surface, 1e-8);
    this->AddShape(face);
}

void MainWindow::AddShape(const TopoDS_Shape &shape)
{
    // 推入形状
    m_shapes.push_back(shape);
}

void MainWindow::AddTriangulation(Handle(Poly_Triangulation) triangulation)
{
    m_triangulations.push_back(triangulation);
}

void MainWindow::ReadSTP(const char *path)
{
    // 创建读取器
    STEPControl_Reader reader;
    reader.ReadFile(Standard_CString(path));

    // 检查读取信息
    reader.PrintCheckLoad(Standard_True, IFSelect_ItemsByEntity);

    // 获得可翻译的根实体数量，然后翻译所有实体，获得成功的数量
    Standard_Integer NbRoots = reader.NbRootsForTransfer();
    Standard_Integer NbTrans = reader.TransferRoots();

    // 检查翻译信息
    reader.PrintCheckTransfer(Standard_True, IFSelect_ItemsByEntity);

    // 添加获得的形状
    for (int i = 1; i <= reader.NbShapes(); i++)
    {
        auto shape = reader.Shape(i);
        this->AddShape(shape);
    }

    std::cout << "Read " << reader.NbShapes() << " shapes." << std::endl;
}

void MainWindow::WriteSTP(const char *path)
{
    // 创建写入器
    STEPControl_Writer writer;

    // 翻译所有形状
    for (auto sh : m_shapes)
        writer.Transfer(sh, STEPControl_AsIs);

    // 写入文件
    writer.Write(Standard_CString(path));
}

void MainWindow::ReadIGS(const char *path)
{
    // 读取器
    IGESControl_Reader reader;

    // 读取IGES文件
    Standard_Integer status = reader.ReadFile(path);

    Standard_Integer NbRoots = reader.NbRootsForTransfer();
    Standard_Integer NbTrans = reader.TransferRoots();

    // 添加获得的形状
    for (int i = 1; i <= reader.NbShapes(); i++)
    {
        auto shape = reader.Shape(i);
        this->AddShape(shape);
    }
}

void MainWindow::WriteIGS(const char *path)
{
    // 创建写入器
    IGESControl_Writer writer;

    // 翻译所有形状
    for (auto sh : m_shapes)
        writer.AddShape(sh);

    // 写入文件
    writer.Write(Standard_CString(path));
}

void MainWindow::ReadBRep(const char *path)
{
    // 创建空形状保存
    TopoDS_Shape shape;

    // 构建器
    BRep_Builder builder;
    if (BRepTools::Read(shape, path, builder))
        this->AddShape(shape);
    else
        QMessageBox::information(this, "Tip", "Failed.");
}

void MainWindow::WriteBRep(const char *path)
{
}

void MainWindow::ReadCloud(const char *path)
{
    // 创建空形状保存
    TopoDS_Shape shape;

    // 构建器
    BRep_Builder builder;
    if (BRepTools::Read(shape, path, builder))
    {
        if (shape.ShapeType() == TopAbs_COMPOUND)
        {
            TopExp_Explorer Ex;
            BRep_Tool tool;

            std::vector<gp_Pnt> points;
            for (Ex.Init(shape, TopAbs_VERTEX); Ex.More(); Ex.Next())
            {
                TopoDS_Vertex V;
                V = TopoDS::Vertex(Ex.Current());

                gp_Pnt P = tool.Pnt(V);
                points.push_back(P);
            }

            if (!points.empty())
                this->AddPointCloud(points);
        }
        else
            this->AddShape(shape);
    }
    else
        QMessageBox::information(this, "Tip", "Failed.");
}

void MainWindow::WriteCloud(const char *path)
{
    // 创建复合体
    TopoDS_Compound compound;
    BRep_Builder builder;
    builder.MakeCompound(compound);

    auto cloud = ui->occWidget->SelectedCloud();
    if (!cloud.IsNull())
    {
        auto points = cloud->GetPoints();

        // 将每个点转换为一个顶点并添加到复合体中
        for (int i = 1; i <= points->VertexNumber(); ++i)
        {
            Standard_Real x, y, z;
            points->Vertice(i, x, y, z);
            TopoDS_Vertex vertex = BRepBuilderAPI_MakeVertex({x, y, z});
            builder.Add(compound, vertex);
        }

        // 将复合体导出为 BREP 文件
        BRepTools::Write(compound, path);
    }
}

void MainWindow::resizeEvent(QResizeEvent *event)
{
    auto s = event->size();
    ui->occWidget->resize(s.width(), s.height());
}

void MainWindow::initializeChildren()
{
    ui->occWidget->InitializeWindow();
}

void MainWindow::openFile()
{
    static QString directoryPath = TEST_DATA_PATH;
    QString filename = QFileDialog::getOpenFileName(
        this, "Choose Model", directoryPath, "All Files(*.*);;STEP (*.stp);;BRep (*.brep);;IGS (*.igs);;OBJ (*.obj)");
    if (filename != "")
    {
        QFileInfo fileInfo(filename);
        directoryPath = fileInfo.absolutePath();

        if (filename.endsWith(".stp") || filename.endsWith(".step"))
            ReadSTP(filename.toStdString().c_str());
        else if (filename.endsWith(".brep"))
            ReadBRep(filename.toStdString().c_str());
        else if (filename.endsWith(".igs"))
            ReadIGS(filename.toStdString().c_str());

        // 刷新交互
        ui->occWidget->UpdateInteractiveContext();
    }

    // 存在物体则适应物体
    ui->occWidget->FitAllShapes();
}

void MainWindow::openCloud()
{
    static QString directoryPath = TEST_DATA_PATH;
    QString filename = QFileDialog::getOpenFileName(this, "Choose Model", directoryPath, "BRep (*.brep)");
    if (filename != "")
    {
        QFileInfo fileInfo(filename);
        directoryPath = fileInfo.absolutePath();

        if (filename.endsWith(".brep"))
            ReadCloud(filename.toStdString().c_str());

        // 刷新交互
        ui->occWidget->UpdateInteractiveContext();
    }

    // 存在物体则适应物体
    ui->occWidget->FitAllShapes();
}

void MainWindow::openDirectory()
{
    static QString directoryPath = TEST_DATA_PATH;
    QString dirname = QFileDialog::getExistingDirectory(this, "Choose Directory", directoryPath,
                                                        QFileDialog::ShowDirsOnly | QFileDialog::DontResolveSymlinks);
    if (dirname != "")
    {
        QDir dir(dirname);

        // 获取目录下的所有文件名（不包括子目录中的文件）
        QStringList fileList = dir.entryList(QDir::Files);
        for (const QString &filename : fileList)
        {
            if (filename.endsWith(".brep"))
                ReadBRep((dirname + "/" + filename).toStdString().c_str());
            else if (filename.endsWith(".stp") || filename.endsWith(".step"))
                ReadSTP((dirname + "/" + filename).toStdString().c_str());
            else if (filename.endsWith(".igs"))
                ReadIGS((dirname + "/" + filename).toStdString().c_str());
        }

        // 刷新交互
        ui->occWidget->UpdateInteractiveContext();
    }

    // 存在物体则适应物体
    ui->occWidget->FitAllShapes();
}

void MainWindow::saveFile()
{
    static QString directoryPath = TEST_DATA_PATH;
    QString filename = QFileDialog::getSaveFileName(this, "Save File", directoryPath,
                                                    "STEP (*.stp);;BRep (*.brep);;IGS (*.igs);;OBJ (*.obj)");
    if (filename != "")
    {
        QFileInfo fileInfo(filename);
        directoryPath = fileInfo.absolutePath();

        // 获取路径，添加后缀
        std::string name = filename.toStdString();

        if (filename.endsWith(".stp") || filename.endsWith(".step"))
            WriteSTP(filename.toStdString().c_str());
        else if (filename.endsWith(".brep"))
            WriteBRep(filename.toStdString().c_str());
        else if (filename.endsWith(".igs"))
            WriteIGS(filename.toStdString().c_str());

        QMessageBox::information(this, "Tip", "Saved.");
    }
}

void MainWindow::saveCloud()
{
    static QString directoryPath = TEST_DATA_PATH;
    QString filename = QFileDialog::getSaveFileName(this, "Save File", directoryPath, "BRep (*.brep)");
    if (filename != "")
    {
        QFileInfo fileInfo(filename);
        directoryPath = fileInfo.absolutePath();

        // 获取路径，添加后缀
        std::string name = filename.toStdString();
        if (filename.endsWith(".brep"))
            WriteCloud(filename.toStdString().c_str());

        QMessageBox::information(this, "Tip", "Saved.");
    }
}

void MainWindow::saveSpecific()
{
    static QString directoryPath = TEST_DATA_PATH;
    QString filename = QFileDialog::getSaveFileName(this, "Save File", directoryPath, "STEP (*.stp)");
    if (filename != "")
    {
        QFileInfo fileInfo(filename);
        directoryPath = fileInfo.absolutePath();

        // 获取路径，添加后缀
        std::string name = filename.toStdString();

        if (filename.endsWith(".stp") || filename.endsWith(".step"))
        {
            // 创建写入器
            STEPControl_Writer writer;

            // 翻译所有形状
            auto shapes = ui->occWidget->SelectedShapes();
            for (auto sh : shapes)
                writer.Transfer(sh, STEPControl_AsIs);

            // 写入文件
            writer.Write(Standard_CString(filename.toStdString().c_str()));
        }

        QMessageBox::information(this, "Tip", "Saved.");
    }
}

void MainWindow::deleteFile()
{
    m_shapes.clear();
    m_selected.clear();
    m_clouds.clear();
    m_triangulations.clear();

    // 刷新窗口
    ui->occWidget->UpdateInteractiveContext();
}

void MainWindow::coverFPIA()
{
}
```



#### OCCWidget

加载 OCC 的渲染框架

```cpp
#pragma once

#include <occmesh/frame/MainWindow.h>

#include <QWidget>

// 渲染头文件
#include <Prs3d_LineAspect.hxx>
#include <Prs3d_ShadingAspect.hxx>

// 图形交互
#include <AIS_InteractiveContext.hxx>
#include <OpenGL_GraphicDriver.hxx>
#include <Prs3d_PointAspect.hxx>
#include <StdSelect_BRepOwner.hxx>
#include <V3d_Viewer.hxx>

// 视口和窗口
#include <V3d_View.hxx>
#include <WNT_Window.hxx>

// 光照头文件
#include <V3d_AmbientLight.hxx>
#include <V3d_DirectionalLight.hxx>

// 交互头文件
#include <AIS_PointCloud.hxx>
#include <AIS_Shape.hxx>
#include <AIS_Triangulation.hxx>

// 形状头文件
#include <TopExp_Explorer.hxx>
#include <TopoDS.hxx>
#include <TopoDS_Compound.hxx>
#include <TopoDS_Edge.hxx>
#include <TopoDS_Face.hxx>
#include <TopoDS_Shape.hxx>
#include <TopoDS_Wire.hxx>

// 序列头文件
#include <TColGeom_HArray1OfCurve.hxx>
#include <TColGeom_HArray1OfSurface.hxx>
#include <TColStd_HSequenceOfTransient.hxx>
#include <TColgp_HArray1OfPnt.hxx>
#include <TColgp_HSequenceOfPnt.hxx>
#include <TopTools_HSequenceOfShape.hxx>

// 构造器头文件
#include <BRepBuilderAPI_MakeEdge.hxx>
#include <BRepBuilderAPI_MakeFace.hxx>
#include <BRepBuilderAPI_MakeVertex.hxx>
#include <BRepBuilderAPI_MakeWire.hxx>

// 几何头文件
#include <Geom_BSplineCurve.hxx>
#include <Geom_BSplineSurface.hxx>
#include <Geom_Plane.hxx>
#include <gp_Circ.hxx>
#include <gp_Pnt.hxx>

// 用来获得几何信息
#include <BRep_Tool.hxx>

// 网格头文件
#include <MeshVS_MeshPrsBuilder.hxx>
#include <MeshVS_NodalColorPrsBuilder.hxx>

// 纹理信息
#include <Graphic3d_Texture2D.hxx>
#include <Graphic3d_TypeOfTexture.hxx>
#include <Image_PixMap.hxx>

// 读写头文件
#include <BRepTools.hxx>
#include <BRep_Builder.hxx>
#include <IGESControl_Reader.hxx>
#include <IGESControl_Writer.hxx>
#include <STEPControl_Reader.hxx>
#include <STEPControl_Writer.hxx>

// 坐标指示器
#include <AIS_Trihedron.hxx>
#include <Aspect_DisplayConnection.hxx>
#include <Font_FontMgr.hxx>
#include <Geom_Axis2Placement.hxx>
#include <Graphic3d_ArrayOfSegments.hxx>
#include <OSD_Environment.hxx>
#include <Prs3d_ArrowAspect.hxx>
#include <Prs3d_LineAspect.hxx>

namespace Ui
{
class OCCWidget;
}

class OCCWidget : public QWidget
{
    Q_OBJECT

  public:
    enum class ShadeMode
    {
        Shade,
        Wire
    };

    enum class Action3d
    {
        Nothing,
        Panning, // 平移
        Zooming, // 缩放
        Rotation // 旋转
    };

    explicit OCCWidget(QWidget *parent);
    ~OCCWidget();

    void InitializeWindow();                                                   // 初始化窗口
    void UpdateInteractiveContext();                                           // 刷新上下文数据
    void UpdateAxis();                                                         // 更新坐标轴
    void AdjustSelectionStyle(const Handle(AIS_InteractiveContext) & context); // 修改选择风格
    void FitAllShapes();                                                       // 适应窗口
    void SetShadeMode(ShadeMode mode);                                         // 设置着色模式
    void DrawTriangulation(Handle(Poly_Triangulation) triangle);               // 绘制曲面

    std::vector<TopoDS_Shape> SelectedShapes(); // 获得被选中的形状
    Handle(AIS_PointCloud) SelectedCloud();     // 获得被选中的点云

    virtual void resizeEvent(QResizeEvent *event);
    virtual void paintEvent(QPaintEvent *);
    virtual void mousePressEvent(QMouseEvent *);   // 按下
    virtual void mouseReleaseEvent(QMouseEvent *); // 抬起
    virtual void mouseMoveEvent(QMouseEvent *);    // 移动
    virtual void wheelEvent(QWheelEvent *);        // 滚轮
    virtual QPaintEngine *paintEngine() const;

  private:
    Ui::OCCWidget *ui;

    Action3d m_mode;       // 记录操作模式
    ShadeMode m_shadeMode; // 记录网格渲染模式
    QPoint m_pos;          // 记录鼠标平移坐标

    Handle(V3d_View) m_hView;                     // 视口句柄
    Handle(V3d_Viewer) m_hViewer;                 // 定义 V3d_Viewer 句柄
    Handle(AIS_InteractiveContext) m_hAisContext; // 定义交互上下文
    Handle(AIS_Trihedron) m_trihedron;            // 坐标轴指示器

    MainWindow *m_parent; // 保存父窗口
};
```

以及对应实现

```cpp
#include <occmesh/frame/OCCWidget.h>

#include "ui_MainWindow.h"
#include "ui_OCCWidget.h"

OCCWidget::OCCWidget(QWidget *parent)
    : QWidget(parent), ui(new Ui::OCCWidget), m_mode(Action3d::Nothing), m_shadeMode(ShadeMode::Shade), m_pos({0, 0}),
      m_parent(static_cast<MainWindow *>(parent->parentWidget()))
{
    // 注意 m_parent 应该是父窗口的父窗口，因为上级窗口为 centralWidget
    ui->setupUi(this);

    // 逐层级启用鼠标追踪
    this->setMouseTracking(true);
    ui->frame->setMouseTracking(true);
    this->setUpdatesEnabled(false);
    ui->frame->setUpdatesEnabled(false);
}

OCCWidget::~OCCWidget()
{
    delete ui;
}

void OCCWidget::InitializeWindow()
{
    // 创建 OpenGL 引擎
    Handle(Aspect_DisplayConnection) theDisp = new Aspect_DisplayConnection();
    Handle(OpenGl_GraphicDriver) H_OpenGL_GraphicDriver = new OpenGl_GraphicDriver(theDisp);

    // 创建视口
    m_hViewer = new V3d_Viewer(H_OpenGL_GraphicDriver);

    // 光照
    Handle(V3d_DirectionalLight) LightDir = new V3d_DirectionalLight(V3d_Zneg, Quantity_Color(Quantity_NOC_GRAY97), 1);
    Handle(V3d_AmbientLight) LightAmb = new V3d_AmbientLight();

    // 设置光照方向
    LightDir->SetDirection(1.0, -2.0, -10.0);

    // 添加光照
    m_hViewer->AddLight(LightDir);
    m_hViewer->AddLight(LightAmb);
    m_hViewer->SetLightOn(LightDir);
    m_hViewer->SetLightOn(LightAmb);

    // 创建交互上下文
    m_hAisContext = new AIS_InteractiveContext(m_hViewer);

    // 配置全局设置，渲染边界
    const Handle(Prs3d_Drawer) &contextDrawer = m_hAisContext->DefaultDrawer();
    if (!contextDrawer.IsNull())
    {
        const Handle(Prs3d_ShadingAspect) &SA = contextDrawer->ShadingAspect();
        const Handle(Graphic3d_AspectFillArea3d) &FA = SA->Aspect();
        contextDrawer->SetFaceBoundaryDraw(true);
        // contextDrawer->SetDeviationCoefficient(1e-5);  // 更高的精度

        FA->SetEdgeOff();

        // 设置最大绘制线路
        contextDrawer->SetMaximalParameterValue(1000);
    }

    // 更新视口
    UpdateInteractiveContext();

    // 通过 V3d_Viewer 创建视口
    m_hView = m_hViewer->CreateView();
    m_hView->ZBufferTriedronSetup(Quantity_NOC_RED, Quantity_NOC_GREEN, Quantity_NOC_BLUE, 0.8, 0.05, 12);
    m_hView->TriedronDisplay(Aspect_TOTP_LEFT_LOWER, Quantity_NOC_WHITE, 0.2, V3d_ZBUFFER);

    // 通过当前窗口句柄创建 WNT_Window，并将视口连接到 WNT_Window
    ui->frame->setAttribute(Qt::WA_PaintOnScreen);
    Handle(WNT_Window) H_WNT_Window = new WNT_Window((Aspect_Handle)ui->frame->winId());
    m_hView->SetWindow(H_WNT_Window);

    // 窗口映射
    if (!H_WNT_Window->IsMapped())
        H_WNT_Window->Map();

    // 设置视口
    m_hView->MustBeResized();
    m_hView->SetShadingModel(V3d_PHONG);

    // 设置白色背景
    Quantity_Color bgColor(1.0, 1.0, 1.0, Quantity_TOC_RGB);
    m_hView->SetBackgroundColor(bgColor);

    // 配置渲染参数，例如反走样、阴影等
    Graphic3d_RenderingParams &RenderParams = m_hView->ChangeRenderingParams();
    RenderParams.IsAntialiasingEnabled = true;
    RenderParams.NbMsaaSamples = 8;
    RenderParams.IsShadowEnabled = false;
    RenderParams.CollectedStats = Graphic3d_RenderingParams::PerfCounters_NONE;
}

void OCCWidget::UpdateInteractiveContext()
{
    // 移除所有物体
    m_hAisContext->RemoveAll(false);

    // 遍历每个形状绘制，并激活选择风格
    for (auto sh : m_parent->m_shapes)
    {
        Handle(AIS_Shape) shape = new AIS_Shape(sh);
        shape->SetColor(Quantity_NOC_GREEN);

        // 使用 false 会快很多
        m_hAisContext->Display(shape, false);
        m_hAisContext->SetDisplayMode(shape, AIS_Shaded, false);
    }

    for (auto sh : m_parent->m_selected)
    {
        TopExp_Explorer Ex;
        for (Ex.Init(sh, TopAbs_EDGE); Ex.More(); Ex.Next())
        {
            Handle(AIS_Shape) shape = new AIS_Shape(TopoDS::Edge(Ex.Current()));
            shape->SetColor(Quantity_NOC_RED);

            // 使用 false 会快很多
            m_hAisContext->Display(shape, false);
            m_hAisContext->SetDisplayMode(shape, AIS_Shaded, false);
        }

        for (Ex.Init(sh, TopAbs_FACE); Ex.More(); Ex.Next())
        {
            Handle(AIS_Shape) shape = new AIS_Shape(TopoDS::Face(Ex.Current()));
            shape->SetColor(Quantity_NOC_GREEN);

            Handle(Prs3d_LineAspect) aBoundaryAspect =
                new Prs3d_LineAspect(Quantity_NOC_BLUE, Aspect_TOL_SOLID, 3.0); // 蓝色边界，线宽 1.0
            shape->Attributes()->SetFaceBoundaryAspect(aBoundaryAspect);

            // 使用 false 会快很多
            m_hAisContext->Display(shape, false);
            m_hAisContext->SetDisplayMode(shape, AIS_Shaded, false);
        }
    }

    // 绘制点云
    for (auto cloud : m_parent->m_clouds)
    {
        cloud->SetColor(Quantity_NOC_BLUE);
        cloud->Attributes()->ShadingAspect()->Aspect()->SetMarkerScale(5);
        m_hAisContext->Display(cloud, false);
    }

    // 绘制坐标轴
    m_hAisContext->Display(m_trihedron, false);

    // 修改选择风格
    AdjustSelectionStyle(m_hAisContext);

    // 激活选择
    m_hAisContext->Activate(8, true); // triangle
    m_hAisContext->Activate(4, true); // faces
    m_hAisContext->Activate(2, true); // edges
    // m_hAisContext->Activate(1, true);  // vertex
}

void OCCWidget::AdjustSelectionStyle(const Handle(AIS_InteractiveContext) & context)
{
    // 设置渲染信息以及选中物体的高亮信息
    Handle(Prs3d_Drawer) selDrawer = new Prs3d_Drawer;

    selDrawer->SetLink(context->DefaultDrawer());
    selDrawer->SetFaceBoundaryDraw(true);
    selDrawer->SetDisplayMode(1);
    selDrawer->SetTransparency(1.0f);
    selDrawer->SetZLayer(Graphic3d_ZLayerId_Topmost);
    selDrawer->SetColor(Quantity_NOC_BLUE);
    selDrawer->SetBasicFillAreaAspect(new Graphic3d_AspectFillArea3d());

    // 调整填充区域颜色材质等信息
    const Handle(Graphic3d_AspectFillArea3d) &fillArea = selDrawer->BasicFillAreaAspect();

    fillArea->SetInteriorColor(Quantity_NOC_GREEN);
    fillArea->SetBackInteriorColor(Quantity_NOC_GREEN);

    fillArea->ChangeFrontMaterial().SetMaterialName(Graphic3d_NOM_NEON_GNC);
    fillArea->ChangeFrontMaterial().SetTransparency(1.0f);
    fillArea->ChangeBackMaterial().SetMaterialName(Graphic3d_NOM_NEON_GNC);
    fillArea->ChangeBackMaterial().SetTransparency(1.0f);

    selDrawer->UnFreeBoundaryAspect()->SetWidth(1.0);

    // 更新 AIS context
    context->SetHighlightStyle(Prs3d_TypeOfHighlight_LocalSelected, selDrawer);
}

void OCCWidget::FitAllShapes()
{
    // 存在物体则适应物体
    auto window = static_cast<MainWindow *>(parentWidget());
    if (!m_parent->m_shapes.empty() || !m_parent->m_clouds.empty() || !m_parent->m_triangulations.empty())
        m_hView->FitAll();
}

void OCCWidget::SetShadeMode(ShadeMode mode)
{
    m_shadeMode = mode;
}

void OCCWidget::DrawTriangulation(Handle(Poly_Triangulation) triangle)
{
    // 一般三角化
    if (m_shadeMode == ShadeMode::Shade)
    {
        Handle(AIS_InteractiveObject) triObject = new AIS_Triangulation(triangle);
        triObject->SetColor(Quantity_NOC_GREEN);
        m_hAisContext->SetDisplayMode(triObject, AIS_WireFrame, true);
        m_hAisContext->Display(triObject, Standard_True);
        triObject->Attributes()->ShadingAspect()->Aspect()->SetInteriorStyle(Aspect_IS_SOLID);
        // 网格三角化
    }
}

std::vector<TopoDS_Shape> OCCWidget::SelectedShapes()
{
    std::vector<TopoDS_Shape> shapeList;
    for (m_hAisContext->InitSelected(); m_hAisContext->MoreSelected(); m_hAisContext->NextSelected())
    {
        Handle(SelectMgr_EntityOwner) anOwner = m_hAisContext->SelectedOwner();
        Handle(AIS_InteractiveObject) anObj = Handle(AIS_InteractiveObject)::DownCast(anOwner->Selectable());

        // 将 SelectMgr_EntityOwner 对象下降为 StdSelect_BRepOwner 对象
        if (Handle(StdSelect_BRepOwner) aBRepOwner = Handle(StdSelect_BRepOwner)::DownCast(anOwner))
        {
            TopoDS_Shape aShape = aBRepOwner->Shape();
            shapeList.push_back(aShape);
        }
    }
    return shapeList;
}

Handle(AIS_PointCloud) OCCWidget::SelectedCloud()
{
    for (m_hAisContext->InitSelected(); m_hAisContext->MoreSelected(); m_hAisContext->NextSelected())
    {
        Handle(SelectMgr_EntityOwner) anOwner = m_hAisContext->SelectedOwner();
        Handle(AIS_PointCloud) pointCloud = Handle(AIS_PointCloud)::DownCast(anOwner->Selectable());
        if (!pointCloud.IsNull())
            return pointCloud;
    }
    return Handle(AIS_PointCloud){};
}

void OCCWidget::resizeEvent(QResizeEvent *)
{
    ui->frame->resize(size());
    // qDebug() << size();

    // 重绘视口
    if (m_hView.IsNull() == FALSE)
        m_hView->MustBeResized();
}

void OCCWidget::paintEvent(QPaintEvent *event)
{
    // 重绘视口
    if (m_hView.IsNull() == FALSE)
    {
        m_hView->MustBeResized();
        m_hView->Redraw();
    }
}

void OCCWidget::mousePressEvent(QMouseEvent *event)
{
    QPoint pos = event->pos();

    if (event->button() == Qt::LeftButton)
    {
        Handle(AIS_InteractiveContext) ctx = m_hAisContext;
        ctx->Select(Standard_True);
    }
    else if (event->button() == Qt::MiddleButton)
    {
        // 如果未选中，则进入旋转模式
        m_mode = Action3d::Rotation;
        m_hView->StartRotation(pos.x(), pos.y());
    }
    else if (event->button() == Qt::RightButton)
    {
        // 若选中，则进入平移模式
        m_mode = Action3d::Panning;
        m_pos = pos;
    }
}

void OCCWidget::mouseReleaseEvent(QMouseEvent *event)
{
    // 松开鼠标退出模式
    m_mode = Action3d::Nothing;
}

void OCCWidget::mouseMoveEvent(QMouseEvent *event)
{
    QPoint pos = event->pos();

    switch (m_mode)
    {
    case Action3d::Nothing: {
        // 判断是否选中
        Handle(AIS_InteractiveContext) ctx = m_hAisContext;
        ctx->MoveTo(pos.x(), pos.y(), m_hView, Standard_True);
        break;
    }
    case Action3d::Panning: {
        // 平移视角
        m_hView->Pan(pos.x() - m_pos.x(), -pos.y() + m_pos.y());
        m_pos = pos;
        break;
    }
    case Action3d::Rotation: {
        // 旋转视角
        m_hView->Rotation(pos.x(), pos.y());
        break;
    }
    }

    // 鼠标移动就刷新
    m_hView->Redraw();
}

void OCCWidget::wheelEvent(QWheelEvent *event)
{
    QPoint delta = event->angleDelta();

    if (m_mode == Action3d::Nothing)
    {
        // 缩放视口
        if (delta.y() > 0)
        {
            double scar = 1.2;
            if (m_hView->Scale() <= 10000)
                m_hView->SetZoom(scar);
        }
        else
        {
            double scar = 0.8;
            if (m_hView->Scale() > 0.0001)
                m_hView->SetZoom(scar);
        }
    }
}

QPaintEngine *OCCWidget::paintEngine() const
{
    return nullptr;
}
```



#### UI

我们给出

```xml
<<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>MainWindow</class>
 <widget class="QMainWindow" name="MainWindow">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>1200</width>
    <height>842</height>
   </rect>
  </property>
  <property name="minimumSize">
   <size>
    <width>800</width>
    <height>800</height>
   </size>
  </property>
  <property name="maximumSize">
   <size>
    <width>16777215</width>
    <height>16777215</height>
   </size>
  </property>
  <property name="windowTitle">
   <string>OCCMesh</string>
  </property>
  <property name="styleSheet">
   <string notr="true"/>
  </property>
  <widget class="QWidget" name="centralWidget">
   <property name="minimumSize">
    <size>
     <width>0</width>
     <height>0</height>
    </size>
   </property>
   <property name="maximumSize">
    <size>
     <width>16777215</width>
     <height>16777215</height>
    </size>
   </property>
   <property name="autoFillBackground">
    <bool>false</bool>
   </property>
   <property name="styleSheet">
    <string notr="true">background-color: rgb(240, 240, 240);</string>
   </property>
   <widget class="OCCWidget" name="occWidget" native="true">
    <property name="geometry">
     <rect>
      <x>0</x>
      <y>0</y>
      <width>800</width>
      <height>800</height>
     </rect>
    </property>
   </widget>
  </widget>
  <widget class="QStatusBar" name="statusBar">
   <property name="statusTip">
    <string/>
   </property>
   <property name="accessibleName">
    <string/>
   </property>
   <property name="styleSheet">
    <string notr="true">background-color: rgb(240, 240, 240);</string>
   </property>
  </widget>
  <widget class="QMenuBar" name="menuBar">
   <property name="geometry">
    <rect>
     <x>0</x>
     <y>0</y>
     <width>1200</width>
     <height>21</height>
    </rect>
   </property>
   <property name="styleSheet">
    <string notr="true">background-color: rgb(240, 240, 240);
selection-background-color: rgb(144, 200, 246);
color: rgb(0, 0, 0);</string>
   </property>
   <widget class="QMenu" name="menu">
    <property name="title">
     <string>文件</string>
    </property>
    <widget class="QMenu" name="menu_2">
     <property name="title">
      <string>导入</string>
     </property>
     <addaction name="actionObject"/>
     <addaction name="actionCloud"/>
     <addaction name="actionDir"/>
    </widget>
    <widget class="QMenu" name="menu_3">
     <property name="title">
      <string>导出</string>
     </property>
     <addaction name="actionSaveObj"/>
     <addaction name="actionSaveCloud"/>
     <addaction name="actionSpecific"/>
    </widget>
    <addaction name="menu_2"/>
    <addaction name="menu_3"/>
    <addaction name="actionDelete"/>
   </widget>
   <widget class="QMenu" name="menu_4">
    <property name="title">
     <string>算法</string>
    </property>
    <addaction name="actionFPIA"/>
   </widget>
   <addaction name="menu"/>
   <addaction name="menu_4"/>
  </widget>
  <action name="actionDelete">
   <property name="text">
    <string>清除</string>
   </property>
  </action>
  <action name="actionDir">
   <property name="text">
    <string>批量</string>
   </property>
  </action>
  <action name="actionObject">
   <property name="text">
    <string>物件</string>
   </property>
  </action>
  <action name="actionCloud">
   <property name="text">
    <string>点云</string>
   </property>
  </action>
  <action name="actionSaveObj">
   <property name="text">
    <string>物件</string>
   </property>
  </action>
  <action name="actionSaveCloud">
   <property name="text">
    <string>点云</string>
   </property>
  </action>
  <action name="actionFPIA">
   <property name="text">
    <string>围板法</string>
   </property>
  </action>
  <action name="actionSpecific">
   <property name="text">
    <string>指定</string>
   </property>
  </action>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <customwidgets>
  <customwidget>
   <class>OCCWidget</class>
   <extends>QWidget</extends>
   <header location="global">OCCWidget.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources>
  <include location="MainWindow.qrc"/>
 </resources>
 <connections/>
</ui>

```

以及

```cpp
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>OCCWidget</class>
 <widget class="QWidget" name="OCCWidget">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>800</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <widget class="QFrame" name="frame">
   <property name="geometry">
    <rect>
     <x>0</x>
     <y>0</y>
     <width>800</width>
     <height>800</height>
    </rect>
   </property>
   <property name="frameShape">
    <enum>QFrame::StyledPanel</enum>
   </property>
   <property name="frameShadow">
    <enum>QFrame::Raised</enum>
   </property>
  </widget>
 </widget>
 <resources/>
 <connections/>
</ui>

```



#### QRC

空的资源文件

```html
<RCC>
    <qresource prefix="MainWindow">
    </qresource>
</RCC>

```



#### main

主程序

```cpp
#include <occmesh/frame/MainWindow.h>

#include <QtWidgets/QApplication>

int main(int argc, char* argv[]) {
    QApplication a(argc, argv);
    MainWindow w;
    w.show();
    return a.exec();
}
```



## VTK

VTK 是一种用于计算机图形、可视化和图像处理的面向对象的开源软件系统。我们首先介绍 VTK 系统的基本架构。



### 低级对象模型

所有 VTK 对象模型可以看作从 vtkObject 超类中继承而来。几乎所有的 VTK 类都继承自它，除了某些特殊情况继承自 vtkObjectBase 超类。所有的 VTK 对象需要用 New() 方法来创建，使用 Delete() 方法销毁。 VTK 对象不能分配在栈上，因为构造函数被保护。



#### 智能指针

为了简化 New()/Delete() 操作，VTK 提供了智能指针类型。主要有两种

```cpp
template <class T> class vtkSmartPointer: public vtkSmartPointerBase;
template <class T> class vtkNew;
```

前者可以隐式转换为原始指针 `T*`，后者需要通过 `Get()` 获得。一般在局部使用更轻量的 vtkNew，而全局使用 vtkSmartPointer 。



#### 运行类型信息

所有 VTK 的公共类接口都可以获得字符串类名，因此足以通过字符串区分它们。例如

```cpp
const char* type = obj->GetClassName();
```

我们可以用 `IsA()` 方法判断某个类是不是某种特殊类的实例。例如

```cpp
vtkNew<vtkBaseClass> obj;
if(obj->IsA("vtkExampleClass")) { ... }
```

然后使用 `SafeDownCase()` 安全地转化为继承类的指针

```cpp
vtkNew<vtkExampleClass> example = vtkExampleClass::SafeDownCase(obj->GetPointer());
```

这个函数会检查转换目标是否真的是子类，如果不是就会返回空指针。



#### 物体状态

VTK 提供了输出物体当前状态信息的函数用于调试

```cpp
obj->Print(cout);
```



### 渲染引擎

下面主要说明渲染过程中常用的对象。



#### vtkProp

场景中的数据的可见描述由 vtkProp 的子类表示。在 3D 显示物体时，最常用的是 vtkActor（用于表示几何数据）和 vtkVolume（用于表示体积数据），以及 vtkActor2D（表示二维数据）。这些子类负责了解其在场景中的位置、大小和方向。对于 3D 数据，可以直接控制位置、选择和伸缩，或者使用 4x4 变换矩阵。除此之外，它通常持有 mapper 对象控制渲染方法，以及属性对象控制颜色、不透明度。



#### vtkAbstractMapper

Prop 类通常持有此类的子类获得输入数据的引用，并提供准确的渲染方法。vtkPolyDataMapper 是渲染多边形几何体的主要映射器。另外 vtkFixedPointVolumeRayCastMapper 用于渲染 vtkImageData，而 vtkProjectedTetrahedraMapper 可用于渲染无结构网格数据。



#### vtkProperty

属性对象保存数据外观的各种参数。例如 vtkActor 使用它储存颜色、不透明度、材质的环境光、漫反射、镜面反射系数等参数。



#### vtkCamera

包含控制观察场景的参数。它包含位置、焦点以及定义“上方”的向量。还有控制平行或透视投影的参数，控制图像比例或视角，以及视锥的近、远裁剪平面。



#### vtkLight

当需要计算照明时，使用该对象储存光的位置、方向、颜色和强度。灯光也具有类型，描述光如何相对相机移动

* Headlight 始终位于相机位置，照射焦点方向；
* Scenelight 则处于场景中的固定位置；



#### vtkRenderer

它提供一个场景来包含相机、灯光、物体等。负责管理场景的渲染过程。多个渲染器可以在一个渲染窗口中使用，用于渲染窗口的不同矩形区域，或者可以重叠。在创建此对象时，会同时创建 vtkCamera 和 vtkLight 对象。



#### vtkRenderWindow

提供操作系统和 VTK 渲染引擎之间的连接，它独立于平台。包含一组 vtkRenderer 和控制渲染功能的参数，例如抗锯齿、焦点和深度。



#### vtkRenderWindowIneractor

用于处理鼠标、按键和计时器事件，通过 VTK 的命令/观察者设计模式实现。vtkIneractorStyle 监听并处理这些事件，提供旋转、平移和缩放等运动机制。此类将会自动创建适用于 3D 场景的默认交互程序样式。



#### vtkTransform

场景中的对象（物体、灯光、相机）都具有此参数，用于控制对象的位置和方向。它可描述 3D 任意仿射变换，内部表示为 4x4 转换矩阵。以默认的单位阵开始，可以添加转换。



#### vtkLookupTable

用于建立从标量值到颜色和不透明度的映射，适用于几何曲面渲染。



## VIS

下面我们具体介绍 VTK 与 OCC 结合使用的方法。



### 组件体系结构

VIS 组件有如下几个部分：

* IVtk 基本物体的通用接口。它完全不需要依赖 VTK 库，且只需要 OCCT 中的 TKernel 库；
* IVtkOCC 实现 CAD 领域相关的接口。这个包中的类处理 OCCT 的拓扑形状、面形和交互式选择工具。它也只依赖 OCCT 库；
* IVtkVTK 实现 VTK 可视化工具箱的接口。它只依赖 VTK 库；
* IVtkTools 集成 VTK 可视化管线的高级工具；

![](Screenshot_20230519_112016.jpg|500)

上述包背后的思想是通过接口与特定库 (OCCT, VTK) 的依赖关系将接口与实际实现分离，避免对其它工具包的过度依赖。**通常只需要使用前三个库就足够了**。



#### IVtk

包含如下类：

* IVtk_Interface 所有接口的基类。提供 Handle (OCCT 智能指针) 功能的继承；
* IVtk_IShape 表示任意性质的 3D 形状。提供其 ID 属性。对所有可视化形状这一接口应当保持唯一 ID 。这些 ID 可以转换为程序中的原始形状；
* IVtk_IShapeData 表示多面数据。提供添加坐标和单元格（顶点、直线、三角形）的方法；
* IVtk_IShapeMesher 多面数据接口，即从 IVtk_IShape 输入的形状构造 IVtk_IShapeData 数据；
* IVtk_IShapePickerAlgo 用于在场景中交互挑选形状的算法接口。提供根据选定的选择模式在给定位置查找形状或者其部分的方法；
* IVtk_IView 获得视图变换参数的接口。它被 IVtk_IShapePickerAlgo 使用；



#### IVtkOCC

它将 IVtk 中的抽象类接口通过结合 OCCT 中的类实现，包含如下依赖 OCCT 的类：

* IVtkOCC_Shape 实现 IVtk_IShape 接口作为 TopoDS_Shape 类的包装器
* IVtkOCC_ShapeMesher 实现 IVtk_IShapeMesher 接口，用于从 TopoDS 形状构造多面体；
* IVtkOCC_ShapePickerAlgo 实现交互式选取算法。可以选择鼠标所在位置的 IVtk_IShape 实例或者它的面；
* IVtkOCC_ViewerSelector 交互式选择器，对 IVtkOCC_IShapePickerAlgo 选取算法实现了 Pick() 方法，并借助抽象接口 IView 链接到可视化层。**它是一个内部类，不是公共 API 的一部分**；
* IVtkOCC_SelectableObject 在选取算法中使用的 OCCT 形状包装器，用于计算给定选取模式下被选择的部分；



#### IVtkVtk

包含依赖 VTK 的类：

* IVtkVTK_ShapeData 对 VTK 的 polydata 实现 IVtk_IShapeData 接口。还存放了子形状的 ID 信息以及网格类型 IVtk_MeshType；
* IVtkVTK_View 对 VTK 视口实现 IVtk_IView 接口。用于将 IVtkOCC_ViewerSelector 连接到 VTK 渲染器；



### 显示 OCCT 形状

要在 VTK 视口可视化一个 OCCT 拓扑形状，需要按照如下步骤：

1. 创建 IVtkOCC_Shape 实例，用一个 TopoDS_Shape 物体初始化。例如

    ```c++
    BRepPrimAPI_MakeBox box(2, 2, 2);		 // 2*2*2 的盒子
    const TopoDS_Shape &shape = box.Shape(); // 获得形状 shape
    
    // 包装为 IVtkOCC_Shape
    IVtkOCC_Shape::Handle shapeHd = new IVtkOCC_Shape(shape);
    ```

    

2. 创建 VTK 多面体数据，使用 IVtkOCC_Shape 实例初始化。例如

    ```cpp
    vtkNew<IVtkTools_ShapeDataSource> occSource;	// 创建一个可以被 vtk 使用的 occ 数据源
    occSource->SetShape(shapeHd); 					// 将 shape 添加到数据源中
    ```

    前两步可以合并为一步，直接使用

    ```cpp
    vtkNew<IVtkTools_ShapeDataSource> occSource;	// 创建一个可以被 vtk 使用的 occ 数据源
    occSource->SetShape(new IVtkOCC_Shape(shape));	// 将 shape 添加到数据源中
    ```

    

3. 用 VTK 的方式可视化载入的形状。例如

    ```cpp
    vtkNew<vtkPolyDataMapper> mapper; 						// 创建一个 vtk 数据类型
    mapper->SetInputConnection(occSource->GetOutputPort()); // 创建一个管道，将 occ 数据导入到 vtk 数据中
    
    vtkNew<vtkActor> actor;				   // 创建一个 vtk actor
    actor->SetMapper(mapper.GetPointer()); // 将 vtk 数据交给 actor
    render->AddActor(actor.GetPointer());  // 在渲染器中加入 vtk actor
    ```



## 源码学习

某些模块的用法可以参照源码。在论坛上查找关键词后，搜索讨论对应的源码。

- `XSDRAWSTLVRML.cxx` 包含演示 MeshVS 的拾取功能
- `OpenGlTest_Commands.cxx` 和 `ViewerTest_OpenGlCommands.cxx` 包含演示 Shader 的链接功能

