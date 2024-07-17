---
layout: post
title: "Rust 语言学习笔记"
date: 2023-02-28 22:00:00
comments: true
categories: 
- Document
tags:
- Rust
- 基础知识
- Programming Language
- Trend
---

文档地址：[https://kaisery.github.io/trpl-zh-cn/](https://kaisery.github.io/trpl-zh-cn/)

Rust 是一种 **预编译静态类型**（*ahead-of-time compiled*）语言，这意味着你可以编译程序，将编译好的程序直接运行而不需要任何环境配置。

### Hello World!

main 函数是一个特殊的函数，它总是最先运行的代码。这里有一些重要细节：

- 缩进风格使用 4 个空格；字符串以双引号包裹，语句以分号结尾；注释使用双斜杠。
- Rust 宏和 Rust 函数不一样，调用宏时需要在调用方法后面加叹号 !
- Rust 语言编译和运行隔离。首先执行 `rustc mian.rs` 进行编译，然后运行编译后的 main 文件

<!-- more -->

### Hello Cargo!

Cargo 是 Rust 的构建系统和包管理器，它可以提供构建代码、下载依赖库并进行编译等功能。 

使用 `cargo new project_name` 创建新的项目。创建出的目录中，src 目录下有 [main.rs](http://main.rs) 文件。还有一个 Cargo.toml 文件（Tom's Obvious, Minimal Language，TOML 格式，是 Cargo 配置文件格式）。

- [package] 片段标题，表明下面的语句
- [dependencies] 片段标题，是项目依赖片段的开始，这些代码包被称为 crates

使用 `cargo build` 进行构建，使用 `cargo run` 直接运行，使用 `cargo check` 检查代码有无编译错误。

当项目最终准备好发布时，可以使用 `cargo build --release`来优化编译项目。这会在 target/release **而不是 target/debug **下生成可执行文件。这些优化可以让 Rust 代码运行的更快，不过启用这些优化也需要消耗更长的编译时间。

🎖️ 在 Rust 中，`String::new()` 就类似 `String.new()` ，是类型的静态方法，而 new 也是创建类型实例的关键字。

🎖️ & 可以表示某个参数是一个引用，类似 let 声明变量，如果需要可变，则需要写成 `&mut param`

🎖️ Rust 中结构体无法直接使用 println! 宏来打印，需要在 println! 中使用 “{:?}” 或 ”{:#?}“，并在结构体之前加上外部属性 `#[derive(Debug)]`

🎖️ 不能在相同作用域中同时存在可变和不可变引用，会导致变量的值不同步的情况

🎖️ main 函数返回值为 ()

## 常见编程概念

### 变量与可变性

使用 let 关键字声明变量并用等号进行赋值；**变量默认不可变**，如果需要一个变量可以重复被赋值，可以在 let 关键字后添加 mut 关键字。

使用 const 关键字声明常量，且不允许使用 mut 关键字。

可以重新定义相同变量名的变量，这被称为“隐藏”，实质就是重新创建了一个变量。

### 数据类型

Rust 是静态类型语言，意味着编译时就必须知道所有变量类型。

**标量类型**

- 整型
    - 有符号以 i 开头，无符号以 u 开头
    - 分别有 i8、u8、i16、u16、i32、u32、i64、u64、i128、u128、isize、usize；数字表示 2 的 n 次方（例如 i8 的数字范围是 -(2^7) 到 （2^7) - 1；isize 指的是计算机的位数（64 还是 32）
    - 十进制（98_222，_ 表示分隔符易于阅读），十六进制（0xff），八进制（0o77），二进制（0b1111_0000），Byte（单字节字符，b’A’）
    - 当整型溢出时，在 debug 模式下会 panic，在 release 模式下会循环绕回到补码值
- 浮点型：分为 f32（单精度浮点数） 和 f64（双精度浮点数） 两种位数
- 布尔类型：分为 true 和 false 两个值
- 字符类型：char 值是单个字符，用**单引号**声明；字符串字面量用双引号**声明**

**复合类型**

可以将多个值组合成一个类型，Rust 有两个原生的复合类型：元组（tuple）和数组（array）。**元组和数组在创建时就需要知道数量大小，所以是存储在栈中的**。

- 元祖类型（tuple)
    - 声明元组：使用 `let tup: (i32, f64, u8) = (500, 6.4, 1);` 声明元组，不包含任何值的元组称作单元元组：()
    - 支持解构赋值：`let (x, y, z) = tup;`
    - 支持索引访问：`let a = tup.0;`
- 数组类型（array）
    - 声明数组：`let a: [i32; 5] = [1, 2, 3, 4, 5];`
    - 访问数组元素： `let first = a[0];`
    - 如果访问无效索引，程序会直接退出

### 函数

fn 关键字用来声明新函数，函数名使用下划线 snake_case 规范。函数的参数声明**必须有类型**。如果函数需要返回值，需要使用 return（**如果不使用 return 关键字，则返回值后不能加分号！**）。

### 控制流

**if 表达式**

```rust
if number % 4 == 0 {
    println!("number is divisible by 4");
} else if number % 2 == 0 {
    println!("number is divisible by 2");
} else {
    println!("number is not divisible by 4, 3, or 2");
}

// 类似 js 中的 ? : 表达式
let number = if condition { 5 } else { 6 };
```

if…else… 语句的条件必须是布尔类型的值，如果是其他类型会直接报错！

**循环控制流**

loop 关键字可以声明循环语句，break 关键字可以终止循环，break 之后可以声明循环返回的值。

```rust
let mut counter = 0;

let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;
    }
};
```

如果有多层嵌套的 loop 循环语句，可以使用循环标签来指定 break 结束哪个循环：

```rust
'counting_up: loop {
    println!("count = {count}");
    let mut remaining = 10;

    loop {
        println!("remaining = {remaining}");
        if remaining == 9 {
            break;
        }
        if count == 2 {
            break 'counting_up;
        }
        remaining -= 1;
    }

    count += 1;
}
```

while 条件循环

```rust
let mut number = 3;
while number != 0 {
    println!("{number}!");
    number -= 1;
}
```

for…in… 循环

```rust
let a = [10, 20, 30, 40, 50];
for element in a {
    println!("the value is: {element}");
}
```

## 所有权

所有程序都必须管理其**运行时使用计算机内存的方式**。一些语言中具有**垃圾回收机制**；另一些语言中须**亲自分配和释放内存**。

Rust 则通过**所有权系统**管理内存，编译器在编译时会根据一系列的规则进行检查。如果违反了任何这些规则，程序都不能编译。在运行时，所有权系统的任何功能都不会减慢程序。

Rust 中，值是位于栈上还是堆上在更大程度上影响了语言的行为以及为何必须做出这样的抉择。栈和堆都是代码在运行时可供使用的内存，但是它们的结构不同。

- 栈：编译时，栈中的所有数据都**必须占用已知且固定的大小**，操作有入栈和出栈。
- 堆：编译时，**大小未知或大小可能变化**的数据存在堆中，内存分配器会找一块足够大的空位进行**分配**

入栈比堆分配速度快，因为堆分配需要**搜索内存空间并进行计算**。访问栈数据比访问对数据快，因为堆数据必须通过指针访问（处理器在内存中跳转越少越快）。

所有权系统就是:

- 跟踪哪部分代码正在使用堆上的哪些数据
- 最大限度的减少堆上的重复数据的数量
- 以及清理堆上不再使用的数据确保不会耗尽空间

### 所有权规则

1. Rust 中的每一个值都有一个 **所有者**（变量，*owner*）。
2. 值在任一时刻有且只有一个所有者。
3. 当所有者离开**作用域**，这个值将被丢弃。

### 变量作用域

作用域是一个项（item）在程序中有效的范围。在 Rust 中，一般是花括号开始和结束。

### String 类型

字符串字面值（直接用双引号包括并赋值给变量的值），编译时知道其内容，会被硬编码进最终可执行文件。而 String 类型是为了支持可变的文本片段，所以需要一块未知大小的内存来存放。所以

- 必须在运行时向内存分配器（memory allocator）请求内存：`String::from()`
- 需要一个当我们处理完 `String` 时将内存返回给分配器的方法：当一个 String 类型值离开作用域时，就自动释放其内存（在 } 处自动调用 drop 函数来释放内存）

