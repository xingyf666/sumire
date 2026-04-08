# OpenMesh

## 基本介绍

### 安装编译

在[官网](https://www.graphics.rwth-aachen.de/software/openmesh/)上找到 Download 点击 Daily Builds 链接，进入后可以找到 Windows 版本的 exe 安装包下载安装。

<img src="OpenMesh.assets/image-20230813161436519.png" alt="image-20230813161436519" style="zoom:80%;" />



然后在 cmake 搜索到 OpenMesh 的库目录和头文件目录，最后链接到项目即可

```cmake
# OpenMesh
set(OPENMESH_LIBRARY_DIR "D:/lib/OpenMesh-10.0/lib")
set(OPENMESH_INCLUDE_DIR "D:/lib/OpenMesh-10.0/include")
file(GLOB OPENMESH_LIBRARIES ${OPENMESH_LIBRARY_DIR}/*.lib)
include_directories(${OPENMESH_INCLUDE_DIR})

# 添加编译定义
set_property(DIRECTORY APPEND PROPERTY COMPILE_DEFINITIONS _USE_MATH_DEFINES)

add_executable(main main.cpp)
target_link_libraries(main ${OPENMESH_LIBRARIES})
set_target_properties(main PROPERTIES VS_DEBUGGER_ENVIRONMENT "PATH=D:/lib/OpenMesh-10.0;%PATH%")
```

如果需要进行 debug，应该显式指定不同的链接模式

```cmake
# OpenMesh
set(OPENMESH_LIBRARY_DIR "D:/lib/OpenMesh-10.0/lib")
set(OPENMESH_INCLUDE_DIR "D:/lib/OpenMesh-10.0/include")
include_directories(${OPENMESH_INCLUDE_DIR})

# 对不同模式指定静态库
target_link_libraries(main PRIVATE debug ${OPENMESH_LIBRARY_DIR}/OpenMeshCored.lib)
target_link_libraries(main PRIVATE debug ${OPENMESH_LIBRARY_DIR}/OpenMeshToolsd.lib)
target_link_libraries(main PRIVATE optimized ${OPENMESH_LIBRARY_DIR}/OpenMeshCore.lib)
target_link_libraries(main PRIVATE optimized ${OPENMESH_LIBRARY_DIR}/OpenMeshTools.lib)
```

注意由于显式指定 debug 和 optimized，其它链接库也需要指定。



### 创建立方体

我们先给出一个简单的使用范例，创建一个立方体并保存为 obj 文件

```cpp
#include <iostream>
// -------------------- OpenMesh
#include <OpenMesh/Core/IO/MeshIO.hh>
#include <OpenMesh/Core/Mesh/PolyMesh_ArrayKernelT.hh>

// 命名多边形网格内核
typedef OpenMesh::PolyMesh_ArrayKernelT<> MyMesh;

int main()
{
    MyMesh mesh;
	
    // 通过顶点句柄保存顶点
    MyMesh::VertexHandle vhandle[8];
	
    // 创建顶点，保存到句柄数组
    vhandle[0] = mesh.add_vertex(MyMesh::Point(-1, -1, 1));
    vhandle[1] = mesh.add_vertex(MyMesh::Point(1, -1, 1));
    vhandle[2] = mesh.add_vertex(MyMesh::Point(1, 1, 1));
    vhandle[3] = mesh.add_vertex(MyMesh::Point(-1, 1, 1));
    vhandle[4] = mesh.add_vertex(MyMesh::Point(-1, -1, -1));
    vhandle[5] = mesh.add_vertex(MyMesh::Point(1, -1, -1));
    vhandle[6] = mesh.add_vertex(MyMesh::Point(1, 1, -1));
    vhandle[7] = mesh.add_vertex(MyMesh::Point(-1, 1, -1));

	// 顶点句柄数组
    std::vector<MyMesh::VertexHandle> face_vhandles;
	
    // 取 4 个顶点句柄，构造一个面
    face_vhandles.clear();
    face_vhandles.push_back(vhandle[0]);
    face_vhandles.push_back(vhandle[1]);
    face_vhandles.push_back(vhandle[2]);
    face_vhandles.push_back(vhandle[3]);
    mesh.add_face(face_vhandles);

    face_vhandles.clear();
    face_vhandles.push_back(vhandle[7]);
    face_vhandles.push_back(vhandle[6]);
    face_vhandles.push_back(vhandle[5]);
    face_vhandles.push_back(vhandle[4]);
    mesh.add_face(face_vhandles);

    face_vhandles.clear();
    face_vhandles.push_back(vhandle[1]);
    face_vhandles.push_back(vhandle[0]);
    face_vhandles.push_back(vhandle[4]);
    face_vhandles.push_back(vhandle[5]);
    mesh.add_face(face_vhandles);

    face_vhandles.clear();
    face_vhandles.push_back(vhandle[2]);
    face_vhandles.push_back(vhandle[1]);
    face_vhandles.push_back(vhandle[5]);
    face_vhandles.push_back(vhandle[6]);
    mesh.add_face(face_vhandles);

    face_vhandles.clear();
    face_vhandles.push_back(vhandle[3]);
    face_vhandles.push_back(vhandle[2]);
    face_vhandles.push_back(vhandle[6]);
    face_vhandles.push_back(vhandle[7]);
    mesh.add_face(face_vhandles);

    face_vhandles.clear();
    face_vhandles.push_back(vhandle[0]);
    face_vhandles.push_back(vhandle[3]);
    face_vhandles.push_back(vhandle[7]);
    face_vhandles.push_back(vhandle[4]);
    mesh.add_face(face_vhandles);

    // 写入 obj 文件
    try
    {
        if (!OpenMesh::IO::write_mesh(mesh, "output.obj"))
        {
            std::cerr << "Cannot write mesh to file 'output.obj'" << std::endl;
            return 1;
        }
    }
    catch (std::exception &x)
    {
        std::cerr << x.what() << std::endl;
        return 1;
    }

    return 0;
}
```



### 网格核心

使用网格前，需要指定网格内核。例如前面构造立方体，使用多边形网格内核

```cpp
#include <OpenMesh/Core/Mesh/PolyMesh_ArrayKernelT.hh>

// 命名多边形网格内核
typedef OpenMesh::PolyMesh_ArrayKernelT<> MyMesh;
```

如果使用三角网格，则定义

```cpp
#include <OpenMesh/Core/Mesh/TriMesh_ArrayKernelT.hh>

typedef OpenMesh::TriMesh_ArrayKernelT<> Mesh;
```

为了便于管理网格，可以使用智能指针

```cpp
typedef std::unique_ptr<Mesh> MeshPtr;
```



### 网格读写

引入头文件

```cpp
#include <OpenMesh/Core/IO/MeshIO.hh>
```

可以使用 OpenMesh 的 IO 模块实现网格的读取和写入。封装函数为

```cpp
MeshPtr ReadMesh(const std::string& path, bool exitOnFail = false)
{
    MeshPtr mesh = std::make_unique<Mesh>();
    Mesh& meshRef = *mesh;
	
    // 要求面法向、顶点法向、纹理坐标
    meshRef.request_face_normals();
    meshRef.request_vertex_normals();
    meshRef.request_vertex_texcoords2D();

    // 要求顶点法向、面法向和顶点纹理坐标
    OpenMesh::IO::Options opts;
    opts += OpenMesh::IO::Options::VertexNormal;
    opts += OpenMesh::IO::Options::FaceNormal;
    opts += OpenMesh::IO::Options::VertexTexCoord;

    if (!OpenMesh::IO::read_mesh(meshRef, path, opts))
    {
        std::cerr << "Failed to read mesh at [" << path << "]" << std::endl;

        if (exitOnFail)
            exit(1);

        return nullptr;
    }
	
    // 更新面法向（使用面顶点），更新顶点法向（使用面法向）
    meshRef.update_face_normals();
    meshRef.update_vertex_normals();

    return mesh;
}
```

写入函数封装为

```cpp
bool Write(const std::string& path, const MeshPtr& mesh)
{
    // 指定纹理选项，如果是 .ply 文件就增加颜色选项
    OpenMesh::IO::Options options = OpenMesh::IO::Options::VertexTexCoord;
    if (path.rfind(".ply") != std::string::npos)
        options += OpenMesh::IO::Options::VertexColor;

    // 开始写入
    if (!OpenMesh::IO::write_mesh(*mesh, path, options))
    {
        std::cerr << "Failed to write mesh to [" << path << "]" << std::endl;
        return false;
    }

    return true;
}
```



### 数据类型

OpenMesh 提供了许多有用的数据类型。首先指定网格核心

```cpp
typedef OpenMesh::TriMesh_ArrayKernelT<> Mesh;
```

网格上的迭代器也属于数据类型，但是我们主要介绍其它常用的几种。



#### Mesh::Scalar

网格标量数据，保存浮点数。



#### Mesh::Point

三维点数据结构，重载了基本运算符，可以执行元素的基本四则运算。更重要的是，提供了对点乘、叉乘的重载

```cpp
Mesh::Point p1, p2;
double cosv = p1 | p2;		// 点乘
Mesh::Point p3 = p1 % p2;	// 叉乘
```

使用 `[]` 访问元素。



#### Mesh::TexCoord2D

二维纹理数据结构，用来保存纹理数据，使用 `[]` 访问元素。



## 半边数据结构

下面我们介绍 OpenMesh 的核心数据结构，也就是半边数据结构。其中主要定义了

* 顶点：每个顶点是一个 `Mesh::Vertex` 结构，通常操作其句柄 `Mesh::VertexHandle`
* 边：每个边是一个 `Mesh::Edge` 结构，通常操作其句柄 `Mesh::EdgeHandle`
* 半边：每个半边是一个 `Mesh::Halfedge` 结构，通常操作其句柄 `Mesh::HalfedgeHandle`
* 面：每个面是一个 `Mesh::Face` 结构，通常操作其句柄 `Mesh::FaceHandle`

OpenMesh 通过句柄来管理网格数据，例如顶点、面、边、半边等。因此通常我们不需要直接使用基本网格数据，而是通过句柄来进行各种操作。

<img src="OpenMesh.assets/halfedge_structure3.png" alt="halfedge_structure3" style="zoom:80%;" />

需要注意的是，在 OpenMesh 中**所有边界上的边也都有两条半边**，其中某一条半边没有保存对应的面。



可以获得顶点、边、半边、面的数量

```cpp
mesh->n_vertices();
mesh->n_edges();
mesh->n_halfedges();
mesh->n_faces();
```



### 句柄结构

OpenMesh 通过句柄来管理网格数据，例如顶点、面、边、半边等。因此通常我们不需要直接使用基本网格数据，而是通过句柄来进行各种操作。



#### 顶点句柄

可以遍历网格的**未删除**顶点句柄

```cpp
for (auto& vertex : mesh->vertices())
{
	// vertex 是智能顶点句柄
}
```

如果要访问所有顶点句柄，应该使用

```cpp
for (auto v_it = mesh->vertices_begin(), v_end = mesh->vertices_end(); v_it != v_end; v_it++)
{
	// *v_it 是顶点句柄
}
```



通过顶点句柄可以获得顶点的位置、纹理、法向等信息

```cpp
auto& p = mesh->point(vertex);			// 获得顶点的位置
auto& tex = mesh->texcoord2D(vertex);	// 获得顶点的纹理坐标

// 设置顶点纹理坐标
Mesh::TexCoord2D t;
mesh->set_texcoord2D(vertex, t);
```



#### 边句柄

可以遍历网格的**未删除**边句柄

```cpp
for (auto& edge : mesh->edges())
{
	// edge 是智能边句柄
}
```

如果要访问所有边句柄，应该使用

```cpp
for (auto e_it = mesh->edges_begin(), e_end = mesh->edges_end(); e_it != e_end; e_it++)
{
	// *e_it 是边句柄
}
```



通过边句柄可以获得它的两条半边句柄

```cpp
auto& h1 = mesh->halfedge_handle(edge, 0);
auto& h2 = mesh->halfedge_handle(edge, 1);
```

还可以计算网格属性，例如计算边两侧的两个面的二面角

```cpp
mesh->calc_dihedral_angle_fast(edge);
```



#### 半边句柄

可以遍历网格的**未删除**半边句柄

```cpp
for (auto& halfedge : mesh->halfedges())
{
	// halfedge 是智能半边句柄
}
```

如果要访问所有半边句柄，应该使用

```cpp
for (auto h_it = mesh->halfedges_begin(), h_end = mesh->halfedges_end(); h_it != h_end; h_it++)
{
	// *h_it 是半边句柄
}
```



通过半边句柄可以获得它所在的边句柄和所在面的句柄

```cpp
auto& edge = mesh->edge_handle(halfedge);
auto& face = mesh->face_handle(halfedge);
```

需要注意**边界半边获得的面句柄无效**，可以通过

```cpp
halfedge->is_boundary();
face->is_valid();
```

判定是否是边界半边或面句柄是否有效。



还可以获得起点和终点的顶点句柄

```cpp
auto& tv = mesh->to_vertex_handle(halfedge);
auto& fv = mesh->from_vertex_handle(halfedge);
```

可以计算半边方向向量

```cpp
Mesh::Point p = mesh->calc_edge_vector(halfedge);
```



#### 面句柄

可以遍历网格的**未删除**面句柄

```cpp
for (auto& face : mesh->faces())
{
	// face 是智能面句柄
}
```

如果要访问所有面句柄，应该使用

```cpp
for (auto f_it = mesh->faces_begin(), f_end = mesh->faces_end(); f_it != f_end; f_it++)
{
	// *f_it 是面句柄
}
```



### 循环迭代器

OpenMesh 网格操作的核心就是循环迭代器。通过迭代器访问网格具有拓扑连接的结构。



迭代器按照其功能命名，以 c 开头的方法会得到常量迭代器，例如

```cpp
auto v_it = mesh->cvv_begin(vertex);
```

获得常量顶点迭代器。



之后依次出现输入迭代器类型和输出迭代器类型，例如以顶点句柄为输入的方法

| 命名 | 作用                        |
| ---- | --------------------------- |
| cvv  | 常量+输入顶点+输出相邻顶点  |
| cvf  | 常量+输入顶点+输出相邻面    |
| ve   | 输入顶点+输出相邻边         |
| vih  | 输入顶点+输出 income 半边   |
| voh  | 输入顶点+输出 outgoing 半边 |

其它输入句柄类型的方法也按照类似的方式命名。例如面句柄输入

| 命名 | 作用                  |
| ---- | --------------------- |
| fh   | 输入面+输出面上的半边 |
| fv   | 输入面+输出面上的顶点 |
| ff   | 输入面+输出相邻面     |
| fe   | 输入面+输出面上的边   |



可以指定迭代器的迭代顺序，例如

| 命名   | 作用                                                         |
| ------ | ------------------------------------------------------------ |
| vv_cw  | 输入顶点+输出相邻顶点+顺时针方向 **c**lock**w**ise           |
| vv_ccw | 输入顶点+输出相邻顶点+逆时针方向 **c**ounter-**c**lock**w**ise |

注意**顺序迭代器效率更低**，因此如果没有顺序要求最好不指定顺序。



### 网格属性

#### 默认属性

网格的许多属性，例如法向、纹理等信息需要开启以后才能使用

```cpp
mesh->request_face_normals();			// 启用面法向
mesh->request_vertex_normals();			// 启用顶点法向
mesh->request_vertex_texcoords2D();		// 启用顶点二维纹理
```

这些属性可以通过句柄来指定，有些也可以调用 OpenMesh 的方法来生成。例如

```cpp
mesh->update_face_normals();	// 通过面的顶点位置自动生成面法向
mesh->update_vertex_normals();	// 通过面法向生成顶点法向
```



#### 自定义属性

可以自己为网格添加属性，例如定义一个类型句柄

```cpp
OpenMesh::VPropHandleT<int> ids;
```

其中 V 表示顶点，类似地可以有

```cpp
OpenMesh::HPropHandleT<int> ids;	// 半边
OpenMesh::EPropHandleT<int> ids;	// 边
OpenMesh::FPropHandleT<int> ids;	// 面
```

然后可以通过句柄添加属性

```cpp
mesh->add_property(ids);
```

表示为每个顶点句柄增加了 int 类型的属性，命名为 ids 。这样可以通过顶点句柄访问，例如给指定顶点的 ids 属性赋值为 10

```cpp
mesh->property(ids, vertex) = 10;
```



如果不需要再使用该属性，应该删除属性

```cpp
mesh->remove_property(ids);
```



如果自定义属性出现内存错误，那么大概率是使用错误的迭代器给属性赋值。例如

```cpp
auto h_it = m_mesh->fh_ccwbegin(face), h_end = m_mesh->fh_ccwbegin(face);
```

错误地将起始和结束迭代器设为相同位置，导致后续迭代过程错误。

