---
title: Django - 基礎 models 概念與關聯
date: 2024-05-19 18:24:05
category: Django
---
# Django - Models

Django 所採用的模式是 MTV 模式

上週介紹的 Views 文章有提到 MTV 模式，我們再來複習一下

使用者瀏覽網頁時，伺服器的運作流程如下：

![MTV](https://imgur.com/wg4bxVE.jpg)


urls.py 會將 request 傳送至 View

View 收到 request，處理過程中需要資料庫的資料時，就會向 Model 要資料

Model 收到 View 來的指令，將處理好的資料給 View (* Model 不是代表資料庫，而是 負責與資料庫溝通 的一個 Layer )

View 收到資料後，將資料做最後的處理

View 拿 Template 的東西來進行 render，最後由 View 回應給使用者

## Model

我們這週要來介紹的是 Model

當我們需要處理資料時，就會需要 Model

Model 並不是只資料庫，而是跟資料庫溝通的一層 Layer

我們先來了解什麼是 `ORM`

### ORM

Django 採用 ORM 來處理資料，ORM 全名為 `Object Relational Mapping` ，

原本我們要使用 SQL 語法 跟資料庫溝通

不過 ORM 讓我們可以用 `物件` 的方式來處理資料

假設我想看某個 table 的所有資料

我必須要用 SQL 語法來查詢

```
SELECT * FROM table_name;
```

但如果是用 Django 的 ORM 語法呢？會簡單易懂許多

```
User.objects.all()
```

了解了 ORM 語法後，我們可以先來看一下該怎麼定義 table 以及 column

## models.py

我們會將要定義的 table 以及 column 放在 models.py 這個檔案之中

### fields

在 Django 的手冊中指出，每個 model 都是 class，因此可以繼承 django.db.models.Model 的特性

> Each model is a Python class that subclasses django.db.models.Model

所以第一件事就是用 `class` 的方式定義 `table`

並在該 class 中定義屬性，這些屬性代表 `table` 中的 `column` ，在 Django 稱作為 field

> Each field is specified as a class attribute, and each attribute maps to a database column.

我們要先 import `models`，讓 class 去繼承 models 中的 Model

接著定義 table 的名稱以及屬性

```
from django.db import models

class User(models.Model):
    name = models.CharField(max_length=20, default="")
```

理解基本的 table 以及 column 定義後，我們可以來點關聯！

### one to one

有些資料會有一對一的關聯，舉例來說

除了基本 User 資料，需要有另一個 Model 來存個人資訊，像是 地址 / mail / 電話等等

這時候 User 與個人資料 table 就會有一對一的關聯

該怎麼做呢？

#### ForeignKey

當我們知道一個 User 會有一個 Profile 之後，

就可以用 `關係` 來想看看 `ForeignKey` 設定該放在哪裡

通常我們會將 Foreign Key 放置在「屬於」的 Model 中

以剛剛的例子來說，是 Profile 會 `屬於` 一個 User

所以我們會將 `ForeignKey` 放在 `Profile` 該 table 中

ForeignKey 就像是一條線索， Profile 會透過該線索，去找到 User

![one to one](https://imgur.com/6CSSwpF.jpg)

接著我們就來把它轉變成程式碼吧！

在 ForeignKey 中有幾個必填的參數

> class ForeignKey(to, on_delete, **options)

第一個參數是 ForeignKey 會指向哪個 model

第二個參數 on_delete 意思是當 `關聯 model` 的資料被刪除時，要做什麼事，這邊我們設定成 models.CASCADE ，代表當 User 被刪除時，跟該筆有關的 Profile 都會被刪除

第三個參數是為了一對一的關聯而設定的，我們希望一筆 User 只對應一筆 Profile ，對於 Profile 來說，每一個 User 的 id 都是獨特的，不可重複

(如果沒有設定這個參數，那就會變成 User 可以有多筆 Profile 囉)

```
from django.db import models

class User(models.Model):
    ...


class Profile(models.Model):
    ...
    user = models.ForeignKey(User, on_delete=models.CASCADE, unique=True)
```


#### OneToOneField

Django 也有提供更明確的一對一關聯方法，

也就是 OneToOneField

這個方法會會直接在 Profile 中增加一個欄位，存放 Foreign Key (也就是 User 的 id )

```
from django.db import models

class User(models.Model):
    name = models.CharField(max_length=10)
    age = models.IntegerField()


class Profile(models.Model):
    email = models.CharField(max_length=10, default="")
    address = models.CharField(max_length=50, default="")
    phone = models.CharField(max_length=10, default="")
    user = models.OneToOneField(User, on_delete=models.CASCADE)
```

上述兩種方法都可以建立一對一的關聯，

相較於 `ForeignKey` 的方式，`OneToOneField` 在程式碼中會顯得更易讀明確

### many to one

再來就是稍微複雜的關聯，多對一，

舉例來說，今天一個 User 會有多筆 Order

但一筆 Order 只能對應一個 User

這時候就會形成 `多對一` 的關聯

我們來看一下該怎麼設定

#### ForeignKey

就像剛剛舉例的，我們會將 `Foreign Key` 放在 「屬於」 的 table 

這次的例子是 多筆 Order 會屬於一個 User

所以我們會將 `ForeignKey` 該設定放在 Order 中

我們直接用圖片來理解比較快

![many to one](https://imgur.com/NR6Jwsv.jpg)

理解了多對一的關係後，我們就可以來轉變成程式碼了

跟剛剛的一對一不同，這時每個 Order 的 User 是可以重複的，所以我們不需要加上 `unique=True`

```
from django.db import models

class User(models.Model):
    ...
    
class Order(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
```

### many to many

再來就會更複雜一點，也就是多對多的關聯

舉例來說，使用者與群組的關聯，

一個使用者可以參加多個群組，

而一個群組可以擁有多個使用者，

這時候就會形成多對多的關聯

![many to many](https://imgur.com/NWvEUxd.jpg)

#### ManyToManyField

Django 提供了我們 ManyToManyField 的方法

這次的關係是 一個使用者可以 `屬於` 多個群組 且 一個群組可以 `屬於` 多個使用者

不過通常多對多的關係，都會用第三方表格來存放雙方的關聯

所以 `ForeignKey` 會放在第三方表格中

而 User 及 Group 則是要設定 `ManyToManyField` 以及 through 參數，

當你需要用 User 去查詢 Group 的時候就會比較方便，不需要透過第三方表格

* 屬性的定義 是要用複數，不然會跑出提醒你要改名的錯誤訊息
* User 在建立的時候，Group 還不存在，因此要用字串來取代

```
from django.db import models

class User(models.Model):
  groups = models.ManyToManyField('Group', through="UserGroup")
  

class Group(models.Model):
  users = models.ManyToManyField(User, through="UserGroup")
  
class UserGroup(models.Model):
  user = models.ForeignKey(User, on_delete=models.CASCADE)
  group = models.ForeignKey(Group, on_delete=models.CASCADE)
```

雖然多對多的關係可以在第三方表格用 `ForeignKey` 來設定關聯，

但如果沒有透過 `ManyToManyField`

在查找資料的時候就都得透過第三方表格，其實蠻不方便的

所以還是要設定 `ManyToManyField` 以及 `through` 參數唷

基本的關聯差不多都介紹完了，接下來就剩怎麼將定義變成資料庫中的 table 以及 column

### migration

當我們將 table 及 column 都設定好之後，就可以來做 migration 啦

migration 是什麼呢？

我們可以把它想像成是 Database 的歷史紀錄

有任何的更動都會被記錄在 migration 中

在 Django 中，我們只要下

```
python manage.py makemigrations
```

就會依照更動來做出新的 migration 囉

### migrate

當 migration 做完，我們就該來改變資料庫的 table 以及 column

這時候就需要下 

```
python manage.py migrate
```

資料庫這時候就會依據設定改變了

#### 狀況題：如果 migrate 之後，又要做更動該怎麼辦？

只要動到 models 這個檔案，再重新做一次 makemigrations 再 migrate 就可以了

#### 狀況題：如果需要更改 migration 的時候該怎麼辦

在 Django 中，有可以讓我們 rollback 的指令

什麼是 rollback 呢？可以把它想像成是鍵盤的 ctrl + z 復原鍵

我們要將 migration 復原可以下該指令

```
python manage.py migrate YOUR-APP-NAME YOUR-ROLLBACK-MIGRATION-NUMBER
```

這時候就可以進行復原

以上是基礎的 Django model 使用介紹

如果還有狀況題想要提出討論，歡迎在底下留言，會傳送到我的 github 作為 issue 通知我，屆時可以一起討論

如果有幫助的話也歡迎按讚 ヾ(*´∀ ˋ*)ﾉ