### 变量与数据交互的方式

- 移动

```rust
let x = 5;
let y = x;
```

如果是已知大小的值，类似上述代码中，5 被绑定到 x，然后 x 的拷贝绑定到 y，相当于是两个 5 值被放入了栈中。

```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{},world!", s1) // error!
```

如果是 String 类型（未知大小）时，s1 是由指针 ptr、长度 len 和 容量 capacity（从分配器获取了多少字节的内存）三个数据构成，其中指针 ptr 指向堆上存在的内容。当 s1 被赋值给 s2，实际上是复制了一份 { ptr, len, capacity } 到 s2，且 ptr 指向堆上相同的内容。

Rust 认为这时的 s1 不再有效，因此 s1 离开作用域不会进行任何清理操作。也就是说，上述第二段打印代码无法正常编译。所以这样的赋值被称为**移动**，很容易理解。Rust 永远不会自动创建数据的”深拷贝“。

- 克隆

如果想要在赋值时，同时将堆上的变量复制一份，需要进行克隆。而存在栈内的数据只会有克隆类型的交互（所有整型、布尔类型、浮点类型、字符类型和元组）。

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2); // success!
```

### 所有权与函数

向函数传递值也可能会移动或者复制，简单类型是直接复制栈内容传给函数，复杂类型是直接新建引用传给函数并释放旧的引用，使用 clone() 方法则是复制堆数据。

```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域
    takes_ownership(s);             // s 的值赋值到函数里，所以到这里不再有效，之后再访问 s 将会报错
    let x = 5;                      // x 进入作用域
    makes_copy(x);                  // x 应该移动函数里，但 i32 是 copy 的，所以在后面可继续使用 x
}
// 这里，x 先移出了作用域，然后是 s。但因为 s 的值已被移走，没有特殊之处

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} 
// 这里，some_string 移出作用域并调用 `drop` 方法，占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} 
// 这里，some_integer 移出作用域。没有特殊之处
```

函数的返回值也能够转移所有权，类似上面的转移方式。简单类型总是复制栈内容，复杂类型值总是会在不同变量不同作用域之间转移。如果想避免这种形式主义，可以使用 Rust 提供的**引用**。

### 引用

引用允许使用堆上的值，而不进行值的转移。

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

上面代码，将 s1 传递给函数时，前面加了 & 符号，就是将新建了一个 s1 堆内容的引用，也就是说 &s1 代表的是 s1 的指针 ptr。这样的话，s1 没有被转移，依旧可以在传参之后继续使用。需要注意的是，**函数的参数也需要加 & 符号**！

在函数中，s 在函数调用结束时也停止使用，但是因为 s 没有 s1 的所有权，所以并不会释放掉堆上的内容。默认引用是**不可以被修改**的。

**可变引用**：使用 &mut 关键词，函数的参数也要添加 &mut 进行修饰。

```rust
change(&mut s);

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

