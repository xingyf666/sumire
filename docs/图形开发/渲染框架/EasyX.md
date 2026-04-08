# EasyX

## 安装配置

### 基本介绍

EasyX Graphics Library 是针对 Visual C++ 的免费绘图库，支持 VC 6.0 ~ VC 2022，简单易用，学习成本极低，应用领域广泛。目前已有许多大学将 EasyX 应用在教学当中。

```embed
title: "EasyX Graphics Library for C++"
image: "https:// easyx.cn/Content/images/logo.svg"
description: "EasyX Graphics Library 是针对 Visual C++ 的绘图库，支持 VC6.0 ~ VC2019，简单易用，学习成本极低，应用领域广泛。目前已有许多大学将 EasyX 应用在教学当中。"
url: "https:// easyx.cn/"
```

打开安装程序，点击在 Visual C++ 中直接安装即可。



### 使用框架

启动 Visual C++ 创建一个空的控制台项目（`Win32 Console Application`），然后添加一个新的代码文件 (`.cpp`)，并引用 `graphics.h` 头文件

```cpp
#include <graphics.h>		// 引用图形库头文件
#include <conio.h>
int main()
{
	initgraph(640, 480);	// 创建绘图窗口，大小为 640x480 像素
	circle(200, 200, 100);	// 画圆，圆心(200, 200)，半径 100
	_getch();				// 按任意键继续
	closegraph();			// 关闭绘图窗口
	return 0;
}
```

当然，EasyX 也可以在 `Win32 Application` 项目上使用。



## 基本概念

### 颜色

使用 RGB 宏合成颜色，实际上合成出的颜色是一个 16 进制的整数

```cpp
RGB(red,green,blue); // 每个颜色部分取值范围为0~255
```



### 坐标

在 EasyX 中，坐标分两种：物理坐标和逻辑坐标。



**物理坐标**

* 物理坐标是描述设备的坐标体系
* **坐标原点在设备的左上角，X 轴向右为正，Y 轴向下为正**，度量单位是像素
* 坐标原点、坐标轴方向、缩放比例都不能改变



**逻辑坐标**

* 逻辑坐标是在程序中用于绘图的坐标体系
* 坐标默认的原点在窗口的左上角，X 轴向右为正，Y 轴向下为正，度量单位是点
* 默认情况下，逻辑坐标与物理坐标是一一对应的，一个逻辑点等于一个物理像素

在本手册中，凡是没有注明的坐标，均指逻辑坐标。



### 设备

设备是指绘图表面。在 EasyX 中，设备分两种，一种是默认的绘图窗口，另一种是 IMAGE 对象。通过 SetWorkingImage 函数可以设置当前用于绘图的设备；设置当前用于绘图的设备后，所有的绘图函数都会绘制在该设备上。



## 窗口函数

### Initgraph

用于初始化绘图窗口（创建窗口）

```cpp
initgraph(int width, int height, int flag = NULL);
```

* width 指定宽度
* height 指定高度
* flag 窗口样式



### Closegraph

关闭绘图窗口

```cpp
closegraph(); 
```



### Cleardevice

使用当前背景色清空绘图设备

```cpp
cleardevice();
```



### Setbkmode

设置绘制图像和文字填充时的背景模式

```cpp
setbkmode(int mode);
```

模式

| 值          | 描述                         |
| ----------- | ---------------------------- |
| OPAQUE      | 背景用当前背景色填充（默认） |
| TRANSPARENT | 背景是透明的                 |



### Getbkcolor

获取背景颜色

```
getbkcolor();
```



### Setbkcolor

设置背景颜色

```cpp
setbkcolor(COLORREF color); 
```



### MessageBox

弹出窗口函数，返回用户操作

```cpp
int MessageBox(
    HWND hWnd,
    LPCTSTR lpText,
    LPCTSTR lpCaption,
    UINT uType
);
```



**常用属性**

系统默认图标，可在消息框上显示

* 错误 MB_ICONHAND, MB_ICONSTOP, MB_ICONERROR
* 询问 MB_ICONQUESTION
* 警告 MB_ICONEXCLAMATION, MB_ICONWARNING
* 信息 MB_ICONASTERISK, MB_ICONINFORMATION



**按钮的形式**

* MB_OK  默认
* MB_OKCANCEL 确定取消
* MB_YESNO 是否
* MB_YESNOCANCEL 是否取消



**返回值**

* IDCANCEL 取消被选（取消）
* IDNO 否被选（否）
* IDOK 确定被选（确定）
* IDYES 是被选（是）



