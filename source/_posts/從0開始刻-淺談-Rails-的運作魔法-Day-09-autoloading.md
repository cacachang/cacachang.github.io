---
title: 從0開始刻 淺談 Rails 的運作魔法 - Day 09 Automatic Loading
date: 2022-08-25 03:10:39
tags:
description: 自動載入好方便！還原 Automatic Loading 運作
---

在 Rails 中，如果沒有自動載入
我們就需要使用 require 來載入相關檔案

有了 automatic loading 
Rail 會自動載入 lib 、標準函式庫、gem等

Controller 本身是個 class
假如今天找不到 Controller 會跑出一串錯誤訊息

const_missing 可以幫我們傳遞不一樣的訊息
現在我們就來設定 const_missing 吧！

## 設定 const_missing

##### step 1 在 Object 中設定

```ruby=
# rainbow/lib/rainbow/const_missing.rb
class Object
  # 將 const_missing 定義為類別方法
  def self.const_missing(e)
    # 如果找不到方法就印出
    STDERR.puts "Missing constant: #{e.inspect}!"
  end 
end

# Bobo 有被偵測到但沒有被定義，所以會錯誤
Bobo # Missing constant: Bobo
```

既然沒被定義，那就來幫他定義
新增 bobo.rb 並在裡面定義 Bobo

##### step 2 定義 class

```ruby=
# rainbow/lib/rainbow/bobo.rb
class Bobo
  def print_bobo
    puts "Bobo!"
  end 
end
```

##### step 3 載入 class，做出實體並使用方法

回到 const_missing.rb 
把 self.const_missing 方法修改一下

```ruby=
# rainbow/lib/rainbow/const_missing.rb
class Object
  def self.const_missing(e)
    # 載入 bobo.rb
    require "./bobo"
    # 執行 Bobo
    Bobo
  end 
end

# new 一個實體，然後使用 print_bobo 方法
Bobo.new.print_bobo # Bobo!
```

📃 web小辭典

▶ STDOUT、STDERR:
當我們要將資訊傳遞出去時，會有 STDOUT 及 STDERR 這兩種格式

![](https://i.imgur.com/xej6XP5.jpg)

▶ inspect:將字串完整印出(包含跳脫字元等)

▶ const_missing: 類似 Object 的 method_missing，當找不到 constant 時，會呼叫 const_set 根據運算結果去定義值

</br>

## Controller 名稱與檔名

還記得嗎？
Rails 的 Controller 為 CamelCase
而檔名為 snake_case

我們要把 Controller 名字轉換為檔名
讓 Controller 知道要去找哪個檔名並執行

```ruby=
# rainbow/lib/rainbow/util.rb
module Rulers
  def self.to_underscore(string)
    # 字串中的::替換為/ ex.Namespace::Controller
    string.gsub(/::/, '/')
          # 首字大寫為區隔，出現第二個首字大寫時，前面加_
          .gsub(/([A-Z]+)([A-Z][a-z])/,'\1_\2')
          # 首字小寫為區隔，出現第二個首字大寫時，前面加_
          .gsub(/([a-z\d])([A-Z])/,'\1_\2')
          # 把 - 換成 _
          .tr("-", "_")
          # 轉換成小寫
          .downcase 
  end
end
```

</br>

## 自動載入設定

##### step 1 設定 :path 讓框架自動加載

```ruby=
 # quotation/Gemfile
source 'https://rubygems.org'
gem "rainbow", :path => "../rainbow"
```

##### step 2 bundle install

改了 Gemfile 記得做 bundle install
bundle exec 會依照 Gemfile 內容去自動載入

```shell=
> bundle install
```

##### step 3 uninstall rainbow

有了 bundle exec 小幫手
我們就不用一直刪除 gem、重新安裝
rainbow 就先可以拿掉了

```ruby=
> gem uninstall rainbow
```


## 結合！

把我們剛剛做的 
const_missing、CamelCase 轉 snake_case 包在 Object class 中

##### step 1 在 Object 中設定方法

```ruby=
# rainbow/lib/rainbow/dependencies.rb
class Object
  def self.const_missing(e)
    # 載入 Rainbow 並轉成 snake_case
    require Rainbow.to_underscore(e.to_s)
    # const_get 會去尋找 snake_case 的檔案
    Object.const_get(e)
  end 
end
```

##### step 2 把方法們載入框架中

當 Qutotation 載入並執行時
運作流程如下

![](https://i.imgur.com/lnlUmnK.jpg)

```ruby=
# rainbow/lib/rainbow.rb 
    require "rainbow/version"
    require "rainbow/routing"
    require "rainbow/util"
    require "rainbow/dependencies"
```

##### step 3 啟動 rackup

因為我們用 bundle exec 來載入 rainbow
所以啟用 rackup 前要加 bundle exec

```shell=
bundle exec rackup -p 3001
```

http://localhost:3001/quotes/a_quote

看到畫面就成功囉！

![](https://i.imgur.com/No2YmLI.png)


參考文獻

https://rderik.com/blog/basics-of-stderr-and-stdout-on-ruby-scripts/
https://stackoverflow.com/questions/3385201/confused-about-stdin-stdout-and-stderr
https://www.geeksforgeeks.org/ruby-string-inspect-method/
https://riptutorial.com/ruby-on-rails/example/5875/filenames-and-matic 
https://guides.rubyonrails.org/autoloading_and_reloading_constants.html
https://bundler.io/v2.3/man/bundle-exec.1.html
https://www.oreilly.com/library/view/the-ruby-programming/9780596516178/ch08s09.html
https://www.rubyguides.com/2017/07/ruby-constants/