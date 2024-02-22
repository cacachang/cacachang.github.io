---
title: 從0開始刻 淺談 Rails 的運作魔法 Day02 - Rack
date: 2022-08-16 02:21:25
tags:
categories: Rebuild Rails

---
## Rack是什麼？

Rack 就像 Ruby 與 Web Browser 之間的溝通橋樑
使用簡單的方法來傳遞 HTTP request
HTTP 收到 request 後會傳回應給我們

Rack 有固定的規格，它需要有個可以回應 call 方法的物件

以下 Ruby 程式碼來說
call 的參數是一個 hash
這個 hash 包含了 CGI-like 環境的變數
回傳為陣列，分別為 HTTP 狀態、HTTP header、Body

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

當我們跑 Rack 時，會啟用一個叫做 WEBrick 的伺服器
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

📃 web 小辭典

▶  CGI: 是一種標準介面程式，讓網頁跟 www server 溝通，讓使用者能跟網頁互動

▶  WEBrick：屬於 Ruby library，提供簡單的 HTTP server


</br>


## Rack 在 Rails 中的角色

Rack 是一種 Middleware，是框架與伺服器之間的 API
Rails.application 是屬於 Rack 應用程式物件
與 Rack 相容的 web server 都應該要採用 Rails.application 來運作、執行

### rails server

Rails::Server 繼承 Rack::Server特性
可以用來啟動 web server

當我們輸入 rails server 時
會建立 Rack::Server 物件
並且開啟 server

### rackup

如果不想用 rails server 啟動伺服器
可改用 rackup
不過你得先有 config.ru 的設定檔(.ru 就是 rackup 的縮寫)

```shell=
# 開啟 server
rackup config.ru 
```

#### Rack 並不是只拿來啟動伺服器

Rack 這麼厲害，怎麼可能只會啟動伺服器
Rails 中許多應用程式都是使用 Rack Middleware 來操作

不過這些應用程式運作的對象也都跟瀏覽器脫離不了關係

這邊來提幾個應用程式

Rack::Sendfile 👉 用標準化的資料格式([X-Sendfile](https://www.nginx.com/resources/wiki/start/topics/examples/xsendfile/))設定 Header

Rack::Lock 👉 設定應用程式是否可以多工作業([Mutex](https://zh.wikipedia.org/zh-tw/%E4%BA%92%E6%96%A5%E9%94%81))

Rack::MethodOverride 👉 有設定 params[:_method]，就可以重新設定方法

其他應用程式詳細資料可以參照 [Rails 官方網站](https://guides.rubyonrails.org/rails_on_rack.html)



## Rack Middleware 運作

採用 pipeline 方式(管線化)
用這種方式運作會讓效率大幅提升
讓我們來看一下他的運作流程

![](https://i.imgur.com/LbGVuOM.jpg)


Rack 會依照不同類型請求來區分
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

Rack 是一種 Middleware，主要負責處理跟 HTTP request 有關的任務
Rake 是 Rails 中的 task 執行者，讀取 Rakefile 會開始執行 task

遇到這兩兄弟要記得拿放大鏡看清楚🧐
今天就先介紹到這邊哩 😉