### GetHWnd

获取窗口句柄

```cpp
HWND hwnd = GetHWnd();
```



### SetWindowText

设置窗口标题

```cpp
SetWindowText(hwnd, "Hello"); 
```



使用范例

```cpp
int isok = MessageBox(hwnd, "恭喜！", "提示", MB_OKCANCEL);
if (isok == IDOK)
{
    cout << "OK" << endl;
}
else if (isok == IDCANCEL)
{
    cout << "CANCEL" << endl;
}

// 退出窗口
HWND hWnd = GetHWnd();
SendMessage(hWnd, WM_CLOSE, NULL, NULL);
```



## 绘图函数

绘图函数分为无填充，有边框填充，无边框三种

**前缀**

* fill 边框填充
* solid 无边框填充

```cpp
circle(); // 无填充
fillcircle(); // 有边框填充
solidcircle(); // 无边框填充
```

| 绘图函数  |   图形   |
| :-------: | :------: |
|  circle   |    圆    |
|  ellipse  |   椭圆   |
|    pie    |   扇形   |
|  polygen  |  多边形  |
| rectangle |   矩形   |
| roundrect | 圆角矩阵 |
|   line    |    线    |
| putpixel  |    点    |



### Setfillcolor

设置填充颜色

```cpp
setfillcolor(COLORREF color);
```



### Setlinecolor

设置划线颜色

```cpp
setlinecolor(COLORREF color); 
```



### Getfillcolor

获取填充颜色

```cpp
getfillcolor(); 
```



### Getlinecolor

获取划线颜色

```cpp
getlinecolor(); 
```



## 文字输出

由于文字输出使用宽字符串，因此需要在字符串前加 "L" 前缀，也可以直接在 "项目-属性-配置属性-常规-字符集-改为多字节字符集"



### Settextcolor

设置文字颜色

```cpp
settextcolor(COLORREF color); 
```



### Outtextxy

在指定位置输出文字

```cpp
outtextxy(int x, int y, LPCTSTR str);
outtextxy(int x, int y, TCHAR c);
```



### Textwidth

得到文字像素宽度

```cpp
int width = textwidth(arr);
```



### Textheight

得到文字像素高度

```cpp
int height = textheight(arr); 
```



### Drawtext

这个函数用于在指定区域内以指定格式输出字符串。

```cpp
// 矩形结构
typedef struct tagRECT
{
    LONG    left;
    LONG    top;
    LONG    right;
    LONG    bottom;
} RECT, *PRECT, NEAR *NPRECT, FAR *LPRECT;

int drawtext(
	LPCTSTR str, // 待输出的字符串
	RECT* pRect, // 矩形指针
	UINT uFormat // 指定格式化输出文字的方法
);

int drawtext(
	TCHAR c,     // 待输出的字符
	RECT* pRect, // 矩形指针
	UINT uFormat // 指定格式化输出文字的方法
);
```



## 图像输出

### 操作流程

$$
定义图像 \to 加载图片 \to 输出图片
$$

```cpp
IMAGE img; 						// 定义一个图像对象
loadimage(&img, image_path); 	// 加载图片
putimage(int x, int y, &img); 	// 输出图片
```



### 基本函数

#### Getimage

从当前绘图设备获取图像

```cpp
void getimage(
	IMAGE* pDstImg,		// 保存图像的 IMAGE 对象指针
	int srcX,			// 要获取图像区域左上角 x 坐标
	int srcY,			// 要获取图像区域的左上角 y 坐标
	int srcWidth,		// 要获取图像区域的宽度
	int srcHeight		// 要获取图像区域的高度
);
```



#### Loadimage

```cpp
// 从图片文件获取图像 (bmp/gif/jpg/png/tif/emf/wmf/ico)
void loadimage(
	IMAGE* pDstImg,			// 保存图像的 IMAGE 对象指针
	LPCTSTR pImgFile,		// 图片文件名
	int nWidth = 0,			// 图片的拉伸宽度
	int nHeight = 0,		// 图片的拉伸高度
	bool bResize = false	// 是否调整 IMAGE 的大小以适应图片
);

// 从资源文件获取图像 (bmp/gif/jpg/png/tif/emf/wmf/ico)
void loadimage(
	IMAGE* pDstImg,			// 保存图像的 IMAGE 对象指针
	LPCTSTR pResType,		// 资源类型
	LPCTSTR pResName,		// 资源名称
	int nWidth = 0,			// 图片的拉伸宽度
	int nHeight = 0,		// 图片的拉伸高度
	bool bResize = false	// 是否调整 IMAGE 的大小以适应图片
);
```



