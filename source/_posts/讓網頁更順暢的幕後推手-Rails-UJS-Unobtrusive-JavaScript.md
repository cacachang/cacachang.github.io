---
title: 小機制大效用 一用就回不去的 Rails UJS - Unobtrusive JavaScript
date: 2022-08-12 01:58:53
tags:
categories: Rails

---

# 小機制大效用 一用就回不去的 Rails UJS - Unobtrusive JavaScript

真正認識UJS後，就會發現他是個超好用的東西
讓程式碼更簡潔，也能~~少寫一些程式碼~~



### JQuery UJS

咦，不是說要介紹Rails UJS嗎？
為什麼會出現一個毫無相關的JQuery呢？

其實在Rails 5以前
Rails就有包含JQuery UJS

寫Rails時可以更輕鬆的發送AJAX Request
有時候甚至不用撰寫
他自動就會幫你送出

不過Rails 5之後，Rails就把JQuery拔掉了
改換內建的 Rails UJS

應該是太好用才這樣設計吧 (笑



### 本次主角 - Rails UJS

Rails UJS會幫我們做幾件事情
1. 在表單、連結物件上，自動加入data-*屬性
2. 轉換GET為其他方法，讓Rails執行
3. data confirm防呆確認
4. 防止雙點擊


### 1. 在表單、連結物件上，加入data-*屬性

來看看哪些特殊的物件會被加入屬性吧～

#### form helper
神奇的表單工具，會猜測你的路徑以及表單要做的事

當我們使用form_for、form_tag、form_with來產生表單時
Rails UJS會生出一個隱藏表單
在裡面加入屬性，像是方法，並帶token去發送請求

```ruby
# 我們使用form_for來做表單
<%= form_for @article do |form| %>
  標題：
  <%= form.text_field :title %>
  內容：
  <%= form.text_field :content %>
  <%= form.submit '送出' %>
<% end %>
```

HTML原始碼

```html
<!-- Rails UJS 生出來的隱藏欄位 -->
<input type="hidden" name="_method" value="patch" autocomplete="off" />
<!-- Form生出來的，Rails UJS 會一起打包帶走 -->
<input type="hidden" name="authenticity_token" value="xxxxx" autocomplete="off" />
```

#### link_to

產生超連結的方法
如果加入了remote: true，就會呼叫js，可避免雙重點擊狀況(詳見第4點)

```ruby
# 使用 link_to 來做超連結
<%= link_to '新增文章', new_blog_path, remote: true %>
```

HTML原始碼

```html
<!-- Rails UJS 加入 data-remote 屬性 -->
<a href="/blog/new" data-remote="true">新增文章</a>
```


### button_to

產生按鈕，跟link_to一樣，加入了remote: true，就會呼叫js

```ruby
# 使用 button_to 來做按鈕
<%= button_to "新增文章", new_blog_path, remote: true %>
```

HTML原始碼

```html
<!-- Rails UJS 加入 data-remote 、 method 屬性 -->
<form action="blog/new" class="button_to" data-remote="true" method="post">
  <input type="submit" value="新增文章" />
</form>
```


### 2. data-方法

HTTP僅支援GET及POST方法
其中只有表單可以使用POST方法
那編輯、更新、刪除怎麼辦呢？

當我們在屬性中設定method="方法"時
Rails UJS會產生_method
讓Rails知道你要執行什麼方法
是不是超方便的！🥳
    
```ruby
# 使用 link_to 來做刪除功能的超連結
<%= link_to '刪除', article, method: "delete", data: {confirm: "are you sure?"}  %>
```

HTML原始碼

```html
<!-- Rails UJS 加入data-method屬性 -->
<a data-confirm="are you sure?" rel="nofollow" data-method="delete" href="/books/2">destroy</a>
```


### 3. 用確認來做警告、提示，並執行刪除方法

data-confirm屬性會生出一個警告框
在觸發時讓使用者二度確認

```ruby
# 我們在 link_to 中加入 data 及 confirm 讓使用者二度確認
<%= link_to '刪除', article, method: "delete", data: {confirm: "are you sure?"}  %>
```

確認畫面

![](https://i.imgur.com/RtmVOEt.png)


### 4. 防止雙點擊

防止使用者雙重點擊，避免HTTP傳送多次request，導致伺服器當機的狀況

```ruby
# 使用 disable_with 來防止儲存時雙重點擊狀況
<%= form_with(model: User.new) do |form| %>
  <%= form.submit data: { disable_with: "Saving..." } %>
<% end %>
```

HTML原始碼

```html
<!-- Rails UJS 加入了防雙重點擊的屬性 -->
<input data-disable-with="Saving..." type="submit">
```

### 說了這麼多，Rails UJS的運作流程是怎麼樣呢？

每一小塊流程，可以想像成Rails的callback
在AJAX還沒發送HTTP請求前，需要做哪些事
發送HTTP請求後，又會做哪些事

![](https://i.imgur.com/T2pWXRv.jpg)


### 額外補充：AJAX

AJAX(Asynchronous JavaScript and XML)
是一種非同步的JS及XML機制

早期網頁載入資料時，需要把所有資料先載入到伺服器中，載入完畢才會渲染到畫面中，這樣的做法不僅增加伺服器的負荷，也讓使用者增加了等待時間

AJAX採取需要哪些資料，就先載入哪些資料，並且向HTTP傳送request，這樣的機制不僅減少伺服器的負擔，也讓瀏覽器能快速地回應使用者，減少等待時間


