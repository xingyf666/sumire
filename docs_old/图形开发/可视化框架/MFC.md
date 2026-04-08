# MFC

## 基本知识

### SAL批注

用于有助于识别可能未初始化的值，无效的空指针将以正式准确的方式使用

SAL定义了四种基本类型的参数

* `_In_` 数据将床底给被调用的函数，并视为只读
* `_Inout_` 可用数据传递到函数中，并可能被修改
* `_Out_` 调用方只为所调用的函数提供空间来写入，被调用函数将数据写入该空间
* `_Outptr_` 类似于 `_Out_`，被调用函数返回一个指针



### 数据类型

句柄：一个 4 字节的数值，用于表示程序中不同类和相同类中的不同实例；应用程序通过句柄访问相同对象的信息

| 数据类型 |                             说明                             |
| :------: | :----------------------------------------------------------: |
|   WORD   |                        16位无符号整型                        |
|   LONG   |                        32位有符号整型                        |
|  DWORD   |                        32位无符号整型                        |
|  HANDLE  |                             句柄                             |
|   UINT   |                        32位无符号整型                        |
|   BOOL   |                            布尔值                            |
|  LPTSTR  |                  指向字符串的32位指针 CHAR*                  |
| LPCTSTR  |             指向字符串常量的32位指针 const CHAR*             |
|  TCHAR   | 当定义了 UNICODE 宏，则表示 const wchar_t *，否则表示 const char * |
| LPCWSTR  |                     宽字符指针 wchar_t *                     |

当在项目中使用 unicode 字符集，则默认定义了 UNICODE 宏，使用多字节字符集则不会有这一宏；如果需要添加 UNICODE 宏，应该在 `windows.h` 头文件之前定义该宏

```cpp
TCHAR* tcText = _TEXT("Hello"); // 使用_TEXT转换，应对两种不同的情况
```



### 字符编码

非常令人头疼的问题就是编程过程中的字符编码。我们在此对不同的窗口函数和其接受的参数加以分别：对于一些以 W 结尾的函数或变量，例如 `CreateWindowW, WNDCLASSW` 等，它们接收的是宽字符版本，这些是在 Unicode 字符集下使用的；相对的，以 A 结尾的函数或变量，例如 `CreateWindowA. WNDCLASSA` 等，它们接收的是普通字符版本，这些是在多字节字符集下使用的。下面这些是宽字符内容：

```cpp
LPCWSTR pName = L"123456";
WCHAR Name[100]={0};
```

这些是多字节字符的内容：

```cpp
LPCSTR pName = "123456";
CHAR Name[100]={0};
```

值得一提的是，在 VS 中默认使用的是 Unicode 字符集，其给出的模板使用宽字符的版本，因此如果要改为多字节字符，就应当将其中的函数和变量重新设置。



### 宽字符串

宽字符串的使用一般要导入 `windows.h` 头文件，其每个字符占2个字节

```cpp
// 定义宽字符串
wchar_t * wcText = L"Hello";
wchar_t wc[] = L"Hello";
```

可以实现宽字符和窄字符之间的转换

```c++
// 宽字符转窄字符
std::string ws2s(const std::wstring& ws)
{
	// 创建临时内存并清空
	char* dest = new char[ws.size() + 1];
	memset(dest, 0, ws.size() + 1);

	// 转换函数
	size_t i;
	wcstombs_s(&i, dest, ws.size() + 1, ws.c_str(), ws.size());

	// 获得结果并释放内存
	std::string result = dest;
	delete[] dest;
	return result;
}

// 窄字符转宽字符
std::wstring s2ws(const std::string& s)
{
	wchar_t* dest = new wchar_t[s.size() + 1];

	// 转换函数
	size_t i;
	mbstowcs_s(&i, dest, s.size() + 1, s.c_str(), s.size());

	// 获得结果并释放内存
	std::wstring result = dest;
	delete[] dest;
	return result;
}
```



### 字符串函数

#### wcslen

获取宽字符串长度

```cpp
size_t wcslen(
    wchar_t const* _String
);
```



#### wprintf_s

打印输出宽字符，建议使用 wprintf_s 标准，该标准会进行输入检查，更加安全

```cpp
int wprintf_s(
    wchar_t const* const _Format,
    ...
);
```



#### swprintf_s

输出到宽字符串中

```cpp
int swprintf_s(
    wchar_t*       const _Buffer,
    size_t         const _BufferCount,
    wchar_t const* const _Format,
    ...
);
```



#### strltol

将输入的字符串转为整型

```cpp
long strtol(
    char const* _String,	// 字符串
    char**      _EndPtr,	// 返回字符串结尾指针
    int         _Radix		// 转换基数（如10进制）
);
```



#### ltoa

将整型转换为字符串

```cpp
char *ltoa(
    long value,		// 整型
    char *string,	// 字符串指针
    int radix		// 转换基数（如10进制）
);
```



#### atoi

将字符串转换为整型

```cpp
int atoi(char const* _String);
```



#### _ttoi

此函数在 Unicode 和 ANSI 编码中编译成不同的函数，但是都可以将宽字符串类型转换为 int 类型

```cpp
int _ttoi(wchar_t const* _String);
```



#### _ttof

此函数在 Unicode 和 ANSI 编码中编译成不同的函数，但是都可以将宽字符串类型转换为 float 类型

```
float _ttof(wchar_t const* _String);
```





## MFC 应用

### MFC 控制台应用

在 VS 中创建 Windows 桌面向导，选择 MFC 标头即可创建有 MFC 标头的控制台程序

![[image-20221023201836616.png]]

MFC 控制台程序比 Win32 控制台多出一个全局对象 `CWinApp theApp;` 。通常以 Afx 开头的是 MFC 库中的全局函数，以 `::` 开头的是 Win32 的 API 函数，使用这种形式来区分不同库的函数。



### MFC 桌面应用

创建 MFC 应用程序，选择单文档类型，可以产生标准的 MFC 架构程序

![image-20221023204240202](image-20221023204240202.png)

相关的类如下

![[image-20221023204532489.png]]



### MFC 中的类

![[image-20221023224718006.png]]



### 创建项目

首先使用 Windows 向导创建一个空的桌面项目

![[image-20221023230840567.png]]

在项目属性中更改高级选项，使用静态 MFC 库

![[image-20221023231059465.png]]



### 创建控制台

我们一般直接创建桌面应用，但是这样调试起来不一定方便，因此最好自己添加一个控制台。找一个初始化的位置，给出如下代码

```cpp
#ifndef NDEBUG
	// 创建支持显示中文的控制台
	AllocConsole();
	freopen("CONOUT$", "w", stdout);
#endif
```

然后在一个销毁位置给出如下代码

```cpp
#ifndef NDEBUG
	// 释放控制台
	FreeConsole();
#endif
```

其中 NDEBUG 宏在 Release 模式下定义，因此当项目处于 Debug 模式时就会产生控制台，而处于 Release 模式就没有。





## MFC 程序分析

我们在空项目中创建一个源文件 Base.cpp，然后编写如下代码：

```c++
#include <afxwin.h>

// 自定义框架类
class CMyFrameWnd : public CFrameWnd
{
};

// 自定义应用程序类
class CMyWinApp : public CWinApp
{
public:
	virtual BOOL InitInstance()
	{
        // 创建框架类
		CMyFrameWnd* pFrame = new CMyFrameWnd;
		pFrame->Create(NULL, L"Base");
        // 将创建的窗口保存到成员变量 m_pMainWnd 中
		m_pMainWnd = pFrame;
		pFrame->ShowWindow(SW_SHOW);
		pFrame->UpdateWindow();
		return TRUE;
	}
};

// 应用程序对象的全局变量
CMyWinApp theApp;
```

这一段代码已经可以编译运行生成窗口程序。注意，如果在 x64 环境下需要将项目属性中的 MFC 使用改为“共享 DLL 中使用 MFC”。



### MFC 程序启动

我们上面书写的代码中甚至没有包含入口函数。MFC 与 Win32 窗口程序相同，都是从 WinMain 入口，但是 MFC 库已经实现了该入口，所以程序不需要实现。因此 MFC 代替程序员完成程序的流程，而程序员通过自定义全局变量 theApp 控制整个程序流程

```
CMyWinApp theApp;
```



在 MFC 中定义了全局变量

```cpp
AFX_MODULE_STATE aaa;			// 当前程序模块状态信息
AFX_MODULE_THREAD_STATE bbb;	// 当前程序线程状态信息
```

在上面的程序流程中，定义全局变量 theApp 时调用了构造函数 `CMyWinApp()`，我们知道 C++ 会自动调用基类构造函数。现在我们通过代码调试进入基类构造函数

```c++
// 构造全局对象 theApp
CWinApp::CWinApp(LPCTSTR lpszAppName)
{
	if (lpszAppName != NULL)
		m_pszAppName = _tcsdup(lpszAppName);
	else
		m_pszAppName = NULL;

	// 获取全局变量 &aaa，_AFX_CMDTARGET_GETSTATE 是一个宏，等价于 AfxGetModuleState()
	AFX_MODULE_STATE* pModuleState = _AFX_CMDTARGET_GETSTATE();
	ENSURE(pModuleState);
    // 获取全局变量 &bbb
	AFX_MODULE_THREAD_STATE* pThreadState = pModuleState->m_thread;
	ENSURE(pThreadState);
	ASSERT(AfxGetThread() == NULL);
    // this 指针是 theApp 的地址，将 &theApp 存放到 bbb 中的成员
	pThreadState->m_pCurrentWinThread = this;
    // AfxGetThread() 返回 &theApp
    // {
    // 		调用 AfxGetModuleThreadState() 获取 &bbb
    // 		返回 bbb 指向的成员变量的值，即 &theApp
	// }
	ASSERT(AfxGetThread() == this);
	m_hThread = ::GetCurrentThread();
	m_nThreadID = ::GetCurrentThreadId();

	ASSERT(afxCurrentWinApp == NULL);
    // 将 &theApp 保存到 aaa 的成员
	pModuleState->m_pCurrentWinApp = this;
    // AfxGetApp() 返回 &theApp
	ASSERT(AfxGetApp() == this);

	...
}
```



接下来我们观察 WinMain 函数的执行流程，首先在 InitInstance 函数的第一行打一个断点，调试执行到此断点，然后通过调用堆栈找到最初调用的函数 wWinMain，在此函数的返回值处添加断点，我们要进入 AfxWinMain 函数

```cpp
extern "C" int WINAPI
_tWinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance,
	_In_ LPTSTR lpCmdLine, int nCmdShow)
#pragma warning(suppress: 4985)
{
	// call shared/exported WinMain
	return AfxWinMain(hInstance, hPrevInstance, lpCmdLine, nCmdShow);
}
```

重新调试进入函数，得到 AfxWinMain 函数流程

```c++
int AFXAPI AfxWinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance,
	_In_ LPTSTR lpCmdLine, int nCmdShow)
{
	ASSERT(hPrevInstance == NULL);

	int nReturnCode = -1;
    // 这两步都是获取 &theApp，因为函数通过 theApp 指导函数执行流程
	CWinThread* pThread = AfxGetThread();
	CWinApp* pApp = AfxGetApp();
    // pApp,pThread 都是 &theApp

	// AFX internal initialization
	if (!AfxWinInit(hInstance, hPrevInstance, lpCmdLine, nCmdShow))
		goto InitFailure;

	// 利用 theApp 调用应用程序类成员虚函数
    // InitApplication 是虚函数，可以在自定义类 CMyWinApp 中重载，因此这里实现了多态
	if (pApp != NULL && !pApp->InitApplication())
		goto InitFailure;
	
	// 我们先前已经重载了 InitInstance 函数
	if (!pThread->InitInstance())
	{
		if (pThread->m_pMainWnd != NULL)
		{
			TRACE(traceAppMsg, 0, "Warning: Destroying non-NULL m_pMainWnd\n");
            // DestroyWindow 可重载
			pThread->m_pMainWnd->DestroyWindow();
		}
        // ExitInstance 可重载
		nReturnCode = pThread->ExitInstance();
		goto InitFailure;
	}
    // Run 可重载
	nReturnCode = pThread->Run();

InitFailure:
#ifdef _DEBUG
	// Check for missing AfxLockTempMap calls
	if (AfxGetModuleThreadState()->m_nTempMapLock != 0)
	{
		TRACE(traceAppMsg, 0, "Warning: Temp map lock count non-zero (%ld).\n",
			AfxGetModuleThreadState()->m_nTempMapLock);
	}
	AfxLockTempMaps();
	AfxUnlockTempMaps(-1);
#endif

	AfxWinTerm();
	return nReturnCode;
}
```



继续执行完 InitInstance，程序初始化并创建窗口，之后会重新进入 AfxWinMain 函数，这时可以执行到 Run 函数，我们不断进入函数就得到运行流程函数

```c++
int CWinThread::Run()
{
	ASSERT_VALID(this);
	_AFX_THREAD_STATE* pState = AfxGetThreadState();

	// for tracking the idle time state
	BOOL bIdle = TRUE;	// 用于判断是否退出循环
	LONG lIdleCount = 0;

	// 这就是消息循环，死循环直到接收到 WM_QUIT 消息
	for (;;)
	{
		// 没有消息时循环
		while (bIdle &&
			!::PeekMessage(&(pState->m_msgCur), NULL, NULL, NULL, PM_NOREMOVE))
		{
			// OnIdle 也是 theApp 的成员函数，可以重写
            // 如果没有消息，进行空闲处理
			if (!OnIdle(lIdleCount++))
				bIdle = FALSE; // assume "no idle" state
		}

		// phase2: pump messages while available
		do
		{
			// 处理消息，进入函数可以看到它调用 Win32 的消息处理函数
            // AfxInternalPumpMessage()
            // {
            // 		// 如果 GetMessage 获得 WM_QUIT 则返回 FALSE
            // 		if (!::GetMessage(&(pState->m_msgCur), NULL, NULL, NULL))
            // 		{
            //    		...	return FALSE;
            // 		}
            // 		...
            // 		// 在这里获取和翻译消息
			// 		if (...)
			// 		{
			// 			::TranslateMessage(&(pState->m_msgCur));
			// 			::DispatchMessage(&(pState->m_msgCur));
			// 		}
            // 		return TRUE;
        	// }
			if (!PumpMessage())
                // ExitInstance 也是 theApp 的成员虚函数，可以重载
				return ExitInstance();

			// reset "no idle" state after pumping "normal" message
			// if (IsIdleMessage(&m_msgCur))
			if (IsIdleMessage(&(pState->m_msgCur)))
			{
				bIdle = TRUE;
				lIdleCount = 0;
			}
		// 当没有收到消息时结束循环
		} while (::PeekMessage(&(pState->m_msgCur), NULL, NULL, NULL, PM_NOREMOVE));
	}
}
```



### CWinApp 的成员

总结我们调试得到的信息

