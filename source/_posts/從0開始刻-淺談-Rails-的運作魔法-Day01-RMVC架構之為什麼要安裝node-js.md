---
title: 從0開始刻 淺談 Rails 的運作魔法 Day01 RMVC架構之為什麼要安裝node.js
date: 2022-08-15 02:25:55
tags:
---

# 從0開始刻 淺談 Rails 的運作魔法 Day01 RMVC架構之為什麼要安裝node.js

 
開始手刻之前，我們來快速認識Rails的運作以及架構吧 🥳
直接進入主題 Go!

Rails 採用 MVC 架構，另外還有 R (Route)
MVC 分別為 Model、View、Controller

![](https://i.imgur.com/W0Ml2sq.jpg)


我們用生活中的例子來解說MVC會更好理解
到餐廳點餐時，會先看到菜單(View)
選好自己想吃的菜餚
會由廚師(Controller)依菜餚到食材庫(Model)挑選原料


挑好適合的原料(Model)後，廚師會開始料理(Controller)
料理完成就會送到你的眼前(View)


View 主要處理呈現的畫面，以 HTML 為架構
Controller 會跟 Model 拿取資料並運算，算完會回傳給 View 呈現
Model 負責跟資料庫對話，拿取原始資料並物件化


還有一個很重要的角色，是 Route !
Route 就很像路標，告訴你該何去何從 ➡➡➡


### Route 

Rails 的路徑表
當路徑被觸發時
Rails 會依照路徑及 REST 判斷要去找哪個 Controller 及 action
當我們用 resources 做路徑時，會做出8條路徑7個方法

👩‍🏫 實做小學堂
```ruby=
# rails檔案中的routes檔
resources :restaurant
```

```ruby=
# 終端機下指令
> rails routes -c restaurant

Prefix Verb   URI Pattern                     Controller#Action
    restaurants GET    /restaurants(.:format)          restaurants#index
                POST   /restaurants(.:format)          restaurants#create
 new_restaurant GET    /restaurants/new(.:format)      restaurants#new
edit_restaurant GET    /restaurants/:id/edit(.:format) restaurants#edit
     restaurant GET    /restaurants/:id(.:format)      restaurants#show
                PATCH  /restaurants/:id(.:format)      restaurants#update
                PUT    /restaurants/:id(.:format)      restaurants#update
                DELETE /restaurants/:id(.:format)      restaurants#destroy
```


### View

有寫過 Rails 的你
應該對於 erb 這個檔名不陌生
erb = Embedded Ruby
可以在 HTML 中加入 Ruby 的語法

#### View helper

舉凡 form_for、form_with、link_to 這些表單、連結方法
都是 view helper 的一種
當我們把 Ruby 寫進 HTML 中
程式碼會變得複雜又不直觀
Rails 的 app/helper 可以幫助我們整理 View 裡面的程式碼
將簡單的運算、程式放到 helper 中
讓View更簡潔 

**💎 Rails developer 必知的CoC**
erb 檔所在的資料夾必須與 Controller 呼應
view 的檔案名稱需要對應到 Controller 的 action 名稱
當 Controller 運算完畢才知道要往哪邊送

以下圖為例
路徑對應的 Controller 為 article、action 為 show
我們就要在 view 資料夾下面建立 article 資料夾
article 的 show 方法會取找資料夾中的 show.html.erb

![](https://i.imgur.com/wtM7ljI.png)


### Controller

接收需求、依照指令將任務辦妥是 Controller 的職責
就像是上述提到的廚師一樣
顧客點了哪些餐點、需要哪些原料
從 View 那邊接收到資訊後
會依需求去找 Model 要資料
要完就會開始進行運算

相較 View 與 Model
Controller 聽起來真的很辛苦呢！
不僅要會溝通、還要懂做事

#### action

Controller 裡的方法
通常會由 routes 來指定要使用哪個方法
當路徑被觸發時
action 就會被執行

**💎Rails developer 必知的CoC**
Controller 都要用複數來命名
檔名是 snake_case
名稱是 camelCase

以 article 來看
檔名是 articles_controller.rb

![](https://i.imgur.com/5gPqtxj.png)

不過檔案中的 Controller 是 ArticlesController
```ruby=
class ArticlesController < ApplicationController
end
```


### Model

很多人常把 Model 跟資料庫 Database 搞混
不過他們是不一樣的東西！

把 Model 跟 Database 想像成郵局的窗口與倉庫
Model 會依照 Controller 提出的需求
跟 Database 要資料

Database 會把資料 table 傳給 Model
Model 會遵循 ORM 將這些資料物件化後給 Controller
* ORM 為物件關聯對映：無需使用 SQL 語法，用程式碼就能以物件化的方式操作資料表

![](https://i.imgur.com/iXhvW2V.jpg)

用剛剛舉的郵局例子來說
我們去郵局領過期信件時，會先跟窗口提出需求
窗口依我們需求去倉庫拿信件，蓋完章後再交給我們

**💎Rails developer 必知的CoC**
Model 的 class 名稱為大寫、單數
table 則是小寫、複數
檔名則是小寫、單數


| 檔名| Model | table |
| -----| ----- | ----- | 
|menu| Menu  | appetizers  |
|   |   | dishes  | 
|   |   | desserts  | 



### ActiveRecord

是一種設計模式，將資料物件化
存入及存取都是以物件模式進行

ActiveRecord 在 Rails 中廣泛應用
以下簡短介紹一下各個功能
* ActiveRecord Migration :對資料庫欄位做新增、修改、刪除
* ActiveRecord Validations :對即將寫入的資料做驗證
* ActiveRecord Callbacks :物件在新增到刪除以及資料庫讀取的作業循環
* ActiveRecord Associations :資料表與資料表之間的關係
* ActiveRecord Query Interface :取得資料庫資料的語法

詳細資料可再多參照 [Rails 官方網站](https://guides.rubyonrails.org/index.html)


### Asset Pipeline

JavaScript、CSS 等被稱為靜態檔案
瀏覽器從 public 資料夾中讀取靜態檔案，所以靜態檔案都要放在這個資料夾
不過 Asset Pipeline 中的 Tilt 會去解析檔名，解析後讓 Sprockets 去各個目錄中收集靜態檔案並打包，放到 public 資料夾中

![](https://i.imgur.com/0r9y96F.jpg)

把Pipeline的功能整理一下：

1. 整合靜態檔案，減少瀏覽器的 request
2. 改用 fingerprint，快速比對版本差異，減少對瀏覽器的 request
3. 支援 CSS、coffeeScript 等語言，寫 css 跟 js 更方便



#### 📌 題外話：Rails 的安裝之路既繁瑣又複雜

為什麼安裝 Rails 還要另外安裝 node.js 呢？

早期的 Rails 版本其實不用安裝
但後期引進了用 node.js 寫的 webpack 套件
會將所有的 js 檔案快速打包
所以我們就得安裝 node.js

不過 rails 7 有另外的打包方式
所以使用 rails 7 就不用裝囉
