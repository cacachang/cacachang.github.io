---
title: 成為程式界的 F1 賽車手 - Rust 與變數們
date: 2024-09-06 00:13:47
category: Rust
---

### 淺談 Rust 淵源

Rust 是由 Mozilla 主導開發

Rust 本來是 Mozilla 員工 Graydon Hoare 的 side project

後來 Mozilla 開始支持這個專案，在 2010 年公開

並於 2015 年釋出第一個穩定版本

截至 2024 年 8 月已釋出 1.80.1 穩定版

Rust 在 stack overflow 的 2024 年調查中，Rust 被列為最讚賞的語言之一

Rust 的高效能特性，讓許多的工具都掀起以 Rust 的改寫風潮


### 該怎麼安裝

本系列文章將以 MacOS 為主

官方推薦使用 rustup 來安裝， Window 需要另行安裝

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

```
info: downloading installer

Welcome to Rust!

This will download and install the official compiler for the Rust
programming language, and its package manager, Cargo.

Rustup metadata and toolchains will be installed into the Rustup
home directory, located at:

  /Users/zhangjianing/.rustup

This can be modified with the RUSTUP_HOME environment variable.

The Cargo home directory is located at:

  /Users/zhangjianing/.cargo

This can be modified with the CARGO_HOME environment variable.

The cargo, rustc, rustup and other commands will be added to
Cargo's bin directory, located at:

  /Users/zhangjianing/.cargo/bin

This path will then be added to your PATH environment variable by
modifying the profile files located at:

  /Users/zhangjianing/.profile
  /Users/zhangjianing/.bash_profile
  /Users/zhangjianing/.bashrc
  /Users/zhangjianing/.zshenv

You can uninstall at any time with rustup self uninstall and
these changes will be reverted.

Current installation options:


   default host triple: aarch64-apple-darwin
     default toolchain: stable (default)
               profile: default
  modify PATH variable: yes

1) Proceed with installation (default)
2) Customize installation
3) Cancel installation
```

沒有特別要設定就選 1 吧

```
>1

info: profile set to 'default'
info: default host triple is aarch64-apple-darwin
info: syncing channel updates for 'stable-aarch64-apple-darwin'
781.5 KiB / 781.5 KiB (100 %) 699.0 KiB/s in  1s ETA:  0s
info: latest update on 2023-08-03, rust version 1.71.1 (eb26296b5 2023-08-03)
info: downloading component 'cargo'
  5.0 MiB /   5.0 MiB (100 %)   1.1 MiB/s in  4s ETA:  0s
info: downloading component 'clippy'
  1.9 MiB /   1.9 MiB (100 %)   1.1 MiB/s in  1s ETA:  0s
info: downloading component 'rust-docs'
 13.6 MiB /  13.6 MiB (100 %)   1.6 MiB/s in 10s ETA:  0s
info: downloading component 'rust-std'
 23.7 MiB /  23.7 MiB (100 %)   1.9 MiB/s in 14s ETA:  0s
info: downloading component 'rustc'
 52.6 MiB /  52.6 MiB (100 %)   3.3 MiB/s in 27s ETA:  0s
info: downloading component 'rustfmt'
  1.4 MiB /   1.4 MiB (100 %)   1.0 MiB/s in  2s ETA:  0s
info: installing component 'cargo'
info: installing component 'clippy'
info: installing component 'rust-docs'
 13.6 MiB /  13.6 MiB (100 %)   7.2 MiB/s in  1s ETA:  0s
info: installing component 'rust-std'
 23.7 MiB /  23.7 MiB (100 %)  19.2 MiB/s in  1s ETA:  0s
info: installing component 'rustc'
 52.6 MiB /  52.6 MiB (100 %)  21.6 MiB/s in  2s ETA:  0s
info: installing component 'rustfmt'
info: default toolchain set to 'stable-aarch64-apple-darwin'

  stable-aarch64-apple-darwin installed - rustc 1.71.1 (eb26296b5 2023-08-03)


Rust is installed now. Great!

To get started you may need to restart your current shell.
This would reload your PATH environment variable to include
Cargo's bin directory ($HOME/.cargo/bin).

To configure your current shell, run:
source "$HOME/.cargo/env"

```