* 成员虚函数
    * InitInstance 程序的初始化函数，完成窗口创建等初始化操作
    * ExitInstance 程序退出时调用，清理资源
    * Run 消息循环
    * OnIdle 空闲处理函数
* 成员变量
    * m_pMainWnd 当前应用程序的主窗口



### 钩子函数

钩子函数可用于捕捉消息，绑定应用程序的钩子函数可以捕获窗口消息，然后立即调用钩子处理函数

```cpp
// 创建钩子
HHOOK SetWindowsHookEx(
    _In_ int idHook,			// 钩子类型，如 WH_CBT
    _In_ HOOKPROC lpfn,			// 钩子处理函数，自定义
    _In_opt_ HINSTANCE hmod,	// 应用程序实例句柄
    _In_ DWORD dwThreadId		// 线程ID
);

// 钩子处理函数
LRESULT CALLBACK CBTProc(
	int nCode,			// 钩子码，如 HCBT_CREATEWND
    WPARAM wParam,		// 刚刚创建成功的窗口句柄
    LPARAM lParam
);

// 更改窗口处理函数
LONG_PTR SetWindowLongPtr(
    _In_ HWND hWnd,				// 窗口句柄
    _In_ int nIndex,			// 函数功能：更改处理函数——GWLP_WNDPROC，更改窗口风格——...
    _In_ LONG_PTR dwNewLong		// 新的窗口处理函数名（函数地址）
);
```



### 窗口创建过程

我们观察窗口创建函数的流程，在前面调用 Create 时只传入了两个参数，因为剩余参数都有默认值

```cpp
BOOL CFrameWnd::Create(LPCTSTR lpszClassName,
	LPCTSTR lpszWindowName,
	DWORD dwStyle,
	const RECT& rect,
	CWnd* pParentWnd,
	LPCTSTR lpszMenuName,
	DWORD dwExStyle,
	CCreateContext* pContext)
{
	HMENU hMenu = NULL;
    // 如果传入了菜单名，就会加载菜单
	if (lpszMenuName != NULL)
	{
		// load in a menu that will get destroyed when window gets destroyed
		HINSTANCE hInst = AfxFindResourceHandle(lpszMenuName, ATL_RT_MENU);
		if ((hMenu = ::LoadMenu(hInst, lpszMenuName)) == NULL)
		{
			TRACE(traceAppMsg, 0, "Warning: failed to load menu for CFrameWnd.\n");
			PostNcDestroy();            // perhaps delete the C++ object
			return FALSE;
		}
	}

	m_strTitle = lpszWindowName;    // save title for later
	
    // CreateEx 是 pFrame 的成员函数，注意我们传入的 lpszWindowName 为 NULL
	if (!CreateEx(dwExStyle, lpszClassName, lpszWindowName, dwStyle,
		rect.left, rect.top, rect.right - rect.left, rect.bottom - rect.top,
		pParentWnd->GetSafeHwnd(), hMenu, (LPVOID)pContext))
	{
		TRACE(traceAppMsg, 0, "Warning: failed to create CFrameWnd.\n");
		if (hMenu != NULL)
			DestroyMenu(hMenu);
		return FALSE;
	}

	return TRUE;
}
```

我们进入 CreateEx 函数

```cpp
BOOL CWnd::CreateEx(DWORD dwExStyle, LPCTSTR lpszClassName,
	LPCTSTR lpszWindowName, DWORD dwStyle,
	int x, int y, int nWidth, int nHeight,
	HWND hWndParent, HMENU nIDorHMenu, LPVOID lpParam)
{
	...

	// allow modification of several common create parameters
	CREATESTRUCT cs;
	cs.dwExStyle = dwExStyle;
	cs.lpszClass = lpszClassName;	// 这里赋值为 NULL 会有问题，在后面会调用其它函数修改
	...
    // 调用函数获取当前程序实例句柄，我们也可以调用 AfxGetInstanceHandle 来获取实例句柄
	cs.hInstance = AfxGetInstanceHandle();
	cs.lpCreateParams = lpParam;
	
    // 这里进行预先准备，执行注册窗口类，同时检查 cs 成员的值，修改有问题的值
    // PreCreateWindow(CREATESTRUCT& cs)
    // {
    // 		if (cs.lpszClass == NULL)
    //     	{
    //          // 注册窗口类，其中会使用 _afxWndFrameOrView 作为窗口类名，窗口处理函数是默认处理函数
    //         	VERIFY(AfxDeferRegisterClass(AFX_WNDFRAMEORVIEW_REG));
    //          // 在这里函数将为 NULL 的窗口类名重新赋值，右端是一个字符串
    //          cs.lpszClass = _afxWndFrameOrView;  // COLOR_WINDOW background
    //      }
    //      ...
	// 
    //      return TRUE;
    // }
	if (!PreCreateWindow(cs))
	{
		PostNcDestroy();
		return FALSE;
	}
	
    // 这一行非常重要，因为使用默认处理函数注册窗口，如果直接创建窗口，就无法处理消息
    // void AFXAPI AfxHookWindowCreate(CWnd* pWnd)
    // {
    // 		// 获取当前程序线程信息
    //  	_AFX_THREAD_STATE* pThreadState = _afxThreadState.GetData();
    //    	...
    //    	if (pThreadState->m_hHookOldCbtFilter == NULL)
    //    	{
    // 			// 创建钩子
    //        	pThreadState->m_hHookOldCbtFilter = ::SetWindowsHookEx(WH_CBT, _AfxCbtFilterHook, NULL, 	// 				::GetCurrentThreadId());
    //          ...
    //      }
    //    	...
    // 		// 传入参数 pWnd 是 pFrame 的地址，这里存储了该地址
    //    	pThreadState->m_pWndInit = pWnd;
    // }
	AfxHookWindowCreate(this);
    // CreateWindowEx 是 Win32 的创建窗口函数
    // 由于前面使用了钩子函数，创建窗口产生的 WM_CREATE 消息会被钩子函数捕获，进入钩子处理函数
	HWND hWnd = CreateWindowEx(cs.dwExStyle, cs.lpszClass,
			cs.lpszName, cs.style, cs.x, cs.y, cs.cx, cs.cy,
			cs.hwndParent, cs.hMenu, cs.hInstance, cs.lpCreateParams);
    // 钩子处理函数 _AfxCbtFilterHook 会建立句柄 hWnd 和框架类对象指针 pFrame 的绑定关系
    // 使用 SetWindowLongPtr 将窗口处理函数改为 AfxWndProc

#ifdef _DEBUG
	if (hWnd == NULL)
	{
		TRACE(traceAppMsg, 0, "Warning: Window creation failed: GetLastError returns 0x%8.8X\n",
			GetLastError());
	}
#endif

	if (!AfxUnhookWindowCreate())
		PostNcDestroy();        // cleanup if CreateWindowEx fails too soon

	if (hWnd == NULL)
		return FALSE;
	ASSERT(hWnd == m_hWnd); // should have been set in send msg hook
	return TRUE;
}
```



### 迭代器

MFC 中类保存的链表都通过迭代器访问。访问函数都具有如下形式

```cpp
POSITION GetFirstXXXPosition();			// 获取链表第一个元素的前一个地址
XXX* GetNextXXX(POSITION& rPosition);	// 获得下一个地址
```

注意 GetFirstXXXPosition 获取的是第一个元素的前一个地址。





## 窗口

### 窗口结构体

WNDCLASS 结构体包含一个窗口类的全部信息

```cpp
// 创建窗口
HWND WINAPI CreateWindow(
    LPCTSTR lpClassName, 		  // 窗口类名称
    LPCTSTR lpWindowName, 		  // 窗口名称
    DWORD dwStyle, 				 // 窗口样式
    int x, 						// 初始x坐标（窗口左上角起始位置）
    int y, 						// 初始y坐标
    int nWidth, 				// 窗口宽度，CW_USEDEFAULT表示默认宽度和高度
    int nHeight, 				// 窗口高度，CW_USEDEFAULT表示默认宽度和高度
    HWND hWndParent, 			// 父窗口句柄
    HMENU hMenu, 				// 窗口菜单的句柄
    HINSTANCE hInstance, 		 // 模块实例的句柄
    LPVOID lpParam 				// 通过WM_CREATE消息的lpParam参数指向的CREATESTRUCT结构
);

// 创建层叠的、自动弹出的扩展风格窗口
HWND WINAPI CreateWindowEx(
    DWORD DdwExStyle, 			  // 窗口扩展风格
    LPCTSTR lpClassName, 		  // 窗口类名称
    LPCTSTR lpWindowName, 		  // 窗口名称
    DWORD dwStyle, 				 // 窗口样式
    int x, 						// 初始x坐标（窗口左上角起始位置）
    int y, 						// 初始y坐标
    int nWidth, 				// 窗口宽度，CW_USEDEFAULT表示默认宽度和高度
    int nHeight, 				// 窗口高度，CW_USEDEFAULT表示默认宽度和高度
    HWND hWndParent, 			// 父窗口句柄
    HMENU hMenu, 				// 窗口菜单的句柄
    HINSTANCE hInstance, 		// 模块实例的句柄
    LPVOID lpParam 				// 通过WM_CREATE消息的lpParam参数指向的CREATESTRUCT结构
);
```

返回值：如果成功，返回新窗口的实例句柄；如果失败，返回NULL

```cpp
int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPreInstance, LPSTR lpCmdLine, int nCmdShow)
{
    // 注册窗口
    // ...
    
    // 创建窗口（返回之前发送的WM_CREATE） 
	HWND hWnd = CreateWindow("MyWindow", "第一个窗口程序", WS_OVERLAPPEDWINDOW, 
                             100, 100, 300, 300, NULL, NULL, hInstance, NULL);
    
    return 0;
}
```



| 窗口类    | 作用       |
| --------- | ---------- |
| Listbox   | 列表框     |
| ComboBox  | 下拉组合框 |
| ScrollBar | 滚动条     |
| Button    | 按钮       |
| Static    | 静态标签   |
| Edit      | 编辑框     |



### 窗口样式

如果需要使用多个[窗口样式](https:// blog.csdn.net/jadeshu/article/details/70477539)，就用 `|` 进行按位或运算；相反的，如果想去除某个样式，可以 `^` 进行按位异或运算，例如：

```cpp
WS_OVERLAPPEDWINDOW ^ WS_THICKFRAME
```

这意味着使用去除了 WS_THICKFRAME 风格的 WS_OVERLAPPEDWINDOW 风格，从而禁止调整窗口

|       设置值        |                             解说                             |
| :-----------------: | :----------------------------------------------------------: |
| WS_OVERLAPPEDWINDOW | 层叠式窗口，有边框、标题栏、系统菜单、最大最小化按钮，是以下几种风格的集合：WS_OVERLAPPED, WS_CAPTION, WS_SYSMENU, WS_THICKFRAME, WS_MINIMIZEBOX, WS_MAXIMIZEBOX |
|   WS_POPUPWINDOW    | 弹出式窗口，是以下几种风格的集合： WS_BORDER,WS_POPUP,WS_SYSMENU。WS_CAPTION 与 WS_POPUPWINDOW 风格一起使用时窗口菜单才能可见 |
|    WS_OVERLAPPED    |       层叠式窗口，有标题栏和边框，与 WS_TILED 风格类似       |
|      WS_POPUP       |             弹出式窗口，与 WS_CHILD 不能同时使用             |
|      WS_BORDER      |                        窗口有单线边框                        |
|     WS_CAPTION      |                         窗口有标题栏                         |
|      WS_CHILD       |               子窗口，不能与 WS_POPUP 同时使用               |
|     WS_DISABLED     |                          为无效窗口                          |
|     WS_HSCROLL      |                          水平滚动条                          |
|      WS_ICONIC      |                        初始化为最小化                        |
|     WS_MAXIMIZE     |                        初始化为最大化                        |
|   WS_MAXIMIZEBOX    |                         有最大化按钮                         |
|     WS_MINIMIZE     |                     与 WS_MAXIMIZE 一样                      |
|   WS_MINIMIZEBOX    |                       窗口有最小化按钮                       |
|     WS_SIZEBOX      |                   边框可进行大小控制的窗口                   |
|     WS_SYSMENU      |   创建一个有系统菜单的窗口，必须与 WS_CAPTION 风格同时使用   |
|    WS_THICKFRAME    |       创建一个大小可控制的窗口，与 WS_SIZEBOX 风格一样       |
|      WS_TILED       |                 创建一个层叠式窗口，有标题栏                 |
|     WS_VISIBLE      |                          窗口为可见                          |
|     WS_VSCROLL      |                       窗口有垂直滚动条                       |



### 窗口函数

#### ShowWindow

显示窗口

```cpp
BOOL WINAPI ShowWindow(
    HWND hWnd, // 窗口句柄
    int nCmdShow // 控制如何显示窗口
);
```

nCmdShow 参数：在第一次调用 ShowWindow 函数时，该值应为在函数 WinMain 中 nCmdShow 参数

|         参数         |                             作用                             |
| :------------------: | :----------------------------------------------------------: |
|   SW_FORCEMINIMIZE   | 在 WindowNT 5.0 中最小化窗口，即使拥有窗口的线程被挂起也会最小化。在从其他线程最小化窗口时才使用这个参数 |
|       SW_MIOE        |                    隐藏窗口并激活其他窗口                    |
|     SW_MAXIMIZE      |                       最大化指定的窗口                       |
|     SW_MINIMIZE      |       最小化指定的窗口并且激活在Z序中的下一个顶层窗口        |
|      SW_RESTORE      | 激活并显示窗口。如果窗口最小化或最大化，则系统将窗口恢复到原来的尺寸和位置。在恢复最小化窗口时，应用程序应该指定这个标志 |
|       SW_SHOW        |          在窗口原来的位置以原来的尺寸激活和显示窗口          |
|    SW_SHOWDEFAULT    | 依据在 STARTUPINFO 结构中指定的 SW_FLAG 标志设定显示状态，STARTUPINFO 结构是由启动应用程序的程序传递给 CreateProcess 函数的 |
|   SW_SHOWMAXIMIZED   |                     激活窗口并将其最大化                     |
|   SW_SHOWMINIMIZED   |                     激活窗口并将其最小化                     |
| SW_SHOWMINNOACTIVATE |             窗口最小化，激活窗口仍然维持激活状态             |
|      SW_SHOWNA       |      以窗口原来的状态显示窗口。激活窗口仍然维持激活状态      |
|  SW_SHOWNOACTIVATE   | 以窗口最近一次的大小和状态显示窗口。激活窗口仍然维持激活状态 |
|     SW_SHOWNOMAL     | 激活并显示一个窗口。如果窗口被最小化或最大化，系统将其恢复到原来的尺寸和大小。应用程序在第一次显示窗口的时候应该指定此标志 |

返回值：如果窗口以前可见，则返回值为非零。如果窗口以前被隐藏，则返回值为零。



#### MoveWindow

移动窗口，还可以用来调整窗口大小

```cpp
BOOL MoveWindow(
    HWND hWnd,
    int X,
    int Y,
    int nWidth,
    int nHeight,
    BOOL bRepaint	// 是否重绘
);
```



#### SetWindowPos

设置窗口位置，也可用来调整窗口大小

```cpp
BOOL SetWindowPos(
    HWND hWnd,
    HWND hWndInsertAfter,
    int X,
    int Y,
    int cx,
    int cy,
    UINT uFlags
);
```

hWndInsertAfter 参数可选值：

| 参数           | 作用                           |
| -------------- | ------------------------------ |
| HWND_TOP       | 前置窗口                       |
| HWND_BOTTOM    | 后置窗口                       |
| HWND_TOPMOST   | 在前面，位于任何顶部窗口的前面 |
| HWND_NOTOPMOST | 在前面，位于其他顶部窗口的后面 |



uFlags 参数可选值：

| 参数               | 作用                                                         |
| ------------------ | ------------------------------------------------------------ |
| SWP_NOSIZE         | 忽略 cx、cy，保持大小                                        |
| SWP_NOMOVE         | 忽略 X、Y，不改变位置                                        |
| SWP_NOZORDER       | 忽略 hWndInsertAfter ，保持 Z 顺序                           |
| SWP_NOREDRAW       | 不重绘                                                       |
| SWP_NOACTIVATE     | 不激活                                                       |
| SWP_FRAMECHANGED   | 强制发送 WM_NCCALCSIZE 消息，一般只是在改变大小时才发送此消息 |
| SWP_SHOWWINDOW     | 显示窗口                                                     |
| SWP_HIDEWINDOW     | 隐藏窗口                                                     |
| SWP_NOCOPYBITS     | 丢弃客户区                                                   |
| SWP_NOOWNERZORDER  | 忽略 hWndInsertAfter ，不改变 Z 序列的所有者                 |
| SWP_NOSENDCHANGING | 不发出 WM_WINDOWPOSCHANGING 消息                             |
| SWP_DRAWFRAME      | 画边框                                                       |
| SWP_DEFERERASE     | 防止产生 WM_SYNCPAINT 消息                                   |
| SWP_ASYNCWINDOWPOS | 若调用进程不拥有窗口, 系统会向拥有窗口的线程发出需求}        |



