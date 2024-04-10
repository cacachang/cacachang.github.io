---
title: JavaScript - 運行背後的世界(1)
date: 2024-03-31 23:41:20
category: JavaScript
description: JavaScript 運行背後到底做了哪些事，所謂的 Execution Context 又是什麼？
---

當 JavaScript 開始執行的時候，背後會做一連串複雜的事情，我們今天就來看看這背後複雜的流程吧

![Ning-draw (4)](https://imgur.com/nsX7yj6.jpg)

JS 開始運行時，就會啟動`執行環境`

### Execution Context 執行環境

> An execution context is a specification device that is used to track the runtime evaluation of code by an ECMAScript implementation.

依照 ECMA 官方的解釋，執行環境是在程式碼執行的時候發生，並且用來追蹤程式碼的執行狀況

而執行環境又分成兩種，分別是 
`Global Execution Context`
`Function Execution Context`

當程式碼開始執行時，`Global Execution Context` 就會開始，他的作用域在程式碼開始到結束

### Global Execution Context (GEC)

又稱作全域執行環境，這個時候，會將 window 做出來，並且將 this 綁定在 window 身上

我們可以在檢查中輸入 `window` `this`，並且看他們兩個是否是一一樣

```
> window 
> this

> window === this // true
```
另外還會將變數存放到「倉庫」中，

倉庫又分成兩種，

1. LexicalEnvironment 存放 `let` / `const` / `function`
2. VariableEnvironment 存放 `var`

我們先知道有這兩個倉庫，待會再來仔細介紹這兩個倉庫

全域執行環境會一路到程式碼結束，那另一個執行環境呢？

### Function Execution Context (FEC)

又稱為函式執行環境，當程式碼遇到 Function 或 block 時，就會產生 `Function Execution Context` ，

跟全域執行環境一樣，會將 function 中的變數及 function 存到倉庫中

到現在應該對於執行環境還有一點不了解，讓我們用 Stack 來看會更清楚

來看看下面的程式碼

```
function aboutMe() {
  console.log("Hi, my name is Ning");

  function myAge() {
    console.log("I'm 18 years old");
  }

}
aboutMe();
```

當程式碼開始執行時，就像是大隊接力開始，執行到哪行程式碼，就是將控制所有權交給該 `Execution Context`

1. 開始執行時，第一棒會是 `Global Execution Context`，

2. 接著執行 `aboutMe()` ，產生一個 `Function Execution Context` ，並將所有權交給 `aboutMe()`

3. 接著在 `aboutMe()` 中執行了 `myAge()` ，因此產生一個新的 `Function Execution Context` ，並將所有權交給 `myAge()`

4. 當 `myAge()` 執行完後，`Function Execution Context` 就會消失，並且將控制權轉還交給 `aboutMe()` 

5. `aboutMe()` 執行後，`Function Execution Context` 就會消失，並將控制權轉交給 `Global Execution Context`

![Ning-draw (10)](https://imgur.com/uKucytz.jpg)

接著我們要來看每個 `Execution Context` 中做了哪些事

剛剛有提到，不管是 `Global Execution Context` 還是 `Function Execution Context` 都會將他們的變數放到倉庫中

倉庫還會分成兩種倉庫

我們先來看第一種 `Lexical Environment`


### Lexical Environment

Lexical Environment 又稱作詞彙環境，

會放置 `let` / `const` / `function` 不過還會有個地方放置父層的變數，

當執行的時候需要這個作用域沒有定義的變數，就可以往父層的變數小房間去找

因此 Lexical Environment 又分成兩個小盒子

第一個盒子是 `Environment Record`，放置 `let` / `const` / `function` 等變數

第二個盒子是 `reference to outer environment`，放置父層的變數

讓我們來看看幾個例子：


```
function aboutMe() {
  var myName = "Ning";
  console.log(`Hi, my name is ${myName}`);

  function myAge() {
    let age = 18;
    console.log(`I'm ${age} years old`);
  }

  myAge();
}

aboutMe();
```

1. 呼叫 `AboutMe()`
2. 進入到 `AboutMe` 的 `Function Execution Context`
3. 放置變數及 Function 到倉庫中， `myName` / `myAge` 
    * `myName` 是 var 變數，因此會放到 `Variable Environment` (待會會提到)
    * `myAge` 是 Function ，會放到 `Environment Record`
4. 印出 `Hi, my name is ${myName}`，由於 `Variable Environment` 有 `myName` 變數，因此印出 `Hi, my name is Ning`
5. 呼叫 `myAge()`
6. 進入 `myAge` 的 `Function Execution Context`
7. 放置變數
    * `age` 是 let 變數，因此放到 `Environment Record`
8. 印出 `I'm ${age} years old` ，這時候 `myAge` 的 `Environment Record` 有 `age` 這個變數，因此印出  `I'm 18 years old`

![Ning-draw (6)](https://imgur.com/PI2lvyO.jpg)

接著我們來看有使用到父層變數的程式碼

```
function aboutMe() {
  var myName = "Ning";
  let age = 18;
  console.log(`Hi, my name is ${myName}`);

  function myAge() {
    console.log(`I'm ${age} years old`);
  }

  myAge();
}