#### Putimage

绘制图像

```cpp
void putimage(
	int dstX,				// 绘制位置的 x 坐标
	int dstY,				// 绘制位置的 y 坐标
	IMAGE *pSrcImg,			// 要绘制的 IMAGE 对象指针
	DWORD dwRop = SRCCOPY	// 三元光栅操作码
);

void putimage(
	int dstX,				// 绘制位置的 x 坐标
	int dstY,				// 绘制位置的 y 坐标
	int dstWidth,			// 绘制的宽度
	int dstHeight,			// 绘制的高度
	IMAGE *pSrcImg,			// 要绘制的 IMAGE 对象指针
	int srcX,				// 绘制内容在 IMAGE 对象中的左上角 x 坐标
	int srcY,				// 绘制内容在 IMAGE 对象中的左上角 y 坐标
	DWORD dwRop = SRCCOPY	// 三元光栅操作码
);
```

三元光栅操作码（即位操作模式），支持全部的 256 种三元光栅操作码，常用的几种如下：

| 值          | 含义                                                  |
| ----------- | ----------------------------------------------------- |
| DSTINVERT   | 目标图像 = NOT 目标图像                               |
| MERGECOPY   | 目标图像 = 源图像 AND 当前填充颜色                    |
| MERGEPAINT  | 目标图像 = 目标图像 OR (NOT 源图像)                   |
| NOTSRCCOPY  | 目标图像 = NOT 源图像                                 |
| NOTSRCERASE | 目标图像 = NOT (目标图像 OR 源图像)                   |
| PATCOPY     | 目标图像 = 当前填充颜色                               |
| PATINVERT   | 目标图像 = 目标图像 XOR 当前填充颜色                  |
| PATPAINT    | 目标图像 = 目标图像 OR ((NOT 源图像) OR 当前填充颜色) |
| SRCAND      | 目标图像 = 目标图像 AND 源图像                        |
| SRCCOPY     | 目标图像 = 源图像                                     |
| SRCERASE    | 目标图像 = (NOT 目标图像) AND 源图像                  |
| SRCINVERT   | 目标图像 = 目标图像 XOR 源图像                        |
| SRCPAINT    | 目标图像 = 目标图像 OR 源图像                         |



### 透明背景图

通过源图像与掩码图像的叠加来消去边缘部分，实现透明背景

```cpp
IMAGE img[2];

loadimage(&img[0], "sun1.png", 100, 100); // 掩码图像
loadimage(&img[1], "sun0.png", 100, 100); // 源图像

putimage(0, 0, &img[0], NOTSRCERASE); // 掩码图与背景或取反
putimage(0, 0, &img[1], SRCINVERT); // 源图像与背景异或
```

**直观理解：由于按位运算是依次对某一位做运算，因此只需要考察一位数字的变化**

```cpp
// 不妨假设白色为1，黑色为0

// 需要变成透明的部分，需要显示最初的背景色
// 背景色为a，~a表示反a，掩码此部分是黑色0，源图像此部分是白色1
~ (a | 0); // -> ~a 或取反
(~a) ^ 1; // -> a 异或
// 结果仍为背景色

// 需要显示源图像的部分
// 背景色为a，源图像此部分为b，掩码此部分是白色1
~ (a | 1); // -> 0 或取反（可以看到第一部分与a无关）
0 ^ b; // -> b 异或
// 结果为源图像
```



另一种选择：直接通过图像处理来解决问题

