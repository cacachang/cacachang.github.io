---
title: Rails 實作應用 - Model 篇
date: 2022-08-21 03:28:43
category: Rails
description: Rails 實作的功能拆分 -  資料庫翻譯蒟蒻 Model
---
## Model

我們會把這些東西放在Model

1. 商業邏輯方法
2. 資料驗證(寫入資料庫前)
3. 資料表關聯

### 商業邏輯方法

簡短方法可用 scope 來寫
*小提醒：where 都放在 Model 寫，不要放 Controller

```ruby=
scope :available, -> { where(deleted_at: nil) }
```
預設方法：default_scope (設定就會套用在所有的搜尋中)
```ruby=
default_scope { where(deleted_at: nil) }
```
解除預設方法：unscope(:where)
```ruby=
scope :available, -> { unscope(:where).where(deleted_at: nil) }
```

### 資料驗證

針對寫入的資料做驗證，不符合就無法存入資料庫中
```ruby=
validates :title, presence: true, 
                  length: { minimum: 2 }
```

### 資料表關聯

讓 table 與 table 之間產生方法來互相存取，並非真的有關聯

#### 一對一

```ruby
# user.rb
# 一個 user 可以擁有一個 blog
class User < ApplicationRecord
  has_one :blog
end

# blog.rb
# blog 屬於 user
class Blog < ApplicationRecord
  belongs_to :user
end
```

#### 一對多

```ruby
# user.rb
# 一個 user 可以擁有很多 article 
class User < ApplicationRecord
  has_many :articles
end

# article.rb
# article 屬於 user
class Article < ApplicationRecord
  belongs_to :user  
end
```

#### 多對多

* 如果 has_many 的 table 與 has_one 的 table 重複，要使用 source 來指定foreign key

```ruby
# user.rb
# user 可以喜歡很多篇 articles
class User < ApplicationRecord
  has_many :like_articles
  has_many :liked_articles, through: :like_articles, source: :article
end

# article.rb
# article 可以被很多個 user 喜歡
class Article < ApplicationRecord
  has_many :like_articles
  has_many :users, through: :like_articles          
end

# like_article.rb
# 透過第三方 table 來存取兩個 table 的資料
class LikeArticle < ApplicationRecord
  belongs_to :user
  belongs_to :article
end
```

#### 一旦定義了關聯，就會產出相關方法

以上述 user 來舉例

設定 has_many、has_one，會得到四個方法
而 belong_to，只會有上面兩個方法( articles、article= )

```shell
# 增加 article 跟 article= 方法
> u1.articles
  Article Load (0.5ms)  SELECT "articles".* FROM "articles" WHERE "articles"."deleted_at" IS NULL AND "articles"."user_id" = ? /* loading for inspect */ LIMIT ?  [["user_id", 1], ["LIMIT", 11]]
  
> u1.articles = [t1]
  Article Load (0.3ms)  SELECT "articles".* FROM "articles" WHERE "articles"."deleted_at" IS NULL AND "articles"."user_id" = ?  [["user_id", 1]]
  TRANSACTION (0.1ms)  begin transaction
  Article Update (0.7ms)  UPDATE "articles" SET "updated_at" = ?, "user_id" = ? WHERE "articles"."id" = ?  [["updated_at", "2022-08-20 17:05:13.955092"], ["user_id", 1], ["id", 1]]
  TRANSACTION (1.0ms)  commit transaction
  
# 增加 build_blog 跟 create_blog 方法

> u2.create_blog(title: "hey")
  TRANSACTION (0.1ms)  begin transaction
  Blog Create (0.8ms)  INSERT INTO "blogs" ("title", "user_id", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["title", "hey"], ["user_id", 9], ["created_at", "2022-08-20 19:50:55.440307"], ["updated_at", "2022-08-20 19:50:55.440307"]]
  TRANSACTION (0.5ms)  commit transaction
  Blog Load (0.1ms)  SELECT "blogs".* FROM "blogs" WHERE "blogs"."deleted_at" IS NULL AND "blogs"."user_id" = ? LIMIT ?  [["user_id", 9], ["LIMIT", 1]]
  
> u3.create_blog(title:'yo')
  TRANSACTION (0.1ms)  begin transaction
  Blog Create (0.7ms)  INSERT INTO "blogs" ("title", "user_id", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["title", "yo"], ["user_id", 10], ["created_at", "2022-08-20 19:52:20.563980"], ["updated_at", "2022-08-20 19:52:20.563980"]]
  TRANSACTION (0.9ms)  commit transaction
  Blog Load (0.1ms)  SELECT "blogs".* FROM "blogs" WHERE "blogs"."deleted_at" IS NULL AND "blogs"."user_id" = ? LIMIT ?  [["user_id", 10], ["LIMIT", 1]]

```
#### table欄位 增加、修改、刪除 

專案做到後面一定會有欄位增減的問題
這時候 migration 就很好用
* 記得確認完欄位，要做 ```rails db:migrate```

相關設定可以參考[官方網站](https://guides.rubyonrails.org/active_record_migrations.html)

```shell=
# 終端機指令
> rails g migration Add_Delted_At_To_Article
```
```ruby
# db/migrate/xxxxxxx_add_delted_at_to_article

class AddDeltedAtToArticle < ActiveRecord::Migration[6.1]
  def change
    # 增加欄位
    add_column :articles, :deleted_at, :datetime
    add_index :articles, :deleted_at
  end
end
```

##### 小補充：index

使用 index 可以增加效率，但因為多了檢索欄位，會多佔一些空間

```shell=
# 建立 model 時可以這樣設定
> rails g model Article title content deleted_at:datetime:index
```


#### 密碼加密

密碼相對其他資料機密且重要，所以我們會對密碼做雙重加密的動作

```ruby=
private

def encrypt_password
  # step1 導入 digest 模組
  require 'digest'

  # step2 加鹽 salting (讓密碼更複雜，難以被破解)
  # 加上 self 是指 user(class) 的 password
  pw = "xx-#{self.password}-yy"

  # step3 加密
  self.password = Digest::SHA1.hexdigest(pw)
end
```