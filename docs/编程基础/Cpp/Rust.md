# Rust

## 基本介绍

### 安装配置

在官网下载安装器运行即可安装

```embed
title: "Install Rust"
image: "https://www.rust-lang.org/static/images/rust-social-wide.jpg"
description: "A language empowering everyone to build reliable and efficient software."
url: "https://www.rust-lang.org/tools/install"
```

安装完成后命令行输入

```shell
rustc --version
```

如果要更新或卸载，输入

```shell
rustup update
rustup self uninstall
```

可以打开本地文档

```shell
rustup doc
```



### 基本示例

#### 简单程序

编写一个简单的 Rust 程序

```rust
fn main() {
    println!("Hello, world!");
}
```

命令行输入

```shell
rustc main.rs
```

将文件编译为 exe 可执行文件。在 Windows 上编译还会生成 pdb 文件包含调试信息。



#### Cargo

编写项目程序通常使用 Cargo，执行

```shell
cargo new hello_cargo
```

自动生成一个 `hello_cargo` 项目，其中包含 git 配置文件、源码和目标文件夹。



要构建项目，使用

```shell
cargo build
```

将会在 `target/debug` 下生成可执行程序。可以在编写代码中经常检查代码，确保能够通过编译

```shell
cargo check
```

要运行项目，使用

```shell
cargo run
```

要发布项目，使用

```shell
cargo build --release
```

将会在 `target/release` 下生成优化的可执行程序。



### 依赖库

要安装依赖库，只需要在配置文件的依赖项中添加对应的库和版本

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

[dependencies]
rand = "0.8.5"
```

然后执行 `cargo build` 即可自动下载并编译库文件。



也可以更简单地使用

```shell
cargo add rand
cargo remove rand
```

还可以指定版本、特性甚至仓库

```shell
cargo add serde --vers 1.0.130
cargo add serde --vers ">=1.0.0, <2.0.0"
cargo add serde --features derive
cargo add serde --git https://github.com/serde-rs/serde
```



### 猜数游戏

```rust
// 导入模块
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    // println! 宏
    println!("猜数字游戏");

    // 生成一个随机数，类型自动推导（根据后续代码推导）为 u32
    let secret_number = rand::thread_rng().gen_range(1..101);

    loop {
        println!("请输入一个数字：");

        // mut 关键字用于声明变量为可变的
        // new 构造空字符串
        let mut guess = String::new();

        // &mut 可变引用
        io::stdin().read_line(&mut guess).expect("无法读取行");
        // readline 返回 io::Result 类型 Ok, Err
        // expect 处理返回类型

        // 转换字符串为数字（变量隐藏）
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                println!("输入的不是数字，请重新输入");
                continue;
            }
        };

        // 格式化字符串
        println!("你猜的数字是：{}", guess);

        // 比较猜测的数字和随机数的大小
        match guess.cmp(&secret_number) {
            Ordering::Less => println!("猜小了，再试试"),
            Ordering::Greater => println!("猜大了，再试试"),
            Ordering::Equal => {
                println!("恭喜你猜对了！");
                println!("游戏结束");
                break;
            }
        }
    }
}
```



## 基本语法

### 变量与可变性

#### 变量

Rust 中使用 `let` 声明变量，默认是不可变的变量

```rust
let x = 5;
x= 6;	// 编译器报错
```

要声明一个可变变量，需要添加 `mut` 关键字

```rust
let mut x - 5;
x = 6;
```



#### 常量

使用大写字母命名常量

```rust
const MAX_POINTS: u32 = 100_000;	// 整数可以添加 _ 分隔，便于阅读
```

常量也是不可变的，同时

* 不能使用 `mut` 关键字
* 使用 `const` 声明，必须标注类型
* 可以在任何作用域声明
* 只能绑定到常量表达式，不能绑定到函数调用结果或运行时计算出的值

常量在其声明的作用域内一直有效。



#### 隐藏 Shadowing

新声明的同名变量将会隐藏之前的变量

```rust
let x = 5;
let x = "   ";	// 之前的 x 被隐藏
```

这种用法的一种用途是解析字符串

```rust
let s = "   ";
let s = s.len();
```

这样可以省去多余的变量命名。



### 数据类型

Rust 是静态编译语言，在编译时必须知道所有变量的类型。基于使用的值，编译器通常能够推断出变量的类型；但是如果可能的类型较多，例如在解析字符串为整数时，就必须添加类型标注，否则编译器会报错。

```rust
let guess: u32 = "42".parse().expect("Not a number");
```



#### 标量类型

标量类型包括

- 整数类型，命名方式直接说明它的位数（`isize,usize` 的位数由计算机位数决定）
	- 有符号 `i8,i16,i32,i64,i128,isize`
	- 无符号 `u8,u16,u32,u64,u128,usize`
	整数默认为 `i32` 类型；整数字面值：十进制 `98_222`，十六进制 `0xff`，八进制 `0o77`，二进制 `0b111`，字节（Byte）`b'A'`；除了字节以外，允许直接在字面值后添加后缀声明类型 `57u8`
- 浮点类型 `f32,f64` 分别对应单精度和双精度，默认为双精度。现代 CPU 下双精度与单精度的开销已经相差无几
- 布尔类型占用 1 字节
- 字符类型 `char` 占用 4 字节，是 Unicode 标量值，甚至可以表示 Emoji 表情

```rust
let x: i8 = 6;
let y: i32 = 0.16;
let z: bool = true;
let w: char = 'K';
```



#### 复合类型

Rust 提供元组和数组两种复合类型，是能够将多个不同元素存放在一起的类型。



###### 元组

元组 Tuple 可以将多个类型的值放在一个变量中，长度固定。使用 `()` 创建元组

```rust
let tup: (i32, i64, u8) = (500, 6.4, 1);
```

其中每个元素值的类型在可推导的情况下可省略。



通过索引值或者模式匹配直接获得元组中的值

```rust
let (x, y, z) = tup;	// x,y,z 依次获得元组中的值
let x = tup.0;
let y = tup.1;
let z = tup.2;
```



###### 数组

数组中的元素类型相同，长度固定。使用 `[]` 创建数组

```rust
let a: [i32; 5] = [1,2,3,4,5];
```

其中 `[]` 指定类型和长度。如果每个元素值都相同，可以使用

```rust
let a = [3; 5];	// 等价于 let a = [3,3,3,3,3];
```

访问数组元素的方式与其它语言类似。如果显式访问越界，则编译器会报错

```rust
fn main() {
    let x = [1, 2, 3];
    println!("{} {} {} {}", x[0], x[1], x[2], x[3]);
}
```



### 控制流

#### 分支

使用 `if` 表达式创建分支，表达式必须是布尔类型

```rust
let number = 10;
if number > 5 {
	println!("{} is greater than 5", number);
} else if number > 2 {
	println!("{} is greater than 2", number);
} else {
	println!("{} is less than or equal to 2", number);
}
```

由于 `if` 是表达式，因此可以放在等号右侧作为值

```rust
let condition = true;
let number = if condition { 10 } else { 5 };
```

右侧表达式返回的类型必须相同。



#### 循环

###### Loop

使用 `loop` 表达式无限循环

```rust
loop {
    println!("Hello, world!");
    break;
}
```

由于 `loop` 是表达式，因此也可以放在等号右侧作为值

```rust
let mut counter = 0;

