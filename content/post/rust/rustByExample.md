---
title: "RustByExample"
date: 2021-04-21T17:59:51+08:00
lastmod: 2021-04-21T17:59:51+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: ["rust"]
categories: ["develop"]
author: "王清"
---

# rust install

## windows安装 

最开始在rustlang官网直接下载的exe安装程序, 但是写完helloworld之后 rustc main.rs, 会报错找不到link.exe, 好像是要安装 c++ build tool, 然后就去windows官网下载了, 国语都说不清的人 英语也不很强, 没找到正确的安装包, 百度出来的多是 visualcppbuildtools_full.exe 2015版的, 没找到2019版的. 这里我本来的想法是一定要在windows上搞出来, 结果就悲剧了, 安装完就是 x86_64-pc-windows-msvc 这个版本的rust, gun版的是不需要安装 c++ build tool的,  后来就不想装了 突然发现了单独安装的版本

### 单独安装 

[Standalone installers](https://forge.rust-lang.org/other-installation-methods.html)

安装完之后没有 cargo rustup 等命令, 还是选择了用 rustup 安装, 相当悲剧了, 简直了

### rustup 安装

版本: x86_64-pc-windows-gnu 

首先下载 rustup-init.exe,  打开之后选择第2项 customize install, 默认安装的是msvc版本, 然后我输入 strip host: x86_64-pc-windows-gnu , 选择 stable 版本, 不知道有没有安装成功

默认是安装在C盘家目录 .cargo .rust 目录, 不想的话可以剪切走, 然后设置环境变量 CARGO_HOME:F:\.cargo , RUSTUP_HOME:F:\.rustup, 

设置国内源: 在cargo目录下添加config文件, 需要这一步??

```shell
[registry]
index = "https://mirrors.ustc.edu.cn/crates.io-index/"
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index/"
```

再添加环境变量

```
RUSTUP_DIST_SERVER:http://mirrors.ustc.edu.cn/rust-static
RUSTUP_UPDATE_ROOT:http://mirrors.ustc.edu.cn/rust-static/rustup
```

`PATH 添加 %CARGO_HOME%\bin`

然后再安装gnu版的rust

`rustup install stable-x86_64-pc-windows-gnu`

将 gnu 版设为 default

`rustup default stable-x86_64-pc-windows-gnu`

安装 fmt

`cargo +stable-x86_64-pc-windows-gnu install rustfmt`

安装 racer

`rustup component add rust-src --toolchain stable-x86_64-pc-windows-gnu`

## linux 安装

我在命令行环境下安装的, `curl https://sh.rustup.rs -sSf | sh` 这个命令就行, 会自己安装, 有时候会网络不好, 翻个墙就可以, 这个脚本可以看看 里边写了挺多适配的系统的

## macos 安装

很qiong

# rust for example

## 关键字


## 变量 常量

可变: mmutable, 不可变: immutable

声明不可变变量: `let x = 5`  
声明可变变量: `let mut x = 5`

声明常量: `const MUX_POINTS: u32 = 100_00`, 100_00 用来提高可读性

隐藏(shadowing): 一个变量总是可以被重新定义 `let x = 5; let x = x + 1; let x = x * 2;` 最终x变成12

对于不可变变量可以使用 let 重新赋值并且可以改变变量类型 `let space = "   "; let space = space.len();` 变成了我们长度, 不过结果仍然是不可变的

使用 `let mux space = "   "; space = space.len();` 将会报错, 不可改变变量类型

## 数据类型

两类数据类型子集: 标量(scalar) 复合(compound)

`let guess: u32 = "42".parse().expect("Not a number!");` 可以将 string 转变为数字, 需要显示标注 `guess: u32`

标量类型: 代表一个单独的值, rust有四种标量类型: 整型、浮点型、布尔类型和字符类型

### 整形

```shell
长度    有符号 无符号
8-bit   i8    u8
16-bit  i16   u16
32-bit  i32   u32
64-bit  i64   u64
arch    isize usize
```

有符号: i, 无符号: u

isize 和 usize 类型依赖运行程序的计算机架构：64 位架构上它们是 64 位的， 32位架构上它们是 32 位的

Rust 中的整型字面值
```shell
数字字面值  例子
Decimal     98_222
Hex         0xff
Octal       0o77
Binary      0b1111_0000
Byte(u8 only) b'A
```

### 浮点型

rust的浮点数类型是 f32 和 f64, 分别占 32位, 64位 默认是 f64, f32 是单精度浮点数， f64 是双精度浮点数。

### 数值运算

`+ - * / %`: 加减乘除取余操作

### 布尔型

`let t = true; let t:bool = true;`

### 字符类型

char 类型

用单引号表示 `let a = 'Z'` 因为是 unicode 可以代表许多字符,特殊字符或表情 跟golang 一样  `let heart_eyed_cat = 'ᛀ';` `let z = 'ℤ';`

## 符合类型

包含两种 元组(tuple) 和 数组(array)

### 元组 tuple

`let tup: (i32, f64, u8) = (500, 6.4, 1);` 将 i32 f64 u8 组合成一个新的元素

从元组中获得单独一个值就需要解构, 使用 模式匹配 或 索引

模式匹配: `let tup = (500, 6.4, 1);let (x, y, z) = tup;` 

索引: 索引下标从0开始
```rust
let x: (i32, f64, u8) = (500, 6.4, 1);
let five_hundred = x.0;
let six_point_four = x.1;
let one = x.2;
```

### 数组 array

`let a = [1, 2, 3, 4, 5];`

数组长度不允许改变

声明月份: `let months = ["January", "February", "March", "April", "May", "June", "July","August", "September", "October", "November", "December"];`

声明数组类型: `let a: [i32; 5] = [1, 2, 3, 4, 5];`

访问数组元素: `a[0]` 使用下标

当数组下标超出实际的值后, 报错 但不会退出

## 函数

函数定义 

```rust
fn main() {
    anothor_func()
}
fn anothor_func() {
    println!("anothor func")
}

带参数的函数定义

```rust
fn main() {
    another_function(5);
} 
fn another_function(x: i32) {
    println!("The value of x is: {}", x);
}
fn another_function(x: i32,y: i32) {
    println!("The value of x is: {}, y is {}", x, y);
}
```

包含语句和表达式的函数体

`5 + 6`, `5`, `let y = { let x = 3; x + 1};` 最后没有分号, 

具有返回值的函数, 注意 {} 中 有分号的是语句, 没有分号才能是表达式

```rust
fn func4(x:i32) -> i32 {
    let y = x * 2
    y + 6
}
```

## 注释

`// 这是一个注释` 单行生效

`/* 这中间的都是注释 */` 包裹起来的生效

## 控制流

### if

```rust
if number < 5 {
    println!("condition was true");
} else {
    println!("condition was false");
}
```

多个else

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

### let 中使用 if

```rust
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);
}
```

返回值类型必须一样

```rust
fn main() {
    let condition = true;

    let number = if condition {
        5
    } else {
        "six"
    };

    println!("The value of number is: {}", number);
}
```

### 循环语句

Rust 有三种循环：loop、while 和 for, 使用 break 可以跳出循环语句

从循环中返回

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2; // 将返回值加入break中返回
        }
    };

    assert_eq!(result, 20); // debug strack 如果不想等会返回debug内容
}
```

while 循环

```rust
let mut number = 3;
while number != 0 {
    println!("{}!", number);

    number = number - 1;
}
println!("LIFTOFF!!!");
```

size 类型还是有区别的  好像不能小于0

for 循环

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() { // element 是一个指针类型 &i32
        println!("the value is: {}", element);
    }
}
```
反转的例子， 左开右闭？？？

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```

## rust 所有权

十分重要， 使得rust不必垃圾回收

栈： 后进先出

堆： 