在**同一个作用域**下，针对同一个堆数据，**不能创建两个可变引用，也不可同时创建可变与不可变引用**！这是为了避免**数据竞争**，产生数据竞争的情况如下：

- 两个或更多指针同时访问同一数据。
- 至少有一个指针被用来写入数据。
- 没有同步数据访问的机制。

另外，Rust 不会造成**悬垂引用**（数据内存被释放单指针仍然存在，会造成空指针）的情况，Rust 在编译时会直接报错。

### Slice 引用

slice 允许你引用集合中一段连续的元素序列，而不用引用整个集合。slice 是一类引用，所以它没有所有权。

```rust
let s = String::from("hello");
let len = s.len();

let slice1 = &s[0..2];
let slice2 = &s[..2]; // 与上条语句等价
let slice3 = &s[3..len];
let slice4 = &s[..len]; // 与上条语句等价
let slice5 = &s[..]
```

在 IDE 中，可以看到代码提示 slice1 变量的类型是 &str，也就是说 slice 的类型就是一个字符串字面值，而字符串字面量也是 slice。

同时，数组的 slice 也同理。

## 结构体 struct

结构体类似于 JS 中的对象，使用 struct 关键字可以定义结构体的类型。使用结构体创建**实例**时，也是遵循默认不可变（不仅是结构体不可变，结构体中的字段也不可变），使用 mut 关键字使其成为可变（但是**不允许单个字段变成可变**）。

