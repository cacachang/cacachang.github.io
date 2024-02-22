---
title: rails 的 params 背後的運作巧思
date: 2022-12-11 00:34:30
tags:
description: params 不僅可以存到資料庫中，也可以拿來判斷，不過他背後究竟是怎麼運作的呢？
---
# Parameter

我們有時候會需要存取參數來作業，在 web application 中會有兩種參數的來源

1. query string (放在?後面)
2. 以 POST 方法傳送 request 時，data 裡面的 query 


在這之前，我們先來看一下 Rails 是怎麼去存取 parameter 的

![](https://i.imgur.com/RHP3DpY.jpg)

是的，你沒看錯，params 其實是一個方法，把 @routing_params(已塞入 controller / action) 塞進去 parameter 裡面


釐清流程後，讓我們進入正題吧！


### 基本 params

我們在做一般的 CRUD 時，常常需要存取 params 來獲得資料
```
class BlogsController < ApplicationController

  def create
    @blog = Blog.new(params[:blog])
  end
  
end
```

params 由一個或多個 key 及 value 組成

```
{"authenticity_token"=>"[FILTERED]", "blog"=>{"title"=>"what is parameter", "content"=>"hello", "author"=>"Ning"}, "commit"=>"送出"}
```

很像 hash 物件，但他其實不是 hash 物件，而是 params 物件

value 的型態只會是 string 

key 的型態就蠻多元的，可以是 string, symbol, nil 等等
([可參考 Rails 官方文件](https://guides.rubyonrails.org/action_controller_overview.html#parameters))


### 從 route 傳進來的 params

我們可以藉由 route 的設定，來定義 params 的 key

```
get '/blogs/:id', to: 'blogs#show'

# 假設網址為 hostname/blogs/2 ，此時 params 就會是 { "id" => "2" }

get '/blogs/:status', to: 'blogs#show'

# 預設參數 :id 也可以改成你想要的 key 名稱
# 網址為 hostname/blogs/active ， 此時 params 就會是 { "status" => "active" }
```

### 沒有准許是不能隨便存進資料庫的

我們常會透過 form 表單來傳送參數給 controller / action

controller / action 將資料處理完畢後，會存進資料庫中

如果沒有做資料驗證，很難防止有心人士破壞你的資料庫

因此 Rails 會去對這包資料進行身份驗證，怎麼驗證呢？ 

他會透過一組隨機的 token 來辨識

假設是用一般的 form 表單，我們就必須要將 authenticity_token 藏在一個欄位中

Rails 驗證後才會讓這包資料通過

如果驗證不通過，就會噴 ActiveModel::ForbiddenAttributesError 的訊息

```
<form action="/blogs" method="post">
  <input type="hidden" accept-charset="UTF-8" name="authenticity_token" value="<%= form_authenticity_token %>" autocomplete="off">
  <label for="">title</label>
  <input name="blog[:title]"></input>
  <label for="">content</label>
  <input name="blog[:content]"></input>
  <label for="">author</label>
  <input name="blog[:author]"></input>
  
  <button value="submit">送出</button>
</form>
```
如果用 form_helper 就會方便很多

```
<%= form_with model: @blog, url: blogs_path do |b| %>
  <%= b.label :title %>
  <%= b.text_field :title %>

  <%= b.label :content %>
  <%= b.text_field :content %>

  <%= b.label :author %>
  <%= b.text_field :author %>

  <%= b.submit '送出' %>
<% end %>
```

讓我們來看看 html 長怎麼樣

value 這邊就會隨機做出一個 token

Rails 就會去辨識這個 token 而決定是否要將資料存入資料庫中
```
<form action="/blogs" method="post">
  <input type="hidden" accept-charset="UTF-8" name="authenticity_token" value="q8EXIx_tiIwP0NK8jyzANff8Po552XQBNyyMWhKIo905qhCftWf1gI2ZA43_CQcn8N6EXrXD8u_nSG9ELEc7Rg" autocomplete="off">
  
  <label for="">title</label>
  <input name="blog[:title]"></input>

  <label for="">content</label>
  <input name="blog[:content]"></input>

  <label for="">author</label>
  <input name="blog[:author]"></input>
  
  <button value="submit">送出</button>
</form>
```

### params 也可以設定預設值？

在這之前，有件事情我們必須先知道

ActionController 主要的工作是處理 HTTP

![](https://i.imgur.com/zcR4ZTl.jpg)

當 request 給 controller 時，

ActionController 會去呼叫 url_for 方法，

依照 HTTP 的 request (在 Rack 規格裡面，包括 params) 來定義 URL

但是當我們設定了 default_url_options 預設 params 時

預設 params 就會跟著 controller / action 被塞進 params 中

結果會將 url_for 定義的 params 複寫

```
class BlogController << ApplicationController

  def default_url_option
    { locale: I18n.locale }
  end

end
```

### 設定 JSON 格式

若是今天的 requset 中的 Content-Type 為 application/json 時

Rails 會自動將 params 轉成 json 格式

另外有打開 config.wrap_parameters 的話

就會將 params 包在一個 key 中(名稱為 controller 的名稱)

格式為 json 格式

```
## 原 request 中的參數

{ "name": "ninglab" }

## 發送 request 給 BlogController

{ "name": "ninglab" , 
  "blog": { "title" => "parameter", "content" => "hello", "author" => "Ning" }}
```