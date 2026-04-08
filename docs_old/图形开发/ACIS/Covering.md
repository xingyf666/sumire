# Covering

## 基本介绍

Covering 组件使用一条或多条 edge 环路来拟合 surface 构建 face 所需要的拓扑。环路 circuit 是指 edge 首尾相接形成的闭合回路 loop 。使用 circuit 是为了与拓扑中的 loop 相区别。Covering 算法将 edges 的每个 circuit 变为 face 上的 loop 。Covering 可以用于通过 wire 创建 sheet 或开放 solid，还可以向现有 sheet 或 solid 中添加 face 。输入 edge 必须没有上层拓扑结构。



Covering 的两个不同目标是

* 覆盖没有对应 face 或 surface 信息的 wire 物体。

下图中由 4 条 edge 组成的 wire 被 surface 覆盖，构建出必要的拓扑来创建 sheet 物体。

![[image-20231010173254563.png]]

* 覆盖一个 sheet 或 solid 中的开口。

这通过覆盖一个或更多自由边构成的回路实现。当存在相邻面 faces 时，使用其中之一的底层 surface 来覆盖开口。下图展示了同一个曲面 surface 上的两个 faces 实现的覆盖。

![[image-20231010193054129.png]]

这种覆盖的变体是覆盖 solid 中的一组循环 edge 来创建中间 face 。下图展示了如何使用现有曲面来创建中间 face 。

![[image-20231010210346845.png]]

注意：覆盖 sheet 来创建 solid 时，首先要使用 `api_body_to_1d` 将 sheet 转换为开放 solid 然后执行 covering 操作。



## Covering 变体

### Wire 的基本平面 Covering

基本平面 covering 是最高效的平面 covering 操作。它通过共面的封闭 wire 来生成平面。这个平面成为一个 sheet 的单/双边 face 的底层几何。第一个 wire 的 edge 转换为 face 的外边界环路，其它的 wire 转换为洞环路。每个 wire 必须封闭且没有分支，互相不交。

```cpp
outcome api_cover_wire_loops(ENTITY_LIST &wires, BODY *&sheet, AcisOptions *ao = NULL)
```

![[image-20231010214722735.png]]



### Wire 的增强平面 Covering

增强平面 covering 执行一系列平面 covering 操作和布尔操作。它支持

* 创建覆盖以多个 wire 为边界的区域的单个 face
* 创建覆盖分支 wire 的多个 face
* 创建有多个洞的多个 face

它通过共面的封闭 wire 来生成平面。这个平面成为一个 sheet 的单/双边 face 的底层几何。每个 wire 互相不交。

```cpp
outcome api_cover_planar_wires(ENTITY_LIST &in_wbs, BODY *&out_bdy, ENTITY_LIST &wires, int nesting = TRUE, AcisOptions *ao = NULL)
```

增强平面 covering 可以在考虑或不考虑嵌套 nesting 的情况下使用。如果不考虑嵌套，每个 wire 上的封闭区域都会被覆盖，并且合并；如果考虑嵌套，就存在包含关系。最外侧边界作为外边界环路，其它被该边界直接包含的边界作为洞环路。所有被洞边界直接包含的边界作为外边界。



下图展示了分支 wire 的覆盖结果。

![[image-20231011124316577.png]]

下图展示了使用嵌套 nesting 的覆盖，在上图的基础上新增加了两个 wire 。如果将 nesting 设为 `TRUE`，则得到下图的两个内部环路；如果设为 `FALSE`，则与上图结果相同。

![[image-20231011133141646.png]]

下图进一步展示了嵌套效果，covering 算法通过 wire 之间的嵌套关系来决定外部环路。

![[image-20231011133420807.png]]



### Edge 的增强平面 Covering

Edge 的增强平面 covering 与 wire 的增强平面 covering 类似，有几个主要区别

* 它接受 edge 的列表而不是 wire 作为输入
* 如果任何 edge 有上层拓扑，则该上层拓扑实体下的所有 edge 都会被拷贝，然后原始上层拓扑实体会被删除
* 通过 输入 edge 的连通集创建 wire 。相邻的输入 edge 不需要共享顶点，但是其端点必须在 `SPAresabs` 下重合

执行完这些操作后，就会使用 wire 的增强平面 covering 算法。

```cpp
outcome api_cover_planar_edges(ENTITY_LIST &eds, BODY *&out_bdy, ENTITY_LIST &wires, int nesting = TRUE, AcisOptions *ao = NULL)
```

下图展示了可以通过覆盖线 edge 的封闭回路，然后衔接，将 sheet 转换为 solid 来构建  4 面体。 注意边已经被复制，因此 covering 算法将获得没有更高级别拓扑的边。

![[image-20231011134107182.png]]

此算法可以被更高层的 covering 算法调用。例如用于生成覆盖 sheet 或 solid 中的洞的平面 face：复制自由 edge 然后添加新的 edge 来创建平面区域边界，使用平面 face 覆盖 edge 的平面回路，然后将 face 衔接到现有 body 中。

![[image-20231011134508304.png]]



### 单 Wire 的非平面 Covering

覆盖非平面几何的最简单算法就是覆盖单个、封闭、无分支 wire 。该 wire 转换为 face 上的外边界环路。如果 wire 不能转换为 loop，则函数将会警告而不是报错。之后 covering 曲面生成算法将通过新创建的 loop 的 edge 来产生一张曲面，后者成为开放 solid 的单侧面的底层几何。

```cpp
outcome api_cover_wire(WIRE *wire, surface const &surf, FACE *&face, AcisOptions *ao = NULL)
outcome api_cover_wire(WIRE *wire, cover_options *cov_opts = NULL, AcisOptions *ao = NULL)
```

![[image-20231011134902910.png]]



### 多 Wire 的非平面 Covering

此算法通过一个或多个封闭 wire 的 edge 来创建曲面，后者成为一个或多个开放 solid 的单侧面的底层几何。

```cpp
outcome api_cover_wires(BODY *wire_body, surface const &surf, ENTITY_LIST &faces_list, AcisOptions *ao = NULL)
outcome api_cover_wires(BODY *wire_body, cover_options *cov_opts, AcisOptions *ao = NULL)
```

使用的 wire 必须封闭、无分支、不交。每个封闭 wire 的 edge 转换为 face 上的 loop 。如果 covering 曲面生成算法不能找到一个曲面符合这些 loop，就会尝试对每个 loop 都产生一个平面。因此此算法可能创建有多个 loop 的单个面，或者有一个 loop 的多个面。下图展示了通过指定的曲面来覆盖多个 wire 的情况；如果没有指定曲面，则算法不能找到这个面，因此产生了两张平面。 

![[image-20231011141552601.png]]



### 自由 Edge 的非平面 Covering

此算法通过开放 solid 的自由边来创建曲面，后者成为一个或多个开放 solid 的单侧面的底层几何。该曲面成为实体中一个或多个单侧面的底层几何体，从而覆盖实体边界中的孔。每个自由边构成的回路都被转换为新 face 中的 loop 。

```cpp
outcome api_cover_sheet(BODY *sheet, surface const &surf, ENTITY_LIST &faces_list, logical multiple_cover = FALSE, AcisOptions *ao = NULL)
outcome api_cover_sheet(BODY *sheet, cover_options *cov_opts = NULL, AcisOptions *ao = NULL)
```

上面两个 API 具有不同的表现。前者接收 `multiple_cover` 参数，如果为 `TRUE`，则首先识别所有外部边的分离环路，然后检查这些环路能否被多个平面覆盖。如果能，就会返回这些平面；否则就会尝试用一张曲面覆盖。

![[image-20231011143625498.png]]

下图展示了单个开口如何被多个平面覆盖

![[image-20231011145354082.png]]

下图展示了多个开口如何被平面覆盖，但是在这一步操作不能覆盖非平面开口；后续的覆盖操作使用边界边的底层曲线实现。

![[image-20231011150117863.png]]

下图展示了覆盖平面 face 的情况。第一个模型是单侧面，当它被另一张单侧面覆盖，则会产生与原始平面法向相反的面。这导致层状实体结构。虽然它在 ACIS 中非法，但是可能用于其它应用创建更复杂模型的中间过程。第二个模型则会创建另一张有两个 loop 的平面，其法向会原始平面相反。

![[image-20231011150353507.png]]



### Edge 环路的非平面 Covering

最一般的 covering 算法就是环路 covering 算法。曲面生成算法通过一个或多个封闭环路的边来拟合一张曲面。

```cpp
outcome api_cover_circuits(int num_circuits, ENTITY_LIST *array_circuits[], const surface &surf, ENTITY_LIST &faces_list, AcisOptions *ao = NULL)
outcome api_cover_circuits(int num_circuits, ENTITY_LIST *array_circuits[], cover_options *cov_opts = NULL, AcisOptions *ao = NULL)
```