#### EnableWindow

禁止或激活窗口

```cpp
BOOL EnableWindow(
    HWND hWnd,
    BOOL bEnable
);
```



#### InvalidateRect

**窗口被另一个窗口覆盖的区域称为无效区域**。添加一个矩形（无效区域）到指定窗口的更新区域（必须重绘的窗口客户区域部分），当消息队列为空时，发送 `WM_PAINT` 消息

```cpp
BOOL InvalidateRect(
    HWND hWnd, 				// 窗口句柄
    const RECT * lpRect, 	// 矩形区域，若为NULL则将整个客户区添加到更新区域
    BOOL bErase 			// 指定更新区域内的背景在处理更新区域时是否要擦除，当为 true 时发送 WM_ERASEBKGND 消息
);
```



#### UpdateWindow

若更新区域不为空，会立即发送 WM_PAINT 消息，该消息直接发送到指定窗口的窗口过程，绕过应用程序队列。使用 UpdateWindow 时要先指定更新区域，如使用 InvalidateRect 函数添加更新区域

```cpp
BOOLWIN UpdateWindow(
	HWND hWnd // 窗口句柄
);
```



#### RedrawWindow

更新窗口的客户区指定的矩形或区域，相当于 UpdateWindow 和 InvalidateRect 两个函数的组合

```cpp
BOOL RedrawWindow(
    HWND hWnd, 					// 窗口句柄，若为NULL则更新桌面窗口
    const RECT * lprcUpdate, 	// 矩形区域
    HRGN hrgnUpdate, 			// 更新区域的句柄，若两个...Update参数都为NULL，则将整个客户区添加到更新区域
    UINT flags 					// 一个或多个重绘标志
);
```

flags 参数

* RDW_ERASE 窗口收到 WM_ERASEBKGND 消息，必须同时指定 RDW_INVALIDATE
* RDW_FRAME 与更新区域相交的窗口非客户区域收到 WM_NCPAINT 消息，必须同时指定 RDW_INVALIDATE
* RDW_INTERNALPAINT 窗口收到 WM_PAINT 消息
* RDW_INVALIDATE 使 lprcUpdate 或 hrgnUpdate 无效



#### DestroyWindow

销毁窗口

```cpp
BOOL WINAPI DestroyWindow(
	HWND hWnd // 要销毁的窗口句柄
);
```



#### SetWindowText

改变指定窗口句柄的窗口名

```cpp
BOOL SetWindowText(
    HWND hWnd,
    LPCSTR lpString
);
```



#### GetWindowText

获取指定窗口标题或文本

```cpp
int GetWindowText(
    HWND hWnd,
    LPWSTR lpString,	// 缓冲区
    int nMaxCount		// 最大长度
);
```



#### GetWindowTextLength

获取指定窗口标题或文本长度

```cpp
int GetWindowTextLengthW(
    HWND hWnd
);
```



#### GetWindowRect

获取窗口尺寸，结果存放在矩形指针中

```cpp
BOOL GetWindowRect(
    HWND hWnd,
    LPRECT lpRect	// 矩形指针
);
```



#### GetClientRect

获取客户区尺寸，结果存放在矩形指针中

```cpp
BOOL GetClientRect(
    HWND hWnd,
    LPRECT lpRect	// 矩形指针
);
```



#### SetFocus

设置具有输入焦点的窗口

```cpp
HWND SetFocus(
    HWND hWnd
);
```





## 消息

### 消息处理

在自定义框架类中重载消息处理函数

```cpp
class CMyFrameWnd : public CFrameWnd
{
public:
	virtual LRESULT WindowProc(UINT msgID, WPARAM wParam, LPARAM lParam)
	{
		switch (msgID)
		{
		case WM_CREATE:
			AfxMessageBox(L"WM_CREATE 消息被处理");
			break;
		case WM_PAINT:
		{
			PAINTSTRUCT ps = { 0 };
			HDC hdc = ::BeginPaint(this->m_hWnd, &ps);
			::TextOut(hdc, 100, 100, L"Hello", 5);
			::EndPaint(m_hWnd, &ps);
			break;
		}
		default:
			break;
		}
		// 调用基类默认的处理函数
		return CFrameWnd::WindowProc(msgID, wParam, lParam);
	}
};
```

通过调用堆栈，我们找到最初调用的消息处理函数。前面窗口创建时，将 AfxWndProc 作为消息处理函数，因此所有的窗口消息都会通过它，我们观察它如何进入到自定义的消息处理函数中

```cpp
LRESULT CALLBACK AfxWndProc(HWND hWnd, UINT nMsg, WPARAM wParam, LPARAM lParam)
{
	// special message which identifies the window as using AfxWndProc
	if (nMsg == WM_QUERYAFXWNDPROC)
		return 1;

	// 通过句柄 hWnd 获取对应窗口的框架类对象 CFrameWnd 的指针，pWnd 得到 pFrame
	CWnd* pWnd = CWnd::FromHandlePermanent(hWnd);
	ASSERT(pWnd != NULL);
	ASSERT(pWnd==NULL || pWnd->m_hWnd == hWnd);
	if (pWnd == NULL || pWnd->m_hWnd != hWnd)
        // 首先会使用 Win32 的默认处理函数
		return ::DefWindowProc(hWnd, nMsg, wParam, lParam);
	return AfxCallWndProc(pWnd, hWnd, nMsg, wParam, lParam);
}

LRESULT AFXAPI AfxCallWndProc(CWnd* pWnd, HWND hWnd, UINT nMsg,
	WPARAM wParam = 0, LPARAM lParam = 0)
{
	...
	// Catch exceptions thrown outside the scope of a callback
	// in debug builds and warn the user.
	LRESULT lResult;
	TRY
	{
        ...
		// 在这里调用用户自定义的消息处理函数
		lResult = pWnd->WindowProc(nMsg, wParam, lParam);
        ...
	}
    ...
}
```



### 消息映射机制

消息映射机制可以在不重写 WindowProc 虚函数的前提下，仍然可以处理消息。



使用消息映射的类应当满足：

* 类内要添加声明宏 DECLARE_MESSAGE_MAP()
* 类外必须添加实现宏
    * BEGIN_MESSAGE_MAP(theClass, baseClass)
    * END_MESSAGE_MAP()

只要具有这些宏，这个类就可以使用消息映射处理消息。



以 WM_CREATE 消息为例：

* BEGIN_MESSAGE_MAP 和 END_MESSAGE_MAP 之间添加 ON_MESSAGE(WM_CREATE, OnCreate) 宏
* 在 CMyFrameWnd 类内添加 OnCreate 函数的声明和定义

```cpp
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	LRESULT OnCreate(WPARAM wParam, LPARAM lParam)
	{
		AfxMessageBox(L"WM_CREATE");
		return 0;
	}
};

// 用要处理消息的类名和基类作为参数
BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
    // 参数为：要处理的消息名，要处理消息的函数名，函数名可以随意取
	ON_MESSAGE(WM_CREATE, OnCreate)
END_MESSAGE_MAP()
```

我们将上面使用的宏展开得到

```cpp
// DECLARE_MESSAGE_MAP()
protected:
	static const AFX_MSGMAP* PASCAL GetThisMessageMap();
	virtual const AFX_MSGMAP* GetMessageMap() const;

// BEGIN_MESSAGE_MAP(theClass, baseClass)
PTM_WARNING_DISABLE		// 用此声明包含后，我们可以省略函数前面的取地址符 &
    // 下面两个函数都是对上面函数声明的具体定义
	const AFX_MSGMAP* theClass::GetMessageMap() const
		{ return GetThisMessageMap(); }
	const AFX_MSGMAP* PASCAL theClass::GetThisMessageMap()
	{
		typedef theClass ThisClass;						   
		typedef baseClass TheBaseClass;
        
		// struct AFX_MSGMAP_ENTRY
        // {
        // 	UINT nMessage;   // 消息id
        // 	UINT nCode;      // 通知码
        // 	UINT nID;        // 命令id
        // 	UINT nLastID;    // 最后一个命令id
        // 	UINT_PTR nSig;   // 处理消息的函数类型
        // 	AFX_PMSG pfn;    // 处理消息的函数名（地址）
        // };
		// 创建一个结构体数组，可以看到一共有两个结构体元素
		static const AFX_MSGMAP_ENTRY _messageEntries[] =
		{
            
// ON_MESSAGE(message, memberFxn) 
            // message：消息id
            // 由于使用了 PTM_WARNING_DISABLE 声明，memberFxn 就是函数的地址
			{ message, 0, 0, 0, AfxSig_lwl, (AFX_PMSG)(AFX_PMSGW) 
			(static_cast< LRESULT (AFX_MSG_CALL CWnd::*)(WPARAM, LPARAM) > (memberFxn)) },
            
// END_MESSAGE_MAP()
            {0, 0, 0, 0, AfxSig_end, (AFX_PMSG)0 } 
		}; 
        // struct AFX_MSGMAP
        // {
        //    const AFX_MSGMAP* (PASCAL* pfnGetBaseMap)();	// 父类宏展开的静态变量地址
        //    const AFX_MSGMAP_ENTRY* lpEntries;			// 本类宏展开的静态数组首地址
        // };
		// 结构体第一个变量是父类中宏展开的 GetThisMessageMap 返回 messageMap 的地址
        // 第二个变量是本类中 _messageEntries[0] 的首地址
		static const AFX_MSGMAP messageMap = { &TheBaseClass::GetThisMessageMap, &_messageEntries[0] }; 
        return &messageMap; 
	}								  
PTM_WARNING_RESTORE
```

宏展开得到从基类到父类的链表结构，每个基类中的 messageMap 都储存了父类的 messageMap 的地址，可以不断向上追溯。

![[image-20221024215658345.png|600]]

* `GetThisMessageMap` 静态函数，定义静态变量和静态数组，并返回本类静态变量地址，即获取链表头
* `_messageEntries` 静态数组，每个元素保存消息ID，处理消息的函数等信息
* `messageMap` 静态变量，维持链表结构
* `GetMessageMap` 虚函数，返回本类静态变量地址，即链表头

由于我们不重写 WindowProc 虚函数，当有消息产生时，程序会直接调用给定的 WindowProc，在此函数中将获取当前 pFrame 中产生的 messageMap 地址，然后不断向父类上溯，对比 msg 中存储的消息ID 与产生的消息ID，匹配成功后返回 msg 的地址。最后从 msg 中得到 OnCreate 的地址从而进入我们自定义的函数。



理解了消息映射机制，我们可以添加处理新的消息 WM_PAINT

```cpp
// 自定义框架类
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	LRESULT OnCreate(WPARAM wParam, LPARAM lParam)
	{
		AfxMessageBox(L"WM_CREATE");
		return 0;
	}

	LRESULT OnPaint(WPARAM wParam, LPARAM lParam)
	{
		PAINTSTRUCT ps = { 0 };
		HDC hdc = ::BeginPaint(this->m_hWnd, &ps);
		::TextOut(hdc, 100, 100, L"Hello", 5);
		::EndPaint(m_hWnd, &ps);
		return 0;
	}
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_MESSAGE(WM_CREATE, OnCreate)
	ON_MESSAGE(WM_PAINT, OnPaint)
END_MESSAGE_MAP()
```

我们多添加了一行宏 `ON_MESSAGE(WM_PAINT, OnPaint)`，根据前面的宏展开形式

```cpp
// ON_MESSAGE(message, memberFxn) 
{ message, 0, 0, 0, AfxSig_lwl, (AFX_PMSG)(AFX_PMSGW) 
(static_cast< LRESULT (AFX_MSG_CALL CWnd::*)(WPARAM, LPARAM) > (memberFxn)) },
```

这一行实际上是向 _messageEntries 数组中添加了一个新的元素，这样它就有三个元素。类似地可以添加更多消息处理。



### 消息分类

先前使用 ON_MESSAGE 可以对任意消息指定任意函数作为处理函数。但是对于一般的 windows 消息，应该直接使用 ON_WM_XXX 宏定义建立消息映射。

* 标准 windows 消息：ON_WM_XXX
* 自定义消息：ON_MESSAGE
* 命令消息：ON_COMMAND

不同的宏定义对应的消息处理函数的声明形式不同，我们可以根据宏展开后的结构体成员判断函数类型，例如 ON_MESSAGE 展开为

```cpp
{ message, 0, 0, 0, AfxSig_lwl, ...},
```

观察第四个成员 `AfxSig_lwl`，跳转到它的定义可以看到对应的函数类型（在注释中）

```cpp
AfxSig_lwl = AfxSig_l_w_l,     // LRESULT (WPARAM, LPARAM)
```