安裝完畢就可以確認安裝的版本（記得要重開終端機）

```
> rustc --version

# rustc 1.80.1 (3f5fd8dd4 2024-08-06)
```


### 印出 Hello World

我們先來做一個 rs 檔案 ( Rust 的原始碼檔案) 

```
$ touch hello.rs
```

打開 hello.rs 檔案，輸入下列這段程式碼後存檔

```
fn main() {
    println!("Hello, World!");
}
```

Rust 是編譯語言，在進入執行其之前需要先編譯成電腦看得懂的語言，
所以我們用 rustc 來編譯

```
rustc hello.rs
```

接著執行這個檔案

```
./hello.rs
```

就會出現 `Hello, World!` 囉


### 註解

為了防止程式語言寫得過於艱深，或者是比較特殊的功能
我們會用註解 (comments) 來解釋程式碼

在 Rust 中，該怎麼下註解呢？

```
// 這是單行註解
/*...*/ 這是多行註解
```

以剛剛的 hello.rs 來示範單行註解

```
fn main() {
    // first, say hello to the world.
    println!("Hello, World!");
}
```

那多行註解呢？

```
fn main() {
    /* 
    I'm beginner of rust,
    but I'll work hard!
    */
    println!("Hello, World!");
}
```

寫好再重新編譯，結果依然是印出 Hello, World! 唷


### print

在 Rust 中，要印出東西，可以用兩種 Macro，
分別是 print! 跟 println!

* 什麼是 Macro?

Macro 有點像 Function ，但其實不能跟 Function 劃上等號，Macro 的概念更像是 Meta Programming，就是以程式來撰寫程式的意思


我們直接在 hello.rs 中試試

```
fn main() {
  print!("Hello,");
  print!("World!");
  
  println!("Hello,");
  println!("World!");
}
```

一樣編譯後再執行，這時候終端機的結果是這樣：

```
Hello,World!Hello,
World!
```

由上述結果可知道 `print!` 不會換行
`ln` 會在後面加上換行的動作
所以 `println!` 會換行

不過這個只能印字串，如果要印變數，必須這樣使用

```
let a = "Hello";
println!("{}", a);
```

cannot be formatted with the default formatter
有時候會跑出該型態不支援預設 formatter 的錯誤，這時我們就必須這樣用


```
let a = [1, 2, 3];
println!("{:?}", a);
```

### 變數

當我們需要將一個值儲存起來，在日後做使用的時候，我們就需要用到變數

將變數指向這個值，需要的時候再拿出來用他

我們來看一下在 Rust 中該怎麼定義變數

#### 定義變數

我們會用 `let` 來定義變數 (有點像 JavaScript)，來看看以下範例

```
let aa = 1;
```

我們會定義出一個 aa 的變數，並將它的值指向 1

接下來我們試著印出變數

```
fn main() {
  let age = 18;
  println!("{}", age)
}
``` 

```
$ rustc main.rs
$ ./main

# 18
```

Rust 對於變數相對嚴謹，基本上定義後就不能再改變


如果有天突然想給他改值呢？

```
fn main() {
  let age = 18;
  println!("{}", age)
}
``` 

```
$ rustc main.rs

error[E0384]: cannot assign twice to immutable variable `age`
 --> main.rs:5:3
  |
2 |   let age = 18;
  |       ---
  |       |
  |       first assignment to `age`
  |       help: consider making this binding mutable: `mut age`
...
5 |   age = 19;
  |   ^^^^^^^^ cannot assign twice to immutable variable

error: aborting due to previous error

For more information about this error, try `rustc --explain E0384`.
```

在編譯的時候就直接噴錯

但我們一定會需要改變變數值的時候，該怎麼做呢？

使用 `mut` 就可以定義一個可變的變數

```
let mut age = 18;
```

我們實際來試試

```
fn main() {
  let mut age = 18;
  println!("{}", age);

  age = 19;
  println!("{}", age);
}
```

```
$ rustc main.rs
$ ./main

# 18
# 19
```

以上是 Rust 定義變數的方式