每个环路 circuit 是边的有序集，形成 face 上的封闭 loop 。其中的边可能在 face 或 wire 上。所有的边必须在同一个 body 中。下图展示了通过添加 wire 定义 face 的边界，逐步创建 face 的过程。

![[image-20231011153404026.png]]

下图展示给环路指定一张覆盖曲面得到的结果。初始是由一个带有 L 形洞的面构成的 sheet，通过周围 face 底层的 surface 来覆盖洞。注意新创建的 face 是双侧 face，因为周围 face 是双侧面。 

![[image-20231011153531940.png]]

下图展示用单张面覆盖多个环路的结果

![[image-20231011153603874.png]]

下图展示封闭 solid 中的环路被覆盖，这里使用了原始圆柱的底层圆柱面实现覆盖。

![[image-20231011153647085.png]]



### Covering 曲面生成算法

此算法对 rubber face，即无底层几何的 face 的所有边尝试找到插值曲面。它在成功创建出这样的曲面后停止。

* 尝试找到单个平面曲面拟合所有边
* 如果存在某个单边，它的底层几何是面面交线，而另一张曲面是解析曲面，则会尝试使用这一曲面。注意此技术只在覆盖面上的洞时使用，而不用于覆盖 wire
* 如果有 4 个非外部边界曲线，则会尝试 B 样条曲面
* 如果有 4 条边界曲线，其中两条相对的边是直线，则将尝试使用直纹面
* 如果有 4 条边界曲线，则尝试 net 曲面
* 如果有 2 条边界曲线，则尝试直纹面
* 如果有 1,3 或超过 4 条边界曲线，或者 `new_cover` 选项为 `TRUE`，则会尝试 $n$ 边曲面。这种 $n$ 边曲面设计用于桥接顶点，并且要求凸边界
* 尝试 B 样条插值
* 如果有多个边界环路，且每个环路都是平面，则尝试对每个环路构建平面

这种多步曲面生成算法拟合 rubber face 的第一个 loop 的边。如果存在多个 loop，则它假设这些 loop 位于同一曲面上。如果超过一个 loop 存在，指定曲面会更好。



### 通过 Covering 创建容差实体

创建非平面几何的 API 函数

| api_cover_wire | api_cover_wires | api_cover_sheet | api_cover_circuits |
| :------------: | :-------------: | :-------------: | :----------------: |

可以创建容差边和顶点。如果

* API 函数被传入的 `cover_options` 函数中 gap 容差超过 `SPAresabs` 且
* 指定了 covering 曲面或一个或多个平面 face 被创建

这些情况下会产生容差。其它情况下都会精确覆盖边界边。



## dcl_cover

### 头文件

```cpp
/*******************************************************************/

/*    Copyright (c) 1989-2020 by Spatial Corp.                     */
/*    All rights reserved.                                         */
/*    Protected by U.S. Patents 5,257,205; 5,351,196; 6,369,815;   */
/*                              5,982,378; 6,462,738; 6,941,251    */
/*    Protected by European Patents 0503642; 69220263.3            */
/*    Protected by Hong Kong Patent 1008101A                       */
/*******************************************************************/
#ifndef DECL_COVR

#if defined(__cover) || defined(__SpaACIS) || defined(__SPAAcisDs)
#define BUILDING_LOCAL_FILE
#endif

#include "importexport.h"
#ifdef IMPORT_EXPORT_SYMBOLS
#ifdef BUILDING_LOCAL_FILE
#define DECL_COVR EXPORT_SYMBOL
#else
#define DECL_COVR IMPORT_SYMBOL
#endif
#else
#define DECL_COVR
#endif

/* force link in VC++ */

#if !defined(CONCAT)
#define CONCAT2(a, b) a##b
#define CONCAT(a, b) CONCAT2(a, b)
#endif

#ifndef SPA_NO_AUTO_LINK
#ifndef BUILDING_LOCAL_FILE
#if defined(_MSC_VER)
#if defined(SPA_INTERNAL_BUILD) || defined(NOBIGLIB)
#define spa_lib_name "cover"
#else
#define spa_lib_name "SpaACIS"
#endif
#if defined(_DEBUG) && !defined(SPA_INTERNAL_BUILD)
#pragma comment(lib, CONCAT(spa_lib_name, "d.lib"))
#else
#pragma comment(lib, CONCAT(spa_lib_name, ".lib"))
#endif
#endif
#endif
#endif

#undef BUILDING_LOCAL_FILE
#undef spa_lib_name

#endif /* DECL_COVR */

```



### cover_options

`cover_options` 用来控制 covering API 函数的行为。



#### 头文件

```cpp
/*******************************************************************/

/*    Copyright (c) 1989-2020 by Spatial Corp.                     */
/*    All rights reserved.                                         */
/*    Protected by U.S. Patents 5,257,205; 5,351,196; 6,369,815;   */
/*                              5,982,378; 6,462,738; 6,941,251    */
/*    Protected by European Patents 0503642; 69220263.3            */
/*    Protected by Hong Kong Patent 1008101A                       */
/*******************************************************************/

#ifndef COVER_OPTIONS_HXX
#define COVER_OPTIONS_HXX
#include "dcl_covr.h"

// 内存管理头文件
#include "mmgr.hxx"

class cover_options_impl;
class surface;
class tolerize_entity_opts;
class ENTITY_LIST;
class outcome;
class AcisOptions;
class BODY;
class WIRE;

class DECL_COVR cover_options : public ACIS_OBJECT
{
public:
    void set_covering_surface(const surface &surf);

    void set_gap_tol(double tol);

    double get_gap_tol() const;

    void set_tolerize_entity_opts(tolerize_entity_opts *te_opts);

    tolerize_entity_opts const *get_tolerize_entity_opts() const;

    void get_output_faces(ENTITY_LIST &out_faces) const;

    void set_allow_rubber_faces(logical allow_rubber_face);

    logical get_allow_rubber_faces() const;

    void set_failsafe_mode(logical failsafe);

    logical get_failsafe_mode() const;

    cover_options();

    cover_options_impl *get_impl();

    ~cover_options();

private:
    cover_options_impl *_impl;
};
#endif
```



#### 成员函数

