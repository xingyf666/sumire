# Doxygen

## 基本介绍

### [安装方法](https://sourceforge.net/projects/doxygen/files/)

在下载界面选择最新的 Release 版本的 `.exe` 程序进行安装，将路径添加到环境变量中。



### 生成流程

创建如下项目结构

DoxygenDemo

└─Test

创建 Test 类：头文件

```cpp
/**
 * @file Test.h
 * @author your name (you@domain.com)
 * @brief
 * @version 0.1
 * @date 2023-12-06
 *
 * @copyright Copyright (c) 2023
 *
 */

/**
 * @brief Test 类，测试 Doxygen 文档生成
 *
 */
class Test {
private:
  const char *name;
  int age;

public:
  /**
   * @brief Construct a new Test object
   *
   * @param name
   * @param age
   */
  Test(const char *name, int age);

  /**
   * @brief Destroy the Test object
   * 
   */
  ~Test();
};
```

源文件

```cpp
/**
 * @file Test.cpp
 * @author your name (you@domain.com)
 * @brief 
 * @version 0.1
 * @date 2023-12-06
 * 
 * @copyright Copyright (c) 2023
 * 
 */

#include "Test.h"

/**
 * @brief Construct a new Test:: Test object
 * 
 * @param name 
 * @param age 
 */
Test::Test(const char *name, int age) : name(name), age(age) {}

/**
 * @brief Destroy the Test:: Test object
 * 
 */
Test::~Test() {}

```

程序入口

```cpp
/**
 * @file main.cpp
 * @author your name (you@domain.com)
 * @brief 
 * @version 0.1
 * @date 2023-12-06
 * 
 * @copyright Copyright (c) 2023
 * 
 */
#include "Test/Test.h"

/**
 * @brief 程序入口
 * 
 * @param argc 
 * @param argv 
 * @return int 
 */
int main(int argc, char const *argv[]) {
  Test test("Li", 10);
  return 0;
}
```



打开 Doxygenwizard 进入 GUI 界面。配置项目名、项目简介、项目版本，项目 logo、源码路径，选择递归搜索源码，最后选择目标路径。

![](image-20231206165637913.png)

可以在 Expert 界面中修改输出语言

![](image-20231206165748017.png)

在 Run 界面点击 Run Doxygen 即可生成文档

![](image-20231206165816562.png)

在生成路径下找到 html 目录下的 index.html 打开，可以看到生成结果：

![](image-20231206165926726.png)



### Expert 设置

* 勾选 Build 中的 `EXTRACT_ALL` 选项，这样会将所有文件包含进来；关闭 `SHOW_USED_FILES` 隐藏所属文件信息。勾选 `EXTRACT_PRIVATE` 和 `EXTRACT_STATIC` 选项来显示私有和静态信息。如果需要显示更多信息，勾选对应的选项即可。

![](image-20231206192518143.png)

![](image-20231206191511533.png)

* 勾选 HTML 中的 `DISABLE_INDEX` 和 `GENERATE_TREEVIEW` 禁用索引导航，开启树状图导航。

![](image-20231206191351765.png)



### 导出设置

将当前设置保存为设置文件，如果之后还需要使用该设置，重新打开这个文件即可。

![](image-20231206191831767.png)



### 注释命令

在注释中添加 `@` 命令，可以创建不同的样式。例如

```cpp
/**
 * @file Test.h
 * @author your name (you@domain.com)
 * @brief
 * @version 0.1
 * @date 2023-12-06
 *
 * @copyright Copyright (c) 2023
 *
 */
```

其中每个 `@` 后都是一个命令。一些常用的命令包括

* `brief` 简述
* `details` 详述
* `see` 后跟包含另一个类、方法、变量的描述
* `todo` 待办事项
* `param` 说明参数
* `return` 说明返回值
* `note` 提示
* `attention` 注意事项
* `warning` 警告



### 注释位置

建议将不会更改的注释内容放在头文件中，而将更具体的注意事项等放在源文件中。例如