可变结构体中的字段可以使用点号进行赋值。

结构体实例化时，需要**结构体名+花括号**创建，可以直接使用变量作为字段简写：`User { name, email }`

```rust
struct User {
    active: bool,
    username: String,
    email: String,
}

fn main() {
    let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}
```

结构体中也可以使用 .. 进行解构，和 JS 不同的是，解构其余字段时是需要放在结构体最后的。注意！将 user1 解构赋值给 user2 时相当于使用 = 赋值，**字段值是复杂类型**时是**值的移动**，所以**解构赋值完 user1 就不再可用了**！ 如果结构体中只有简单类型值则没有问题。

```rust
let user2 = User {
    email: String::from("another@example.com"),
    ..user1
};
```

### 元组结构体

当我们不需要知道具体结构体中的字段名时（或其字段意义非常明显的），就可以使用元组结构体。需要注意的是，使用 struct 定义的每一个结构体都是单独的类型，尽管字段类型都相同也不能互相赋值！

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);
fn main() {
    let black = Color(0, 0, 0);
		let origin = Point(0, 0, 0);
}
```

### 类单元结构体

```rust
struct AlwaysEqual;
fn main() {
    let subject = AlwaysEqual;
}
```

### 方法

与函数类似，但是方法仅在结构体（枚举、trait 对象）中被定义，且第一个参数总是 self（调用该方法的结构体实例）。

使用 **impl** 块来对结构体进行功能实现，其中在方法签名中，&self 是 self: &self 的缩写。使用引用是因为我们只想读取结构体的数据而非写入。

```rust
struct Rectangle {
    width: u32,
    height: u32,
}
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
```

**还可以设置与结构体中字段相同的方法**，访问时，如果名称后有圆括号则调用方法，反之则访问字段值。这个特性与 JS 中的 getter 类似。

Rust 提供一个叫做**自动引用和解引用**的功能。当使用 object.something() 调用方法时，Rust 会自动为 `object`添加 `&`、`&mut`或 `*`以便使 `object`与方法签名匹配。例如：

```rust
p1.distance(&p2);
(&p1).distance(&p2);
```

方法传入更多参数，直接在 &self 后添加即可，调用时直接从第二个参数开始传即可

```rust
impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

所有在 impl 块中定义的函数都被称为**关联函数**（对象的静态方法），使用 :: 进行调用。关联函数就是直接在 impl 块中按照函数定义方式定义，所以不需要传入 self 作为第一个参数。

```rust
impl Rectangle {
    fn square(size: u32) -> Self {  // 此处的 Self 指的就是 Rectangle 类型
        Self {
            width: size,
            height: size,
        }
    }
}
let square1 = Rectangle::square(30);
println!("{:#?}", square1);
```

## 枚举与模式匹配

枚举是将一些值聚合成为一个集合的数据结构。枚举之中甚至还可以嵌套枚举。

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}
let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