```cpp
// 绘图函数，补充透明度 AA
inline void drawAlpha(IMAGE* image, int x, int y, int width, int height, int pic_x, int pic_y, double AA = 1)
{
	// 变量初始化
	DWORD* dst = GetImageBuffer();			// GetImageBuffer()函数，用于获取绘图设备的显存指针，EASYX自带
	DWORD* draw = GetImageBuffer();
	DWORD* src = GetImageBuffer(image);		// 获取picture的显存指针
	int imageWidth = image->getwidth();		// 获取图片宽度
	int imageHeight = image->getheight();	// 获取图片宽度
	int dstX = 0;							// 在 绘图区域 显存里像素的角标
	int srcX = 0;							// 在 image 显存里像素的角标

	// 实现透明贴图 公式： Cp=αp*FP+(1-αp)*BP ， 贝叶斯定理来进行点颜色的概率计算
	for (int iy = 0; iy < height; iy++)
	{
		for (int ix = 0; ix < width; ix++)
		{
			// 防止越界
			if (ix + pic_x >= 0 && ix + pic_x < imageWidth && iy + pic_y >= 0 && iy + pic_y < imageHeight &&
				ix + x >= 0 && ix + x < WindowWidth && iy + y >= 0 && iy + y < WindowHeight)
			{
				// 获取像素角标
				int srcX = (ix + pic_x) + (iy + pic_y) * imageWidth;
				dstX = (ix + x) + (iy + y) * WindowWidth;

				int sa = ((src[srcX] & 0xff000000) >> 24) * AA;			// 0xAArrggbb;AA是透明度
				int sr = ((src[srcX] & 0xff0000) >> 16);				// 获取 RGB 里的 R
				int sg = ((src[srcX] & 0xff00) >> 8);					// G
				int sb = src[srcX] & 0xff;								// B

				// 设置对应的绘图区域像素信息
				int dr = ((dst[dstX] & 0xff0000) >> 16);
				int dg = ((dst[dstX] & 0xff00) >> 8);
				int db = dst[dstX] & 0xff;
				draw[dstX] = ((sr * sa / 255 + dr * (255 - sa) / 255) << 16)  // 公式： Cp=αp*FP+(1-αp)*BP  ； αp=sa/255 , FP=sr , BP=dr
					| ((sg * sa / 255 + dg * (255 - sa) / 255) << 8)         // αp=sa/255 , FP=sg , BP=dg
					| (sb * sa / 255 + db * (255 - sa) / 255);              // αp=sa/255 , FP=sb , BP=db
			}
		}
	}
}
```



## 批量绘图

在设备上不断进行绘图操作时，会产生闪屏现象

* `BeginBatchDraw()` 执行后不会输出绘图结果，直到执行 `FlushBatchDraw()` 或 `EndBatchDraw()`
* `FlushBatchDraw()` 执行未完成的绘图任务
* `EndBatchDraw()` 结束批量绘图

```cpp
// 循环绘图
while (...)
{
	BeginBatchDraw(); // 开始批量绘图
	// 放置绘图代码
	FlushBatchDraw(); // 执行未完成的绘图任务
}

EndBatchDraw(); // 结束批量绘图
```



## 消息处理函数

### 消息类型

**鼠标消息**

* WM_MOUSEMOVE 鼠标移动
* WM_LBUTTONDOWN 按下鼠标左键
* WM_LBUTTONUP 抬起鼠标左键
* WM_LBUTTONDBLCLK 双击鼠标左键
* WM_RBUTTONDOWN 按下鼠标右键
* WM_RBUTTONUP 抬起鼠标右键
* WM_RBUTTONDBLCLK 双击鼠标右键
* WM_MOUSEWHEEL 鼠标滚轮



### Peekmessage

这个函数用于获取一个消息，并**立即返回**

```cpp
bool peekmessage(
    ExMessage *msg,        // 指向消息结构体ExMessage的指针，用来保存获取到的消息
    BYTE filter = -1,      // 指定要获取的消息范围，默认 -1 获取所有类别的消息
    bool removemsg = true  // 在 peekmessage 处理完消息后，是否将其从消息队列中移除
);
```

**filter 参数**

| 标志      | 描述     |
| --------- | -------- |
| EM_MOUSE  | 鼠标消息 |
| EM_KEY    | 按键消息 |
| EM_CHAR   | 字符消息 |
| EM_WINDOW | 窗口消息 |

* 如果获取到了消息，返回 true
* 如果当前没有消息，返回 flase



### Getmessage

这个函数用于获取一个消息；**如果当前消息队列中没有，就一直等待**；使用方法与 peekmessage 类似

```cpp
ExMessage getmessage(BYTE filter = -1);
void getmessage(ExMessage *msg, BYTE filter = -1);
```



## 键盘消息

### _kbhit

返回是否有键盘按下

```cpp
bool _kbhit();
```



### _getch

获取按下的键的虚拟键值，需要头文件 `conio.h`，如果没有输入会一直等待

```cpp
// 需要使用返回值判断
char key = _getch();  
// 虚拟键值（上：72，下：80，左：75，右：77）
```

* 与非 ASCII 表字符的按键比较
* 与字母比较直接写字母

```cpp
// 判断是否有键按下
if (_kbhit())
{
    // 获取按键，不能同时获取多个按键操作
	char key = _getch();

	switch (key)
	{
	case 72:
	case 'w':
		y -= 5; break;
	case 80:
	case 's':
		y += 5; break;
	case 75:
	case 'a':
		x -= 5; break;
	case 77:
	case 'd':
		x += 5; break;
	default:
		break;
	}
}
```



