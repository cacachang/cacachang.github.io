---
title: Django - urls 的那些基礎用法
date: 2024-04-28 21:24:45
category: Django
---

當使用者點選網址進入網站時，會發送 `request` 給伺服器，
不過伺服器怎麼知道他要做什麼，並且需要什麼資料呢？

今天要來介紹 Django 中的 urls 


![Ning-draw (30)](https://imgur.com/L01upJS.jpg)


## ROOT_URLCONF

當使用者進入到網域中的其中一個頁面時，會發送 request 給 server

Django 收到時，會先去 `settings.py` 找 `ROOT_URLCONF`

看看 URL 的設定放在哪個檔案中

可以將 `ROOT_URLCONF` 想像成百貨公司的樓層介紹，

當我們想要吃麥當勞的時候，該去哪層樓的哪個櫃位

假設今天要做一個電商網站，那麼就可以將 `urls.py` 在最上層的 module 中整合

以下方例子來說，我就會將 `ROOT_URLCONF` 設定為 `cake_shop.urls`

```
# project/cake_shop/settings.py

ROOT_URLCONF = 'cake_shop.urls'
```

每個模組會有自己的 `urls` ，但個別模組的 urls.py 最後都會 include 到 `cake_shop.urls` 來彙整

```
project
├── cake_shop
│  ├── __init__.py
│  ├── asgi.py
│  ├── settings.py
│  ├── urls.py
│  └── wsgi.py
├── member
│  └── migration
│  ├── __init__.py
│  ├── app.py
│  ├── admin.py
│  ├── models.py
│  ├── tests.py
│  ├── urls.py
│  └── views.py
└── manage.py
```

Django 知道該去哪裡找 `urls` 後，就會進到該 `urls.py` 中

## urlpatterns

`urls.py` 中會有個變數叫做 `urlpatterns` ，

在這個變數中，會指定一系列的 `path` 給他，

而這一系列的 `path` 會是 urls 模組底下的 `path 方法` 或者 `re_path 方法` 的實體

> This should be a sequence of django.urls.path() and/or django.urls.re_path() instances

Django 已經先幫我們做出一個管理員的路徑了

```
urlpatterns = [
    path('admin/', admin.site.urls),
]
```

讓我們來看一下 `path` 該怎麼寫吧

### path

Django 官方文件對於 path 方法規範如下：

> path(route, view, kwargs=None, name=None)

讓我們來看一下參數：

`route` 表示要顯示在網址上的路徑，前面不需要加上 `/` 因為原本的 URL 就會有了
`view` 表示這個路徑要去找 `views.py` 中的哪個方法
`kwargs` 指的是 `keyword parameter` 也就是需要給他明確的 `keyword` 跟 `value`
`name` 方便我們用更簡潔且易讀的方式代表 URL

假設我們今天有個頁面需要顯示所有使用者的，這時候我就先會設定他的路徑

希望該頁面顯示的網址是 `https://網域/users`
並且會去找 views 中的 index 方法

```
# project/member/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('users/', views.index),
]
```

### Path Converter

如果今天是要顯示該使用者的基本資料畫面呢？也就是俗稱的 `show` 頁面

`path` 使用 `<>` 來捕捉裡面的參數，並會當作 `keyword parameter` 傳入 views 中，所以我們就可以這樣寫：

```
# project/member/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('users/', views.index),
    path('users/<id>/', views.show),
]
```

Path Converter 預設的型態是字串，不過還有支援 `integer` / `slug` / `uuid` 甚至是完整的路徑也可以

當我們要讓參數傳進 views 中的方法，方法必需要加上 `position argument` (`id`)，也名字必須要一樣，否則就會跑出錯誤訊息

```
# project/memeber/views.py

...

def show(request, id):
    ...
    return render(request, "show.html")
```

#### integer

如果型態不識字串必須要在前面加上 `int` 作為 `keyword parameter`

```
# project/member/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('users/<int:height>/', views.show),
]
```

這時候的網址就會是 `https://網域/users/190/` 

#### slug

同上，只要在前面加上 `slug` 即可

```
# project/member/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('users/<slug:jfowejro>/', views.show),
]
```

#### uuid

同上，只要在前面加上 `uuid` 即可

```
# project/member/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('users/<uuid:075194d3-6885-417e-a8a8-6c931e272f00>/', views.show),
]
```

### 選填參數

剛剛有提到，如果要傳參數進去，必須要在方法中也給他一個 `position argument` 

但如果有兩個路徑要同時去找同一個方法，一個路徑有傳遞參數，一個沒有傳遞參數呢？

我們可以在方法中改成 `keyword argument` ，並且給他預設值，就算路徑沒有傳參數進來，也不會壞掉了

```
# project/memeber/views.py

...

def show(request, id = None):
    ...
    return render(request, "show.html")
```

#### include

當我們要在最上層的 `cake_shop` urls 中引入 `member` 的路徑時，

可以直接將 `member.urls` include 進來

```
# project/cake_shop/urls.py

from django.urls import path, include

urlpatterns = [
    path('', include("member.urls")),
]
```

### name

如果未來想要在 form 表單中使用 url 這種方法，為了簡化路徑，我可以加上 `name` 關鍵字參數

```
# project/member/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('users/', views.index, name="index"),
]
```

還可以搭配 `reverse` 讓程式碼更簡潔且易讀！

#### path reverse

當加入了 `name` 這個 `keyword parameter` 之後，就可以在方法中用 `reverse` 簡化路徑的寫法

```
# project/member/views.py
from django.urls import reverse
from django.http import HttpResponseRedirect

# Create your views here.

...

def show(request, id=None):
    return HttpResponseRedirect(reverse("index"))
```

* 如果搭配 `HttpResponseRedirect` 的話不要重新導向到自己本身的方法，不然就會造成無窮迴圈，以上述的例子來說，如果我在 index 方法導向到 index，就會無窮迴圈

在 form 的話我們就可以在 url 的方法，用易讀又簡短的方式寫

```
<form action="{% url 'index' %}" method="post">
  <div>
    ...
  </div>
</form>
```

#### namespaces

當專案複雜度增加，模組也可能會跟著增加，這時候可能會遇到相同路徑名稱的問題，

name 命名可能因此越來越長，這時候我們可以使用 `namespace` 來做區隔 (前提是要給 name 參數唷！)

namespace 有分成兩種

1. application namespace

每個 application 都可以有自己的 namespace ，且他們的實體前綴也會有 namespace 名稱

我們可以用 `app_name` 來設定 application namespace

```
# project/member/urls.py

app_name = "users"
urlpatterns = [
    path('users/', views.index, name="index"),
]
```

這時候 views 的 reverse 路徑就可以設定成 `users:index` 

```
# project/member/views.py

def show(request, id=None):
    print(id)
    return HttpResponseRedirect(reverse("users:index"))
```

2. instance namespace

實體的 namespace 在這專案中必須是要獨一無二的，如果沒有特別設定，預設就會是 application 的 namespace

namespace 會用 `:` 來區隔

如果我們在 include 的時候有指定 namespace ，這時候該路徑就會被套上 instance namespace

我們可以在 cake_shop 底下的 urls.py 設定 `instance namespace`

```
# project/cake_shop/urls.py

urlpatterns = [
    path('', include("member.urls", namespace="admin")),
]
```

這時候 views 的 reverse 路徑就可以設定成 `admin:index` ，我們可以指定 current_app 告訴他要去找哪個 `application namespace` 底下的 `instance namespace`

```
# project/member/views.py

def show(request, id=None):
    print(id)
    return HttpResponseRedirect(reverse("admin:index", current_app="users"))
```


`reverse` 若有指定 current_app 這個參數時，就會去找這個值的 namespace

沒有的話，就會去找 application namespace 

再沒有的話，就會去找 instance namespace

![Ning-draw (30)](https://imgur.com/PUBtSuT.jpg)


今天先介紹比較基礎的 URL 用法，下一篇我們會再介紹更深入的 URL 用法

如果喜歡這篇文章歡迎幫我點選下方的 Like

如果上述文章有任何疑慮或者指教，也歡迎留言