枚举也**支持使用 impl 块为其定义方法**。

### Option 枚举

Rust 是没有空值的，但是拥有一个可以编码存在或不存在概念的枚举 Option<T>，它不需要被引入作用域，可以直接使用。

```rust
enum Option<T> {
    None,
    Some(T),
}
let some_char = Some('e');
let absent_number: Option<i32> = None;
```

### match 控制流

match 控制流接收一个变量作为判断条件，根据不同的条件进行不同的操作。最好能够使用枚举来标识各种条件。判断条件还可以是 Option<T>，没有值的时候使用 `None => {}` 分支，有值的时候任何符合类型的值都可以走  `Some(i) => {}`   分支。

匹配条件必须是有穷尽的。由于入参要制定类型，所以这个类型中所有的可能值都需要有对应的分支去处理，否则会在编译时报错。

对应 JS 中的 default 分支，Rust 中需要使用 other => {} 去进行匹配，other 也可以替换为 _ 下划线

```rust
#[derive(Debug)] // 这样可以立刻看到州的名称
enum UsState {
    Alabama,
    Alaska,
    // ...
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
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

### if let 语法

是 match 控制流的一个语法糖，当只匹配一个条件时，可以用这个语法。

```rust
let config_max = Some(3u8);
if let Some(max) = config_max {
    println!("The maximum is configured to be {}", max);
}
```

## 代码组织

Rust 提供了以下功能来管理和组织代码，它们被统称为”模块系统“：

- **包**（*Packages*）：Cargo 的一个功能，它允许你构建、测试和分享 crate。
- **Crates** ：一个模块的树形结构，它形成了库或二进制项目。
- **模块**（*Modules*）和 **use**：允许你控制作用域和路径的私有性。
- **路径**（*path*）：一个命名例如结构体、函数或模块等项的方式。

### crate

是 Rust 在编译时最小的代码单位，可以包含其他模块，编译时一起编译。

crate 有两种形式：二进制项和库。

- 二进制项可以被编译为可执行程序，必须有一个 `main` 函数作为入口。一般我们创建的文件都是 crate。
- 库没有 main 函数，也不会被编译为可执行程序，只是提供一些变量、函数等功能。

### 包

提供一系列功能的一个或者多个 crate。一个包会包含一个 ***Cargo.toml*** 文件，阐述如何去构建这些 crate。

包中可以包含至多一个库 crate(library crate)。包中可以包含任意多个二进制 crate(binary crate)，但是必须至少包含一个 crate（无论是库的还是二进制的）。

### 模块

模块可以让我们对一个 crate 中的代码进行分组。一个模块内的代码**默认是私有**的。模块由 mod 声明。

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
        fn seat_at_table() {}
    }
}
```

### 路径

为了调用一个函数，需要知道其路径。路径分为绝对路径、相对路径两种。

- 绝对路径：`crate::front_of_house::hosting::add_to_waitlist()`，最前面有 crate 开头，模块在同一个 crate 下时可以这么使用，更推荐使用绝对路径。
- 相对路径：`front_of_house::hosting::add_to_waitlist()`，以模块名开头。相对路径需要根据代码移动而变化。另外，相对路径还可以用 super 开头，表示

实际上，上面的 hosting 模块仅对于 front_of_house 模块可用，而 front_of_house 是对于 crate 模块可用的，所以 hosting 目前是不可用的，需要给该模块前添加 pub 关键字将路径暴露出去（hosting 里面的方法也要添加！）。 

**创建共有的结构体和枚举**：