```cpp
/**
 * @brief 将两数相加的函数
 * 
 * @param a 被加数
 * @param b 加数
 * @return int 两数之和
 */
int add(int a, int b);

/**
 * @details 输入 a, b 两个整数，返回 a + b
 * @note 输入的类型是两个整型
 * @warning 最好不要输入浮点型
 */
int add(int a, int b)
{
    return a + b;
}
```



## 注释结构

### 特殊注释块

特殊注释块是带有一些附加标记的 C 或 C++ 样式注释块，因此 doxygen 知道它是一段结构化文本，需要最终出现在生成的文档中。



对于代码中的每个实体，有两种（或在某些情况下是三种）类型的描述，它们共同构成了该实体的文档：

* 简要说明 brief
* 详细说明 detailed

两者都是可选的。允许有多个简短或详细的描述（但不建议这样做，因为没有指定描述的显示顺序）。对于方法和函数，还有第三种类型的描述，即

* 主体描述 body

它由在方法或函数的主体中找到的所有注释块的串联组成。



#### detail

有几种方法将注释块标记为 detail 说明：

1. 使用 Javadoc 样式

    ```cpp
    /**
     * ... text ...
     */
    ```

2. 使用 Qt 样式

    ```cpp
    /*!
     * ... text ...
     */
    ```

3. 使用包含至少两个注释行的块，每行以额外的斜杠或感叹号开头

    ```cpp
    ///
    /// ... text ...
    ///
    
    //!
    //!... text ...
    //!
    ```

4. 使用更明显的注释

    ```cpp
    /********************************************//**
     *  ... text
     ***********************************************/
    ```

    注意在使用 clang-format 时注释可能会产生错位。如果开启 `JAVADOC_BANNER` 选项，可以使用

    ```cpp
    /////////////////////////////////////////////////
    /// ... text ...
    /////////////////////////////////////////////////
    
    /************************************************
     *  ... text
     ***********************************************/
    ```



下面给出一个典型的 Javadoc 格式

```cpp
/**
 * A brief history of JavaDoc-style (C-style) comments.
 *
 * This is the typical JavaDoc-style C-style comment. It starts with two
 * asterisks.
 *
 * @param theory Even if there is only one possible unified theory. it is just a
 *               set of rules and equations.
 */
void cstyle( int theory );
 
/*******************************************************************************
 * A brief history of JavaDoc-style (C-style) banner comments.
 *
 * This is the typical JavaDoc-style C-style "banner" comment. It starts with
 * a forward slash followed by some number, n, of asterisks, where n > 2. It's
 * written this way to be more "visible" to developers who are reading the
 * source code.
 *
 * Often, developers are unaware that this is not (by default) a valid Doxygen
 * comment block!
 *
 * However, as long as JAVADOC_BLOCK = YES is added to the Doxyfile, it will
 * work as expected.
 *
 * This style of commenting behaves well with clang-format.
 *
 * @param theory Even if there is only one possible unified theory. it is just a
 *               set of rules and equations.
 ******************************************************************************/
void javadocBanner( int theory );
 
/***************************************************************************//**
 * A brief history of Doxygen-style banner comments.
 *
 * This is a Doxygen-style C-style "banner" comment. It starts with a "normal"
 * comment and is then converted to a "special" comment block near the end of
 * the first line. It is written this way to be more "visible" to developers
 * who are reading the source code.
 * This style of commenting behaves poorly with clang-format.
 *
 * @param theory Even if there is only one possible unified theory. it is just a
 *               set of rules and equations.
 ******************************************************************************/
void doxygenBanner( int theory );
```



#### brief

1. 使用 `\brief` 开头表示简要说明，空行后进行详细说明

    ```cpp
    /*! \brief Brief description.
     *         Brief description continued.
     *
     *  Detailed description starts here.
     */
    ```