所有 ON_MESSAGE 都使用这种类型的函数，另外函数名可以自定义。



#### 标准消息

WM_CREATE 消息映射宏为 ON_WM_CREATE，展开后查找第四个成员为

```cpp
AfxSig_is = AfxSig_i_v_s,      // int (LPTSTR)
```

注意有时候我们查找到的注释对应的类型不正确，可以通过**查询父类 CFrameWnd 中的同名函数声明**来确定函数类型，例如

```cpp
int CWnd::OnCreate(LPCREATESTRUCT);
```

即 `int (LPCREATESTRUCT)` 才是 OnCreate 函数的类型。



WM_PAINT 消息映射宏为 ON_WM_PAINT，对应的函数类型为

```cpp
AfxSig_vv = AfxSig_v_v_v,      // void (void)
```

因此我们的消息处理代码修改为

```cpp
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
    // 这里两个函数名还是之前使用过的，是因为我们之前使用的恰好就是系统定义的函数名
	int OnCreate(LPCREATESTRUCT pcs)
	{
		AfxMessageBox(L"WM_CREATE");
        // 最后还要执行父类的同名函数进行处理
		return CFrameWnd::OnCreate(pcs);
	}

	void OnPaint()
	{
		PAINTSTRUCT ps = { 0 };
		HDC hdc = ::BeginPaint(this->m_hWnd, &ps);
		::TextOut(hdc, 100, 100, L"Hello", 5);
		::EndPaint(m_hWnd, &ps);
	}
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
    // windows 消息的宏不需要传入参数
	ON_WM_CREATE()
	ON_WM_PAINT()
END_MESSAGE_MAP()
```

windows 消息宏使用的函数名通过查询对应的宏定义中结构体的最后一个成员变量得到，例如

```cpp
// 鼠标移动消息 ON_WM_MOUSEMOVE
{ WM_MOUSEMOVE, 0, 0, 0, AfxSig_vwp, (AFX_PMSG)(AFX_PMSGW) 
(static_cast< void (AFX_MSG_CALL CWnd::*)(UINT, CPoint) > ( &ThisClass :: OnMouseMove)) },
```

最后传入的函数名是 OnMouseMove，只需要重写这一函数就可以进行处理。通过查找父类中 OnMouseMove 的定义

```cpp
void CWnd::OnMouseMove(UINT, CPoint)
```

我们就可以仿照这一形式重写函数。



不同消息携带的参数直接参考 Win32 中的消息传递参数即可，我们写一个可以让文字跟随鼠标的程序

```cpp
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	int OnCreate(LPCREATESTRUCT pcs)
	{
		AfxMessageBox(L"WM_CREATE");
		return CFrameWnd::OnCreate(pcs);
	}

	void OnPaint()
	{
		PAINTSTRUCT ps = { 0 };
		HDC hdc = ::BeginPaint(this->m_hWnd, &ps);
		::TextOut(hdc, m_x, m_y, L"Hello", 5);
		::EndPaint(m_hWnd, &ps);
	}

	void OnMouseMove(UINT nKey, CPoint pt)
	{
		m_x = pt.x;
		m_y = pt.y;
		::InvalidateRect(this->m_hWnd, NULL, TRUE);
	}

public:
    // 用于储存鼠标位置
	int m_x;
	int m_y;
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
	ON_WM_PAINT()
	ON_WM_MOUSEMOVE()
END_MESSAGE_MAP()
```



#### 自定义消息

和 Win32 一样，用户可以自定义用户消息并进行处理

```cpp
#define WM_MYMESSAGE WM_USER + 1001

// 自定义框架类
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	int OnCreate(LPCREATESTRUCT pcs)
	{
        // 向窗口发送自定义消息，并附带信息 1,2
		::PostMessage(this->m_hWnd, WM_MYMESSAGE, 1, 2);
		return CFrameWnd::OnCreate(pcs);
	}

	LRESULT OnMyMessage(WPARAM wParam, LPARAM lParam)
	{
        // MFC 封装的字符串类型
		CString str;
        // 格式化字符串
		str.Format(L"wParam=%d,lParam=%d", wParam, lParam);
		AfxMessageBox(str);
        // 由于是自定义函数，没有父类同名函数，直接返回 0
		return 0;
	}
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
	ON_MESSAGE(WM_MYMESSAGE, OnMyMessage)
END_MESSAGE_MAP()
```



#### 命令消息

通过查找定义，ON_COMMAND 消息的处理函数类型为

```cpp
AfxSigCmd_v,				// void ()
```

绑定的处理函数名可以随意取。现在我们添加处理菜单项的方法

```cpp
#include <afxwin.h>
#include "resource.h"

// 自定义框架类
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	afx_msg int OnCreate(LPCREATESTRUCT pcs)
	{
		menu.LoadMenu(IDR_MENU1);
		this->SetMenu(&menu);
		return CFrameWnd::OnCreate(pcs);
	}

	afx_msg void OnNew()
	{
		AfxMessageBox(L"新建菜单项被点击");
	}

public:
	CMenu menu;
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()
```

注意 ON_COMMAND 的消息ID 应当是被点击的菜单项的ID，这里是“新建”项 ID_NEW；另外，从这里开始我们为所有消息处理函数添加前缀 afx_msg，这是由 MFC 提供的消息处理函数标准前缀，方便与普通成员函数区分，实际程序编译运行不会有区别。



我们也可以在应用程序类中进行命令消息的处理

```cpp
class CMyWinApp : public CWinApp
{
	DECLARE_MESSAGE_MAP()
public:
	virtual BOOL InitInstance()
	{
		CMyFrameWnd* pFrame = new CMyFrameWnd;
		pFrame->Create(NULL, L"Base");
		m_pMainWnd = pFrame;
		pFrame->ShowWindow(SW_SHOW);
		pFrame->UpdateWindow();
		return TRUE;
	}

	afx_msg void OnNew()
	{
		AfxMessageBox(L"新建菜单项被点击");
	}
};

BEGIN_MESSAGE_MAP(CMyWinApp, CWinApp)
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()
```

事实上，可以在任何一个类中处理命令消息，但是命令消息存在处理顺序的优先级

* 视图类 > 文档类 > 框架类 > 应用程序类

当在多个类中处理命令消息时，会按照优先级决定由哪个类进行处理。



#### 控件消息

控件消息也是命令消息，但是这里我们主要针对控件产生的**不同命令消息**

* wParam
    * LOWORD(wParam) 菜单项/控件 ID
    * HIWORD(wParam) 菜单项返回 0；控件返回通知码
* lParam 无用

当菜单被点击时，通过 ON_COMMAND(ID, OnFunc) 形式的消息映射可以处理。但是对于控件来说，这种形式只能处理**单击**消息，对于双击等其它消息不能处理，这时候需要针对不同的消息需求使用，例如双击应当使用 ON_BN_DOUBLECLICKED 形式。消息的种类非常复杂，实际应用时，通过我们后面会介绍的**类向导**自动添加即可。



### 键盘消息

键盘消息通过 WM_KEYDOWN，WM_KEYUP 等消息映射获取。以 WM_KEYDOWN 为例，对应的消息处理函数为

```cpp
void OnKeyDown(
    UINT nChar,		// 按键的虚拟码
    UINT nRepCnt,	// 重复次数（由于用户按住键而重复击键的次数）
    UINT nFlags
);
```

虚拟码参照 Win32 中的相关宏即可。





## 菜单和工具

Win32 中使用 HMENU 标记菜单，MFC 则使用 CMenu 类对象绑定句柄 HMENU 控制菜单，其中封装了各种操作成员函数，还使用成员变量 m_hMenu 保存菜单句柄。



我们首先新建一个 .rc 文件，添加菜单项，将“新建”项的 ID 修改为 ID_NEW，注意菜单本身的 ID 是文件名 IDR_MENU1

![[image-20221025095743468.png]]

在 Create 函数中可以传入并加载菜单

```cpp
#include <afxwin.h>
#include "resource.h"

// 自定义框架类
class CMyFrameWnd : public CFrameWnd
{
};

// 自定义应用程序类
class CMyWinApp : public CWinApp

public:
	virtual BOOL InitInstance()
	{
		CMyFrameWnd* pFrame = new CMyFrameWnd;
        // 参数类型可以通过查看函数声明获得
		pFrame->Create(NULL, L"Base", WS_OVERLAPPEDWINDOW,
			CFrameWnd::rectDefault, NULL, (LPCTSTR)IDR_MENU1);
		m_pMainWnd = pFrame;
		pFrame->ShowWindow(SW_SHOW);
		pFrame->UpdateWindow();
		return TRUE;
	}
};

// 应用程序对象的全局变量
CMyWinApp theApp;
```

但是一般不在这里直接传入，而是在 WM_CREATE 消息处理时进行加载

```cpp
#include <afxwin.h>
#include "resource.h"

// 自定义框架类
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	int OnCreate(LPCREATESTRUCT pcs)
	{
		menu.LoadMenu(IDR_MENU1);
        // 调用 CMenu 的成员函数设置菜单
		this->SetMenu(&menu);
        // 其实也可以调用 Win32 中的函数加载
        // ::SetMenu(this->m_hWnd, menu.m_hMenu);
		return CFrameWnd::OnCreate(pcs);
	}

public:
    // 定义为成员变量，确保生命周期与窗口一致
	CMenu menu;
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
END_MESSAGE_MAP()


// 自定义应用程序类
class CMyWinApp : public CWinApp
{
public:
	virtual BOOL InitInstance()
	{
		CMyFrameWnd* pFrame = new CMyFrameWnd;
		pFrame->Create(NULL, L"Base");
		m_pMainWnd = pFrame;
		pFrame->ShowWindow(SW_SHOW);
		pFrame->UpdateWindow();
		return TRUE;
	}
};

// 应用程序对象的全局变量
CMyWinApp theApp;
```



### 处理函数

菜单的处理函数直接参考 Win32 中的函数，例如 WM_INITMENUPOPUP 消息在菜单弹出时触发，使用 CheckMenuItem 函数来为菜单项添加一个选中标记；WM_CONTEXTMENU 在右键点击时触发，使用 TrackPopupMenu 在点击位置弹出菜单

```cpp
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	afx_msg int OnCreate(LPCREATESTRUCT pcs)
	{
		menu.LoadMenu(IDR_MENU1);
		this->SetMenu(&menu);
		return CFrameWnd::OnCreate(pcs);
	}
	
    // 初始化弹出菜单
	afx_msg void OnInitMenuPopup(CMenu* pPopup, UINT nPos, BOOL i)
	{
		pPopup->CheckMenuItem(ID_NEW, MF_CHECKED);
	}
	
    // 右键弹出菜单
	afx_msg void OnContextMenu(CWnd* pWnd, CPoint pt)
	{
		CMenu* pPopup = menu.GetSubMenu(0);
		pPopup->TrackPopupMenu(TPM_LEFTALIGN | TPM_TOPALIGN, pt.x, pt.y, this);
	}

public:
	CMenu menu;
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
	ON_WM_INITMENUPOPUP()
	ON_WM_CONTEXTMENU()
END_MESSAGE_MAP()
```



### 点击消息

可以点击的菜单项通过 ID 来绑定点击消息。例如“打开”按钮的 ID 如果是 ID_OPEN，在类向导的命令栏中找到对应的 ID 双击即可自动创建消息函数的声明和定义

![[image-20230116210646298.png]]



### 菜单项操作

在 CFrameWnd 继承类的成员函数中通过 GetMenu 函数获得主菜单对象指针。根据要操作的菜单项在菜单中的位置调用 GetSubMenu 获得子菜单指针，最后得到我们要操作的菜单指针。这里对主菜单的第三个子菜单下的第一个菜单项进行选中操作

```cpp
CMenu* mainMenu = GetMenu();
CMenu* meshMenu = mainMenu->GetSubMenu(2);
CMenu* showMenu = meshMenu->GetSubMenu(0);
showMenu->CheckMenuItem(ID_FILL, MF_CHECKED);
```

其中 CheckMenuItem 名称和 Win32 中对应的操作函数名相同，区别在于参数少了一个菜单项句柄

```cpp
DWORD CheckMenuItem(
    HMENU hMenu,
    UINT uIDCheckItem, 	// 菜单项标识（ID/在菜单中的位置）
    UINT uCheck 		// 菜单项操作标识
);
```

* MF_BYCOMMAND 以 ID 值标识菜单项
* MF_CHECKED 添加选中标志
* MF_BYPOSITION 以位置标识菜单项
* MF_UNCHECKED 删除选中标志

其它的菜单项操作函数也是类似的。



### 快捷键

可以在资源中创建加速键资源 Accelerator，在右边表格中填写要绑定的 ID，指定修饰符 Ctrl, Shift 以及组合的键。

![[image-20230116211316489.png]]

只要用户按下了快捷键，就会关联到对应 ID 的消息处理函数。实际上，正如前面点击消息处理中的流程，只要在类向导中双击命令栏中的 ID 建立消息函数，然后创建对应 ID 的快捷键，按下快捷键就会自动调用该消息函数。



在使用快捷键前，还需要导入快捷键资源。需要在类的构造函数中初始化加速键并保存，然后重载消息翻译函数

```cpp
class CMyFrame : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	HACCEL m_hAccKey;	// 加速键

public:
	CMyFrame()
	{
		// 加速键初始化
		m_hAccKey = LoadAccelerators(AfxGetInstanceHandle(), MAKEINTRESOURCE(IDR_ACCELERATOR1));
	}
    
    // 重载消息翻译函数
	virtual BOOL PreTranslateMessage(MSG* pMsg)
    {
        // 开启加速键翻译
        if (TranslateAccelerator(this->m_hWnd, m_hAccKey, pMsg))
            return true;

        return CFrameWnd::PreTranslateMessage(pMsg);
    }
};
```



### 工具栏

MFC 新增的头文件 afxext.h 中定义了工具栏类型，我们可以先在资源中添加 ToolBar，资源 ID 为 IDR_TOOLBAR1，新建三个小图标，分别将 ID 改为 ID_NEW, ID_SET, ID_SAVE；如果要删除图标，点击图标向下拖动即可。

![image-20221025153813810](image-20221025153813810.png)

注意第一个图标的 ID 与菜单的“新建”项相同，因此它们会产生相同的消息。



通过成员函数创建和加载工具栏

```cpp
// 创建工具栏
BOOL CreateEx(
    CWnd* pParentWnd, 											// 所在窗口
    DWORD dwCtrlStyle = TBSTYLE_FLAT,							// 按钮风格
	DWORD dwStyle = WS_CHILD | WS_VISIBLE | CBRS_ALIGN_TOP,		// 工具栏窗口风格
	CRect rcBorders = CRect(0, 0, 0, 0),
	UINT nID = AFX_IDW_TOOLBAR
);

// 加载工具栏
BOOL CToolBar::LoadToolBar(
    UINT nIDResource	// 工具栏 ID
);
```

