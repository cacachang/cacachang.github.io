---
title: 叫我套件王 - 會員系統 Devise【1】
date: 2024-02-22 23:02:40
tags:
---
### 為什麼要用 Devise

 支援多種會員系統模組
 有許多方便的方法

前置作業：
* 建立一個 rails 專案 (本文章使用 7 版本)
* 安裝 [devise gem](https://github.com/heartcombo/devise)

那我們就進入正題！

#### Step 1 把 devise 安裝起來

```
$ bundle add devise
```

```
$ rails generate devise:install
```
這個指令會幫初始化 devise 跟產出 devise 的 i18n yml 設定檔

```
create  config/initializers/devise.rb
create  config/locales/devise.en.yml
```

會跳一大串的英文，不外乎就是在跟你說基本的設定等等等

```
Depending on your application's configuration some manual setup may be required:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

     * Required for all applications. *

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"
     
     * Not required for API-only Applications *

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

     * Not required for API-only Applications *

  4. You can copy Devise views (for customization) to your app by running:

       rails g devise:views
       
     * Not required *
```

devise 的安裝就大概完成囉

#### Step 2 用 devise 產出 model

如果今天 model 想叫 admin 或其他的，直接把 `User` 的地方改 `Admin` 或其他名字就可以了

```
$ rails generate devise User
```

這個指令幫我們做出了 routes 及 model 及 migration 以及測試檔案

```
invoke  active_record
create  db/migrate/20230430165603_devise_create_users.rb
create  app/models/user.rb
invoke  test_unit
create  test/models/user_test.rb
create  test/fixtures/users.yml
insert  app/models/user.rb
route  devise_for :users
```

我們可以來看一下 user.rb 這個檔案

```
# Include default devise modules. Others available are:
# :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
devise :database_authenticatable, :registerable,
       :recoverable, :rememberable, :validatable
```

devise 除了做出基本的會員註冊、登入功能，還包含了多種功能，依序介紹一下

```
confirmable: 發送註冊驗證信，並且在登入的時候，驗證使用者是否已經透過信件驗證
lockable: 在登入失敗幾次後，會將使用者帳號鎖住
timeoutable: 在一段時間內使用者沒有活動紀錄，就中止此 session
trackable: 追蹤登入次數、時間及 IP 位置
omniauthable: 第三方登入設定
database_authenticatable: 針對密碼加密及儲存，會在使用者登入的時候做驗證
registerable: 處理使用者的註冊流程，另外也提供編輯、刪除功能
recoverable: 重設密碼功能
rememberable: 透過 cookie 的資料來設定「記住我」
validatable: 提供信箱及密碼的驗證機制，也可以另外自訂驗證
```

假設我們今天要使用 confirmable 功能，那就必須把 :confirmable 解除註解，並且加到 `devise` 後方

```
# Include default devise modules. Others available are:
# :lockable, :timeoutable, :trackable and :omniauthable
devise :database_authenticatable, :registerable,
            :recoverable, :rememberable, :validatable, :confirmable
```

在做 migration 以前，也要將 migration 的 :confirmable 解除註解

```
# db/migrate/20230701155654_devise_create_users.rb
    ...
    ...
    ...
    # Confirmable
    t.string   :confirmation_token
    t.datetime :confirmed_at
    t.datetime :confirmation_sent_at
    t.string   :unconfirmed_email # Only if using reconfirmable
    ...
    ...
    ...
```

設定完成後我們就可以 migrate 了

```
> rails db:migrate
```

然後重啟 server


### 試試 Devise 的基本註冊功能

先來檢視一下 devise 為我們做出來的路徑

```
> rails routes -c devise
```
注意這邊要用 devise 才會顯示出 devise 做出來的路徑

```
                  Prefix Verb   URI Pattern                    Controller#Action
        new_user_session GET    /users/sign_in(.:format)       devise/sessions#new
            user_session POST   /users/sign_in(.:format)       devise/sessions#create
    destroy_user_session DELETE /users/sign_out(.:format)      devise/sessions#destroy
       new_user_password GET    /users/password/new(.:format)  devise/passwords#new
      edit_user_password GET    /users/password/edit(.:format) devise/passwords#edit
           user_password PATCH  /users/password(.:format)      devise/passwords#update
                         PUT    /users/password(.:format)      devise/passwords#update
                         POST   /users/password(.:format)      devise/passwords#create
cancel_user_registration GET    /users/cancel(.:format)        devise/registrations#cancel
   new_user_registration GET    /users/sign_up(.:format)       devise/registrations#new
  edit_user_registration GET    /users/edit(.:format)          devise/registrations#edit
       user_registration PATCH  /users(.:format)               devise/registrations#update
                         PUT    /users(.:format)               devise/registrations#update
                         DELETE /users(.:format)               devise/registrations#destroy
                         POST   /users(.:format)               devise/registrations#create
```

接下來我們可以直接到會員註冊的路徑了

```
http://127.0.0.1:3000/users/sign_up
```

貼到瀏覽器中會出現以下畫面

![](https://i.imgur.com/y6RCOqN.png)

輸入註冊的 Email 及密碼就能註冊成功囉！


### devise 幫你做的方法

Devise 有做出一些 controller 及 view 常見的方法
需要注意的是，假設 model 是 `Admin`，請記得要把方法的 `_user` 改成 `_admin` ，其他依此類推

#### Controller action

舉例來說，假設我們的網站是使用者登入才可以使用，我們可以在 application controller 中加入 `before_action :authenticate_user!`
設定後，使用者要進入所有頁面前都需要先登入

#### View helper

如果頁面中有特定區塊是使用者登入才看得到的，我們可以加入 `user_sign_in?` 在 view 裡面我們可以這樣寫：
```
<% if user_signed_in? %>
    已登入使用者才能看到的區塊
<% end %>
``` 

另一個更常見的 helper 是 `current_user`
如果我們今天要在 view 顯示每個 user 的 email，就可以在 view 裡這樣寫：
```
<%= current_user.email %>
```

還有其他方法，要使用的話可以直接至 Devise 手冊翻閱
今天的介紹內容相對簡單，下一篇我們再繼續挖深一點！