let result = loop {
	counter += 1;

	if counter == 10 {
		// 通过 break 返回值
		break counter * 2;
	}
};

println!("The result is: {}", result);
```



###### While

使用 `while` 表达式在满足条件时循环

```rust
let mut number = 3;

while number != 0 {
	println!("{}!", number);
	number -= 1;
}
```



###### For

使用 `for` 循环遍历集合

```rust
let a = [1,2,3,4,5];
for i in a.iter() {
    println!("{}", i);
}
```

使用 `(..)` 来生成范围数据，使用 `rev()` 反转范围

```rust
for number in (1..10).rev() {
    println!("{}!", number);
}
```



### 函数

#### 命名规范

强制使用 snake 风格命名函数

```rust
fn main() {
	println!("another_function!");	
}

fn another_function() {
	println!("another_function!");
}
```

允许函数在定义前调用。



#### 返回值

函数参数必须指定类型和返回类型

```rust
fn main() {
    let x = add(1, 2);
    println!("{}", x);
}

fn add(a: i32, b: i32) -> i32 {
    return a + b;
}
```

返回值可以通过默认的最后一个表达式直接返回

```rust
fn add(a: i32, b: i32) -> i32 {
	// 表达式直接作为返回值
    a + b
}
```

这里 `{}` 中的内容作为一个块，其最后一个表达式是块的返回值。例如

```rust
let y = {
	let x = 1;
	x + 3
};
```

这里 `x+3` 作为块的返回值传递给 `y`；如果没有表达式，则返回一个空的元组

```rust
let y = {
	let x = 1;
	x + 3;
};
// 等价于 let y = ();
```



### 所有权

#### 所有权规则

1. 每个值由一个变量所有
2. 每个值只能有一个所有者
3. 当所有者超出作用域时，改值被删除

以字符串类型为例

```rust
let mut s = String::from("Hello, world!");
s.push_str(", Rust!");
println!("{}", s);
```

首先从字面值创建字符串类型，然后向后面添加新的字符串。字符串字面值被硬编码到可执行文件中，因此不可变；而字符串类型在堆上分配内存，这一步通过 `String::from` 实现。

> 当变量离开作用域时，会调用 `drop` 函数将内存交还。



#### 移动

简单的基础数据类型在栈中保存，因此直接复制

```rust
let x = 5;
let y = x;
```

这两个 `5` 都进入栈。而复杂数据类型则使用**移动**数据

```rust
let s1 = String::from("hello");
let s2 = s1;
```

这里 Rust 将 `s1` 的值移动给 `s2`，导致 `s1` 失效。

> Rust 不会自动创建数据的深拷贝，因此所有自动赋值操作都是廉价的。



#### 克隆

如果想要对堆上的数据进行深拷贝，需要显式调用 `clone` 方法

```rust
let s1 = String::from("hello");
let s2 = s1.clone();
```



#### 复制

基本数据类型存放在栈中，因此不需要执行克隆操作，而是自动执行 `copy` 方法

```rust
let x = 5;
let y = x;
```

如果一个类型实现了 `copy` 方法，则它将执行复制操作；如果它或者它的一部分实现了 `drop` 方法，则不允许再实现 `copy` 方法。



任何简单标量类型的组合类型都是 `copy` 的，任何需要分配内存或某种资源的都不是 `copy` 的。例如

```rust
let a: (i32, i32) = (1,2);
let b: (i32, String) = (3, "Hello".to_string());
```

其中 `a` 是 `copy` 的，但 `b` 不是 `copy` 的。



#### 函数参数

函数参数的传值过程也是所有权的转移

```rust
fn main() {
    let s = String::from("Hello, world!");
    take_ownership(s);
    // println!("s: {}", s);
    // 报错，s 的所有权已经转移到函数参数中

    let x = 5;
    make_copy(x);
    println!("x: {}", x);
}

fn take_ownership(s: String) {
    println!("{}", s);
}

fn make_copy(n: i32) {
    println!("{}", n);
}

```

具有移动特性的变量所有权将会随着函数调用被转移到函数参数中，从而失去所有权。



#### 返回值

返回值也是所有权的转移

```rust
fn main() {
    let s = give_ownership();
    println!("{}", s);
    let s2 = String::from("Hello, world!");

	// 此时 s2 已经失去所有权
    let s3 = take_and_return_ownership(s2);
    println!("{}", s3);
}

// 所有权从函数返回
fn give_ownership() -> String {
    let s = String::from("Hello, world!");
    s
}

