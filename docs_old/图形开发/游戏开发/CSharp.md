# CSharp

## 基本介绍

### Hello World

在 VS 中安装 `.Net` 框架开发组件，然后创建“控制台应用(`.NET Framework`)”即可得到一个基本的框架程序

![[CSharp.assets/image-20240221201354504.png]]

在其中添加 Hello World 输出

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ConsoleApp
{
    internal class Program
    {
        static void Main(string[] args)
        {
            // 控制台输出
            Console.WriteLine("Hello World.");
        }
    }
}
```



### 代码结构

最简单的 CSharp 代码格式为

```csharp
// 类型
class Program
{
    // 入口函数
    static void Main()
    {
    }
}
```

其中类型必须定义入口函数 Main()，程序会首先调用这个方法。



### 代码折叠

使用 `#region` 来创建可折叠代码段

```csharp
#region Block
    
#endregion
```



### 命名空间

使用 `namespace` 声明命名空间，通过 `.` 连接嵌套命名空间

```cpp
namespace SpaceA.SpaceB
{
    public class A
    {
        public int x;

        public A(int x)
        {
            this.x = x;
        }
    }
}
```



## 变量

### 声明变量

使用 var 声明变量

```csharp
var n = 100;
var m = 200;
var sum = n + m;
```

这里 var 类似于 C++ 中的 auto 声明，程序会自动推导变量类型。



最好指定具体的变量类型

```csharp
int a = 10;
char c = 'a';
string str = "hello";
int[] arr = new int[10];
```



常量通过 `const` 前缀修饰

```cpp
const int a = 10;
const char c = 'a';
const string str = "hello";
```



#### 类型

常用的内置变量类型

- 整型 `byte, sbyte, ushort, short, uint, int, ulong, long`
- 浮点型 `float, double, decimal` 通过 `f, d, m` 结尾区分，默认为 `double` 型（`decimal` 具有 28 位精度，用于货币计算）
- 布尔型 `bool`

CSharp 区分值类型和引用类型

- 简单的标准类型如 int, double, float 都是值类型，都分配在栈上；
- 而 string 等自定义对象属于引用类型，分配在堆上，引用类型保存在堆上的地址，类似于指针。

这意味几乎总是应该将自定义对象看作指针。例如

```cpp
Person p1 = new Person();
Person p2 = p1;
```

此时对于 `p1` 的修改都会同时改变 `p2`，因为它们指向同一个地址。



使用 `typeof` 获得对象类型

```cpp
Type type = typeof(string);
Console.Write(type.Name);
```

可以通过 `Type` 获得类型信息。



使用 `default` 获得对象类型的默认值

```csharp
int defaultInt = default(int); // 等同于 0 
bool defaultBool = default(bool); // 等同于 false 
double defaultDouble = default; // C# 7.1+ 语法，等同于 0.0
```



#### 枚举

枚举声明形式

```csharp
public enum Color
{
    red,
    green,
    blue
}
```

创建枚举时使用 `.` 获得属性

```csharp
Color color = Color.red;
```

使用枚举方法 Parse 将字符串转换为指定类型的枚举变量，反之通过 ToString 直接转换

```csharp
Color color = (Color)Enum.Parse(typeof(Color), "red");
string c = color.ToString();
```

枚举和整型之间可以直接强制转换。



#### 结构

结构声明形式

```csharp
public struct Person
{
    public string name;
    public int age;
    public int score;
}
```

结构类型属于值类型，与 `class` 完全不同，后者继承 `Object`，总是可以赋值为 `null`，但结构类型不能为 `null` 。



#### 空对象

一个空的对象是 null，类似于 C++ 中的空指针

```csharp
string str = null;
```



### 字符串

字符串是引用类型，它指向堆上分配的内存。修改字符串的值实际上只是修改了它指向的地址，原先分配的内存依然存在。不过 CSharp 的自动回收机制会销毁没有使用的内存空间。

> 频繁地修改字符串的值会造成大量的内存浪费。



#### 读取字符

使用 `[]` 访问字符串中的字符，但是这是只读操作，不能修改

```csharp
string s = "hello";
char c = s[2];
```



#### 与数组转换

可以与字符数组相互转换

```csharp
string str = new string(char[] chs);	// 数组转字符串
char[] chs = str.ToCharArray();			// 字符串转数组
```



#### 字符串构建

如前面提到的，频繁修改字符串会浪费资源。对于大规模修改的情况应当使用字符串构建器

```csharp
StringBuilder sb = new StringBuilder();

// 计时器
Stopwatch sw = new Stopwatch();
sw.Start();

// 大规模添加字符
for (int i = 0; i < 1000000; i++)
{
    sb.Append(i);
}

// 输出运行时间
sw.Stop();
Console.WriteLine(sw.Elapsed);

// 最后转换为 string
string str = sb.ToString();
```

字符构建器在原地修改字符串，因此避免了资源浪费。



### 类型转换

#### 对象方法

不同类型支持一些方法来进行转换，例如

```csharp
string str = "123";            
int num = int.Parse(str);
string str2 = num.ToString();
```



#### 强制转换

使用与 C++ 相同的方式强制转换

```csharp
int a = (int)5.6;
```

该方式性能较高，但如果转换失败，会抛出 `InvalidCastException` 异常。



#### 转换工厂

使用 Convert 转换工厂进行转换

```csharp
string str = "123";
int a = Convert.ToInt32(str);
```

该方式性能较低，因为需要额外的类型检查。如果转换不合法，会抛出 `FormatException` 或 `OverflowException` 等异常，有利于基本类型转换时的精度范围判断。



## 基本数据结构

### 集合结构

| 类型        | 作用     | 类型        | 作用              |
| --------- | ------ | --------- | --------------- |
| ArrayList | 动态数组类型 | HashTable | 提供 object 之间的映射 |

这些集合结构都派生自 Collection 类。



#### ArrayList

动态数组类型，能够容纳任意类型的值

```cpp
ArrayList arr = new ArrayList();
arr.Add(1.2);
arr.Add("12");
```

由于可能出现装箱拆箱，因此不建议使用。



#### List

泛型动态数组类型，可以指定统一的元素类型

```cpp
List<int> list = new List<int>();
list.Add(12);
list.Add(10);
list.RemoveAll(id => id == 12);	// 其中 id => id == 12 是 lambda 表达式
```