如果要实现工具栏拖动效果，需要使用 CBRS_GRIPPER 风格，并且设定工具栏可停靠的范围

* CToolBar 设定工具栏可停靠的位置
* CMyFrameWnd 设定允许工具栏停靠的位置
* CToolBar 设定当存在多个可停靠位置时选择停靠的位置

```cpp
#include <afxwin.h>
#include <afxext.h>
#include "resource.h"

class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	afx_msg int OnCreate(LPCREATESTRUCT pcs)
	{
        // 创建工具栏
		toolBar.CreateEx(this, TBSTYLE_FLAT, WS_CHILD | WS_VISIBLE
			| CBRS_ALIGN_TOP | CBRS_GRIPPER);
        
        // 加载工具栏
		toolBar.LoadToolBar(IDR_TOOLBAR1);
        
        // 设置工具栏可以停靠的位置
		toolBar.EnableDocking(CBRS_ALIGN_ANY);
		this->EnableDocking(CBRS_ALIGN_ANY);
		this->DockControlBar(&toolBar, AFX_IDW_DOCKBAR_BOTTOM);
		return CFrameWnd::OnCreate(pcs);
	}

public:
    // 定义为成员变量延长生命周期
	CToolBar toolBar;
};
```





## 特殊机制

### 运行时类信息机制

此机制帮助程序判断对象属于哪个类。使用时

* 类必须派生自 CObject
* 类内必须添加声明宏 DECLARE_DYNAMIC(theClass)
* 类外必须添加实现宏 IMPLEMENT_DYNAMIC(theClass, baseClass)

当类具备这三个要素时，CObject::IsKindOf 函数可以判断对象所属的类。

```cpp
// 定义具有运行时类信息机制的类
class CAnimal : public CObject
{
	DECLARE_DYNAMIC(CAnimal)
};
IMPLEMENT_DYNAMIC(CAnimal, CObject)

CAnimal animal;
// 调用 IsKindOf 判断所属类
animal.IsKindOf(RUNTIME_CLASS(CAnimal));
```



我们将宏展开，第一个类内宏定义了一个静态常量结构体和一个虚函数，第二个类外宏给出了静态常量结构体的值。

```cpp
// DECLARE_DYNAMIC(class_name)
public: 
	static const CRuntimeClass class##class_name; 
	virtual CRuntimeClass* GetRuntimeClass() const; 

// IMPLEMENT_DYNAMIC(class_name, base_class_name) 
// IMPLEMENT_RUNTIMECLASS(class_name, base_class_name, wSchema, pfnNew, class_init) 
AFX_COMDAT const CRuntimeClass class_name::class##class_name = { 
	#class_name, 				// 类名，class_name 用双引号包含，因此是字符串
    sizeof(class class_name), 	// 类的大小
    wSchema, 
    pfnNew, 
	// RUNTIME_CLASS(base_class_name)
    // _RUNTIME_CLASS(base_class_name)
    ((CRuntimeClass*)(&base_class_name::class##base_class_name))	// 本类的静态变量地址
    NULL, 
    class_init 
}; 

CRuntimeClass* class_name::GetRuntimeClass() const 
{ 
    // return RUNTIME_CLASS(class_name); 
    return ((CRuntimeClass*)(&class_name::class##class_name));
}
```

注意 `#` 表示将宏替换后用双引号包含，但是 `##` 会取消这一效果，上面 `class##class_name` 在 `class_name` 到 `CAnimal` 的替换结果是 `classCAnimal` 。



我们会发现 CAnimal 类中的静态常量结构体的第五个成员是基类 CObject 类中静态常量结构体的地址，如果 CAnimal 又派生出其它具有此机制的类，我们就再次得到一个链表结构。

* classCAnimal 静态常量，保存类名和类大小等信息，以及父类静态常量的地址
* GetRuntimeClass() 虚函数，获取本类的静态常量地址，即获取链表头节点

当调用 IsKindOf 函数时，首先获取了本类的静态常量地址，然后开始向上追溯，每次获取父类静态常量的地址，对比传入的类的静态常量地址，如果相同就说明有继承关系。

```cpp
// 传入参数是 CAnimal 的静态常量地址
animal.IsKindOf(RUNTIME_CLASS(CAnimal));
```



### 动态创建机制

此机制可以在不知道类名的情况下，将类的对象创建出来。使用时

* 类必须派生自 CObject
* 类内必须添加声明宏 DECLARE_DYNCREATE(theClass)
* 类外必须添加实现宏 IMPLEMENT_DYNCREATE(theClass, baseClass)

当类具备这三个要素时，CRuntimeClass::CreateObject 对象加工厂可以将类创建出来。

```cpp
class CAnimal : public CObject
{
	DECLARE_DYNCREATE(CAnimal)
};
IMPLEMENT_DYNCREATE(CAnimal, CObject)

// 对象加工厂
CObject* pob = RUNTIME_CLASS(CAnimal)->CreateObject();
```

动态创建机制的主要作用是为了**让 MFC 库能够产生用户定义的对象**，其实现机制和类信息机制基本相同，区别在于 CRuntimeClass 结构体中的第四个成员 pfnNew 被赋值为 CAnimal::CreateObject，系统将会通过此函数产生并返回 CAnimal 类对象。





## 视图与文档

### 视图窗口

视图窗口提供了一个用于显示数据的窗口。CView 及其子类，父类为 CWnd 类，封装了关于视图窗口的操作以及文档类的数据交互。



首先要定义一个自己的视图类，派生自 CView，并重写父类成员纯虚函数 OnDraw 用于绘图。在处理框架窗口的 WM_CREATE 消息时，定义我们的视图类，并调用 Create 函数创建视图窗口。

```cpp
BOOL Create(
    LPCTSTR lpszClassName,				// 类名
	LPCTSTR lpszWindowName, 			// 窗口名
    DWORD dwStyle,						// 窗口风格
	const RECT& rect,					// 窗口尺寸
	CWnd* pParentWnd, 					// 父窗口指针
    UINT nID,							// 窗口ID
	CCreateContext* pContext = NULL
);
```

如果 nID 小于 AFX_IDW_PANE_FIRST，则会按照给定的尺寸产生视图窗口，否则会平铺到整个窗口。

```cpp
// 自定义视图窗口类
class CMyView : public CView
{
public:
	void OnDraw(CDC* pDC) 
    {
        pDC->TextOutW(100, 100, L"CMyView::OnDraw");
    }
};

// 自定义框架类
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	afx_msg void OnPaint()
	{
		PAINTSTRUCT ps = { 0 };
		HDC hdc = ::BeginPaint(this->m_hWnd, &ps);
		::TextOut(hdc, 100, 100, L"框架窗口客户区", wcslen(L"框架窗口客户区"));
		::EndPaint(this->m_hWnd, &ps);
	}

	afx_msg int OnCreate(LPCREATESTRUCT pcs)
	{
		CMyView* pView = new CMyView;
		pView->Create(NULL, L"MyView", WS_CHILD | WS_VISIBLE | WS_BORDER,
			CRect(0, 0, 200, 200), this, AFX_IDW_PANE_FIRST);
        // 将我们创建的视图作为活动视图窗口
        m_pViewActive = pView;
		return CFrameWnd::OnCreate(pcs);
	}
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
	ON_WM_PAINT()
END_MESSAGE_MAP()
```

纯虚函数 OnDraw 是父类 CView 用于处理 WM_PAINT 消息的函数，我们必须在派生类中重写。虽然也可以在自定义视图窗口类中建立消息映射机制来处理 WM_PAINT 消息，但是完全没有必要，通过 OnDraw 函数直接绘图即可。



对象关系

```cpp
m_pMainWnd = pFrame;
m_pViewActive = pView;
```

* theApp
    * m_pMainWnd 框架类对象地址
        * m_pViewActive 活动视图类对象地址

通过全局变量 theApp，可以很容易获得框架类和视图类。



### 文档类

文档类 CDocument 提供了管理数据的类，封装了对数据的管理，并和视图类进行数据交互。

```cpp
#include <afxwin.h>
#include <afxext.h>
#include "resource.h"

class CMyDoc : public CDocument
{
};

// 自定义视图窗口类
class CMyView : public CView
{
	DECLARE_DYNCREATE(CMyView)
public:
	void OnDraw(CDC* pDC)
	{
		pDC->TextOutW(100, 100, L"我是视图窗口");
	}
};
IMPLEMENT_DYNCREATE(CMyView, CView)

// 自定义框架类
class CMyFrameWnd : public CFrameWnd
{
    // 我们在这里不定义消息处理
};

// 自定义应用程序类
class CMyWinApp : public CWinApp
{
public:
	virtual BOOL InitInstance()
	{
		CMyFrameWnd* pFrame = new CMyFrameWnd;
		CMyDoc* pDoc = new CMyDoc;

		CCreateContext cct;
		cct.m_pCurrentDoc = pDoc;						// 文档类对象地址
		cct.m_pNewViewClass = RUNTIME_CLASS(CMyView);	// CMyView 的静态常量地址
		
		// 加载框架，会自动创建窗口
		pFrame->LoadFrame(IDR_MENU1, WS_OVERLAPPEDWINDOW, NULL, &cct);
		m_pMainWnd = pFrame;
		pFrame->ShowWindow(SW_SHOW);
		pFrame->UpdateWindow();
		return TRUE;
	}
};

// 应用程序对象的全局变量
CMyWinApp theApp;
```



注意我们没有在框架类中定义处理 WM_CREATE 消息，因此由父类处理 WM_CREATE 消息。由于传入了 cct 结构体，LoadFrame 会动态创建视图类对象，并创建视图窗口。如果我们在框架类中定义了处理 WM_CREATE 的函数，会覆盖掉父类函数，导致视图类对象不能够创建出来，因此需要在自定义函数中调用父类函数。修改后的框架类为

```cpp
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	afx_msg int OnCreate(LPCREATESTRUCT pcs)
	{
        // 调用父类函数进行动态创建视图类对象，并创建视图窗口
		return CFrameWnd::OnCreate(pcs);
	}
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
END_MESSAGE_MAP()
```



在处理视图窗口的 WM_CREATE 消息时，会将文档类对象和视图类对象建立关联关系。如果我们用自定义的 OnCreate 函数覆盖了父类的函数，会导致关联关系不能建立，同理需要调用父类函数

```cpp
class CMyView : public CView
{
	DECLARE_DYNCREATE(CMyView)
	DECLARE_MESSAGE_MAP()
public:
	void OnDraw(CDC* pDC)
	{
		pDC->TextOutW(100, 100, L"我是视图窗口");
	}

	afx_msg int OnCreate(LPCREATESTRUCT pcs)
	{
        // 调用父类函数将文档类对象和视图类对象建立关联关系
		return CView::OnCreate(pcs);
	}
};
IMPLEMENT_DYNCREATE(CMyView, CView)
BEGIN_MESSAGE_MAP(CMyView, CView)
	ON_WM_CREATE()
END_MESSAGE_MAP()
```



对象关系

* theApp
    * m_pMainWnd 框架类对象地址 pFrame
        * m_pViewActive 活动视图类对象地址 pView
            * m_pDocument 文档类对象地址 pDoc
                * m_viewList 所有视图类对象地址

视图类对象保存了文档类对象的地址，而文档类成员 m_viewList 链表保存了所有视图类对象地址。**一个视图类对象只对应一个文档类对象，而一个文档类对象可以对应多个视图类对象。**



### 文档类宏

在使用自定义单文档类架构时需要建立一个字符串资源

```cpp
// MFC 已经定义
#define AFX_IDS_UNTITLED                0xF003
```

它是 MFC 在未指定文档标题时使用的字符串资源，但是由于我们是自己写单文档架构，项目中不存在对应的字符串，因此需要手动在字符串资源中添加。具体细节参考单文档架构章节。



### 窗口切分

一个文档类对象可以使用多个视图类对象，通常应用于窗口切分显示。相关父类是不规则框架窗口类 CSplitterWnd，封装了相关操作。我们需要重写 CFrameWnd 类的成员虚函数 OnCreateClient，在该函数中

* 调用虚函数 CSplitterWnd::CreateStatic 创建不规则框架窗口
* 调用虚函数 CSplitterWnd::CreateView 创建视图窗口

```cpp
// 创建不规则框架窗口
BOOL CreateStatic(
    CWnd* pParentWnd,						// 父类框架类
	int nRows, int nCols,					// 切分行数和列数
	DWORD dwStyle = WS_CHILD | WS_VISIBLE,	// 风格
	UINT nID = AFX_IDW_PANE_FIRST			// ID，默认会填满窗口
);

// 创建视图窗口
BOOL CreateView(
    int row, int col, 			// 窗口位置，从 0 开始计算
    CRuntimeClass* pViewClass,	// 视图窗口类的静态常量
	SIZE sizeInit, 				// 窗口规模（一般没用，随便设置）
    CCreateContext* pContext	// 传入 pContext
);
```



当我们在重写的 OnCreateClient 函数中 CreateView 调用创建视图后，每个视图窗口都会产生 WM_CREATE 消息。由于动态创建机制，会自动调用 OnCreate 函数产生视图窗口。

```cpp
#include <afxwin.h>
#include <afxext.h>
#include "resource.h"

class CMyDoc : public CDocument
{
};

// 自定义视图窗口类
class CMyView : public CView
{
	DECLARE_DYNCREATE(CMyView)
	DECLARE_MESSAGE_MAP()
public:
	void OnDraw(CDC* pDC)
	{
		pDC->TextOutW(100, 100, L"我是视图窗口");
	}

	afx_msg int OnCreate(LPCREATESTRUCT pcs)
	{
		return CView::OnCreate(pcs);
	}

	afx_msg void OnNew()
	{
		AfxMessageBox(L"视图类窗口活动");
	}
};
IMPLEMENT_DYNCREATE(CMyView, CView)
BEGIN_MESSAGE_MAP(CMyView, CView)
	ON_WM_CREATE()
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()

// 自定义框架类
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	afx_msg int OnCreate(LPCREATESTRUCT pcs)
	{
		return CFrameWnd::OnCreate(pcs);
	}

	BOOL OnCreateClient(LPCREATESTRUCT lpcs, CCreateContext* pContext)
	{
		// 创建 1 行 2 列的不规则框架窗口
		split.CreateStatic(this, 1, 2);
		// 下面第三个参数的两种写法效果相同，都是 CMyView 的类静态常量
		split.CreateView(0, 0, RUNTIME_CLASS(CMyView), CSize(100, 100), pContext);
		split.CreateView(0, 1, pContext->m_pNewViewClass, CSize(100, 100), pContext);
		// 获取创建出的视图窗口，将其设为活动窗口
		m_pViewActive = (CView*)split.GetPane(0, 0);
		// 注意父类 OnCreateClient 原本会动态创建一个视图窗口
		// 我们重写以后创建两个窗口，不需要再次调用
		return TRUE;
	}

public:
	// 不规则框架窗口，延长生命周期
	CSplitterWnd split;
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
END_MESSAGE_MAP()

// 自定义应用程序类
class CMyWinApp : public CWinApp
{
public:
	virtual BOOL InitInstance()
	{
		CMyFrameWnd* pFrame = new CMyFrameWnd;
		CMyDoc* pDoc = new CMyDoc;

		CCreateContext cct;
		cct.m_pCurrentDoc = pDoc;						// 文档类对象地址
		cct.m_pNewViewClass = RUNTIME_CLASS(CMyView);	// CMyView 的静态常量地址

		// 加载框架，会自动创建窗口
		pFrame->LoadFrame(IDR_MENU1, WS_OVERLAPPEDWINDOW, NULL, &cct);
		m_pMainWnd = pFrame;
		pFrame->ShowWindow(SW_SHOW);
		pFrame->UpdateWindow();
		return TRUE;
	}
};

// 应用程序对象的全局变量
CMyWinApp theApp;
```