// 所有权传入函数再返回
fn take_and_return_ownership(s: String) -> String {
    s
}
```



如果在防止所有权转移的情况下使用值，可以将其传入后再返回

```rust
fn main() {
    let s = "Hello, world!".to_string();
    let (s, length) = calculate_length(s);
    println!("The length of '{}' is {}.", s, length);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len();
    (s, length)
}
```

更合理的方式是使用引用。



#### 引用

使用 `&` 表示引用值而不转移所有权

```rust
fn main() {
    let s = "Hello, world!".to_string();
    let length = calculate_length(&s);
    println!("The length of '{}' is {}.", s, length);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

把引用作为参数行为称为借用。



借用的变量不能修改，如果需要修改借用值，需要添加 `mut` 关键字

```rust
fn main() {
    let mut s = "Hello, world!".to_string();
    let length = calculate_length(&mut s);
    println!("The length of '{}' is {}.", s, length);
}

fn calculate_length(s: &mut String) -> usize {
    s.push_str(" world!");
    s.len()
}
```

可变引用所引用的变量必须也是可变的。在特定的作用域内，对某一块数据，只能有一个可变引用

```rust
let mut s = String::from("hello");
let s1 = &mut s;
// let s2 = &mut s;
// println!("s1: {}, s2: {}", s1, s2);
```

这样的好处在于防止编译时数据竞争

* 两个或多个指针同时访问同一个数据
* 至少有一个指针用于写入数据
* 没有使用任何机制来同步对数据的访问

通过创建新的作用域，可以非同时创建多个可变引用

```rust
let mut s = String::from("hello");
let s1 = &mut s;
{
	let s2 = &mut s;
}
```

不能同时拥有一个可变引用和一个不变引用

```rust
let mut s = String::from("hello");
let s1 = &s;
// let s2 = &mut s;
// println!("s1: {}, s2: {}", s1, s2);
```

但是可以创建多个不变引用

```rust
let mut s = String::from("hello");
let s1 = &s;
let s2 = &s;
println!("s1: {}, s2: {}", s1, s2);
```



#### 悬空引用

悬空指针是指一个指针引用了内存中的某个地址，而这块内存可能已经释放或分配给其它用途。在 Rust 中，编译器确保所有引用不可能是悬空引用——如果引用了某些数据，编译器保证引用离开作用域前，数据不会离开作用域。例如

```rust
fn main() {
    let r = dangle();
}

// 返回一个作用域内数据的引用，这不被允许
fn dangle() -> &String {
    let s = "Hello, world!";
    &s
}
```

这段代码将在编译时报错。



#### 切片

切片是一种不持有所有权的数据类型。在不使用切片时，字符串的元素索引与字符串本身缺少关联，例如

```rust
fn main() {
    let mut s = String::from("hello world");
    let word_len = first_word(&s);
    s.clear();
    println!("The first word is {} characters long", word_len);
}

fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

	// 遍历字符串中的每个字节，比较它是否为空格
    for (i, &byte) in bytes.iter().enumerate() {
        if byte == b' ' {
            return i;
        }
    }
    s.len()
}
```

当字符串被销毁时，对应查找到的索引应该无效。



使用切片获得字符串的一部分

```rust
let s = String::from("hello world");
let hello = &s[0..5];
let world = &s[6..11];
```

字符串切片时指向字符串中一部分内容的引用。



利用切片将前面的函数修改为

```rust
fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s);
    // s.clear();
    println!("The first word is {} characters long", word);
}

fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &byte) in bytes.iter().enumerate() {
        if byte == b' '{
            return &s[0..i];
        }
    }
    &s[..]
}
```

由于函数返回字符串一部分的不可变引用保存在 `word` 中，因此无法将其销毁，因此 `s.clear()` 将会报错。

> 查看方法手册可以看到 `clear()` 对应的声明 `pub fn clear(&mut self)`，它需要将字符串自身作为可变引用传入，这与引用规则矛盾。

更好的写法是将字符串切片作为参数传递，从而可以同时接受 `String, &str` 类型的参数

```rust
fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s[..]);
    // s.clear();
    println!("The first word is {} characters long", word);
}

fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &byte) in bytes.iter().enumerate() {
        if byte == b' '{
            return &s[0..i];
        }
    }
    &s[..]
}
```

> 字符串字面值就是字符串切片，因此也可以作为参数传入。



### 结构体

#### 声明结构

使用 `struct` 声明结构体

```rust
struct User {
    name: String,
    age: u32,
}

fn main() {
    let user1 = User {
        name: String::from("John"),
        age: 30,
    };
}
```

使用 `mut` 声明可变的实例，此时实例中所有成员都是可变的。



#### 返回值

结构体可以直接用作返回值。当字段名与字段值对应变量名相同时，可以简写字段

```rust
fn build_user(name: &str, age: u32) -> User {
    User {
        name: String::from(name),
        // 简写 age: age
        age,
    }
}
```



#### 更新语法

当需要基于某个实例创建新实例，可以使用更新语法

```rust
let user1 = User {
    name: String::from("John"),
    age: 30,
};

let user2 = User {
    name: String::from("Jane"),
    // 其余成员使用 user1 中的成员值
    ..user1
};
```



#### 元组结构

可以声明元组结构，用于给元组起名。例如

```rust
// 声明两个不同的元组结构
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

这里 `Color, Point` 将是两个不同的元组结构类型。



#### 空结构

没有任何字段的结构体（Unit-Like Struct）用于需要在某个类型上实现某个 `trait`，但是希望内部没有存储数据，类似于接口函数。



#### 所有权

结构体实例拥有其所有数据的所有权，只要该实例有效，则其中的字段数据也有效。结构体内部可以存放引用，但需要指定生命周期。



#### 调试输出

基本数据类型都可以通过 `{}` 输出，这是因为它们实现了 `std::fmt::Display` 接口；而 `Debug` 实现了 `std::fmt::Debug` 接口，只需要让结构体继承 `Debug` 即可使用对应的调试输出

```rust
// 使用 derive 宏来自动生成 Debug trait
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect = Rectangle { width: 50, height: 30 };
    // {:?} 用于格式化输出 Debug trait 的内容
    println!("Rectangle is {:?}", rect);
    // {:#?} 用于美化格式化输出 Debug trait 的内容
    println!("Rectangle is {:#?}", rect);
    println!("Area is {}", area(&rect));
}

fn area(rect: &Rectangle) -> u32 {
    rect.width * rect.height
}
```



#### 方法

通过 `impl` 声明的命名空间将结构体与其方法绑定在一起。方法以 `self` 作为第一个参数，是实例化本身

```rust
// 使用 derive 宏来自动生成 Debug trait
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // 定义 Rectangle 结构体的实例方法 area
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // 定义 Rectangle 结构体的实例方法 can_hold
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 50, height: 30 };
    let rect2 = Rectangle { width: 40, height: 20 };
    println!("Area is {}", rect1.area());
    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
}
```

也可以定义不以 `self` 作为第一个参数的关联函数，通常用于构造器

```rust
impl Rectangle {
    // 定义一个类方法 square 来创建正方形矩形
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
}