当需要储存数据时，首选此类型。



#### Dictionary

泛型字典类型

```cpp
Dictionary<int, string> map = new Dictionary<int, string>();
map[10] = "hello";
map[13] = "world";
map.Add(12, ",");
```



#### 指针

使用 unsafe 关键字编写不安全的代码，从而可以直接操作内存和指针。由于 GC 可能会移动对象来压缩内存，因此操作指针时需要 fixed 关键字确保内存地址不移动

```csharp
unsafe
{
    int[] numbers = { 1, 2, 3, 4, 5 };
    fixed (int* ptr = numbers) // 使用 fixed 固定数组内存地址
    {
        for (int i = 0; i < 5; i++)
        {
            *(ptr + i) *= 2; // 通过指针修改值
        }
    }
}
```



### 文件结构

#### 路径操作

路径操作通过 Path 类实现

```csharp
string str = "E:/Desktop/test.txt";
Path.GetFileName(str);
```

这里 Path 是一个静态类，只用来封装方法，不创建实例。



#### 文件操作

文件操作主要在 `System.IO` 模块中的 File 类，它也是静态类

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

internal class Program
{
    static void Main(string[] args)
    {
        string fpath = "E:/Desktop";
        DirectoryInfo root = new DirectoryInfo(fpath);

        // 获取目录下的文件数组，转换为列表
        FileInfo[] files = root.GetFiles();
        List<FileInfo> listFiles = files.ToList();

        // 循环读取
        for (int i = 0; i < listFiles.Count; i++)
        {
            Console.WriteLine(listFiles[i].Name);

            // 删除指定文件
            if (listFiles[i].Name == "test.txt")
            {
                File.Delete(listFiles[i].FullName);
                Console.WriteLine("test.txt 已经被删除。");
            }

            // 查找指定文件，并重命名
            if (listFiles[i].Name.Contains("hello"))
            {
                // 修改 hello.txt 为 world.txt
                string srcName = listFiles[i].FullName;
                string dstName = listFiles[i].Directory.FullName + "\\world" + listFiles[i].Extension;

                // 移动操作重命名
                File.Move(srcName, dstName);
            }
        }

        Console.ReadKey();
    }
}
```



#### 文件读写

使用 FileStream 对文件内容进行读写操作，由于它处理字节，因此可以用于所有类型的文件读写

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ConsoleApp
{
    internal class Program
    {
        static void Main(string[] args)
        {
            // 创建文件流，读取字节
            FileStream fsRead = new FileStream("E:/Desktop/test.txt", FileMode.Open);
            byte[] buffer = new byte[255];

            // 返回读取到的有效字节数
            int r = fsRead.Read(buffer, 0, buffer.Length);

            // 将读取内容解码为字符串
            string s = Encoding.Default.GetString(buffer);

            // 关闭流
            fsRead.Close();

            // 释放资源（GC 不会主动释放，需要手动操作）
            fsRead.Dispose();
            
            // 输出内容
            Console.WriteLine(s);
            
            // 使用 using 自动释放 FileStream 资源
            using (FileStream fwWrite = new FileStream("E:/Desktop/test.txt", FileMode.Open))
            {
                string str = "Hello World";
                
                // 写入字节
                byte[] buffer = Encoding.Default.GetBytes(str);
                fwWrite.Write(buffer, 0, buffer.Length);
            }
            
            Console.ReadKey();
        }
    }
}
```



使用 StreamReader 读写文本文件

```csharp
using (StreamReader sr = new StreamReader("E:/Desktop/test.txt", Encoding.Default))
{
    string str = sr.ReadToEnd();
    Console.WriteLine(str);
}

using (StreamWriter sw = new StreamWriter("E:/Desktop/test.txt"))
{
    string str = "Hi";
    sw.Write(str);
}
```



## 语法格式

### 输入输出

使用 WriteLine 输出时，可以利用占位符填入变量值

```csharp
Console.WriteLine("Hello, {0},{1},{2}", var1, var2, var3);
```

可以指定保留小数位数

```csharp
double d = 1.23456;
Console.WriteLine("Hello, {0:0.000}", d);
```

这里保留 3 位小数。



使用 `@` 表示原始字符串

```csharp
Console.WriteLine(@"Hello\n\n\n");
```



使用 ReadLine, ReadKey 等方法获取用户输入

```csharp
string str = Console.ReadLine();
char c = Console.ReadKey();
```



### 命名空间

CSharp 程序内置了许多命名空间，通过这些命名空间来调用函数。例如

```cs
var n = 100;
var m = 200;
var sum = n + m;

// 输出一行的函数
System.Console.WriteLine(sum);

// 读取输入键盘的函数
System.Console.ReadKey();
```

可以像 C++ 一样使用 using 来引入命名空间

```csharp
using System;

// 可以省略 System 前缀
Console.WriteLine(sum);
Console.ReadKey();
```



### 逻辑运算

CSharp 的逻辑运算与 C++ 形式基本一致，不再赘述。此外，CSharp 提供 `is` 算符用来判断类型。例如

```csharp
char c = 'a';
if (c is char)
{
    Console.WriteLine(c);
}
```



### 循环语句

CSharp 的循环语句，基本与 C++ 中的形式一致。对于数组和集合，可以使用

```csharp
foreach (var item in collection)
{
    // do something
}
```

以此遍历所有元素。



### 异常处理

CSharp 使用类似于 C++ 的 `try...catch` 格式捕获错误，但是更加简洁

```csharp
try 
{
    // 可能出错的代码
}
catch
{
    // 如果出错执行这部分
}
```

不需要在 catch 后面指定错误类型。



## 面向对象

### 基本形式

CSharp 中类的形式与 C++ 类似，但也有一些不同。类的形式为

```csharp
[public] class class_name
{
    // 属性、方法、字段
}
```