2. 如果开启 `JAVADOC_AUTOBRIEF` 选项，则自动将以第一个 `.` 加空格或换行结束

    ```cpp
    /** Brief description which ends at this dot. Details follow
     *  here.
     */
    
    /// Brief description which ends at this dot. Details follow
    /// here.
    ```

3. 使用特殊的 C++ 样式，不超过一行

    ```cpp
    /// Brief description.
    /** Detailed description. */
    ```

    或者

    ```cpp
    //! Brief description.
    
    //! Detailed description
    //! starts here.
    ```

如果有多个 detail 描述，则它们会被依次加入

```cpp
//! Brief description, which is
//! really a detailed description since it spans multiple lines.
/*! Another detailed description!
 */
```

上面就是两个 detail 描述，因为它跨了多行。



#### 成员后注释

在注释块中加入 `<` 标记来进行成员后的注释。例如

```cpp
int var; /*!< Detailed description after the member */
int var; /**< Detailed description after the member */
int var; //!< Detailed description after the member
         //!<
int var; ///< Detailed description after the member
         ///<
```

如果要添加 brief 描述，使用

```cpp
int var; //!< Brief description after the member
int var; ///< Brief description after the member
```



对于函数，可以使用 `@param` 来记录参数，用 `[in],[out],[in,out]` 来记录方向。对于内联文档，还可以用方向属性开头

```cpp
void foo(int v /**< [in] docs for input parameter v. */);
```

下面是一个例子

```cpp
/*! A test class */
 
class Afterdoc_Test
{
  public:
    /** An enum type. 
     *  The documentation block cannot be put after the enum! 
     */
    enum EnumType
    {
      int EVal1,     /**< enum value 1 */
      int EVal2      /**< enum value 2 */
    };
    void member();   //!< a member function.
    
  protected:
    int value;       /*!< an integer value */
};
```



#### 例子

下面是一个 Qt 风格的注释

```cpp
//!  A test class. 
/*!
  A more elaborate class description.
*/
 
class QTstyle_Test
{
  public:
 
    //! An enum.
    /*! More detailed enum description. */
    enum TEnum { 
                 TVal1, /*!< Enum value TVal1. */  
                 TVal2, /*!< Enum value TVal2. */  
                 TVal3  /*!< Enum value TVal3. */  
               } 
         //! Enum pointer.
         /*! Details. */
         *enumPtr, 
         //! Enum variable.
         /*! Details. */
         enumVar;  
    
    //! A constructor.
    /*!
      A more elaborate description of the constructor.
    */
    QTstyle_Test();
 
    //! A destructor.
    /*!
      A more elaborate description of the destructor.
    */
   ~QTstyle_Test();
    
    //! A normal member taking two arguments and returning an integer value.
    /*!
      \param a an integer argument.
      \param s a constant character pointer.
      \return The test results
      \sa QTstyle_Test(), ~QTstyle_Test(), testMeToo() and publicVar()
    */
    int testMe(int a,const char *s);
       
    //! A pure virtual member.
    /*!
      \sa testMe()
      \param c1 the first argument.
      \param c2 the second argument.
    */
    virtual void testMeToo(char c1,char c2) = 0;
   
    //! A public variable.
    /*!
      Details.
    */
    int publicVar;
       
    //! A function variable.
    /*!
      Details.
    */
    int (*handler)(int a,int b);
};
```

然后是一个 Javadoc 风格的例子