fn main() {
	// 使用 :: 调用关联函数
    let square = Rectangle::square(10);
}
```

可以将方法和关联函数放在多个 `impl` 命名空间中。



### 枚举

#### 定义枚举

枚举定义格式为

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

使用方法与 C++ 类似

```rust
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```



#### 附加数据

Rust 允许为枚举附加数据

```rust
enum IpAddrKind {
    V4(u8, u8, u8, u8),
    V6(String),
}
```

创建变量时为枚举提供对应数据

```rust
let four = IpAddrKind::V4(127, 0, 0, 1);
let six = IpAddrKind::V6(String::from("::1"));
```

> 标准库中定义了 `IpAddr` 枚举。



#### 定义方法

枚举还可以实现方法

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },		// 附加结构体
    Write(String),					// 附加字符串
    ChangeColor(i32, i32, i32),		// 附加数组
}

impl Message {
    fn call(&self) {}
}

fn main() {
    let q = Message::Quit;
    let m = Message::Move { x: 10, y: 20 };
    let w = Message::Write(String::from("hello"));
    let c = Message::ChangeColor(100, 200, 255);

    m.call();
}
```



#### Option 枚举

标准库中定义了类似 `Null` 概念的枚举

```rust
enum Option<T> {
	Some(T),
	None
}
```

它包含在 Prelude 预导入模块，可以直接使用。



利用 `Some, None` 保存包括 `Null` 在内的数据

```rust
let sn = Some(5);
let ss = Some("Hello, world!");
let sn_none: Option<i32> = None;
```

由于 `None` 无法推导类型，因此需要显式指定。



使用 `Option<T>` 的好处在于不可以将 `Option<T>` 直接作为 `T` 使用。例如

```rust
let x: i8 = 5;
let y: Option<i8> = Some(5);

// 将会报错
let sum = x + y;
```

这就能够避免不应该是空值但实际上是空值的值参与运算。



### Match

Rust 提供了非常强大的控制流方法

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

通过 `=>` 返回其后的表达式作为 `match` 的返回值，同时作为函数的最后一个表达式，成为函数的返回值。



#### 附加数据

由于枚举可以附加数据，因此可以在匹配同时获得枚举中的数据

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            // 获得枚举保存的值 state
            println!("State quarter from {:?}", state);
            25
        }
    }
}

fn main() {
    let coin = Coin::Quarter(UsState::Alaska);
    let value = value_in_cents(coin);
    println!("Value of coin: {} cents", value);
}
```



前面使用过的 `Option` 枚举也可以用来匹配

```rust
fn main() {
    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
    println!("{:?}", six);
}

fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(n) => Some(n + 1),
        None => None,
    }
}
```



#### 通配符

使用 `match` 匹配必须穷举所有的可能，否则编译器会报错。如果可能性过多，可以使用 `_` 通配符匹配所有未列出的值。例如

```rust
fn main() {
    let v = 0u8;
    match v {
        0 => println!("v is 0"),
        1 => println!("v is 1"),
        _ => println!("v is something else"),
    }
}
```



#### if let

只匹配一种情况时，写法为

```rust
let v = 0u8;
match v {
    0 => println!("v is 0"),
    _ => println!("others"),
}
```

这时可以使用 `if let` 语法糖

```rust
if let Some(0) = v {
	println!("v is 0");
} else {
	println!("others");
}
```



## 模块与包

### 代码组织

代码组织包括公共和私有的细节、作用域内有效的名称等。Rust 的模块系统构成为：

* Package（包）：Cargo 的特性，用于构建、测试、共享 crate
* Crate（单元包）：一个模块树，可以产生一个 library 或可执行文件
* Module（模块）/use：控制代码的组织、作用域、私有路径
* Path（路径）：为 struct/function/module 命名的方式


#### Crate

Crate 将相关功能组合到一个作用域内，便于在项目间进行共享，防止冲突。Crate 包括 

- binary
- library

Crate Root 是源代码文件，Rust 编译器从这里开始，它是 Crate 的根模块。



#### Package
一个 Package 包含

- 一个 `Cargo.toml` 文件，描述如何构建这些 Crates
- 0-1 个 library crate
- 任意数量的 binary crate
- 至少一个 crate（library 或 binary）

通过 `cargo` 创建的就是项目或包，创建出的 `main.rs` 就是项目或包的入口文件，这是 Cargo 的默认设置。Cargo 将 crate root 文件交由 rustc 构建 binary 或 library 。

> 默认情况下
> 
> - `src/main.rs` 是 binary crate 的 crate root，这个 crate 的名称与 package 相同
> - `src/lib.rs` 是 package 包含的 library crate，这个 crate 的名称与 package 相同
> 
> 如果有一个 `main.rs` 就表示这个 package 包含一个 binary crate；如果有 `lib.rs` 就表示包含一个 library crate 。如果想要有多个 binary crate，则应当将源代码放在 `src/bin` 下，每个文件都是一个单独的 binary crate 。



#### Module

Module 用于控制作用域和私有性。在一个 crate 中，将代码分组，增加可读性，易于复用。使用 `mod` 关键字创建，例如在 `src/lib.rs` 创建

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}
```

文件 `src/main.rs, src/lib.rs` 就是 crate roots，其中 `src/lib.rs` 形成一个 crate 模块

```shell
crate
└─front_of_house
    ├─hosting
       ├─add_to_waitlist
```



### Path

要在 Rust 的模块中找到某个条目，需要使用路径。包括

- 绝对路径：从 crate root 开始
- 相对路径：从当前模块开始，使用 `self, super` 或当前模块标识符

路径至少由一个标识符组成。



#### 路径调用

例如在 `src/lib.rs` 中给出

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    crate::front_of_house::hosting::add_to_waitlist();	// 绝对路径写法
    front_of_house::hosting::add_to_waitlist();			// 相对路径写法
}
```

在 `eat_at_restaurant` 中给出两种路径调用方法。



使用 `super` 关键字进入父级目录

```rust
fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        super::serve_order();
    }
}
```



#### 私有边界

然而这段代码不能通过编译，因为默认情况下所有条目（模块、函数、方法、结构等）都是私有的。这就是私有边界

- 由模块定义私有边界
- 放在模块中的函数或结构都默认是私有的
- 父模块不能访问子模块的私有条目
- 子模块可以访问父模块的任意条目
- 同级条目可以互相访问

如果希望能够访问子模块条目，需要添加 `pub` 前缀

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}
```



