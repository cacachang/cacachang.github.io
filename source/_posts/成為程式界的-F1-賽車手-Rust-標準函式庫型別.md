---
title: 成為程式界的 F1 賽車手 - Rust 標準函式庫型別
date: 2024-09-08 22:58:58
category: Rust
---

我們將會從標准函式庫提供的型別開始講起，在了解型別以前，

想來提一個重要的東西，而且我們未來也會一直提到他 - Macro

## Macro

可以把 Macro 看成是一種函式，不過他提供的功能更廣泛，概念偏向 metaprogramming，在程式碼裡面編寫程式碼，減少重複的程式碼，並且在編譯時產生代碼

## Vector

Vector 是一種存放資料的集合型別，放置在同一個 vector 裡面的資料都是要一樣型態的

讓我們來看一下該怎麼做出 vector

如果是要產生空的 vector，我們可以用這個方式

```
fn main() {
    let collection: Vec<i8> = Vec::new();
    println!("{:?}", collection)
}
```

不過要記得產生空的，一定要給他內容的型態，不然就會噴錯誤唷！

如果不是要產生空的，一開始我們就知道內容要放什麼的話，可以用以下方式做：

```
fn main() {
    let collection = vec![1, 2, 3, 4];
    println!("{:?}", collection)
}

結果：
[1, 2, 3, 4]
```

當我們在 VS Code 中打出這段時，會自動多出了 `Vec<i32>`

會去自動判斷裡面資料的型態

但如果裡面有個型態不一樣呢？

登愣，就會噴出型態不一樣的錯誤啦！

```
fn main() {
    let collection = vec![1, 2, 3, 4, 'o'];
    println!("{:?}", collection)
}

結果：
error[E0308]: mismatched types
 --> src/main.rs:2:39
  |
2 |     let collection = vec![1, 2, 3, 4, 'o'];
  |                                       ^^^ expected integer, found `char`

For more information about this error, try `rustc --explain E0308`.
error: could not compile `hello_world` (bin "hello_world") due to previous error
```

vector 裡面的元素 index 值跟 array 一樣，從 0 開始算

要存取其中一個 index 的方式也跟 array 一樣

```
fn main() {
    let mut collection = vec![1, 2, 3, 4];
    collection.push(5);
    println!("{:?}", collection[1])
}

結果：
2
```

如果要在 vector 加入新元素，可以用 push() 方法
不過要記得在 vector 定義的時候給予 mut 讓他知道是可以更改的

```
fn main() {
    let mut collection = vec![1, 2, 3, 4];
    collection.push(5);
    println!("{:?}", collection)
}

結果：
[1, 2, 3, 4, 5]
```

如果要移除某個元素呢？ 使用 remove() 方法，不過要加的是 index 值

```
fn main() {
    let mut collection = vec![1, 2, 3, 4];
    collection.remove(0);
    println!("{:?}", collection)
}

結果：
[2, 3, 4]
```


## String

我們在高階語言，字串通常都是程式語言的核心型態，不過在 Rust 中，字串是標準函示庫提供的型態唷

所以在產出 String 的時候是需要用方法去做的

今天如果要做出一個空字串，我們可以用下列這個方法做

```
fn main() {
    let my_name = String::new();

    println!("{:?}", my_name)
}

結果：
""
```

如果已經知道字串要有哪些內容了：

```
fn main() {
    let my_name = String::from("Ning");
    println!("{:?}", my_name)
}

結果：
"Ning"
```

假設我們今天要對字串增加內容呢？

使用 push_str 方法

```
fn main() {
    let mut my_name = String::from("Ning");
    my_name.push_str(" Chang");
    println!("{:?}", my_name)
}

結果：
"Ning Chang"
```

如果想要取出字串某個個單字呢？

使用字串切片

```
fn main() {
    let mut my_name = String::from("Ning");
    my_name.push_str(" Chang");

    let name = &my_name[0..4];
    println!("{:?}", name)
}

結果：
"Ning"
```


在 Rust 中，標准函式庫提供了兩種 hash 格式，分別是 HashMap 及 HashSet 

兩者差異在哪呢？

## HashMap

就像是我們在高階語言的 hash 型態，有一組組 key 跟 value

要使用需要先導入 HashMap 模組

