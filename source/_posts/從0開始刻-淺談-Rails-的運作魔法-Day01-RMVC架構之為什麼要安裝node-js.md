---
title: 從0開始刻 淺談 Rails 的運作魔法 Day01 RMVC架構之為什麼要安裝node.js
date: 2022-08-15 02:25:55
tags:
---

# 從0開始刻 淺談 Rails 的運作魔法 Day01 RMVC架構之為什麼要安裝node.js

 
開始手刻之前，我們來快速認識Rails的運作以及架構吧 🥳
直接進入主題 Go!

Rails採用MVC架構，另外還有R(Route)
MVC分別為Model、View、Controller

![](https://i.imgur.com/W0Ml2sq.jpg)


我們用生活中的例子來解說MVC會更好理解
到餐廳點餐時，會先看到菜單(View)
選好自己想吃的菜餚
會由廚師(Controller)依菜餚到食材庫(Model)挑選原料


挑好適合的原料(Model)後，廚師會開始料理(Controller)
料理完成就會送到你的眼前(View)


View主要處理呈現的畫面，以HTML為架構
Controller會跟Model拿取資料並運算，算完會回傳給View呈現
Model負責跟資料庫對話，拿取原始資料並物件化


還有一個很重要的角色，是Route!
Route就很像路標，告訴你該何去何從 ➡➡➡


### Route 

Rails的路徑表
當路徑被觸發時
Rails會依照路徑及REST判斷要去找哪個Controller及action
當我們用resources做路徑時，會做出8條路徑7個方法

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

有寫過Rails的你
應該對於erb這個檔名不陌生
erb = Embedded Ruby
可以在HTML中加入Ruby的語法

#### View helper

舉凡form_for、form_with、link_to這些表單、連結方法
都是view helper的一種
當我們把Ruby寫進HTML中
程式碼會變得複雜又不直觀
Rails的app/helper可以幫助我們整理View裡面的程式碼
將簡單的運算、程式放到helper中
讓View更簡潔 

**💎 Rails developer 必知的CoC**
erb檔所在的資料夾必須與controller呼應
view的檔案名稱需要對應到controller的action名稱
當controller運算完畢才知道要往哪邊送

以下圖為例
路徑對應的 controller為article、action為show
我們就要在view資料夾下面建立article資料夾
article的show方法會取找資料夾中的show.html.erb

![](https://i.imgur.com/wtM7ljI.png)


### Controller

接收需求、依照指令將任務辦妥是Controller的職責
就像是上述提到的廚師一樣
顧客點了哪些餐點、需要哪些原料
從View那邊接收到資訊後
會依需求去找Model要資料
要完就會開始進行運算

相較View與Model
Controller聽起來真的很辛苦呢！
不僅要會溝通、還要懂做事

#### action

controller裡的方法
通常會由routes來指定要使用哪個方法
當路徑被觸發時
action就會被執行

**💎Rails developer 必知的CoC**
Controller都要用複數來命名
檔名是snake_case
名稱是camelCase

以article來看
檔名是articles_controller.rb

![](https://i.imgur.com/5gPqtxj.png)

不過檔案中的controller是ArticlesController
```ruby=
class ArticlesController < ApplicationController
end
```


### Model

很多人常把Model跟資料庫Database搞混
不過他們是不一樣的東西！

把Model跟Datebase想像成郵局的窗口與倉庫
Model會依照Controller提出的需求
跟Database要資料

Database會把資料table傳給Model
Model會遵循ORM將這些資料物件化後給Controller
*ORM為物件關聯對映：無需使用SQL語法，用程式碼就能以物件化的方式操作資料表

![](https://i.imgur.com/iXhvW2V.jpg)

用剛剛舉的郵局例子來說
我們去郵局領過期信件時，會先跟窗口提出需求
窗口依我們需求去倉庫拿信件，蓋完章後再交給我們

**💎Rails developer 必知的CoC**
Model的class名稱為大寫、單數
table則是小寫、複數
檔名則是小寫、單數


| 檔名| Model | table |
| -----| ----- | ----- | 
|menu| Menu  | appetizers  |
|   |   | dishes  | 
|   |   | desserts  | 



### ActiveRecord

是一種設計模式，將資料物件化
存入及存取都是以物件模式進行

ActiveRecord在Rails中廣泛應用
以下簡短介紹一下各個功能
* ActiveRecord Migration:對資料庫欄位做新增、修改、刪除
* ActiveRecord Validations:對即將寫入的資料做驗證
* ActiveRecord Callbacks:物件在新增到刪除以及資料庫讀取的作業循環
* ActiveRecord Associations:資料表與資料表之間的關係
* ActiveRecord Query Interface:取得資料庫資料的語法

詳細資料可再多參照 [Rails 官方網站](https://guides.rubyonrails.org/index.html)


### Asset Pipeline

JavaScript、CSS等被稱為靜態檔案
瀏覽器從public資料夾中讀取靜態檔案，所以靜態檔案都要放在這個資料夾
不過Asset Pipeline中的Tilt會去解析檔名，解析後讓Sprockets去各個目錄中收集靜態檔案並打包，放到public資料夾中

![](https://i.imgur.com/0r9y96F.jpg)

把Pipeline的功能整理一下：

1. 整合靜態檔案，減少瀏覽器的request
2. 改用fingerprint，快速比對版本差異，減少對瀏覽器的request
3. 支援CSS、coffeeScript等語言，寫css跟js更方便



#### 📌 題外話：Rails的安裝之路既繁瑣又複雜

為什麼安裝Rails還要另外安裝node.js呢？

早期的Rails版本其實不用安裝
但後期引進了用node.js寫的webpack套件
會將所有的js檔案快速打包
所以我們就得安裝node.js

不過rails 7 有另外的打包方式
所以使用rails 7 就不用裝囉