![[image-20221026103346991.png]]



### 刷新窗口

我们在自定义文档类中添加一个字符串变量，将这个字符串变量输出到视图窗口中。步骤如下：

* 获取和视图类对象关联的文档类对象，调用 GetDocument()
* 当文档类数据发生变化时，调用 UpdateAllViews 刷新和文档类对象相关联的视图类对象，即视图窗口

我们修改上面代码中的文档和视图部分

```cpp
class CMyDoc : public CDocument
{
	DECLARE_MESSAGE_MAP()
public:

	afx_msg void OnNew()
	{
		this->str = L"Hello World";
		// 刷新和这个文档类对象关联的所有视图窗口
		this->UpdateAllViews(NULL);	
	}

public:
	CString str;
};

BEGIN_MESSAGE_MAP(CMyDoc, CDocument)
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()

// 自定义视图窗口类
class CMyView : public CView
{
	DECLARE_DYNCREATE(CMyView)
	DECLARE_MESSAGE_MAP()
public:
	afx_msg int OnCreate(LPCREATESTRUCT pcs)
	{
		return CView::OnCreate(pcs);
	}

	afx_msg void OnDraw(CDC* pDC)
	{
        // 获取与窗口关联的文档类指针进行绘制
        // 因为 m_pDocument 是一个保护成员，在类外访问需要调用此函数，为了保持一致，建议使用 GetDocument
		// CMyDoc* pDoc = (CMyDoc*)this->m_pDocument;
        CMyDoc* pDoc = (CMyDoc*)this->GetDocument();
		pDC->TextOutW(100, 100, pDoc->str);
	}
};

IMPLEMENT_DYNCREATE(CMyView, CView)
BEGIN_MESSAGE_MAP(CMyView, CView)
	ON_WM_CREATE()
	ON_WM_PAINT()
END_MESSAGE_MAP()
```



如果我们希望不刷新某个特定的视图窗口，可以向 UpdateAllViews 中传入**不希望刷新的窗口指针**

```cpp
POSITION pos = this->GetFirstViewPosition();
CView* pView = this->GetNextView(pos);
// 刷新除了 pView 以外和这个文档类对象关联的所有视图窗口
this->UpdateAllViews(pView);
```

我们在文档类中获取视图窗口指针，又在视图类中获取文档类指针。





## 标准架构方案

这一部分我们给出 MFC 程序的两种标准架构方案，实际应用中在这两个架构的基础上进行修改即可。



### 单文档架构

单文档架构使用文档模板类 CDocTemplate 的子类单文档模板类 CSingleDocTemplate 协调创建。



文档类、视图类和框架类全部使用动态创建机制，由应用程序类中的单文档模板类自动创建。我们还需要建立一个字符串资源

```cpp
// MFC 已经定义
#define AFX_IDS_UNTITLED                0xF003
```

它是 MFC 在未指定文档标题时使用的字符串资源，但是由于我们是自己写单文档架构，项目中不存在对应的字符串，因此需要手动在字符串资源中添加。注意 `AFX_IDS_UNTITLED` 的值必须是 `0xF003`，标题可以随意取。

```cpp
#include <afxwin.h>
#include <afxext.h>
#include "resource.h"

// 文档类，管理数据的类，封装了对数据的管理，并和视图类进行数据交互
class CMyDoc : public CDocument
{
	DECLARE_DYNCREATE(CMyDoc)
};
IMPLEMENT_DYNCREATE(CMyDoc, CDocument)

// 视图类，负责显示数据
class CMyView : public CView
{
	DECLARE_DYNCREATE(CMyView)
public:
	virtual void OnDraw(CDC* pDC)
	{
		pDC->TextOutW(100, 100, L"视图窗口");
	}
};
IMPLEMENT_DYNCREATE(CMyView, CView)

// 框架类，作为文档显示的框架
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_DYNCREATE(CMyFrameWnd)
};
IMPLEMENT_DYNCREATE(CMyFrameWnd, CFrameWnd)

// 应用程序类
class CMyWinApp : public CWinApp
{
public:
	virtual BOOL InitInstance()
	{
        // 创建流程
		CSingleDocTemplate* pTemplate = new CSingleDocTemplate(
			IDR_MENU1,
			RUNTIME_CLASS(CMyDoc),
			RUNTIME_CLASS(CMyFrameWnd),
			RUNTIME_CLASS(CMyView)
        );
        // 添加模板
		AddDocTemplate(pTemplate);
        // 创建新的窗口
		OnFileNew();
		m_pMainWnd->ShowWindow(SW_SHOW);
		m_pMainWnd->UpdateWindow();
		return TRUE;
	}
};

// 应用程序对象的全局变量
CMyWinApp theApp;
```



### 多文档架构

多文档架构使用多文档模板类 CMultiDocTemplate 协调创建。



类似于单文档架构，区别在于主框架窗口不再直接继承自 CFrameWnd，而是继承其子类 CMDIFrameWnd，这个类不支持动态创建；为了产生多文档，还需要继承自 CMDIChildWnd 的子框架窗口类。

```cpp
#include <afxwin.h>
#include <afxext.h>
#include "resource.h"

class CMyDoc : public CDocument
{
	DECLARE_DYNCREATE(CMyDoc)
};
IMPLEMENT_DYNCREATE(CMyDoc, CDocument)

// 自定义视图窗口类
class CMyView : public CView
{
	DECLARE_DYNCREATE(CMyView)
public:
	virtual void OnDraw(CDC* pDC)
	{
		pDC->TextOutW(100, 100, L"视图窗口");
	}
};
IMPLEMENT_DYNCREATE(CMyView, CView)

// 子框架窗口类
class CMyChild : public CMDIChildWnd
{
	DECLARE_DYNCREATE(CMyChild)
};
IMPLEMENT_DYNCREATE(CMyChild, CMDIChildWnd)

// 主框架窗口类，注意此类继承自 CMDIFrameWnd，不支持动态创建
class CMyFrameWnd : public CMDIFrameWnd
{
};

// 自定义应用程序类
class CMyWinApp : public CWinApp
{
public:
	virtual BOOL InitInstance()
	{
        // 由于不能动态创建，需要手动 new 一个主框架窗口
		CMyFrameWnd* pFrame = new CMyFrameWnd;
        // 挂载了第一个菜单，并显示
		pFrame->LoadFrame(IDR_MENU1);
		m_pMainWnd = pFrame;
		pFrame->ShowWindow(SW_SHOW);
		pFrame->UpdateWindow();
		
        // 创建多文档模板，挂载第二个菜单
		CMultiDocTemplate* pTemplate = new CMultiDocTemplate(
			IDR_MENU2,
			RUNTIME_CLASS(CMyDoc),
			RUNTIME_CLASS(CMyChild),
			RUNTIME_CLASS(CMyView)
		);
        // 添加模板
		AddDocTemplate(pTemplate);
        // 创建三个子窗口
		OnFileNew();
		OnFileNew();
		OnFileNew();

		return TRUE;
	}
};

// 应用程序对象的全局变量
CMyWinApp theApp;
```

多文档架构中创建的每个子窗口单独关联一个文档类，并且当子窗口开启时，显示多文档模板挂载的菜单；当关闭所有子窗口后，显示主框架窗口的菜单。每当调用 OnFileNew 就会创建一个子窗口。



### 类向导

从现在开始，我们正式创建 MFC 项目，即 MFC 桌面应用。一般选择单文档程序，项目样式为标准 MFC 程序，去掉多余的用户界面功能，例如去掉初始状态栏、使用经典菜单等，就得到一个自动生成的项目模板。

标准项目模板中将给出我们前面学习的应用程序类、框架类、视图类和文档类结构。包括消息映射、动态创建机制、类的初始化过程等都已经自动写好了。我们在这一基础上添加成员变量、成员函数或重载消息处理函数即可。



类向导是 VS 提供为指定类快捷添加成员变量、成员函数和消息处理的工具。在“项目-类向导”打开向导窗口。

![[image-20221026215653218.png]]

选择要操作的类名，在下面有几个可修改的选项：命令、消息、虚函数、成员变量、方法。如果要添加命令消息的处理，首先搜索命令消息的 ID，然后双击 ID 即可创建消息处理函数；如果要处理标准消息，在“消息”选项中搜索对应的消息，双击自动添加对消息处理函数的重载代码；如果要重载某个虚函数，可以在这里搜索父类虚函数进行重载。

**注意：如果资源文件保持打开状态，就不能添加命令消息的处理代码！！！**



## MFC 绘图

绘图设备类 CDC 封装了各种绘图相关的函数，以及两个重要的成员变量 m_hDC 和 m_hAttribDC 。



### 消息绘图

子类 CPaintDC 封装了在 WM_PAINT 消息中绘图的绘图设备句柄 HDC，我们可以通过该类的对象实现绘图操作。



使用类向导，我们为视图窗口添加一个处理 WM_PAINT 消息的虚函数重写来绘制图形

```cpp
void CMyView::OnPaint()
{
	CPaintDC dc(this); // device context for painting
					   // TODO: 在此处添加消息处理程序代码
					   // 不为绘图消息调用 CView::OnPaint()
	dc.Rectangle(100, 100, 300, 300);
}
```

CPaintDC 对象封装了各种 Win32 的绘图函数作为成员函数，使用时直接调用即可。



子类 CClientDC 封装了在客户区绘图的绘图设备。添加处理 WM_COMMAND 消息的处理函数，通过菜单点击触发消息，然后调用这一函数进行绘图

```cpp
// 处理菜单消息时触发
void CMyView::OnNew()
{
	// TODO: 在此添加命令处理程序代码
	CClientDC dc(this);
	dc.Ellipse(200, 100, 400, 300);
}
```

这两种绘图方式的区别在于：OnPaint 在每个 WM_PAINT 消息触发都会调用，因此只要窗口刷新就会保持绘图；而 OnNew 是通过点击菜单调用函数绘图，因此窗口刷新后就会消失。



### 禁用重绘

由于我们每次重绘都会使用背景色先将背景重新刷新，然后再绘制我们想绘制的图像，因此容易发生闪烁。可以重载 WM_ERASEBKGND 消息，取消默认的背景绘制

```cpp
BOOL CMyView::OnEraseBkgnd(CDC* pDC)
{
	// 禁用重绘背景，防止闪烁
	return TRUE;
}
```



### 双缓冲绘图

先创建一个缓冲内存，载入一张位图用于绘图，最后将绘制好的内容一次性复制到当前内存中，从而解决绘图闪烁的问题。需要注意，**由于我们在一张位图中绘制后直接将其复制到内存中，因此可以省略默认的窗口重绘，只有这样才能防止闪烁**，即

```cpp
this->InvalidateRect(NULL, FALSE);
```

无效窗口的自动刷新参数设为 FALSE 来关闭自动重绘。

```cpp
void CMyFrameWnd::OnPaint()
{
	CPaintDC dc(this); // device context for painting
					   // TODO: 在此处添加消息处理程序代码
					   // 不为绘图消息调用 CFrameWnd::OnPaint()
	if (m_onDraw)
	{
		// 获取客户区大小
		CRect winRect;
		this->GetClientRect(&winRect);

		// 创建内存 dc
		CDC memdc;
		memdc.CreateCompatibleDC(&dc);

		// 创建兼容的位图
		CBitmap bitmap;
		bitmap.CreateCompatibleBitmap(&memdc, winRect.Width(), winRect.Height());

		// 选入位图，并填充背景
		CBitmap* oldBmp = memdc.SelectObject(&bitmap);
		memdc.FillSolidRect(winRect, RGB(255, 255, 255));
		
        // 在 memdc 中绘图
		
		// 将内存 dc 中的图像输出到当前 dc
		dc.BitBlt(0, 0, winRect.Width(), winRect.Height(), &memdc, 0, 0, SRCCOPY);

		// 选入旧的位图
		memdc.SelectObject(oldBmp);

		// 释放资源
		bitmap.DeleteObject();
		memdc.DeleteDC();
	}
}
```

此时创建的位图是单色位图，如果想要绘制彩色图，需要使用

```cpp
// 创建位图
BOOL CreateBitmap(
    int nWidth,			// 宽度
    int nHeight,		// 高度
    UINT nPlanes,		// 颜色平面数
    UINT nBitcount,		// 每个显示像素的颜色位的数量
    const void* lpBits	// 包含的初始位图位值的字节的数组。如果它是 NULL，新位图则未初始化。
);
```

对于彩色位图，应当设置 nPlanes 为 1，nBitcount 为 32；如果它们均为 1，则产生单色位图。



便于使用的双缓冲类，在框架类中定义双缓冲成员变量，传入 this 指针，绘图时使用返回的内存 DC 的引用。

```cpp
class DoubleBuffer
{
public:
	DoubleBuffer(CFrameWnd* pWnd) : m_pWnd(pWnd), m_oldBmp(nullptr) {}

public:
	CDC& beginPaint(CDC& dc)
	{
		// 获取客户区大小
		CRect winRect;
		m_pWnd->GetClientRect(&winRect);

		// 创建内存 dc
		m_memdc.CreateCompatibleDC(&dc);

		// 创建兼容的位图
		m_bitmap.CreateBitmap(winRect.Width(), winRect.Height(), 1, 32, 0);

		// 选入位图，并填充背景
		m_oldBmp = m_memdc.SelectObject(&m_bitmap);
		m_memdc.FillSolidRect(winRect, RGB(255, 255, 255));

		return m_memdc;
	}

	void endPaint(CDC& dc)
	{
		// 获取客户区大小
		CRect winRect;
		m_pWnd->GetClientRect(&winRect);

		// 将内存 dc 中的图像输出到当前 dc
		dc.BitBlt(0, 0, winRect.Width(), winRect.Height(), &m_memdc, 0, 0, SRCCOPY);

		// 选入旧的位图
		m_memdc.SelectObject(m_oldBmp);

		// 释放资源
		m_bitmap.DeleteObject();
		m_memdc.DeleteDC();
	}

public:
	CFrameWnd* m_pWnd;
	CDC m_memdc;
	CBitmap m_bitmap;
	CBitmap* m_oldBmp;
};
```



