---
title: SQL - Joins、Includes 差別在哪裡？
date: 2023-04-08 23:19:24
category: 資料庫
description: Joins 與 Includes 常常分不清楚，圖表化差別一次懂
---
# Join and Includes

我們在開發的時候免不了關聯資料庫，在撈資料的時候也常常會需要用到 joins 跟 includes

## 所以，我們什麼時候會用到 joins 跟 includes

* 要尋找跟其他 model 有關聯的資料
* 避免 N + 1 問題

** N + 1 問題是什麼呢？
當我們查詢多筆資料時，資料庫會先將所有的資料撈出，並一個一個去比對，就會有 N(筆資料) + 1(次撈出全部) 的問題存在


## joins 跟 includes 有什麼差別？

![](https://i.imgur.com/ZEftaXX.jpg)

joins 跟 includes 其實有蠻大的差別，

不過因為他們都是在查詢關聯資料，所以我們很常搞混，

不過搞懂他們的差別就會知道什麼時候該派誰上場。

** lazy loading：並不會一開始就載入所有資料，而是需要用到的時候才會去收集並載入
** eager loading：一開始就把需要的資料都載入，需要時就可以從 Cache 中拿取

先來說結論：joins 只會去比對資料，而 includes 會去查詢有關聯的資料，並存到 cache 中

那我們來看一下實際使用吧

### joins

使用 joins 時，會去「比對」Tag 中有 post_id 的資料

```ruby=
find_tags = Post.joins(:tags)

# Post Load (1.0ms)  SELECT "posts".* FROM "posts" INNER JOIN "tags" ON "tags"."post_id" = "posts"."id" 
```
當我們需要這包資料時，
這時會再去資料庫撈一次資料，把有 tag 的資料撈出來呈現

```ruby=
find_tags.each do |post|
  p post.tags.first.name
end

# Tag Load (0.3ms)  SELECT "tags".* FROM "tags" WHERE "tags"."post_id" = $1 ORDER BY "tags"."id" ASC LIMIT $2  [["post_id", 4], ["LIMIT", 1]]
1 

# Tag Load (0.1ms)  SELECT "tags".* FROM "tags" WHERE "tags"."post_id" = $1 ORDER BY "tags"."id" ASC LIMIT $2  [["post_id", 7], ["LIMIT", 1]]                                         
2 

# Tag Load (0.1ms)  SELECT "tags".* FROM "tags" WHERE "tags"."post_id" = $1 ORDER BY "tags"."id" ASC LIMIT $2  [["post_id", 8], ["LIMIT", 1]]                                         
3 
```

看起來是不是很沒效率呢？
那我們來看看 includes 會怎麼運作

### includes

使用 includes 時，會去把所有 Tag 的資料一次撈出並查詢哪些有 post_id

```ruby=
find_tags = Post.includes(:tags)

# Post Load (0.4ms)  SELECT "posts".* FROM "posts" 
# Tag Load (0.3ms)  SELECT "tags".* FROM "tags" WHERE "tags"."post_id" IN ($1, $2, $3, $4, $5, $6, $7, $8)  [["post_id", 3], ["post_id", 4], ["post_id", 7], ["post_id", 8], ["post_id", 9], ["post_id", 2], ["post_id", 1], ["post_id", 10]] 
```

當我們需要使用這包資料時，
會直接從 Cache 中拿出這包已經處理好的資料

```ruby=
find_tags.each do |post|
  p post.tags.first.name
end

1
2
3
```

由上述例子可以知道， includes 在查詢及撈資料的過程都相對有效率


## 所以用 includes 就比較好嗎？

要看使用的狀況，以剛剛的 Post 及 Tag 例子來看

我們改變一下需求，要撈出有 tag 的 post 資料，只需要呈現**特定 tag 的 post 資訊**

#### joins

這時候 joins 只有去比對條件，並沒有去撈出資料

```ruby=
find_tags = Post.joins(:tags).where(tag: {name: 'hello'})

#  Post Load (0.5ms)  SELECT "posts".* FROM "posts" INNER JOIN "tags" ON "tags"."post_id" = "posts"."id" WHERE "tags"."tag_name" = $1  [["tag_name", "hello"]]

find_tags.each do |post|
  p post.id
end

# 1
```

#### includes

使用 includes ，會到資料庫將所有有關聯的資料撈出來做一次查詢

```ruby=
find_tags = Post.includes(:tags).where(tag: {name: 'hello'})

# SQL (1.0ms)  SELECT "posts"."id" AS t0_r0, "posts"."title" AS t0_r1, "posts"."description" AS t0_r2, "posts"."content" AS t0_r3, "posts"."created_at" AS t0_r4, "posts"."updated_at" AS t0_r5, "posts"."slug" AS t0_r6, "tags"."id" AS t1_r0, "tags"."tag_id" AS t1_r1, "tags"."post_id" AS t1_r2, "tags"."created_at" AS t1_r3, "tags"."updated_at" AS t1_r4 FROM "posts" LEFT OUTER JOIN "tags" ON "tags"."post_id" = "posts"."id" WHERE "tags"."tag_name" = $1  [["tag_name", "hello"]] 

find_tags.each do |post|
  p post.tags.first.name
end

# 1
```

在不需要印出關聯資料庫的資料狀況時， joins 只有去比對資料，並沒有撈資料的動作，所以會比 includes 更有效率。

## 結論

在查詢關聯資料時，joins 只會去比對資料，而 includes 則會去查詢有關聯的資料並存到 Cache 中。在實際使用中，joins 會比 includes 更加高效，特別是當我們不需要印出關聯資料時。