#### 公有结构

如果将 `struct` 设为 `pub`，则它本身是公有的，但是其字段默认私有

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,      // 公有字段
        seasonal_fruit: String, // 私有字段
    }

    impl Breakfast {
        // 公有方法，创建 summer 实例
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    let mut meal = back_of_house::Breakfast::summer("wheat");
    // meal.toast 可以访问，但 meal.seasonal_fruit 不能访问
    println!("I'm eating {} toast with {}.", meal.toast, meal.seasonal_fruit);
}
```



#### 公有枚举

如果将 `enum` 设为 `pub`，则它的枚举值默认公有

```rust
mod back_of_house {
    pub enum Appetizer {
        Soup,  // 公有枚举
        Salad, // 公有枚举
    }
}
```



### 引入条目
#### use

使用 `use` 将路径引入当前作用域

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

// 通过绝对路径引入
use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
	// 直接访问 hosting
    hosting::add_to_waitlist();
    println!("Hello, world!");
}
```

引入的模块默认是私有的，即此模块成为当前作用域中的私有模块，不能被外部访问。如果希望引入的模块也能够被外部访问，需要添加 `pub` 关键字

```rust
pub use crate::front_of_house::hosting;
```



如果需要引入同一模块下的多个条目，例如

```rust
use std::cmp::Ordering;
use std::io;
use std::io::Write;
```

可以简写为

```rust
use std::{cmp, io::{self, Write}};
```



如果要引入所有条目，使用

```rust
use std::collections::*;
```

主要用于批量导入测试代码。



#### as

使用 `as` 为路径起别名

```rust
use std::io::Result as IoResult;
```



### 拆分模块

Cargo 提供将模块保持嵌套结构不变的拆分技术。例如 `src/lib.rs` 文件

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}
```

首先将 `front_of_house` 的内容拆分到 `src/front_of_house.rs` 中，在 `src/lib.rs` 中只保留声明

```rust
// src/front_of_house.rs
pub mod hosting {
    pub fn add_to_waitlist() {}
}

// src/lib.rs
mod front_of_house;	// 只保留模块声明
```

然后再将 `hosting` 的内容拆分出去：由于 `hosting` 是 `front_of_house` 的子模块，因此它被要求拆分到子目录中，即

```rust
// src/front_of_house/hosting.rs
pub fn add_to_waitlist() {}

// src/front_of_house.rs
pub mod hosting;

// src/lib.rs
mod front_of_house;	// 只保留模块声明
```




## 集合类型

### Vector

标准库提供 `Vec<T>` 类型，可在堆上储存连续的相同类型的值。例如

```rust
let v: Vec<i32> = Vec::new();	// 空 vector，需要指定元素类型
let v2 = vec![1, 2, 3];			// 使用 vec! 宏创建 vector
```



#### 添加元素

可以向其中添加元素

```rust
// 注意使用 mut
let mut v: Vec<i32> = Vec::new();
v.push(1);
```



#### 访问元素

可以使用索引访问元素

```rust
let v = vec![1, 2, 3];
let thrid = &v[2];
println!("The third element is {}", thrid);
```

如果索引超出范围，则程序将会 panic（恐慌）。



通过 `get()` 方法返回元素的 `Option` 枚举

```rust
let v = vec![1, 2, 3];
match v.get(2)
{
    Some(thrid) => println!("The third element is {}", thrid),
    None => println!("There is no third element"),
}
```

如果索引超出范围，则会进入 `None` 分支。



#### 所有权

引用 Vector 元素时依然需要考虑所有权规则

```rust
let mut v = vec![1, 2, 3];
let first = &v[0];
v.push(6);  // 将会报错，因为 v[0] 被借用为不可变的引用，而 push 方法需要自身的可变引用 push(&mut self, value: T)
println!("The first element is: {}", first);
```

注意如果去掉最后一行输出，则可以正常运行，因为 `first` 没有使用。

> 之所以 `push` 需要可变引用，是因为 Vector 在扩容时可能需要将原先的数据复制到新的空间中，导致原先数据的引用悬空。



#### 遍历

使用 `for` 循环遍历

```rust
fn main() {
    let mut v = vec![1, 2, 3];

    // 需要将 v 声明为 &mut 表示可以修改 v 的元素
    for i in &mut v {
        *i *= 2;
    }

    for i in &v {
        println!("{}", i);
    }
}
```



#### 枚举

由于枚举可以附加数据，因此可以向 Vector 中保存不同类型数据的枚举

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

fn main() {
    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Float(3.14),
        SpreadsheetCell::Text("Hello, world!".to_string()),
    ];
}
```



### String

#### 字符串

Rust 核心语言层面只有一个字符串类型：字符串切片 `str` 或 `&str`，它保存对其它位置 UTF-8 编码字符串的引用。

> 字符串字面值储存在二进制文件中，因此也是字符串切片。

String 类型来自标准库而非核心语言，它可增长、可修改、可拥有。



#### 创建

使用 `new` 创建或使用 `to_string` 创建

```rust
let mut s1 = String::new();
let mut s2 = "hello".to_string();
let s3 = String::from("hello");
```



#### 更新

使用 `push_str` 添加字符串切片，使用 `push` 添加单个字符

```rust
let mut s = String::from("Hello ");
s.push_str("World!");
println!("{}", s);

let s2 = String::from("World!");
s.push_str(&s2);
println!("{}", s);
```



使用 `+` 连接字符串

```rust
let s1 = String::from("Hello ");
let s2 = String::from("Hello ");
let s3 = s1 + &s2;
println!("{}", s1);	// 报错
println!("{}", s2);
println!("{}", s3);
```

由于 `+` 操作类似于

```rust
fn add(self, s: &str) -> String { ... }
```

因此 `s1` 在操作后**失去所有权**，`s2` 必须转换为字符串切片传入（解引用强制转换将 `&String` 变为 `&str`）。



使用 `format!` 格式化拼接

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