- 结构体前添加 pub 表示结构体变为公有，但是其字段仍为私有。如果想要字段也变为公有，需要在字段前添加 pub 关键字
- 枚举前添加 pub 则会将枚举所有成员默认变为公有

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }
}
```

**use**

在作用域中增加 `use`和路径类似于在文件系统中创建软连接，创建模块的快捷方式。需要注意的是，**use 声明必须与引用该模块的方法处于同一作用域**，在子作用域里也是行不通的（除非移到子作用域里）。

use 关键字可以引入外部包，但需要在 Cargo.toml 文件中进行包和其版本的配置

```rust
use rand::Rng;
fn main() {
    let secret_number = rand::thread_rng().gen_range(1..=100);
}
```

- 在使用 use 语法时，不能直接缩写到某个函数，而是应该**带上最低一级的模块名称**。
- 如果引入的模块名称相同，可以使用 as 关键字进行重命名：`use std::io::Result as IoResult;`
- 使用 pub use 可以重新导出名称
    
    ```rust
    // 未重新导出，外部使用该模块需要使用路径：restaurant::front_of_house::hosting::add_to_waitlist()
    mod front_of_house {
        pub mod hosting {
            pub fn add_to_waitlist() {}
        }
    }
    // 重新导出模块后，外部使用路径为：restaurant::hosting::add_to_waitlist
    pub use crate::front_of_house::hosting;
    ```
    
- 合并引入，减少 use 和路径的代码
    
    ```rust
    use std::{cmp::Ordering, io, io::Write};
    ```
    
- 引入路径下所有公有项，在路径后跟 *：`use std::collections::*;`

### 一个模块的样例

用下面例子来梳理 mod、use、pub、as 关键字和 glob 运算符。

- 从 **crate 根节点**（库的 src/lib.rs，二进制的 src/main.rs）开始寻找
- 使用了 mod 声明的模块，首先从内联（mod someMod 后跟着 {} 中的代码）开始寻找，然后是 src/garden.rs，最后是 src/garden/mod.rs
- 接着会寻找子模块，寻找顺序同上

```rust
// 这是一个 crate 的目录
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs

// main.rs
use crate::garden::vegetables::Asparagus; // use 关键字后面跟代码路径，创建快捷方式，减少长路径重复
// mod 声明模块，告诉编译器应该包含在src/garden.rs文件中发现的代码
// pub 关键字，使一个模块的代码可以公用
pub mod garden;  
fn main() {
    let plant = Asparagus {};
    println!("I'm growing {:?}!", plant);
}
```

### 模块拆分

当模块变大时，不能将所有模块都定义到 crate 根文件中，而是 提取到各自的文件里。将模块拆分到不同文件里，需要在公用的模块前添加 pub mod 关键字（注意！公用的方法、结构体和枚举也需要添加 pub 关键字）。

## 集合

集合是值能够包含多个值的数据结构，与内建的数组/元组不同，集合是存在对上的数据，所以声明时不需要知道其大小。内建的集合数据结构有序列类型：Vec、VecDeque、LinkedList，键值对类型：HashMap、BtreeMap，还有 Set 和 BinaryHeap。广泛使用三种集合：**Vector**、**字符串**和**哈希map**。

由于集合都是存在堆上的，也就是说会有**数据转移**的问题，所以在访问和使用这些值时，都需要进行**引用**。

### Vector

类似数组的数据结构，但是可以声明未知大小的空值，并且**只能存储相同类型的值**。

```rust
let v: Vec<i32> = Vec::new();  // 新建
let v = vec![1, 2, 3];

let third: &i32 = &v[2];  // 访问，如果访问超出其长度，程序崩溃 panic
let fourth: Option<&i32> = v.get(2);  // 访问，如果访问超出其长度，返回 None

v.push(4);  // 尾部添加元素，但是**前面有可变引用，这里不可变引用会报错！**
v.pop();  // 移除并返回最后一个元素

****for i in &mut v {  // 循环遍历，需要可变引用才能进行修改
    *i += 50;
}
```

如果要在 Vector 中存储**不同类型的值**，可以使用**枚举**。因为枚举的成员都被定义为相同的枚举类型。

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
]  
```

### 字符串

