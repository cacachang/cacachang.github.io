---
title: 成為程式界的 F1 賽車手 - Rust 的 Function 與 Block
date: 2024-10-06 21:36:13
category: Rust
---

我們在開發的時候，很常會使用到 Function

大家有想過為什麼要使用 Function 嗎？

當程式碼開始變多，且開始在不同的地方寫了相同的程式碼之後，我們就會用 Function 把這些相同地方包起來

對我來說，使用 Function 有幾個不同的意義

1. 定義輸入值與輸出值的關係（可參考[龍哥的文章](https://kaochenlong.com/2023/09/22/function.html))
2. 針對該 Function 內的程式碼，賦予名稱意義，使開發者更容易懂
3. 減少程式碼重複的部分，用 Function 包起來可達成重複使用的效果

現在我們就來看一下 Rust 中的 Function

### Function

在 Rust 中，Function 以 fn 關鍵字開頭，並有一個名稱、參數和返回值

這邊我們定義了一個叫做 hello 的 Function

```
fn hello(){
    println!("hello");
}
```

### 參數

剛剛使用 Function 意義有提到， Function 可以定義輸入值與輸出值的關係

當我們需要用到參數的時候，就代表著有輸入值

而 Function 回傳的結果則是輸出值

我們會透過 Function 來讓開發者知道，輸入值與輸出值的關係

讓我們來看一下該怎麼使用吧

要注意的是，Function 的參數是需要指定型別的，有助於在編譯的時候就找到錯誤

```
fn calculator(a: i32, b: i32) {
    let result = (a + b) * b;

    println!("{}", result)
}
```

### main

每個 Rust 的專案都一定會有 main 這個 Function

main Function 不能放任何參數(但某些特殊狀況會需要放參數)，也不能有 return 值

我們可以將 main Function 想像成是專案程式進入點，它會比任何 Function 還優先執行

因此我們會將 main 作用域以外的 Function 放在 main 裡面呼叫

以剛剛的例子來看，我們會這樣寫

```
fn calculator(a: i32, b: i32) {
    let result = (a + b) * b;

    println!("{}", result)
}

fn main(){
    calculator(1, 2)
}

// 6
```

### return 指定型別

return 不一定要指定型別，如果沒指定的話，就會是以 `()` 來代替

```
fn calculator(a: i32, b: i32) -> i32 {
    let result = (a + b) * b;

    println!("{}", result)
}

fn main() {
    calculator(10, 2);
}

結果： 24
```


### Block

接下來我們就來談談 Rust 的作用域吧！

每個 block 都是獨立的作用域，我們所定義的變數只會存在於產出他的 block 中，

function 跟 {} 都算是 block

讓我們來看看範例吧

在 {} ，character 被重新賦予值了，所以在 {} 中的 character 就會變成 B

不過出了這個 block ，就會變回原本的 A

```
fn main() {
    let character: char = 'A';
    {
        let character: char = 'B';
        println!("{}", character)
    }
    println!("{}", character)
}

結果：
B
A
```

現在我們知道，Rust 的作用域為一個 block

在 block 以內就是全新的世界

這邊我們就必須要提到一個名詞，叫做變數遮蔽 Variable Shadowing

當我們進入到一個新的 block 時，遇到相同變數，他會蓋掉就有的定義，指向新的定義值

出了這個 block 後，又會變成原本的值

讓我們來看看範例

進入到 block 中， character 就變成了 B 了

出了 block 後，變成原本的 A

重新定義就變成了 C

```
fn main() {
    let character: char = 'A';
    {
        let character: char = 'B';
        println!("{}", character)
    }

    println!("{}", character);
}

結果：
B
A
```

如果想要讓變數不被改，我們可以用 Variable Shadowing 來遮蔽原本的 character 的值，且該變數沒有加上 mut (表示該變數原本就不可變)，所以是不可更改的

```
fn main() {
    let character: char = 'A';
    {
        let character = character;

        character = 'B';
        println!("{}", character)
    }

    println!("{}", character)
}

結果：
error[E0384]: cannot assign twice to immutable variable `character`
 --> src/main.rs:6:9
  |
4 |         let character = character;
  |             ---------
  |             |
  |             first assignment to `character`
  |             help: consider making this binding mutable: `mut character`
5 |
6 |         character = 'B';
  |         ^^^^^^^^^^^^^^^ cannot assign twice to immutable variable

```


接下來來談談匿名函式吧

匿名函式在 Rust 中就是閉包的概念

我們先來看閉包是什麼意思

當作用域中有需要使用到外部作用域的變數時，

閉包可以向外部作用域拿取該變數進行使用

引用的方式有三種

分別為借用(再分成可變與不可變) / 轉移所有權

Rust 會依照狀況來決定要使用哪種方式

而我們之後也會提到所有權

舉例來說

我們在 block 中定義了一個匿名函式，會需要使用到 y 變數

這時候匿名函式就會去外部作用域抓 y 的值，並進行處理

```
fn main() {
    let y = 10;
    {
        let calculation = |x: i32| {
            let sum = x + y;
            return sum;
        };
        let sum = calculation(2);
        println!("the result = {}", sum)
    }
}

結果：
the result = 12
```

閉包跟函式雖然看起來很像，

不過閉包並不需要嚴格定義參數型別與回傳值，

主要是因為，閉包不像函式一樣，是公開使用的，

且通常都比較簡短，因此 Rust 可以推斷型別
