---
title: 從0開始刻 淺談 Rails 的運作魔法 - Day 07 建立 Controller
date: 2022-08-23 04:42:05
category: Rebuild Rails
description: 為框架打造運算中心 Controller
---
框架終於有一點雛形了
相關的 gem 也都安裝完成了
接下來就是做 Controller 啦！

首先，我們先將 version 升成 0.0.2

## 變更 version

##### step 1 刪除 rainbow-0.0.1.gem 

##### step 2 git add .

##### step 3 更改 version 為 0.0.2

```ruby=
module Rainbow
  VERSION = "0.0.2"
end
```

##### step 4 重新安裝套件

```shell=
> gem build rainbow.gemspec

  Successfully built RubyGem
  Name: rainbow
  Version: 0.0.2
  File: rainbow-0.0.2.gem

> gem install rainbow-0.0.2.gem

  Successfully installed rainbow-0.0.2
  Parsing documentation for rainbow-0.0.2
  Installing ri documentation for rainbow-0.0.2
  Done installing documentation for rainbow after 0 seconds
  1 gem installed
```

##### step 5 quotation 也要記得更新

```shell=
> bundle update rainbow
```


### Rainbow 設定 Controller

當請求抵達 web server 及 application server 時

Rack 會把程式碼傳過去，這時候需要知道哪個 Controller 及 action 要處理這個請求

[點我複習 Rack](https://ninglab.com/2022/08/16/%E5%BE%9E0%E9%96%8B%E5%A7%8B%E5%88%BB-%E6%B7%BA%E8%AB%87-Rails-%E7%9A%84%E9%81%8B%E4%BD%9C%E9%AD%94%E6%B3%95-Day02-Rack/)

```ruby=
# /rainbow/lib/rainbow.rb
require "rainbow/routing"

module Rainbow
  class Application

    def call(env)
      # Controller 是 class，而 class 為保留字，所以這邊用 klass 代替
      klass, act = get_controller_and_action(env)
      controller = klass.new(env)
      text = controller.send(act)
      [
        200, 
        {'Content-Type' => 'text/html'},
        # 記得這裡要換成 text
        [text]
      ]
    end

    class Controller
      # 初始化載入環境變數
      def initialize(env)
        @env = env 
      end

      def env
        @env
      end 
    end
      
  end
end
```

📃 web小辭典： 

▶ Rack Env: 一個 hash 物件，裡面含有 HTTP request、rack資訊、其他資訊(ex. WEBRick增加的資料)

![](https://i.imgur.com/D35BmW3.jpg)


▶ PATH_INFO: 以偽靜態頁面方式運作，不包含 query 參數(放置於?後面)的 URL，會告訴 Rails 要用哪個 Controller 跟 action
![](https://i.imgur.com/9zfgqw4.jpg)



### Rainbow 設定 routing

在 rainbow/lib/rainbow 新增 routing.rb
設置路徑，把路徑拆開拿到 Controller 及 action

```ruby=
# /rainbow/lib/rainbow/routing.rb

module Rainbow
  class Application
    def get_controller_and_action(env)
        
      # 把路徑分成四份，分為 Controller前 / Controller / action /剩下的
      _, cont, action, after = env["PATH_INFO"].split('/', 4)
        
      # 首字轉大寫，剩下轉小寫 ex."Quotes"
      cont = cont.capitalize
        
      # 加上 Controller ex. "QuotesController"
      cont += "Controller"
        
      # const_get 會去尋找大寫開頭的 Controller class
      [Object.const_get(cont), action]
        
    end
  end 
end

```

### 在 Quotation 中建立 controller

##### step 1 設定 Controller 及 action 

是不是很像在 Rails 中的 Controller 呢
現在我們要設定 Controller 並加入 action

```ruby=
# /quotation/app/controller/quotes_controller.rb
class QuotesController < Rainbow::Controller
  def a_quote
    "You Only Live Once, " + "hang on to your dreams."
  end
end
```

我們還沒在 Controller 加入 Rails autoloading
所以在 config/application.rb 中設定 LOAD_PATH 並把 Controller 載入

```ruby=
# quotations/config/application.rb
# 檔案路徑加入 app/controllers 讓 routes 知道要找哪個 Controller 跟 action
$LOAD_PATH << File.join(File.dirname(__FILE__),
"..", "app", "controllers")

require "quotes_controller"
```

📃 web小辭典

▶ Rails autoloading: 使用 require 載入文件時，Ruby 會去找 $LOAD_PATH 並把對應的檔名找出來，後面會更詳細的介紹 



##### step 2 刪除 rainbow-0.0.2.gem 

##### step 3 git add .

##### step 4 重新安裝套件

```shell=
> gem build rainbow.gemspec

  Successfully built RubyGem
  Name: rainbow
  Version: 0.0.2
  File: rainbow-0.0.2.gem
  
> gem install rainbow-0.0.2.gem

  Successfully installed rainbow-0.0.2
  Parsing documentation for rainbow-0.0.2
  Done installing documentation for rainbow after 0 seconds
  1 gem installed
```


### Quotation 終端機啟用 rackup

```shell=
rackup -p 3001
```

http://localhost:3001/quotes/a_quote

就會看到quote在畫面上囉

![](https://i.imgur.com/aeiiy0M.png)



### 錯誤訊息

回來看終端機，會發現出現了一行錯誤訊息

```shell=
NameError: wrong constant name Favicon.icoController
```
瀏覽器會自動讀取favicon.ico
但暫時先用避開的方法處理

```ruby=
# rulers/lib/rulers.rb /module Rainbow

class Application
  def call(env)
    if env['PATH_INFO'] == '/favicon.ico'
      return 
        [
          404,
          {'Content-Type' => 'text/html'}, 
          []
        ]
    end
        
    klass, act = get_controller_and_action(env)
    controller = klass.new(env)
    text = controller.send(act)
    [
      200, 
      {'Content-Type' => 'text/html'},
      [text]
    ] 
  end
end
```

基本的 routing、Controller、action 建立好囉
我們明天來建立 debug 功能！

參考文獻

https://stackoverflow.com/questions/4299289/what-is-the-difference-between-class-and-klass-in-ruby
https://webapps-for-beginners.rubymonstas.org/rack/rack_env.html
https://blog.csdn.net/qq_39221436/article/details/116272231
https://httpd.apache.org/docs/2.4/mod/core.html#acceptpathinfo