### 绘图对象类

CGdiObject 类封装了各种绘图对象的操作，保存了一个重要成员变量 m_hObject 绘图对象句柄。包括画笔、画刷、字体、位图等 Win32 中的结构都进行了封装。

```cpp
void CMyView::OnPaint()
{
	CPaintDC dc(this); // device context for painting
					   // TODO: 在此处添加消息处理程序代码
					   // 不为绘图消息调用 CView::OnPaint()

	// 画笔绘图
	CPen pen(PS_SOLID, 2, RGB(255, 0, 0));
	CPen* oldpen = dc.SelectObject(&pen);
	dc.Rectangle(100, 100, 300, 300);
	dc.SelectObject(oldpen);
	pen.DeleteObject();

	// 画刷绘图
	CBrush brush(RGB(255, 0, 0));
	CBrush* oldbrush = dc.SelectObject(&brush);
	dc.Rectangle(400, 100, 600, 300);
	dc.SelectObject(oldbrush);
	pen.DeleteObject();

	// 修改字体
	CFont font;
	font.CreatePointFont(300, L"黑体");
	CFont* oldfont = dc.SelectObject(&font);
	dc.TextOutW(100, 300, L"Hello World");
	dc.SelectObject(oldfont);
	font.DeleteObject();

	// 绘制位图
	CDC memdc;
	memdc.CreateCompatibleDC(&dc);
	CBitmap bmp;
	bmp.LoadBitmapW(IDR_TOOLBAR1);
	CBitmap* oldbmp = memdc.SelectObject(&bmp);
	dc.BitBlt(100, 400, 48, 48, &memdc, 0, 0, SRCCOPY);
	memdc.SelectObject(oldbmp);
	bmp.DeleteObject();
	memdc.DeleteDC();
}
```

上述结构的使用过程本质上和 Win32 一致，区别仅在于现在调用的函数是成员函数。



### CImage

头文件 `atlimage.h` 中定义了 CImage 类提供增强的位图支持，能够加载和保存 JPEG、GIF、BMP 和可移植网络图形格式 (PNG) 的图像。



通过 Create 函数创建位图

```cpp
BOOL Create(
    int nWidth,			// 图像宽度
    int nHeight,		// 图像高度
    int nBPP,			// 像素位数，决定颜色显示，一般是 4 8 ... 32
    DWORD dwFlags = 0	// 指定位图对象是否有 alpha 通道
);
```

可以将位图附加到图像中

```cpp
void Attach(
    HBITMAP hBitmap, 
    DIBOrientation eOrientation = DIBOR_DEFAULT
);

// 移除并销毁位图
void Destroy();
```

指定位图的方向。 可以是以下值之一：

- `DIBOR_DEFAULT` 位图的方向由操作系统确定
- `DIBOR_BOTTOMUP` 位图的行按相反顺序排列
- `DIBOR_TOPDOWN` 位图的行按从上到下的顺序排列



使用 Load 函数加载图像

```cpp
// 加载制定文件图像
HRESULT Load(LPCTSTR pszFileName);

// 从资源中加载位图
void LoadFromResource(
    HINSTANCE hInstance,
    LPCTSTR pszResourceName
);
void LoadFromResource(
    HINSTANCE hInstance,
    UINT nIDResource
);

// 判断是否为空
BOOL IsNULL();
```

通过 Get 函数获取图像尺寸信息

```cpp
int GetWidth();
int GetHeight();
```



CImage 可以显示透明或半透明的图像

```cpp
BOOL AlphaBlend(
    HDC hDestDC,						// 目标设备句柄
    int xDest, int yDest,				// 目标 x,y 坐标
    int nDestWidth,	int nDestHeight,	// 目标宽高
    int xSrc, int ySrc,					// 源文件截取位置
    int nSrcWidth, int nSrcHeight,		// 源文件宽高
    BYTE bSrcAlpha = 0xff,				// 使用的 alpha 透明度值，默认不透明
    BYTE bBlendOp = AC_SRC_OVER
);
```



获取和设置像素颜色

```cpp
// 如果位于裁剪区外，返回 CLR_INVALID
COLORREF GetPixel(
    int x, int y
);

void SetPixel(
    int x, int y, 
    COLORREF color
);
```



CImage 中 Draw 函数结合了 BitBlt、StretchBlt 和 TransparentBlt 函数的功能，直接使用它即可

```cpp
BOOL Draw(
    HDC hDestDC,
    int xDest, int yDest,
    int nDestWidth, int nDestHeight,	// 默认为图像宽高
    int xSrc, int ySrc,					// 默认为 0,0
    int nSrcWidth, int nSrcHeight		// 默认为图像宽高
);
```

Draw 函数会自动根据源图像的尺寸和目标尺寸进行伸缩。如果图像伸缩，有可能导致失真，这时需要设置变形模式

```cpp
::SetStretchBltMode(dc, HALFTONE);
::SetBrushOrgEx(dc, 0, 0, NULL);
m_img.Draw(dc, rect);
```



修改图像后，可以将其保存为指定格式。通常是在位图中绘制完成后，将其附加到 CImage 对象中调用 Save 进行保存

```cpp
HRESULT Save(
    LPCTSTR pszFileName,				// 图像文件名
    REFGUID guidFileType = GUID_NULL	// 文件类型
)
```

保存图像的文件类型可以是以下值之一：

- `ImageFormatBMP` 未压缩的位图图像
- `ImageFormatPNG` 可移植网络图形格式 (PNG) 压缩图像
- `ImageFormatJPEG` JPEG 压缩图像
- `ImageFormatGIF` GIF 压缩图像



要进行像素操作，需要熟悉如下几个函数

```cpp
void* GetBits();		// 返回图像缓冲区像素指针
int GetBPP() const;		// 返回每个像素的位数
int GetPitch() const;	// 返回像素间距
```

其中 GetBits 获得的是图像开始位置的像素指针，根据图像的类型，需要用 GetPitch 获得像素排列的方位，因为有些图像的像素是反向排列，它会返回负值，我们用它进行遍历。GetBPP 获取的是每个像素占的位数，灰度图一般是 8 或 32，三通道 RGB 是 24 。





## 序列化机制

### 文件操作

