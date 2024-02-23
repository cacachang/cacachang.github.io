---
title: 從0開始刻 淺談 Rails 的運作魔法 - Day 06 完整框架的基本功能
date: 2022-08-22 00:32:51
category: Rebuild Rails
description: 添加基本功能，讓架構更完整
---
建立好基本的框架及應用程式後
我們要來把框架做得更完善一點啦！

</br>

## debugging

我們先從建立 debugging 開始
這邊並不是真的寫 debug 功能

而是先用簡單的方式來模擬 debug
在之後我們會再更深入的介紹

#### step 1 在 rainbow 加入 debugging 訊息

```ruby=
module Rainbow
  class Application
    def call(env)
      # 執行時會產生一個 debug.txt檔，並在檔案中印出 debug
      `echo debug > debug.txt`
      [
        200, 
        {'Content-Type' => 'text/html'},
        ["Hello from Ruby on Rulers!"]
      ]
    end
  end 
end
```
📃 web小辭典：

▶ echo：與 puts 一樣是印出的意思


#### step 2 刪除 rainbow-0.0.1.gem

#### step 3 重新安裝套件

```shell=
> gem build rainbow.gemspec

  Successfully built RubyGem
  Name: rainbow
  Version: 0.0.1
  File: rainbow-0.0.1.gem

> gem install rainbow-0.0.1.gem
```

#### step 4 到 quotations 啟用 rackup -p 3001

啟動後就會建立一個debug.txt的檔案

