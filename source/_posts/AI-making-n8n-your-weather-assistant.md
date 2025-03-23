---
title: 讓 n8n 成為你的天氣小助理
date: 2025-03-23 18:29:16
category: AI
---

近期 AI 工具使用率越來越廣泛，市面上推出許多方便的 AI 工具

舉凡一般聊天、查詢知識、製作圖片、翻譯等，讓我們的生活更加輕鬆便利

使用 n8n 不需要有程式相關知識，想好整個流程，並且學習如何設定就可以輕鬆完成

今天我們用簡單的範例來教大家如何使用 n8n - 天氣通知

我們先來列些條件 (中間可自行列條件)

1. 溫度超過 30 度：炎熱，建議穿輕薄透氣的短袖、短褲或裙子，紫外線強，需要SPF30+防曬霜，戴帽子和太陽眼鏡，注意補充水分。
2. 溫度低於 30 度，高於 25 度：溫和，建議穿長袖衣物或薄毛衣，可考慮穿長褲，氣溫適中，適合戶外活動。
...
7. 溫度低於 5 度：非常寒冷，需要穿厚重冬衣、保暖內衣、厚外套、圍巾、手套和帽子。


## step 1. 至 n8n 官網註冊帳號

[官方帳號](https://n8n.io/features/) 註冊，註冊帳號後就會有 7 天的免費試用期

#### 進入到 dashboard ，並且點選 `Start from scratch`

![](https://imgur.com/ynwfSD6.jpg)

#### 新增第一個步驟，右邊就會出現一排小工具

![](https://imgur.com/i2GeGO7.jpg)

![](https://imgur.com/KVOacW9.jpg)

## step 2. 設定排程

每天出門前先告訴我，今天天氣如何，並且該如何應對

![](https://imgur.com/F82wwUi.jpg)

* trigger 觸發器，指的是哪些行為會觸發工作流程，就像是當我們在電商網站下單，有了 `下單這個觸發行為`，才會有後續的包裝商品、出貨等工作流程

介紹一下其他的 triggers

1. Trigger manually (手動觸發) - 這是最基本的觸發方式，手動點擊按鈕才會啟動。

2. On app event (應用事件觸發) - 當連接的第三方應用(如 Gmail、Slack 等)中發生特定事件時觸發。

3. On a schedule (定時觸發) - 根據設定的時間表自動執行，可以設置為每小時、每天、每週等特定時間運行。

4. On webhook call (Webhook 觸發) - 當這個 URL 收到請求時觸發。

5. On form submission (表單觸發) - 處理網頁表單提交的觸發器，當用戶填寫並送出 n8n 生成的表單時觸發。

6. When Executed by Another Workflow (由其他工作流執行觸發) - 一個工作流啟動另一個工作流程。

7. On chat message (聊天消息觸發) - 當在連接的聊天平台(如 Slack、Discord 等)收到特定消息或命令時啟動工作流。

8. Other ways... (其他方式) - 各種專門的觸發器，如檔案變更、RSS訂閱更新等特定場景的觸發機制。

點選 `On a schedule` 後就會有一系列的選項要填

1. Trigger interval：可選擇以日為單位、以星期為單位，這些都無法滿足你的話，可以嘗試進階客製玩法

2. Days Between Triggers：幾天觸發一次，我這邊選擇每天，所以填寫 1

3. Trigger at Hours: 可選擇幾點觸發，可以設定起床梳妝完成後， 8:00 am 或者 9:00 am 之類的

4. Trigger at Minute: 觸發的分鐘數，選 0 會在整點的時候觸發

![](https://imgur.com/6pL3DVR.jpg)

設定完之後就可以點選左上角的 `Back to canvas` 囉

這時候畫面會是這樣子

![](https://imgur.com/nB3edVc.jpg)

排成設定完後，我們就要來蒐集天氣資料拉

## step 3. 使用天氣偵測工具

為了要搜集到今天的天氣，我們必須依賴外部的天氣預報小工具

而我在這邊用的工具叫 `OpenWeatherMap` 

![](https://imgur.com/yubOkw8.jpg)

因為要的是今天的天氣，所以選擇 `Return current weather data`

![](https://imgur.com/Ic2b1Rs.jpg)

![](https://imgur.com/DwU58sD.jpg)

接著我們要去設定這個工具的 API token

![](https://imgur.com/EtKB0rX.jpg)

* API token 就像是一把鑰匙，給你權限讓你可以使用裡面的功能，為什麼需要這個 token 呢？如果沒有這種身份驗證機制，任何人都可以自由使用，而所有的使用量和相關費用都會記在我們自己的帳戶上，可能產生未預期的費用。

我們先到 [OpenWeatherMap](https://openweathermap.org/) 的網站去註冊並且拿取 API key

![](https://imgur.com/agf6iZB.jpg)

點選產生，你的 API Key 就會出現了，出現後就把這串複製起來

![](https://imgur.com/t63GLXh.jpg)

接著回到 n8n ，把這串貼上去並且存檔

![](https://imgur.com/4VuPBFg.jpg)

存檔後就可以設定城市及語系

![](https://imgur.com/lFYpIcU.jpg)

設定後可以點選右上角的 `Test step` 

如果成功的話會出現結果！（在 `main` 區塊會有溫度、體感溫度、能見度等資料）

![](https://imgur.com/fDVC3Rs.jpg)

接著點選左上方的 `Back to canvas`

這邊我們就把天氣設定完成囉

接下來要幫天氣分類一下

## step 4. 天氣分類

怎麼分類呢？我們要先分辨今天的溫度多少

所以會需要設定判斷式，我們來用最簡單且好懂的 `if`

拿出剛剛設定的條件

```
1. 溫度超過 30 度：炎熱，建議穿輕薄透氣的短袖、短褲或裙子，紫外線強，需要SPF30+防曬霜，戴帽子和太陽眼鏡，注意補充水分。
2. 溫度低於 30 度，高於 25 度：溫暖，適合穿短袖或輕薄長袖，長褲或短褲都可，紫外線開始增強，戶外活動超過30分鐘建議使用SPF15+防曬。。
3. 溫度低於 25 度，高於 20 度：溫和，建議穿長袖衣物或薄毛衣，可考慮穿長褲，氣溫適中，適合戶外活動。
4. 溫度低於 20 度，高於 15 度：涼爽，適合穿輕便外套、長袖衣物和長褲，早晚溫差大，注意增減衣物。
5. 溫度低於 15 度，高於 10 度：寒冷，建議穿保暖外套、毛衣、長褲和保暖配件，注意保暖，特別是頸部和手腳。
6. 溫度低於 10 度，高於 5 度：注意防寒保暖，避免長時間在戶外活動，預防感冒。
7. 溫度低於 5 度：非常寒冷，需要穿厚重冬衣、保暖內衣、厚外套、圍巾、手套和帽子。
```

點選 `+` 的符號

![](https://imgur.com/uFE9bf6.jpg)

在工具列表搜尋 `if` 並點選

![](https://imgur.com/rB5gO6H.jpg)

因為我們要判斷的是溫度，所以我們可以直接把 `temp` 給拉過去判斷式，並且設定低於 5

![](https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExeGVrcTlmeWZycWMzbGVrcW93bTlpc2N3MGN2Y2l0Z2R4OWI4cWp6aCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/T1UOj0XZrWBgBjzkUu/giphy.gif)

設定完成後就可以點選 `Back to canvas` 囉

這邊我們就完成了一個基本的溫度判斷式，不過我們的條件很多，待會會再示範一個，我們先完成整個流程

## step 5. 設定 Email 通知

這邊就會有點小複雜，需要去申請 Google Cloud Platform 的服務

### 先到 Google Cloud Platform

點進去[註冊](https://cloud.google.com/cloud-console?userloc_9197983-network_g)

接著來建立專案，填寫完專案名稱就可以建立專案囉

![](https://imgur.com/bJOXVMn.jpg)

![](https://imgur.com/I5MxIeK.jpg)

回到首頁，並且選取剛剛建立的專案

![](https://imgur.com/L44vhOB.jpg)

點進去後就會出現這個畫面

![](https://imgur.com/FNHp8Hz.jpg)


接著我們要來設定 OAuth 同意畫面 (這個畫面就是我們在用 Google 登入其他平台時會出現的授權畫面)

左上角的漢堡選單，點選 `OAuth 同意畫面`

![](https://imgur.com/XzV1mu9.jpg)

![](https://imgur.com/stqhb8s.jpg)

填寫專案資訊

- 應用程式名稱可自訂
- 電子郵件請填寫你自己的

![](https://imgur.com/VMZzWR6.jpg)

選擇外部（內部我們也沒辦法選）

![](https://imgur.com/wEz5cEn.jpg)

填寫你的聯絡資訊

![](https://imgur.com/j2GrUt7.jpg)

勾選同意後建立

![](https://imgur.com/A2QkXy0.jpg)

接著我們要點選建立 OAUTH 用戶端，讓 Google 知道是哪個平台要存取

![](https://imgur.com/hTBJsku.jpg)

由於我們用的是網頁應用程式 (n8n) ，所以選擇 `網頁應用程式` 

![](https://imgur.com/QfpvxpG.jpg)

接著填寫名稱

已授權的重新導向 URI 這邊，我們先將這個畫面放旁邊

![](https://imgur.com/gqCZEQl.jpg)

回到 n8n 畫面

![](https://imgur.com/zFCRPnx.jpg)

接著點選 `true` (就是溫度低於 5 度了，要發送寒冷通知了！)

沒有低於 5 度就會走 `false` 那一條

來設定 Gmail 囉！

![](https://imgur.com/fkKAREo.jpg)

選擇 `Send a message` （寄信屬於 Send a message 動作，如果有其他想嘗試的，可以看看[官方文件](https://developers.google.com/gmail/api/guides?hl=zh-tw)）

![](https://imgur.com/7cugPax.jpg)

選擇 `Credential to connect with` 並選 `Create new credential`

![](https://imgur.com/dzDJYF1.jpg)

我們這時候就會看到 `OAuth Redirect URL`，把他複製起來！

接著回到 Google Cloud Platform

![](https://imgur.com/TUnazrX.jpg)

貼上並且建立 

![](https://imgur.com/gqCZEQl.jpg)

接著點進去該 OAuth

![](https://imgur.com/gBt5JFN.jpg)

複製 `Cliend ID` 與 `Client Secret`

* Cliend ID 就有點像是登入的帳號，當應用程式跟 Google 進行存取時，會驗證是否有這個 ID
* 而 Client Secret 就像是密碼一樣

![](https://imgur.com/dfBym4a.jpg)

回到 n8n 畫面

分別貼上 `Cliend ID` 與 `Client Secret`，並且點選 `Sign in with Google`

![](https://imgur.com/KX19c9U.jpg)

選擇你的帳號

![](https://imgur.com/Go4axwZ.jpg)

接著會出現沒有權限的狀況，這是因為我們沒有設定哪個使用者可以進行測試，如果沒有這個設計，就會變成所有人都可以一直測試，使用服務，最可怕的是賬單爆表

![](https://imgur.com/s2Qfeas.jpg)

所以我們要回到 Google Cloud Platform 設定白名單

左邊選單，選擇 `目標對象`

![](https://imgur.com/sPNJkwk.jpg)

點選 `新增測試使用者`

![](https://imgur.com/iIHBeCG.jpg)

接著將 email 填上，並儲存

![](https://imgur.com/4SMf8nO.jpg)

我們再回來測試一次

![](https://imgur.com/KX19c9U.jpg)

會跳出授權的功能選單，請點選繼續

![](https://imgur.com/UTbwS9B.jpg)

這邊我會全選，有疑慮的話可選取需要的存取範圍就好，點選後繼續

![](https://imgur.com/mcfaXtL.jpg)

看到這個畫面代表連接成功囉！

![](https://imgur.com/RXRUlvJ.jpg)

接著我們就可以來測試囉

回到選單，點選兩下 Gmail

![](https://imgur.com/oi3NQUd.jpg)

填寫 `收件人` `主旨` `內容` `傳送型態`

Resource 為 Message ，可處理信件

Operation 為 Send ，負責處理寄信

Email Type 還有支援 HTML 

![](https://imgur.com/dqVpy1v.jpg)

填寫完可以點選右上角的 `Test step`

不過因為我們的條件設定低於 5 度，我當天的溫度高達 26 度

判斷式會走 false 那條，所以不會寄出任何信，

所以我們就在 if 的 false 端再加上一個判斷吧！

![](https://imgur.com/6mzQjFC.jpg)

這邊的流程跟剛剛一樣，我就不一一介紹了

條件是就可以這樣填 (為了 demo ，先填寫今天氣溫符合的條件)

![](https://imgur.com/uqdwg1g.jpg)

接著在 `true` 加入 Gmail 發送

![](https://imgur.com/6mMzY4o.jpg)

流程也都跟剛剛一樣哦，可以直接這樣設定

![](https://imgur.com/s3HagtN.jpg)

接著可以點選右上角的 `Test step`

會出現一個奇怪的警告

![](https://imgur.com/8rwZJoN.jpg)

這是因為我們沒有把 Gmail API 的功能開啟

所以我們去 Google Cloud Platform 把 Gmail API 開啟

![](https://imgur.com/j8GC34c.jpg)

![](https://imgur.com/nS80uHv.jpg)

回到 n8n 再來測試一次 `Test step`


成功就會寄信出去囉！

![](https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExenc5OWoxaGJ2MWJ3b3RneWY4N2RxNHkxbzlpbDhhYWIyeWpvZ2JqayZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/Cbbv8tj8MwmM5QoqVy/giphy.gif)

![](https://imgur.com/SGYDWtq.jpg)


剩下的天氣條件可自己嘗試玩看看唷！

如果這篇文章對你有幫助，歡迎點選 `Like`

如果在嘗試其他條件有問題時，也可以下方留言給我知道哦！

如果有想使用 n8n 來達成哪些工作流程，也歡迎留言！
