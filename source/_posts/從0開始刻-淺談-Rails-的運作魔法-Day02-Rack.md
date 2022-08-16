---
title: 從0開始刻 淺談 Rails 的運作魔法 Day02 - Rack
date: 2022-08-16 02:21:25
tags:
---
## Rack是什麼？

Rack就像Ruby與Web Browser之間的溝通橋樑
使用簡單的方法來傳遞HTTP request
HTTP收到request後會傳回應給我們

Rack有固定的規格，它需要有個可以回應call方法的物件

以下Ruby程式碼來說
call的參數是一個hash
這個hash包含了CGI-like環境的變數
回傳為陣列，分別為HTTP狀態、HTTP header、Body

```ruby=
class Application
  def call(env)
      [
        200, 
        {'Content-Type' => 'text/html'},
        ["Hello from Ruby on Rulers!"]
      ]
  end
end

run Application.new
```

當我們跑Rack時，會啟用一個叫做WEBrick的伺服器
看到下列訊息就可以進入 http://localhost:9292/ 伺服器囉！
(之後手刻也會看到它)

```shell=
[2022-08-16 02:08:12] INFO  WEBrick 1.6.0
[2022-08-16 02:08:12] INFO  ruby 2.7.2 (2020-10-01) [arm64-darwin20]
[2022-08-16 02:08:12] INFO  WEBrick::HTTPServer#start: pid=45085 port=9292
```

看看流程圖加速理解👇

![](https://i.imgur.com/ZyYTeDl.jpg)

</br>

📃 web小辭典

▶  CGI: 是一種標準介面程式，讓網頁跟www server溝通，讓使用者能跟網頁互動

▶  WEBrick：屬於Ruby library，提供簡單的HTTP server


</br>


## Rack在Rails中的角色

Rack是一種Middleware，是框架與伺服器之間的API
Rails.application 是屬於 Rack 應用程式物件
與Rack相容的web server都應該要採用Rails.application來運作、執行

### rails server

Rails::Server 繼承 Rack::Server特性
可以用來啟動 web server

當我們輸入rails server時
會建立 Rack::Server 物件
並且開啟server

### rackup

如果不想用rails server啟動伺服器
可改用rackup
不過你得先有config.ru的設定檔(.ru就是rackup的縮寫)

```shell=
# 開啟server
rackup config.ru 
```

#### Rack並不是只拿來啟動伺服器

Rack這麼厲害，怎麼可能只會啟動伺服器
Rails中許多應用程式都是使用Rack Middleware來操作

不過這些應用程式運作的對象也都跟瀏覽器脫離不了關係

這邊來提幾個應用程式

Rack::Sendfile 👉 用標準化的資料格式([X-Sendfile](https://www.nginx.com/resources/wiki/start/topics/examples/xsendfile/))設定Header

Rack::Lock 👉 設定應用程式是否可以多工作業([Mutex](https://zh.wikipedia.org/zh-tw/%E4%BA%92%E6%96%A5%E9%94%81))

Rack::MethodOverride 👉 有設定params[:_method]，就可以重新設定方法

其他應用程式詳細資料可以參照 [Rails官方網站](https://guides.rubyonrails.org/rails_on_rack.html)



## Rack Middleware 運作

採用pipeline方式(管線化)
用這種方式運作會讓效率大幅提升
讓我們來看一下他的運作流程

![](https://i.imgur.com/LbGVuOM.jpg)


Rack會依照不同類型請求來區分
執行順序1：認證的讀取指令開始執行
執行順序2：認證的指令解碼及讀取暫存、授權的讀取指令會一起進行
⋯⋯依此類推


📌 生活小例子

以選舉流程來說
當A開始進入投票流程時，其他人的個別進度也可以同時被推進
減少等待時間、提升效率


![](https://i.imgur.com/2NCyqsQ.jpg)



## Rake、Rack 傻傻分不清

他們簡直就是雙胞胎
稍一不留神就認錯了啊

Rack 是一種Middleware，主要負責處理跟HTTP request有關的任務
Rake 是Rails中的task執行者，讀取Rakefile會開始執行task

遇到這兩兄弟要記得拿放大鏡看清楚🧐
今天就先介紹到這邊哩 😉