### GetAsyncKeyState

需要传入一个键值，如果按下返回真，需要头文件 `windows.h`，但 EasyX 包含了该头文件，无需再次包含（更好使用）

```cpp
GetAsyncKeyState(int);
// （上：VK_UP，下：VK_DOWN，左：VK_LEFT，右：VK_RIGHT）
// 键盘字母使用大写
```

直接获取键是否按下，可以得到多个按键操作

```cpp
if (GetAsyncKeyState(VK_UP) || GetAsyncKeyState('W'))
{
    y -= 5;
}
if (GetAsyncKeyState(VK_DOWN) || GetAsyncKeyState('S'))
{
    y += 5;
}
if (GetAsyncKeyState(VK_LEFT) || GetAsyncKeyState('A'))
{
    x -= 5;
}
if (GetAsyncKeyState(VK_RIGHT) || GetAsyncKeyState('D'))
{
    x += 5;
}
```



## 播放音乐

需要包含 `mmsystem.h` 头文件，还需加载静态库 `winmm.lib` 。

> > 使用 `Mp3tag` 去除播放音乐的封面，否则无法播放；确保路径中没有空格，否则加载会出问题。

```cpp
// 调用该函数播放音乐
MCIERROR mciSendString(
	LPCTSTR lpszCommand, 　　　// MCI命令字符串
	LPTSTR　lpszReturnString,　// 存放反馈信息的缓冲区
	UINT　　cchReturn, 　　　　 // 缓冲区的长度
	HANDLE　hwndCallback 　　　// 回调窗口的句柄，一般为NULL
); 
```

**MCI 命令字符串**

* open　打开设备
* close　关闭设备
* play　开始设备播放
* stop　停止设备的播放或记录
* record　开始记录
* save　保存设备内容
* pause　暂停设备的播放或记录
* resume　恢复暂停播放或记录的设备
* status   获取设备播放状态

```cpp
#include <graphics.h>
#include <conio.h>
#include <mmsystem.h> // 包含多媒体设备接口头文件
#pragma comment(lib, "winmm.lib") // 加载静态库

// 播放音乐
void BGM()
{
    // 打开音乐（支持多种音乐格式）
    mciSendString("open ./music_name.mp3", 0, 0, 0);
    // 播放音乐
    mciSendString("play ./music_name.mp3", 0, 0, 0);
    
    // 获取设备播放信息
    char arr[255];
	mciSendString("status ./music_name.mp3 mode", arr, 255, 0);
	cout << arr << endl;
    // 返回三种状态：playing stopped paused
    
    // 获取长度
    char arr[255];
	mciSendString("status ./music_name.mp3 length", arr, 255, 0);
	cout << arr << endl;
    // 返回毫秒数
    
    // 获取音量
    char arr[255];
	mciSendString("status ./music_name.mp3 volume", arr, 255, 0);
	cout << arr << endl;
    // 返回1-1000
    
    // 设置音量
	mciSendString("setaudio ./music_name.mp3 volume to 数值", 0, 0, 0);
    
    // 获取播放位置(毫秒)
    char arr[255];
    mciSendString ("status ./music_name.mp3 position", arr, 255, 0);
    cout << arr << endl;
    
   	// 设置播放位置(毫秒)
	mciSendString ("seek ./music_name.mp3 to ", t, 0, 0);           // 定位到t位置
	mciSendString ("seek ./music_name.mp3 to start", 0, 0, 0);      // 定位到开头位置
	mciSendString ("play ./music_name.mp3 FROM t1 to t2", 0, 0, 0); // t1是起始位置，t2是停止位置
	mciSendString ("seek ./music_name.mp3 to end", 0, 0, 0);        // 定位到最后位置
    
    // 取别名alias，重复播放repeat
    // mciSendString("open ./music_name.mp3 alias BGM", 0, 0, 0);
    // mciSendString("play BGM repeat", 0, 0, 0);
    
    // 关闭音乐
    mciSendString("close BGM", 0, 0, 0);
}
```



## 定时器

返回时间的函数

```cpp
DWORD GetTickCount(); // 返回系统启动以来经过的毫秒数，类型为DWORD

// 创建一个定时器函数
#include <ctime>

bool Timer(int ms)
{
    static DWORD t1, t2;
    if (t1 - t2 > ms)
    {
        t1 = t2;
        return true;
    }
    t2 = clock();
    return false;
}
```