// let s3 = s1 + "-" + &s2 + "-" + &s3;
let s3 = format!("{}-{}-{}", s1, s2, s3);
println!("{}", s3);
```



#### 索引

String 不支持通过索引访问元素

```rust
let s = String::from("tic");
let c = s[0];	// 报错
```



#### 内部表示

String 是对 `Vec<u8>` 的包装，使用 `len()` 试图获得长度

```rust
let len = String::from("Hello, world!").len();
let len2 = String::from("こんにちは、世界！").len();
println!("The length of 'Hello, world!' is {}.", len);
println!("The length of 'こんにちは、世界！' is {}.", len2);
```

由于使用 UTF-8 编码，每个字符的长度不同，因此返回的字节数不等于字符数。



可以将 String 通过其它方式遍历

```rust
let message = "こんにちは、世界！";

// 依次输出每个字符
for c in message.chars() {
    println!("{}", c);
}

// 依次输出每个字节
for b in message.bytes() {
    println!("{}", b);
}
```



#### 切片

使用 `[0..4]` 创建切片

```rust
let s = "Hello, world!";
let cut = &s[0..5];
println!("{}", cut);
```

切片时必须沿着 UTF-8 编码字符的边界切割，否则程序将会 panic 。



### HashMap

哈希表以键值对形式储存数据，通过一个 Hash 函数决定如何在内存中存放。



#### 创建

创建空哈希表时需要显式指定数据类型

```rust
use std::collections::HashMap;

fn main() {
    let mut map : HashMap<&str, u8> = HashMap::new();
}
```

而如果插入数据，则会自动推断类型

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert("hello", 10);
}
```




可以将两个 Vector 组合为一个哈希表

```rust
use std::collections::HashMap;

fn main() {
    let teams = vec!["A", "B", "C", "D"];
    let scores = vec![10, 20, 30, 40];

    // 需要指明 collect() 返回的格式 HashMap<_, _>，我们不关心 key 和 value 的类型，因此用 _ 代替
    let scores_map: HashMap<_, _> = teams.iter().zip(scores.iter()).collect();
}
```



#### 所有权

如果将值插入到哈希表，则值的所有权转移到哈希表中

```rust
use std::collections::HashMap;

fn main() {
    let field_name = "field_name".to_string();
    let field_value = "field_value".to_string();
    let mut map = HashMap::new();
    map.insert(field_name, field_value);

    // 将会报错，因为 field_name 和 field_value 所有权被转移到 map 中
    println!("{}: {}", field_name, field_value);
}
```

如果插入引用则不会转移所有权

```rust
map.insert(&field_name, &field_value);
```



#### 访问元素

通过 `get()` 方法获得 `Option` 返回值

```rust
let score = map.get(&field_name);
match score {
	Some(value) => println!("{}: {}", field_name, value),
	None => println!("{} not found", field_name),
}
```



#### 遍历

使用 `(k, v)` 获得键值对

```rust
for (k, v) in &map {
    println!("{}: {}", k, v);
}
```



#### 更新

每个键只能对应一个值，如果插入同样键的键值对，则原先的键值对被覆盖

```rust
let name = "name".to_string();
let val = 10;
let mut map = HashMap::new();

map.insert(&name, val);
map.insert(&name, 15);
```

如果希望只有当不存在对应键时才插入键值对，使用 `entry().or_insert()` 方法

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();

    let e = map.entry(String::from("hello"));
    e.or_insert(15);

    map.entry(String::from("world")).or_insert(20);

    println!("{:?}", map);
}
```

如果键存在，则 `entry()` 返回对应值的可变引用，否则插入新值并返回可变引用。



基于键修改值

```rust
use std::collections::HashMap;

fn main() {
    let text = "hello world wonderful world";
    let mut map = HashMap::new();

	// 使用空格分割
    for word in text.split_whitespace() {
    	// 获得 count 为可变引用，需要解引用后修改
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{:?}", map);
}
```



#### Hash 函数

默认情况下使用加密功能强大的 Hash 函数，可以抵抗拒绝服务（DOS）工具，但这不是最快的 Hash 算法。可以指定不同 hasher 切换算法。



## 错误处理

### 概述

Rust 在大部分情况下会在编译时提示错误并给出错误信息。错误可分为

1. 可恢复：例如文件未找到，可再次尝试
2. 不可恢复：例如索引超出范围

Rust 没有异常机制

- 可恢复错误 `Result<T,E>`
- 不可恢复错误 `panic!` 宏



### 不可恢复错误

当 `panic!` 宏执行，程序将打印错误信息，展开 (unwind)、清理调用栈 (stack)，然后退出程序。默认情况下

- 程序展开 (unwind) 调用栈（工作量大）
	- Rust 沿着调用栈返回
	- 清理每个函数中的数据
- 或立即终止 (abort) 调用栈
	- 不清理，直接停止程序
	- 内存需要 OS 进行清理

如果希望二进制文件更小，可以在 `Cargo.toml` 中将“展开”改为“终止”。例如

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

[dependencies]
rand = "0.8.5"

[profile.release]
panic = "abort"
```



可以调用 `panic!` 宏手动报错

```rust
fn main() {
    panic!("Hello, world!");
}
```

在调用时指定 `RUST_BACKTRACE=1` 可以显示回溯信息

```shell
set RUST_BACKTRACE=1 && cargo run
```

通过回溯信息定位问题代码。



### 可恢复错误

#### Result

使用 `Result` 枚举处理可恢复错误，结构为

```rust
enum Result<T, E> {
	Ok(T),
	Err(E),
}
```

操作成功时返回 `Ok(T)`，可以获得其中对应的数据；失败时返回 `Err(E)`，获得错误信息。例如

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
}
```



#### 匹配错误

使用 `match` 表达式处理错误返回

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => println!("File opened successfully: {:?}", file),
        Err(error) => println!("Error opening file: {:?}", error),
    };
}
```

对于错误信息，也可以匹配不同的错误类型

```rust
let f = match f {
    Ok(file) => println!("File opened successfully: {:?}", file),
    Err(error) => match error.kind() {
        std::io::ErrorKind::NotFound => println!("File not found"),
        std::io::ErrorKind::PermissionDenied => println!("Permission denied"),
        _ => println!("An error occurred: {:?}", error),
    },
};
```

通过闭包 (closure) 简化代码，例如使用 `unwrap_or_else` 处理