Rust 中只有一种字符串类型：字符串 slice，一般以 &str 的形式出现。字符串都是 UTF-8 编码的。

字符用单引号，字符串用双引号。

字符串是 Vec<u8> 的封装，但是由于某些字符需要两个字节存储，所以会导致索引出的内容不正确。所以 String **不支持索引取值**，并且**谨慎使用 slice**！

```rust
let mut s1 = String::new();    // 新建空字符串
let s2 = "initial contents".to_string();    // 字面值转字符串
let s3 = String::from("initial contents");  

s1.push_str("bar");  // 字符串拼接
let s4 = s2 + &s3;    // 这里 s2 被移动无法继续使用，但是 s3 是引用，可以继续使用
let s5 = format!("{s1}-{s3}");

let sliceS = &s1[0..2];  // 注意！针对异常字符需要两个字节存储的，这样取值是有问题的

for c in "Зд".chars() {  // 针对字符的字符串遍历
    println!("{c}");
}
for b in "Зд".bytes() {  // 针对字节的字符串遍历
    println!("{b}");
}
```

### HashMap

`HashMap<K, V>` 类型储存了一个键类型 K 对应一个值类型 V 的映射。它通过一个**哈希函数**来实现映射。但是 HashMap 没有内置，需要从 std 库中引入。

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();  // 新建空的 hashmap

scores.insert(String::from("Blue"), 10);  // 插入/更新键值对，如果没有对应 key 则插入，反之则更新
scores.entry(String::from("Yellow")).or_insert(50);  // entry 函数返回 Entry 枚举；该操作是在 key 对应值不存在时插入新值

let score = scores.get(&team_name) // 根据 key 返回 value，如果有值返回 Option<&V>，如果没有值返回 None

for (key, value) in &scores {  // 遍历 hashmap
    println!("{key}: {value}");
}
```

对于存在堆上的值，如果在 hashmap 中，或插入其中，则所有权就会移动到 hashmap 上。

## 错误处理

Rust 将错误分为两大类：**可恢复的**和**不可恢复的**错误。可恢复的错误（例如找不到文件）只报告给开发者，不可恢复的错误会导致程序终止。

### panic! 处理不可恢复错误

一般遇到不可恢复的操作代码执行时会直接造成程序终止。另外一种显式调用 panic! 宏方法也可以达到相同效果。一般用来寻找代码出问题的地方（debug）。

### 用 Result 处理可恢复错误

Result 枚举有两个成员，T 和 E 是泛类型参数。

```rust
enum Result<T, E> {
    Ok(T),  // T 代表成功时返回成员的数据类型
    Err(E),  // E 代表失败时返回成员的数据类型
}
```

其中，Err 返回有不同的额原因，可以通过错误的 kind 属性来进行判断和匹配（if 条件或 match 匹配）

```rust
fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error);
            }
        },
    };
}
```

**unwrap**：如果返回成功，则返回 Ok 中的值，如果是失败会自动调用 panic! 宏

```rust
let greeting_file = File::open("hello.txt").unwrap();
```

**expect**：可以选择 panic! 报错的信息，语义化更容易追踪错误根源

```rust
let greeting_file = File::open("hello.txt").expect("hello.txt should be included in this project");
```

**错误传递**：是指向调用者返回成功或错误的信息。可以使用 ? 运算符，这里的 ? 不同于三元运算符。如果发生错误会中断后续程序，直接返回一个 Err 类型，如果正确需要指定返回 Ok 类型。

注意：? 运算符只能在返回值为 Result 枚举类型、Option<T> 枚举类型（返回 None 就提前返回）的函数中（例如 main 函数就不能使用），且这两种不能混搭

? 运算符后也支持链式调用方法

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();
    File::open("hello.txt")?.read_to_string(&mut username)?; // 如果 File::open 发生错误则不会执行 read_to_string 方法
    Ok(username)
}
```

### **何时使用 panic!**

