---
title: WEB 資料放哪裡？資料們的家 - Database資料庫 
date: 2022-08-20 02:01:17
tags:
description: 關聯式資料庫與非關聯式資料庫有什麼不一樣？規劃好資料庫，效率沒煩惱
categories: 資料們的家 - 資料庫
---
# Database 資料庫

顧名思義就是存放資料的地方
資料庫會依照資料分成許多表格(table)
這樣說有點抽象
我們用點生活例子：

資料庫其實就像ikea的倉庫賣場
倉庫賣場每一道就像表格
依照家具特性分門別類

</br>

## 資料庫的分類

資料庫可分為關聯式資料庫以及非關聯式資料庫

</br>

### 關連式資料庫( RDBMS )

由許多 Table 組成，Table 與 Table 之間可以建立關聯
使用 SQL 語法與資料庫進行溝通

另外，RDBMS 須符合ACID的規定：

不可分割性(Atomic):所有操作都要完整的執行，成功就是整筆成功寫入(commit)，失敗就是整筆沒有寫入(Rollback)
一致性 (Consistent):寫入的資料必須符合格式、關聯、以及驗證條件
獨立性 (Isolated):當多個操作一起進行時，獨立性可確保每個操作獨立進行，防止資料交互感染
耐用性 (Durable):操作完的資料會永久保存，所以刪除就是直接刪除、修改過的資料也無法還原

常見的關聯式資料庫有 MySQL、PostgreSQL

</br>

### 非關聯式資料庫( NOSQL )

與關聯式資料庫有很大的不同
非關聯式資料庫並沒有 Table、關聯這些東西
資料庫由 collection 組成
而 collection 的每筆資料都是一個 document
而資料會分散存在雲端系統

資料量很大的時候很適合使用非關聯式資料庫

常見的 NOSQL 有 MongoDB、Redis、Cassandra

</br>

## DBMS 資料庫管理系統

資料庫管理系統可以直接操作資料庫
我們常聽到的 MySQL、PostgreSQL
會依照指令針對資料庫的資料做
新增（C）、讀取（R）、更新（U）、刪除（D）的動作

就像ikea的庫存管理系統會針對倉庫進行
新增家具（C）、讀取家具數量（R）、更新庫存(U)、刪除已賣出的家具(D)

</br>

## Model - 我們與資料庫之間的翻譯蒟蒻 ( MVC 架構中)

資料庫的指令跟一般程式長得不太一樣
當我們沒特地去學習
就不會知道要怎麼請資料庫幫我們撈出資料、運用資料
這時候，Model 就會是你的最佳翻譯器
跟 Model 下達程式指令，
他就會幫我們去跟資料庫溝通，
並且取出資料

</br>

## 資料型態

資料表中的 table 大多會支援下列格式：