```
fn main() {
    use std::collections::HashMap;

    let collection: HashMap<String, i8> = HashMap::new();

    println!("{:?}", collection)
}

結果：
{}
```

如果要加入元素的話，我們可以用 `insert`

```
fn main() {
    use std::collections::HashMap;

    let mut collection: HashMap<String, i8> = HashMap::new();

    collection.insert(String::from("first"), 2);

    println!("{:?}", collection)
}

結果：
{"first": 2}
```

如果要拿取其中一個元素，可以用 `get`

```
fn main() {
    use std::collections::HashMap;

    let mut collection: HashMap<i32, i8> = HashMap::new();

    collection.insert(1, 2);

    println!("{:?}", collection.get(&1))
}

結果：
Some(2)
```

要注意的是 get(&) 只接受 i32 資料格式

#### Some 、 Option

如果要移除其中一個 pair 呢? 使用 `remove` 及 i32 的值就可以囉

```
fn main() {
    use std::collections::HashMap;

    let mut collection: HashMap<i32, i8> = HashMap::new();

    collection.insert(1, 2);
    collection.insert(2, 3);

    collection.remove(&1);

    println!("{:?}", collection)
}

結果：
{2, 3}
```

如果要更新呢？ 一樣是使用 `insert`

```
fn main() {
    use std::collections::HashMap;

    let mut collection: HashMap<i32, i8> = HashMap::new();

    collection.insert(1, 2);
    collection.insert(1, 3);

    collection.insert(1, 4);

    println!("{:?}", collection)
}

結果：
{1, 4}
```

## HashSet

HashSet 也是類似高階語言中的 hash 型態，不過跟 HashMap 不一樣的是，他不允許重複的 key 跟 value，而且不能更改(下 mut 也不行)

我們在建立一個 HashSet 的時候也只能指定他的 value 是什麼

來看一下該怎麼建立吧！

```
fn main() {
    use std::collections::HashSet;

    let mut collection: HashSet<i32> = HashSet::new();

    println!("{:?}", collection)
}

結果：
{}
```

一開始就有預設值的話，我們可以這樣設定

```
fn main() {
    use std::collections::HashSet;

    let collection: HashSet<i32> = HashSet::from([2, 3, 4, 5]);

    println!("{:?}", collection)
}
```

如果要加入一個值的話，我們一樣是使用 `insert()`

```
fn main() {
    use std::collections::HashSet;

    let mut collection: HashSet<i32> = HashSet::new();

    collection.insert(2);

    println!("{:?}", collection)
}

結果：
{2}
```

移除一樣是使用 remove()

不過這邊參數是要帶 value，而非 key

```
fn main() {
    use std::collections::HashSet;

    let mut collection: HashSet<i32> = HashSet::new();

    collection.insert(2);
    collection.insert(3);
    collection.insert(4);

    collection.remove(&2);

    println!("{:?}", collection)
}
```

接下來要介紹一些 HashSet 進階用法

首先我們要來找出兩個 HashSet 中的交集，可以使用 `intersection()` 方法來做

```
fn main() {
    use std::collections::HashSet;

    let collection1: HashSet<i32> = HashSet::from([2, 3, 4, 5]);
    let collection2: HashSet<i32> = HashSet::from([1, 2, 5]);

    let collection3 = collection1.intersection(&collection2);

    println!("{:?}", collection3)
}

結果：
[2, 5]
```

如果我們要讓他們結合兩個 HashSet ，且刪除重複值，可以使用 `union()`

```
fn main() {
    use std::collections::HashSet;

    let collection1: HashSet<i32> = HashSet::from([2, 3, 4, 5]);
    let collection2: HashSet<i32> = HashSet::from([1, 2, 5]);

    let collection3 = collection1.union(&collection2);

    println!("{:?}", collection3)
}

結果：
[3, 5, 2, 4, 1]
```

** 解釋為什麼第一個是 3 (跟迭代有關)

如果要找出兩個之間不一樣的值呢？ 使用 `difference()`

```
fn main() {
    use std::collections::HashSet;

    let collection1: HashSet<i32> = HashSet::from([2, 3, 4, 5]);
    let collection2: HashSet<i32> = HashSet::from([1, 2, 5]);

    let collection3 = collection1.difference(&collection2);

    println!("{:?}", collection3)
}

結果：
[4, 3]
```