```cpp
/**
 *  A test class. A more elaborate class description.
 */
 
class Javadoc_Test
{
  public:
 
    /** 
     * An enum.
     * More detailed enum description.
     */
 
    enum TEnum { 
          TVal1, /**< enum value TVal1. */  
          TVal2, /**< enum value TVal2. */  
          TVal3  /**< enum value TVal3. */  
         } 
       *enumPtr, /**< enum pointer. Details. */
       enumVar;  /**< enum variable. Details. */
       
      /**
       * A constructor.
       * A more elaborate description of the constructor.
       */
      Javadoc_Test();
 
      /**
       * A destructor.
       * A more elaborate description of the destructor.
       */
     ~Javadoc_Test();
    
      /**
       * a normal member taking two arguments and returning an integer value.
       * @param a an integer argument.
       * @param s a constant character pointer.
       * @see Javadoc_Test()
       * @see ~Javadoc_Test()
       * @see testMeToo()
       * @see publicVar()
       * @return The test results
       */
       int testMe(int a,const char *s);
       
      /**
       * A pure virtual member.
       * @see testMe()
       * @param c1 the first argument.
       * @param c2 the second argument.
       */
       virtual void testMeToo(char c1,char c2) = 0;
   
      /** 
       * a public variable.
       * Details.
       */
       int publicVar;
       
      /**
       * a function variable.
       * Details.
       */
       int (*handler)(int a,int b);
};
```



#### 结构命令

Doxygen 允许你把你的文档块放在几乎任何地方（例外是在函数的主体内或普通的 C 风格的注释块中）。不将文档块直接放在项目之前（或之后）所付出的代价是需要将结构命令放在文档块中，这会导致一些信息重复。因此，在实践中，您应该避免使用结构命令，除非其他要求迫使您这样做。



结构命令（与所有其他命令一样）以反斜杠 （ `\` ） 或 at 符号 （ `@` ） 开头（如果您更喜欢 Javadoc 样式），后跟命令名称和一个或多个参数。例如

```cpp
/*! \class Test
    \brief A test class.

    A more detailed class description.
*/
```

其它结构命令包括：

- `\struct` 记录 C 结构。
- `\union` 记录联合。
- `\enum` 记录枚举类型。
- `\fn` 记录函数。
- `\var` 记录变量、typedef 或枚举值。
- `\def` 记录 `#define` 。
- `\typedef` 记录类型定义。
- `\file` 记录文件。
- `\namespace` 记录命名空间。
- `\package` 记录 Java 包。
- `\interface` 记录 IDL 接口。



#### 全局对象

要记录全局对象，例如函数、`typedef`、枚举、宏等，需要记录它们所在的文件，即

```cpp
/*! \file */ 
```

或者

```cpp
/** @file */ 
```

需要放在文件中。例如下面这个定义全局对象的头文件

```cpp
/*! \file structcmd.h
    \brief A Documented file.
    
    Details.
*/
 
/*! \def MAX(a,b)
    \brief A macro that returns the maximum of \a a and \a b.
   
    Details.
*/
 
/*! \var typedef unsigned int UINT32
    \brief A type definition for a .
    
    Details.
*/
 
/*! \var int errno
    \brief Contains the last error code.
 
    \warning Not thread safe!
*/
 
/*! \fn int open(const char *pathname,int flags)
    \brief Opens a file descriptor.
 
    \param pathname The name of the descriptor.
    \param flags Opening flags.
*/
 
/*! \fn int close(int fd)
    \brief Closes the file descriptor \a fd.
    \param fd The descriptor to close.
*/
 
/*! \fn size_t write(int fd,const char *buf, size_t count)
    \brief Writes \a count bytes from \a buf to the filedescriptor \a fd.
    \param fd The descriptor to write to.
    \param buf The data buffer to write.
    \param count The number of bytes to write.
*/
 
/*! \fn int read(int fd,char *buf,size_t count)
    \brief Read bytes from a file descriptor.
    \param fd The descriptor to read from.
    \param buf The buffer to read into.
    \param count The number of bytes to read.
*/
 
#define MAX(a,b) (((a)>(b))?(a):(b))
typedef unsigned int UINT32;
int errno;
int open(const char *,int);
int close(int);
size_t write(int,const char *, size_t);
int read(int,char *,size_t);
```



### Markdown 支持

#### 段落

使用空行来分段

```cpp
Here is text for one paragraph.

We continue with more text in another paragraph.
```



#### 标头

使用 `#` 决定标头的级别

```cpp
# This is a level 1 header

### This is level 3 header #######
```



#### 引用块