例如在命名空间中定义类

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ConsoleApp
{
    internal class Program
    {
        static void Main(string[] args)
        {
            // 创建类
            Person person = new Person("Li Hua", 15, 85);
            person.PrintInfo();
        }
    }

    public class Person
    {
        // 访问权限分别修饰每个成员和方法
        private string name;
        public int age;
        public int score;
        
        // 定义 Name 属性
        public string Name
        {
            get { return name; }
            set { name = value; }
        }

        // 构造函数
        public Person(string name, int age, int score)
        {
            // 通过 this 获取类对象
            this.name = name;
            this.age = age;
            this.score = score;
        }

        // 输出信息
        public void PrintInfo()
        {
            Console.WriteLine("My name is: " + name);
            Console.WriteLine("My age is: " + age);
            Console.WriteLine("My score is: " + score);
        }
    }
}
```



### 属性
#### 属性方法

与 C++ 不同，类中除了方法、字段以外，还有属性。属性的作用是提供访问成员变量的方法

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        // 创建类
        Person person = new Person("Li Hua", 15, 85);
        
        // 通过 Name 属性修改 name 字段的值
        person.Name = "Test";
        person.PrintInfo();

        Console.ReadKey();
    }
}

public class Person
{
    // 私有成员
    private string name;			// 普通字段
    private static string type;		// 静态字段

    // 定义 Name 属性
    public string Name
    {
        get { return name; }
        set { name = value; }
    }
    // 属性可以展开为一个普通方法
    
    // 定义 Type 属性
    public static string Type
    {
        get { return type; }
        set { type = value; }
    }
}
```



#### 自动属性

CSharp 的属性功能，不仅可以提供访问成员变量的接口，还可以自动提供字段。例如定义自动属性，它会自动生成一个对应的字段，以及访问用的 get, set 方法。

```csharp
public class Person
{
    public void SayHello()
    {
        Console.WriteLine("Hello");
    }
	
    // 自动属性
    public string Name
    {
        // 不能写函数体
        // get { return ... }
        get;
        set;
    }
}
```



### 访问权限

CSharp 提供 4 种访问修饰，用于修饰成员

* public 公共权限，任何地方可以访问
* private 私有权限，只有类内部可访问（默认）
* protected 保护权限，只有类、子类内部可访问
* internal 内部权限，只能在**当前项目**中访问

以及 2 种访问修饰，用于修饰类本身

* public 公共权限，任何地方可以访问
* internal 内部权限，只能在**当前项目**中访问

子类的访问权限不能高于父类的访问权限。



### 构造函数

CSharp 的构造函数格式与 C++ 类似，但是对于初始化列表有一些不同。例如

```csharp
public class Person
{
	// 构造函数
    public Person(string name, int age, int score)
    {
        this.name = name;
        this.age = age;
        this.score = score;
    }

    // 重载构造函数，通过 this 调用其它构造函数
    public Person(string name, int age, int score, int height) : this(name, age, score)
    {
        this.height = height;
    }
}
```

可以直接将字段初始化写在声明处

```cpp
public class A
{
    public int x = 0;
}
```



### 析构函数

CSharp 的析构函数格式与 C++ 类似，用于手动释放资源。通常 CSharp 由垃圾回收器 GC(Garbage Collection) 自动释放资源。

```csharp
public class Person
{
    // 析构函数
    ~Person()
    {
        // 释放资源
    }
}
```



### 静态方法

与 C++ 不同，CSharp 中的静态方法通过 `.` 来调用。例如

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        Person.Print();
    }
}

class Person
{
   	static void Print()
    {
		Console.WriteLine("Hello World");
    }
}
```

所有方法都是静态方法的类就是静态类，只提供方法，不需要构建实例。



### 成员函数

CSharp 中的函数几乎都是以成员函数出现，因此我们直接通过介绍成员函数来介绍函数。CSharp 中函数可以直接返回数组，例如

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        // 获得长度为 5 的数组
        var a = GetIntArr(5);
        
        Console.WriteLine(a.Length);
        Console.ReadKey();
    }

    public static int[] GetIntArr(int n)
    {
        return new int[n];
    }
}
```



#### out 参数

使用 out 参数来返回多个具有不同类型的值。例如

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        // 可以不赋值
        float a;
        double b;
        
        // 使用时添加 out 前缀
        Test(5, out a, out b);
        
        Console.WriteLine(a);
        Console.WriteLine(b);
        Console.ReadKey();
    }

    public static void Test(int n, out float a, out double b)
    {
        // out 参数必须赋值，作为函数返回值
        a = (float)(n / 2.0);
        b = n / 3.0;
    }
}
```

out 侧重于返回多个不同类型的值，外部变量可以先不赋值，但传入后必须赋值。



#### in 参数

使用 in 参数以常量引用外部变量，确保在函数内部不能修改这些变量

```cpp
public float F(in Vector3 x, in Vector3 y)
{
    return x.x * y.x + x.y * y.y + x.z * y.z;
}
```



#### ref 参数

使用 ref 参数引用外部变量

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        // 必须赋值
        float a = 10;
        double b = 50;

        // 使用时添加 ref 前缀
        Test(5, ref a, ref b);

        Console.WriteLine(a);
        Console.WriteLine(b);
        Console.ReadKey();
    }

    public static void Test(int n, ref float a, ref double b)
    {
        a = n + a;
        b = n + b;
    }
}
```

ref 要求外部变量传入前必须赋值，传入后可以不赋值。



#### params 参数

使用 params 提供可变参数

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        int c = Test(5, 6, 7, 8);

        Console.WriteLine(c);
        Console.ReadKey();
    }
	
    // 可变参数只能位于最后一个，因此不能有多个可变参数
    public static int Test(int n, params int[] score)
    {
        for (int i = 0; i < score.Length; i++) { n += score[i]; }
        return n;
    }
}
```

从效果上来看，它和下面代码一致

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        // 定义一个数组再传入
        int[] s = { 6, 7, 8 };
        int c = Test(5, s);

        Console.WriteLine(c);
        Console.ReadKey();
    }

    public static int Test(int n, int[] score)
    {
        for (int i = 0; i < score.Length; i++) { n += score[i]; }
        return n;
    }
}
```



### 类继承

#### Object

CSharp 中所有类都具有一个公共基类 Object，因此可以用 Object 作为类型接收任意类型参数

```csharp
void SayHello(Object obj);
```

另外还有 object 类型，它作为一个关键字表示任何类型，与 Object 是同一个含义。



Object 类提供了几个虚方法

| 方法          | 作用                     | 方法       | 作用         |
| ------------- | ------------------------ | ---------- | ------------ |
| GetType()     | 获得类型                 | ToString() | 转换为字符串 |
| GetHashCode() | 获得当前对象的 Hash 代码 | Equals()   | 比较相等     |

