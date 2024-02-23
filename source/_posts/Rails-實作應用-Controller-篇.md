---
title: Rails 實作應用 - Controller 篇
date: 2022-08-21 03:24:42
category: Rails
description: Rails 實作的功能拆分 - 運算處理中心 Controller
---
### Controller

針對 View 的需求，跟 model 拿資料
再回來處理這些資料

#### Routes 會指引到對應的 Controller 及 action

```ruby=
Prefix  Verb   URI Pattern                   Controller#Action
blogs   GET    /@:handler/blogs(.:format)    blogs#show
```

#### 全站都會用到的方法，就放 ApplicationController

```ruby=
class ApplicationController < ActionController::Base
  def authenticta_user
    redirect_to sign_in_users_path if not user_sign_in?
  end
end
```

#### private method

僅限於 此 Controller 採用的方法

```ruby
private
  def encrypt_password
    require 'digest'
    pw = "xx-#{self.password}-yy"
    self.password = Digest::SHA1.hexdigest(pw)
  end
end
```

### 功能應用

#### callback

多個方法都會做的動作，就會拉出來做callback
動作通常會被放在private中

以下舉例

```ruby
# show、unlock方法，在執行前需要去找 article
before_action :find_article, only: [:show, :unlock]

private
def find_article
  @article = Article.find(params[:id])
end
```
    
#### 清洗資料

```ruby
# 針對要資料做清洗，避免存入髒資料
def params_article
  params.require(:article).permit(:title, :content, :pincode)
end

# 在一對多、多對多的關聯中，存入需要塞其他資料的話(像是 comment 存入前要有 user )，可以使用merge
def comment_params
  params.require(:comment).permit(:content).merge(user: current_user)
end
```

#### 軟刪除

講白話一點，就是這個刪除是演出來的
這筆資料並不會被刪除

那這筆資料到底發生了什麼事？
首先，我們會有 deleted_at 欄位
在刪除的時候把確切時間寫入
之後再將這筆資料隱藏起來

```ruby=
# controller
def destroy
  @article = Article.find(params[:id])
  # 更新 deleted_at 的時間
  @article.update(deleted_at: Time.current) 
    
  redirect_to blogs_path, notice: "文章刪除成功"
end
```

```ruby=
# model
# 記得在 model 中做資料篩選，不然使用者刪除，卻還顯示在畫面就尷尬了
# default_scope 會在 model 中提到

default_scope { where(deleted_at: nil) }
```

#### session

當使用者登入時，我們需要發給他號碼牌
這時候我們就會用 session 來存使用者的 id
這個 session 會跟 cookie 核對並儲存

下次使用者再來
session跟cookie核對成功就不用再登入了

```ruby=
def create
  # 使用者登入
  user = User.login(params[:user])

  if user
    # session 會存在於瀏覽器的 cookie
    session[:user_session] = user.id

    if user.blog.present?
       redirect_to "/@#{user.blog.handler}", notice: "登入成功"
    else
       redirect_to new_blog_path , notice: "登入失敗，請先建立部落格"
    end
  else
   # 不能用 render，因為在 form_with 設定路徑是到 sessions
   # 如果去找 session，session 沒有 view，重新整理會壞掉
   redirect_to "/users/sign_in", notice: "錯誤，請重新登入"
  end
end
```

### 記憶 memorization

為了減少資料庫的存取次數，我們可以在判斷時加上 ||= 或 &. 

```ruby
def current_user
  # 有 @user 就回傳，沒有就繼續執行後面的條件
  @user ||= User.find_by(id: session[:user_session])
end
```

#### rescue_from

如果使用者按到沒有資料的頁面(例如 第2000筆user)
應出現 404 頁面
除了告訴使用者沒有這筆紀錄外
也要跟瀏覽器說這個頁面是錯誤的

rescue_from可以幫我們處理error的資料
避免使用者瀏覽的時候出現錯誤的畫面

```ruby=
class ApplicationController < ActionController::Base
  rescue_from ActiveRecord::RecordNotFound, with: :record_not_found
    
  private
  def record_not_found
    # file: 從根目錄去抓404檔案
    # layout: 不要套用全站layout(不然會有多餘的html)
    # status: 告訴瀏覽器頁面狀態，不然他會顯示2xx（正常的狀態）
    render file: "#{Rails.root}/public/404.html", layout: false, status: 404
  end
end
```

* 額外補充：放在public的東西不會走正常流程的(routes)，怕伺服器出狀況，會連檔案都找不到
