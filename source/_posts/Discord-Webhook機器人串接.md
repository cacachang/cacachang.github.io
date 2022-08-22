---
title: Discord Webhook機器人串接
date: 2022-08-12 01:57:57
tags:
categories: 其他
---

🗣️ OOO頻道 歡迎 林小姐 加入
🗣️ 陳先生你好 我們是省很大購物網

機器人式招呼已經充斥我們的生活
不曉得你是否好奇 這些機器人是怎麼設定的呢？

今天我們就來看一下 怎麼在 Discord 上串接機器人



## 1. **到dicord建立伺服器**

通常加入一個群組、頻道，機器人就會被派出來接待
所以起手式當然是 建立伺服器 ！
跟我一起來打造你的秘密頻道😎

step1 到discord左側按+ 之後選擇伺服器類型

![](https://i.imgur.com/y47V5CR.png)


step2 建立完成

![](https://i.imgur.com/WTALSKK.png)



## 2. **做一個機器人**

環境都架設好了，接下來就是主角登場的時候了吧！
機器人打造區 👉 https://discord.com/developers/applications

打造你的機器人囉

step1 新增Application

![](https://i.imgur.com/nKAzTC9.png)

step2 創建一個機器人

![](https://i.imgur.com/qFAYHzU.png)

step3 do it！

![](https://i.imgur.com/Yz8rlvK.png)

step4 幫機器人取名

![](https://i.imgur.com/Y19AHHn.png)

step5 取得Token(務必好好保存啊！)

![](https://i.imgur.com/MFs1JO2.png)


🤓 小知識時間
token：token就像是許可證，只有認證過的API及程式才能導入。
簡單來說，就是防衛機制，防止想搞事的壞程式來破壞你的伺服器，我想這種情況大家都不想遇到吧？
所以我們要發token給機器人，讓他順利進入我們的秘密頻道
＊注意！不要隨便把token給別人，不然你很有可能會引狼入室

成功做出機器人後，我們就要先讓機器人入場啦！



## 3. **產出Oauth2 URL，讓機器人加入頻道！**

step1 到URL產生頁面，選取bot

![](https://i.imgur.com/TdQ1eAF.png)

step2 選取權限及功能，選完複製網址

![](https://i.imgur.com/G8IbOyl.png)

step3 到新分頁貼上網址，授權機器人

<img src="https://i.imgur.com/uHLDYWz.png" height="50%" />

🤓 小知識時間
Oauth2：將保護的資料授權給應用程式的驗證方式，我們使用步驟來說明
step1 客戶端(我們)向資源擁有者(discord頻道)提出授權請求
step2 授權伺服器(discord)核發Token給客戶端(我們)
step3 客戶端(我們)要存取時，必須出示這個Token
step4 資源擁有者決定是否要授權



## 4. **取得discord webhook**

step1 伺服器下拉選單選伺服器設定

<img src="https://i.imgur.com/nK6AbFZ.png" height="50%" />

step2 產生新的webhook

![](https://i.imgur.com/UJpgxF4.png)

step3 複製webhook網址

![](https://i.imgur.com/zSYsr1J.png)


機器人雖然加入到我們的頻道了
但需要一個開關讓他開口

剛剛有提到webhook可以幫我們送請求給伺服器
請他呼應我們的需求

這時候，我們就要拿出webhook叫醒機器人！！



## 5. **開啟replit，開始寫程式囉～**

環境、主角都有了，接下來就是缺劇本啦！
讓我們開始來編寫劇本(程式)

我們採取axios這個套件來解析JSON，並存取裡面的資料

🤓 小知識時間
axios：一種發送HTTP request給伺服器的套件，發送請求後會回傳Promise物件

打開新的分頁 👉 https://replit.com/~
用下面的程式碼來修改

```js
// 導入axios套件
var axios = require('axios')
var data = JSON.stringify({
  "content": "TEST BOT"
})

// 發送API請求的內容，method為打入的方法，url為打入路徑，headers為打入的API是什麼格式，data則是API的參數
var config = {
  method: 'post',
  url: 'webhook的URL',
  headers: { 
    'Content-Type': 'application/json'
  },  
  data: data
}

axios(config)
.then(function (response) {
  console.log(response.data)
})
.catch(function (error) {
  console.log(error)
})
```



## 6. **啟動**

萬事俱備，只欠按下按鈕，現在就給他按下去！

按下去的瞬間會聽到discord清脆的訊息通知聲！

![](https://i.imgur.com/h8hv6Sz.png)

噹啷！機器人成功發訊息囉！