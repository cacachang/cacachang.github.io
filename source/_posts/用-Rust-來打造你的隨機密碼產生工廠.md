---
title: 用 Rust 來打造你的隨機密碼產生工廠
date: 2024-09-13 02:08:31
category: Rust
---

在 stack overflow 的 2024 年調查中， Rust 被列為最讚賞的語言之一

Rust 的高效能特性，讓許多的工具都掀起以 Rust 的改寫風潮

這次我們會用 Rust 來做一個工具，讓大家體驗用 Rust 開發的樂趣

### 事前準備

* 安裝 Rust
* 可參考[專案成果](https://github.com/cacachang/password_generator)

### step 0. 用 Cargo 新增專案

Cargo 是 Rust 的套件管理工具，我們可以用 Cargo 來新增專案，也可以用它來編譯 Rust 檔案以及執行

* 新增專案

```
cargo new password_generator
```

當我們要編譯 Rust 專案時，就可以下該指令：

```
cargo build
```

但如果是編譯加上執行專案的話，就可以改以下指令：

```
cargo run
```

### Step 1. 打造 CLI

首先，我們要來打造 CLI ，來創造使用者與程式互動的介面

我們會用一個套件，叫做 `inquire` 來做出 CLI

第一次加套件，就先用手動的方式來加入吧！

Cargo.toml 檔案中不只是放置檔案的基本資料，也會放套件的資訊哦！

```
# Cargo.toml

[dependencies]
inquire = "0.7.5"
```

這個套件有支援很多種輸入方式，有包含以下：

* 文字輸入
* 日期選擇
* 選擇題
* 複選題

有興趣的話可直接進入到該套件的 [github](https://github.com/mikaelmello/inquire?tab=readme-ov-file) 看看

接著我們來設定密碼的長度

### Step 2 - 1. 設定密碼長度

我們要先到 `main.rs` 檔案中，

#### main.rs

每一個 Rust 專案都必須要有 main.rs 檔案，

我們可以把 main.rs 當作是一個進入點，

main.rs 有幾個特性

* 沒有參數
* 不會有 return 值

介紹完 main.rs 後，我們就來設定密碼長度吧！

由使用者自行決定密碼長度相對彈性，

因此我們會使用套件的文字輸入，讓使用者自行輸入需要的密碼長度

在使用 Text 模組前，我們要在檔案上面使用 `use` 來載入該套件的 Text 模組，並且做出文字輸入的問題

```
# main.rs
use inquire::Text;

fn main() {
    let length_ans = Text::new("請問您的密碼長度要設定多少？").prompt()
}
```

#### 定義變數

這邊要跟大家提一下，在 Rust 中的變數定義需要用 let 或者 const 

```
let number_one = “1”
const number_two = “2”
```

跟 JavaScript 一樣， let 是可以做更改的， const 是不能做更改的

如果要讓該變數的值可以更改，我們就必須這樣用

```
let mut number = “1234”
```

#### Error Handling

我們在使用 `Text::new` 模組中的 `prompt` 方法時，會回傳一個 `Result` 型態的值

`Result` 是 Error Handling 的一種

當我們需要判斷程式結果時，就可以使用 `Error Handling`

`Error Handling` 分為 `可復原的` 以及 `不可復原的`

可復原的為 `Result` 與 `Option`

`Result<T, E>` 來表示結果是成功還是失敗，需要兩個參數，一個是 T，一個是 E，其實這兩個參數是

T 代表著成功後回傳的變數
E 則是失敗回傳的錯誤訊息

來個小範例：

```
fn main() {
    let number: Result<i32, _> = "123".parse();

    match number {
        Ok(num) => println!("解析成功：{}", num),
        Err(error) => println!("無法解析：{}", error),
    }
}
```

`Option<T>` 適用於此變數或者結果是否存在，會有 None 以及 Some 這兩個結果

這邊的 T 跟 Result 不太一樣，可能會是 Some 或者是 None，不過他們都是參數

Some(c) 表示存在，並且回傳 c 變數(不一定要叫 c ，想叫什麼都可)
None 表示不存在

來個小範例：

```
fn main() {
    let a = [1, 2, 3];
    let b = a.get(2);

    let _c = match b {
        None => println!("沒有這個值"),
        Some(c) => println!("結果為：{}", c),
    };
}
```

所以他們用法上會有一些差別

#### print

接著我們來介紹在 Rust 中要怎麼把東西印出來

如果是要印一般的字串，其實蠻簡單的，用下方寫法就可以了

```
println!("我是 Rust 我最棒！")
```

但如果是要印出變數，我們就必須使用 `{:?}` ，來看一下用法

```
let language = "Rust";
println!("我是 {}", language)

// 我是 Rust
```

```
let language = ["Rust", "Ruby"]
println!("{:?}", language)


// ["Rust", "Ruby"]
```

用文字輸入做出密碼長度的問題後，

我們還要再去定義密碼長度的變數，

因為使用 `Text::new("請問您的密碼長度要設定多少？").prompt()` 做出來的會是一個 `Result`，

之後要產生密碼是需要抓數字來判斷長度要多少，

所以我們可以要另外定義
( 預設可給 8，如果使用者輸入的值不是 8 ，就得改變，所以要給 `mut` )

```
fn main() {
    ...
    let mut length = 8;
}
```

到這邊，我們就把密碼長度設定好了！

大家可以在終端機下 `cargo run` 來試試是否跳出 CLI

如果成功的話，我們就進行下一步吧！來判斷使用者所輸入的答案

### Step 2 - 2. 設定密碼長度 - 判斷輸入的文字

在 Rust 中，判斷變數符合哪個情況，可以使用 `match`

`match` 類似 JavaScript 中的 switch 

會依照值符合哪個狀況來執行不同的程式碼

假設 `length_ans` 是沒有錯誤的，那我們就去解析是否為數字

如果是數字，那就顯示設定的密碼長度為多少，並且將 length 重新賦值

如果不符合數字，那就會顯示 `您輸入的不是有效的整數`

```
fn main() {
    ...
    match length_ans {
        Ok(length_ans) => match length_ans.parse::<i32>() {
            Ok(number) => {
                println!("設定的密碼長度為: {}", number);
                length = number;
            }
            Err(_) => {
                println!("您輸入的不是有效的整數");
            }
        },
        Err(_) => {
            println!("請輸入密碼長度");
        }
    }
}
```

設定完後，就可以輸入 `cargo run` 來試試執行結果！



### Step 3. 定義密碼範本變數

我們會從密碼範本中產出隨機碼做成密碼，
由於密碼範本可能會更改，所以我們要讓他是 `mut`

```
fn main() {
    ...
    let mut charset = String::from("");
}
```

### Step 4 - 1. 是否包含數字 - 定義密碼的數字範本變數

在判斷是否包含數字以前，我們先將數字的密碼範本定義好

```
fn main() {
    ...
    const PASSWORD_NUMBER: &str = "1234567890";
}
```

#### &str

在這邊我們看到了一個陌生的型態 `&str` 

它叫做字串切片，顧名思義就是從字串切下來的一段字

要注意的是，他與字串是不一樣的喔

字串切片是不能被更改的

### Step 4 - 2. 是否包含數字 - 密碼是否包含數字

這次我們會用選擇題來讓使用者回答，密碼是否要包含數字

所以我們會使用到 `Confirm` 模組

`with_default` 可以讓你建立預設值，假設使用者直接按 `enter` 下去，他就會直接帶入預設值

```
use inquire::{Confirm, Text};

fn main() {
    ...
    let contain_number = Confirm::new("密碼是否包含數字").with_default(true).prompt();
}
```

### Step 4 - 3. 是否包含數字 - 判斷輸入的答案

一樣我們會用 `match` 來判斷狀況，並且透過 `Result` 來判斷程式碼是否有誤

假設沒有錯誤，就會印出 `你的密碼將包含數字`

且將 `數字範本` 塞入 `密碼範本` 中

```
fn main() {
    ...
    match contain_number {
        Ok(contain_number) => {
            if contain_number {
                println!("你的密碼將包含數字");
                charset.push_str(PASSWORD_NUMBER)
            } else {
                println!("你的密碼不包含數字");
            }
        }
        Err(_) => println!("請選擇是否包含數字"),
    }
 }
```

### Step 5 - 1. 密碼是否包含字元 / 符號 - 定義密碼的字元 / 符號範本變數

在設定是否要讓密碼包含字元 / 符號前，

我們也先來定義 字元 / 符號 的密碼範本

```
fn main() {
    ...
    const PASSWORD_LOWERCASE: &str = "abcdefghijklmnopqrstuvwxyz";
    const PASSWORD_UPPERCASE: &str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    const PASSWORD_SYMBOL: &str = "!@#$%^&*()_+-=[]{}|;:',.<>?";
}
```

### Step 5 - 2. 密碼是否包含字元 / 符號 - 複選題

接著我們會讓使用者用複選的方式來看是否要包含大小寫英文字母 還是 符號

我們會需要用到 `MultiSelect` 模組

```
use inquire::{Text, Confirm, MultiSelect};

fn main() {
    ...
    let options = vec!["lowercase", "uppercase", "symbol"];
    let char_ans = MultiSelect::new("請選擇密碼是否要包含:", options).prompt();
}
```

#### Vector

Vector 是一種存放資料的集合型別，在 Rust 中是屬於標準函式庫型別，

放置在同一個 Vector 裡面的資料都是要一樣型態的

```
fn main() {
    let collection = vec![1, 2, 3, 4];
    println!("{:?}", collection)
}

結果：
[1, 2, 3, 4]
```

### Step 5 - 3. 密碼是否包含字元 / 符號 - 判斷輸入的答案

一樣是透過 `match` 來判斷 char_ans 

比較特別的是， `choice` 會是一個 `Vector`，因此我們需要用 `for ... in` 將裡面的答案一個個拿出來比對，

如果有 match 到大寫英文，我們就將大寫英文範本塞進去密碼範本中

match 選項最後有個 `_` ，這表示如果沒有到 match 上述的任何一個，就會走這條

```
fn main() {
    ...
    match char_ans {
        Ok(choices) => {
            for choice in choices {
                match choice {
                    "lowercase" => charset.push_str(PASSWORD_LOWERCASE),
                    "uppercase" => charset.push_str(PASSWORD_UPPERCASE),
                    "symbol" => charset.push_str(PASSWORD_SYMBOL),
                    _ => (),
                }
            }
        }
        Err(_) => println!("選擇出現錯誤"),
    }
}
```

### Step 6 - 1. 產生隨機碼 - 安裝套件

接著我們需要用密碼範本來隨機產生

所以我們會需要用到 `Rand` 這個套件，

這個套件可以讓我們隨機產生數字，

所以直接下 `cargo add rand`

就會自動在 Cargo.toml 中加上 `rand = "0.8.5"` 了

### Step 6 - 2. 產生隨機碼 - 產出密碼

這邊我們會使用到 rand 的 Rng 模組，

我們會透過 `rand::thread_rng()` 來產生隨機數字

會先將密碼範本中的每個字或者數字轉成 u8 格式，並且集合成一個陣列，`char_bytes` 會指向這個陣列

接著我們會由 0 ~ 密碼長度依序隨機抓出一個 index

最後在 `char_byte` 取出該 index 的值，並且轉為 `字元` 格式

最後將他們集合起來成為密碼


```
use rand::Rng;

fn main() {
    let mut rng = rand::thread_rng();
    let char_bytes: &[u8] = charset.as_bytes();
    let password: String = (0..length).map(|_| {
        let idx = rng.gen_range(0..char_bytes.len());
        char_bytes[idx] as char
    }).collect();

    println!("您的密碼為：{:?}", password)
}
```

#### u8 

為正數的數字，不包含負數，u 是指 `unsigned` ， 8 則是指 8-bit

#### 字元

我們可以定義任何符號，甚至是數字為字元，型別設定為 char 即可
但只能放一個字，後面務必要用單引號

```
fn main() {
    let money: char = '$';
    println!("{}", money);
}

# 結果： $
```

### Step 6 - 3. 包成方法

待會要去判斷密碼中是否包含數字(常出現結果沒有數字)，所以隨機產生密碼的方法會重複使用

所以就來把他包成一個方法，記得要把它放在 main 方法外面，並且回傳為一個 `String` 字串格式

而剛剛放在 main 方法中的隨機產生碼就可以先刪除了


```
fn generate_password(charset: &str, length: i32) -> String {
    let mut rng = rand::thread_rng();
    let char_bytes: &[u8] = charset.as_bytes();

    let result = (0..length)
        .map(|_| {
            let idx = rng.gen_range(0..char_bytes.len());
            char_bytes[idx] as char
        })
        .collect();

    result
}
```

#### function

當 function 會需要參數的時候，必須要將型別定義清楚，就像上方的 `charset: &str` / `length: i32` 一樣，

如果有回傳值，也必須要定義好型態，以上方例子來說，會回傳一個 String 型態


### Step 7 - 1. 密碼未包含數字，就重來

沒有說要包含數字的話，我們就直接將產生密碼，不需要再做檢查

```
fn main() {
    ...
    let mut password = generate_password(&charset, length);
}
```


接著必須先判斷使用者有沒有說要包含數字

所以一樣用 `match` 與 `Result` 去判斷

```
fn main() {
    ...
    match contain_number {
        Ok(contain) => {
            if contain {
                ... 
            }
        }
        Err(_) => {}
    }
}
```

如果有包含數字的話，就必須要去跑 loop 檢查是否有數字，沒有就重新產生，並且重新賦值

```
fn main() {
    ...
    match contain_number {
        Ok(contain) => {
            if contain {
                loop {
                    let origin_result = generate_password(&charset, length);
                    if origin_result.chars().any(|a| a.is_digit(10)) {
                        password = origin_result;
                        break;
                    }
                }
            }
        }
        Err(_) => {}
    }
    println!("您的密碼為：{:?}", password)
}
```

以上就完成密碼產生器囉！