使用 `>` 加上一个空格表示引用

```cpp
> This is a block quote
> spanning multiple lines
```



#### 列表

使用 `-,+,*` 生成列表

```cpp
- Item 1

  More text for this item.

- Item 2
  + nested list item.
  + another nested item.
- Item 3
```

或者编号

```cpp
1. First item.
2. Second item.
```



使用对齐的缩进来生成列表。例如

```cpp
/*! 
*  A list of events:
*    - mouse events
*         -# mouse move event
*         -# mouse click event\n
*            More info about the click event.
*         -# mouse double click event
*    - keyboard events
*         1. key down event
*         2. key up event
*
*  More text here.
*/
```

如果使用制表符作为列表缩进，需要将 `TAB_SIZE` 设置为正确的制表符大小。可以通过开启新段落或者在空行处放置一个 `.` 表示列表结束。例如

```cpp
/**
 * Text before the list
 * - list item 1
 *   - sub item 1
 *     - sub sub item 1
 *     - sub sub item 2
 *     . 
 *     The dot above ends the sub sub item list.
 *
 *     More text for the first sub item
 *   .
 *   The dot above ends the first sub item.
 *
 *   More text for the first list item
 *   - sub item 2
 *   - sub item 3
 * - list item 2
 * .
 * More text in the same paragraph.
 *
 * More text in a new paragraph.
 */
```



#### 代码块

通过缩进至少 4 个额外空格来创建预先格式化的代码块

```cpp
This a normal paragraph

    This is a code block

We continue with a normal paragraph again.
```



#### 水平线

用 `-,_` 生成水平线

```cpp
- - -
______
```



#### 强调

使用 `*,_` 来加粗显示

```cpp
 *single asterisks*

 _single underscores_

 **double asterisks**

 __double underscores__
```



#### 删除

使用 `~` 来表示删除文本

```cpp
 ~~double tilde~~
```



#### 代码范围

用反引号表示代码的范围

```cpp
Use the `printf()` function.
```

如果要在代码范围显式反引号或单引号，使用双反引号

```cpp
To assign the output of command `ls` to `var` use ``var=`ls```.

To assign the text 'text' to `var` use ``var='text'``.
```



#### 链接

Doxygen 支持 Markdown 定义的两种样式的 make 链接：内联和引用。



##### 内联链接

使用 `[]()` 格式

```cpp
[The link text](http://example.net/)
[The link text](http://example.net/ "Link title")
[The link text](/relative/path/to/index.html "Link title")
[The link text](somefile.html)
```

还可以链接已记录的实体

```cpp
[The link text](@ref MyClass)
```

如果引用的第一个非空格字符是 `#`，例如

```cpp
[The link text](#MyClass)
```

将会替换为 `@ref` 命令

```cpp
@ref MyClass "The link text"
```



##### 引用链接

链接定义为

```cpp
[link name]: http://www.example.com "Optional title"
```

这里可以使用单引号或者括号。定义链接后，再进行引用

```cpp
[link text][link name]
```

如果链接文本和名称相同，可以使用

```cpp
[link name][]
```

甚至是

```cpp
[link name]
```

例如

```cpp
I get 10 times more traffic from [Google] than from
[Yahoo] or [MSN].

[google]: http://google.com/        "Google"
[yahoo]:  http://search.yahoo.com/  "Yahoo Search"
[msn]:    http://search.msn.com/    "MSN Search"
```

也可以在链接中定义 `@ref`，例如

```cpp
[myclass]: @ref MyClass "My class"
```



##### 自动链接

若要创建指向 URL 或电子邮件的链接，使用

```cpp
<http://www.example.com>
<https://www.example.com>
<ftp://www.example.com>
<mailto:address@example.com>
<address@example.com>
```



##### 标头 ID

可以对标题进行标记

```cpp
Header 1                {#labelid}
========

## Header 2 ##          {#labelid2}
```

然后链接跳转