这些方法都可以在子类中重载。



#### 单一继承

CSharp 中的类继承只允许继承一个父类

```csharp
public class Person
{
    public string name;
    public int age;
    public int score;

    // 构造函数
    public Person(string name, int age, int score)
    {
        this.name = name;
        this.age = age;
        this.score = score;
    }
}

public class Student : Person
{
    // 通过 base 来调用父类构造函数
    public Student() : base("Li Hua", 15, 85)
    {
    }
}
```



#### 隐藏方法

在子类中定义的与父类同名的方法会覆盖父类的方法，使用 new 声明在子类中定义一个新的同名方法，而不是覆盖父类的方法。这意味着**父类对象仍然可以调用自身的方法而非被子类方法覆盖**。

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        Person p1 = new Person();
        Student s1 = new Student();

        p1.SayHello();	// 调用父类方法
        s1.SayHello();	// 调用子类方法

        Person p2 = s1;
        p2.SayHello();	// 调用父类方法
        Console.ReadKey();
    }
}

public class Person
{
    public void SayHello()
    {
        Console.WriteLine("Person say Hello.");
    }
}

public class Student : Person
{
    // 使用 new 关键字，显式声明要隐藏父类的方法
    public new void SayHello()
    {
        Console.WriteLine("Student say Hello.");
    }
}
```

注意到当父类引用子类对象时，调用的会是父类方法，只有子类对象会隐藏父类方法。



#### 里氏转换

类似于 C++，可以将子类成员赋值给父类。例如

```csharp
Person p = new Student();
Student s = (Student)p;
```

通过 `is` 算符可以判断对象类型，实现分支强转

```csharp
if (p is Teacher)
{
    Teacher t = (Teacher)p;
}
else
{
    Student s = (Student)p;
}
```

通过 `as` 算符转换，如果成功返回对象，否则返回 null

```csharp
Person p = new Student();
Teacher t = p as Teacher;
// 这里转换失败，因此 t 是 null
```



#### 虚方法

借助虚方法实现多态，其中 override 声明覆盖父类方法的子类方法，这意味着**父类对象调用自身的方法会被子类方法覆盖**。 

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        Person p1 = new Person();
        Student s1 = new Student();

        p1.SayHello();	// 调用父类方法
        s1.SayHello();	// 调用子类方法

        Person p2 = s1;
        p2.SayHello();
        Console.ReadKey();	// 调用子类方法
    }
}

public class Person
{
	// 声明虚方法
    public virtual void SayHello()
    {
        Console.WriteLine("Person say Hello.");
    }
}

public class Student : Person
{
    // 使用 override 关键字，显式声明要重写父类的方法
    public override void SayHello()
    {
        Console.WriteLine("Student say Hello.");
    }
}
```



#### 抽象方法

CSharp 中抽象方法通过 abstract 声明，并且所在类也必须由 abstract 声明

```csharp
// 必须同时声明抽象类
public abstract class Person
{
    // 声明虚方法，无实现
    public abstract void SayHello();
}

public class Student : Person
{
    public override void SayHello()
    {
        Console.WriteLine("Student say Hello.");
    }
}

public class Teacher : Person
{
    public override void SayHello()
    {
        Console.WriteLine("Teacher say Hello.");
    }
}
```



### 序列化

数据传输需要序列化，即将对象转换为二进制数据。接收到数据后进行反序列化，得到原始的对象数据。需要先将类标记为可序列化

```csharp
// 标记为可序列化
[Serializable]
public class Person
{
    public void SayHello()
    {
        Console.WriteLine("Hello");
    }
}
```

使用 BinaryFormatter 进行序列化操作

```csharp
Person p = new Person();

using (FileStream fsWrite = new FileStream("E/Desktop/test.txt", FileMode.Open))
{
    // 创建序列化装置
    BinaryFormatter bf = new BinaryFormatter();
    bf.Serialize(fsWrite, p);
}

using (FileStream fsRead = new FileStream("E/Desktop/test.txt", FileMode.Open))
{
    // 创建序列化装置
    BinaryFormatter bf = new BinaryFormatter();
    Person p2 = (Person)bf.Deserialize(fsRead);
}
```



### 特殊类

#### 部分类

使用 partial 声明一个部分类，这允许同时存在多个同名的类，并且这些同名类的成员之间都可以互相访问。例如

```csharp
// 部分类
public partial class Person
{
    public string name;
}

public partial class Person
{
    public void SayWorld()
    {
        // 可访问 name
        Console.WriteLine(name);
    }
}
```



#### 密封类

使用 sealed 声明一个密封类，其它类都不能继承这个类

```csharp
// 当前类不可继承
public sealed class Person
{
    public void SayHello()
    {
        Console.WriteLine("Hello");
    }
}
```



#### 接口类

使用 interface 声明一个接口类，其它类可以继承多个接口类，但接口不能继承其它类。

> 接口类可以作为基类被继承，它**相当于一个特殊的抽象类**。

```csharp
public class Person
{
}

// 允许多继承接口类，注意普通类应该写在接口类之前
public class Student : Person, IPerson
{
    // 必须实现接口函数
    public void Play()
    {
        Console.WriteLine("I can play");
    }
}

public interface IPerson
{
    // 声明一个接口功能，注意不能添加访问权限修饰，默认为 public
    // 接口方法不能有定义
    void Play();
    
    // 不能添加任何字段
    // string name;
    
    // 由于自动属性没有声明任何字段，也没有定义方法，因此可以使用
    string Name
    {
        get;
        set;
    }
}
```



当需要定义的接口类方法与类自身的方法重名时，应该显式定义

```csharp
public class Student : IPerson
{
    // 显式实现 IPerson 的接口函数，注意不能加修饰符
    void IPerson.Play()
    {
        Console.WriteLine("I can play");
    }

    // Student 自身的 Play 方法
    public void Play()
    {
        Console.WriteLine("i can play");
    }
}

public interface IPerson
{
    void Play();
}
```

在多态时，两个方法可以看出区别

```csharp
// 这里会调用 Student 自身的 Play
Student s = new Student();
s.Play();

// 这里会调用显式实现的 IPerson 接口 
IPerson p = new Student();
p.Play();
```



### 多文件项目

#### 单项目