文件操作类 [CFile](https:// learn.microsoft.com/zh-cn/cpp/mfc/reference/cfile-class?view=msvc-170) 封装了关于文件读写的操作。我们列举常用的成员函数，更多操作包括重命名文件、删除文件等操作点击链接查看。



注意宽字符不能直接输出，需要转换为窄字符。为了避免这一麻烦，下面是使用**多字节字符集**后的文件操作流程

```cpp
#include <afxwin.h>
#include <iostream>

using namespace std;

void File()
{
	CFile file;
	file.Open("test.txt", CFile::modeCreate | CFile::modeReadWrite);
	char str[] = "hello file";
	file.Write(str, strlen(str));
	file.SeekToBegin();
	char buf[256] = { 0 };
	long nLen = file.Read(buf, 255);
	cout << buf << " " << nLen << endl;
	file.Close();
}

int main()
{
	File();
	return 0;
}
```



#### Open

打开函数，返回是否打开成功

```cpp
BOOL Open(
    LPCTSTR lpszFileName, 	// 文件名
    UINT nOpenFlags, 		// 打开方式
    CFileException* pError = NULL
);
```

flags

* CFile::modeCreate 如果不存在文件，就创建文件
* CFile::modeWrite 写入文件
* CFile::modeRead 读取文件
* CFile::modeReadWrite 写入和读取

不同方式可用按位或 | 连接。



#### Read

读取函数，返回读取长度

```cpp
UINT Read(
    void* lpBuf, 	// 缓冲区，读取内容存放在这里
    UINT nCount		// 缓冲区大小
);
```



#### Write

写入缓冲区的内容

```cpp
void Write(
    const void* lpBuf, 	// 写入的内容
    UINT nCount			// 缓冲区大小
);
```



#### Seek

移动文件指针

```cpp
ULONGLONG Seek(
    LONGLONG lOff,		// 搜索长度，为正则向后搜索，为负则向前搜索
    UINT nFrom			// 指定开始搜索的位置
);

ULONGLONG SeekToEnd();	// 文件指针达到尾部，返回文件长度
void SeekToBegin();		// 文件指针到达头部
```

from

* CFile::begin 从开头查找
* CFile::current 从当前位置查找
* CFile::end 从结尾查找



#### Close

关闭同时销毁文件对象

```cpp
void Close();
```



### 序列化操作

序列化机制以二进制流形式读写硬盘文件，效率很高，比标准库中的文件操作更高效。它使用归档类 [CArchive](https:// learn.microsoft.com/zh-cn/cpp/mfc/reference/carchive-class?view=msvc-170) 完成读写操作。

```cpp
class CArchive
{
public:
	enum Mode { store = 0, load = 1, bNoFlushOnDelete = 2, bNoByteSwap = 4 };
    ...
        
protected:
    BOOL m_nMode;		// 访问方式
	BOOL m_bUserBuf;	// 缓冲区
	int m_nBufSize;		// 缓冲区大小
	CFile* m_pFile;		// 文件指针
	BYTE* m_lpBufCur;	// 当前位置
	BYTE* m_lpBufMax;	// 终止位置
	BYTE* m_lpBufStart;	// 开始位置
    ...
};


// 构造函数
CArchive(
    CFile* pFile,			// 文件指针
    UINT nMode,				// 打开方式
    int nBufSize = 4096,	// 缓冲区大小
    void* lpBuf = NULL		// 指定缓冲区
);
```

mode

* CArchive::load 需要文件读取权限
* CArchive::store 需要文件写入权限

如果指定缓冲区，则会使用该缓冲区进行文件读写操作，否则系统在堆空间中申请缓冲区，并且当对象析构时会释放申请的缓冲区。



#### 写入操作

ar 对象维护一个缓冲区，将各个数据依次序列化存储到缓冲区中，并将 m_lpBufCur 指针移动到相应字节。如果缓冲区不足，就先将数据写入硬盘文件，并重置 m_lpBufCur 为开始指向。当关闭 ar 对象时，将缓冲区数据写入硬盘文件，并释放维护的缓冲区。

```cpp
#include <afxwin.h>
#include <iostream>

using namespace std;

// 序列化操作
void Store()
{
	CFile file;
	file.Open("test.txt", CFile::modeCreate | CFile::modeWrite);
	CArchive ar(&file, CArchive::store, 4096);
	long age = 18;
	ar << age;
	float score = 88.5;
	ar << score;
	CString name = "ZhanSan";
	ar << name;
	ar.Close();
	file.Close();
}

int main()
{
	Store();
	return 0;
}
```



#### 读取操作

```cpp
#include <afxwin.h>
#include <iostream>

using namespace std;

// 反序列化操作
void Load()
{
	CFile file;
	file.Open("test.txt", CFile::modeRead);
	CArchive ar(&file, CArchive::load, 4096);
	long age;
	ar >> age;
	float score;
	ar >> score;
	CString name;
	ar >> name;
	ar.Close();
	file.Close();

	cout << age << " " << score << " " << name << endl;
}

int main()
{
	Load();
	return 0;
}
```



#### 序列化类对象

此机制可以用于储存类的信息。使用的类必须满足如下条件

* 类要派生自 CObject
* 类内必须添加声明宏 DECLARE_SERIAL(theClass)
* 类外必须添加实现宏 IMPLEMENT_SERIAL(theClass, baseClass, 1)，最后一项是版本号，可以随意填
* 类必须重写虚函数 Serialize

```cpp
#include <afxwin.h>
#include <iostream>

using namespace std;

class CMyDoc : public CDocument
{
	DECLARE_SERIAL(CMyDoc)
public:
	CMyDoc(int age = 0, float score = 0, CString name = "") :
		m_age(age), m_score(score), m_name(name) {}
	
    // 重写序列化操作
	virtual void Serialize(CArchive& ar)
	{
		if (ar.IsStoring())
		{
			ar << m_age << m_score << m_name;
		}
		else
		{
			ar >> m_age >> m_score >> m_name;
		}
	}

public:
	int m_age;
	float m_score;
	CString m_name;
};
IMPLEMENT_SERIAL(CMyDoc, CDocument, 1)

void Store()
{
	CFile file;
	file.Open("test.txt", CFile::modeCreate | CFile::modeWrite);
	CArchive ar(&file, CArchive::store, 4096);
	CMyDoc data(18, 88.5, "ZhangSan");
    // 注意输出的是对象指针
	ar << &data;
	ar.Close();
	file.Close();
}

void Load()
{
	CFile file;
	file.Open("test.txt", CFile::modeRead);
	CArchive ar(&file, CArchive::load, 4096);
    // 初始化为空指针
	CMyDoc* pdata = NULL;
    // 这一步操作 pdata 将指向 MFC 动态创建的 CMyDoc 对象
	ar >> pdata;
	ar.Close();
	file.Close();

	cout << pdata->m_age << " " << pdata->m_score << " " << pdata->m_name << endl;
}

int main()
{
	Store();
	Load();
	return 0;
}
```



## 对话框

MFC 包含模式对话框（假）和无模式对话框。其中模式对话框是通过无模式对话框加上对窗口的禁用得到的假模式对话框。

* 在进行控件和对象或数据的绑定时，一定要先重载 DoDataExchange 函数，否则会绑定失败。
* 注意当要使用两个不同的对话框时，应当定义两个不同的类，因为它们的不同可能绑定了不同的数据或对象，因此不能共有



### 无模式对话框

首先添加对话框资源，自定义对话框类 CMyNoModleDlg，管理对话框资源，派生自 CDialog 或 CDialogEx 均可。只需要先在资源中添加对话框，在图形界面中边界对话框的 ID 和控件等。

```cpp
// 自定义无模式对话框
class CMyNoModleDlg : public CDialog
{
};

// 自定义框架类
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	int OnCreate(LPCREATESTRUCT pcs)
	{
		menu.LoadMenu(IDR_MENU1);
		// 调用 CMenu 的成员函数设置菜单
		this->SetMenu(&menu);
		return CFrameWnd::OnCreate(pcs);
	}

	void OnNew()
	{
        // 创建无模式对话框的指针，调用 Create 进行窗口创建
		CMyNoModleDlg* pdlg = new CMyNoModleDlg;
		pdlg->Create(IDD_DIALOG1);
		pdlg->ShowWindow(SW_SHOW);
	}

public:
	// 定义为成员变量，确保生命周期与窗口一致
	CMenu menu;
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()
```

在 CMyNoModleDlg 类中建立消息映射，通过 WM_COMMAND 消息可以实现对控件按钮消息的处理。



默认创建出的对话框自带有 IDOK 和 IDCANCEL 按钮，并且 CDialog 中有对应的 OnOK 和 OnCancel 虚函数处理对应的消息。但是无模式对话框自带的 OnOK 和 OnCancel 函数只会隐藏对话框，并不会销毁对话框。这就需要我们重载对应的消息处理函数

```cpp
class CMyNoModleDlg : public CDialog
{
	DECLARE_MESSAGE_MAP()
public:
	void OnOK()
	{
		::DestroyWindow(this->m_hWnd);
	}

	void OnCancel()
	{
		::DestroyWindow(this->m_hWnd);
	}

	void OnDestroy()
	{
		AfxMessageBox(L"Destroyed");
	}
};

BEGIN_MESSAGE_MAP(CMyNoModleDlg, CDialog)
	ON_WM_DESTROY()
	ON_COMMAND(IDOK, OnOK)
	ON_COMMAND(IDCANCEL, OnCancel)
END_MESSAGE_MAP()
```



### 模式对话框

我们之前提到，MFC 中的模式对话框是从无模式对话框修改得到的。在创建模式对话框时调用 DoModal 函数，其中包含了禁用父窗口、创建无模式对话框、开启对话框消息循环、结束消息循环、启用父窗口、销毁无模式对话框等一系列过程。

```cpp
// 自定义模式对话框
class CMyModleDlg : public CDialog
{
public:
    // 定义枚举变量，用于初始化对话框资源
	enum { IDD = IDD_DIALOG2 };
    // 调用父类构造函数直接获取对话框资源
	CMyModleDlg() : CDialog(IDD) {}

	DECLARE_MESSAGE_MAP()
public:
	void OnDestroy()
	{
		AfxMessageBox(L"Destroyed");
	}
};

BEGIN_MESSAGE_MAP(CMyModleDlg, CDialog)
	ON_WM_DESTROY()
END_MESSAGE_MAP()

// 自定义框架类
class CMyFrameWnd : public CFrameWnd
{
	DECLARE_MESSAGE_MAP()
public:
	int OnCreate(LPCREATESTRUCT pcs)
	{
		menu.LoadMenu(IDR_MENU1);
		// 调用 CMenu 的成员函数设置菜单
		this->SetMenu(&menu);
		return CFrameWnd::OnCreate(pcs);
	}

	void OnNew()
	{
        // 创建对话框对象
		CMyModleDlg dlg;
		dlg.DoModal();
	}

public:
	// 定义为成员变量，确保生命周期与窗口一致
	CMenu menu;
};

BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
	ON_COMMAND(ID_NEW, OnNew)
END_MESSAGE_MAP()
```

注意模式对话框通过对象即可创建，不需要再加载对话框资源，并且它会在 DoModal 函数中自动创建和销毁。



### 文件对话框

CFileDialog 类封装用于文件打开操作或文件保存操作的常见对话框，此类需要 afxext.h 头文件。

```cpp
explicit CFileDialog(
    BOOL bOpenFileDialog, 					// 指定对话框类型，TRUE 表示打开文件，FALSE 表示保存文件
	LPCTSTR lpszDefExt = NULL,				// 默认文件扩展名。若为 NULL 则不会追加扩展名
	LPCTSTR lpszFileName = NULL,			// 初始文件名。如果为 NULL，不显示初始文件名
	DWORD dwFlags = OFN_HIDEREADONLY | OFN_OVERWRITEPROMPT,	// 风格
	LPCTSTR lpszFilter = NULL,				// 用于筛选文件
	CWnd* pParentWnd = NULL,				// 窗口指针
	DWORD dwSize = 0,						// OPENFILENAME 结构的大小。为 0 表示自动确定。
	BOOL bVistaStyle = TRUE					// 新样式对话框
);
```

例如我们可以通过调用此对话框读取文件

```cpp
void CMyDlg::OnBnClickedLoad()
{
	// 设置筛选器，允许打开 obj,dat 文件
	TCHAR szFilter[] = TEXT("3D Files (*.obj,*.dat)|*.obj;*.dat|");
	CFileDialog cfdlg(TRUE, 0, 0, OFN_HIDEREADONLY | OFN_OVERWRITEPROMPT, szFilter);

	// DoModal 返回 IDOK 或 IDCANCEL
	if (cfdlg.DoModal() == IDOK)
	{
		// 获取文件路径名和扩展名
		CString name = cfdlg.GetPathName();
		CString ext = cfdlg.GetFileExt();
	}
}
```

类似地可以保存文件

```cpp
void CMyDlg::OnBnClickedSave()
{
	// 设置筛选器，只允许保存为 .dat 格式文件
	TCHAR szFilter[] = TEXT("Data Files (*.dat)|");
	CFileDialog cfdlg(FALSE, 0, 0, OFN_HIDEREADONLY | OFN_OVERWRITEPROMPT, szFilter);

	if (cfdlg.DoModal() == IDOK)
	{
		// 获取路径，添加后缀
		CString name = cfdlg.GetPathName() + L".dat";
	}
}
```



### 消息对话框

这里的消息对话框是对 Win32 中 MessageBox 对话框的封装

```cpp
int MessageBox(
    HWND hWnd,				// 拥有消息框的窗口
    LPCWSTR lpText,			// 显示的字符串
    LPCWSTR lpCaption,		// 作为标题的字符串
    UINT uType				// 指定消息框的内容
);
```

uType 标识

* MB_ABORTRETRYIGNORE 含有 Abort、Retry 和 Ignore 按钮的消息框
* MB_ICONSTOP 含有停止图标的消息框
* MB_OK 含有一个 OK 按钮的消息框
* MB_OKCANCLE 含有 OK 和 CANCEL 按钮的消息框
* MB_YESNOCANCLE 含有 YES、NO 和 CANCLE 按钮的消息框

封装后的函数是 AfxMessageBox，区别在于不需要第一个窗口句柄参数，只需要直接在类的成员函数中调用。例如

```cpp
AfxMessageBox(L"是否删除？", MB_OKCANCEL | MB_ICONWARNING, 0);
```

这表示带有 OK 和 CANCEL 按钮，图标是警告的消息框。通过返回值 IDOK / IDCANCEL 获得点击的结果

```cpp
if (AfxMessageBox(L"是否删除？", MB_OKCANCEL | MB_ICONWARNING, 0) == IDOK)
{
    // ...
}
```





## 控件

MFC 封装构建了大量有用的控件对象。借助图形化界面，可以直接拖放来构建对话框所需的各种控件。要操作这些控件，我们需要先对控件与对话框成员变量进行绑定，然后通过这些成员变量来操作这些控件。



### 绑定控件

MFC 通过控件与成员变量的绑定来实现对控件的直接操作。有两种绑定方式

* 绑定数据对象：这种绑定适合于直接操作数据的控件
* 绑定控件对象：这种绑定适合于需要改变控件形态的复杂控件



#### 绑定数据对象

重写父类成员虚函数 DoDataExchange 在函数内部通过一系列 DDX_xxx 函数，实现数据类型对象和控件的交互。当我们希望进行数据交互时，调用 UpdateData 函数发出数据交互的消息，从而调用 DoDataExchange 来实现交互。**注意一定要先重写 DoDataExchange 函数才可以进行绑定，否则会失败！**

* UpdateData(TRUE) 控件数据存放到变量
* UpdateData(FALSE) 变量数据存放到控件

**无模式对话框不能绑定，因此我们使用模式对话框。**



添加控件后，可以右键添加事件处理程序，系统会自动添加指定的事件处理代码。下面消息处理代码通过类向导生成后整理得到

```cpp
#include <afxwin.h>
#include "resource.h"

// 自定义模式对话框
class CMyModleDlg : public CDialog
{
	DECLARE_MESSAGE_MAP()
public:
	enum { IDD = IDD_DIALOG2 };
	CMyModleDlg() : CDialog(IDD) {}

	afx_msg void OnBnClickedButton2()
	{
		// 传递编辑框数据到字符串变量
		UpdateData(TRUE);
	}

	afx_msg void OnBnClickedButton1()
	{
		// 传递字符串变量到编辑框
		UpdateData(FALSE);
	}

	virtual void DoDataExchange(CDataExchange* pDX)
	{
		// 绑定编辑框控件和字符串，可通过类向导添加
		DDX_Text(pDX, IDC_EDIT1, m_editstr);
	}

public:
	CString m_editstr;
};

BEGIN_MESSAGE_MAP(CMyModleDlg, CDialog)
	ON_BN_CLICKED(IDC_BUTTON2, &CMyModleDlg::OnBnClickedButton2)
	ON_BN_CLICKED(IDC_BUTTON1, &CMyModleDlg::OnBnClickedButton1)
END_MESSAGE_MAP()


// 自定义应用程序类
class CMyWinApp : public CWinApp
{
public:
	virtual BOOL InitInstance()
	{
        // 使用模式对话框作为主窗口
		CMyModleDlg dlg;
		m_pMainWnd = &dlg;
		dlg.DoModal();
		return TRUE;
	}
};

// 应用程序对象的全局变量
CMyWinApp theApp;
```



要通过类向导添加控件与数据对象的绑定，首先在类向导中的虚函数选项中双击 DoDataExchange 创建该函数，然后在成员变量中选择编辑框控件的 ID，双击添加绑定的对象，选择类别为“值”，然后自定义变量名，点击完成即可生成绑定的代码。

![[image-20221027195659287.png]]



#### 绑定控件对象

通过控件对象绑定，可以实现对象表示整个控件。实现方式与上面类似，使用类向导绑定时选择类别为“控件”，其余同理

```cpp
#include <afxwin.h>
#include "resource.h"

// 自定义模式对话框
class CMyModleDlg : public CDialog
{
	DECLARE_MESSAGE_MAP()
public:
	enum { IDD = IDD_DIALOG2 };
	CMyModleDlg() : CDialog(IDD) {}

	virtual void DoDataExchange(CDataExchange* pDX)
	{
        // 绑定编辑框和控件对象，可通过类向导添加
		DDX_Control(pDX, IDC_EDIT1, m_edit);
	}

	afx_msg void OnBnClickedOk()
	{
        // 当点击确定按钮时移动编辑框
		m_edit.MoveWindow(0, 0, 200, 200);
	}

public:
	CEdit m_edit;
};

BEGIN_MESSAGE_MAP(CMyModleDlg, CDialog)
	ON_BN_CLICKED(IDOK, &CMyModleDlg::OnBnClickedOk)
END_MESSAGE_MAP()


// 自定义应用程序类
class CMyWinApp : public CWinApp
{
public:
	virtual BOOL InitInstance()
	{
		CMyModleDlg dlg;
		m_pMainWnd = &dlg;
		dlg.DoModal();
		return TRUE;
	}
};

// 应用程序对象的全局变量
CMyWinApp theApp;
```

注意到 CEdit 调用了 MoveWindow 函数，事实上由于众多空间都继承自 CWnd，因此它们都可以调用 Win32 中的**窗口函数**。



### 基本控件

#### 静态控件

静态控件即静态文本控件，默认 ID 为 IDC_STATIC，绑定 Static Text 时需要修改 ID 为其它，否则不能添加变量。



#### 按钮控件

除了要更改按钮样式的情况，如果只是单纯需要点击消息，就不需要绑定控件，而是直接添加点击消息处理；相对的，更复杂的操作需要绑定 CButton 类对象，通过 SetWindowText / GetWindowText 等成员函数操作。



#### 列表控件

使用 CListCtrl 类，封装了列表控件的操作。可以通过 ModifyStyle 调整列表风格

* 图标 LVS_ICON
* 小图标 LVS_SMALLICON
* 列表 LVS_LIST
* 报表 LVS_REPORT

使用 InsertItem 向其中加入项

```cpp
int InsertItem(
    int nItem,			// 索引
    LPCTSTR lpszItem	// 字符串
);

int InsertItem(
    int nItem,
    LPCTSTR lpszItem,
    int nImage			// 图标
);
```



#### 列表盒控件

CListBox 与列表控件的不同在于后者功能更加强大，而此控件只能显示一列信息。两者的使用方法类似，绑定控件对象后，常用的成员函数例如

```cpp
int GetCount() const;		// 获得列表中字符串的个数
int GetCurSel() const;		// 获得列表中被选中的字符串的索引
int SetCurSel(int nSelect);	// 指定选中字符串的索引，如果参数为 -1，则不会选中
```

还可以使用类似于 InsertItem 的 InsertString 函数插入字符串。为了能够更新选中的索引，双击控件设置点击消息来更新数据

![image-20230116223335148](image-20230116223335148.png)



#### 单选控件

Radio Button 需要设置 Group 属性为 true 才能绑定。双击 Radio Button 组件编辑对应的点击消息，注意只要进入了该函数，如果什么都不做立即返回，似乎依然会选中该选项。因此一定要在消息处理函数中修改选中状态

```cpp
m_ffd.SetCheck(FALSE);
m_dmffd.SetCheck(TRUE);
```



#### 滑块控件

CSliderCtrl 类产生滑动进度条控件，需要通过 WM_HSCROLL 消息获取滑动信息，消息处理函数内部基本格式为

```cpp
void CMyDlg::OnHScroll(UINT nSBCode, UINT nPos, CScrollBar* pScrollBar)
{
	// 强制转换为 CSliderCtrl 指针，根据 ID 判断是哪个滑块
	CSliderCtrl* sCtrl = (CSliderCtrl*)pScrollBar;
	int id = sCtrl->GetDlgCtrlID();
    
    // 注意这里用 short 接收，因为没有开启“设置合作者整数”，首位为 1 有错误，需要用 short 忽略掉错误
	short pos = sCtrl->GetPos();

	CDialog::OnHScroll(nSBCode, nPos, pScrollBar);
}
```

成员函数查阅官方文档即可。



双击滑块控件可以产生针对该滑块的消息处理函数，如果我们已经绑定了滑块控件变量，使用它更有效。函数包含结构体变量

```cpp
struct NMUPDOWN{
    NMHDR hdr;    // 通知代码的其他信息
    int iPos;     // 当前位置
    int iDelta;   // 位置的增减量
};
```

我们通过它得到位置信息和移动方向，例如

```cpp
void CMyDlg::OnDeltaposSpinx(NMHDR* pNMHDR, LRESULT* pResult)
{
	LPNMUPDOWN pNMUpDown = reinterpret_cast<LPNMUPDOWN>(pNMHDR);

	// 获得滑动方向和当前位置
	int d = pNMUpDown->iDelta;
	int pos = pNMUpDown->iPos;

	*pResult = 0;
}
```



#### 组合框控件

要调整组合框下拉的列表高度，要点击组合框右边的下箭头，这时显示的组合框大小与直接点击组合框并不相同，调节下图中的框选大小就可以调整下拉框高度

![image-20230116212919980](image-20230116212919980.png)

![image-20230116213007998](image-20230116213007998.png)

组合框常用的成员函数和列表盒控件类似，也是通过点击消息来更新选中的索引。