```rust
let f = File::open("hello.txt").unwrap_or_else(|err| {
    if err.kind() == std::io::ErrorKind::NotFound {
        File::create("hello.txt").unwrap_or_else(|err| {
            panic!("Failed to create file: {}", err);
        })
    } else {
        panic!("Failed to open file: {}", err);
    }
});
```



#### 简化处理

使用 `unwarp` 简化 `match` 分支

```rust
let f = File::open("hello.txt");

let f = match f {
    Ok(file) => file,
    Err(error) => {
        println!("Error: {}", error);
    }
};

// 简化等价写法
let f = File::open("hello.txt").unwrap();
```

改为使用 `expect` 输出错误信息

```rust
let f = File::open("hello.txt").expect("Failed to open file");
```



#### 传播错误

当函数出错时，将错误返回给调用者，这就是传播错误。例如

```rust
fn read_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    // 打开文件时的错误处理
    let mut f = match f {
        Ok(file) => file,
        // 返回错误
        Err(error) => return Err(error),
    };

    // 写入字符串时的错误处理
    let mut s = String::new();
    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        // 返回错误
        Err(error) => Err(error),
    }
}
```

使用 `?` 运算符快速传播错误

```rust
fn read_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

其中 `?` 运算符表示如果执行成功，则正常执行；否则就将错误信息直接返回。

> 使用 `?` 时，其返回的错误类型将被隐式转换为函数的返回类型（不是任何时候都可以转换），因此只能用于返回 `Result` 的函数。

借助链式调用继续简化代码

```rust
fn read_file() -> Result<String, io::Error> {
    let mut s = String::new();
    let f = File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```



可以修改 `main` 函数的返回类型，从而可以在 `main` 中使用 `?` 运算符

```rust
use std::fs::File;
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let f = File::open("hello.txt")?;
    Ok(())
}
```

其中 `Box<dyn Error>` 是一个 trait 对象，可以理解为任意错误类型。



## 泛型

### 概述

泛型的目的是提高代码的复用能力，在编译时编译器将占位符替换为具体的类型，称为单态化。



### 泛型函数

将下面函数改写为泛型函数

```rust
fn largest(list: &[i32]) -> i32 {
    let mut largest = list[0];
    for &num in list {
        if num > largest {
            largest = num;
        }
    }
    largest
}

fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];
    for &num in list {
        // 这里会报错，因为没有指定实现 PartialOrd trait
        if num > largest {
            largest = num;
        }
    }
    largest
}
```

由于泛型类型不确定，因此需要指定比较 trait，这里将会报错。



### 泛型类型

#### 结构体

声明泛型结构体

```rust
struct Point<T> {
	x: T,
	y: T,
}
```



#### 枚举

声明泛型枚举

```rust
enum Result<T, E> {
	Ok(T),
	Err(E),
}
```



#### 方法

对于泛型结构体实现泛型方法

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }

    fn y(&self) -> &T {
        &self.y
    }
}
```

可以特化泛型方法

```rust
impl Point<f32> {
    fn distance(&self, other: &Point<f32>) -> f32 {
        ((self.x - other.x).powi(2) + (self.y - other.y).powi(2)).sqrt()
    }
}
```

只有 `Point<f32>` 实现此 `distance` 方法。



可以为方法指定新的泛型

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
	// 混合顶点坐标
    fn mixup<V, W>(&self, other: &Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}
```



### Trait

Trait 类似于概念，它告诉编译器泛型所具有的特性；Trait bounds（约束）指定泛型类型参数必须实现特定的行为。


#### 定义 Trait

使用 `trait` 关键字声明，它只有方法签名，没有具体实现。实现该 trait 的类型必须提供方法的具体实现。例如

```rust
pub struct NewsArticle {
    pub headline: String,
    pub author: String,
    pub content: String,
}

pub struct Tweet {
    pub username: String,
    pub content: String,
}

// 定义 Summary trait
pub trait Summary {
    fn summarize(&self) -> String;
}

// 为 NewsArticle 实现 Summary trait
impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {}: {}", self.headline, self.author, self.content)
    }
}

// 为 Tweet 实现 Summary trait
impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

将这些代码定义在 `src/lib.rs` 中，然后在 `src/main.rs` 中调用

```rust
use hello_cargo::NewsArticle;
use hello_cargo::Tweet;

// 引入 trait 才能使用
use hello_cargo::Summary;

fn main() {
    let article = NewsArticle {
        headline: String::from("Penguins win the Stanley Cup Championship"),
        author: String::from("The Associated Press"),
        content: String::from("The Penguins defeated the Sharks 4-0 in the Stanley Cup Championship."),
    };

    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("What a great game!"),
    };

    println!("New article available! {}", article.summarize());
    println!("Someone tweeted: {}", tweet.summarize());
}

```

其中 `hello_cargo` 是 `Cargo.toml` 中定义的 package 名

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

[dependencies]
rand = "0.8.5"

[profile.release]
panic = "abort"
```



#### 实现约束

只有当一个 trait 或类型在当前 crate 中定义时，才能够实现类型的 trait，因此不能为外部类型实现外部的 trait，从而确保安全性。

> 如果不加限制，则可以在两个 crate 里为同一个类型实现同一个 trait，从而导致歧义。



#### 默认实现

可以指定默认的实现

```rust
pub trait Summary {
	// 默认实现
    fn summarize(&self) -> String {
        String::from("This is the summary")
    }
}

// 空，使用默认实现
impl Summary for NewsArticle {
    // fn summarize(&self) -> String {
    //     format!("{}, by {}: {}", self.headline, self.author, self.content)
    // }
}
```

可以在 `trait` 的默认实现中调用未默认实现的 trait，例如

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
    	// 调用 summarize_author
        format!("This is the summary {}.", self.summarize_author())
    }
}
```



#### 参数约束

可以指定类型必须实现某个 trait 才能作为参数。例如