![](https://i.imgur.com/6CwcmJd.jpg)

</br>

## Primary key

資料庫給表格的獨特標誌
有點像是發身分證一樣
內容要具備唯一不重複，不能是 null
而每個 table 只能有一個 primary key

</br>

## Foreign key

在關聯式資料庫中很常看到
foreign key 像是 table 中的外交官
負責建立與其他 table 的關聯

</br>

## 資料庫正規化 normalization

可以把資料庫正規化想像成把資料庫裡的資料做精簡
目的在於降低資料的重複性以及相依性
減少資料庫的浪費，以及運算中不必要的麻煩

正規化分成好幾個階段，讓我們來看一下每個階段該做的事情

</br>

### 第一正規化

排除重複的資料

讓我們來看一下例子
price、quantity、store 這幾個欄位不符合第一正規化的標準

| id |  product  |   price  |  quantity | store
| ---|  ----  | -------- | -------- | -------- |
| 1  |  chair | 500, 450 |    100,90,120    | taipei, taichung, kaohsiung |
| 2  |  arm chair  | 2,000, 1,800 |    80,200   |taipei, taichung |
| 3  |  couch  | 3,000, 2,700 |   60,150    | taichung, kaohsiung |

所以我們把它拆開來
讓每個內容都有自己的家
這樣看起來是不是清楚多了呢？

| id |  product  |   price  |   best price  |  quantity | store
| ---|  ----  | -------- | -------- | -------- |-------- |
| 1  |  chair |   500    |     450  |    100   | taipei |
| 1  |  chair |   500    |     450  |    90   |taichung|
| 1  |  chair |   500    |     450  |    120   |kaohsiung |
| 2  |arm chair|  2,000  |  1,800   |    80    |taipei |
| 2  |arm chair|  2,000  |  1,800   |    200   |taichung |
| 3  |  couch  | 3,000   |  2,700   |   60     | taichung |
| 3  |  couch  | 3,000   |  2,700   |   150     | kaohsiung |

</br>

### 第二正規化

經過第一正規化後，雖然資料變精簡，但表格還是有重複性的東西

第二正規化劃就是在減少資料的相依性
相依性是指同一個資料，會對應到他該有的值

三個分店有相同的 product 及 price，造成了資料表的資料重複

| id |  product  |   price  |   best price  |  quantity | store
| ---|  ----  | -------- | -------- | -------- |-------- |
| 1  |  chair |   500    |     450  |    100   | taipei |
| 1  |  chair |   500    |     450  |    90   |taichung|
| 1  |  chair |   500    |     450  |    120   |kaohsiung |
| 2  |arm chair|  2,000  |  1,800   |    80    |taipei |
| 2  |arm chair|  2,000  |  1,800   |    200   |taichung |
| 3  |  couch  | 3,000   |  2,700   |   60     | taichung |
| 3  |  couch  | 3,000   |  2,700   |   150     | kaohsiung |

現在就讓我們把這些值在抽出來另外做成一個表格

product 一個表格

| id |  product  |   price  |   best price  |
| ---|  ----  | -------- | -------- |
| 1  |  chair |   500    |     450  |
| 2  |arm chair|  2,000  |  1,800   |
| 3  |  couch  | 3,000   |  2,700   |

store 一個表格

| id | store    |
| ---|--------  |
| 1  | taipei   |
| 2  |taichung  |
| 3  |kaohsiung |

product_store 一個表格

| id |  product_id  |store_id|  quantity | 
| ---|  ----  | ------ | -------- | 
| 1  |  1     |   1    |  100   | 
| 1  |  1     |   2    |   90   |
| 1  |  1     |   3    |  120   |
| 2  |  2     |     1  |  80    |
| 2  |  2     |     2  |  200   |
| 3  |  3     |    2   | 60     | 
| 3  |  3     |    3   |150     | 

</br>

### 第三正規化

第三正規劃須消除「遞移相依」
遞移相依指的是表格中其中一個欄位跟 Primary key 是間接關聯的(跟其中一個 table 有關聯，再跟 Primary key 有關聯)

換句話說，第三正規化的目的在於
除了 Primary key 以外，每個欄位與每個欄位應該都是獨立不相關的

以下面這個表格來舉例，price 與 best price 存在關聯性，並且與 product 存在間接關聯

| id |  product  |   price  |   best price  |
| ---|  ----  | -------- | -------- |
| 1  |  chair |   500    |     450  |
| 2  |arm chair|  2,000  |  1,800   |
| 3  |  couch  | 3,000   |  2,700   |

所以我們要把 price 以及 best price 拆開來

| id |  product  |   price  | price_id
| ---|  ----  | -------- |-------- |
| 1  |  chair |   500    |1|
| 2  |arm chair|  2,000  |2|
| 3  |  couch  | 3,000   |3|

</br>

| id |   price_id  |   best price  |
| ---|-------- | -------- |
| 1  |  1  |     450  |
| 2  | 2   | 1,800   |
| 3  | 3   |  2,700   |

第三正規化雖然讓每個資料表更加精簡
不過在查詢的時候會變得很沒效率，要一直依照關聯性去撈下個資料表

所以在設計資料表的時候也要考量一下需求
不用每次都照著正規化走

基本的資料庫介紹就到這邊
下次我們再來看看資料庫的語法吧


參考文獻
https://zh.wikipedia.org/zh-tw/ACID
https://aws.amazon.com/tw/relational-database/
https://medium.com/@eric248655665/rdbms-vs-nosql-%E9%97%9C%E8%81%AF%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB-vs-%E9%9D%9E%E9%97%9C%E8%81%AF%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB-1423c9fbb91a
https://ithelp.ithome.com.tw/articles/10229472
