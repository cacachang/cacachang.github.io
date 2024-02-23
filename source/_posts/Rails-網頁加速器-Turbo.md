---
title: Rails 網頁加速器 - Turbo
date: 2022-11-22 00:42:46
category: Rails
description: 使用 Rails 7 ，Turbo 讓你的網頁比別人更快
---
點選按鈕，常常給你一大片空白，或者是一動也不動

這是所謂的 Turbolinks 在作祟

大家都避之唯恐不及的 Turbolinks 

Rails 團隊為什麼要把它放進來呢？

甚至在 Rails 7 升級成 Turbo ？

### 加速器

Turbolinks 監控網頁中的所有連結，將預設事件停止

並且在按下連結那一刻，Turbolinks 用 AJAX 來傳送 Request 給 HTTPS

用 AJAX 來傳送的好處是，一開始就會把所需要的東西全部載入，讓瀏覽器快速回應給使用者

而 Turbolinks 會做三件事，這部分我們稍後會提到

在 Rails 7 中，Turbolinks 已經被 Turbo Drive 取代

雖然 Turbolinks 讓網頁載入的速度更快，但卻會干擾 JS 的程式碼運作

如果想要讓 JS 跟 Turbolinks 完美配合，可以採取 Stimulus JS 作為前端的框架

說了這麼多，我們趕快進入本文章的重頭戲吧

## Turbo

Turbo 相較 Turbolinks ，會使用到更少的 JavaScript

事件監聽、動作可以使用 Turbo 來達成

甚至可以搭配 Stimulus 做到更多的互動效果

### Turbo 分成三個部分

![](https://i.imgur.com/gl6gFNA.jpg)

接下來就為大家一一介紹他們各自的功用


### Turbo Drive

其實就是以前的 Turbolinks，主要會做三件事情

在點下連結或按鈕的時候

1. 會將頁面的 Header 融合 (是融合，不是取代哦)
2. 將網頁的 Body 替換掉
3. 將網址路徑換掉

![](https://i.imgur.com/qRkScdy.jpg)

由於 AJAX 已經幫我們載入所需要的資料

所以在換頁的時候會讓你完全無感(但其實他根本沒換頁)

這個模式很像 SPA (Single Page Application)

![](https://i.imgur.com/iIg6XHj.jpg)

AJAX 會將所有的內容載入在同一頁，只要在同一個頁面渲染就可以了
( 就是 Turbolinks 做的那三件事 )


### Turbo Frame

一個頁面會區分成多個區塊，我們要做跨區的互動時，往往要寫很多個 JS 監聽事件、動作、方法等等

但 Turbo Frame 只需要用 Frame 包起來，就可以在裡面做互動

而且每個區塊可以同時傳送 request ，以往單執行緒的JS 發送 HTTP request 只能一個一個傳送，Turbo Frame 使用平行處理，時間相對少很多，且更有效率

不過需要注意的是，Turbo Frame 僅使用 GET 方法

可以應用這幾個 action ：append, prepend, replace, remove 

Turbo Frame 其實是 HTML custom element 的一種，可以把它看成一種 DOM 元素

屬性是 block ，也可以應用 CSS 及 部分 JS 語法

![](https://i.imgur.com/qmpxGPh.jpg)


### Turbo Stream

Turbo Stream 是應用 Websocket 來傳遞、更新資料

就像一個廣播器一樣，只要資料有更新，就會直接渲染到目標區塊中

跟 Turbo Frame 一樣，會依照 DOM ID 去尋找目標區塊渲染

Turbo Stream 使用 POST 發送 Request 所以可以應用的 action 很多種：append, prepend, replace, update, remove, before, and after

![](https://i.imgur.com/y3iqnRq.jpg)