```cpp
[Link text](#labelid)
[Link text](@ref labelid)
```

这只适用于 1 至 4 级的标头。



#### 图像

图像语法与 Markdown 一致

```cpp
![Caption text](/path/to/img.jpg)
![Caption text](/path/to/img.jpg "Image title")
![Caption text][img def]
![img def]

[img def]: /path/to/img.jpg "Optional Title"
```

还可以使用 `@ref` 链接图像

```cpp
![Caption text](@ref image.png)
![img def]

[img def]: @ref image.png "Caption text"
```



通过属性控制图像显示。例如

```cpp
![Doxygen Logo](https://www.doxygen.org/images/doxygen.png){html: width=50%, latex: width=5cm}
```



#### 目录

Doxygen 支持特殊的链接标记 `[TOC]`，将其放置在页面中在页面开头生成目录。使用 `[TOC]` 与使用 `\tableofcontents` 指令相同。另外，`TOC_INCLUDE_HEADINGS` 选项必须设为大于 0，否则不会显示任何目录。



#### 表格

可以使用 Markdown 中的简单表格

```cpp
/// First Header  | Second Header
/// ------------- | -------------
/// Content Cell  | Content Cell
/// Content Cell  | Content Cell

/// | Right | Center | Left  |
/// | ----: | :----: | :---- |
/// | 10    | 10     | 10    |
/// | ^     | 1000   | 1000  |

/// | Right | Center | Left  |
/// | ----: | :----: | :---- |
/// | 10    | 10     | 10    |
/// | 1000  |||
```

其中下面的表格使用 `:` 控制对齐方式，`^` 表示上面的单元格应该跨行，`|` 表示左边的表格要跨列。



#### 嵌入页面

可以将 Markdown 直接嵌入到页面中。被标记为 `index` 或 `mainpage` 的 md 文件会被作为首页。例如给出 README.md 文件

```cpp
My Main Page                         {#mainpage}
============

Documentation that will appear on the main page
```

如果页面有 label 标记，则可以通过 `@ref` 链接。否则可以使用文件名。例如

```cpp
See [the other page](other.md) for more info.
```



### 自动链接

#### 网页链接

Doxygen 将自动通过链接（HTML 格式）替换文档中的任何 URL 和邮件地址。若要手动指定链接文本，请使用 HTML ' `a` ' 标记：

```cpp
<a href="linkURL">link text</a> 
```

这将被 doxygen 自动转换为其他输出格式。



#### 类链接

文档中的任何单词，如果与某个类名相同，并且至少有一个非小写字母，那么它会被自动替换为指向该页面的链接。如果不想被替换，应该在前面放一个 `%`；如果要链接全小写字母，使用 `\ref` 命令。



#### 文件链接

所有包含 `.` 并且不以 `.` 结尾的单词都认为是文件名。如果它与某个输入文件名相同，则会自动生成指向该文件的链接。



#### 函数链接

如果遇到以下模式之一，就会创建指向函数的链接：

1. `<functionName>"("<argument-list>")"`
2. `<functionName>"()"`
3. `"::"<functionName>`
4. `(<className>"::")n<functionName>"("<argument-list>")"`
5. `(<className>"::")n<functionName>"("<argument-list>")"<modifiers>`
6. `(<className>"::")n<functionName>"()"`
7. `(<className>"::")n<functionName>`

其中要求 `n>0` 。注意

* 使用正确的类型指定

    ```cpp
    func(int) const;
    fun(int);
    ```

    是两个不同的函数；

* 上面模式中的 `::` 可以使用 `#` 替换；

* 如果出现同名的全局变量和成员变量。例如

    ```cpp
    int foo;
    
    class A
    {
    public:
        int foo;
    };
    ```

    则 `::foo` 链接全局变量，`#foo` 链接成员变量。

* 非重载成员可以省略参数列表。

* 如果函数被重载并且没有指定匹配的参数列表，则将创建一个指向其中一个重载成员的文档的链接。



#### 成员链接