对于具有多个文件但是在同一个项目中的情况，可以直接访问不同文件中的命名空间。例如

```csharp
// Program.cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Utils

namespace ConsoleApp
{
    internal class Program
    {
        static void Main(string[] args)
        {
            Person person = new Person("Li Hua", 15, 85);
            person.PrintInfo();
            
            // 暂停程序
            Console.ReadKey();
        }
    }
}

// Person.cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Utils
{
    public class Person
    {
        public string name;
        public int age;
        public int score;

        // 构造函数
        public Person(string name, int age, int score)
        {
            this.name = name;
            this.age = age;
            this.score = score;
        }

        // 输出信息
        public void PrintInfo()
        {
            Console.WriteLine("My name is: " + name);
            Console.WriteLine("My age is: " + age);
            Console.WriteLine("My score is: " + score);
        }
    }
}
```

当两个文件中的类处于不同的命名空间中，只需要 `using` 就可以使用。



#### 多项目

可以新创建一个项目，存放 Person 类。在其项目设置中修改输出类型为“类库”，然后再当前项目添加引用，这样就可以使用 Person 项目中的类

![[CSharp.assets/image-20240222004500972.png]]



## 泛型
### 泛型类型

CSharp 也提供类似于 C++ 模板的泛型编程方法。例如

```csharp
List<int> list = new List<int>();
int[] arr = list.ToArray();		// 泛型转数组
List<int> list2 = arr.ToList();	// 数组转泛型
```

提供针对不同类型的数组。



HashTable 是 object 与 object 之间的映射，而 Dictionary 则提供泛型映射

```csharp
Dictionary<int, string> dic = new Dictionary<int, string>();
```



### 装箱拆箱

将值类型转换为对象引用类型称为装箱，而将对象引用类型转换为值类型则称为拆箱。例如

```csharp
int n = 10;
object o = n;		// 装箱
int nn = (int)o;	// 拆箱
```

装箱拆箱过程**取决于是否具有继承关系**，如果没有继承关系则没有装箱拆箱。

>[!note]
>慎重使用子类对象与基类引用之间的转换，尽可能借助泛型。



装箱拆箱过程会造成大量的性能浪费。例如

```csharp
ArrayList list = new ArrayList();
list.Add(1);
list.Add("Hello");
```

ArrayList 通过 object 类型管理其中的元素，因此向其中推入的元素会转换为 object 类型，这个过程中就会发生装箱。而泛型集合

```csharp
List<int> list = new List<int>();
list.Add(1);
list.Add(2);
```

就不会发生装箱拆箱，因此效率更高。


### 泛型方法

考虑一个获得类型名的方法

```cpp

internal class Program
{
    static void Main(string[] args)
    {
        string a = "abc";
        var type = GetClassType(a);

        Console.Write(type);
        Console.ReadKey();
    }

	public static string GetClassType(object obj)
    {
    	// 获得类型名
        return obj.GetType().Name;
    }
}
```

为了让该方法对任何类型都可用，一种方法是用 `object` 作为参数类型，但是这样会造成装箱拆箱过程的浪费。可以按照如下方式定义并使用泛型方法

```cpp
internal class Program
{
    static void Main(string[] args)
    {
        string a = "abc";
        var type = GetClassType(a);

        Console.Write(type);
        Console.ReadKey();
    }

    public static string GetClassType<T>(T obj)
    {
    	// 获得类型名
        return obj.GetType().Name;
    }
}
```



如果需要返回泛型对象，而不确定泛型是值还是引用，可以使用 `default` 关键字

```cpp
public T GetDefault<T>()
{
    return default; // 获取 T 类型的默认值
}
```

此时

```cpp
int defaultInt = default(int);         		// 默认值是 0
bool defaultBool = default(bool);       	// 默认值是 false
string defaultString = default(string); 	// 默认值是 null
MyClass defaultClass = default(MyClass); 	// 默认值是 null (引用类型)
```



### 泛型类

定义可以容纳任意类型的泛型类

```cpp
public class MyList<T>
{
    List<T> list = new List<T>();

    public void Add(T obj)
    {
        list.Add(obj);
    }
}
```



### 泛型约束

在泛型定义中，有时使用 `where` 子句限制类型的范围。规则为

- 可以同时使用多个约束
- 值类型约束与引用类型约束不能同时使用
- 引用类型约束与基类约束不能同时使用
- 如果有基类约束，那么它应当在最前面
- 如果有 `new()` 约束，那么它应当在最后面



#### 值类型约束

使用 `struct` 约束泛型类型必须是值类型

```cpp
public T ToType<T>(string str) where T : struct
{
    return (T)Convert.ChangeType(str, typeof(T));
}

ToType<int>("3.9");		// 正确，int 是值类型
ToType<string>(3.9);	// 错误，string 是引用类型
```



#### 引用类型约束

使用 `class` 约束泛型类型必须是引用类型

```cpp
public T ToType<T>(string str) where T : class
{
    return (T)Convert.ChangeType(str, typeof(T));
}

ToType<int>("3.9");		// 错误，int 是值类型
ToType<string>(3.9);	// 正确，string 是引用类型
```



#### 基类约束

给出具体的类型约束泛型类型必须是该类型或该类型的子类

```cpp
public class Base
{
    public static Base ToBase<T>(T obj) where T : Base
    {
        return (Base)Convert.ChangeType(obj, typeof(Base));
    }
}
```



#### 接口约束

接口约束要求泛型类型必须实现接口，也就是要求泛型是接口类的子类，等价于基类约束

```cpp
public class Person : IPerson
{
    // 实现接口函数
    public void Play()
    {
        Console.WriteLine("I can play");
    }
}

public interface IPerson
{
    void Play();
}

public class A
{
    public static void Play<T>(T obj) where T : IPerson
    {
        obj.Play();
    }
}
```



#### 无参构造函数约束

使用 `new()` 要求泛型必须具有无参构造函数

```cpp
public class A
{
    public static T CreateObject<T>() where T : new()
    {
        return new T();
    }
}
```



#### 多重约束

将前面的约束组合在一起，需要满足规则顺序

```cpp
public class Person
{
}

public class Student : Person, IPerson
{
    // 实现接口函数
    public void Play()
    {
        Console.WriteLine("I can play");
    }
}

public interface IPerson
{
    void Play();
}

public class A
{
    public static void Play<T>(T obj) where T : Person, IPerson, new()
    {
        obj.Play();
    }
}
```