```rust
// 指定 item 类型必须实现 Summary
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

如果有多个需要约束的参数，使用 `trait bound` 语法

```rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {}
```

多个约束通过 `+` 连接

```rust
pub fn notify<T: Summary + Display>(item1: &T, item2: &T) {}
```

当存在多个约束和参数时，可以使用 `where` 子句使函数签名更清晰

```rust
pub fn notify<T, U>(a: &T, b: &U)
where
    T: Summary + Display,
    U: Clone + Debug,
{
}
```



#### 返回约束

可以要求返回类型必须实现某个 trait

```rust
pub fn notify() -> impl Summary {
    NewsArticle {
        headline: String::from("Penguins win the Stanley Cup Championship"),
        author: String::from("Bruce Wayne"),
        content: String::from("The Penguins defeated the Blues 4-0 in Stanley Cup quarterfinals."),
    }
}
```

注意返回类型必须完全确定，不能返回多个不同的类型，例如

```rust
pub fn notify(flag: bool) -> impl Summary {
    if flag {
        NewsArticle {
            headline: String::from("Penguins win the Stanley Cup Championship"),
            author: String::from("Bruce Wayne"),
            content: String::from("The Penguins defeated the Blues 4-0 in Stanley Cup quarterfinals."),
        }
    } else {
    	// 另一个返回类型，将会报错
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from("I am writing Rust."),
        }
    }
}
```



#### 泛型约束

使用泛型时，如果执行某些 trait，就必须指定它实现了对应的 trait

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
	// 执行 copy，必须实现 Copy trait
    let mut largest = list[0];

    for &item in list {
    	// 执行比较，必须实现 PartialOrd trait
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let numbers = [1, 2, 3, 4, 5];
    let result = largest(&numbers);
    println!("The largest number is {}", result);
}
```

如果传入字符串数组，则要对应修改

```rust
fn largest<T: PartialOrd + Clone>(list: &[T]) -> T {
    // 需要克隆，指定 Clone trait
    let mut largest = list[0].clone();

    // 需要引用，因此去掉 &item
    for item in list {
        // 执行比较，指定 PartialOrd trait
        if item > &largest {
            largest = item.clone();
        }
    }
    largest
}

fn main() {
    let str_list = [String::from("hello"), String::from("world"), String::from("rust")];
    let result = largest(&str_list);
    println!("The largest number is {}", result);
}
```



#### 实现特定方法

使用泛型时，可以指定当 `T` 实现了某个 trait 时，才实现某个方法

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    // 为所有 T 类型实现构造函数
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

// 只有当 T 实现了 Display trait 时，才会实现 fmt 方法
impl<T: Display> Display for Pair<T> {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```



#### 实现特定 trait

可以指定一个 trait 只有在另一个 trait 实现后才能够实现

```rust
// 只有当 T 实现了 Display trait 时，才为 T 实现 ToString trait
impl<T: Display> ToString for T {
    // ...
}
```

例如整型实现了 `Display trait`，因此实现了 `ToString trait`，可以转换为字符串

```rust
let s = 3.to_string();
```



## 生命周期

### 标注生命周期

Rust 的每个引用都有自己的生命周期，生命周期是引用保持有效的作用域。大多数情况下，生命周期是隐式、可推断的。然而，如果引用的生命周期可能以不同的方式互相关联时，需要手动标注生命周期。



如果函数返回类型的生命周期与传入参数的生命周期相关，则可能需要显式声明生命周期。例如

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

编译器将会报错，因为不能确定返回类型的生命周期与 `x` 还是 `y` 相关。此时需要显式声明

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

使用 `'a` 表示传入参数和返回值类型的生命周期一致。



### 标注语法

生命周期标注不会改变生命周期长度。生命周期参数名

- 以 `'` 开头
- 通常全小写

标注位置

- 在引用的 `&` 符号后
- 使用空格将标注和引用类型分开

单个生命周期标注没有意义，例如

```rust
fn longest(x: &str) -> &str {
	x
}
```

不需要显式指定生命周期。



#### 多参数

对于多个参数，返回值的生命周期是其中最短的一个

```rust
fn main() {
    let s1 = String::from("hello");
    let result;
    {
        let s2 = String::from("world");
        // 返回值 result 的生命周期是 s1,s2 中较短的一个，即 s2
        result = longest(&s1, &s2);
    }
    // 由于 result 生命周期与 s2 相同，因此此时 result 失效，将会报错
    println!("The longest string is: {}", result);
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```



#### 返回值

返回值的生命周期标注只考虑与返回值相关的参数，因此

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

其中 `y` 的生命周期标注可以省去。



如果返回值与任何传入参数无关，则函数不能返回引用。例如

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    let x = String::from("hello");
    x.as_str()
}
```

可以改为返回值，此时内部变量 `x` 的所有权转移到函数外部

```rust
fn longest<'a>(x: &'a str, y: &str) -> String {
    let x = String::from("hello");
    x
}
```



#### Struct

Struct 中的引用类型必须添加生命周期标注。例如

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}
```

要求字段的生命周期不小于结构体实例的生命周期。



### 省略规则

在 Rust 引用分析中编入的模式称为生命周期省略规则。在没有显式声明生命周期的情况下，编译器将尝试

1. 为每个引用类型输入参数分配一个生命周期
2. 如果只有一个输入生命周期，则将这个生命周期作为输出生命周期
3. 如果有多个输入生命周期，其中有一个是 `&self` 或 `&mut self`，则将 `self` 的生命周期作为输出生命周期

如果应用上述规则后，无法推断出所有引用生命周期，则编译器将会报错。



#### 函数参数

例如考虑下面函数

```rust
fn first(s: &str) -> &str;
```

按照省略规则，首先为输入参数指定生命周期

```rust
fn first(s: &'a str) -> &str;
```

由于只有一个输入生命周期，将其作为输出生命周期

```rust
fn first(s: &'a str) -> &'a str;
```

此时所有引用生命周期已经明确，可以通过编译。



再给出一个反例

```rust
fn first(x: &str, y: &str) -> &str;
fn first(x: &'a str, y: &'b str) -> &str;
```

到这一步无法确定返回引用的生命周期，编译器报错。



#### 结构方法

由于省略规则存在，通常结构体的方法中的生命周期可以省略。例如

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl <'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }

    fn announce(&self, announcement: &str) -> &str {
        println!("{}", announcement);
        self.part
    }
}
```

其中两个方法中引用的生命周期都可以自动推导。



### 静态生命周期

使用 `'static` 指定全局生命周期，所有字符串字面值都拥有此生命周期。例如

```rust
let s: &'static str = "Hello";
```



## 编写测试