```cpp
/*! \file autolink.cpp
  Testing automatic link generation.
  
  A link to a member of the Autolink_Test class: Autolink_Test::member, 
  
  More specific links to the each of the overloaded members:
  Autolink_Test::member(int) and Autolink_Test#member(int,int)
 
  A link to a protected member variable of Autolink_Test: Autolink_Test#var, 
 
  A link to the global enumeration type #GlobEnum.
 
  A link to the define #ABS(x).
  
  A link to the destructor of the Autolink_Test class: Autolink_Test::~Autolink_Test, 
  
  A link to the typedef ::B.
 
  A link to the enumeration type Autolink_Test::EType
  
  A link to some enumeration values Autolink_Test::Val1 and ::GVal2
*/
 
/*!
  Since this documentation block belongs to the class Autolink_Test no link to 
  Autolink_Test is generated.
 
  Two ways to link to a constructor are: #Autolink_Test and Autolink_Test().
 
  Links to the destructor are: #~Autolink_Test and ~Autolink_Test().
  
  A link to a member in this class: member().
 
  More specific links to the each of the overloaded members: 
  member(int) and member(int,int). 
  
  A link to the variable #var.
 
  A link to the global typedef ::B.
 
  A link to the global enumeration type #GlobEnum.
  
  A link to the define ABS(x).
  
  A link to a variable \link #var using another text\endlink as a link.
  
  A link to the enumeration type #EType.
 
  A link to some enumeration values: \link Autolink_Test::Val1 Val1 \endlink and ::GVal1.
 
  And last but not least a link to a file: autolink.cpp.
  
  \sa Inside a see also section any word is checked, so EType, 
      Val1, GVal1, ~Autolink_Test and member will be replaced by links in HTML.
*/
 
class Autolink_Test
{
  public:
    Autolink_Test();               //!< constructor 
   ~Autolink_Test();               //!< destructor 
    void member(int);     /**< A member function. Details. */
    void member(int,int); /**< An overloaded member function. Details */
 
    /** An enum type. More details */
    enum EType { 
      Val1,               /**< enum value 1 */ 
      Val2                /**< enum value 2 */ 
    };                
 
  protected:
    int var;              /**< A member variable */
};
 
/*! details. */
Autolink_Test::Autolink_Test() { }
 
/*! details. */
Autolink_Test::~Autolink_Test() { }
 
/*! A global variable. */
int globVar;
 
/*! A global enum. */
enum GlobEnum { 
                GVal1,    /*!< global enum value 1 */ 
                GVal2     /*!< global enum value 2 */ 
              };
 
/*!
 *  A macro definition.
 */ 
#define ABS(x) (((x)>0)?(x):-(x))
 
typedef Autolink_Test B;
 
/*! \fn typedef Autolink_Test B
 *  A type definition. 
 */
```



#### typedefs

使用别名时创建一个说明原型的注释，这样可以通过别名跳转到 `typedef`，然后再跳转到原型。

```cpp
/*! \file restypedef.cpp
 * An example of resolving typedefs.
 */
 
/*! \struct CoordStruct
 * A coordinate pair.
 */
struct CoordStruct
{
  /*! The x coordinate */
  float x;
  /*! The y coordinate */
  float y;
};
 
/*! Creates a type name for CoordStruct */ 
typedef CoordStruct Coord;
 
/*! 
 * This function returns the addition of \a c1 and \a c2, i.e:
 * (c1.x+c2.x,c1.y+c2.y)
 */
Coord add(Coord c1,Coord c2)
{
}
```



### 公式

在 HTML 设置中启用 `USE_MATHJAX` 选项，这样可以不必在本地配置，由客户端 javascript 进行转换。

![](image-20231206224237166.png)



有四种方法可以包含公式：

1. 使用显示在运行文本中的文本内公式。这些公式应该放在一对 `\f$` 命令之间，例如

    ```cpp
    The distance between \f$(x_1,y_1)\f$ and \f$(x_2,y_2)\f$ is 
      \f$\sqrt{(x_2-x_1)^2+(y_2-y_1)^2}\f$.
    ```