## 反射

反射允许程序在运行时访问类中的成员和成员信息，即**元数据**。需要的类包括

- `System.Type` 类的信息类，可以获得类的构造函数、方法、字段、属性、事件等
- `System.Relection.Assembly` 程序集类，加载其它程序集，之后能用 `Type` 获得其中某个类的信息
- `Activator` 快速实例化对象的类，可以将 `Type` 对象直接实例化为对应的类



### 动态实例化

考虑定义在另一个命名空间中的类

```cpp
namespace Test
{
    public class A
    {
        public int x;

        public A(int x)
        {
            this.x = x;
        }
    }
}
```

现在可以通过全名 `Test.A` 获得信息类，然后用 `Activator` 创建实例

```cpp
Type type = Type.GetType("Test.A");
Test.A a = (Test.A)Activator.CreateInstance(type, 10);
```

由于构造函数需要参数，因此 `CreateInstance` 需要获得类型和参数。也可以用 `typeof` 获得类型来创建

```cpp
Type type = typeof(A);
Test.A a = (Test.A)Activator.CreateInstance(type, 10);
```



### 元数据

#### 属性

假设有如下的类

```cpp
public class Person
{
    public int Age { set; get; }
    public string Name { set; get; }
    public string Address { set; get; }

    public Person(int age, string name, string address)
    {
        Age = age;
        Name = name;
        Address = address;
    }
}
```

可以通过 `PropertyInfo` 获得属性，以类实例为参数可以修改其中的属性值

```cpp
Person person = new Person(10, "Li", "Bei", "110", "man");
Type type = typeof(Person);

// 获得属性列表
PropertyInfo[] properties = type.GetProperties();
foreach (var prop in properties)
{
	// 获得属性名，属性值
    Console.WriteLine(prop.Name);
    Console.WriteLine(prop.GetValue(person));
}

// 获得单个属性
PropertyInfo property = type.GetProperty("Age");
property.SetValue(person, 12);
Console.WriteLine(property.GetValue(person));
```



#### 字段

假设有如下的类

```cpp
public class Person
{
    public string tel;
    private string sex;
    public static int count;

    public Person(string tel, string sex)
    {
        this.tel = tel;
        this.sex = sex;
    }
}
```

可以通过 `FieldInfo` 获得字段，以类实例为参数可以修改其中的字段值

```cpp
FieldInfo[] fields = type.GetFields();
foreach (var field in fields)
{
    Console.WriteLine(field.Name);
    Console.WriteLine(field.GetValue(person));
}

FieldInfo tel = type.GetField("tel");
tel.SetValue(person, "120");
Console.WriteLine(tel.GetValue(person));
```

默认情况下只能获得公有字段，如果需要获得其它字段，使用 `BindingFlags` 枚举

- `NonPublic` 非公有
- `Instance` 实例成员
- `Static` 静态成员

获得 `sex` 字段需要使用前两个枚举

```cpp
FieldInfo sex = type.GetField("sex", BindingFlags.NonPublic | BindingFlags.Instance);
sex.SetValue(person, "woman");
Console.WriteLine(sex.GetValue(person));
```



#### 方法

假设有如下的类

```cpp
public class Person
{
    public string Tel { set; get; }
    private string sex;

    public Person(string tel, string sex)
    {
        this.Tel = tel;
        this.sex = sex;
    }

    public string GetSex()
    {
        return sex;
    }

    public void SetSex(string sex)
    {
        this.sex = sex;
    }

    public void SetAndPrint(string tel, string sex)
    {
        this.Tel = tel;
        this.sex = sex;
        Console.WriteLine(Tel);
        Console.WriteLine(sex);
    }
}
```

可以通过 `MethodInfo` 获得方法

```cpp
MethodInfo[] methods = type.GetMethods();
foreach (var m in methods)
{
    Console.WriteLine(m.Name);
}

// 无参调用需要 null
MethodInfo m1 = type.GetMethod("GetSex");
m1.Invoke(person, null);

// 指定函数参数类型列表
MethodInfo m2 = type.GetMethod("SetSex", new Type[] { typeof(string) });
m2.Invoke(person, new object[] { "woman" });
Console.WriteLine(person.GetSex());

// 多参数调用
MethodInfo m3 = type.GetMethod("SetAndPrint");
m3.Invoke(person, new object[] { "121", "man" });
```

注意到 `MethodInfo.Invoke` 需要 `object, object[]` 两个参数，因此即使无参调用也需要传入 `null` 作为 `object[]` 类型。在 `GetMethod` 中可以执行获得方法的参数列表，例如

```cpp
type.GetMethod("SetSex", new Type[] { typeof(string) });
```

表示获得名称为 `SetSex` 的方法，并要求它以 `string` 为参数类型。在 `Invoke` 中多参数通过数组传入

```cpp
m3.Invoke(person, new object[] { "121", "man" });
```



#### 泛型方法

假设有如下的类

```cpp
public class Person
{
    public T Print<T>(T t)
    {
        Console.WriteLine(t);
        return t;
    }
}
```

获得方法时要先转换为泛型方法

```cpp
MethodInfo m = type.GetMethod("Print");
MethodInfo mt = m.MakeGenericMethod(typeof(string));	// T = string
mt.Invoke(person, new object[] { "hi" });
```



#### 静态成员

静态成员不需要传入类实例

```cpp
public class Person
{
    public static int count;
}

FieldInfo f = type.GetField("count");
f.SetValue(null, 12);
Console.WriteLine(f.GetValue(null));
```



#### 构造函数

还可以将构造函数提取出来，通过 `GetConstructor` 传入参数类型列表获得指定的构造函数

```cpp
ConstructorInfo cons = type.GetConstructor(new Type[] { typeof(string), typeof(string) });
Person p = (Person)cons.Invoke(new object[] { "120", "woman" });
```



### 动态加载程序集

有时需要加载一个动态库中的类型，首先要加载动态库，共有 3 种方式

```cpp
Assembly ass = null;
ass = Assembly.Load("name");			// 在 .Net Framework 下可行，在 .Net8 需要引用项目
ass = Assembly.LoadFile("path");
ass = Assembly.LoadFrom("name.dll");
```