#### 變數命名規則

在 Rust 中，大寫與小寫是不同的，所以 age != AGE

還有幾個規則務必要遵守

* 開頭不可用數字
* 除了英文 / 數字 / 底線以外，不可加入其他符號或語言文字
* 如果會有兩個以上的單字，請用蛇式命名法，比如說變數名稱是我的年齡，可取作 my_age

### 常數是什麼

常數顧名思義就是永恆不變的，所以基本上他是不能被改變的

### 定義常數

在 Rust 中定義常數的方式跟 JavaScript 很像

```
const PI: f32 = 3.14;
```

* 定義方式：使用 const
* 名稱：const 後面必須接大寫
* 指定型別：需指定型別給它，否則他會自行判斷，稍後我們會介紹型別

之後再賦予一個值給常數即可

我們來試試印出常數

```
fn main() {
  const AGE: i8 = 18;
  println!("{}", AGE);
}
```

```
$ rustc main.rs
$ ./main

# 18
```

如果我們想改常數的值呢？會發生什麼事

應該想都不用想就是會噴錯，不過會噴什麼錯誤呢？

```
fn main() {
  const AGE: i8 = 18;
  println!("{}", AGE);

  AGE = 19;
  println!("{}", AGE);
}
```
```
$ rustc main.rs

error[E0070]: invalid left-hand side of assignment
 --> main.rs:5:7
  |
5 |   AGE = 19;
  |   --- ^
  |   |
  |   cannot assign to this expression

error: aborting due to previous error

For more information about this error, try `rustc --explain E0070`.
```

它會跟你說左邊的定義是不合法的唷！

### 基本型別

Rust 屬於強型別語言，必須明確定義型別給它，否則他會自行判斷型別

我們剛剛在定義常數的時候有加上了型別，相信大家在定義的時候知道怎麼加入型別了

所以就來看一下基本型別有哪些吧！

* 整數 Integer
* 浮點數 Floating-Point
* 布林值 Boolean
* 字元 Character


#### 整數 Integer

年齡都是用整數來算，用以下範例來舉例

```
fn main() {
  let age: i8 = 18;
  println!("{}", age);
}

```
來一一拆解一下

`i` 指的是 integer
`8` 指的是 在記憶體中佔據 8 bits

如果今天要存取的數字比較大，可以用 `32` `64`
不管是正數還是負數都算在 integer 的型別範圍內

假設今天要指定這個型別只能為正數，我們需把 `i` 換成 `u` 就可以囉

```
fn main() {
  let age: i8 = 18;
  println!("{}", age);
}

# 結果： 18
```

如果把它定義成負數會怎樣呢？

```
fn main() {
  let price: u32 = -100;
  println!("{}", price);
}
```
```
$ rustc main.rs

error[E0600]: cannot apply unary operator `-` to type `u32`
 --> main.rs:2:20
  |
2 |   let price: u32 = -100;
  |                    ^^^^ cannot apply unary operator `-`
  |
  = note: unsigned values cannot be negated

error: aborting due to previous error

For more information about this error, try `rustc --explain E0600`.
```

會出現不可以加上 `-` 的錯誤訊息哦！

#### 浮點數 Floating-Point

浮點數有 32 bits 跟 64 bits 兩種選項，小數點後面的位數越多，要使用的 bits 數就要越大 (因為要存在記憶體中)

```
fn main() {
  let product_price: f32 = 5.5;
  println!("{}", product_price);
}

# 結果： 5.5
```

#### 布林值 Boolean

布林值的定義很簡單，只要加上 `bool` 即可

```
fn main() {
  let have_fun: bool = true;
  println!("{}", have_fun);
}

# 結果： true
```

#### 字元 Character

我們可以定義任何符號，甚至是數字為字元的型別，型別設定為 `char` 即可
但只能放`一個字元`喔，後面務必要用`單引號`，雙引號是用來定義字串的

```
fn main() {
  let money: char = '$';
  println!("{}", money);
}

# 結果： $
```

型別一旦定義後就不能再變了嗎？ Rust 提供了變換型別的方法，下一篇將介紹型別轉換方法！