![](https://i.imgur.com/Owzevzp.png)

</br>

## 建立函式庫

接下來我們要來做「陣列」
但我們不只要做出來
還要讓使用 Rainbow 框架的應用程式都能用「Array」


#### step 1 建立 Array.rb 檔

定義一個 class Array
在 Array 中定義實體方法
Array new 出來的實體物件就可以使用
(可參考小辭典中的繼承鏈)
 
```ruby=
# rainbow/lib/rainbow/array.rb
class Array
  def deeply_empty?
    # empty? 判斷是否為空
    # all? 參數都是true 就會回傳 true 否則回傳 false 或 nil
    # & 轉成 proc 物件
    # 如果是空字串，或者所有物件都為空
    empty? || all?(&:empty?)
  end 
end
```

📃 web小辭典：

▶ ActiveSupport：Rails 中的工具函式庫，像是 Ruby 的擴充工具
常見的 present?、blank?、presence 都是方法之一

▶ 繼承鏈：
Obj 為 classA new 出來的實體物件，會繼承 classA 的所有方法
而 Object 為 classA 的 superclass
所以 classA 會繼承 Object 的所有方法

可以把繼承鏈想像成種族
最上層的 Object 是 classA、Module、Obj 的遠古祖先
這些後輩都會有他留下來的基因(方法)
classA 是其中一種種族，obj 屬於 classA 種族 
因此 obj 帶有 classA 的基因(方法)
依此類推


![](https://i.imgur.com/uiUj7sQ.jpg)


#### step 2 讓所有應用程式都可以使用陣列方法

把 Array.rb require 到 rainbow.rb 中
到時候 gem 安裝的時候就會一起載入囉

```ruby=
# /rainbow/lib/rainbow.rb

require "rainbow/array"
```

#### step 3 刪除 rainbow-0.0.1.gem

#### step 4 git add .

需要在這邊把檔案放到 git 暫存區是因為

rainbow.gemspec 會叫 git 去找我們的 gem 包含哪些檔案

讓我們來看看 gemspec 檔案

```ruby=
spec.files = Dir.chdir(__dir__) do
  # 把 git 追蹤的所有檔案逐字列出，直到NUL出現，再做後面的方法(split、reject)
  `git ls-files -z`.split("\x0").reject do |f|
     (f == __FILE__) || f.match(%r{\A(?:(?:bin|test|spec|features)/|\.(?:git|travis|circleci)|appveyor)})
  end
end
```

#### step 5 重新安裝套件

```shell=
> gem build rainbow.gemspec

  Successfully built RubyGem
  Name: rainbow
  Version: 0.0.1
  File: rainbow-0.0.1.gem

> gem install rainbow-0.0.1.gem
```

#### step 4 測試 array

```ruby=
# /quotations/array.rb
a = [1, 3]
p a.sum # 4
```

</br>

## 測試檔

#### step 1 在 gemspec 檔案加入測試語法

我們目前在開發以 Rack 為基底的套件
使用 rack-test 會讓測試更方便

到 gemspec 加入下列兩個測試工具

```ruby=
spec.add_development_dependency "rack-test"
spec.add_development_dependency "minitest"
```

📃 web小辭典：

▶ rack-test: 輕量又簡單的 Rack 測試 API，可使用 cookie jar 傳送request、持續傳送 request等

▶ minitest: 輕量又快速的測試框架，提供豐富的斷言
（斷言指的是測試的判斷結果），讓測試結果簡單又好讀

忘記 gemspec 在幹嘛了嗎？[點我回想](https://ninglab.com/2022/08/16/%E5%BE%9E0%E9%96%8B%E5%A7%8B%E5%88%BB-%E6%B7%BA%E8%AB%87-Rails-%E7%9A%84%E9%81%8B%E4%BD%9C%E9%AD%94%E6%B3%95-Day03-GEM/)

#### step 2 bundle install

只要 gemspec 有做修改
就要用 bundle install
避免剛加入的套件沒加到


#### step 3 新增 test 資料夾及 test_helper

```ruby=
# /rainbow
> mkdir test
> cd test
```

```ruby=
# /rainbow/test
> test_helper.rb
```

做測試會需要用到測試工具
接下來我們就把 rack/test 跟 minitest 這兩個測試工具載入

語法解釋：

▶ LOAD_PATH 是 Ruby 用來存放檔案路徑的模組
當接收到 request 請求時
Ruby 就會去搜尋同樣名字的檔案並載入

▶ expand_path 會把相對路徑轉換成絕對路徑，當檔案有任何變動時，比較不會找錯檔案

▶ __FILE__ 會把前面參數的路徑完整秀出來


```ruby=
$LOAD_PATH.unshift File.expand_path("../../lib", __FILE__)
require "rulers"
require "rack/test"
require "minitest/autorun"
```

#### step 5 新增 application_test.rb 並加入測試語法

```ruby=
# /rainbow/test/application_test.rb

# 從資料夾中確認是否有 test_helper，而不採用 LOAD_PATH 去找
require_relative "test_helper"

class TestApp < Rulers::Application
end

class RulersAppTest < Minitest::Test
  include Rack::Test::Methods
  def app
    TestApp.new
  end
  def test_request
    get "/"
    assert last_response.ok?
    body = last_response.body
    assert body["Hello"]
  end 
end
```

#### step 6 跑測試

```ruby=
> ruby test/application_test.rb

# Running:

.

Finished in 0.019610s, 50.9944 runs/s, 101.9888 assertions/s.
1 runs, 2 assertions, 0 failures, 0 errors, 0 skips 
```

</br>

## 使用其他 server

我們目前使用的是 Ruby 內建的 server(WEBRick)
但預設的 WEBRick 並不會同步改變顯示畫面
所以每次變更都要重啟 server

我們可以用其他 Application Server 代替

Unicorn 會自動去尋找 config.ru 檔並建立起 server
我們就來用 Unicorn 吧

#### step 1 

安裝 Unicorn
```ruby=
> gem install unicorn

Fetching unicorn-6.1.0.gem
Fetching kgio-2.11.4.gem
Fetching raindrops-0.20.0.gem
Building native extensions. This could take a while...
Successfully installed kgio-2.11.4
....
3 gems installed
```


#### step 4 重啟server

```shell=
unicorn -p 3001
```
</br>

📃 web小辭典

▶ Application Server: 在傳送 HTTP request 時，用來安裝、運作、執行應用程式的服務

▶ HTTP request 流程圖

![](https://i.imgur.com/ciCvdM6.jpg)

</br>

## 排除版控

一開始安裝 rainbow 時
會送給我們一個 .gitignore 檔

.gitignore 檔 主要是收錄不想進版控的檔案
舉凡金鑰、套件的檔案(rainbow-0.0.1.gem)

```ruby=
rainbow-*.gem
```

</br>

參考文獻
https://guides.rubyonrails.org/active_support_core_extensions.html
https://git-scm.com/docs/git-ls-files
https://github.com/rack/rack-test
https://stackoverflow.com/questions/31270461/what-is-the-difference-between-cookie-and-cookiejar
https://webapps-for-beginners.rubymonstas.org/libraries/load_path.html
https://stackoverflow.com/questions/224379/what-does-file-mean-in-ruby
https://www.educba.com/what-is-application-server/