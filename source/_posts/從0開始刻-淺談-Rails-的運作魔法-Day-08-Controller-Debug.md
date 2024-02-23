---
title: 從0開始刻 淺談 Rails 的運作魔法 - Day 08 Controller Debug
date: 2022-08-24 03:09:06
category: Rebuild Rails
description: 當 exception 出現該怎麼辦？begin rescue 是你的救星！
---

今天我們來加點 debug 機制吧

## 用 Rack Environment debug

</br>

##### step 1 印出環境變數

我們在 a_quote 中把 Rack env 這個東西印出來看看

```ruby=
# quotation/app/controllers/quotes_controller.rb
class QuotesController < Rainbow::Controller
  def a_quote
    "You Only Live Once, " +
    "hang on to your dreams." +
    "\n<pre>\n#{env}\n</pre>"
  end
end
```

</br>

##### step 2 重新載入伺服器

```shell=
rackup -p 3001
```

昨天有稍微說明 Rack env 本身是個 hash
裡面會包含 HTTP Request、Rack 參數、其他資訊

當 Rack 傳送 Request 出去後
Application 會收到下面這包資料並進行處理

Rails 的 routes 會顯示 Method 對應到的 Controller#action
當 Controller 的 action 是使用 Post 方法時
Rack 會去這包資料中找 REQUEST_METHOD 是否為 Post