2. 使用出现在运行文本中的文本公式。这样公式应该放在 `\f(,\f)` 之间，例如

    ```cpp
    The LaTeX and Tex logos are: \f(\LaTeX \f) and \f(\TeX \f).
    ```

3. 未编号的显示公式，这些公式独占一行。放在 `\f[,\f]` 之间，例如

    ```cpp
    \f[
        |I_2|=\left| \int_{0}^T \psi(t) 
                 \left\{ 
                    u(a,t)-
                    \int_{\gamma(t)}^a 
                    \frac{d\theta}{k(\theta,t)}
                    \int_{a}^\theta c(\xi)u_t(\xi,t)\,d\xi
                 \right\} dt
              \right|
      \f]
    ```

4. 可以使用 `\f{environment}` 指定不在数学环境中的公式或其它 latex 元素，其中 `environment` 是环境名称，以 `\f}` 结束。例如

    ```cpp
    \f{eqnarray*}{
            g &=& \frac{Gm_2}{r^2} \\ 
              &=& \frac{(6.673 \times 10^{-11}\,\mbox{m}^3\,\mbox{kg}^{-1}\,
                  \mbox{s}^{-2})(5.9736 \times 10^{24}\,\mbox{kg})}{(6371.01\,\mbox{km})^2} \\ 
              &=& 9.82066032\,\mbox{m/s}^2
       \f}
    ```

应确保公式包含 LATEX 数学模式下的有效命令。



要自定义命令，启用配置项 `FORMULA_MACROFILE`，可以使用

```cpp
\newcommand{\E}{\mathrm{E}}
\newcommand{\ccSum}[3]{\sum_{#1}^{#2}{#3}}
```



### [特殊指令](https://www.doxygen.nl/manual/commands.html#cmdimage)

特殊指令直接查询文档，遇到有需要使用的命令时才查看。



## 生成选项

### 图形

Doxygen 内置支持为 C++ 类生成继承图。在 [Graphviz](https://www.graphviz.org/) 官网下载最新版本安装，在 Dot 选项中开启 `HAVE_DOT`，并配置路径

![](image-20231206225827385.png)

然后 Doxygen 就可以根据类的继承关系，自动生成继承图。注意 `EXTRACT_ALL` 选项需要开启。



### 搜索

如果您在 Windows 上运行 doxygen，则可以使用 doxygen 生成的 HTML 文件制作一个编译的 HTML 帮助文件 （.chm）。这是一个包含所有 HTML 文件的单个文件，它还包括一个搜索索引。



只需要开启 `GENERATE_HTMLHELP` 选项，指定 hhc.exe 路径即可在 html 文件夹下生成 .chm 帮助文件。

![](image-20231206231614598.png)

需要下载 [HTML Help Workshop](https://www.helpandmanual.com/downloads_mscomp.html) 找到 hhc.exe 文件。注意开启此选项以后，将不会生成侧边栏目录。



### 自定义输出

#### 导航

默认情况下，doxygen 在每个 HTML 页面的顶部显示导航选项卡，与下面设置对应

* `DISABLE_INDEX = NO`
* `GENERATE_TREEVIEW = NO`

![](layout_index_and_notreeview.png)

可以修改为交互式导航树

* `DISABLE_INDEX = YES`
* `GENERATE_TREEVIEW = YES`
* `FULL_SIDEBAR = NO`

![](layout_noindex_and_treeview.png)

可以让内容跨越屏幕的标题位置

* `DISABLE_INDEX = YES`
* `GENERATE_TREEVIEW = YES`
* `FULL_SIDEBAR = YES`

![](layout_noindex_and_sidebar.png)

甚至同时拥有两种形式的导航：

* `DISABLE_INDEX = NO`
* `GENERATE_TREEVIEW = YES`

![](layout_index_and_treeview.png)