然后在动态库中获得类型

```cpp
Type type = ass.GetType("...");
```



### 结合泛型

反射与泛型结合，可以对任意对象信息进行收集和输出

```cpp
public class BaseAct<T>
{
    public List<T> list = new List<T>();

    public void Add(T t)
    {
        list.Add(t);
    }

    // 获得泛型中属性的信息并格式化输出
    public string GetInfo(T t)
    {
        string str = "";
        Type type = typeof(T);
        var props = type.GetProperties();
        foreach (var prop in props)
        {
            if (str != "")
                str += ",";
            str += prop.GetValue(t).ToString();
        }
        str += "\r\n";
        return str;
    }

    // 在泛型列表中查找对应特定属性 idName 为 id 的泛型对象
    public T FindModel(int id, string idName)
    {
        Type type = typeof(T);
        var propId = type.GetProperty(idName);
        if (propId != null)
        {
            foreach (var t in list)
            {
                if ((int)propId.GetValue(t) == id)
                    return t;
            }
        }
        // 返回空泛型
        return default;
    }
}
```

使用时先根据属性查找泛型对象，然后获得泛型对象的信息

```cpp
BaseAct<Person> baseAct = new BaseAct<Person>();
baseAct.Add(new Person(110));
baseAct.Add(new Person(120));
Person p = baseAct.FindModel(110, "Id");
string info = baseAct.GetInfo(p);
Console.WriteLine(info);
```

这样就将任意类型的同类操作通过一个泛型对象完成。



## 委托

### 显式委托

用 `delegate` 声明一个委托类型，它可以放在任何位置，其效果类似于仿函数。通过传入函数名来实例化，用于保存要调用的函数。例如

```cpp
delegate int Calculate(int a, int b);

internal class Program
{
    static void Main(string[] args)
    {
    	// 委托实例化
        Calculate cal = new Calculate(add);
        cal(10, 12);

		// 直接获得函数签名，赋值委托
        cal = minus;
        minus(12, 10);

        Console.ReadKey();
    }


    public static int add(int a, int b)
    {
        return a + b;
    }
    public static int minus(int a, int b)
    {
        return a - b;
    }
}
```



### 多播委托

还可以分步同类委托，用 `+=` 添加委托函数，用 `-=` 移除委托函数。最后得到委托函数的列表，此时执行委托会按照顺序执行委托函数。例如

```cpp
delegate void ShowMsg();

internal class Program
{
    static void Main(string[] args)
    {
        ShowMsg show = ShowHello;
        show += ShowWorld;
        show();

        Console.ReadKey();
    }

    public static void ShowHello()
    {
        Console.Write("Hello");
    }

    public static void ShowWorld()
    {
        Console.Write("World");
    }
}
```

如果委托函数有返回值，委托最终得到最后一个委托函数的返回值。



### 匿名委托

可以直接在委托位置使用 `delegate` 声明

```cpp
Calculate cal = delegate (int a, int b)
{
    return a - b;
};
```

这就是匿名委托过程。



在原始的匿名委托基础上，后来 CSharp 又提供了 lambda 表达式，保存为委托

```cpp
Calculate cal = (a, b) =>
{
    var c = a + b;
    return c * c;
};
```

单行 lambda 表达式可以通过 `=>` 连接输入参数和返回值表达式

```cpp
Calculate cal = (a, b) => a + b;
```

某些方法可以接收委托，因此可以接收 lambda 表达式

```cpp
List<int> list = new List<int>();
list.Add(12);
list.Add(10);
list.RemoveAll(id => id == 12);
```



### 标准委托

在 `.Net` 中提供了自带的委托类型。



#### Action 委托

用于封装无返回值的方法

```cpp
// 无参数委托（注意 WriteLine 无返回值）
Action act1 = () => Console.WriteLine("Hello World");
act1();

// 单参数委托（注意 WriteLine 无返回值）
Action<string> act2 = name => Console.WriteLine(name);
act2("Bad");

// 双参数委托
Action<int, int> act3 = (a, b) =>
{
    var c = a + b;
    Console.WriteLine(c);
};
act3(10, 12);
```



#### Func 委托

用于封装带有返回值的方法，其中前面的泛型为参数类型，最后的泛型为返回类型

```cpp
// 无参数委托，返回 int
Func<int> func1 = () => 0;
func1();

// 双参数委托，返回 string
Func<int, int, string> func2 = (a, b) => (a + b).ToString();
func2(2, 3);
```



### 事件委托

事件允许类或对象的某些状态发生改变时通知其它类或对象。事件在一个类中声明和触发，并通过委托与事件处理程序关联。事件分为

- 发布器：包含事件和委托的类
- 订阅器：接收事件并处理事件的类



#### 空参数事件

首先通过 `event` 声明事件

```cpp
public class Sender
{
    public event EventHandler handler;

    public void DoSomething()
    {
        OnEvent();
    }

    public void OnEvent()
    {
    	// 根据委托的形式，调用格式要求提供 this 和事件参数，默认为空
        handler?.Invoke(this, EventArgs.Empty);
    }
}
```

其中 `EventHandler` 是标准库提供的事件句柄，它是一个委托，形式为

```cpp
delegate void EventHandler(object sender, EventArgs e);
```



在其它类中订阅事件，只需要实例化发布器，然后将类的某个需要回调的函数注册到发布器中

```cpp
internal class Program
{
    static void Main(string[] args)
    {
    	// 定义发布器，注册两个订阅器
        Sender some = new Sender();
        some.handler += Dealer1.Sender_handler;
        some.handler += Dealer2.Sender_handler;

		// 发布消息
        some.DoSomething();

        Console.ReadKey();
    }
}

public class Dealer1
{
    public static void Sender_handler(object sender, EventArgs e)
    {
        Console.WriteLine("Hello");
    }
}

public class Dealer2
{
    public static void Sender_handler(object sender, EventArgs e)
    {
        Console.WriteLine("World");
    }
}
```

当发布器发布事件时，就由订阅器的回调函数执行处理事件。



#### 带参数事件

可以为事件指定参数，做法是继承 `EventArgs` 类，然后在子类中给出参数。例如