```json
{"rack.version"=>[1, 6], "rack.errors"=>#>>, "rack.multithread"=>true, "rack.multiprocess"=>false, "rack.run_once"=>false, "rack.url_scheme"=>"http", "SCRIPT_NAME"=>"", "QUERY_STRING"=>"", "SERVER_PROTOCOL"=>"HTTP/1.1", "SERVER_SOFTWARE"=>"puma 5.6.4 Birdie's Version", "GATEWAY_INTERFACE"=>"CGI/1.2", "REQUEST_METHOD"=>"GET", "REQUEST_PATH"=>"/quotes/a_quote", "REQUEST_URI"=>"/quotes/a_quote", "HTTP_VERSION"=>"HTTP/1.1", "HTTP_HOST"=>"localhost:3001", "HTTP_CONNECTION"=>"keep-alive", "HTTP_SEC_CH_UA"=>"\"Chromium\";v=\"104\", \" Not A;Brand\";v=\"99\", \"Google Chrome\";v=\"104\"", "HTTP_SEC_CH_UA_MOBILE"=>"?0", "HTTP_SEC_CH_UA_PLATFORM"=>"\"macOS\"", "HTTP_UPGRADE_INSECURE_REQUESTS"=>"1", "HTTP_USER_AGENT"=>"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.0.0 Safari/537.36", "HTTP_ACCEPT"=>"text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9", "HTTP_SEC_FETCH_SITE"=>"none", "HTTP_SEC_FETCH_MODE"=>"navigate", "HTTP_SEC_FETCH_USER"=>"?1", "HTTP_SEC_FETCH_DEST"=>"document", "HTTP_ACCEPT_ENCODING"=>"gzip, deflate, br", "HTTP_ACCEPT_LANGUAGE"=>"zh-TW,zh;q=0.9,en-US;q=0.8,en;q=0.7", "HTTP_COOKIE"=>"__profilin=p%3Dt; _blogger_sys_scaffold_session=maB4R8eL4Sfeld0xkxVe0pQn91E5w0%2FLb9DdnYfrDtQg4DRVGwCpe3LsSz%2FyGDQNFFxwDbAlQdyI%2BH7P7sCAH8TrYAwjB7SYf6slcU0fMiRc03pSgP%2BPBXwC6PFCCQgK%2Frt5M2IbwbOQhatObhUazUFgEsXPk%2BmjareYI%2FxARsgOdZmmp9H%2FfDuaTPQwhHwNcwXuGkvJ8qLvx6a8VwEmssu9Aw%2BN3JAIveFhl7XxoFsmHnv02S4Kg%2FVIYo3Jy2ZZXCRyJzIH0FDxB7niBvKPS66a09l8RX3PgoDPnWIvG9HZmmpc5w%3D%3D--Ism1m1sPiM07OR%2BD--6FwHTuNeoYdlhz7Vk2rdyg%3D%3D; _vote_me_session=VDzZOezckDdb9S77I8Wb7Mqmib6CIr9OYCT4G9E3v7yE2sy%2BxbStPrQjvrDuPOFje2YqMOCWAda3DyM7A8b34kQnRcBttiWK%2FRHeH4rIWiCjS8Mr5Yw9RPN0DfhXIDTraFOEGiyko27FtfbjvIa5BQeDj8BCwijKnWXu5vBEO3LQFkqVQSlQZGSk76vozIOvdhXPExkrbRuWciZql9vZtUL9vonlMtz6F7FQGlGwzj7WjNuyvpTWb3zgl%2FRD0p92eiJDTdSJkdOCmDIIy6rab6%2BM6hxuUYkP--vZyoErRnhq0RPbF2--NsjQfsXOt%2F6xfDcBRXA4qA%3D%3D; _vote_sys_handmade_session=248B0FmQ74bA3%2Fg8gMIUhZPLs73g596x%2FtQlEG9aKU3bOQR6dTuebnjEwS2sfSrHFjsAYstqh9zjeCwCPFVr7NkN3NEAMfDElTwHa3e1f%2B0VJk%2FRLxSRR2%2BFZp72MXFvBy0W3rBwzFyfm%2BN7s7Za1TaIETGeO28GmAVXo1r3F7CcavpKbVFUBFC5tpTq6YPAE5BxE7MaPnyo4L6Rs0GABDPOtH06TC53n487wu2%2BF5OeFbxGAjDKRstuixrtjRaeuvPJU%2BUmOyB6nSgJw9kaXRBrI29gVXFejmeg%2FBG39M11jA%3D%3D--xVKJk%2B0sPe3rPBtm--MJVlt07ZhwTwkL0WDcOwAA%3D%3D; _book_sys_session=RIb%2Fuk3mFHLEW5WCmeMwW244NmYFYO1k%2BU4DgxLVwvy0zamgNod5r30bGT%2BTQOriD8l9UeEuiAVGkkADw08kb2JKJ8QPmLLhlYb5PcwRJEHzJl3LwFg5FJZFPqF%2F35R%2FCgSKvZur9lBS4w4LrN7VguEZzxN1fnj5RrVtefGLPEIt%2F8f1xaJpqdOgxdrViwLtJjZgQdCbvHSlw%2F0FemFXZL%2BwVkI03KoELTQGhDqY83u0Atyw%2FlBBgaFwt8AJiur%2B4iICTxT1WqNjVat0hyZu3RQloOOkT7oZjQ%3D%3D--A1P38yGTndEohQjR--LjnB9HbsbdXK7PUAW6JjaA%3D%3D; _rich_blogger_session=8Zd%2FmvRwh%2FWi3EMkHceie8d1iH1PSzx4FnSa55WbzIqwXXp5JvAm3saGEr3KEphFsAllB7nK0%2BccOyR7hkHa1HqrKhgaOOtGn7IZJSKb0BSkp46fm8vqzJJkIkhgzFKpBKTHLAs6j9fUkSmM3O9K2cD%2B%2FLmEhQnC0MB7JPNNvU%2B0k6vthoxx7EFbZDOp9bktDyVgywg0sqDzwxJgIcgwztUDZCJTlqnda0ryLtikL%2F36lrCQs9eL%2FsgfK7w6Tzzyi%2BVw3r4psBIWO8r3kPyh0ZtkkoNTagI%2B846nmIw%3D--NyVj4ygxD0AUgUfm--5kF0ub%2BMWwnjrL2BNpTJWQ%3D%3D", "puma.request_body_wait"=>0, "SERVER_NAME"=>"localhost", "SERVER_PORT"=>"3001", "PATH_INFO"=>"/quotes/a_quote", "REMOTE_ADDR"=>"::1", "puma.socket"=>#, "rack.hijack?"=>true, "rack.hijack"=>#, "rack.input"=>#>, "rack.after_reply"=>[], "puma.config"=>#"development", :pid=>nil, :Port=>"3001", :Host=>"localhost", :AccessLog=>[], :config=>"/Users/zhangjianing/Desktop/rebuild rails/ruler3/quotations/config.ru", :binds=>["tcp://localhost:3001"], :app=>#>, @content_length=nil>>, @logger=#>>>}, @file_options={}, @default_options={:min_threads=>0, :max_threads=>5, :log_requests=>false, :debug=>false, :binds=>["tcp://0.0.0.0:9292"], :workers=>0, :silence_single_worker_warning=>false, :mode=>:http, :worker_check_interval=>5, :worker_timeout=>60, :worker_boot_timeout=>60, :worker_shutdown_timeout=>30, :worker_culling_strategy=>:youngest, :remote_address=>:socket, :tag=>"quotations", :environment=>"development", :rackup=>"config.ru", :logger=>#>, :persistent_timeout=>20, :first_data_timeout=>30, :raise_exception_on_sigterm=>true, :max_fast_inline=>10, :io_selector_backend=>:auto, :mutate_stdout_and_stderr_to_sync_on_write=>true, :Verbose=>false, :Silent=>false, :preload_app=>false}>, @plugins=#, @user_dsl=#, @options={:environment=>"development", :pid=>nil, :Port=>"3001", :Host=>"localhost", :AccessLog=>[], :config=>"/Users/zhangjianing/Desktop/rebuild rails/ruler3/quotations/config.ru", :binds=>["tcp://localhost:3001"], :app=>#>, @content_length=nil>>, @logger=#>>>}, @plugins=[]>, @file_dsl=#, @options={}, @plugins=[]>, @default_dsl=#, @options={:min_threads=>0, :max_threads=>5, :log_requests=>false, :debug=>false, :binds=>["tcp://0.0.0.0:9292"], :workers=>0, :silence_single_worker_warning=>false, :mode=>:http, :worker_check_interval=>5, :worker_timeout=>60, :worker_boot_timeout=>60, :worker_shutdown_timeout=>30, :worker_culling_strategy=>:youngest, :remote_address=>:socket, :tag=>"quotations", :environment=>"development", :rackup=>"config.ru", :logger=>#>, :persistent_timeout=>20, :first_data_timeout=>30, :raise_exception_on_sigterm=>true, :max_fast_inline=>10, :io_selector_backend=>:auto, :mutate_stdout_and_stderr_to_sync_on_write=>true, :Verbose=>false, :Silent=>false, :preload_app=>false}, @plugins=[]>>, "rack.tempfiles"=>[]}
```

