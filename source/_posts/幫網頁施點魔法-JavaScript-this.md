---
title: 幫網頁施點魔法 - JavaScript this
date: 2022-08-13 19:28:44
tags:
categories: JavaScript
---
## this到底是哪個this？傻傻分不清楚

相信大家對於this又愛又恨
好用的時候超好用
失控的時候很失控

今天就來帶大家認識一下該怎麼判斷
現在的this指的是誰

廢話不多說，直接進入主題！

---

## 先讓大家了解一下環境的差異

node.js的環境叫做 global
瀏覽器的環境叫做 window

在這兩個環境中
this會依環境的不同，而指向不一樣的地方(沒設定嚴謹模式)
但其實指向這兩個地方，都是所謂的全域範圍

如果設定嚴謹模式，this則會指向undefined

## 存在物件中的 this

this存在物件中才會發揮它的效用
一旦離開了物件
this會被賦予預設的值，也就是 window / global

---

### 1. 誰呼叫，誰就是this
#### 沒明確的呼叫者，this就是window/global

owner為呼叫者，所以this會指向owner
在owner中尋找ability並印出來

```javascript
const owner = {
  name: "Nick",
  age: 20,
  ability: "baking",
  skill: function() {
    console.log(this.ability)
  }
}
owner.skill() // baking
```

沒有明確呼叫者，就等於是window呼叫function
於是this就會指向 window / global

```javascript
function sleep() {
  let bedtime = 12
  console.log(this.bedtime)
}

sleep() // undefined
```

<br/>


### 2. 箭頭函式的 this 不會指向自己
#### 指向 window / global

箭頭函示中的this會強制綁定
儘管今天使用嚴謹模式、apply()等方法綁定
this仍然指向預設的全域

以下面這個範例來看
在箭頭函式裡面定義了變數
裡面的this還是會往外指向 window / global
並且尋找 window / global 中的變數(num1、num2)

```javascript
var num1 = 1
var num2 = 2

let numberCalculator = () => {
  let num1 = 3
  let num2 = 4
  console.log(this.num1 + this.num2)
}

numberCalculator() // 3
```

<br/>


### 3. 有沒有使用 new

如果使用new來產生新的物件，this就會指向空物件，
而這個空物件的__proto__會引領指向父層的prototype，
this就會指向父層的參數

這樣講有點難以理解，讓我們來點範例！
this會依__proto__指向父層(function)的name以及variety
this就會指向name及variety這兩個的值

神奇的是，用new產生的物件
不需要特別寫return，就會直接把this回傳過去！

```javascript
const cat = new function (name, variety) {
  this.name = "kitty"
  this.variety = "yellow"
}
cat // {name: 'kitty', variety: 'yellow'}
```


<br/>


### 4. 有沒有使用apply()、call()、bind()

使用上面三個方法時，this會指向這三個方法的參數，而不是物件的參數
其中apply()以及bind()會有限制用法

從下面例子來看，studentA、studentB為apply()及call()的參數，而非function的參數

```javascript
const teacher = {
  name: "Ray",
  age: 45,
  ability: "reading ",
  skill: function(){
    console.log(this.ability += "& conversation")
  }
}

const studentA = {
  name: "Ray",
  age: 25,
  ability: "listening "
}

const studentB = {
  name: "Any",
  age: 20,
  ability: "writing "
}

// 老師今天學習了會話，然後要教給學生
teacher.skill() // reading & conversation
teacher.skill.apply(studentA) // listening & conversation
teacher.skill.call(studentB) // writing & conversation
```

而bind比較特別，會回傳一個function，所以要記得呼叫它

```javascript
const teacher = {
  name: "Ray",
  age: 45,
  ability: "reading ",
  skill: function(){
    console.log(this.ability += "& conversation")
  }
}

const studentA = {
  name: "Ray",
  age: 45,
  ability: "listening "
}

teacher.skill.bind(studentA)() // listening & conversation
```

如果要在apple()、call()後面加上function的參數，需要寫在第二個位置
其中apply()需要使用陣列的方式

```javascript
const num = {
  type: "number"
}

function add (a, b){
  console.log(`${this.type} = ${a} + ${b}`)
}


add.apply(num, [3, 4]) // number = 3 + 4
add.call(num, 3, 4) // number = 3 + 4
```

<br/>


### 5. 是否使用嚴謹模式

嚴謹模式是ES6新增的功能
只要標注"use_strict"
this就不會隨便指向 window / global

也可以針對單一物件做設定

```javascript

function receipt() {
  "use strict"
  console.log(this)
}

receipt() // undefined

```

## this綁定原則

依照上述五種狀況做個總結，this可分為幾個綁定原則

1. 預設綁定(default binding)
  依照環境指向不一樣的地方
    ▶ 嚴謹模式指向undefined
    ▶ 沒有呼叫者的情況，會指向預設的全域(window / global)

2. 隱含式綁定(implicit binding)
  即便是在全域範圍，有明確呼叫者的情況，this就會指向呼叫者，而非全域

3. 顯式綁定(explicit binding)
  透過apply()、call()、bind()直接綁定this的值

4. new 綁定
  new出來的物件，this會指向父層的參數