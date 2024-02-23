---
title: 幫網頁施點魔法 - JavaScript 變數提升解密
date: 2022-08-12 23:20:40
category: JavaScript
---
### 不可不知的小把戲 - hoisting

剛接觸JavaScript的朋友們，一定會被hoisting搞得霧傻傻

今天就讓我們來了解一下它到底是怎麼運作的

以下是宣告完變數，要印出變數的語法

```javascript
var name = "Amy";
console.log(name); // "Amy"
```
這時會在console中印出Amy


但，如果是這樣呢？

```javascript
console.log(name);
var name = "Amy";
```

這時候會console出什麼呢？

A: Amy
B: undefined
C: not defined

**選完往下滑看答案！**

</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>

答案：B


為什麼是B?
~~因為 hoisting作用，會把var name往上移，值留在原地
就變成undefined了！(才怪)~~

這是因為JS在run code的時候
會分為 **建立期** 以及 **執行期**

##### 建立期

只針對變數本人作業，先對你預設立場：

**step1** 找到變數 var name 

**step2** 此時不想管你的值！先定義值 undefined

~~console.log在這個時期就是邊緣人被忽略(誤)~~

##### 執行期

這時才會抓 **定義給變數的值(Amy)** 以及 **console.log**

**step1** 執行console.log，印出name的值，這時的值為undefined

**step2** 抓當初定義給name的值(Amy)

~~看吧！不要先預設立場，都已經誤會一輪了~~

這樣對於變數提升的流程更清楚了嗎？
那接下來我們再看let

#### 你可以用更好的方法定義變數

let，是ES6推出的變數定義方法
用let定義會讓變數運作的更加嚴謹

```javascript
let name = "Amy";
console.log(name); //Amy
```

無庸置疑地，「正常順序」下印出的是Amy

那我們轉換一下

```javascript
console.log(name);
let name = "Amy";
```

這樣會印出什麼？再來四個答案選項

A: Amy
B: undefined
C: not defined
D: cannot access 'name' before initialization

**選完往下滑看答案**

</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>


答案：D

答案好像太明顯了，有一個就很明顯不一樣嘛！

咦，剛剛不是說建立期的時候會幫name賦予undefined嗎？
可以不要欺騙我的感情嗎！
~~不是我欺騙你感情，是JS欺騙你感情(誤)~~

使用let定義變數，也是會經過 **建立期** 與 **執行期**
但是！但是！但是
~~我們不能隨意先入為主，程式語言也不例外！(誤)~~

let在建立期的時候，為了不要預設立場，會先將undefined這個想法隱藏起來
後來執行期的時候，抓到了值才會幫你定義給變數
聽起來是不是好棒棒？

所以運作流程會是這樣：

##### 建立期

只針對變數本人作業：

**step1** 找到變數 var name 

**step2** 不做任何定義，目前name**沒有被定義**值

##### 執行期

這時才會抓 **定義給變數的值(Amy)** 以及 **console.log**

**step1** 執行console.log，印出name的值，這時並沒有被定義值，所以會跑出未初始化前無法存取name

與let同時期出生的const，運作原理也跟let一樣哦！
