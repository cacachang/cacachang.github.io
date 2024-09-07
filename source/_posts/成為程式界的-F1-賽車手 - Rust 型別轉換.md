---
title: 成為程式界的 F1 賽車手 - Rust 型別轉換
date: 2024-09-07 17:17:53
category: Rust
---

### 轉換型別

在 Rust 中該怎麼轉換型別呢？

只要加上 `as` 就可以了

讓我們來看一下吧

```
fn main() {
    let price: f32 = 18.5;
    println!("{}", price);

    let modified_price = price as i8;
    println!("{}", modified_price);
}

# 結果： 18.5
# 結果： 18
```

不過如果今天要把整數轉成字呢？

他會依照 Unicode 編碼來進行轉換，不過要注意的是，只有 `u8` 能轉換成 `char` 哦，因為 Unicode 的編碼並不多，所以只適用在 `u8` 範圍

而這個數字在 Unicode 編碼沒有對應的字，就會顯示空的

```
fn main() {
    let num: u8 = 33;
    println!("{}", num);

    let character = num as char;
    println!("{}", character);
}

# 結果： 33
# 結果： !
```

雖然 Rust 可以用 `as` 方式去轉換型別，不過也不是所有的型別都可以轉換過去哦

就像我們剛剛要轉換成 `char` 時，只有 `u8` 能夠轉

### 運算式

在介紹基本型別後，我們要開始進入邏輯階段了！

首先要來認識一下運算子

在 Rust 中有幾個運算式

* 算術運算式
* 複合指定運算式
* 邏輯運算式
* 比較運算式

我們直接一個一個來介紹

#### 算術運算式

顧名思義就是加減乘除，還有多一個餘數


##### 要用加法的話，使用 `+` 

```
fn main() {
    let num1: i8 = 33;
    let num2: i8 = 20;
    let total = num1 + num2;

    println!("num1 + num2 = {}", total);
}

# 結果：num1 + num2 = 53
```

##### 減法使用 `-`

```
fn main() {
    let num1: i8 = 33;
    let num2: i8 = 20;
    let total = num1 - num2;

    println!("num1 - num2 = {}", total);
}

# 結果：num1 + num2 = 13
```

##### 乘法使用 `*`

這邊我把型別都改成了 `i32`，因為 `i8` 無法支援相乘後的結果

```
fn main() {
    let num1: i32 = 33;
    let num2: i32 = 20;

    let total = num1 * num2;

    println!("num1 * num2 = {}", total);
}

# 結果：num1 * num2 = 660
```

##### 除法使用 `/`

這邊要注意，整數相除的結果會是整數，浮點數相除的結果會是浮點數

```
fn main() {
    let num1: i8 = 33;
    let num2: i8 = 20;

    let total = num1 / num2;

    println!("num1 / num2 = {}", total);
}

# 結果：num1 / num2 = 1
```

##### 取餘數使用 `%`

```
fn main() {
    let num1: i8 = 33;
    let num2: i8 = 20;

    let total = num1 % num2;

    println!("num1 % num2 = {}", total);
}

# 結果：num1 % num2 = 13
```

以上是基本的加減乘除，相信大家到這邊應該都可以輕易理解，接下來我們就進入 `複合指定運算式`

### 複合指定運算式

在前幾天有提到，要定義變數使用 `=` 去變動，這是所謂的指定運算子

那 `複合指定運算式` 是什麼？

當我們想針對原本的數值做加減乘除，就可以使用 複合指定運算式

我們直接用舉例會比較清楚

以加法來說，假設今天有個變數 a 的值是 2 ，我希望改變他的值，讓他 +1 後變成 3，我們就可以用 `+=` 來達成

在看範例之前，我們來了解 `+=` 是怎麼解讀

`num += 2` 可以解讀成 `num = num + 2`

減乘除、求餘數同理，在運作子後面加上 `=` 即可成立囉

減乘除、求餘數會分別是(`-=` `*=` `/=` `%=`) 

(記得要加上 mut ，不然會噴錯喔！)

```
fn main() {
    let mut num1: i8 = 33;

    num1 += 2;

    println!("{}", num1);
}

# 結果：35
```

### 比較運算式

基本上就是 `>` `<` `=` 這類的比較

結果會返回一個布林值，讓我們直接來看範例

```
fn main() {
    let greater = 5 > 3;

    println!("{}", greater);
}

# 結果：true
```

會有哪些比較運算式呢？

`>` 左邊大於右邊 成立的時候返回 `true` ，反之 `false`
`>=` 左邊大於或等於右邊 成立的時候返回 `true`，反之 `false`
`<` 左邊小於右邊 成立的時候返回 `true` ，反之 `false`
`<=` 左邊小於或等於右邊 成立的時候返回 `true` ，反之 `false`
`==` 左邊等於右邊 成立的時候返回 `true` ，反之 `false`
`!=` 左邊不等於右邊 成立的時候返回 `true` ，反之 `false`

### 邏輯運算式

這邊可能就要稍微動動理解力！

會有三個邏輯運算式，我們以下詳細講解：

##### &&

`&&` 當兩邊的條件式都成立時(結果都是 true ) 即返回 true，反之 `false` 

```
fn main() {
    let greater = 5 > 3;
    let less = 6 > 4;

    # greater 跟 less 都是 true，因此 operate 也會是 true
    let operation = greater && less;

    println!("{}", operation);
}

# 結果： true
```

來稍微動個手腳，把 less 結果變成 `false`

這時候 operation 就會變成 `false`

因為 `&&` 是在兩個條件式都是 `true` 的情況下才會是 `true`

其中一個不是 `true` 或者是 第一個就是 `false` ，就會直接返回 `false`

```
fn main() {
    let greater = 5 > 3;
    let less = 6 < 4;

    let operation = greater && less;

    println!("{}", operation);
}

# 結果： false
```

##### ||

`||` 當其中一邊的條件式成立時(某一邊結果是 true) 即返回 true，反之 `false`


拿剛剛的例子來做範例，我們改一下運算子為 `||`

```
fn main() {
    let greater = 5 > 3;
    let less = 6 < 4;

    # greater 為 true ， less 為 false，以 || 來看只要有個 true，就無條件返回 true
    let operation = greater || less;

    println!("{}", operation);
}

# 結果： true
```

如果兩個結果都是 false 呢？ 就會返回 false

```
fn main() {
    let greater = 5 < 3;
    let less = 6 < 4;

    # greater 為 true ， less 為 false，以 || 來看只要有個 true，就無條件返回 true
    let operation = greater || less;

    println!("{}", operation);
}

# 結果： false
```

##### !

`!` 代表相反的意思

```
fn main() {
    let less = 6 < 4;

    # 原本的 less 為 false，加了 ! 後反轉，因此 operation 就成了 true
    let operation = !less;

    println!("{}", operation);
}

# 結果： ture
```
