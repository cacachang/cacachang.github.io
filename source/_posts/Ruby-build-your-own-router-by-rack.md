---
title: Ruby-用 Rack 做出自己的 Router
date: 2024-04-14 16:39:15
category: Ruby
tags:
---

在之前 Rebuild Rails 時有介紹到 [Rack](https://ninglab.com/%E5%BE%9E0%E9%96%8B%E5%A7%8B%E5%88%BB-%E6%B7%BA%E8%AB%87-Rails-%E7%9A%84%E9%81%8B%E4%BD%9C%E9%AD%94%E6%B3%95-Day02-Rack/) 

對於 Rack 更精準的解釋是， 
Rack 本身是個 Ruby 與 framework 的規範，
符合規範的 framework 稱作是 `Rack application` ，會將程式碼處理成 Rack 規範認可的格式(也就是 object ，且可以 call) 給 WebServer

而 `Rack` 這個套件，可以透過 `rackup` 指令產生介面，讓 Rack application 運行在支援的 Server 上

當我們用純 Ruby 來寫程式，要讓用戶連上我們的服務時，要怎麼樣才可以讓程式連上 Server ？

以下來示範一段程式碼

```
require 'webrick'

# 建立 WEBrick HTTP 伺服器
server = WEBrick::HTTPServer.new(Port: 3000)

# 設定 Route 和處理請求
server.mount_proc '/' do |req, res|
  res.body = 'Hello! Rack Practice'
end

# 設定伺服器需要停止的狀況
trap 'INT' do server.shutdown end

# 啟動伺服器
server.start
```

這樣可以讓程式碼連上 Server ，不過除了要手動一個一個寫 Route 以外，還要額外設定回應的 Body

而且，如果今天要回應的是一個 `html.erb` 檔案或者需要執行到 `rb` 檔呢？

需要寫一段程式碼去讀取檔案，或者 `require` 檔案並且渲染

```
require 'webrick'

server = WEBrick::HTTPServer.new(Port: 3000)

server.mount_proc '/' do |req, res|
  res.body = File.open('public/index.html.erb', File::RDONLY)
end

trap 'INT' do server.shutdown end
server.start
```

其實這樣是蠻麻煩的

Ruby 的框架大多遵守 Rack 的規範，讓開發者可以輕鬆處理 Route 以及 Response

為了以防大家混淆，如果講到 `Rack 規範` 我會直接使用 `Rack 規範` 來稱呼

Rack 本身支援多種 Server，舉凡開發階段使用的 `Puma` / `Thin` ，或者是正式階段使用的 `NGINX` 都有

> This specification aims to formalize the Rack protocol.

Rack 的官方文件提到，這些規範是為了要正式化 Rack 的 protocol


### 用 Rack 來刻一個 Router

[Source Code](https://github.com/cacachang/rack_practice)

檔案的架構長這樣

`public` 會放置靜態檔案
`Gemfile` `Gemfile.lock` 會放我們該專案所需要的套件
`config.ru` Rack 會需要解析這個檔案，去決定他該做什麼事

```
rack_practice
├── public
│  └──index.html.erb
├── Gemfile
├── Gemfile.lock
└── config.ru
```

#### step 1. 建立一個 Rack base 的 Application

剛有提到 Rack 會去解析 `config.ru` ，而我們會將設定寫在 `Rack::Builder.new` 的 block 之中，讓 Rack 知道，我們需要藉由這些設定來連接 Server

```
# config.ru

app = Rack::Builder.new do
end
```

* `Rack::Builder` - 用來架構一個 Rack application

> Rack::Builder provides a domain-specific language (DSL) to construct Rack applications.


#### step 2. 定義該路徑要渲染哪個檔案

接著會需要設定 Route ，我們先從最簡單的 `/` 開始
我希望 root_path 要渲染 `public/index.html.erb` 這個靜態檔案

這時候我們可以使用 `Rack::Static` 這個方法

* `Rack::Static` - 去攔截靜態檔案的 url 的 prefix 或者是 option 裡面的路徑參數，並且用 `Rack::Files` 來渲染畫面(root 參數指的是要在哪個資料夾找檔案)

> The Rack::Static middleware intercepts requests for static files (javascript files, images, stylesheets, etc) based on the url prefixes or route mappings passed in the options, and serves them using a Rack::Files object.

```
# config.ru

app = Rack::Builder.new do
  map '/' do
    use Rack::Static, urls: ['/'], root: 'public', index: 'index.html.erb'
  end
end
```

#### step 3. 設定頁面的 Response

在 run 這個 lambda 中，我們要回應 `status` / `headers` / `body`

* `Rack 規範`中有規定 Rack application 是個 Ruby 物件，並且要透過 call 來回應，需要以 env 為參數，並且回傳一個陣列，要包含著 `status` / `headers` / `body`

> A Rack application is a Ruby object (not a class) that responds to call. It takes exactly one argument, the environment and returns a non-frozen Array of exactly three values: The status, the headers, and the body.

```
# config.ru

app = Rack::Builder.new do
  map '/' do
    use Rack::Static, urls: ['/'], root: 'public', index: 'index.html.erb'
    run lambda { |_env|
      [
        200,
        {
          'Content-Type': 'text/html',
          'Cache-Control': 'no-cache'
        },
        File.open('public/index.html.erb', File::RDONLY)
      ]
    }
  end
end
```

#### step 4. 設定 Server 停止的條件

Signal 指的是信號，簡單來說，就是收到中斷訊號時( `Ctrl+C` ) 的時候中斷 Server 

```
# config.ru

Signal.trap 'INT' do
  Rack::Handler::WEBrick.shutdown
end
```

#### step 5. 啟動 Server

這邊我使用 `WEBrick` 作為 Server，指定 Port 3000 給他

```
# config.ru

Rack::Handler::WEBrick.run app, Port: 3000
```

不過這樣我們只能瀏覽 `/`，接著我們來做點進階的吧！


#### step 1. 設定 Router

檔案架構長這樣

會多兩個檔案，

`router.rb` 設定哪個 route 會回傳什麼訊息
`application.rb` 依照路徑來做出 Response (呼叫 Rack 規定的 object)

```
rack_practice
├── public
│  └──index.html.erb
├── router.rb
├── application.rb
├── Gemfile
├── Gemfile.lock
└── config.ru
```

Router 初始化的時候是個空的 {} ，這邊裝的資料會是 Response 的 body

```
# router.rb

class Router
  def initialize
    @routes = {}
  end
end
```

接著要要設定一個方法，放置渲染的訊息

```
# router.rb

class Router
  def initialize
    @routes = {}
  end

  def get(path, &block)
    @routes[path] = block
  end
end
```

最後建立一個 Response ，將對應的訊息回傳

```
# router.rb

class Router
  def initialize
    @routes = {}
  end

  def get(path, &block)
    @routes[path] = block
  end

  def build_response(path)
    handler = @routes[path] || -> { "no route found for #{path}" } 
    handler.call
  end
end
```

以上我們就做好 Router 了，接著要來路徑所對應的訊息內容

#### step 2. 設定 Application

一開始 Application 被 new 出來的時候，要做一個 Router ，

```
require "./router.rb"

class Application
  attr_reader :router

  def initialize
    @router = Router.new
  end
end
```

接著會使用到 `Router.rb` 裡面的 `get` 方法來設定對應的渲染訊息

```
require "./router.rb"

class Application
  attr_reader :router

  def initialize
    @router = Router.new

    @router.get('/') { 'Welcome to Rack Practice'}
    @router.get('/article') { 'All Articles' } 
    @router.get('/article/1') { 'First Articles' } 
  end
end
```

這時候 @router 會長這樣

每個路徑會指向一個 `Proc`，而這個 `Proc` 就會放置我們給他的訊息

```
{
  "/"=>#<Proc:0x000000010aed8ce8   
  /Users/.../rack_practice/application.rb:9>, 
  "/article"=>#<Proc:0x000000010aed8c98 
  /Users/.../rack_practice/application.rb:10>, 
  "/article/1"=>#<Proc:0x000000010aed8c48 
  /Users/.../rack_practice/application.rb:11>
}
```

設定完成後，我們就可以來做 Response 了

進入到這些畫面的時候

status 會是 200
headers 都是 "text/html"
只有 body 會不一樣，會像上一步做的，依照進入的頁面不同，渲染不同的訊息

這時候我們會需要用到 `Router.rb` 中的  `build_response` 
將 `env['PATH_INFO']` 當參數傳進去，
`build_response` 就會去判斷他是 @router 中的哪個路徑，
並依照路徑渲染訊息

```
require "./router.rb"

class Application
  attr_reader :router

  def initialize
    @router = Router.new

    @router.get('/') { 'Welcome to Rack Practice'}
    @router.get('/article') { 'All Articles' } 
    @router.get('/article/1') { 'First Articles' } 
  end

  def call(env)
    headers = {
      "Content-Type" => "text/html"
    }

    response = @router.build_response(env['PATH_INFO'])

    [200, headers, [response]]
  end
end
```

#### step 3. 設定進入點 

我們只要將 `Rack::Builder` 中改成 `run Application.new` 
在跑 `rackup` 指令的時候，就會去 new 一個 Application ，並且 run 起來囉

```
# config.ru

require 'webrick'
require './application'

app = Rack::Builder.new do
  run Application.new
end

Signal.trap 'INT' do
  Rack::Handler::WEBrick.shutdown
end

Rack::Handler::WEBrick.run app, Port: 3000
```

以上是比較陽春的 Router ，要達到 Rails 的版本還有很多東西要處理，這部分我們之後會再介紹

參考

https://github.com/rack/rack

https://tommaso.pavese.me/2016/07/26/a-rack-application-from-scratch-part-2-routes-and-controllers/

https://www.writesoftwarewell.com/build-your-own-router-in-ruby/