aboutMe();
```

1. 呼叫 `AboutMe()`
2. 進入到 `AboutMe` 的 `Function Execution Context`
3. 放置變數及 Function 到倉庫中， `myName` / `age` / `myAge` 
    * `myName` 是 var 變數，因此會放到 `Variable Environment`
    * `age` 是 let 變數，因此會放到 `Environment Record`
    * `myAge` 是 Function ，會放到 `Environment Record`
4. 印出 `Hi, my name is ${myName}`，由於 `Variable Environment` 有 `myName` 變數，因此印出 `Hi, my name is Ning`
5. 呼叫 `myAge()`
6. 進入 `myAge` 的 `Function Execution Context`
7. 沒有變數
8. 印出 `I'm ${age} years old` ，這時候 `myAge` 的 `Environment Record` 沒有 `age` 這個變數，因此往 `reference to outer environment` 找變數，在 `aboutMe` 找到了 age 是 18，因此印出 `I'm 18 years old`

要注意的是， `reference to outer environment` 只會放上一層的，如果沒有的話，就會從上一層的 `reference to outer environment` 繼續往上找

![Ning-draw (7)](https://imgur.com/xhj9KSY.jpg)

接著我們來看看 `Variable Environment`

### Variable Environment

又稱作變數環境，主要放置 `var` 變數，

讓我們來看看幾個例子

```
function aboutMe() {
  var myName = "Ning";

  function myAge() {
    console.log(`I'm ${age} years old`);
  }

  if (myName == "Ning") {
    var age = 19;
    myAge();
  }
}

aboutMe();
```

1. 執行 `aboutMe()`
2. 進入 `aboutMe` 的 `Function Execution Context` 
3. 放置變數 
    * 放置 var 變數 `myName` 到 `Variable Environment`
    * 放置 Function `myAge` 到 `Lexical Environment` 
    * 放置 var 變數 `age` 到 `Variable Environment`
6. 遇到 if 的 block ，條件符合，進入該 block 的 `Function Execution Context`
7. 由於 var 變數作用範圍僅存於 Function ，因此這邊不會放置
8. 執行 `myAge()`
9. 進入 `myAge` 的 `Function Execution Context`
10. 沒有變數，不放置變數
11. 印出 `I'm ${age} years old`，不過該 `Execution Context` 沒有任何變數，往 `reference to outer environment` 找
12. 在 `aboutMe()` 中找到了
13. 因此印出 `I'm 19 years old`

為什麼不是往 `if (myName == "Ning")` 
主要是因為 JavaScript 是採用 Lexical Scope 的方式，

> The word lexical refers to the fact that lexical scoping uses the location where a variable is declared within the source code to determine where that variable is available.

根據 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#lexical_scoping) 的解釋，Lexical Scope 會取決於該程式碼的 `所在位置`，而非程式碼的 `執行位置`

也就是說，我們要看的是 `myAge` 位在於程式碼的哪裡，並且去找到他的父層，而他的父層就是 `aboutMe`，

因此 `reference to outer environment` 這個倉庫也只會放 `aboutMe` 的變數 

![Ning-draw (9)](https://imgur.com/8xioDlA.jpg)

接下來再看一個範例

```
function aboutMe() {
  var myName = "Ning";

  function myAge() {
    console.log(`I'm ${age} years old`);
  }

  if (myName == "Ning") {
    let age = 19;
    myAge();
  }
}

aboutMe();
```

1. 執行 `aboutMe()`
2. 進入 `aboutMe` 的 `Function Execution Context` 
3. 放置變數 
    * 放置 var 變數 `myName` 到 `Variable Environment`
    * 放置 Function `myAge` 到 `Lexical Environment` 
6. 遇到 if 的 block ，進入該 block 的 `Function Execution Context`
7. 放置變數 `age` 到 `Variable Environment`
8. 執行 `myAge()`
9. 進入 `myAge` 的 `Function Execution Context`
10. 沒有變數，不放置變數
11. 印出 `I'm ${age} years old`，不過該 `Execution Context` 沒有任何變數，往 `reference to outer environment` 找
12. 在 `aboutMe` 中找不到
13. 因此印出錯誤訊息 `Uncaught ReferenceError: age is not defined`

![Ning-draw (8)](https://imgur.com/DPGG7wM.jpg)


看到這邊應該有個疑惑，為什麼 `if (myName == "Ning")` 的 var 會被放在 `aboutMe` 的 `Variable Environment` ？

依照 [MDN](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Statements/var) 的說明， var 只能被包在 `Function` 中，一旦出了 `Function` 這個作用域，就會變成 `Global` 作用域

關於 `Variable Environment` 的詳細介紹，將在下篇文章提出

參考
https://www.borderlessengineer.com/p/how-js-works-understanding-the-execution

https://www.freecodecamp.org/news/javascript-execution-context-and-hoisting/

https://www.frontendmag.com/tutorials/lexical-environment-in-javascript/

https://262.ecma-international.org/6.0/#sec-execution-contexts

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#lexical_scoping