```cpp
// 自定义事件参数类
public class RecordArgs : EventArgs
{
    public string Msg { get; set; }
}

public class Sender
{
	// 现在委托类型为 delegate void EventHandler(object sender, RecordArgs e);
    public event EventHandler<RecordArgs> handler;

    public void DoSomething(string msg)
    {
    	// 输入信息，保存到参数类中，交给处理函数
        RecordArgs args = new RecordArgs();
        args.Msg = msg;
        OnEvent(args);
    }

    public void OnEvent(RecordArgs args)
    {
        handler?.Invoke(this, args);
    }
}
```

接着由其它类订阅该消息

```cpp
internal class Program
{
    static void Main(string[] args)
    {
        Sender some = new Sender();
        some.handler += Dealer1.Sender_handler;
        some.handler += Dealer2.Sender_handler;
        some.DoSomething("Hello ");

        Console.ReadKey();
    }
}

public class Dealer1
{
    public static void Sender_handler(object sender, RecordArgs e)
    {
        Console.WriteLine(e.Msg + "Li");
    }
}
public class Dealer2
{
    public static void Sender_handler(object sender, RecordArgs e)
    {
        Console.WriteLine(e.Msg + "Liu");
    }
}
```



## 特性

### 语法格式

特性本质上是类，继承自 `Attribute` 类。例如如下的特性

```cpp
public class TestAttribute : Attribute
{
}

[Test]
public class Person
{
	[Test]
	public int Id { get; set; }
}
```

特性应当以 `Attribute` 结尾，使用时可以省略 `Attribute` 。



### 配置特性

特性可以放在类、属性、字段、方法等任何位置之前，特性之前也可以放置特性。使用使用标准特性 `AttributeUsage` 配置自定义特性。



#### 特性目标

使用标准特性 `AttributeUsage` 指定当前特性的目标只能是类或者方法

```cpp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class TestAttribute : Attribute
{
}
```

此时 `[Test]` 不能放在属性之前了。



#### 构造特性

特性作为类具有构造函数，因此在标记特性时可以指定构造参数

```cpp
[AttributeUsage(AttributeTargets.Class)]
public class TestAttribute : Attribute
{
    public TestAttribute()
    {
    }

    public TestAttribute(string str)
    {
    }
}

[Test("newTest")]
public class Person
{
    public int Id { get; set; }
}
```



#### 重复特性

可以重复指定特性，此时需要将 `AllowMultiple` 设为 `true`

```cpp
[AttributeUsage(AttributeTargets.Class, AllowMultiple = true)]
public class TestAttribute : Attribute
{
    public TestAttribute()
    {
    }

    public TestAttribute(string str)
    {
    }
}

[Test]
[Test("newTest")]
public class Person
{
    public int Id { get; set; }
}
```



#### 继承特性

将 `Inherited` 设为 `true` 表示子类将继承父类的特性

```cpp
[AttributeUsage(AttributeTargets.Class, Inherited = true)]
public class TestAttribute : Attribute
{
}

[Test]
public class Person
{
    public int Id { get; set; }
}

// Student 具有 Test 特性
public class Student
{
}
```



### 查找特性

可以通过反射查找到特性。我们首先考虑如下特性

```cpp
public class TestAttribute : Attribute
{
    public string TestName { set; get; }
    public TestAttribute()
    {
    }

    public TestAttribute(string name)
    {
        TestName = name;
    }
}
```

将该特性应用于另一个类，并给出构造参数

```cpp
[Test("newTest")]
public class Person
{
    public int Id { get; set; }
}
```

我们可以通过一个泛型函数查找具有 `Test` 特性或者父类具有 `Test` 特性的类型，并返回 `Test` 特性中保存的内容

```cpp
public static string GetName<T>(T t) where T : class
{
    Type type = typeof(T);
    // 检查 T 是否具有 Test 特性；第二个参数表示是否考虑继承，设为 true 则父类具有 Test 特性也可以
    if (type.IsDefined(typeof(TestAttribute), true))
    {
    	// 获得 T 中的特性列表，返回第一个特性的 TestName 属性值
        var attr = type.GetCustomAttributes(typeof(TestAttribute), true);
        return ((TestAttribute)attr[0]).TestName;
    }
    // 没有找到 Test 特性，则返回 T 的类型名
    return type.Name;
}
```



### 特性检测

通过特性，我们可以检查属性是否有效。首先我们定义一个带有抽象接口的特性

```cpp
public abstract class CommmonValidateAttribute : Attribute
{
    public abstract bool IsValidate(object obj);
}
```

然后定义一个需要检查的类

```cpp
public class Person
{
    [StringLength(6, 50)]
    public string Name { get; set; }

    [Required]
    public int Age { get; set; }
}
```

我们希望要求 `Name` 字符串的长度满足其一定范围，要求一定要初始化 `Age` 变量。于是我们继承公共特性

```cpp
public class StringLengthAttribute : CommmonValidateAttribute
{
    public int MinLength { get; set; }
    public int MaxLength { get; set; }

	// 初始化时保存长度范围
    public StringLengthAttribute(int minLength, int maxLength)
    {
        MinLength = minLength;
        MaxLength = maxLength;
    }

    public override bool IsValidate(object obj)
    {
		if (obj != null)
        {
            int length = obj.ToString().Length;
            return length >= MinLength && length <= MaxLength;
        }
        return false;
    }
}

public class RequiredAttribute : CommmonValidateAttribute
{
    public override bool IsValidate(object obj)
    {
        return obj != null && obj.ToString().Length != 0;
    }
}
```

最后通过查找特性并调用抽象接口来判断

```cpp
public static bool Validate<T>(T t) where T : class
{
	// 获得所有属性
    PropertyInfo[] propertyInfos = typeof(T).GetProperties();
    foreach (var prop in propertyInfos)
    {
    	// 判断属性是否有 CommmonValidateAttribute 特性
        if (prop.IsDefined(typeof(CommmonValidateAttribute), true))
        {
        	// 获得属性的所有 CommmonValidateAttribute 特性
            var attrs = prop.GetCustomAttributes(typeof(CommmonValidateAttribute), true);
            foreach (var attr in attrs)
            {
            	// 调用特性接口判断属性是否有效
                var comm = attr as CommmonValidateAttribute;
                if (!comm.IsValidate(prop.GetValue(t)))
                    return false;
            }
        }
    }
    return true;
}
```

因此特性相当于修饰属性，可以用来自动化检查属性信息。

