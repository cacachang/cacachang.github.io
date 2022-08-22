---
title: Rails 實作應用 - Route、View篇
date: 2022-08-21 02:21:52
tags:
description: Rails 實作的功能拆分 - 負責傳遞引導及傳遞資料的角色
categories: Rails 實作 
---
## Routes

* 做功能的第一件事：想路徑

* 路徑命名：

    * 好好分類，直接放在根目錄下不適當
    * 名稱需簡單易懂
    * 有些方法不適合用既有的 routes，這時候就要另外做
        * ex.登入 session、註冊sign_up

* 路徑生成方法：
    
    **resource**
    ![](https://i.imgur.com/UanDl3X.jpg)
     
    **member、collection**
    ![](https://i.imgur.com/GZWRGDZ.jpg)
 
    **namespace、scope**
    ![](https://i.imgur.com/HzabrTq.jpg)
    
    **shallow、二段式**
    ![](https://i.imgur.com/AzObJoS.jpg)

---

## View

將資料呈現在畫面中

基本原則：不要讓 View 去撈資料，盡量從 Controller 餵，如果需要撈，把方法寫在 View Helper

```ruby=
module UsersHelper
  def current_user
    User.find_by(id: session[:user_session])
  end
end
```

如果把方法搬到 Controller 需要寫 helper_method

```ruby=
class ApplicationController < ActionController::Base
    helper_method :current_user
    
    private
    def current_user
      User.find_by(id: session[:user_session])
    end
end
```

如果 Controller 要引用 View Helper 的方法，要使用 include

```ruby=
class BlogsController < ApplicationController
  include UsersHelper
  
  def create
    @blog = current_user.build_blog(blog_params)
      if @blog.save
        redirect_to "/@#{@blog.handler}", notice: "已成功建立"
      else
        render :new
      end
  end
end
```

### 檔案

#### layout資料夾

這邊是放全站的排版，所以如果有全站會用到的資料或樣式
就可以放在這邊 layout 資料夾下的 application.html.erb

#### 一般 View

會對應到 Controller 裡的 action
所以檔名要跟 action 一樣


#### Partial View

1. 檔名前面要加_，讓 Rails 分辨(_form.html.erb)
2. 渲染格式 render + 路徑

#### 完整寫法：

```ruby=
<% render partial: "comments/comment", collection: @comments %>
```

 #### 簡化寫法：

```ruby
# 不同資料夾
<%= render "/articles/form", article: @article  %>
# 同個資料夾下
<%= render "form", blog: @blog %>
```

#### Rails風格寫法：@xxxx

下列兩條件成立，就可以用 @xxx 來 render

▶  Partial View 是 comments 資料夾裡的 comment
▶  Partial view 裡的 comment 是單數

```ruby=
<% render @comments %>
```
Partial View裡的 comment 不叫 comment，就得用原本的寫法加上as

```ruby=  
<% render partial: "comments/comment", collection: @comments, as: :comment %> 
```

### View helper

form 家族

* 如果參數接 model，而這包資料有內容，會自動抓取裡面的內容填到表格中
* 自動生成 token，token 的目的是要防止駭客知道路徑及 action 後，寫程式攻擊，這邊要設定 token 來篩選驗證
* 會產生類似 hash 的物件，裡面包涵 table 的 key

#### form_tag

參數接url，適合路徑導外部連結的表單

```ruby=
<%= form_tag "/books/new" do |form| %>
    <div>
        <%= form.label :title %>
        <%= form.text_field :title %>
    </div> 
    <div>
        <%= form.label :content %>
        <%= form.text_area :content %>
    </div>
    <%= form.submit %>
<% end %>
```

#### form_for

參數接 model，適合需要撈資料的表單(ex.編輯)

```ruby=
<%= form_for @book do |form| %>
    <div>
        <%= form.label :title %>
        <%= form.text_field :title %>
    </div> 
    <div>
        <%= form.label :content %>
        <%= form.text_area :content %>
    </div>
    <%= form.submit %>
<% end %>
```

#### form_with

用來取代 form_for 跟 form_tag
參數可以接 model 跟 url

```ruby=
<%= form_with model: @comment, url: article_comments_path(@article) do |comment| %>
<%= form_with model: [@article, @comment], local: false do |comment| %>
```

#### link_to

效能沒有比```<a herf></a>```好，但是路徑填錯馬上就會有錯誤訊息

```ruby
<%= link_to '新增文章', new_blog_path %>
```