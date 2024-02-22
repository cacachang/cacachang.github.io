---
title: 從0開始刻 淺談 Rails 的運作魔法 - Day 05 建立應用程式
date: 2022-08-19 01:22:41
tags:
categories: Rebuild Rails
---

昨天把初步框架架起來
今天就來做個應用程式來試看看吧！

### step1 初始化

我們要做一個 quotations 的應用程式
首先在 rainbow 外建立一個資料夾

```shell=
# 建立 quotations 資料夾
> mkdir quotations

# 進入 quotations
> cd quotations
```

```shell=
# git 初始化
> git init

Initialized empty Git repository in src/
quotitions/.git/
```

```shell=
# 建立 config 資料夾
> mkdir config

# 建立 app 資料夾
> mkdir app
```

📃 web小辭典

▶ config檔

內容包含框架載入前的程式碼設定
以及框架中的組成元件、環境設定等
可以說框架最基本的設定都放在這個檔案中了

▶ app檔

基本的架構及框架功能都會放在這裡
包含 controllers、models、views、helpers 等

</br>

### step 2 加入 gemfile

quotations 的框架是 Rainbow 
所以我們要在 gemfile 加入 rainbow 並安裝

```shell=
# 加入gemfile檔
> source 'https://rubygems.org'

> gem "rainbow" # Your gem name
```

還記得gemfile是什麼嗎？ [點我回想](https://ninglab.com/2022/08/16/%E5%BE%9E0%E9%96%8B%E5%A7%8B%E5%88%BB-%E6%B7%BA%E8%AB%87-Rails-%E7%9A%84%E9%81%8B%E4%BD%9C%E9%AD%94%E6%B3%95-Day03-GEM/)

</br>

### step 3 bundle install

建立 rainbow 套件

```shell=
> bundle install

Fetching gem metadata from https://rubygems.org/..
Resolving dependencies...
Using bundler 2.3.20
Using rainbow 3.1.1
Bundle complete! 1 Gemfile dependency, 2 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

建立完成就會跑出 Gemfile.lock (版本管理資料)

![](https://i.imgur.com/A9uA8UY.png)

</br>

### step4 建立 Rack

應用程式免不了傳送 HTTP request

所以我們要來設定 Rack

還記得 Rack 嗎？ [點我回想](https://ninglab.com/2022/08/16/%E5%BE%9E0%E9%96%8B%E5%A7%8B%E5%88%BB-%E6%B7%BA%E8%AB%87-Rails-%E7%9A%84%E9%81%8B%E4%BD%9C%E9%AD%94%E6%B3%95-Day02-Rack/)


```ruby=
# quotations/config.ru

# 用 proc 把 block 物件化，讓它可以回應 call 方法

run proc {
  [
     200, 
     {'Content-Type' => 'text/html'},
     ["Hello, world!"]
  ]
}
```

📃 web小辭典

▶ config.ru檔

Rack 基本設定，包含如何用 Rack 啟用應用程式 

</br>

### step5 啟動應用程式！

```shell=
> rackup -p 3001

* Puma version: 5.6.4 (ruby 2.7.2-p137) ("Birdie's Version")
*  Min threads: 0
*  Max threads: 5
*  Environment: development
*          PID: 83450
* Listening on http://127.0.0.1:3001
* Listening on http://[::1]:3001
```

http://localhost:3001

按進超連結就能看到 Hello World!

![](https://i.imgur.com/BQUUL3a.png)


**無法順利啟動，請確認下列幾件事情**

1. 確認PATH environment variable 有沒有包含 gem，沒有請更新
2. 返回到最初，重新安裝 Ruby 及 gem
3. 確認 gemspec 的 rack dependency 是設定在 runtime dependency 而非 development dependency

📃 web小辭典

▶ 怎麼確認 PATH environment variable 

輸入下列指令，就會出現 RubyGems 的相關資訊
如果出現沒有這個指令的訊息就是要去更新囉

```shell=
> gem env

RubyGems Environment:
  - RUBYGEMS VERSION: 3.1.4
  - RUBY VERSION: 2.7.2 (2020-10-01 patchlevel 137) [arm64-darwin20]
  - INSTALLATION DIRECTORY: /Users/zhangjianing/.rvm/gems/ruby-2.7.2
  - USER INSTALLATION DIRECTORY: /Users/zhangjianing/.gem/ruby/2.7.0
  - RUBY EXECUTABLE: /Users/zhangjianing/.rvm/rubies/ruby-2.7.2/bin/ruby
```

</br>

### step 6 框架設定 Rack

應用程式的相關設定都要放在 rainbow.rb 這個檔案中，檔案才會一起打包

然後在 Rainbow 設定 Rack (就不用每次做應用程式都要設定 Rack 了)

```ruby=
# rainbow/lib/rainbow.rb
require "rainbow/version"

module Rainbow
class Application
  def call(env)
    [
      200, 
      {'Content-Type' => 'text/html'},
      ["Hello from Ruby on Rulers!"]
    ]
  end
 end 
end
```

</br>

### step7 刪除 rainbow-0.0.1.gem

因為我們有做一些設定變更
如果不刪除、重新安裝就無法讀取最新的設定
所以記得刪除 + 重新安裝

</br>

### step8 重新安裝套件

```ruby=
> gem build rainbow.gemspec

  Successfully built RubyGem
  Name: rainbow
  Version: 0.0.1
  File: rainbow-0.0.1.gem

> gem install rainbow-0.0.1.gem
```

</br>

### step 9 設定 quotations 框架為 Rainbow

現在我們要來使用 Rainbow::Application 做設定

首先來到 config 檔案中並建立 application.rb


```shell=
# quotations/config

> touch application.rb
```

```ruby
require "rainbow"

# quotations 的 Application 繼承 Rainbow 的 Application

module Quotations
  class Application < Rainbow::Application
  end
end
```

設定完，我們再來 Gemfile 中設定

要記得加上 path，才知道要從哪裡抓 rainbow 套件

```ruby=
# quotations/Gemfile
gem "rainbow", :path => "../rainbow"
```

設定完就可以存檔進行下一步

</br>

### step 10 重新設定 quotation config

在 quotation 的 config.ru 中重新設定，把 Rack 拿掉

引入剛剛 config/application.rb 檔中的設定

```ruby=
require './config/application'
run Quotations::Application.new
```


最後在 quotation 啟動 rackup -p 3001

看到Hello from Ruby on Rulers！就是成功囉！

![](https://i.imgur.com/AEv5Qc0.png)


參考文獻
https://rebuilding-rails.com/
https://guides.rubyonrails.org/index.html