</br>

## Debugging Exceptions

還記得 exception 嗎？
當運算結果出現例外情況時，就會印出錯誤訊息

這時候我們就可以用 begin / rescue / end 來解圍

</br>

##### step 1 在 a_quote 中印出環境變數

```ruby=
class QuotesController < Rainbow::Controller
  def a_quote
    "You Only Live Once, " +
      "hang on to your dreams." +
      "\n<pre>\n#{env}\n</pre>"
  end

  def exception
    raise "It's a bad one!"
  end
end
```

</br>

##### step 2 重新執行 rackup

```shell=
> rackup -p 3001
```

![](https://i.imgur.com/3Y5ZsyC.png)


在 development 時，你可以設定 ENV['RACK_ENV'] 來避免這個狀況

但其實還有更簡單的方法，就是加上 begin / rescue / end

</br>

## 額外設定錯誤

啟動 rackup 到 http://localhost:3001
有發現它壞掉了嗎？
我們用 begin / rescue / end 來修好它！

</br>

##### step 1 建立一個 404頁面

這邊採用 Rails 404頁面的 HTML

```html
<!DOCTYPE html>
<html>
  <head>
      <title>The page you were looking for doesn't exist (404)</title>
      <meta name="viewport" content="width=device-width,initial-scale=1">
      <style>
        .rails-default-error-page {
            background-color: #EFEFEF;
            color: #2E2F30;
            text-align: center;
            font-family: arial, sans-serif;
            margin: 0;
        }

        .rails-default-error-page div.dialog {
            width: 95%;
            max-width: 33em;
            margin: 4em auto 0;
        }

        .rails-default-error-page div.dialog > div {
            border: 1px solid #CCC;
            border-right-color: #999;
            border-left-color: #999;
            border-bottom-color: #BBB;
            border-top: #B00100 solid 4px;
            border-top-left-radius: 9px;
            border-top-right-radius: 9px;
            background-color: white;
            padding: 7px 12% 0;
            box-shadow: 0 3px 8px rgba(50, 50, 50, 0.17);
        }

        .rails-default-error-page h1 {
            font-size: 100%;
            color: #730E15;
            line-height: 1.5em;
        }

        .rails-default-error-page div.dialog > p {
            margin: 0 0 1em;
            padding: 1em;
            background-color: #F7F7F7;
            border: 1px solid #CCC;
            border-right-color: #999;
            border-left-color: #999;
            border-bottom-color: #999;
            border-bottom-left-radius: 4px;
            border-bottom-right-radius: 4px;
            border-top-color: #DADADA;
            color: #666;
            box-shadow: 0 3px 8px rgba(50, 50, 50, 0.17);
        }
      </style>
  </head>
  <body class="rails-default-error-page">
    <div class="dialog">
      <div>
        <h1>The page you were looking for doesn't exist.</h1>
        <p>You may have mistyped the address or the page may have moved.</p>
      </div>
      <p>If you are the application owner check the logs for more information.</p>
    </div>
  </body>
</html>
```

</br>

##### step 2 調整 call 方法

```ruby=
# rainbow/lib/rainbow.rb
module Rainbow
  class Error < StandardError; end
  # Your code goes here...
  class Application
    def call(env)
      if env['PATH_INFO'] == '/favicon.ico'
        return 
        [ 404, {'Content-Type' => 'text/html'}, [] ]
      end

      begin
        klass, act = get_controller_and_action(env)
        controller = klass.new(env)
        text = controller.send(act)
        [ 200, {'Content-Type' => 'text/html'}, [text] ]
      rescue 
        # 找不到 Controller 及 action 時執行
        # 找到 html 檔案
        errors_page = File.read(File.expand_path("../../404.html", __FILE__)) 
        # 設定 Rack
        [ 400, {'Content-Type' => 'text/html'}, [errors_page] ]
      end
    end
  end 
```

</br>

##### step 3 quotation 重啟 rackup

```shell=
> rackup -p 3001
```
進到 http://localhost:3001 出現 404 頁面就是設定成功囉

![](https://i.imgur.com/a06uNjf.png)

參考文獻
https://www.rubyguides.com/2019/06/ruby-rescue-exceptions/
https://stackoverflow.com/questions/10168436/how-do-i-turn-off-exceptions-in-a-rack-app
https://apidock.com/ruby/File/expand_path/class
https://stackoverflow.com/questions/3863781/ruby-rack-mounting-a-simple-web-server-that-reads-index-html-as-default