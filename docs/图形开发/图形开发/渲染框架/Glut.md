# OpenGL

## 介绍

OpenGL 本身并不是一个 API，它仅仅是一个由 [Khronos组织](http://www.khronos.org/) 制定并维护的规范(Specification)。OpenGL 规范严格规定了每个函数该如何执行，以及它们的输出值。至于内部具体每个函数是如何实现的，将由 OpenGL 库的开发者自行决定。因为 OpenGL 规范并没有规定实现的细节，具体的 OpenGL 库允许使用不同的实现，只要其功能和结果与规范相匹配。

> 由于 OpenGL 的大多数实现都是由显卡厂商编写的，当产生一个 bug 时通常可以通过升级显卡驱动来解决。这些驱动会包括你的显卡能支持的最新版本的 OpenGL，这也是为什么总是建议你偶尔更新一下显卡驱动。



### 安装编译

需要将 `glut32.dll` 文件放到系统目录 `C:\Windows\SysWOW64` 中，将 `glut32.lib` 和 `glut.h` 放在项目的 lib 和 include 目录中，然后在 VS 项目的“目录”中添加包含目录和库目录；在“链接器-输入”中添加 `opengl32.lib;glu32.lib;glut32.lib`；在“链接器-命令行”的其它选项中添加 `/SAFESEH:NO`，防止编译出错。



然后我们可以编译运行一个简单程序

```cpp
#include <glut.h>

void mydisplay()
{
	glClear(GL_COLOR_BUFFER_BIT);	// 清空缓存区
	glBegin(GL_POLYGON);			// 绘制图元
	glVertex2d(-0.5, -0.5);
	glVertex2d(-0.5, 0.5);
	glVertex2d(0.5, 0.5);
	glVertex2d(0.5, -0.5);
	glEnd();						// 结束顶点序列
	glFlush();						// 刷新缓冲
}

int main(int argc, char* argv[])
{
	glutInit(&argc, argv);							// 初始化函数
	glutInitDisplayMode(GLUT_SINGLE | GLUT_RGB);	// 输出模式
	glutInitWindowSize(500, 500);					// 定义窗口大小
	glutInitWindowPosition(0, 0);					// 窗口位置
	glutCreateWindow("Test");						// 创建窗口
	glutDisplayFunc(mydisplay);						// 回调函数
	glutMainLoop();									// 进入事件循环
}
```



注意到生成的除了 OpenGL 的窗口以外还包括控制台，如果想去掉控制台，可以在主程序头部添加

```cpp
#pragma comment(linker, "/subsystem:\"windows\" /entry:\"mainCRTStartup\"")
```

然后在“项目属性-链接器-所有选项”中将子系统更改为窗口即可。



### 函数名格式

前缀 gl 表示属于 GL 库的函数，glu 和 glut 前缀分别表示属于 GLU 和 GLUT 库的函数。例如

```cpp
glVertex3f(x, y, z);
glVertex3fv(p);
```

分解为

* gl：属于 GL 库
* Vertex：函数功能
* 3：参数个数
* f：浮点型
* v：向量类型，其实就是指针
* p：p 为指向 float 的指针

常用的变量类型后缀有

* b - byte
* ub - unsigned byte
* s - short
* us - unsigned short
* i - int
* ui - unsigned int
* f - float
* d - double



### 错误处理

当 OpenGL 发现错误时，就在内部记录一个出错编码，造成错误的子程序会被忽略。但是 OpenGL 每次只记录一个出错编码，一旦出现一个出错编码，在程序明确查询 OpenGL 出错状态之前不会再记录其它出错编码。

```cpp
GLenum glGetError(void);	// 获得出错编码
```

此函数返回出错编码并清空出错标记，返回值通常有

* `GL_NO_ERROR` - 没有错误
* `GL_INVALID_ENUM` - GLenum 的参数超出范围
* `GL_INVALID_VALUE` - 数值参数超出范围
* `GL_INVALID_OPERATION` - 有一个操作非法
* `GL_STACK_OVERFLOW` - 栈向上溢出
* `GL_STACK_UNDERFLOR` - 栈向下溢出
* `GL_OUT_OF_MEMORY` - 没有足够的内存执行命令

这些编码本身没有什么特殊信息，我们需要用 GLU 库中的函数为每个 GLU 和 GL 错误返回描述性字符串

```cpp
#include <stdio.h>

GLenum code = glGetError();
const GLubyte *string = gluErrorString(code);
fprintf_s(stderr, "OpenGL error: %s\n", string);
```

注意返回字符串不能由我们重新分配或修改，因而添加了 const 修饰。



## 控制窗口

### 初始化

#### glutInit

argc 和 argv 都是只读的 main 函数传入变量的指针

```cpp
void glutInit(int *argc, char **argv);
```



#### glutInitWindowPosition

定义窗口显示的位置

```cpp
void glutInitWindowPosition(int x, int y);
```

参数表示左边距离和顶部距离。



#### glutInitWindowSize

定义窗口宽度

```cpp
void glutInitWindowSize(int width, int height);
```



#### glutInitDisplayMode

定义显示模式，不同模式通过按位与 `|` 同时使用

```cpp
void glutInitDisplayMode(unsigned int mode)
```

模式用来指定颜色模型和缓冲数量以及类型。预定义常量如下：

* `GLUT_RGBA / GLUT_RGB` 指定颜色模式，RGBA 是默认选项
* `GLUT_INDEX` 颜色索引模式
* `GLUT_SINGLE` 单缓冲窗体
* `GLUT_DOUBLE` 双缓冲窗体，需要支持平滑运动
* `GLUT_ACCUM` 堆缓冲
* `GLUT_STENCIL` 模板缓冲
* `GLUT_DEPTH` 深度缓冲



#### glutCreateWindow

创建窗口，设置窗口标题，返回窗口 ID

```cpp
int glutCreateWindow(char *title);
```



#### glutSetCursor

设置屏幕光标的形状

```cpp
void glutSetCursor(int cursor);
```

可选参数

* `GLUT_CURSOR_UP_DOWN`
* `GLUT_CURSOR_CYCLE`

还有更多可选参数，可查询头文件。



### 创造窗体

利用 GLUT 可以将主窗体切分为不同子窗体区域，每个子窗体有自己的回调函数和相关参数。通常使用多窗口是为了展示图形在多个视角的显示效果。



#### 子窗体

使用 glutCreateSubWindow 函数为当前窗体创建子窗体

```cpp
int glutCreateSubWindow(
    int parentWindow, 		// 父窗体 ID
    int x, int y, 			// 相对父窗体左上角的坐标
    int width, int height	// 子窗体大小
);
```

父窗体 ID 是创建窗体时的返回值，例如

```cpp
mainWindow = glutCreateWindow("Main");
subWindow = glutCreateSubWindow(mainWindow, 10,10,100,100);
```

子窗体也可以创建新的子窗体，这种嵌套关系是任意的。当窗体创建后，新的窗体就成为焦点窗体，在此之后**除了空闲函数以外**的所有回调函数将注册到此窗体中，因为空闲函数是唯一的。



当不需要子窗口时，调用

```cpp
void glutDestroyWindow(int windowIdentifier)
```

销毁指定 ID 的子窗口。



#### 重整窗体

当主窗口尺寸发生改变，这时候需要调整窗口的比例。我们只需要为主窗口注册一个调整窗口的函数即可。



由于任何窗口操作都是针对**当前窗口**进行，我们首先需要获得和修改窗口

```cpp
int glutGetWindow();						// 获得当前窗口 ID
void glutSetWindow(int windowIdentifier);	// 设置当前窗口
```

然后可以调整窗口的位置和尺寸

```cpp
void glutPositionWindow(int x, int y);
void glutReshapeWindow(int width, int height);
void glutFullScreen(void);
```

这里窗口位置的设定是相对于主窗口左上角的坐标。



## 输出图元

### 绘图流程

通常指定绘图操作时，需要注册渲染函数

```cpp
void glutDisplayFunc(void (*funcName)(void));
```

其中的参数是我们自定义的渲染函数。当窗口需要重新渲染时，就会调用此渲染函数执行绘图操作。



使用 glClearColor 函数清空背景

```cpp
glClearColor(red, green, blue, alpha);
```



渲染函数的内部结构一般为

```cpp
void mydisplay()
{
    glClear(GL_COLOR_BUFFER_BIT);
    
    // 填充背景色
    glClearColor(1, 1, 1, 0);
    
    // 绘图内容
    glBegin(Geometry type);
    // 图元顶点
    glEnd();
    
    // 强制刷新
    glFlush();
}
```

其中 glClear 函数原型为

```cpp
void glClear(GLbitfield mask);
```

它接收一个描述缓冲的宏，决定将清空哪些缓冲。例如

* `GL_COLOR_BUFFER_BIT` 清空颜色缓冲
* `GL_DEPTH_BUFFER_BIT` 清空深度缓冲
* `GL_STENCIL_BUFFER_BIT` 清空模板缓冲

如果想清空多个缓冲，使用按位或运算连接即可。函数 glFlush 强制刷新缓冲，实现绘图。



### 重绘函数

GLUT 提供重绘消息函数来通知窗口调用绑定的渲染函数来重绘窗口

```cpp
void glutPostRedisplay(void);
```

如果我们将渲染函数绑定到空闲消息中，这会导致尽管图像没有发生改变，渲染函数却被频繁调用，从而造成大量内存浪费。使用重绘函数的目的就是为了告知窗口什么时候进行重绘操作。



### 顶点

使用 glVertex 相关的函数可以绘制二维和三维点，例如

```cpp
// 在指定位置绘制二维点
void glVertex2f(
    GLfloat x, 
    GLfloat y
);

// 在指定位置绘制三维点
void glVertex3f(
    GLfloat x, 
    GLfloat y, 
    GLfloat z
);
```

也可以通过传入指针绘制

```cpp
// 传入指针绘制二维点
void glVertex2fv(
    const GLfloat *v
);

// 传入指针绘制三维点
void glVertex3fv(
    const GLfloat *v
);
```

这种方法的好处是便于以数组形式传入点。



### 绘图模式

使用 glBegin 开启绘图时需要指定具体的绘图模式，OpenGL 提供了基本的绘图形状

```cpp
#define GL_POINTS			// 只是绘制点
#define GL_LINES			// 两两配对点连线
#define GL_LINE_LOOP		// 循环连接所有点
#define GL_LINE_STRIP		// 依次连接点
#define GL_TRIANGLES		// 三个点配对绘制三角形
#define GL_TRIANGLE_STRIP	// 依次选三个点绘制三角形
#define GL_TRIANGLE_FAN		// 以第一个顶点为公共顶点绘制三角形
#define GL_QUADS			// 四个点配对绘制四边形
#define GL_QUAD_STRIP		// 依次选四个点绘制四边形
#define GL_POLYGON			// 绘制多边形
```

![](图形开发/渲染框架/Glut.assets/image-20221110200041156.png)

![](图形开发/渲染框架/Glut.assets/image-20221110200052218.png)

OpenGL 只能绘制满足如下条件的多边形

* 除了顶点以外，边不相交的简单多边形
* 凸多边形
* 平面多边形

由于三角形自动满足上面条件，因此应当尽量绘制三角形。



### 双缓冲

通过 glutInitDisplayMode 函数，设置绘图模式为 `GLUT_SINGLE` 即可开启单缓冲绘图，其绘图流程如上所示。但是，当我们需要频繁改变窗口内容时，使用单缓冲可能导致出现图像闪烁的问题。这时候就应该使用 `GLUT_DOUBLE` 双缓冲绘图，流程如下

```cpp
void mydisplay()
{
    glClear(GL_COLOR_BUFFER_BIT);
    glBegin(Geometry type);
    // 绘图内容
    glEnd();
    glutSwapBuffers();
}
```

区别仅仅在于最后调用的函数是交换缓冲的函数，将当前绘图缓冲交换到窗口绘图缓冲中立即显示图像。



### $z$ 缓冲区算法

$z$ 缓冲区算法用于储存几何体的深度信息，目的是根据点的深度消除绘制三维图形时被遮挡的部分，从而正确地显示图像，同时提高绘制效率。首先在 glutInitDisplayMode 中开启 `GLUT_DEPTH` 模式，然后调用

```cpp
glEnable(GL_DEPTH_TEST);
```

激活隐藏面消除算法，在渲染函数中清空深度缓冲

```cpp
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```



默认情况下，深度值范围是 0 到 1 之间。可以修改深度值

```cpp
void glDepthRange(GLclampd zNear, GLclampd zFar);
```

两个参数可选 0 到 1 之间的任意值。



进行深度测试时，首先初始化

```cpp
glClearDepth(maxDepth);			// maxDepth 可选 0 到 1
glClear(GL_DEPTH_BUFFER_BIT);	// 清空深度缓冲区
```

默认情况下，深度测试会将新像素的 $z$ 值与缓冲区中的深度比较，如果更小就更新缓冲区。我们可以调整比较方式

```cpp
glDepthFunc(method);
```

其中 method 默认为 `GL_LESS`，其余参数直接查阅头文件。



### 裁剪方法

OpenGL 中处理视景体的 6 个裁剪平面以外，还可以最多指定 6 个其它裁剪平面，进一步限制视景体。



我们通过如下函数指定裁剪平面

```cpp
void glClipPlane(
    GLenum plane, 
    const GLdouble *equation
);
```

其中 plane 是裁剪平面的编号，形如 `GL_CLIP_PLANEi`，i 是具体编号，只能取 0 到 5 之间；equation 是平面方程的 4 个参数的数组。



指定裁剪平面后需要手动启用/禁用对应编号的裁剪平面

```cpp
void glEnable(GLenum cap);
void glDisable(GLenum cap);
```

参数就是裁剪平面的变换。



不同系统中支持的裁剪平面数可能不同，使用

```cpp
glGetIntegerv(GL_MAX_CLIP_PLANES);
```

获得支持的其它裁剪平面的最大数目。



### 显示列表

把要绘制的对象描述为一个命名的语句序列并储存更加高效。通过

```cpp
glNewList(listID, listMode);
// ...
glEndList();
```

创建 ID 为 listID，模式 listMode 可选 `GL_COMPILE` 或 `GL_COMPILE_AND_EXECUTE`，前者只会储存，后者在存储的同时立即执行表中的命令。



我们可以创建任意多显示表，并调用

```cpp
glCallList(listID);
```

执行显示表。例如

```cpp
const double TWO_PI = 6.2831853;

// 注册显示表
int regHex = glGenLists(1);
glNewList(regHex, GL_COMPILE);

// 显示表内容
glBegin(GL_POLYGON);
for (int i = 0; i < 6; i++)
{
	double theta = TWO_PI * i / 6.0;
	double x = 0.8 * cos(theta);
	double y = 0.8 * sin(theta);
	glVertex3f(x, y, 0);
}
glEnd();

// 结束命令
glEndList();

// 调用显示表
glCallList(regHex);
```

**需要注意，在 MFC 中，所有与绘图有关的代码最好放在已经设置当前 RC 的范围内，防止出现问题。**



一个显示表可以嵌套在另一个显示表内。但如果一个显示表使用了另一个显示表的 ID，它会覆盖前者。为了防止错误重用导致显示表丢失，可以让程序自动产生 ID

```cpp
listID = glGenLists(1);
```

它产生一个未使用的正整数标识 ID 。传入参数表示保留连续的整数，例如传入 6 会保留连续 6 个未使用的正整数并返回第一个。



当然，我们可以查询一个正整数是否已经使用

```cpp
glIsList(listID);
```

返回值为 `GL_TRUE` 或 `GL_FALSE` 。



我们可以执行多个显示表

```cpp
// 设置初始位置
void glListBase(GLuint base);

// 调用多个显示表
void glCallLists(
    GLsizei n, 				// 要执行的显示表数量
    GLenum type, 			// 数据类型：GL_BYTE, GL_INT, GL_FLOAT
    const GLvoid *lists		// 显示表数组
);
```

调用多个显示表时，从 base(默认为 0) 指定的显示表开始依次调用 n 个表。



最后，如果不需要显示表，可以删除连续的一组

```cpp
glDeleteLists(startID, nLists);
```

从 startID 开始依次删除 nLists 个显示表。



### 三维对象

#### 多面体函数

OpenGL 提供了预置的函数来绘制多面体

```cpp
void glutWireCube(double size);   // 空心立方体
void glutSolidCube(double size);  // 实心立方体
void glutWireDodecahedron(void);  // 空心12面体
void glutSolidDodecahedron(void); // 实心12面体
void glutWireOctahedron(void);    // 空心8面体
void glutSolidOctahedron(void);   // 实心8面体
void glutWireTetrahedron(void);   // 空心四面体
void glutSolidTetrahedron(void);  // 实心四面体
void glutWireIcosahedron(void);   // 空心18面体
void glutSolidIcosahedron(void);  // 实心18面体
```



#### 曲面函数

还有一些用于绘制曲面的函数

```cpp
void glutWireSphere(double radius, GLint slices, GLint stacks);                        // 空心圆
void glutSolidSphere(double radius, GLint slices, GLint stacks);                       // 实心圆
void glutWireCone(double base, double height, GLint slices, GLint stacks);             // 空心圆锥体
void glutSolidCone(double base, double height, GLint slices, GLint stacks);            // 实心圆锥体
void glutWireTorus(double innerRadius, double outerRadius, GLint sides, GLint rings);  // 空心环形
void glutSolidTorus(double innerRadius, double outerRadius, GLint sides, GLint rings); // 实心环形

/*
 * Teapot rendering functions, found in fg_teapot.c
 * NB: front facing polygons have clockwise winding, not counter clockwise
 */
void glutWireTeapot(double size);  // 空心茶壶
void glutSolidTeapot(double size); // 实心茶壶
```



#### 二次曲面

如果要自己生成二次曲面，首先要创建二次曲面对象

```cpp
GLUquadric* gluNewQuadric (void);
```

然后设定曲面绘制模式

```cpp
void APIENTRY gluQuadricDrawStyle(
    GLUquadric *quadObject, // 对象指针
    GLenum drawStyle		// 指定模式
);
```

可以指定

* `GLU_LINE` 每对顶点之间显示直线段，按线框图显示
* `GLU_POINT` 只显示顶点
* `GLU_FILL` 填充绘制
* `GLU_SILHOUETTE` 另一种风格的线框图



通过如下函数绘制圆柱

```cpp
void APIENTRY gluCylinder(
    GLUquadric *qobj, 		// 对象指针
    GLdouble baseRadius, 	// z = 0 处的半径
    GLdouble topRadius, 	// z = height 处的半径
    GLdouble height, 		// 高度
    GLint slices, 			// z 轴周围的细分数
    GLint stacks			// z 轴上的细分数
);
```

例如我们绘制柱形，底面是 6 边形，高度细分 4 份

```cpp
GLUquadricObj* cylinder = gluNewQuadric();
gluQuadricDrawStyle(cylinder, GLU_LINE);
gluCylinder(cylinder, 0.6, 0.6, 1.5, 6, 4);
```



还有其它可绘制的图像

```cpp
// 球面
void gluSphere(
    GLUquadric *qobj, 	// 对象指针
    GLdouble radius, 	// 半径
    GLint slices, 		// z 轴周围的细分数
    GLint stacks		// z 轴上的细分数
);

// 圆环
void gluDisk(
    GLUquadric *qobj, 		// 对象指针
    GLdouble innerRadius, 	// 内径
    GLdouble outerRadius, 	// 外径
    GLint slices, 			// z 轴周围的细分数
    GLint loops				// 同心环数
);
```



可定义任何二次曲面的前后方向

```cpp
void gluQuadricOrientation(
    GLUquadric *quadObject, // 对象指针
    GLenum orientation		// 方向
);
```

可以指定 `GLU_OUTSIDE` 或 `GLU_INSIDE`，从而确定曲面法向量方向，默认时 `GLU_OUTSIZE` 。



另一种选择是表面法向量生成

```cpp
void gluQuadricNormals(
    GLUquadric *quadObject, // 对象指针
    GLenum normals			// 法向量生成方式
);
```

默认为 `GLU_NONE`，表示不生成表面法向量，而一般情况下不对二次曲面应用光照条件。对于平表面的均匀颜色，使用 `GLU_FLAT`，这为每一多边形面片生成一个表面法向量。当提供其它光照条件时，使用 `GLU_SMOOTH`，为每一顶点位置生成一个法向量。



可以回收分配给二次曲面的储存器并将其消除

```cpp
gluDeleteQuadric(quadricName);
```



可以指定一个当生成曲面错误时调用的函数

```cpp
void gluQuadricCallback(
    GLUquadric *qobj, 		// 对象指针
    GLenum which, 			// 设为 GLU_ERROR
    void (CALLBACK* fn)()	// 回调函数
);
```



### 样条函数

#### Bezier 样条曲线

我们需要激活绘制曲线的子程序

```cpp
void glMap1f(
    GLenum target, 			// 绘制目标
    GLfloat u1, 			// 最小参数
    GLfloat u2, 			// 最大参数
    GLint stride, 			// 跳跃值
    GLint order, 			// 点的数量
    const GLfloat *points	// 控制点数组
);
```

如果使用三维点描述，激活一个三次 Bezier 曲线的方法为

```cpp
glMap1f(GL_MAP1_VERTEX_3, 0, 1, 3, 4, *ctrlPts);
glEnable(GL_MAP1_VERTEX_3);
```

如果要用四维齐次坐标表示，则使用 `GL_MAP1_VERTEX_4`，并将 stride 改为 4 即可。



设定参数后，需要计算沿路径的位置

```cpp
void glEvalCoord1f(GLfloat u);
```

传入曲线参数值即可绘制曲线对应参数值的点。



完整的绘图程序如下

```cpp
// 控制点数组
float ctrlPts[4][3] = {
	{0, 0, 0}, {1, 1, 0}, {2, 1, 0}, {3, 0, 0}
};

// 激活子程序
glMap1f(GL_MAP1_VERTEX_3, 0, 1, 3, 4, *ctrlPts);
glEnable(GL_MAP1_VERTEX_3);

glColor3f(0, 0, 1);
glPointSize(1);

// 绘制 Bezier 曲线
glBegin(GL_LINE_STRIP);
for (int k = 0; k <= 50; k++)
{
	glEvalCoord1f(float(k) / 50);
}
glEnd();

glColor3f(1, 0, 0);
glPointSize(5);

// 绘制控制点
glBegin(GL_POINTS);
for (int k = 0; k < 4; k++)
{
	glVertex3fv(&ctrlPts[k][0]);
}
glEnd();
```



由于我们使用的是均匀间隔的参数，可以直接调用函数产生这类参数

```cpp
void glMapGrid1f(
    GLint un, 		// 分割份数
    GLfloat u1, 	// 起始参数
    GLfloat u2		// 最终参数
);

void glEvalMesh1(
    GLenum mode, 	// 绘制模式
    GLint i1, 		// 起始点的索引
    GLint i2		// 终止点的索引
);
```

因此上面绘制过程可以替换为

```cpp
glMapGrid1f(50, 0, 1);			// (0,1) 划分成 50 份
glEvalMesh1(GL_LINE, 0, 50);	// 从 0 到 50 绘制连线
```

注意划分成 50 份，意味着有 51 个点，索引范围就是 0 到 50；另外，如果绘制模式为 `GL_POINT`，就会只绘制参数点。



除了显示 Bezier 曲线，还可以用 glMap1 函数指定其它类数据的值。使用 `GL_MAP1_COLOR_4` 时，数组 ctrlPts 用于指定 RGBA 颜色列表，然后可以为应用生成一组线性插值的颜色，并且这些颜色不会改变当前的颜色状态

```cpp
// 控制点数组
float ctrlPts[4][3] = {
	{0, 0, 0}, {1, 1, 0}, {2, 1, 0}, {3, 0, 0}
};

// 激活子程序
glMap1f(GL_MAP1_VERTEX_3, 0, 1, 3, 4, *ctrlPts);
glEnable(GL_MAP1_VERTEX_3);

// 颜色数组
float colorPts[4][4] = {
	{1, 0, 0, 1}, {0, 1, 0, 1}, {0, 0, 1, 1}, {1, 1, 1, 1}
};

// 激活颜色插值
glMap1f(GL_MAP1_COLOR_4, 0, 1, 4, 4, *colorPts);
glEnable(GL_MAP1_COLOR_4);

glMapGrid1f(50, 0, 1);
glEvalMesh1(GL_LINE, 0, 50);
```

同理可以用 `GL_MAP1_NORMAL` 可以指定一组三维曲面法向量。



多个 glMap1 函数可以同时激活，它们对 glEvalCoord1 或 glMapGrid1 以及 glEvalMesh1 指定数据信息。但是同一时刻只能存在一个曲面纹理生成器，并且不能同时激活 `GL_MAP1_VERTEX_3` 和 `GL_MAP1_VERTEX_4` 。



#### Bezier 样条曲面

类似地需要激活子程序

```cpp
void glMap2f(
    GLenum target, 
    GLfloat u1, GLfloat u2, GLint ustride, GLint uorder, 
    GLfloat v1, GLfloat v2, GLint vstride, GLint vorder, 
    const GLfloat *points
);
```

使用方法类似于前，只不过要指定两个参数值的信息。



指定参数信息计算对应曲面上的值

```cpp
void glEvalCoord2f (GLfloat u, GLfloat v);
```

并且也有指定均匀间隔参数的函数

```cpp
void glMapGrid2f(
    GLint un, GLfloat u1, GLfloat u2, 
    GLint vn, GLfloat v1, GLfloat v2
);

void glEvalMesh2(
    GLenum mode, 
    GLint i1, GLint i2, 
    GLint j1, GLint j2
);
```

这里模式除了 `GL_POINT` 和 `GL_LINE` 外，还可选 `GL_FILL` 来填充曲面。



#### B 样条曲线

使用 glu 库中的 NURBS 函数生成 B 样条，基本调用序列如下

```cpp
GLUnurbsObj* curveName = gluNewNurbsRenderer();
gluBeginCurve(curveName);
gluNurbsCurve(curveName, nknots, *kontVector, stride, *ctrlPts, degParam, GL_MAP1_VERTEX_3);
gluEndCurve(curveName);
```

其中绘制函数

```cpp
void gluNurbsCurve(
    GLUnurbs *nobj, 	// 曲线对象
    GLint nknots, 		// 节点数
    GLfloat *knot, 		// 节点向量
    GLint stride, 		// 跳跃值
    GLfloat *ctlarray, 	// 控制点数组
    GLint order, 		// 多项式阶数：次数 + 1
    GLenum type			// 控制点类型
);
```

只需指定 nknots 和 order，则有 nknots-order 个控制点。要建立一个有理 B 样条曲线，需要用 `GL_MAP1_VERTEX_4` 指定齐次坐标控制点，结果中的齐次部分生成所要的齐次多项式形式。我们也可以用 gluNurbsCurve 指定一组颜色值、法向量或曲面纹理特性，只要修改最后一个参数类型就可以了。



要删除一个 B 样条，使用

```cpp
gluDeleteNurbsRenderer(curveName);
```



GLU 子程序将 B 样条曲线自动分成一些段并将它显示成折线。但我们也可以修改绘制器参数

```cpp
void gluNurbsProperty(
    GLUnurbs *nobj, 
    GLenum property, 
    GLfloat value 
);
```

此函数设定的大多特征都是曲面参数，我们将在下面介绍。



#### B 样条曲面

我们给出一个基本序列

```cpp
GLUnurbsObj* surfName = gluNewNurbsRenderer();
gluNurbsProperty(surfName, property1, value1);
gluNurbsProperty(surfName, property2, value2);
...

glBeginSurface(surfName);
gluNurbsSurface(surfName, nuKnots, uKnotVector, nvKnots, vKnotVector,
		uStride, vStride, &ctrlPts[0][0][0], 
        uDegParam, vDegParam, GL_MAP2_VERTEX_3);
gluEndSurface(surfName);
```

默认时，一个 B 样条曲面自动由 GLU 子程序显示成一组多边形填充区。我们可以指定不同的显示

```cpp
gluNurbsProperty(surfName, GLU_NURBS_MODE, GLU_NURBS_TESSELLATOR);	// 线框式
gluNurbsProperty(surfName, GLU_DISPLAY_MODE, GLU_OUTLINE_POLYGON);	// 三角网格曲面
```



要确定当前特征，需要调用询问函数

```cpp
void gluGetNurbsProperty(
    GLUnurbs *nobj, 
    GLenum property, 
    GLfloat *value
);
```

样条对象附带的各种事件通过如下函数处理

```cpp
void gluNurbsCallback(
    GLUnurbs *nobj, 
    GLenum which, 			// 赋予宏常量
    void (CALLBACK* fn)()	// 回调函数
);
```

例如 `GL_NURBS_BEGIN` 指出样条起始位置，`GL_NURBS_END` 指出结束位置。



#### 曲面修剪函数

可以指定一条分段折线对曲面进行修剪，基本流程如下

```cpp
gluBeginTrim(surfName);
gluPwlCurve(surfName, nPts, *curvePts, stride, GLU_MAP1_TRIM_2);
gluEndTrim(surfName);
```

当然可以用 gluNurbsCurve 构造 B 样条进行修剪，也可以两者混合修剪。任何指定的 GLU 修剪曲线必须没有自相交，并且必须是一条封闭曲线。



## 图元属性

属性是 OpenGL 状态的一部分，确定对象的外观，包括

* 颜色（点、线、多边形）
* 点的大小
* 线段宽度与实线/虚线模式
* 多边形模式
    * 前后面
    * 填充模式
    * 显示为实心多边形或只显示边界



### 颜色调和

有时我们需要将重叠对象的颜色或对象与背景颜色调和。首先要开启此特性

```cpp
glEnable(GL_BLEND);
```

如果没有开启，那么会直接用新的颜色替换旧的颜色。



OpenGL 中通过指定两组调和因子生成不同的颜色效果
$$
(S_rR_s+D_rR_d,S_gG_s+D_gG_d,S_bB_s+D_bB_d,S_aA_s+D_aA_d)
$$
其中 RGBA 源颜色为 $(R_s,G_s,B_s,A_s)$，目标颜色分量为 $(R_d,G_d,B_d,A_d)$，源调和因子为 $(S_r,S_g,S_b,S_a)$，而目标调和因子为 $(D_r,D_g,D_b,D_a)$ 。计算出的组合颜色分量归一到 0 到 1 之间，即任何大于 1 的和设为 1，小于 0 的和设为 0 。



使用下列函数设置调和因子

```cpp
glBlendFunc(sFactor, dFactor);
```

其中 sFactor 和 dFactor 分别是源和目标因子，通常直接通过 OpenGL 预定义的常量设置。例如

* `GL_ZERO` 表示 $(0,0,0,0)$，是 dFactor 的默认选项
* `GL_ONE` 表示 $(1,1,1,1)$，是 sFactor 的默认选项



### 着色规则

颜色的每个分量在帧缓冲区中分开存储，通常每个分量占用 8 字节。使用 glColor3f 和 glColor3ub 来设定 RGB 颜色值

```cpp
// 浮点型输入，范围为 0 到 1
void glColor3f(
    GLfloat red, 
    GLfloat green, 
    GLfloat blue
);

// 整型输入，范围为 0 到 255
void glColor3ub(
    GLubyte red, 
    GLubyte green, 
    GLubyte blue
);
```

颜色值一旦设定，在后续的构造过程中将一直使用这一颜色，直到被修改为止。



当绘制多边形时，OpenGL 默认会根据多边形顶点的颜色插值出内部点的颜色。当然，也可以只根据第一个顶点的颜色确定填充颜色。当需要修改填充模式时，使用 glShadeModel 调整

```cpp
void glShadeModel(GLenum mode);
```

可选模式有

* `GL_SMOOTH` 颜色连续插值填充
* `GL_FLAT` 根据第一个顶点颜色填充



### 点和线宽

通过函数

```cpp
void glPointSize(GLfloat size);		// 舍入到整数，大小按照像素设置
void glLineWidth(GLfloat width);	// 舍入到整数，宽度为像素
```

分别设定绘制顶点的大小和绘制直线的宽度。



默认情况下，直线段设为实线。但也可以显示划线、点线或短划和点混合的线段。首先要开启特性

```cpp
glEnable(GL_LINE_STIPPLE);
```

通过此函数设定线型

```cpp
void glLineStipple(
    GLint factor, 		// 说明模式中每一位重复应用多少次才轮到下一位。默认为 1
    GLushort pattern	// 描述如何显示线段的 16 位整数
);
```

第一个参数设置划线之间的距离，第二个参数中值为 1 的位对应一个开像素，值为 0 的位对应一个关像素。例如

```cpp
glEnable(GL_LINE_STIPPLE);
glLineStipple(1, 0xff);
```

由于 0xff 是 255，即有连续 8 位为 1，因此产生相同长度的线段和空隙。



### 顶点数组

当我们需要绘制大量图元时，不停地调用 glBegin、glVertex 和 glEnd 会消耗大量时间，我们有必要提高绘制效率。



OpenGL 提供了顶点数组来快速绘制图元，首先启用数组

```cpp
void glEnableClientState(GLenum array);
```

可选参数有

* `GL_VERTEX_ARRAY`
* `GL_COLOR_ARRAY`
* `GL_INDEX_ARRAY`
* `GL_NORMAL_ARRAY`

更多参数查阅头文件即可。



```cpp
void glColorPointer(
    GLint size, 			// 每组颜色占用的大小，RGB 每个分量占 1，因此 size = 3
    GLenum type, 			// 分量的数据类型
    GLsizei stride, 		// 跳跃值，在存在多种数据时用于区分
    const GLvoid *pointer	// 储存颜色数据的数组
);
```

例如考虑数组

```cpp
pointer[] = {
	r1,g1,b1, v1x,v1y,v1z,
	r2,g2,b2, v2x,v2y,v2z
};
```

我们用连续 6 个浮点数保存一个顶点和它的颜色，这时 stride 设为 `6*sizeof(float)`；一般每个数组只存放同一种数据，这时候只需要设为 0 即可。类似地还有

```cpp
void glVertexPointer(
    GLint size, 
    GLenum type, 
    GLsizei stride, 
    const GLvoid *pointer
);
```

假设我们定义了三维顶点数组和它们对应的颜色数组，则可以调用

```cpp
glVertexPointer(3, GL_FLOAT, 0, vertices);
glColorPointer(3, GL_FLOAT, 0, colors);
```

这样我们就一次性传入了所有顶点和颜色，现在要开始绘制过程。



通过如下函数**绘制**对应索引的点

```cpp
void glArrayElement(GLint i);
```

例如当 `i=1` 时，对应的点就是 `vertices[3],vertices[4],vertices[5]`，它对应的颜色则是 colors 数组中对应的值。此函数必须被包围在 `glBegin` 和 `glEnd` 之间

```cpp
glBegin(GL_TRIANGLES);
    glArrayElement(0);
    glArrayElement(1);
    glArrayElement(2);
glEnd();
```



上面这种做法省去了多次传入顶点的麻烦，但是还不够。我们可以直接完成一个图元的绘制

```cpp
void glDrawElements(
    GLenum mode, 			// 绘图模式
    GLsizei count, 			// 使用点的数量
    GLenum type, 			// 数组的类型
    const GLvoid *indices	// 点的索引的数组
);
```

这里最后 indices 传入点的索引列表，函数将会绘制这些索引对应的点；此函数包含了 glBegin 和 glEnd 的内容，因此可以直接调用。



需要注意 type 只能是 `GL_UNSIGNED_BYTE、GL_UNSIGNED_SHORT、GL_UNSIGNED_INT` 之一，取决于使用什么类型的索引数组

```cpp
// 传入顶点
GLfloat v[] = { -2.0, -2.0, 0.0,
				2.0, 0.0, 0.0 ,
				0.0, 2.0, 0.0 };
// 顶点颜色
GLfloat c[] = { 1, 0, 0,
				0, 1, 0,
				0, 0, 1 };

void mydisplay()
{
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    
    // 注意这两步不能合并
	glEnableClientState(GL_VERTEX_ARRAY);
	glEnableClientState(GL_COLOR_ARRAY);

	glVertexPointer(3, GL_FLOAT, 0, v);
	glColorPointer(3, GL_FLOAT, 0, c);
	
    // 通过索引图元绘制
	glBegin(GL_TRIANGLES);
	glArrayElement(0);
	glArrayElement(1);
	glArrayElement(2);
	glEnd();
    
    // 直接绘制
	int ind[] = { 0, 1, 2 };
	glDrawElements(GL_TRIANGLES, 3, GL_UNSIGNED_INT, ind);

	glutSwapBuffers();
}
```



之前提到顶点数组和颜色数组可以指定跳跃值，我们可以通过这一值将顶点和颜色数据混合存放

```cpp
// 混合数组
GLfloat m[] = { -2.0, -2.0, 0.0,  1, 0, 0,
				2.0, 0.0, 0.0,  0, 1, 0,
				0.0, 2.0, 0.0,  0, 0, 1 };

void mydisplay()
{
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    
    // 注意这两步不能合并
	glEnableClientState(GL_VERTEX_ARRAY);
	glEnableClientState(GL_COLOR_ARRAY);
	
    // 传入首地址的位置，设置跳跃值
	glVertexPointer(3, GL_FLOAT, 6 * sizeof(GLfloat), &m[0]);
	glColorPointer(3, GL_FLOAT, 6 * sizeof(GLfloat), &m[3]);

	glBegin(GL_TRIANGLES);
	glArrayElement(0);
	glArrayElement(1);
	glArrayElement(2);
	glEnd();

	glutSwapBuffers();
}
```



如果我们使用的是连续数据，那么可以使用

```cpp
void glDrawArrays(
    GLenum mode, 	// 绘图模式
    GLint first, 	// 第一个数据索引
    GLsizei count	// 点的数量
);
```

此函数会绘制从 first 开始一共 count 个点的图元。



OpenGL 提供一个可以一次性指定所有顶点和颜色或其它信息的函数

```cpp
void glInterleavedArrays(
    GLenum format, 			// 数组格式
    GLsizei stride, 		// 元素偏移量
    const GLvoid *pointer	// 混合数组
);
```

其中数组格式可以指定 `GL_C3F_V3F`，即三个颜色分量加三个顶点分量配对；偏移量就是指定相邻数据的间隔。



### 多边形绘制

我们可以直接绘制一个矩形

```cpp
void glRectd(
    GLdouble x1, GLdouble y1,	// 左上角顶点位置
    GLdouble x2, GLdouble y2	// 右下角顶点位置
);
```

我们将会介绍，根据顶点的绘制顺序，会显示前面和后面。尽管在二维绘制时，前后面一般相同处理，但是仍然需要注意这一点。



#### 两面绘制

从三维的角度来看，一个多边形具有两个面。每一个面都可以设置不同的绘制方式：填充、只绘制边缘轮廓线、只绘制顶点，默认绘制方式是填充。通过如下函数设置

```cpp
void glPolygonMode(GLenum face, GLenum mode);
```

其中 face 可选参数

* `GL_FRONT` 前面
* `GL_BACK` 后面
* `GL_FRONT_AND_BACK` 前面和后面

mode 可选参数

* `GL_FILL` 填充
* `GL_LINE` 边缘绘制
* `GL_POINT` 顶点绘制



#### 多边形偏移

当我们只绘制三维多边形的边，可能会在边之间产生缝隙。这种称为**缝线**的效果由扫描填充算法和边的画线算法的计算差别造成。在对一个多边形进行填充时，深度值按每一 $(x,y)$ 计算，但是边上的深度值计算与画线算法可能不同。消除缝隙的方法是移动填充子程序计算的深度值，使它们与多边形的边深度值不重叠

```cpp
glEnable(GL_POLYGON_OFFSET_FILL);
glPolygonOffset(factor1, factor2);
```

第一个函数激活特性，第二个函数设定位移总量
$$
depthOffset = factor1\cdot maxSlope + factor2\cdot const
$$
其中 $maxSlope$ 是多边形的最大斜率，$const$ 是实现常数。两个因子常用的值是 0.75 和 1 。例如

```cpp
// 对于 GL_FILL GL_POINT GL_LINE 三种模式都开启偏移
glEnable(GL_POLYGON_OFFSET_LINE | GL_POLYGON_OFFSET_POINT | GL_POLYGON_OFFSET_FILL);
glPolygonOffset(1, 1);
```



#### 消除选定边

当我们绘制凸多边形时，需要将其分割为三角形绘制，这在填充绘制时不会有问题。但是，如果我们只需要绘制边框，那么绘制三角形就会导致内部顶点连线被显示出来，为了消去这些连线，可以通过标记函数

```cpp
glEdgeFlag(flag);
```

指定 `GL_FALSE` 从现在开始不显示边。例如

```cpp
glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);

glBegin(GL_POLYGON);
glVertex3f(-0.5, 0.5, 0.5);
glEdgeFlag(GL_FALSE);			// 前一个顶点开始之后所有连线不显示
glVertex3f(-0.5, -0.5, 0.5);
glEdgeFlag(GL_TRUE);			// 前一个顶点开始之后所有连线显示
glVertex3f(0.5, -0.5, 0.5);
glVertex3f(0.5, 0.5, 0.5);
glEnd();
```



也可以在一个数组中指定标记，其使用思路和顶点数组相同，首先要开启特性

```cpp
glEnableClientState(GL_EDGE_FLAG_ARRAY);
```

然后创建边界标记数组

```cpp
void glEdgeFlagPointer(GLsizei stride, const GLvoid *pointer);
```

其中 stride 是两个数据之间的跳跃值。边界标记也可以和顶点、颜色数组混合使用。



#### 确定正面

一般约定“顶点以逆时针顺序出现在屏幕上的面”为“正面”，另一个面即成为“反面”。我们可以交换正反

```cpp
void glFrontFace(GLenum mode);
```

其中 mode 可选参数

* `GL_CCW` 逆时针方向为正面
* `GL_CW` 顺时针方向为正面



#### 多边形剔除

用于去除多边形物体本身的不可见面，例如面与视线方向正交的情况。

```cpp
glEnable(GL_CULL_FACE);
glCullFace(face);
```

其中 face 可选参数与前面一致。



### 反走样

OpenGL 提供三类图元支持反走样。首先激活反走样

```cpp
glEnable(primitiveType);
```

可选参数

* `GL_POINT_SMOOTH`
* `GL_LINE_SMOOTH`
* `GL_POLYGON_SMOOTH`

如果我们用 RGBA 模式指定颜色，反走样还需要我们激活颜色操作用于边界模糊

```cpp
glEnable(GL_BLEND);
```

然后指定调和颜色的方法

```cpp
glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
```



还可以调整反走样的质量

```cpp
glHint(GL_POINT_SMOOTH, GL_NICEST);
```

可选参数为

* `GL_NICEST` 图像质量优先
* `GL_FASTEST` 运行速度优先



### 查询与保存

通过查询函数可以获得包括属性设定在内的任意状态参数的当前值。

```cpp
glGetBooleanv();
glGetFloatv();
glGetIntegerv();
glGetDoublev();
```

每个函数需要指定一个标识常量和一个对应数据类型的数组用于储存。



我们可以将当前属性保存起来，只需要

```cpp
glPushAttrib(attrGroup);
```

其中 attrGroup 用常量标识，如 `GL_POINT_BIT`、`GL_LINE_BIT` 等，使用 `GL_CURRENT_BIT` 可以保存当前颜色信息。此函数将指定信息放进属性栈保存。通过按位或操作可以同时保存多个属性。



通过函数

```cpp
glPopAttrib();
```

弹出最后一个推入的保存属性，将其恢复。



## 光照和表面绘制

### 点光源函数

首先要激活光源

```cpp
glEnable(GL_LIGHTING);
```



通过如下函数设置点光源

```cpp
void glLightf(
    GLenum light,	// 标识符 
    GLenum pname, 	// 符号属性
    GLfloat param	// 特性值
);
```

标识符可选 `GL_LIGHT0, GL_LIGHT1` 等，符号属性例如

```cpp
float posType[] = {2, 0, 3, 1};
glLightfv(GL_LIGHT0, GL_POSITION, posType);
glEnable(GL_LIGHT0);
```

这里 `GL_POSITION` 用于指定光源位置，如果第四个分量设为零，则得到方向光源，否则是点光源。如果不指定位置，默认为负 $z$ 方向的方向光源。



#### 光源颜色

另一些属性，例如设置光源颜色。可以指定

* `GL_AMBIENT` 环境光
* `GL_DIFFUSE` 漫反射光
* `GL_SPECULAR` 镜面反射光

例如我们设置环境光为黑色

```cpp
float blackColor[] = {0, 0, 0, 1};
glLightfv(GL_LIGHT0, GL_AMBIENT, blackColor);
```



#### 距离衰减

设置光源的辐射强度衰减系数

* `GL_CONSTANT_ATTENUATION`
* `GL_LINEAR_ATTENUATION`
* `GL_QUADRATIC_ATTENUATION`

三个系数分别对应 $a+bd+cd^2$ 中不同次项的系数。



#### 投射光源

对于点光源可以指定投射效果，将光线限制在一个圆锥形空间中

* `GL_SPOT_DIRECTION` 方向
* `GL_SPOT_CUTOFF` 圆锥角
* `GL_SPOT_EXPONENT` 衰减因子

衰减系数为 $\cos^{a_l}\theta$，其中 $\theta$ 是圆锥角，当入射光线与中心方向夹角超过圆锥角，则光源没有照射该点。



#### 全局光照

可以指定若干全局光照参数

```cpp
void glLightModelf(GLenum pname, GLfloat param);
```

例如设定 `GL_LIGHT_MODEL_AMBIENT` 参数控制总的环境光。



当在光照计算中加入表面纹理时，表面高光叠加可能导致图案变形。作为一个选项，纹理图案可仅仅应用于对表面颜色有贡献的非镜面反射项上。使用这一选项，在对每一表面进行光照计算时会产生镜面反射颜色和非镜面反射颜色。纹理图案先和非镜面反射颜色混合，然后再和镜面反射颜色混合。

```cpp
glLightModeli(GL_LIGHT_MODEL_COLOR_CONTROL, GL_SEPARATE_SPECULAR_COLOR);
```



#### 表面特征

表面反射系数可以指定

```cpp
void glMaterialfv(GLenum face, GLenum pname, GLfloat param);
```

其中 face 参数可设定 `GL_FRONT, GL_BACK, GL_FRONT_AND_BACK` 等；pname 则用于标识表面参数，例如

* `GL_EMISSION` 设定表面散射颜色，默认是黑色
* `GL_AMBIENT, GL_DIFFUSE, GL_SEPCULAR` 等设定环境光、漫反射和镜面反射系数

注意这些系数设定使用 4 个分量。



#### 表面绘制

OpenGL 使用常数强度表面绘制或 Gouraud 表面绘制。

```cpp
void glShadeModel(GLenum mode);
```

设为 `GL_FLAT` 使用常数强度，`GL_SMOOTH` 指定 Gouraud 绘制（默认）。此函数在“着色规则”中有介绍。



应用于网格曲面时，OpenGL 会使用多边形顶点的法向计算颜色。可以指定

```cpp
void glNormal3f(GLfloat nx, GLfloat ny, GLfloat nz);
```

来控制表面法向的值。对于平面多边形，只需要指定一次法向，而三角形网格需要对每个顶点指定法向。



虽然法向不一定需要指定为单位向量，但是后者可以减少计算。使用

```cpp
glEnable(GL_NORMALIZE);
```

自动规范化程序中的各种表面向量。



我们也可以指定一个法向量数组，类似于顶点数组，首先要激活数组

```cpp
glEnableClientState(GL_NORMAL_ARRAY);
```

然后指定法向量数组

```cpp
void glNormalPointer(
    GLenum type, 
    GLsizei stride, 
    const GLvoid *pointer
);
```

使用方式和顶点数组相同。



### 像素阵列

矩形的网格图案可以通过数字化一张图片或其它图形来获得，也可以使用图形程序来生成。阵列中每一颜色值映射到一个或多个屏幕像素位置。一个彩色像素阵列称为一个像素图。



#### 内存对齐

字长 32 位的计算机上，如果数据在内存中按照 4 字节的边界对齐，那么硬件提取数据的速度就会快得多，类似的在 64 位计算机上，如果数据地址按照 8 字节对齐，数据存取效率会非常高。



我们可以指定像素存储方式

```cpp
void glPixelStoref(GLenum pname, GLfloat param);
```

例如设置字节边界对齐

```cpp
glPixelStorei(GL_UNPACK_ALIGNMENT, 1);
```



#### 位图函数

如下函数定义一个二值的阵列

```cpp
void glBitmap(
    GLsizei width, 			// 像素宽度
    GLsizei height, 		// 像素高度
    GLfloat xorig, 			// 位图中原点 x 位置，从左下角开始
    GLfloat yorig, 			// 位图中原点 y 位置，从左下角开始
    GLfloat xmove, 			// 绘制位图后，对光栅位置 x 的偏移量
    GLfloat ymove, 			// 绘制位图后，对光栅位置 x 的偏移量
    const GLubyte *bitmap	// 位图地址
);
```

此函数将位图显示在**当前光栅位置**，然后产生指定的光栅位置偏移。



我们可以预先指定光栅位置，即确定位图的输出位置

```cpp
void glRasterPos2f(float x, float y);
void glRasterPos3f(float x, float y, float z);
```



例如我们绘制一个箭头形状

```cpp
GLubyte bitShape[] = {
	0x1c, 0x00, 0x1c, 0x00, 0x1c, 0x00, 0x1c, 0x00, 0x1c, 0x00,
	0xff, 0x80, 0x7f, 0x00, 0x3e, 0x00, 0x1c, 0x00, 0x08, 0x00
};

// 指定位图储存模式，参数 1 表示数据用字节边界对齐
glPixelStorei(GL_UNPACK_ALIGNMENT, 1);

// 在 (0,0) 位置输出位图，位图阵列有 9 列 10 行
glRasterPos2i(0, 0);
glBitmap(9, 10, 0, 0, 20, 15, bitShape);
```

我们一共提供了 10 行数据，每行是有两个 16 进制**两位数**，共 16 位，但是由于我们指定列数为 9，因此之后的列都被忽略。下图给出了我们提供的形状，注意填充顺序是**从下层到上层**，右边两列是像素数据，与上面的列表数组对应。

![](image-20221115201845887.png)



#### 像素图函数

可以用彩色阵列定义图案绘制

```cpp
void glDrawPixels(
    GLsizei width, 			// 像素宽度
    GLsizei height, 		// 像素高度
    GLenum format, 			// 像素数据格式
    GLenum type, 			// 像素的数据类型
    const GLvoid *pixels	// 数据指针
);
```

其中 format 参数可接受的部分格式如下

* `GL_COLOR_INDEX` 每个像素是一个颜色索引
* `GL_RGBA` 每个像素顺序为红色、绿色、蓝色、alpha
* `GL_RGB` 每个像素顺序为红色、绿色、蓝色

type 参数指定像素格式

* `GL_BYTE` 8 位整数
* `GL_UNSIGNED_BYTE` 无符号 8 位整数
* `GL_INT` 32 位整数
* `GL_UNSIGNED_INT` 无符号 32 位整数
* `GL_FLOAT` 单精度浮点数



可以指定要绘制到哪一个缓存中

```cpp
void glDrawBuffer(GLenum mode);
```



#### 光栅操作

除了将像素阵列存入缓存，还可以反过来读取缓存。光栅操作用于描述以某种方式处理一个像素阵列的任何功能。

```cpp
void glReadPixels(
    GLint x, GLint y, 				// 相对窗口左下角的位置
    GLsizei width, GLsizei height, 	// 读取尺寸
    GLenum format, GLenum type, 	// 与前面相同
    GLvoid *pixels
);
```

使用此函数在指定缓存中选择一个矩形块的像素值存入 pixels 列表。



可以指定要读取哪个缓存

```cpp
void glReadBuffer(GLenum mode);
```



我们可以将一块像素数据从源缓存复制到目标缓存

```cpp
void glCopyPixels(
    GLint x, GLint y, 
    GLsizei width, GLsizei height, 
    GLenum type
);
```

其中 type 可选 `GL_COLOR`、`GL_DEPTH` 或 `GL_STENCIL` 。通过 glReadBuffer 和 glDrawBuffer 分别指定源缓存和目标缓存。



如果不希望直接覆盖目标缓存，可以开启位逻辑操作进行更细致的设置

```cpp
// 开启逻辑操作
glEnable(GL_COLOR_LOGIC_OP);

// 设置使用的逻辑操作
glLogicOp(logicOp);
```

其中 logicOp 可选 `GL_AND`、`GL_OR` 或 `GL_XOR` 等。



我们还可以放缩图像

```cpp
void glPixelZoom(Glfloat zoomx, Glfloat zoomy);
```

此函数指定 glDrawPixels 和 glCopyPixels 时对 x, y 方向的放缩比例。



### 填充图案

默认时，凸多边形用当前颜色显示一个实心区域。我们可以用掩膜中值为 1 表示填充为当前颜色，值为 0 表示不改变缓冲中的颜色，通过设置不同尺寸的掩膜，实现不同的纹理。首先开启填充

```cpp
glEnable(GL_POLYGON_STIPPLE);
```



填充图案用 OpenGL 数据类型 GLubyte 以无符号字节进行描述，例如这里我们提供了两行两列数据，共 32 x 32 位掩膜

```cpp
glEnable(GL_POLYGON_STIPPLE);
GLubyte fillPartten[] = { 0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00,
						0xff, 0x00, 0xff, 0x00, 0xff, 0x00, 0xff, 0x00 };
glPolygonStipple(fillPartten);
```

注意每两个 16 进制数占 16 位，因为每个 16 进制数都使用了两位数，最大是 $255=2^8-1$，因此占 8 位。绘制三角形的效果如下

![](image-20221115202757351.png)



### 纹理映射

#### 线纹理

使用一维纹理之前需要激活纹理

```cpp
glEnable(GL_TEXTURE_1D);
```



用单下标颜色数组指定的一维 RGBA 纹理图案参数指定为

```cpp
void glTexImage1D(
    GLenum target, 			// 指定宏常量
    GLint level, 			// 表示纹理层级
    GLint internalformat, 	// 颜色模式
    GLsizei width, 			// 颜色数量
    GLint border, 			// 边界参数
    GLenum format,
    GLenum type, 
    const GLvoid *pixels	// 像素数组
);
```

* target 设为 `GL_TEXTURE_1D` 指出正在为一个一维对象定义一个纹理数组，如果不清楚系统是否支持该参数指定的纹理图案，则可以设为 `GL_PROXY_TEXTURE_1D` 询问系统
* level 指定纹理层级，设为 0 表示该数组不是某个大数组的缩减
* internalformat 指定为 `GL_RGBA`
* width 指定纹理图案的颜色数量
* border 设为 0 则颜色数量是 2 的幂，设为 1 则颜色数量是 2 加上 2 的幂，多出的 2 中颜色用于与相邻图案进行调和
* format 和 type 与前面像素图函数中的相同



#### 纹理过滤

当一组纹理元素映射到一个或多个像素区域时，纹理元素的边界通常与像素边界位置不对齐。使用下列函数给出每一像素的**最接近**纹理元素的颜色

```cpp
glTexParameteri(GL_TEXTURE_1D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_1D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
```

纹理子程序**在必须放大纹理图案的一部分以适应指定的场景坐标时使用第一个函数，在必须缩减纹理图案时使用第二个函数**。尽管这样可以加快计算，但可能导致走样。



我们可以把周围一组纹素计算加权平均值，并把结果作为映射到的纹理值，这种方法称为线性滤波 linear filtering，可以用 `GL_LINEAR` 替换 `GL_NEAREST` 进行线性混合

```cpp
glTexParameteri(GL_TEXTURE_1D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_1D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
```



我们在 glTexImage1D 中可以指定边界颜色，也可以由 glTexParameterfv 指定边界

```cpp
glTexParameterfv(GL_TEXTURE_1D, GL_TEXTURE_BORDER_COLOR, borderColor);
```

默认的边界颜色是黑色。



#### 纹理坐标

对于一个一维纹理空间，颜色值用跨越纹理空间的 0 到 1 之间的单个 $s$ 坐标指定，通过将纹理坐标赋给对象位置应用纹理。

```cpp
void glTexCoord1f(GLfloat s);
```

此函数指定具体的 $s$ 值，默认 $s$ 为 0，通过指定 $s$ 来获得纹理空间中的颜色。



例如我们指定红绿交替的一维纹理图案

```cpp
// 指定 4 种颜色纹理
float texLine[] = { 0, 1, 0, 1,
					1, 0, 0, 1,
					0, 1, 0, 1,
					1, 0, 0, 1 };

// 指定边界混合模式
glTexParameteri(GL_TEXTURE_1D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_1D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);

// 指定一维纹理图像
glTexImage1D(GL_TEXTURE_1D, 0, GL_RGBA, 4, 0, GL_RGBA, GL_FLOAT, texLine);

// 激活纹理
glEnable(GL_TEXTURE_1D);

// 绘制纹理线段
glBegin(GL_LINES);
glTexCoord1f(0);	glVertex3f(0, 0, 0);
glTexCoord1f(1);	glVertex3f(1, 1, 0);
glEnd();

glDisable(GL_TEXTURE_1D);
```

这里我们指定端点颜色分别是 $s=0,s=1$，会**将纹理坐标 0 到 1 之间的颜色映射到线段上**，得到交替的绿色和红色。



#### 环绕模式

可以使用超出单位区间的 $s$ 值，此时超出部分的整数部分被忽略，只考虑小数部分。通过指定超出范围的 $s$ 值，可以实现对纹理图案的压缩和拉伸，例如

```cpp
glBegin(GL_LINES);
glTexCoord1f(0);	glVertex3f(0, 0, 0);
glTexCoord1f(n);	glVertex3f(1, 1, 0);
glEnd();
```

将 0 到 n 映射到线段上，这会使得整个纹理图案被重复 n 次。如果取 0.5，则会只映射一半的纹理图案。也可以实现对纹理的平移

```cpp
glBegin(GL_LINES);
glTexCoord1f(v);	glVertex3f(0, 0, 0);
glTexCoord1f(1+v);	glVertex3f(1, 1, 0);
glEnd();
```



默认情况下，一维纹理坐标放大会像前面所说的重复纹理图案，即默认设定为

```cpp
glTexParameteri(GL_TEXTURE_1D, GL_TEXTURE_WRAP_S, GL_REPEAT);
```

如果我们不希望重复，则可以设为截取模式，超出纹理范围将按照边界值截取：超过 1 按照 1 截取，小于 0 按照 0 截取

```cpp
glTexParameteri(GL_TEXTURE_1D, GL_TEXTURE_WRAP_S, GL_CLAMP);
```

对于二维纹理，可在 $s,t$ 两个方向上设定为

```cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP);
```



#### 纹理坐标数组

类似于顶点数组、颜色数组，我们也可以指定纹理数组批量指定**纹理坐标**。首先激活纹理数组

```cpp
glEnableClientState(GL_TEXTURE_COORD_ARRAY);
```

指定纹理数组的函数和指定顶点数组类似

```cpp
void glTexCoordPointer(
    GLint size, 			// 坐标尺寸，默认为 4，即四维齐次坐标
    GLenum type, 			// 坐标值类型
    GLsizei stride, 		// 跳跃值
    const GLvoid *pointer	// 坐标数组
);
```

使用时先确定一个纹理图案，然后建立坐标数组实现纹理映射。



#### 表面纹理

我们可以用类似的函数建立二维纹理

```cpp
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, texWidth, texHeight, 0, 
		dataFormat, dataType, surfTexArray);
glEnable(GL_TEXTURE_2D);
```

区别在于需要指定宽高，它们在没有边界时必须是 2 的幂，有边界时必须是 2 加 2 的幂。注意 surfTexArray 要使用三维数组。



同理给出边界混合函数

```cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
```

以及使用二维坐标指定纹理映射

```cpp
glTexCoord2f(sCoord, tCoord);
```



我们设定一个 16 x 16 的棋盘纹理，将其映射到四边形表面

```cpp
// 指定 16 x 16 颜色纹理
float texArray[16][16][4];
for (int i = 0; i < 16; i++)
{
	for (int j = 0; j < 16; j++)
	{
		if ((i + j) % 2 == 0)
		{
			texArray[i][j][0] = 1;
			texArray[i][j][1] = 1;
			texArray[i][j][2] = 1;
			texArray[i][j][3] = 1;
		}
		else
		{
			texArray[i][j][0] = 0;
			texArray[i][j][1] = 0;
			texArray[i][j][2] = 0;
			texArray[i][j][3] = 1;
		}
	}
}

// 指定边界混合模式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);

// 指定二维纹理图像
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, 16, 16, 0, GL_RGBA, GL_FLOAT, texArray);

// 激活纹理
glEnable(GL_TEXTURE_2D);

// 设定映射坐标
glBegin(GL_QUADS);
glTexCoord2f(0, 0);		glVertex3f(0, 0, 0);
glTexCoord2f(1, 0); 	glVertex3f(1, 0, 0);
glTexCoord2f(1, 1);		glVertex3f(1, 1, 0);
glTexCoord2f(0, 1); 	glVertex3f(0, 1, 0);
glEnd();

glDisable(GL_TEXTURE_2D);
```

同理可以指定体纹理，方法类似，不再赘述。



#### 映射选项

纹理元素映射到对象上时，其值可以与对象颜色混合，或者取代对象颜色。

```cpp
glTexEnvi(GL_TEXTURE_ENV, GL_TEXTURE_ENV_MODE, method);
```

如果 method 设为 `GL_REPLACE` 就会直接取代，设为 `GL_MODULATE` 会用纹理颜色调制当前颜色，这是默认应用方法。



当指定 `GL_BLEND` 时，可以指定一种或多种颜色进行混合

```cpp
glTexEnvf(GL_TEXTURE_ENV, GL_TEXTURE_ENV_MODE, blendingColor);
```



#### 帧缓存纹理

可以从当前帧缓存中截取图案作为纹理

```cpp
void glCopyTexImage2D(
    GLenum target, 					// 目标缓存
    GLint level, 					// 是否缩减
    GLenum internalFormat, 			// 颜色模式
    GLint x, GLint y, 				// 左下角位置
    GLsizei width, GLsizei height, 	// 宽高
    GLint border					// 是否有边界
);
```

例如截取不缩减且没有边界的二维纹理

```cpp
glCopyTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, x0, y0, texWidth, texHeight, 0);
```

我们可以截取一块子图案，然后将其附加到当前纹理图案中

```cpp
glCopyTexSubImage2D(GL_TEXTURE_2D, 0, xTexElement, yTexElement, x0, y0, texWidth, texHeight);
```

此函数将截取的图案放在纹理图案的 `xTexElement, yTexElement` 位置。



#### 纹理子图

我们可以定义一个纹理图案的子图案，从而修改原始图案中的部分，这会比重新创建整个纹理图案更方便。

```cpp
void glTexSubImage2D(
    GLenum target, // 目标纹理
    GLint level, 
    GLint xoffset, GLint yoffset, 	// 相对纹理图案左下角的坐标
    GLsizei width, GLsizei height, 	// 子图案尺寸
    GLenum format, GLenum type, 
    const GLvoid *pixels
);
```



#### 缩减图案

当纹理对象远离观察者位置时，对应的纹理要随之缩减。可以通过反复使用 glTexImage 函数，指定不同层级的纹理。必须要指定所有从原始图案开始除以 2 的缩略图，例如 16 x 16 必须指定 8 x 8, 4 x 4, 2 x 2, 1 x 1 的所有缩略图，否则不能显示效果。**非常重要的是需要设定 GL_TEXTURE_MIN_FILTER 常量选择从缩略图案确定像素颜色。**

```cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST_MIPMAP_NEAREST);
```

该函数指出应当尽可能使用匹配像素尺寸 `MIPMAP_NEAREST` 的缩减图案，然后将图案中靠近纹理元素 GL_NEAREST 的颜色赋给像素。我们也可以使用 `MIPMAP_LINEAR` 或者 `GL_LINEAR` 等选项。



我们指定不同层级使用不同颜色的缩略图，绘制一个迅速远离观察点的四边形

```cpp
// 纹理图案和缩略图案
float texArray16[16][16][4];
float texArray8[8][8][4];
float texArray4[4][4][4];
float texArray2[2][2][4];
float texArray1[1][1][4];

// 激活纹理
glEnable(GL_TEXTURE_2D);

// 16 x 16
for (int i = 0; i < 16; i++)
{
	for (int j = 0; j < 16; j++)
	{
		texArray16[i][j][0] = 1;
		texArray16[i][j][1] = 0;
		texArray16[i][j][2] = 0;
		texArray16[i][j][3] = 1;
	}
}
for (int i = 0; i < 8; i++)
{
	for (int j = 0; j < 8; j++)
	{
		texArray8[i][j][0] = 1;
		texArray8[i][j][1] = 1;
		texArray8[i][j][2] = 0;
		texArray8[i][j][3] = 1;
	}
}
for (int i = 0; i < 4; i++)
{
	for (int j = 0; j < 4; j++)
	{
		texArray4[i][j][0] = 0;
		texArray4[i][j][1] = 1;
		texArray4[i][j][2] = 0;
		texArray4[i][j][3] = 1;
	}
}
for (int i = 0; i < 2; i++)
{
	for (int j = 0; j < 2; j++)
	{
		texArray2[i][j][0] = 0;
		texArray2[i][j][1] = 0;
		texArray2[i][j][2] = 1;
		texArray2[i][j][3] = 1;
	}
}
for (int i = 0; i < 1; i++)
{
	for (int j = 0; j < 1; j++)
	{
		texArray1[i][j][0] = 1;
		texArray1[i][j][1] = 1;
		texArray1[i][j][2] = 1;
		texArray1[i][j][3] = 1;
	}
}

// 指定用缩略图纹理填充的边界过滤模式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST_MIPMAP_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);

// 指定纹理层级
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, 16, 16, 0, GL_RGBA, GL_FLOAT, texArray16);
glTexImage2D(GL_TEXTURE_2D, 1, GL_RGBA, 8, 8, 0, GL_RGBA, GL_FLOAT, texArray8);
glTexImage2D(GL_TEXTURE_2D, 2, GL_RGBA, 4, 4, 0, GL_RGBA, GL_FLOAT, texArray4);
glTexImage2D(GL_TEXTURE_2D, 3, GL_RGBA, 2, 2, 0, GL_RGBA, GL_FLOAT, texArray2);
glTexImage2D(GL_TEXTURE_2D, 4, GL_RGBA, 1, 1, 0, GL_RGBA, GL_FLOAT, texArray1);

// 这里纹理映射从 0 到 10 是为了让纹理更密集，从而使缩略纹理能够更快出现
glBegin(GL_QUADS);
glTexCoord2f(0, 0);		glVertex3f(-2.0, -1.0, 0);
glTexCoord2f(10, 0);	glVertex3f(-2.0, 1.0, 0);
glTexCoord2f(10, 10);	glVertex3f(2000.0, 1.0, -6000.0);
glTexCoord2f(0, 10);	glVertex3f(2000.0, -1.0, -6000.0);
glEnd();

glDisable(GL_TEXTURE_2D);
```



我们可以让 OpenGL 自动生成缩减图案，使用函数

```cpp
gluBuild2DMipmaps(GL_TEXTURE_2D, GL_RGBA, 16, 16, GL_RGBA, GL_FLOAT, texArray);
```

例如自动产生棋盘纹理的缩略图案

```cpp
glEnable(GL_TEXTURE_2D);

// 使用 16 x 16 棋盘纹理
float texArray[16][16][4];
for (int i = 0; i < 16; i++)
{
	for (int j = 0; j < 16; j++)
	{
		if ((i + j) % 2 == 0)
		{
			texArray[i][j][0] = 1;
			texArray[i][j][1] = 1;
			texArray[i][j][2] = 1;
			texArray[i][j][3] = 1;
		}
		else
		{
			texArray[i][j][0] = 0;
			texArray[i][j][1] = 0;
			texArray[i][j][2] = 0;
			texArray[i][j][3] = 1;
		}
	}
}

// 指定两个边界混合模式，第一个混合模式启用缩略图显示
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST_MIPMAP_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);

// 指定纹理层级
gluBuild2DMipmaps(GL_TEXTURE_2D, GL_RGBA, 16, 16, GL_RGBA, GL_FLOAT, texArray);

glBegin(GL_QUADS);
glTexCoord2f(-2, -1);		glVertex3f(-2.0, -1.0, 0);
glTexCoord2f(-2, 1);		glVertex3f(-2.0, 1.0, 0);
glTexCoord2f(2000, 1);		glVertex3f(2000.0, 1.0, -6000.0);
glTexCoord2f(2000, -1);		glVertex3f(2000.0, -1.0, -6000.0);
glEnd();

glDisable(GL_TEXTURE_2D);
```



#### 纹理命名

OpenGL 允许创建多个命名的纹理图案，例如将纹理编号为 3

```cpp
glBindTexture(GL_TEXTURE_1D, 3);
glTexImage1D(GL_TEXTURE_1D, 0, GL_RGBA, 4, 0, GL_RGBA, GL_FLOAT, texLine);
```

当我们需要使用这一纹理时，只要再次调用绑定函数

```cpp
glBindTexture(GL_TEXTURE_1D, 3);
```

将该图案指定为当前纹理状态。如果创建了多个纹理图案，只需调用对应编号的绑定函数即可应用新的纹理。



为了避免重复使用定义纹理名，可以查询一个纹理名是否已经使用

```cpp
glIsTextures(texName);
```

返回 `GL_TRUE` 或 `GL_FALSE` 。



更好的方法是让 OpenGL 自动生成命名，例如

```cpp
GLuint texName;
glGenTextures(1, &texName);
glBindTexture(GL_TEXTURE_2D, texName);
```

其中 glGenTextures 产生了一个纹理名，我们可以一次产生多个纹理名

```cpp
GLuint texNameArray[4];
glGenTextures(4, texNameArray);
```



当纹理使用完毕后，需要释放内存空间

```cpp
glDeleteTextures(nTextures, texNameArray);
```

指定释放的纹理数和纹理名的首地址。



#### 自动纹理

OpenGL 可以自动生成纹理坐标，而不需要再用 glTexCoord 显式指定

```cpp
void glTexGeni(
    GLenum coord, 
    GLenum pname, 
    GLint param
);
```

其中参数 coord 可选 `GL_S, GL_T` 确定坐标方向，pname 可选

* `GL_TEXTURE_GEN_MODE`
* `GL_OBJECT_PLANE`
* `GL_EYE_PLANE`

我们使用第一个纹理生成模式，此时 param 可设为

* `GL_OBJECT_LINEAR` 物体模式，纹理会跟随物体的转动而转动
* `GL_EYE_LINEAR` 视觉模式，纹理不会跟随物体转动，始终保持原状
* `GL_SPHERE_MAP` 环境纹理（球体贴图）具有很好的反射效果
* `GL_NORMAL_MAP` 环境纹理（反射纹理）可替换GL_SPHERE_MAP
* `GL_REFLECTION_MAP` 常用于立方体贴图，或者散射与反射的场景



我们首先设置纹理模式为重复模式

```cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
```

然后设置生成纹理模式

```cpp
glTexGeni(GL_S, GL_TEXTURE_GEN_MODE, GL_OBJECT_LINEAR);
glTexGeni(GL_T, GL_TEXTURE_GEN_MODE, GL_OBJECT_LINEAR);
```

启用自动生成

```cpp
glEnable(GL_TEXTURE_GEN_S);
glEnable(GL_TEXTURE_GEN_T);
```



于是我们产生纹理图像的过程如下，免去了设置纹理坐标的过程

```cpp
// 指定 16 x 16 颜色纹理
float texArray[16][16][4];
for (int i = 0; i < 16; i++)
{
	for (int j = 0; j < 16; j++)
	{
		if ((i + j) % 2 == 0)
		{
			texArray[i][j][0] = 1;
			texArray[i][j][1] = 1;
			texArray[i][j][2] = 1;
			texArray[i][j][3] = 1;
		}
		else
		{
			texArray[i][j][0] = 0;
			texArray[i][j][1] = 0;
			texArray[i][j][2] = 0;
			texArray[i][j][3] = 1;
		}
	}
}

// 指定边界混合模式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);

// 指定二维纹理图像
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, 16, 16, 0, GL_RGBA, GL_FLOAT, texArray);

// 激活纹理
glEnable(GL_TEXTURE_2D);

// 设置自动生成纹理
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);

glTexGeni(GL_S, GL_TEXTURE_GEN_MODE, GL_OBJECT_LINEAR);
glTexGeni(GL_T, GL_TEXTURE_GEN_MODE, GL_OBJECT_LINEAR);

glEnable(GL_TEXTURE_GEN_S);
glEnable(GL_TEXTURE_GEN_T);

// 绘制四边形
glBegin(GL_QUADS);
glVertex3f(0, 0, 0);
glVertex3f(1, 0, 0);
glVertex3f(1, 2, 0);
glVertex3f(0, 2, 0);
glEnd();

glDisable(GL_TEXTURE_2D);
```



## 视图变换

### 坐标系

在 glVertex 中的单位由应用程序确定，称为世界坐标系。视景体相对于世界坐标系指定，它确定出现在图像中的对象。在 OpenGL 内部会把世界坐标转化为照相机（视点）坐标，然后转化为屏幕坐标。



默认情况下，照相机被放置在世界坐标系的原点，**指向 z 轴的负方向**。默认视景体是一个中心在原点，边长为 2 的立方体。

![](图形开发/渲染框架/Glut.assets/image-20221110135125560.png)



默认的正交视图中，点沿着 z 轴投影到 $z=0$ 的平面上。

![](image-20221110135256414.png)

其中投影是利用投影矩阵乘法进行的。OpenGL 是一个状态机，拥有模型视点 modelview、矩阵栈和投影 projection 矩阵栈。



使用 glMatrixMode 设置矩阵模式，有三种矩阵模式

* `GL_MODELVIEW` 模型视图矩阵，确定从局部坐标系到世界坐标系的转换矩阵
* `GL_PROJECTION` 投影矩阵，确定从观察者坐标系到屏幕坐标的转换矩阵
* `GL_TEXTURE` 纹理矩阵



### 模型视图

模型视图用于提供对模型矩阵的变换。由于移动的相对性，改变观察点位置和改变物体本身位置等效，即相机的移动和物体的移动可以等价地看待，因此 OpenGL 用同样的函数实现两种功能。首先要切换到模型视图，然后将当前矩阵设为单位阵

```cpp
glMatrixMode(GL_MODELVIEW);
glLoadIdentity();
```



#### 视图变换

模型视图变换涉及三个函数：

```cpp
// 对物体进行位移，当前矩阵将乘上此位移矩阵
void glTranslatef(
    GLfloat x, 
    GLfloat y, 
    GLfloat z
);

// 旋转物体，当前矩阵将乘上此旋转矩阵。物体绕 (0,0,0) 到 (x,y,z) 的直线以逆时针旋转 angle 度
void glRotatef(
    GLfloat angle, 
    GLfloat x, 
    GLfloat y, 
    GLfloat z
);

// 伸缩物体，当前矩阵将乘上此伸缩矩阵。在 x,y,z 方向上分别以比例 x,y,z 伸缩 
void glScalef(
    GLfloat x, 
    GLfloat y, 
    GLfloat z
);
```

注意：由于每次调用函数都会**右乘上变换矩阵**，最后作用于坐标时的变换顺序与函数调用顺序相反。例如依次作用 $T,R,S$ 做平移、旋转和伸缩变换，最终的变换为 $(TRS)v$，其中 $v$ 是坐标。实际产生的变换是伸缩、旋转，最后平移。



#### 移动视点

如果要改变观察点的位置，使用 gluLookAt 函数，它提供一个简单直观的方式来设置镜头位置和方向。参数分别对应

* 镜头位置
* 视线上的点，即镜头朝向的点
* 镜头顶部的朝向

![](图形开发/渲染框架/Glut.assets/image-20221110185232886.png)

```cpp
void gluLookAt(
    GLdouble eyex, GLdouble eyey, GLdouble eyez, 
    GLdouble centerx, GLdouble centery, GLdouble centerz, 
    GLdouble upx, GLdouble upy, GLdouble upz
);
```

**注意此函数应当在模型视图矩阵模式下调用，在投影模式下不能保证设置正确**。例如

```cpp
glMatrixMode(GL_MODELVIEW);
glLoadIdentity();

// 设置视点
gluLookAt(0.0f, 0.0f, 10.0f,
	0.0f, 0.0f, 0.0f,
	0.0f, 1.0f, 0.0f);
```



### 投影变换

投影变换定义一个可视空间，**可视空间以外的物体不会被绘制到屏幕上**。默认的正交投影矩阵为
$$
M = 
\begin{pmatrix}
1\\
& 1\\
& & 0\\
& & & 1
\end{pmatrix}
$$
即它简单地将所有点的 $z$ 坐标置零进行投影。



首先切换到投影模式，然后载入单位矩阵

```cpp
glMatrixMode(GL_PROJECTION);
glLoadIdentity();
```



#### 透视投影

OpenGL 支持透视投影可以用两个函数实现

```cpp
void glFrustum(
    GLdouble left, 		// 最近投影面的左边位置
    GLdouble right, 	// 最近投影面的右边位置
    GLdouble bottom, 	// 最近投影面的下边位置
    GLdouble top, 		// 最近投影面的上边位置
    GLdouble zNear, 	// 相机到最近投影面的距离，必须为正
    GLdouble zFar		// 相机到最远投影面的距离，必须为正
);
```

![](image-20221110184255523.png)

只有**前后两个面之间**的物体才会显示在屏幕上。



更常用的投影函数是

```cpp
void gluPerspective(
    GLdouble fovy, 		// 可视角
    GLdouble aspect, 	// 宽高比
    GLdouble zNear, 	// 相机到最近投影面的距离，必须为正
    GLdouble zFar		// 相机到最远投影面的距离，必须为正
);
```

![](图形开发/渲染框架/Glut.assets/image-20221110184754107.png)

只有**前后两个面之间**的物体才会显示在屏幕上。



#### 正交投影

正交投影是假定在无穷远处观察的结果，有更好的运行速度。使用

```cpp
void glOrtho(
    GLdouble left, 
    GLdouble right, 
    GLdouble bottom, 
    GLdouble top, 
    GLdouble zNear, 
    GLdouble zFar
);
```

对于正交投影，Near 和 Far 值应当同时为正或同时为负，这是因为相机只能看向一个方向，物体在视点前面时两个值同正，反之同负。

![](图形开发/渲染框架/Glut.assets/image-20221110185514314.png)

其中的长方体就是视景体，只有处于视景体中的物体才会被显示。



#### 二维视景体

二维顶点命令默认将所有顶点放在 $z=0$ 的平面上。如果应用程序只需要绘制二维图像，则我们可以直接使用

```cpp
void gluOrtho2D(
    GLdouble left, 
    GLdouble right, 
    GLdouble bottom, 
    GLdouble top
);
```

设置正交视景体，即退化为裁剪窗口。有一个特殊的使用技巧，如果想要正交投影的 $y$ 方向反向，通常的做法需要进行平移和反射变换，但更快的做法是

```cpp
gluOrtho2D(0, w, h, 0);
```

这里 $w,h$ 分别是视景体的宽度和高度。注意到我们将 bottom 设为大于 top 的值，这会直接将 $y$ 的方向颠倒。



### 视口尺寸

当要将投影完成的图像绘制到窗口上时，我们可以决定将其绘制到窗口的哪个部分。使用 glViewport 来定义视口

```cpp
void glViewport(
    GLint x, 		// 视口左下角 x 坐标
    GLint y, 		// 视口左下角 y 坐标
    GLsizei width,	// 视口宽度
    GLsizei height	// 视口高度
);	
```

单位均为像素。决定视口后，OpenGL 会将视景体中的内容按照视口比例投影绘制到视口当中。



### 矩阵操作

我们可以上传程序中定义的矩阵

```cpp
void glLoadMatrixf(
    const GLfloat *m
);
```

它接收 16 个元素的一维数组，按列定义了 4 x 4 矩阵。此函数将会用我们上传的矩阵**覆盖**当前矩阵。



有时我们希望添加一个变换矩阵，使用

```cpp
void glMultMatrixf(
    const GLfloat *m
);
```

将我们的矩阵乘在当前矩阵的右边。



许多时候需要保存变换矩阵，以供以后使用。OpenGL 为每种类型的矩阵维持一个堆栈，应用如下函数处理相应堆栈中的矩阵

```cpp
void glPushMatrix(void);	// 将当前矩阵压入堆栈
void glPopMatrix(void);		// 弹出最后一个压入堆栈的矩阵
```

这里堆栈是根据 glMatrixMode 设置的矩阵类型来区分的。弹出矩阵会将当前矩阵替换为弹出的矩阵。



如果想要获取当前的变换矩阵，利用查询函数

```cpp
void glGetDoublev(
    GLenum pname, 		// 堆栈名
    GLdouble *params	// 输出地址
);
```

输入代表堆栈名的宏，查询函数将会把变换矩阵存放到 params 开头的数组中。例如

```cpp
double m[16];
glGetDoublev(GL_MODELVIEW_MATRIX, m);
```

可用的宏有

* GL_MODELVIEW_MATRIX 视图矩阵
* GL_PROJECTION_MATRIX 投影矩阵
* GL_TEXTURE_MATRIX 纹理矩阵



我们可以获得栈中有多少矩阵

```cpp
glGetIntegerv(GL_MODELVIEW_STACK_DEPTH, numMats);
```

最初模型视图矩阵栈中只有单位矩阵。



### 屏幕坐标

了解了视图变换的基本形式，我们可以随时计算出空间中一点的屏幕坐标。首先要获得模型视图矩阵和投影变换矩阵

```cpp
// 记录变换矩阵
Matrix Model(4, 4);
Matrix Project(4, 4);

// 获取变换矩阵
double m[16], p[16];
glGetDoublev(GL_MODELVIEW_MATRIX, m);
glGetDoublev(GL_PROJECTION_MATRIX, p);

// 注意矩阵的行列要转置
for (int i = 0; i < 4; i++)
{
	for (int j = 0; j < 4; j++)
	{
		Model(j, i) = m[i * 4 + j];
		Project(j, i) = p[i * 4 + j];
	}
}
```

然后对任何一点依次作用，应用透视除法规范化齐次分量

```cpp
Vector v = {x, y, z, 1};
v = Project * Model * v;
v = v / v[3];
```

现在得到的点是从投影视景体（正交视景体或透视视景体）到标准视景体（棱长为 2 的立方体）映射的结果，现在取 `(v[0],v[1])` 就得到投影到平面上的坐标，最后转换为屏幕坐标

```cpp
CPoint p = { int(v[0] * width / 2 + width / 2), int(height / 2 - v[1] * height / 2) };
```



## 消息处理

### 窗口消息

当改变窗口大小时，窗口比例改变会造成图像扭曲，这时候需要动态调整窗口比例。OpenGL 在窗口改变时会调用函数

```cpp
void glutReshapeFunc(void (*func)(int width, int height));
```

中注册的回调函数，我们可以在此回调函数中重新调整窗口大小。**此函数在窗口初始化时也会调用，可以在其中设置视图投影信息**。



我们先定义重绘函数

```cpp
void changeSize(int width, int height)
{
	// 防止出现宽度为零的窗口
	if (height == 0)
		height = 1;
	double ratio = 1.0 * width / height;

	// 切换到投影矩阵，用于确定可视的体积
	glMatrixMode(GL_PROJECTION);

	// 载入单位矩阵来重置当前矩阵
	glLoadIdentity();

	// 将视口设置为整个窗口
	glViewport(0, 0, width, height);

	// 设置视觉参数：可视角度域的 yz 平面，视口宽高比例，远、近的切面
	gluPerspective(45, ratio, 1, 1000);

	// 切换回默认的模式视觉矩阵
	glMatrixMode(GL_MODELVIEW);
}
```

我们通过调整视口来使得绘图区域与窗口大小一致，通过调整投影变换重建符合新的窗口比例的透视关系。



最终的代码如下

```cpp
#pragma comment(linker, "/subsystem:\"windows\" /entry:\"mainCRTStartup\"")

#include <glut.h>

void changeSize(int width, int height)
{
	// 防止出现宽度为零的窗口
	if (height == 0)
		height = 1;
	double ratio = 1.0 * width / height;

	// 切换到投影矩阵，用于确定可视的体积
	glMatrixMode(GL_PROJECTION);

	// 载入单位矩阵来重置当前矩阵
	glLoadIdentity();

	// 将视口设置为整个窗口
	glViewport(0, 0, width, height);

	// 设置视觉参数：可视角度域的 yz 平面，视口宽高比例，远、近的切面
	gluPerspective(45, ratio, 1, 1000);

	// 切换回默认的模式视觉矩阵
	glMatrixMode(GL_MODELVIEW);
}

int main(int argc, char* argv[])
{
	glutInit(&argc, argv);							// 初始化函数

	glutInitDisplayMode(GLUT_SINGLE | GLUT_RGB);	// 输出模式
	glutInitWindowSize(500, 500);					// 定义窗口大小
	glutInitWindowPosition(0, 0);					// 窗口位置
	glutCreateWindow("Test");						// 创建窗口

	glutReshapeFunc(changeSize);					// 注册窗口改变的回调函数

	glutMainLoop();									// 进入事件循环

	return 0;
}
```



### 空闲消息

当程序没有消息要处理时，会调用应用空闲时的回调函数，通过下面的方式注册

```cpp
void glutIdleFunc(void (*func)(void));
```



利用这一点可以实现动态图像效果，只要在空闲回调函数中调整视图关系，就可以使绘制结果运动。例如可以绘制一个旋转的三角形。我们使用双缓冲模式，同时开启深度缓冲。我们规定一个全局的角度，每次调用渲染函数就修改这个角度。最后的代码如下：

```cpp
#pragma comment(linker, "/subsystem:\"windows\" /entry:\"mainCRTStartup\"")

#include <glut.h>

float angle = 0.0f;

void renderScene(void) 
{
	// 清除颜色和深度缓冲
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

	// 载入单位阵
	glLoadIdentity();

	// 设置摄像机
	gluLookAt(0.0f, 0.0f, 10.0f,
		0.0f, 0.0f, 0.0f,
		0.0f, 1.0f, 0.0f);

	// 旋转摄像机角度
	glRotatef(angle, 0.0f, 1.0f, 0.0f);

	// 开始绘图
	glBegin(GL_TRIANGLES);
	glVertex3f(-2.0f, -2.0f, 0.0f);
	glVertex3f(2.0f, 0.0f, 0.0);
	glVertex3f(0.0f, 2.0f, 0.0);
	glEnd();

	// 增加旋转角，单位是度
	angle += 0.1f;

	// 交换缓冲
	glutSwapBuffers();
}

void changeSize(int width, int height)
{
	// 防止出现宽度为零的窗口
	if (height == 0)
		height = 1;
	double ratio = 1.0 * width / height;

	// 切换到投影矩阵，用于确定可视的体积
	glMatrixMode(GL_PROJECTION);

	// 载入单位矩阵来重置当前矩阵
	glLoadIdentity();

	// 将视口设置为整个窗口
	glViewport(0, 0, width, height);

	// 设置视觉参数：可视角度域的 yz 平面，视口宽高比例，远、近的切面
	gluPerspective(45, ratio, 1, 1000);

	// 切换回默认的模式视觉矩阵
	glMatrixMode(GL_MODELVIEW);
}

int main(int argc, char* argv[]) 
{
	// 初始化操作
	glutInit(&argc, argv);
	glutInitDisplayMode(GLUT_DEPTH | GLUT_DOUBLE | GLUT_RGBA);
	glutInitWindowPosition(100, 100);
	glutInitWindowSize(320, 320);
	glutCreateWindow("Triangle");

	// 注册之前的渲染和改变大小的回调函数
	glutDisplayFunc(renderScene);
	glutReshapeFunc(changeSize);

	// 注册空闲时间回调函数，由于我们需要重新渲染，因此注册的函数和前面的渲染函数相同
	glutIdleFunc(renderScene);

	// 事件循环
	glutMainLoop();

	return 0;
}
```



### 键盘消息

#### 普通按键

GLUT 提供 glutKeyBoardFunc 告知窗体系统处理普通按键事件，例如字母、数字和 ASCII 包含的数码。

```cpp
void glutKeyboardFunc(void (*func) (unsigned char key, int x, int y));
```

其中三个参数分别是 ASCII 码和鼠标位置，位置相对于客户端左上角。



对于普通的按键，例如字母按键，可以直接用字符表表示。例如添加按下 ESC 键退出循环

```cpp
void processNormalKeys(unsigned char key, int x, int y) 
{
	// 27 是 ESC 的 ASCII 码
    if (key == 27)
        std::exit(0);
    if (key == 'r')
        std::cout << "red" << std::endl;
}
```

注意使用 exit 函数需要头文件 `<cstdlib>` 。



#### 特殊按键

对于特殊按键，则使用 glutSpecialFunc 进行处理

```cpp
void glutSpecialFunc(void (*func) (int key, int x, int y));
```

例如我们实现当按下 `F1，F2，F3` 时分别将三角形变成红、绿、蓝色。

```cpp
float red = 1.0f, blue = 1.0f, green = 1.0f;

void processSpecialKeys(int key, int x, int y)
{
	switch (key)
	{
	case GLUT_KEY_F1:
		red = 1.0;
		green = 0.0;
		blue = 0.0; break;
	case GLUT_KEY_F2:
		red = 0.0;
		green = 1.0;
		blue = 0.0; break;
	case GLUT_KEY_F3:
		red = 0.0;
		green = 0.0;
		blue = 1.0; break;
	}
}

void renderScene(void) 
{
    ...
    
	// 更改绘图颜色
    glColor3f(red,green,blue);
    
    ...
}
```



GLUT 头文件中定义了特殊按键的宏

```cpp
/* function keys */
#define GLUT_KEY_F1			1
#define GLUT_KEY_F2			2
#define GLUT_KEY_F3			3
#define GLUT_KEY_F4			4
#define GLUT_KEY_F5			5
#define GLUT_KEY_F6			6
#define GLUT_KEY_F7			7
#define GLUT_KEY_F8			8
#define GLUT_KEY_F9			9
#define GLUT_KEY_F10			10
#define GLUT_KEY_F11			11
#define GLUT_KEY_F12			12
/* directional keys */
#define GLUT_KEY_LEFT			100
#define GLUT_KEY_UP				101
#define GLUT_KEY_RIGHT			102
#define GLUT_KEY_DOWN			103
#define GLUT_KEY_PAGE_UP		104
#define GLUT_KEY_PAGE_DOWN		105
#define GLUT_KEY_HOME			106
#define GLUT_KEY_END			107
#define GLUT_KEY_INSERT			108
#endif
```



#### 编辑键

编辑键指 `Ctrl, Alt, Shift` 这类按键，可以使用

```cpp
int glutGetModifiers(void);
```

监测编辑键，此函数只能在键盘和鼠标输入事件绑定的函数中调用。它返回三种参数

* `GLUT_ACTIVE_SHIFT`
* `GLUT_ACTIVE_CTRL`
* `GLUT_ACTIVE_ALT`

注意此函数只有在**按下编辑键之后立即按下其它按键**时才会返回。



如果我们希望用 `Alt + r` 键将红色变量调到最大，例如

```cpp
void processNormalKeys(int key, int x, int y)
{
	if (key == 'r')
    {
        int mod = glutGetModifiers();
        if (mod == GLUT_ACTIVE_ALT)
        {
            red = 1.0;
        }
    }
}
```

再次提醒这里检测先按下 Alt 再按下 r 会将 red 设为 1；检测 Ctrl 键也是类似的，但是如果要检测 Shift 和其它按键的组合，对于字母按键，Shift 会将其转化为大写字母，因此我们只需要直接检测大写字符

```cpp
void processNormalKeys(int key, int x, int y)
{
    // 按下 Shift 后再按下字母按键会获得大写字母
	if (key == 'R')
    {
        red = 1.0;
    }
}
```



通过按位或运算可以监测是否产生了组合按键，例如

```cpp
int mod = glutGetModifiers();
// 监测 Shift 和 Ctrl 是否同时按下
if (mod == (GLUT_ACTIVE_SHIFT | GLUT_ACTIVE_CTRL))
{
    ...
}
```



#### 重复按键

GLUT 提供禁用和启用重复按键检测的函数。

```cpp
int glutSetKeyRepeat(int repeatMode);
```

可选参数有：

* `GLUT_KEY_REPEAT_OFF` 关闭重复输入模式
* `GLUT_KEY_REPEAT_ON` 开启重复输入模式
* `GLUT_KEY_REPEAT_DEFAULT` 重置为默认值

由于此函数对所有窗口有效，因此最好在调用此函数后重置为默认值。



也许我们并不想完全禁止按键重复，而是试图控制重复检测的间隔。GLUT 提供安全途径禁用按键重复的回调函数

```cpp
int glutIgnoreKeyRepeat(int repeatMode);
```

当 repeatMode 为 0 则启用，非零禁用。



#### 松开按键

使用如下函数控制松开按键后的操作，它们的用法和上面按下按键的消息处理方式相同

```cpp
void glutKeyboardUpFunc(void (*func)(unsigned char key,int x,int y));
void glutSpecialUpFunc(void (*func)(int key,int x, int y));
```



### 鼠标消息

#### 按键

GLUT 提供 glutMouseFunc 函数来注册鼠标点击事件的回调函数

```cpp
void glutMouseFunc(void (*func)(int button, int state, int x, int y));
```

第一个参数是鼠标按下的键，有三种可能

* `GLUT_LEFT_BUTTON`
* `GLUT_MIDDLE_BUTTON`
* `GLUT_RIGHT_BUTTON`

第二个参数确定键的状态

* `GLUT_DOWN`
* `GLUT_UP`

当一个事件回调被 `GLUT_DOWN` 状态触发的时候，应用程序会自动断定 `GLUT_UP` 的状态会在鼠标**移离窗体的时候自动触发**。剩下的两个参数是提供了相对于**窗体客户区域左上角**的 `x, y` 坐标。



#### 移动

GLUT 提供鼠标移动监测的能力。有两类移动：活跃移动在鼠标移动且鼠标键按下时触发；静默移动在鼠标移动且鼠标键没有按下时触发。如果开启追踪移动，程序将在鼠标移动的每一帧触发事件。

```cpp
void glutMotionFunc(void (*func) (int x,int y));			// 检测活跃移动
void glutPassiveMotionFunc(void (*func) (int x, int y));	// 检测静默移动
```

处理函数的参数是相对于窗体客户区域左上角的 `x, y` 坐标。



GLUT 可以检测鼠标离开或进入窗体区域的动作。注册函数如下

 ```cpp
 void glutEntryFunc(void (*func)(int state));
 ```

其参数用于确认鼠标是否离开窗口

* `GLUT_LEFT` 离开
* `GLUT_ENTERED` 进入

> >  在微软的窗体中，该函数不太精确，应当避免在微软窗体中使用该特性。



## 附加功能

### 查询系统

GLUT 提供了一个查询系统用于获得各种程序信息。

```cpp
int glutGet(GLenum state);
```

可选的参数非常多

```cpp
/* glutGet parameters. */
#define GLUT_WINDOW_X				100	// 窗口 x 坐标
#define GLUT_WINDOW_Y				101	// 窗口 y 坐标
#define GLUT_WINDOW_WIDTH			102	// 窗口宽度
#define GLUT_WINDOW_HEIGHT			103	// 窗口高度
#define GLUT_ELAPSED_TIME			700	// 从 glInit 开始的时间长度（毫秒）
```

GLUT 中的许多宏都显示了它是哪个函数的参数，例如上面的 glutGet parameters 。要查询更多参数直接跳转到头文件中寻找即可。



### 菜单功能

#### 弹出菜单

GLUT 中的菜单功能较为有限，但是也完成了可以使用的基本功能。通过函数

```cpp
int glutCreateMenu(void (*func)(int value));
```

创建菜单，参数是处理菜单消息的回调函数，value 是菜单 ID，由用户自定义。**注意此函数要最先调用，并且创建菜单后的所有函数操作都是针对新创建的菜单的操作**。



可以每个菜单项绑定不同回调处理函数，也可以多个项绑定同一个处理函数。通过如下函数将菜单与 ID 绑定

```cpp
void glutAddMenuEntry(char *name, int value);
```

其中 name 是菜单名，value 是选中菜单时的返回值，就是上面传递给回调函数的菜单 ID。这个函数会将新添加的菜单项放在前一个菜单项的后面。GLUT 没有中间插入菜单项的功能，因为它只是设计用来作为产品原型的。



当我们有了菜单以后，需要将其绑定到鼠标按键，这是**唯一弹出菜单的方式**。使用如下函数

```cpp
void glutAttachMenu(int button);
```

其中 button 参数就是前面标记鼠标按键的宏。例如

```cpp
void menuFunc(int value)
{

}

// 创建菜单，绑定处理函数
glutCreateMenu(menuFunc);

// 插入菜单项
glutAddMenuEntry("Red", 1);
glutAddMenuEntry("Green", 2);
glutAddMenuEntry("Blue", 3);

// 绑定鼠标右键
glutAttachMenu(GLUT_RIGHT_BUTTON);
```



相对的，如果想要解除与鼠标按键的绑定关系，使用

```cpp
void glutDetachMenu(int button);
```

最后，如果需要释放被使用过的菜单项资源，使用

```cpp
void glutDestroyMenu(int menuIdentifier);
```

其中参数就是对应菜单项的 ID。



#### 子菜单

子菜单也通过 glutCreateMenu 函数来创建，但这次我们需要保留创建菜单的返回值。使用

```cpp
void glutAddSubMenu(
    char *entryName, 	// 子菜单名
    int menuIndex		// 子菜单索引值
);
```

将子菜单添加到主菜单中。例如

```cpp
// 获得菜单索引值
int sub = glutCreateMenu(menuFunc);

// 为子菜单插入项
glutAddMenuEntry("Red", 1);
glutAddMenuEntry("Green", 2);
glutAddMenuEntry("Blue", 3);

// 创建主菜单
glutCreateMenu(menuFunc);

// 添加主菜单项
glutAddMenuEntry("Open", 4);
glutAddMenuEntry("Save", 5);

// 将子菜单插入主菜单
glutAddSubMenu("Color", sub);

// 绑定鼠标右键
glutAttachMenu(GLUT_RIGHT_BUTTON);
```



#### 修改菜单

我们可能会有修改菜单的需求，需要注意当菜单被占用时不能对菜单进行修改，因此以下修改菜单的函数**不能在菜单消息的回调函数中使用**，我们应当通过其它消息，例如键盘消息来修改菜单。



以防万一我们需要先确认菜单是否被占用。GLUT 允许我们注册一个回调函数来检测菜单是否在弹出状态.原型如下:

 ```cpp
 void glutMenuStatusFunc(void (*func)(int status, int x, int y);
 ```

它的参数是回调函数，分别表示菜单状态和相对窗口左上角的位置。例如定义回调函数

```cpp
void processMenuStatus(int status, int x, int y) 
{
    if (status == GLUT_MENU_IN_USE)
		// 菜单被占用的操作
    else
        // 没有占用的操作
}
```



GLUT 允许更改整个菜单

```cpp
int glutGetMenu(void);		// 获取当前菜单的索引
void glutSetMenu(int menu);	// 切换菜单为指定菜单，menu 是要切换的菜单的索引
```



可以调整具体的菜单项信息

```cpp
void glutChangeToMenuEntry(
    int entry, 	// 菜单项的索引值，介于 1 和菜单项总数之间
    char *name, // 新菜单项的名称
    int value	// 菜单项 ID
);
```



我们可以交换两个**子菜单**

```cpp
void glutChangeToSubMenu(
    int entry, 	// 菜单项的索引值，介于 1 和菜单项总数之间
    char *name, // 新菜单项的名
    int menu	// 用于交换的菜单的索引值
);
```

注意交换的是两个子菜单，而不是单纯的菜单项。



可以根据菜单项在菜单中的位置删除菜单项

```cpp
void glutRemoveMenuItem(int entry);
```



可以随时用 glutGet 函数查询当前菜单项的数量

```cpp
int num = glutGet(GLUT_MENU_NUM_ITEMS);
```



### 位图字体

位图字体一般是二维字体，虽然我们会把它放到三维世界，但这些字体没有厚度，也不能渲染和测量，只能翻译。除此之外，字体会一直面向镜头。虽然这个可以看作是潜在的缺点，但另一方面看我们也不用考虑字体的方向问题。



#### 本地位置显示

GLUT 提供位图来显示字体

```cpp
void glutBitmapCharacter(
    void *font, 	// 字体名
    int character	// 字符
);
```

该函数将**一个字符**输出到**栅格位置**。可选的字体名如下

* GLUT_BITMAP_8_BY_13
* GLUT_BITMAP_9_BY_15
* GLUT_BITMAP_TIMES_ROMAN_10
* GLUT_BITMAP_TIMES_ROMAN_24
* GLUT_BITMAP_HELVETICA_10
* GLUT_BITMAP_HELVETICA_12
* GLUT_BITMAP_HELVETICA_18

其中最后的数字表示字体的像素高度。例如显示高度为 18 像素的字符 3 可以通过

```cpp
glutBitmapCharacter(GLUT_BITMAP_HELVETICA_18,'3');
```



所谓栅格位置实际上是字体输出的位置，通过如下函数设置栅格位置

```cpp
void glRasterPos2f(float x, float y);
void glRasterPos3f(float x, float y, float z);
```



我们知道字体的高度，但是字符宽度未知。这时可以通过

```cpp
int glutBitmapWidth(void *font, int character);
```

获得 font 字体下 character 字符的宽度。如果我们进一步想要知道输出字符串的宽度，使用

```cpp
int glutBitmapLength(void *font, char *string);
```

它将返回 font 字体下 string 的宽度。



我们用一个函数封装输出字符串的操作

```cpp
// 用于绘制字符串的函数
void renderBitmapString(float x, float y, float z, void* font, const std::string str)
{
	glRasterPos3f(x, y, z);
	for (int i = 0; i < str.length(); i++)
		glutBitmapCharacter(font, str.at(i));
}
```



#### 固定位置显示

需要注意的是，前面使用的输出字体方式都是在世界坐标系下进行输出，这意味着当我们的镜头移动时，输出的字体可能会被移出窗口。有时我们希望在窗口的**固定位置**持续显示某种信息，例如每秒的帧数，这时候常用的方法是：在透视投影下设置好绘图需要的变换，然后将此变换矩阵压入堆栈；接着切换为二维正交投影，绘制字体；最后再恢复透视投影的变换矩阵。

```cpp
// 假设之前已经完成了图形绘制，并获得了变换的投影矩阵

// 切换为投影模式
glMatrixMode(GL_PROJECTION);

// 先保存原有的投影矩阵
glPushMatrix();

// 当前重置为单位阵
glLoadIdentity();

// 应用二维正交投影
gluOrtho2D(0, w, 0, h);

// 可能需要的变换

// 切换回模式视图
glMatrixMode(GL_MODELVIEW);

// 保存原有的视图矩阵
glPushMatrix();

// 载入单位阵
glLoadIdentity();

// 绘制字体

// 恢复原有的视图矩阵
glPopMatrix();

// 切换为投影模式
glMatrixMode(GL_PROJECTION);

// 恢复设定好的投影矩阵
glPopMatrix();

// 切换回视图模式
glMatrixMode(GL_MODELVIEW);
```



#### 笔画字体

笔画字体用线条生成，可以支持旋转、测量和转化。通过函数

```cpp
void glutStrokeCharacter(void *font, int character)
```

输出笔画字体，这里可选的字体有

* GLUT_STROKE_ROMAN
* GLUT_STROKE_MONO_ROMAN

另外，由于字体由线条生成，可以设定笔画的宽度

```cpp
void glLineWidth (GLfloat width);
```



类似于位图字体，可以获得字体宽度

```cpp
int glutStrokeWidth(void *font, int character);
```



### 游戏模式

GLUT 的游戏模式是为开启高性能全屏渲染而设计的，像弹出菜单和子窗体等功能会因为增强性能而关闭。

```cpp
void glutEnterGameMode(void);	// 进入游戏模式
void glutLeaveGameMode(void);	// 离开游戏模式
```

需要注意每次进入游戏模式都需要重新注册回调函数，并重定义相关信息，例如矩阵栈。



首先要定义游戏模式的设置，例如全屏。该设置包括屏幕分辨率、像素比特数和刷新频率。通过函数

```cpp
void glutGameModeString(const char *string);
```

全屏模式的设置通过字符串指定

```cpp
"WxH:Bpp@Rr" - "800x600:32@24"
```

* W - 屏幕宽度像素
* H - 屏幕高度像素
* Bpp - 像素比特数
* Rr - 垂直刷新速率，单位是赫兹

如果指定模式不可用，设置会被忽略。通常不需要全部指定这些设置，可以按顺序省略，例如

```cpp
"WxH"
":Bpp"
```



GLUT 允许验证模式的可行性，利用

```cpp
int glutGameModeGet(GLenum info);
```

为了检查有效性，可以通过指定宏获取信息

```cpp
if (glutGameModeGet(GLUT_GAME_MODE_POSSIBLE))
{
    // 如果可以进入游戏模式，就启用
    glutEnterGameMode();
}
else 
{
	printf("The select mode is not available\n");
	exit(1);
}
```

实际上该函数还可以返回更多信息

* `GLUT_GAME_MODE_ACTIVE` - 如果程序正运行在游戏模式下，返回非零值；如果在窗体模式下返回 0
* `GLUT_GAME_MODE_POSSIBLE` - 测试游戏模式设置的配置字符串
* `GLUT_GAME_MODE_DISPLAY_CHANGED` - 测试是否真的进入了游戏模式。如果已经处于游戏模式下，可以检测设置是否被改变
* `GLUT_GAME_MODE_WIDTH` - 返回屏幕的宽度
* `GLUT_GAME_MODE_HEIGHT` - 返回屏幕的高度
* `GLUT_GAME_MODE_PIXEL_DEPTH` - 返回当前模式每个像素使用的比特数
* `GLUT_GAME_MODE_REFRESH` - 实际刷新率

最后四个参数只在游戏模式下有意义。



### MFC 编程

Windows GDI 通过设备句柄 Device Context(设备描述表 DC) 来绘图，而 OpenGL 则需要绘制环境 Rendering Context(着色描述表 RC)。RC 不能直接绘图，只能与特定的 DC 联系起来，完成具体的绘图工作。一旦在一个线程中指定了一个当前 RC，在此线程中其后所有的 OpenGL 命令都使用相同的当前 RC。**需要注意，在 MFC 中，所有与绘图有关的代码最好放在已经设置当前 RC 的范围内，防止出现问题。**



我们需要修改几个虚函数

* 在 OnCreate 函数中获取 DC，设置像素格式，从 DC 中创建 RC
* 在 OnDraw 函数中设置当前 RC，完成 OpenGL 绘图后取消当前 RC
* 在 OnSize 函数中设置当前 RC，调整 OpenGL 绘图比例后取消当前 RC
* 在 OnDestroy 函数中释放 RC



典型的示例代码如下

```cpp
#include <afxwin.h>
#include <glut.h>

//自定义框架类
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	HGLRC m_hglrc;	// rc 句柄

public:
    virtual BOOL PreCreateWindow(CREATESTRUCT& cs);
    
	afx_msg int OnCreate(LPCREATESTRUCT lpCreateStruct);
	afx_msg void OnDestroy();
	afx_msg void OnSize(UINT nType, int cx, int cy);
	afx_msg void OnPaint();
	afx_msg BOOL OnEraseBkgnd(CDC* pDC);
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
	ON_WM_DESTROY()
	ON_WM_SIZE()
	ON_WM_PAINT()
	ON_WM_ERASEBKGND()
END_MESSAGE_MAP()

//自定义应用程序类
class CMyWinApp : public CWinApp
{
public:
	virtual BOOL InitInstance()
	{
		// 创建框架类
		CMyFrameWnd* pFrame = new CMyFrameWnd;
		pFrame->Create(NULL, L"Base");
		pFrame->SetWindowPos(NULL, 0, 0, 500, 400, SWP_NOMOVE);

		// 将创建的窗口保存到成员变量 m_pMainWnd 中
		m_pMainWnd = pFrame;
		pFrame->ShowWindow(SW_SHOW);
		pFrame->UpdateWindow();
		return TRUE;
	}
};

//应用程序对象的全局变量
CMyWinApp theApp;


BOOL CMyFrameWnd::PreCreateWindow(CREATESTRUCT& cs)
{
	// 调整窗口风格适应 OpenGL
	cs.style |= WS_CLIPSIBLINGS | WS_CLIPCHILDREN;

	return CFrameWnd::PreCreateWindow(cs);
}


int CMyFrameWnd::OnCreate(LPCREATESTRUCT lpCreateStruct)
{
	if (CFrameWnd::OnCreate(lpCreateStruct) == -1)
		return -1;
	
	PIXELFORMATDESCRIPTOR pfd =
	{
		sizeof(PIXELFORMATDESCRIPTOR),	// pfd 结构大小
		1,								// 版本号
		PFD_DRAW_TO_WINDOW |			// 支持窗口绘图
		PFD_SUPPORT_OPENGL |			// 支持 OpenGL
		PFD_DOUBLEBUFFER,				// 双缓冲模式
		PFD_TYPE_RGBA,					// RGBA 绘图模式
		24,								// 24 位颜色深度
		0, 0, 0, 0, 0, 0,				// 忽略颜色位
		0,								// 没有非透明度缓存
		0,								// 忽略移位位
		0,								// 无累计缓存
		0, 0, 0, 0,						// 忽略累计位
		32,								// 32 位深度缓存
		0,								// 无模板缓存
		0,								// 无辅助缓存
		PFD_MAIN_PLANE,					// 主层
		0,								// 保留
		0, 0, 0							// 忽略层，可见性和损毁掩模
	};

	CClientDC dc(this);
	int pf = ChoosePixelFormat(dc.m_hDC, &pfd);
	BOOL rt = SetPixelFormat(dc.m_hDC, pf, &pfd);

	// 获得当前 RC
	m_hglrc = wglCreateContext(dc.m_hDC);

	return 0;
}


void CMyFrameWnd::OnSize(UINT nType, int cx, int cy)
{
	CFrameWnd::OnSize(nType, cx, cy);

	GLsizei w = cx;
	GLsizei h = cy;

	// 防止宽度为 0
	if (h == 0)
		h = 1;

	CClientDC dc(this);
	wglMakeCurrent(dc.m_hDC, m_hglrc);

	// 调整视口
	glViewport(0, 0, w, h);

	// 取消 RC
	wglMakeCurrent(NULL, NULL);
}


void CMyFrameWnd::OnPaint()
{
	CPaintDC dc(this); // device context for painting
					   // TODO: 在此处添加消息处理程序代码
					   // 不为绘图消息调用 CFrameWnd::OnPaint()

	// 设置当前 RC
	wglMakeCurrent(dc.m_hDC, m_hglrc);

	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

	glColor3f(1, 0, 0);

	glBegin(GL_POLYGON);
		glVertex3f(0.5f, 0.5f, 0.0f);
		glVertex3f(0.5f, -0.5f, 0.0f);
		glVertex3f(-0.5f, -0.5f, 0.0f);
		glVertex3f(-0.5f, 0.5f, 0.0f);
	glEnd();

	// 刷新缓冲
	glFlush();

	// 与 DC 交换缓冲，这是图像显示的最后一步
	SwapBuffers(dc.m_hDC);

	// 取消 RC
	wglMakeCurrent(NULL, NULL);
}


BOOL CMyFrameWnd::OnEraseBkgnd(CDC* pDC)
{
	// 禁用重绘背景，防止闪烁
	return TRUE;
}


void CMyFrameWnd::OnDestroy()
{
	CFrameWnd::OnDestroy();

	if (wglGetCurrentContext() != NULL)
		wglMakeCurrent(NULL, NULL);

	// 销毁 RC
	if (m_hglrc != NULL)
	{
		wglDeleteContext(m_hglrc);
		m_hglrc = NULL;
	}
}
```



## [Assimp](https://assimp.sourceforge.net/lib_html/usage.html) 库

Assimp 库支持大量三维数据文件格式的读取，需要下载源码编译产生动态库和静态库后使用。



### 编译运行

下载源码后，用 CMake 编译，选择好源码和 build 文件夹路径后

![](image-20230613210308171.png)

配置 Configure 项目为 Win32 项目，然后生成 Genreate 后，编译生成 INSTALL 方案即可完成安装。



基本代码示例如下

```cpp
// 注意这些头文件要在 afxwin 之前导入，不然会由于 min 重定义导致问题
#include <assimp/Importer.hpp>      // C++ importer interface
#include <assimp/scene.h>           // Output data structure
#include <assimp/postprocess.h>     // Post processing flags

// 导入动态库，注意 dll 文件要放在工作目录下，不然找不到
#pragma comment(lib, "assimp-vc140-mt.lib")

// 定义导入函数
bool Import(const std::string& pFile)
{
    // 定义导入器
	Assimp::Importer importer;
	const aiScene* scene = importer.ReadFile(pFile,
		aiProcess_CalcTangentSpace |
		aiProcess_Triangulate |
		aiProcess_JoinIdenticalVertices |
		aiProcess_SortByPType);

	// 读取失败
	if (!scene)
	{
        // MyLog 是自定义的日志类，用于输出信息
		MyLog::log(importer.GetErrorString());
		return false;
	}
    // 相关操作

	return true;
}
```

注意在“链接器-输入”中添加 assimp-vc140-mt.lib 依赖项。



### 基本结构

这是读取数据的基类，我们将三维数据读入到此场景类中。场景类包含材质、光照、相机视角、网格、纹理和动画等信息，为了安全起见，应当先调用相关函数判断这些信息是否存在，然后再读取。

```cpp
bool HasAnimations () const;
bool HasCameras () const;
bool HasLights () const;
bool HasMaterials () const;
bool HasMeshes () const;
bool HasTextures () const;
```

通过成员变量 mMeshes 和 mMaterials 获得网格指针，它们是指针数组，通过 mNumMeshes 和 mNumMaterials 长度遍历数组取出对应的网格指针。



网格类 Mesh 保存了顶点、面、顶点法向、切向，首先要检查变量的存在性

```cpp
unsigned int GetNumColorChannels () const;			// 获得颜色通道数
unsigned int GetNumUVChannels () const; 			// 获得 uv 通道数
bool HasBones () const;								// 判断是否有骨架
bool HasFaces () const;								// 是否有面
bool HasNormals () const;							// 是否有法向
bool HasPositions () const;							// 是否有坐标
bool HasTangentsAndBitangents () const;				// 是否有切向和双切向，后者应该是切向旋转 90 度得到的
bool HasTextureCoords (unsigned int pIndex) const;	// 是否有纹理坐标
bool HasVertexColors (unsigned int pIndex) const;	// 顶点是否有颜色
```

常用基本信息包括

* mName 网格名
* mVertices 顶点数组
* mNormals 法向数组
* mFaces 面数组
* mColors 颜色指针数组
* mTextureCoord 纹理坐标数组

同样的，需要用 mNumVertices, mNumFaces 作为长度遍历，其中顶点、法向、颜色一一对应，长度相同。在计算顶点数时，是对每个面上的顶点累计，所以总数会大于实际数量。

```cpp
// 输出网格名
aiMesh* mesh = scene->mMeshes[0];
printLog(mesh->mName.C_Str());

// 检查是否有位置
if (mesh->HasPositions())
printLog("有位置");

// 检查顶点是否有法向
if (mesh->HasNormals())
printLog("有法向");

// 顶点操作
for (int i = 0; i < mesh->mNumVertices; i++)
{
	// 输出顶点坐标
	printLogV3(mesh->mVertices[i]);

	// 检查顶点是否有颜色
	if (mesh->HasVertexColors(i))
		printLog("有颜色");

	// 检查顶点是否有纹理坐标
	if (mesh->HasTextureCoords(i))
		printLog("有纹理");
}

// 面操作
for (int i = 0; i < mesh->mNumFaces; i++)
{
	// 获得面对象
	aiFace face = mesh->mFaces[i];

	// 输出面顶点的索引
	mLog.m_fp << i << " (";
	for (int j = 0; j < face.mNumIndices; j++)
		mLog.m_fp << face.mIndices[j] << ",";
	mLog.m_fp << ")\n";
}
```



### 读取网格

使用自定义的网格类型，读取 assimp 库模型的基本代码如下

```cpp
// 注意这些头文件要在 afxwin 之前导入，不然会由于 min 重定义导致问题
#include <assimp/Importer.hpp>      // C++ importer interface
#include <assimp/scene.h>           // Output data structure
#include <assimp/postprocess.h>     // Post processing flags

// 导入动态库，注意 dll 文件要放在工作目录下，不然找不到
#pragma comment(lib, "assimp-vc140-mt.lib")

#include "Mesh.h"
#include "MyLog.h"

using namespace Mesh;

// 全局网格变量
HalfEdgeMesh mesh;

// 读取函数
bool Import(const std::string& pFile)
{
	Assimp::Importer importer;
	const aiScene* scene = importer.ReadFile(pFile,
		aiProcess_CalcTangentSpace |
		aiProcess_Triangulate |
		aiProcess_JoinIdenticalVertices |
		aiProcess_SortByPType);

	// 读取失败
	if (!scene)
	{
		printLog(importer.GetErrorString());
		return false;
	}
	printLog("导入成功");

	// 导入网格信息，直接导入所有网格，因为 HalfEdgeMesh 可以储存多个离散网格，只是在执行特殊算法时效率较低
	for (int k = 0; k < scene->mNumMeshes; k++)
	{
		aiMesh* mesh2 = scene->mMeshes[k];

		// 载入顶点和法向
		for (int i = 0; i < mesh2->mNumVertices; i++)
		{
			float x = mesh2->mVertices[i].x;
			float y = mesh2->mVertices[i].y;
			float z = mesh2->mVertices[i].z;
			vertex3f* v = mesh.createVertex({ x,y,z });

			// 如果有法向，就添加顶点法向
			if (mesh2->HasNormals())
			{
				float x = mesh2->mNormals[i].x;
				float y = mesh2->mNormals[i].y;
				float z = mesh2->mNormals[i].z;
				v->normal = { x, y, z };

				// 现在有法向
				mesh.m_hasVertexNormal = true;
			}
		}

		// 载入面，通常模型中的面是连通的，所以可以放心创建
		for (int i = 0; i < mesh2->mNumFaces; i++)
		{
			// 获得面对象
			aiFace face = mesh2->mFaces[i];

			// 输出面顶点的索引
			int i0 = face.mIndices[0];
			int i1 = face.mIndices[1];
			int i2 = face.mIndices[2];

			// 获取对应索引的顶点创建面
			auto vList = mesh.m_vList;
			mesh.createConnectFace(vList[i0], vList[i1], vList[i2]);
		}
	}

	return true;
}
```