| cover_options.hxx                                                                                                    |                                                                                                                                                                                                                                                                    |                                                                          |
| -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------ |
| 返回类型                                                                                                                 | 名称                                                                                                                                                                                                                                                                 | 功能                                                                       |
| void                                                                                                                 | [cover_options](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html#a0d2d67fb3debb876daf8df3bade02e41) ()                                                                                                                                 | 创建 cover_option 对象                                                       |
| **[logical ](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html)**                                | [get_allow_rubber_faces](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html#a8411e1e8d1bd4ec000054a8a6b86c7ec) () const                                                                                                                  | 返回是否允许无底层几何的 face，默认为 `FALSE`                                            |
| logical                                                                                                              | [get_failsafe_mode](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html#a31d08d3d49389bdf70373b6817358f55) () const                                                                                                                       | 返回 fail-safe 模式的状态，默认为 `TRUE`                                            |
| double                                                                                                               | [get_gap_tol](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html#a653b1af92cfeb753925cba2dac76e1d2) () const                                                                                                                             | 返回 gap 容差，默认为 `SPAresabs`，此时不会对结果进行公差化                                   |
| void                                                                                                                 | [get_output_faces](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html#a70949d44a64bfdbf5fb9bf2492342807) ([ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &out_faces) const                    | 将 covering 产生的 faces 放到传入的 list 中，不会移除 list 原本就有的内容                      |
| [tolerize_entity_opts](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classtolerize__entity__opts.html) const * | [get_tolerize_entity_opts](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html#a059c291a004205be7208edcd6d86bd4a) () const                                                                                                                | 返回 `tolerize_entity_opts` 对象指针。如果用户没有指定这个对象，就会返回 `NULL` 指针               |
| void                                                                                                                 | [set_allow_rubber_faces](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html#ac3fdb55a51b955f96eee84b9f6cf51a9) (logical allow_rubber_face)                                                                                               | 设置是否允许 rubber face 的选项。如果指定允许，当 covering 曲面不能生成时，可能获得底层几何为 `NULL` 的 face |
| void                                                                                                                 | [set_covering_surface](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html#a64ba05f4dcb6a954c95e35b16fd3a921) (const [surface](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classsurface.html) &surf)                              | 设置 covering 曲面。如果指定了曲面，则 API 函数将使用该曲面 cover 输入的 loop；否则会计算 covering 曲面   |
| void                                                                                                                 | [set_failsafe_mode](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html#a35d57c13f212a8165c9e4fcc28467250) (logical failsafe)                                                                                                             | 设置 fail safe 模式的状态。如果指定 covering 在此模式下完成，则 covering 连通 loop 中的错误会被忽略     |
| void                                                                                                                 | [set_gap_tol](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html#ae4ab6b9621a1c9d662f0f70c2a55a57c) (double tol)                                                                                                                         | 设置 gap 容差。如果指定了有意义的 gap 容差（即大于 `SPAresabs`），则 API 函数会以此容差拟合曲面，然后公差化      |
| void                                                                                                                 | [set_tolerize_entity_opts](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html#ab82016194ff6b9fae2c6080bd2ea747d) ([tolerize_entity_opts](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classtolerize__entity__opts.html) *te_opts) | 设置公差化效果。cover_option 被销毁时仍然会保留输入的 te_opts                                |
| void                                                                                                                 | [~cover_options](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html#a740479d00da3acab771998970b4c5280) ()                                                                                                                                | 销毁 cover_option 对象                                                       |



### coverapi

在 coverapi.hxx 中定义了 Covering 模块的 API 函数。



#### 头文件

```cpp
/*******************************************************************/
/*    Copyright (c) 1989-2020 by Spatial Corp.                     */
/*    All rights reserved.                                         */
/*    Protected by U.S. Patents 5,257,205; 5,351,196; 6,369,815;   */
/*                              5,982,378; 6,462,738; 6,941,251    */
/*    Protected by European Patents 0503642; 69220263.3            */
/*    Protected by Hong Kong Patent 1008101A                       */
/*******************************************************************/
//
/*******************************************************************/
#if !defined(COVERAPI_HXX)
#define COVERAPI_HXX
#include "base.hxx"
class BODY;
class WIRE;
class FACE;
class ENTITY_LIST;
class EDGE;
class SPAposition;
class SPAvector;
class surface;
class cover_options;
#include "dcl_covr.h"
#include "api.hxx"
DECL_COVR outcome api_initialize_covering();

DECL_COVR outcome api_terminate_covering();

DECL_COVR outcome api_cover_circuits(
    int num_circuits,              // number of circuits
    ENTITY_LIST *array_circuits[], // array of pointers to lists of edges
                                   // denoting circuits to be covered
    cover_options *cov_opts = NULL,
    AcisOptions *ao = NULL);

DECL_COVR outcome api_cover_sheet(
    BODY *sheet,
    cover_options *cov_opts = NULL,
    AcisOptions *ao = NULL);

DECL_COVR outcome api_cover_wire(
    WIRE *wire,
    cover_options *cov_opts = NULL,
    AcisOptions *ao = NULL);

DECL_COVR outcome api_cover_wires(
    BODY *wire_body, // wire body to be covered
    cover_options *cov_opts,
    AcisOptions *ao = NULL);

DECL_COVR outcome api_cover_circuits(
    int num_circuits,              // number of circuits
    ENTITY_LIST *array_circuits[], // array of pointers to lists of edges
                                   // denoting circuits to be covered
    const surface &surf,
    ENTITY_LIST &faces_list,
    AcisOptions *ao = NULL);

DECL_COVR outcome api_cover_sheet(
    BODY *sheet,                    // sheet body to be covered
    surface const &surf,            // covering surface (can be NULL)
    ENTITY_LIST &faces_list,        // list of faces made
    logical multiple_cover = FALSE, // option to cover a loop with multiple planes.
    AcisOptions *ao = NULL,
    cover_options const *co = NULL);

DECL_COVR outcome api_cover_wire(
    WIRE *wire,          // wire to be covered
    surface const &surf, // covering surface (can be NULL)
    FACE *&face,         // face made
    AcisOptions *ao = NULL);

DECL_COVR outcome api_cover_wires(
    BODY *wire_body,         // wire body to be covered
    surface const &surf,     // covering surface (can be NULL)
    ENTITY_LIST &faces_list, // list of faces made
    AcisOptions *ao = NULL);

DECL_COVR outcome api_cover_planar_edges(
    ENTITY_LIST &eds,   // free edges to be covered
    BODY *&out_bdy,     // return body containing covered faces
    ENTITY_LIST &wires, // return wire bodies that can't be covered
    int nest = TRUE,    // does the api expected to handle nested loops
    AcisOptions *ao = NULL);
// better performance with nest = FALSE

DECL_COVR outcome api_cover_planar_wires(
    ENTITY_LIST &in_wbs, // wire bodies to be covered
    BODY *&out_bdy,      // return body containing covered faces
    ENTITY_LIST &wires,  // return wire bodies that can't be covered
    int nest = TRUE,     // does the api expected to handle nested loops
    AcisOptions *ao = NULL);
// better performance with nest = FALSE

DECL_COVR outcome api_combine_edges(EDGE *edge1,      // first edge
                                    EDGE *edge2,      // second edge
                                    EDGE *&new_edge,  // resulting edge
                                    FILE *out = NULL, // output file for errors
                                    AcisOptions *ao = NULL);

DECL_COVR outcome api_heal_edges_to_regions(
    ENTITY_LIST &eds,      // edges to be processed
    double coincident_tol, // vertices within the coincident tolerance will be joined
    double length_limit,   // edges under the length limit will be removed
    BODY *&outbdy,         // output body (may be NULL if no region is enclosed
    int wire_only = FALSE, // flag to return wire only
    FILE *fptr = NULL,     // debug file output
    AcisOptions *ao = NULL);

DECL_COVR outcome api_heal_edges_to_regions(
    ENTITY_LIST &eds,       // edges to be processed
    double coincident_tol,  // vertices within the coincident tolerance will be joined
    double length_limit,    // edges under the length limit will be removed
    ENTITY_LIST &outbodies, // return bodies that contains face regions
    FILE *fptr,             // debug file output
    AcisOptions *ao = NULL);

DECL_COVR outcome api_unite_edges(
    ENTITY_LIST &eds, // input edges
    BODY *&outbdy,    // output wire body
    FILE *fptr,       // debug output
    AcisOptions *ao = NULL);

DECL_COVR outcome api_cover_wire_loops(
    ENTITY_LIST &wires, // input wire bodies
    BODY *&sheet,       // sheet body created.
    AcisOptions *ao = NULL);

#endif
```



#### cover_options

可以在 `cover_option` 中指定 gap 容差。若指定了容差，就会对容差范围内是平面的外部环路进行覆盖，然后对产生的 face 的边界几何体进行公差化。类似地，如果同时指定了曲面和容差，则在指定曲面的容差范围内的所有外部环路都会被覆盖，最终的 face 进行公差化。在 cover 操作之后查询 `cover_options` 获得面。



#### api_combine_edges

| 1                                                            | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_combine_edges](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#ga08213e9fa186aee27a7e191e95214a26) ([EDGE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classEDGE.html) *edge1, [EDGE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classEDGE.html) *edge2, [EDGE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classEDGE.html) *&new_edge, FILE *out=NULL, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_combine_edges                                            |
| 函数说明                                                     | 将两个 edge 端点以 $G^1$ 连续组合成一个。edge 都至少要 $G^1$ 连续，否则产生的 edge 不会在其它 ACIS 操作下表现良好 |
| 函数归类                                                     | 组合接口函数                                                 |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [EDGE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classEDGE.html) *edge1 | 第一个 edge 指针                                             |
| [EDGE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classEDGE.html) *edge2 | 第二个 edge 指针                                             |
| [EDGE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classEDGE.html) *&new_edge | 产生的新 edge 指针                                           |
| FILE *out=NULL                                               | 用于输出错误的文件指针                                       |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     |                                                              |



#### api_cover_circuits

| 2                                                            | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_cover_circuits](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#gac15360f327227da593b0538b372667ab) (int num_circuits, [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) *array_circuits[], [cover_options](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html) *cov_opts=NULL, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_cover_circuits                                           |
| 函数说明                                                     | 用一个或多个 face Cover 一个或多个 edge 的环路               |
| 函数归类                                                     | Cover 接口函数                                               |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| int num_circuits                                             | 要覆盖的环路数                                               |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) *array_circuits[] | 环路 edge 列表的指针                                         |
| [cover_options](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html) *cov_opts=NULL | 指定 cover 选项                                              |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 产生的 face 通过 cover_option 来返回，每个输入列表中的 edge 必须形成回路，否则会报错。创建出的 face 会放在拥有第一个 edge 的 body 的第一个 shell 中。这个 API 不处理同时包含 wire 和 face 的物体 |

可以在 `cover_option` 中指定曲面，如果不指定就会使用平面；如果无法找到单个平面，则会尝试样条；如果找不到样条，尝试对共面的环路使用不同的平面，分别创建 face；如果一个 face 不能找到对应的曲面，就会指向 `NULL` 曲面。



| 3                                                            | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_cover_circuits](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#gaf43a8c6181c221fac63a10e30a009161) (int num_circuits, [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) *array_circuits[], const [surface](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classsurface.html) &surf, [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &faces_list, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_cover_circuits                                           |
| 函数说明                                                     | 用一个或多个 face Cover 一个或多个 edge 的环路               |
| 函数归类                                                     | Cover 接口函数                                               |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| int num_circuits                                             | 要覆盖的环路数                                               |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) *array_circuits[] | 环路 edge 列表的指针                                         |
| const [surface](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classsurface.html) &surf | 指定 suface 作为最终 face 的底层几何，可以为 `NULL`          |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &faces_list | 返回新 face 的列表                                           |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     |                                                              |



#### api_cover_planar_edges

Covering 支持 3/4 边非平面 loop 和 $n$ 边平面 loop 。如果有更多非平面边，则在如下情况下覆盖：

* 没有退化边
* 没有夹角大于 $180^\circ$ 或小于 $0$ 的边
* 边没有过度转动

| 4                                                            | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_cover_planar_edges](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#gabf6c187d7bffe4d4b917e75b2e76f766) ([ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &eds, [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *&out_bdy, [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &wires, int nest=TRUE, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_cover_planar_edges                                       |
| 函数说明                                                     | 用平面覆盖平面 edge                                          |
| 函数归类                                                     | Cover 接口函数                                               |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &eds | 要覆盖的自由边                                               |
| [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *&out_bdy | 返回包含 cover 产生的 face 的 body                           |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &wires | 返回不能被 cover 的 wire                                     |
| int nest=TRUE                                                | 如果设为 `TRUE` 则要考虑嵌套                                 |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 如果开启 nest，则 API 预期使用共面 edge，非共面的 edge 不被允许。 |



#### api_cover_planar_wires

Covering 支持 3/4 边非平面 loop 和 $n$ 边平面 loop 。如果有更多非平面边，则在如下情况下覆盖：

* 没有退化边
* 没有夹角大于 $180^\circ$ 或小于 $0$ 的边
* 边没有过度转动

| 5                                                            | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_cover_planar_wires](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#ga357bd61c0eb3fcb4c462afd0f6e4ab3b) ([ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &in_wbs, [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *&out_bdy, [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &wires, int nest=TRUE, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_cover_planar_wires                                       |
| 函数说明                                                     | 用平面覆盖平面 wire                                          |
| 函数归类                                                     | Cover 接口函数                                               |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &in_wbs | 要覆盖的 wire                                                |
| [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *&out_bdy | 返回包含 cover 产生的 face 的 body                           |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &wires[ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &wires | 返回不能被 cover 的 wire                                     |
| int nest=TRUEint nest=TRUE                                   | 如果设为 `TRUE` 则要考虑嵌套                                 |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL[AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 如果开启 nest，则 API 预期使用共面 edge，非共面的 edge 不被允许。 |



#### api_cover_sheet

当 ACIS 使用此函数通过 `FACE` 来创建 `BODY` 时，它会产生单侧 face body，通常是 solid 而不是 sheet（无限薄）。对于单侧物体，ACIS 认为它是从面的背面向外无限延伸的实体，其边界不明确，原始面的边界向后延伸。后续的操作例如布尔运算可能不能执行，这取决于如何使用该单侧 face body 。

可以在 `cover_options` 中指定曲面，此时将使用这个曲面作为最终 face 的底层几何。如果不指定就会使用平面；如果无法找到单个平面，则会尝试样条；如果找不到样条，尝试对共面的环路使用不同的平面，分别创建 face；如果一个 face 不能找到对应的曲面，就会指向 `NULL` 曲面。

如果给出或找到平面，则 API 将 loop 分配到每个曲面的不同连通 face，新的 face 放在给定 body 的第一个 shell 中，返回指向新 face 的指针的列表。



| 6                                                            | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_cover_sheet](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#gac0aebf6c1011b1ebb80bcc48f729767f) ([BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *sheet, [cover_options](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html) *cov_opts=NULL, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_cover_sheet                                              |
| 函数说明                                                     | 通过覆盖 sheet 中的外部边构成的所有简单环路来创建面          |
| 函数归类                                                     | Cover 接口函数                                               |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *sheet | 要覆盖的 sheet                                               |
| [cover_options](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html) *cov_opts=NULL | 指定 cover 选项                                              |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 通过 `cover_options` 返回一列面。如果没有 cover face，则返回空列表以及 bad outcome 。这个 API 不覆盖 wire |



| 7                                                            | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_cover_sheet](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#gafaee694fc0767452b8403cd4665939d0) ([BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *sheet, [surface](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classsurface.html) const &surf, [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &faces_list, logical multiple_cover=FALSE, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL, [cover_options](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html) const *co=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_cover_sheet                                              |
| 函数说明                                                     | 通过覆盖 sheet 中的外部边构成的所有简单环路来创建面          |
| 函数归类                                                     | Cover 接口函数                                               |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *sheet | 要覆盖的 sheet                                               |
| [surface](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classsurface.html) const &surf | 用于覆盖的曲面，可以为 `NULL`                                |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &faces_list | 返回新 face 的列表                                           |
| logical multiple_cover=FALSE                                 | 使用多个平面覆盖的选项                                       |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| [cover_options](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html) const *co=NULL | 指定 cover 选项                                              |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 通过 `cover_options` 返回一列面。如果没有 cover face，则返回空列表以及 bad outcome 。这个 API 不覆盖 wire |

当 `multiple_cover` 设为 `TRUE`，就会首先检查所有分离的 loop，然后检查是否能对每个 loop 产生平面。如果能，结果就是多个 face；否则，就会按照没有这个选项执行操作。



#### api_cover_wire

当 ACIS 使用此函数通过 `FACE` 来创建 `BODY` 时，它会产生单侧 face body，通常是 solid 而不是 sheet（无限薄）。对于单侧物体，ACIS 认为它是从面的背面向外无限延伸的实体，其边界不明确，原始面的边界向后延伸。后续的操作例如布尔运算可能不能执行，这取决于如何使用该单侧 face body 。



| 8                                                            | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_cover_wire](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#ga7a9e86aa47759d616c415b57745a93cd) ([WIRE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classWIRE.html) *wire, [cover_options](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html) *cov_opts=NULL, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_cover_wire                                               |
| 函数说明                                                     | 用一张 face 覆盖 wire 边的回路                               |
| 函数归类                                                     | Cover 接口函数                                               |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [WIRE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classWIRE.html) *wire | 要覆盖的 wire                                                |
| [cover_options](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html) *cov_opts=NULL | 指定 cover 选项                                              |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 输入 wire 必须是边的简单回路。产生的 face 存放在 wire 所属 body 的新 shell 中，原先的 wire 将被删除 |

如果没有在 `cover_options` 中指定曲面，则会用平面拟合。如果找不到平面，就会使用样条；如果找不到曲面，就会指向 `NULL` 曲面。



| 9                                                            | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_cover_wire](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#gae49bc3f1c62cca8d020c1f928b56d7e8) ([WIRE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classWIRE.html) *wire, [surface](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classsurface.html) const &surf, [FACE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classFACE.html) *&face, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_cover_wire                                               |
| 函数说明                                                     | 用一张 face 覆盖 wire 边的回路                               |
| 函数归类                                                     | Cover 接口函数                                               |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [WIRE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classWIRE.html) *wire | 要覆盖的 wire                                                |
| [surface](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classsurface.html) const &surf | 用于覆盖的曲面，可以为 `NULL`                                |
| [FACE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classFACE.html) *&face | 返回新的 face                                                |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 输入 wire 必须是边的简单回路。产生的 face 存放在 wire 所属 body 的新 shell 中，原先的 wire 将被删除 |

输入边可以是直接构造或切片构造，即通过布尔运算获得。切片必须是通过一张 face 获得，并且 face 不与被切片的 body 的 face 重合；face 的边也不与被切片的 body 的边重合。另外，通过布尔运算添加到 wire 边的属性必须预先移除。



#### api_cover_wire_loops

| 10                                                           | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_cover_wire_loops](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#ga35d3f6ef3e61cfa9f26da88ee43f7855) ([ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &wires, [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *&sheet, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_cover_wire_loops                                         |
| 函数说明                                                     | 将共面环路 wire 印到 wire 的平面上创建具有一张平面的 sheet   |
| 函数归类                                                     | Cover 接口函数                                               |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &wires | 要覆盖的 wire 列表                                           |
| [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *&sheet | 返回新的 sheet                                               |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 此函数创建一张平面，把第一个 wire 作为外边界回路，其它 wire 作为洞回路。属于纯拓扑操作，不涉及任何布尔运算。每个 wire 必须是封闭回路且没有分支。wire 之间不能相交。 |



#### api_cover_wires

如果没有在 `cover_options` 中指定曲面，则会拟合平面；如果找不到平面，就会拟合样条曲面。如果给出或找到平面，则 API 将 loop 分配到连通面中。回路的方向对嵌套回路的处理有很大影响；例如一对反向环分别转换为一张面的外边界和洞边界，同向环产生重叠的面。两种情况产生的 face 都会被放在给定 body 的新 shell 中，body 中的所有 wire 将被删除。



| 11                                                           | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_cover_wires](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#gaab0ad7d3bf16fed94653cbee6d6c6ec7) ([BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *wire_body, [cover_options](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html) *cov_opts, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_cover_wires                                              |
| 函数说明                                                     | 覆盖 wire 环路创建面                                         |
| 函数归类                                                     | Cover 接口函数                                               |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *wire_body | 要覆盖的 wire                                                |
| [cover_options](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classcover__options.html) *cov_opts | 指定 cover 选项                                              |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 给定 body 必须由一个或多个 wire 组成，每个 wire 是边的简单回路。 |



Covering 支持 3/4 边非平面 loop 和 $n$ 边平面 loop 。如果有更多非平面边，则在如下情况下覆盖：

* 没有退化边
* 没有夹角大于 $180^\circ$ 或小于 $0$ 的边
* 边没有过度转动

| 12                                                           | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_cover_wires](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#ga52ef052d3130d6cf5b8fdefde6f23cf6) ([BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *wire_body, [surface](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classsurface.html) const &surf, [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &faces_list, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_cover_wires                                              |
| 函数说明                                                     | 覆盖 wire 环路创建面                                         |
| 函数归类                                                     | Cover 接口函数                                               |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *wire_body | 要覆盖的 wire                                                |
| [surface](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classsurface.html) const &surf | 用于覆盖的曲面，可以为 `NULL`                                |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &faces_list | 返回新的 face 列表                                           |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 给定 body 必须由一个或多个 wire 组成，每个 wire 是边的简单回路。 |

输入边可以是直接构造或切片构造，即通过布尔运算获得。切片必须是通过一张 face 获得，并且 face 不与被切片的 body 的 face 重合；face 的边也不与被切片的 body 的边重合。



#### api_heal_edges_to_regions

| 13                                                           | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_heal_edges_to_regions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#ga2ab018c7a0c57f5aee729669a820b762) ([ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &eds, double coincident_tol, double length_limit, [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *&outbdy, int wire_only=FALSE, FILE *fptr=NULL, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_heal_edges_to_regions                                    |
| 函数说明                                                     | 修复共面边顶点以在平面上形成区域                             |
| 函数归类                                                     | Heal 接口函数                                                |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &eds | 输入边的列表                                                 |
| double coincident_tol                                        | 给出点重合的容差                                             |
| double length_limit                                          | 给出限制长度                                                 |
| [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *&outbdy | 输出 body                                                    |
| int wire_only=FALSE                                          | 如果为 `TRUE` 就返回 wire                                    |
| FILE *fptr=NULL                                              | 用于输出错误的文件指针                                       |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 此函数使用给定容差修复共面边的顶点以在平面上形成区域。如果指定了限制长度，则更短的 edge 会被删除。它也能平滑化非 $G^1$ 边，可能导致其形状有很大变化。建议先调用 [api_split_edge_at_disc](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__CSTRMAKEEDGEAPI.html#ga386090e5cab1bd9e10d5bb2aeae2988e) 分割非 $G^1$ 边 |



| 14                                                           | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_heal_edges_to_regions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#ga2ab018c7a0c57f5aee729669a820b762) ([ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &eds, double coincident_tol, double length_limit, [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &outbodies, FILE *fptr, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_heal_edges_to_regions                                    |
| 函数说明                                                     | 修复共面边顶点以在平面上形成区域                             |
| 函数归类                                                     | Heal 接口函数                                                |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &eds | 输入边的列表                                                 |
| double coincident_tol                                        | 给出点重合的容差                                             |
| double length_limit                                          | 给出限制长度                                                 |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &outbodies | 输出 body 列表                                               |
| int wire_only=FALSE                                          | 如果为 `TRUE` 就返回 wire                                    |
| FILE *fptr=NULL                                              | 用于输出错误的文件指针                                       |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 此函数使用给定容差修复共面边的顶点以在平面上形成区域。如果指定了限制长度，则更短的 edge 会被删除。它也能平滑化非 $G^1$ 边，可能导致其形状有很大变化。建议先调用 [api_split_edge_at_disc](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__CSTRMAKEEDGEAPI.html#ga386090e5cab1bd9e10d5bb2aeae2988e) 分割非 $G^1$ 边 |



#### api_initialize_covering

| 15       | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_initialize_covering](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#gaf50d0f9e96b7a0c61f8b913ce141d0cd) () |
| -------- | ------------------------------------------------------------ |
| 函数名   | api_initialize_covering                                      |
| 函数说明 | 初始化 Covering 组件库                                       |
| 函数归类 | Covering 接口函数                                            |
| 优先级   | 1                                                            |
| 返回类型 | outcome                                                      |
| 返回说明 | 保存 API 运行错误码                                          |
| 参数     | 说明                                                         |
| 访问权限 | public                                                       |
| 补充说明 |                                                              |



#### api_terminate_covering

| 16       | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_terminate_covering](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#ga40920a1d71d6aa967d5f5c0469cbbf90) () |
| -------- | ------------------------------------------------------------ |
| 函数名   | api_terminate_covering                                       |
| 函数说明 | 停止 Covering 组件库                                         |
| 函数归类 | Covering 接口函数                                            |
| 优先级   | 1                                                            |
| 返回类型 | outcome                                                      |
| 返回说明 | 保存 API 运行错误码                                          |
| 参数     | 说明                                                         |
| 访问权限 | public                                                       |
| 补充说明 |                                                              |



#### api_unite_edges

| 17                                                           | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_unite_edges](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#ga56c72b6b9e59a421e90b5b8b07009ae7) ([ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &eds, [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *&outbdy, FILE *fptr, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_unite_edges                                              |
| 函数说明                                                     | 使用非正规化单元将 edge 添加到 wire 中                       |
| 函数归类                                                     | Covering 接口函数                                            |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [ENTITY_LIST](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classENTITY__LIST.html) &eds | 给定 edge 列表                                               |
| [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *&outbdy | 输出 body                                                    |
| FILE *fptr                                                   | 调试文件指针                                                 |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     |                                                              |



### edge_utils

#### 头文件

```cpp
/*******************************************************************/

/*    Copyright (c) 1989-2020 by Spatial Corp.                     */
/*    All rights reserved.                                         */
/*    Protected by U.S. Patents 5,257,205; 5,351,196; 6,369,815;   */
/*                              5,982,378; 6,462,738; 6,941,251    */
/*    Protected by European Patents 0503642; 69220263.3            */
/*    Protected by Hong Kong Patent 1008101A                       */
/*******************************************************************/
#ifndef WARP_UTILS_HXX
#define WARP_UTILS_HXX
#include "dcl_oper.h"
#include "logical.h"
#include "base.hxx"
class EDGE;
class outcome;
class SPAvector;
class AcisOptions;
DECL_OPER outcome api_move_edge(
    EDGE *edge,
    SPAvector dist_vector,
    logical extend = TRUE,
    AcisOptions *ao = NULL);
DECL_OPER outcome api_move_edge(
    EDGE *edge,
    double dist,
    logical extend = TRUE,
    AcisOptions *ao = NULL);
#endif
```



#### api_move_edge

| 1                                                            | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_move_edge](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#gac9c7b208da585602f72a67d84d35909b) ([EDGE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classEDGE.html) *edge, [SPAvector](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classSPAvector.html) dist_vector, logical extend=TRUE, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_move_edge                                                |
| 函数说明                                                     | 在 sheet 限制下内部移动 edge 的位置                          |
| 函数归类                                                     | 工具函数                                                     |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [EDGE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classEDGE.html) *edge | 要移动的 edge                                                |
| [SPAvector](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classSPAvector.html) dist_vector | 偏移向量                                                     |
| logical extend=TRUE                                          | 是否扩展                                                     |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 此 API 在指定距离创建具有相同几何的 edge 并将其延伸到指定距离。这一步后，它被印在 sheet 上，旧的 edge 被合并。选项 `extend` 将 edge 延伸到曲面的边界，否则它只移动 edge 不进行伸长。只有直线和椭圆可以被移动，且只能沿着相邻面移动，body 必须是平面 sheet |



| 2                                                            | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_move_edge](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#ga4909b0e9234c7de4efa1a6091cc70cc0) ([EDGE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classEDGE.html) *edge, double dist, logical extend=TRUE, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_move_edge                                                |
| 函数说明                                                     | 在 sheet 限制下内部移动 edge 的位置                          |
| 函数归类                                                     | 工具函数                                                     |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [EDGE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classEDGE.html) *edge | 要移动的 edge                                                |
| double dist                                                  | 偏移距离                                                     |
| logical extend=TRUE                                          | 是否扩展                                                     |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 此 API 在指定距离创建具有相同几何的 edge 并将其延伸到指定距离。这一步后，它被印在 sheet 上，旧的 edge 被合并。选项 `extend` 将 edge 延伸到曲面的边界，否则它只移动 edge 不进行伸长。只有直线和椭圆可以被移动，且只能沿着相邻面移动，body 必须是平面 sheet |



### operapi

#### 头文件

```cpp
/*******************************************************************/

/*    Copyright (c) 1989-2020 by Spatial Corp.                     */
/*    All rights reserved.                                         */
/*    Protected by U.S. Patents 5,257,205; 5,351,196; 6,369,815;   */
/*                              5,982,378; 6,462,738; 6,941,251    */
/*    Protected by European Patents 0503642; 69220263.3            */
/*    Protected by Hong Kong Patent 1008101A                       */
/*******************************************************************/
#if !defined(OPERAPI_HXX)
#define OPERAPI_HXX
#include "base.hxx"
class BODY;
class WIRE;
class EDGE;
class COEDGE;
class ENTITY;
class ENTITY_LIST;
class SPAposition;
class SPAunit_vector;
class law;
class entity_with_ray;
#include "dcl_oper.h"
#include "api.hxx"
#include "logical.h"
DECL_OPER outcome api_wire_area(
    BODY *body,  // Wire body whose area is to be found
    double &area // Area of the wire body returned
    ,
    AcisOptions *ao = NULL);
DECL_OPER outcome api_wire_area(
    WIRE *wire,  // Wire whose area is to be found
    double &area // Area of the wire body returned
    ,
    AcisOptions *ao = NULL);
#endif
```



#### api_wire_area

| 1                                                            | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_wire_area](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#gac38a8a277dcbe5c62249a8d38dda0c14) ([BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *body, double &area, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_wire_area                                                |
| 函数说明                                                     | 确定平面 wire 或 body 中的所有平面 wire 的面积               |
| 函数归类                                                     | oper 接口函数                                                |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [BODY](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classBODY.html) *body | 需要计算的 wire body                                         |
| double &area                                                 | 返回面积                                                     |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 如果 wire 是开放的，API 假定使用直线连接起点和终点来计算面积。如果给定 body 有超过一个 wire，则只考虑第一个。如果 wire 不是平面，或者存在其它错误情况，将返回零。此 API 不保证给出自交 wire 的正确结果 |



| 2                                                            | [outcome](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classoutcome.html) [api_wire_area](https://doc.spatial.com/get_doc_page/qref/ACIS/html/group__COVRAPI.html#gac38a8a277dcbe5c62249a8d38dda0c14) ([WIRE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classWIRE.html) *wire, double &area, [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL) |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 函数名                                                       | api_wire_area                                                |
| 函数说明                                                     | 确定平面 wire 的面积                                         |
| 函数归类                                                     | oper 接口函数                                                |
| 优先级                                                       | 1                                                            |
| 返回类型                                                     | outcome                                                      |
| 返回说明                                                     | 保存 API 运行错误码                                          |
| 参数                                                         | 说明                                                         |
| [WIRE](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classWIRE.html) *wire | 需要计算的 wire body                                         |
| double &area                                                 | 返回面积                                                     |
| [AcisOptions](https://doc.spatial.com/get_doc_page/qref/ACIS/html/classAcisOptions.html) *ao=NULL | ACIS 选项                                                    |
| 访问权限                                                     | public                                                       |
| 补充说明                                                     | 如果 wire 是开放的，API 假定使用直线连接起点和终点来计算面积。如果 wire 不是平面，或者存在其它错误情况，将返回零。此 API 不保证给出自交 wire 的正确结果 |



### cover

#### 头文件

```cpp
/* ORIGINAL: acis2.1/sg_husk/cover/cover.hxx */

/* $Id: cover.hxx,v 1.11 2002/04/19 14:31:34 sallen Exp $ */
/*******************************************************************/
/*    Copyright (c) 1989-2020 by Spatial Corp.                     */
/*    All rights reserved.                                         */
/*    Protected by U.S. Patents 5,257,205; 5,351,196; 6,369,815;   */
/*                              5,982,378; 6,462,738; 6,941,251    */
/*    Protected by European Patents 0503642; 69220263.3            */
/*    Protected by Hong Kong Patent 1008101A                       */
/*******************************************************************/

// Header for cover routines.

// These routines cover closed wires to make sheets, and sheets to
// make solids.

#if !defined(COVER_HDR_DEF)
#define COVER_HDR_DEF

#include "dcl_covr.h"
// ywoo 08Jan01: added the header file.
#include "logical.h"
// ywoo: end

class BODY;
class WIRE;
class FACE;
class LOOP;
class LUMP;
class COEDGE;

class ENTITY_LIST;

class surface;
class generic_graph;

// Set surface if given or attempt to find a plane that contains
// loops of given face.  Re-arrange loops into properly connected
// faces.

DECL_COVR logical
apply_surface(FACE *, surface const *, ENTITY_LIST &);

// Routine to cover a circuit of connected edges.  It adds a
// new coedge to each edge (unless the edge is a wire when the
// existing loopless coedge is used - and may be reversed) and
// places each new coedge in a loop, newly made, which is returned.
// Orients coedges forward along the sequence of edges and checks
// that the given edges are connected.

DECL_COVR LOOP *
cover_circuit(ENTITY_LIST &);

// Routine to cover a collection of circuits of connected edges.
// It makes a new loop for each circuit and places the new loops
// into a new face.  If the pointer to a (lower_case) surface
// is not NULL, makes an upper-case surface and sets it in
// the face with sense FORWARD.  All edges must belong to one
// body.  The new face is added to the first shell of the body.
// The routine may be used to cover wire or sheet bodies a face
// at a time or to make internal faces for cellular bodies.

DECL_COVR void
cover_circuits(int,
               ENTITY_LIST *[],
               surface const *,
               ENTITY_LIST &);

//  Routine to find and cover all simple circuits of external
//  edges of a sheet.  Returns an entity list of new faces
//  made (will be empty if no new faces are made).  New faces
//  will have no surfaces if none can be found to fit their edges
//  and vertices.
//  If surface found or given is planar, re-arranges loops into
//  connected faces.
//  NB: Does not cover wire edges.

// STI ywoo (5/4/00): new option "multiple_cover"
DECL_COVR void
cover_sheet(BODY *, surface const *, ENTITY_LIST &, logical = FALSE);
// STI ywoo: end

// Cover one or more closed slice-wires.  Slice-wires are made by
// the first phase of boolean operations.  They carry attributes
// giving inside-outside data for each coedge (two coedges per edge).
// After covering, these attributes are removed, and each edge has
// only one coedge.

DECL_COVR void
cover_slice_wires(BODY *, surface const *, ENTITY_LIST &);

// Routine to cover a set of closed wires supplied as the wires of
// a wire body, with one or more faces.  It attempts to find a
// surface from the vertices and edges of the wires;
// if none is found, the surface(s) of the face(s) is left NULL.
// Returns a list of faces made.
// NB: the routine assumes the wires are co-planar and closed and
// correctly oriented (metal on the left of their coedges).

DECL_COVR void
cover_wires(BODY *, surface const *, ENTITY_LIST &, logical = FALSE);

// Routine to cover a single closed wire with a faces.  It attempts to find a
// surface from the vertices and edges of the wire;
// if none is found, the surface of the face is left NULL.
// Returns the face made.
// NB: the routine assumes the wire is planar and closed and
// correctly oriented (metal on the left of the coedges).

DECL_COVR void
cover_wire(WIRE *, surface const *, FACE *&);

DECL_COVR void
cover_loops_of_planar_wires(ENTITY_LIST &wires, BODY *&sheet);

DECL_COVR logical
edges_on_surface(const ENTITY_LIST &edges, surface *surf);

#endif
```



#### 成员函数

| cover.hxx           |                                                                            |                                                                                                                                                                                                                                      |
| ------------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 返回类型                | 名称                                                                         | 功能                                                                                                                                                                                                                                   |
| `DECL_COVR logical` | `DECL_COVR logical apply_surface(FACE *, surface const *, ENTITY_LIST &`); | 如果给定曲面则直接设置，否则寻找包含给定 face 上的 loop 的平面，将 loop 分配到正确的连通 face                                                                                                                                                                           |
| `DECL_COVR LOOP *`  | `cover_circuit(ENTITY_LIST &);`                                            | 覆盖连通 edge 的环路。它给每个 edge 添加一个新的 coedge（除非 edge 是 wire）然后将每个 coedge 放在返回的 loop 中。沿着 coedge 方向顺序向前移动，检查给定的边是否连接。                                                                                                                        |
| `DECL_COVR void`    | `cover_circuits(int, ENTITY_LIST *[], surface const *, ENTITY_LIST &);`    | 覆盖连通 edge 的环路集合。它给每个环路创建一个新 loop 将其放到新的 face 中。如果指向小写曲面(`surface`)的指针不是 `NULL`，创建大写曲面(`SURFACE`)并且将其以 `FORWARD` 放在 face 中。所有 edge 必须属于同一个 body 。新的 face 添加到 body 中的第一个 shell 中。此过程可能同时用一张 face 覆盖 wire 或 sheet，也可能制作单元 body 的内部 face |
| `DECL_COVR void`    | `cover_sheet(BODY *, surface const *, ENTITY_LIST &, logical = FALSE);`    | 找到并覆盖 sheet 的所有外部 edge 构成的回路。返回产生的 face 的列表。如果找不到覆盖 edge 的曲面，则返回 face 的底层为空。如果找到或给定的是平面，则会将 loop 分配到连通 face 上。注意它不覆盖 wire 边。                                                                                                         |
| `DECL_COVR void`    | `cover_slice_wires(BODY *, surface const *, ENTITY_LIST &);`               | 覆盖一个或多个封闭 wire 切片。切片 wire 通过布尔运算第一阶段得到。它们携带每个 coedge (每个 edge 有两个 coedge)的内部外部属性数据。覆盖后，这些属性将被删除，每个 edge 只有一个 coedge                                                                                                                  |
| `DECL_COVR void`    | `cover_wires(BODY *, surface const *, ENTITY_LIST &, logical = FALSE);`    | 使用一张或多张 face 覆盖 wire body 中的封闭 wire 的集合。 如果找不到覆盖曲面，则返回 face 带有 `NULL` 。返回 face 的列表。此过程假设 wire 共面封闭且方向正确（材质在 coedge 的左侧）                                                                                                              |
| `DECL_COVR void`    | `cover_wire(WIRE *, surface const *, FACE *&`);                            | 用一张 face 覆盖单个封闭 wire 。如果找不到覆盖曲面，则返回 face 带有 `NULL` 。此过程假设 wire 共面封闭且方向正确（材质在 coedge 的左侧）                                                                                                                                             |
| `DECL_COVR void`    | `cover_loops_of_planar_wires(ENTITY_LIST &wires, BODY *&sheet);`           | \                                                                                                                                                                                                                                    |
| `DECL_COVR logical` | `edges_on_surface(const ENTITY_LIST &edges, surface *surf);`               | \                                                                                                                                                                                                                                    |





## outcome

此类包含一个错误码，用于指示 API 例程的成功或失败，以及公告板指针。此类还包含 error_info 对象，这些对象封装 API 遇到的错误或问题。



```cpp
/*******************************************************************/
/*    Copyright (c) 1989-2020 by Spatial Corp.                     */
/*    All rights reserved.                                         */
/*    Protected by U.S. Patents 5,257,205; 5,351,196; 6,369,815;   */
/*                              5,982,378; 6,462,738; 6,941,251    */
/*    Protected by European Patents 0503642; 69220263.3            */
/*    Protected by Hong Kong Patent 1008101A                       */
/*******************************************************************/
// Header for outcome and macros used in API routines.
// This header must be included directly (or via api/misc.hxx) in
// any application code that calls API routines.  The macros will also
// be useful when applications call Acis routines directly.
#if !defined(API_HEADER)
#define API_HEADER
#include <stdio.h>
#include "dcl_kern.h"
// #include "api.hxx"
#include "logical.h"
#include <setjmp.h>
#include "bullsmal.hxx"
#include "debugmsc.hxx"
#include "errorsys.hxx"
#include "acis_options.hxx"
#include "vers.hxx"
class BULLETIN_BOARD;
#include "err_info.hxx"
#include "err_info_list.hxx"
#include "problems_list.hxx"

// Simple controls on API functioning.
#define api_checking_on (api_check_on())

DECL_KERN void set_api_checking(logical);

DECL_KERN logical api_check_on();

// Define a class for the result of API calls. This will be adaptable
// in the future to allow extra information, but for now contains an
// error number to indicate success/failure of the API routine, together
// with a bulletin board pointer.

// STI swa (17Mar98) -- added pointer to error_info object to allow
// return of additional error information in outcome class.  Also added
// member functions to handle the pointer properly.
class BULLETIN_BOARD;
#define API_SUCCESS SPA_NO_ERROR

class DECL_KERN outcome : public ACIS_OBJECT
{
    // Error number, 0 for successful outcome.

    err_mess_type quant;

    // Pointer to bulletin board for API call that produced the outcome.

    BULLETIN_BOARD *bb_ptr;

    // pointer to error_info object

    error_info *e_info;

    // pointer to a list of problems
    problems_list *problems_ptr;

    // method to instantiate a problems list, if necessary
    problems_list *get_problems_ptr();

public:
    // new constructors for use with error_info objects
    outcome(err_mess_type i = API_SUCCESS, error_info *e = NULL);
    outcome(const outcome &o);

    // destructor needs to remove any error objects
    ~outcome();

    // Data reading routines
    err_mess_type error_number() const { return quant; }

    BULLETIN_BOARD *bb() const { return bb_ptr; }

    // error_info routines
    error_info *get_error_info() const;

    // This method is for internal use only.
    // Sets the error information of this outcome to the contents of the input item.
    void set_error_info(error_info *e);

    // new assignment operator to handle error_info use count
    outcome &operator=(const outcome &o);

    // Set the bb pointer
    void set_bb(BULLETIN_BOARD *bb) { bb_ptr = bb; }

    void ignore();

    // Test success of an outcome.
    logical ok() const { return quant == API_SUCCESS; }

    // Print out details of an outcome.
    void debug(FILE *fp = debug_file_ptr) const;

    logical encountered_errors() const;

    void get_error_info_list(error_info_list &errors) const;

    // This method is for internal use only
    // Sets the error_info_list stored in this outcome
    void set_error_info_list(error_info_list const &errors);

    // This method is for internal use only
    void add_problems_list(problems_list *problems);

    // This method is for internal use only
    // Add a non-fatal problem (error_info) to the outcome's error_info_list
    void add_problem(error_info *problem);
};

// The SPAparameter is an automatic object, presumably with an interesting constructor
// and destructor, which is declared outside the try block,
// but inside a pair of brackets.  Such an object can setup and tear down
// stuff as it wishes.
#define __ERROR_BEGIN_INTERNAL__(object)          \
    outcome result(0); /* Was in API_SYS_BEGIN */ \
    err_mess_type error_no = 0;                   \
    problems_list_prop problems_prop;             \
    EXCEPTION_BEGIN                               \
    object;                                       \
    EXCEPTION_TRY

#define __ERROR_END_INTERNAL__                                                           \
    EXCEPTION_CATCH_FALSE                                                                \
    /* note, this error_no is the one declared in the EXCEPTION_BEGIN macro.             \
     * Not the one from __ERROR_BEGIN_INTERNAL__.  For compatibility, we will capture it \
     * from the result below                                                             \
     */                                                                                  \
    result = outcome(error_no, ERROR_INFO_PTR);                                          \
    EXCEPTION_END_NO_RESIGNAL                                                            \
    error_no = result.error_number();                                                    \
    error_no = error_no;

// Start and close off bulletin boards, not used directly, but in the following
// system macros. (The delete function doesn't seem to be used anywhere.)

DECL_KERN void api_bb_begin(logical linear = TRUE);

DECL_KERN void api_bb_end(
    outcome &result,
    logical linear = TRUE,
    logical delete_stacked_bb = FALSE);

DECL_KERN void api_bb_delete();

// STI swa (17Mar98) -- made a new API_SYS_BEGIN that initializes
// a global error_info object pointer, and uses this pointer
// when making an outcome following an error longjmp
//
// #define API_SYS_BEGIN \
  //  outcome result; \
  //  /* api_error_begin(); */ \
  //  __ERROR_BEGIN_INTERNAL__ \
  //  if (error_no != 0) { \
  //      result = outcome( error_no ); \
  //  } else {

// Note:  We had some brief concern about using automatic objects in these
//        macros because that would not be compatible with C.  However
//        Earlier versions of __ERROR_BEGIN_INTERNAL__/END used API_MACRO_CHECKER,
//        which uses an automatic apiMacroChecker.  Thus, these
//        macros have not been C compatible for some time.  We will
//        encourage people to do sensitive code in C++, but if
//        push ever comes to shove, we can use techniques similar to
//        what is done in except.cxx and except.h to bridge the gap.
//        (That code does not appear to be used anywhere either so ...)

// Manipulate a signal trap for user interrupt, to clean up bulletin
// board, etc.

// Macros for beginning and ending API routines.  All calls to Acis
// routines need to be bracketed by operations to set up the error
// system, and then to reset it to its prior state.  Anything which
// affects the model also require to initialise and close down the
// bulletin board.  We provide macros to simplify the source code:

//      API_BEGIN, API_END for ordinary api routines which return
//                      a bulletin board
//      API_SYS_BEGIN, API_SYS_END for api system routines, generally
//                      those which manipulate bulletin boards and
//                      rollback, and which therefore do not return
//                      bulletin boards.
//      API_NOP_BEGIN, API_NOP_END for api routines that wish to use
//                      bulletin board facilities but also wish to
//                      leave the bulletin board in the same state
//                      it was in before API_NOP_BEGIN.
//      API_TRIAL_BEGIN, API_TRIAL_END for code that uses a heursitic
//                      that may fail.  The coder may check the result
//                      to see if a more robust algorithm is needed.
//                      In case of failure, the model will be rolled
//                      back as if the _NOP_ macros had been used.
//                      In the case of success, the model will be retained
//                      as if the unadorned api macros had been used.

// Set up the bulletin board, then the error system and the application
// callback flag.

// STI dgh - add exception checking at API level

// STI aed: Add error_begin/end so user interrupt handler does not get
//          called until after the Bulletin Board is closed.

#define API_SYS_BEGIN_INTERNAL(object) \
    set_global_error_info();           \
    __ERROR_BEGIN_INTERNAL__(object)

DECL_KERN void nested_state_check();

#define API_SYS_END_INTERNAL \
    __ERROR_END_INTERNAL__

#define API_SYS_BEGIN \
    API_SYS_BEGIN_INTERNAL(nested_state_check())

#define API_SYS_END      \
    API_SYS_END_INTERNAL \
    problems_prop.process_result(result, PROBLEMS_LIST_PROP_ONLY, FALSE);

// STI jmb: Use api_bb_save class so the appropriate balancing of code
//          occurs regardless of how the api block is exited.
/*
// tbrv
*/
class api_bb_save : public ACIS_OBJECT
{
public:
    enum e_bb_type
    {
        normal,
        nop,
        trial
    };

private:
    e_bb_type bb_type;   // What style of bb handling do we want.
    outcome &result;     // The result will be put here before api_bb_end
    logical was_logging; // The trial and nop cases depend on our
                         // bulletin board logging, so force it on.
public:
    api_bb_save(outcome &r, e_bb_type bt) : bb_type(bt),
                                            result(r),
                                            was_logging(logging)
    {
        if (bb_type != normal)
            set_logging(TRUE);
        api_bb_begin(bb_type == normal);
    }
    ~api_bb_save()
    {
        api_bb_end(result, bb_type != nop, !was_logging);
        set_logging(was_logging);
    }

    // Assignment operator could not be generated warning
    // is taken care of by adding a private one.  PRS
private:
    api_bb_save &operator=(const api_bb_save &) { return *(api_bb_save *)NULL_REF; }
};

#define API_BEGIN                                                                        \
    API_SYS_BEGIN_INTERNAL(api_bb_save make_bulletin_board(result, api_bb_save::normal)) \
    ACIS_EXCEPTION_CHECK("API");                                                         \
    logical call_update_from_bb = TRUE;

// Call the application callback if appropriate, then restore the error
// system, and then sort out the bulletin board
#define API_END                             \
    if (result.ok() && call_update_from_bb) \
        update_from_bb();                   \
    API_SYS_END_INTERNAL                    \
    problems_prop.process_result(result, PROBLEMS_LIST_PROP_ONLY, FALSE);

// STI aed: end

// Macros for bracketing an api function which is to be treated as
// a no operation as far as the data structure is concerned.
// API_NOP_END restores the state as of the corresponding API_NOP_BEGIN
// including any currently-open bulletin board.

// Set up a new bulletin board and then the error system

// STI aed: Add error_begin/end so user interrupt handler does not get
//          called until after the Bulletin Board is closed.

#define API_NOP_BEGIN \
    API_SYS_BEGIN_INTERNAL(api_bb_save make_bulletin_board(result, api_bb_save::nop));

// Restore the error system and then remove the bulletin board and
// restore the previous one.
#define API_NOP_END      \
    API_SYS_END_INTERNAL \
    problems_prop.process_result(result, PROBLEMS_LIST_PROP_IGNORE, FALSE);

// STI aed: end

// Macros for defining a block of code which should be executed as
// a particular version of Acis.
#define API_VERS_BEGIN(opts) \
    API_BEGIN                \
    ALGORITHMIC_VERSION_BLOCK(opts ? &opts->get_version() : NULL);

#define API_SYS_VERS_BEGIN(opts) \
    API_SYS_BEGIN                \
    ALGORITHMIC_VERSION_BLOCK(opts ? &opts->get_version() : NULL);

#define API_NOP_VERS_BEGIN(opts) \
    API_NOP_BEGIN                \
    ALGORITHMIC_VERSION_BLOCK(opts ? &opts->get_version() : NULL);

#define API_TRIAL_VERS_BEGIN(opts) \
    API_TRIAL_BEGIN                \
    ALGORITHMIC_VERSION_BLOCK(opts ? &opts->get_version() : NULL);

// STI jmb: begin
// Macros for bracketing an api function which is to be tested for success
// and treated as a no operation as far as the data structure is concerned
// if it fails.  If it succeeds, the bulletins are merged into the active bulletin
// board.  If if fails, the state is restored as of the corresponding API_TRIAL_BEGIN
// including any currently-open bulletin board.

// Set up a new bulletin board and then the error system

// STI aed: Add error_begin/end so user interrupt handler does not get
//          called until after the Bulletin Board is closed.

#define API_TRIAL_BEGIN                                                                 \
    API_SYS_BEGIN_INTERNAL(api_bb_save make_bulletin_board(result, api_bb_save::trial)) \
    ACIS_EXCEPTION_CHECK("API");                                                        \
    logical call_update_from_bb = TRUE;

// Restore the error system and then remove the bulletin board and
// restore the previous one.

#define API_TRIAL_END                       \
    if (result.ok() && call_update_from_bb) \
        update_from_bb();                   \
    API_SYS_END_INTERNAL                    \
    problems_prop.process_result(result, PROBLEMS_LIST_PROP_OR_IGNORE, FALSE);

// STI jmb: end

// STI jmb:  Macros for the replacement of journalling.
#define API_DEBUG_ENTER(name) \
    DEBUG_LEVEL(DEBUG_CALLS)  \
    fprintf(debug_file_ptr, "calling %s\n", name);

#define API_DEBUG_EXIT(name)                          \
    DEBUG_LEVEL(DEBUG_FLOW)                           \
    fprintf(debug_file_ptr, "leaving %s: %s\n", name, \
            find_err_ident(result.error_number()));
// STI jmb: end
#endif
```





## Global Options

### Option: New cover

决定是否使用新的算法。

|   类型    |       值        |  默认  |  字符串名   |
| :-------: | :-------------: | :----: | :---------: |
| `logical` | `FALSE`, `TRUE` | `TRUE` | `new_cover` |





## [Covering Interface](https://doc.spatial.com/get_doc_page/articles/c/o/v/Covering_Interface_2527.html#Scheme_Extensions)

Covering API 函数分为两类：需要平面输入和不需要平面输入。



### 平面输入

如下每个 API 函数尝试计算插值指定 face 边界的平面曲面。

* `api_cover_planar_edges`：通过覆盖一组封闭的平面边，创建由一个或多个平面（可能带有内孔）组成的 sheet 实体。相邻边应在其端点相交，但不需要共享顶点。边缘应该是没有更高级别拓扑的顶级实体。
* `api_cover_planar_wires`：通过覆盖一组封闭的平面 wire，创建由一个或多个平面（可能带有内孔）组成的 sheet 实体。Wire 可以有分支，可以重叠。
* `api_cover_wire_loops`：通过覆盖一组封闭的平面 wire，创建由单个平面（可能带有内孔）组成的 sheet 实体。第一根 wire 的边缘构成新面的外围环。任何额外的 wire 都会变成面的洞环路。Wire 不能有分支，并且不相交。



### 非平面输入

如下每个 API 函数使用 `cover_options` 参数传递容差，可以产生容差边和顶点。

* `api_cover_circuits`：输入 `ENTITY_LIST` 的数组，其中每个 `ENTITY_LIST` 定义一个边的封闭回路。所有边必须属于同一实体。边必须使用相同的顶点连接，而不是具有相同位置的顶点连接。
* `api_cover_sheet`：搜索自由边（即只有一个 coedge 的 edge）的闭合回路并尝试覆盖每个这样的回路。
* `api_cover_wire`：通过覆盖单个封闭 wire 创建开放 solid 实体。
* `api_cover_wires`：通过覆盖包含一个或多个 wire 的 wire 实体创建开放 solid 实体。



### 总结 API

由两张背靠背的单侧 face 组成一个 lamina，它在 ACIS 中是无效表示，但是可以作为 solid 构造的中间步骤。

|           函数           | 平面几何 | 非平面几何 | 用户指定几何 | 用户指定容差 | 双侧面 | 单侧面 | 生成 Lamina |
| :----------------------: | :------: | :--------: | :----------: | :----------: | :----: | :----: | :---------: |
| `api_cover_planar_edges` |   yes    |     no     |      no      |      no      |  yes   |   no   |     no      |
| `api_cover_planar_wires` |   yes    |     no     |      no      |      no      |  yes   |   no   |     no      |
|  `api_cover_wire_loops`  |   yes    |     no     |      no      |      no      |  yes   |   no   |     no      |
|   `api_cover_circuits`   |   yes    |    yes     |     yes      |     yes      |  yes   |  yes   |     yes     |
|    `api_cover_sheet`     |   yes    |    yes     |     yes      |     yes      |   no   |  yes   |     yes     |
|     `api_cover_wire`     |   yes    |    yes     |     yes      |     yes      |   no   |  yes   |     no      |
|    `api_cover_wires`     |   yes    |    yes     |     yes      |     yes      |   no   |  yes   |     no      |