使用 panic! 宏辉导致程序中断，而使用 Result 则是将选择权交给调用方法的使用者。示例、圆形代码和测试代码都比较适合使用 panic!

## 泛型、Trait 和 生命周期

泛型能够对代码进行抽象，增加代码复用性、减少代码冗余和错误。trait 是一个定义泛型行为的预发，能够限制特定类型。生命周期是一类向编译器提供引用如何相互关联的泛型。

Rust 中泛型并不会使程序比具体类型运行得慢。泛型代码会在编译时进行单态化来保证效率。

### 泛型数据类型

- 函数定义使用泛型：能够对逻辑相同、参数和返回值类型不同的函数进行抽象
    
    ```rust
    use std::cmp::PartialOrd;
    
    fn largest_char<T: PartialOrd>(list: &[T]) -> &T {
        let mut largest = &list[0];
    
        for item in list {
            if item > largest {
                largest = item;
            }
        }
    
        largest
    }
    ```
    
- 结构体使用泛型，定义结构体时在结构体名后面添加 <T,U> 即可，方法定义泛型也类似
    
    ```rust
    struct Point<T> {
        x: T,
        y: T,
    }
    impl<T> Point<T> {
        fn x(&self) -> &T {
            &self.x
        }
    }
    
    fn main() {
        let wont_work = Point { x: 5, y: 4.0 };  // 这样式行不通的，可以使用 Point<T, U> 泛型
    		let work = Point { x: 1.0, y: 4.0 }; // ok
    }
    ```
    
- 枚举使用泛型
    
    ```rust
    enum Option<T> {
        Some(T),
        None,
    }
    
    enum Result<T, E> {
        Ok(T),
        Err(E),
    }
    ```
    

### Trait

类似于接口（interfaces），定义某个特定类型拥有可能与其他类型共享的功能：将方法签名组合起来，定义一个实现某些目的必须的行为的集合。还可以为方法签名指定默认的实现：

```rust
pub trait Summary {
    fn summarize(&self) -> String {
					String::from("(Read more...)")  // 没有 impl 时的默认实现
		}
}
```

为类型实现 trait

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}
```

在使用上面的 struct 和 trait 进行类的实例化时，需要将 struct 和 trait 一起引入到作用域中才能正常使用 trait：

```rust
use aggregator::{Summary, Tweet};

fn main() {
    let tweet = Tweet {
        //...
    };
    println!("1 new tweet: {}", tweet.summarize());
}
```

可以在类所在的作用域下实现内部 trait 和外部 trait，但是**禁止为外部类型实现外部 trait**，为了排除编译时无从得知该使用哪个实现。

**trait 作为参数**

传参的类型，可以是实现了某个 trait 的类型，也可以是实现了多个 trait 的类型：

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

pub fn notify(item: &(impl Summary + Display)) {}  // 实现多个 trait 的类型

fn some_function<T, U>(t: &T, u: &U) -> i32  // 也可以用 where 从句
where
    T: Display + Clone,
    U: Clone + Debug,
{}
```

**trait 作为返回值**

返回值类型也可以是实现了某个 trait 的类型，使用方法：

```rust
fn returns_summarizable() -> impl Summary {}
```

### 生命周期

生命周期其实就是引用保持有效的作用域。Rust 还提供了泛型生命周期参数，确保运行时使用的引用绝对有效。

**Rust 如何判断一个变量是否离开作用域**

Rust 编译器有一个借用检查器，用来比较作用域。如果一个变量在自己的作用域 `a 引用了另一个变量的值（其作用域为 `b），如果作用域 `b 结束后作用域 `a 还在引用，就会无法编译。

生命周期注解，类似泛型类型参数：

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

# 附录

### 实用开发工具

- 自动格式化：rustfmt
- 修复代码（解决错误与报警）：rustfix
- lint 功能：clippy
- vscode 语法助手：rust-analyzer