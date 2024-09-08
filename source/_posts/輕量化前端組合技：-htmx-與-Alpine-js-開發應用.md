---
title: 輕量化前端組合技： htmx 與 Alpine.js 開發應用
date: 2024-09-09 03:09:47
category: JavaScript
---

身為後端工程師，在處理前端的時候，

不是要去學框架就是要用純 JavaScript ，

在 Ruby on Rails 有 Hotwire 與 Stimulus.js，

可以輕鬆達到即時渲染及前端互動的效果，

不過在其他框架就沒有這麼簡單，

htmx 的出現，可以省去後端工程師打 API / 做即時渲染的繁複程序

而前端互動效果，就可以交給 Alpine.js

兩者搭配起來，寫出基本的前端功能並不是難事

今天就要來介紹這兩個工具如何使用

### htmx

在 HTML 中，僅有支援 `GET` 與 `POST` 方法

當我們用純 HTML 來發送 request 為 POST 方法的話，會是這樣寫

並且只有 `form` 表單才能送 `POST` 方法

```
<form action="" method="POST">
</form>
```

一般的 a 標籤只能發送 `GET` 方法

如果我們要遵守 RESTful API 的規範，要使用 PUT / PATCH / DELETE 這些方法

就必須要另外抓 DOM 元素 / 監聽事件 / 打 API ，

甚至還必須處理 JSON 格式的資料

一個畫面可能不是只處理一個，可能是好幾十個，這時候就會很頭痛

為了解決這個問題， htmx 讓我們除了透過 attribute 用 POST / GET 方法

還可以使用其他的 HTTP 方法，例如 PUT / PATCH / DELETE 等來發送 request

接著有另外一個問題是，當我們發送 request 出去時，頁面會重整換掉，

如果今天老闆希望我們做的是點下按鈕就即時渲染，

就必須透過 AJAX 來做，

不過透過 AJAX ，還是得抓 DOM 元素 / 監聽事件 / 非同步方式打 API ，

累積下來也是要耗不少工， htmx 在發送 request 的時候都是採用 AJAX 方式，

所以我們就不需要這麼辛苦了

* AJAX

AJAX 的全名為 Asynchronous JavaScript and XML ，這些字詞的意思為 非同步 JavaScript 與 XML

非同步 JavaScript 大家應該都不陌生，指的是不需要等待上一個任務執行完再執行下一個，因此系統同時可以處理多個任務

而 AJAX 使用 XMLHttpRequest 來發送 Request，格式是採用 XML 格式

回傳的 Response 可以採用 XML 或者是 JSON ，現階段大多以 JSON 格式回傳

![](https://i.imgur.com/vxXZ5xN.jpeg)

如此一來，就能馬上渲染資料

 

### Alpine.js

當我們在全端框架時，偶爾會需要做出前端互動效果，

這時候通常都會使用前端框架來達成，

不過如果只是簡單的互動效果，不使用框架，寫單純的 JavaScript ，光是抓 DOM 元素就會抓到頭痛

Alpine.js 是一款輕量的 JavaScript 框架

我們可以透過 HTML attribute 來輕鬆抓到 DOM 元素

再透過其他的 HTML attribute 來達成互動效果

不僅增添工程師使用上的便利性，也省去了學習前端框架的成本

說了這麼多優點，直接實作會讓大家更了解便利處


### 事前準備

* 已安裝 Python
* 已安裝 Poetry

### clone 專案

這次會帶大家做一次 Todo List 

可直接 clone 已做好 CRUD 的專案

`https://github.com/cacachang/django-todo`

clone 下來後，記得做幾個步驟：

* poetry shell 進入虛擬環境
* poetry install 安裝套件
* python manage.py migrate 對 migration 做 migrate
* python manage.py runserver 啟動 server
* npm install 安裝前端套件
* npm run dev 前端打包

### 加上 htmx 的 cdn 來源

在 HTMX 全站模板中加上 cdn 來源，讓我們可以使用 htmx

```
# templates/shared/base.html

<script src="https://unpkg.com/htmx.org@2.0.2" integrity="sha384-Y7hw+L/jvKeWIRRkqWYfPcvVxHzVzn5REgzbawhxAuQGwX1XWe70vji+VSeHOThJ" crossorigin="anonymous"></script>
```

### New / Create 新增待辦

接著我們就可以來做新增待辦

小目標：
1. 讓 New 畫面出現在 Todo List Index 頁面中，無需另外換頁
2. 新增完成後，在 Todo List 直接出現

那我們就先來做第一個小目標！

#### 讓 New 畫面出現在 Todo List Index 頁面中

當我們在 Todo List Index 頁面點下 `新增待辦` 時，要將 New 畫面渲染出來

所以我們就需要在 `新增待辦` 的按鈕設定 get request 的目標

htmx 支援的 http request method 及用法如下：


![](https://imgur.com/Dldv7ZZ.jpg)

而 new 頁面是使用 GET 方式，所以我們就要在 a 標籤加上 `hx-get=""`

```
# templates/items/index.html

<a href="/new" class="m-4 btn" hx-get="/new">新增待辦</a>
```

加上去之後，就可以點選看看，目前會出現在 Index 頁面，可是出現的位置不太適宜

htmx 提供了 hx-target 的 HTML 屬性，讓我們可以指定 New 畫面出現的 DOM 元素

我們在 a 標籤加上 `hx-target="#items"`

而該 DOM 元素的 id 就必須要是 `new_item`

```
# templates/items/index.html

<a href="/new" class="m-4 btn" hx-get="/new" hx-target="#new_item">新增待辦</a>
<div id="new_item"></div>
```

接著就可以重整畫面，看看結果囉！

![](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExZ2pqYTRzMHY1NmpkM2Rzd241bDRxdzVlZ212eTVua2xkamZyZXNpdSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/ZsN8A5mBBC1v5WGoB6/giphy.gif)

接著我們要來挑戰更困難一點的！

#### 新增完成後，在 Todo List 直接出現

當我把待辦表單填寫完，送出後，

Todo List 的 Index 頁面就必須要出現我剛剛新增的待辦，

新增資料我們通常會使用 POST 方法，所以我們要在 form 表單加上 `hx-post=""`

接著可以把 action 與 method 這兩個 attribute 拿掉，

看看 htmx 發揮的效果

```
# templates/items/new.html

<form class="p-4" hx-post="/new/">
  {% csrf_token %}
  ...
</form>
```

當改完，我們填寫表單並送出後，會發現多了一組 Index 頁面內容

這是因為我們在 views 中 return 了 Index 頁面的關係

必須要改成只回傳 items 這包物件

在 views 中需要改成回傳 items 的 partial 檔案，因此回傳 HttpResponse

```
# items/views.py

from django.http import HttpResponseNotAllowed, HttpResponse
from django.template.loader import render_to_string

...

def create(request):
    if request.method == 'POST':
        form = ItemForm(request.POST)

        if form.is_valid():
            item = Item(**form.cleaned_data)
            item.save()

            items = Item.objects.all()
            items_partial = render_to_string("items/_items.html", {'items': items})
            return HttpResponse(items_partial)
    return render(request, "items/new.html")
    
...
```

不過這時候出現 partial 的地方也不是我們想要出現的地方，而是應該要出現在 `待辦事項` 中，因此我們需要設定 `hx-target="#items"` (在 partial 檔案中的 id 為 items )

```
<form class="p-4" hx-post="/new/" hx-target="#items">
  {% csrf_token %}
  ...
</form>

```

這時候我們點下去，會抓到 id 為 items 的 DOM 元素，並且把整包 partial 塞進去，不過這樣畫面就跑版了

這時候我們可以透過 htmx 的 Swapping 功能，指定他塞入的方式，我們要更新該 DOM 元素的內容，所以就用 `outerHTML`

![](https://imgur.com/31apXX4.jpg)

```
<form class="p-4" hx-post="/new/" hx-target="#items" hx-swap="outerHTML">
  {% csrf_token %}
  ...
</form>
```

這樣樣式就不會跑版了

重新整理再試一次，應該就可以囉

![](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExaWwyeWFxOG50bWRqOG04cnlocjk2cXR2aGxpYW1udWY4aDNxcWJteSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/b8rzeyB4vfyDSGC10d/giphy.gif)


再來我們要來處理 Edit / Update 囉

### Edit / Update 更新待辦

小目標：
1. 讓 Edit 畫面出現在 Todo List Index 頁面中，無需另外換頁
2. 更新完成後，在 Todo List 更新

基本上應用跟 New / Create 差不多

所以我們就直接開始！

#### 讓 Edit 畫面出現在 Todo List Index 頁面中

在編輯的按鈕給予 `hx-get` 與 `hx-target`

```
# templates/items/_items.html

{% load custom_filters %}

<tbody id="items">
  {% for item in items %}
    <tr>
      ...
      <th><a href="{% url 'items:update' item.id %}" class="btn" hx-get="/edit/{{ item.id }}" hx-target="#edit_item">編輯</a></th>
      ...
    </tr>
  {% endfor %}
</tbody>
```

```
# template/items/index.html

...
<div id="new_item"></div>
<div id="edit_item"></div>
...
```

![](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExd3hoc3Q4aTF5enFkczRwcGM1eWtrZGhzeTdpNmE1aWhrcHFrZHR4biZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/dAAE5IhTJuecTLLhIS/giphy.gif)


#### 更新完成後，在 Todo List 更新

當我把表單編輯完成，送出後，

Todo List 的 Index 頁面就必須要更新剛剛的待辦，

更新資料我們會使用 POST 方法，所以我們要在 form 表單加上 `hx-post=""`

接著可以把 action 與 method 這兩個 attribute 拿掉，

看看 htmx 發揮的效果

```
# templates/items/edit.html

<form class="p-4" hx-post="/edit/{{ item.id }}">
  {% csrf_token %}
  ...
</form>
```


一樣到 views 中改成回傳 items 的 partial 檔案

```
# items/views.py

from django.http import HttpResponseNotAllowed, HttpResponse
from django.template.loader import render_to_string

...

def update(request, id):
    item = get_object_or_404(Item, id=id)

    if request.method == 'POST':
        form = ItemForm(request.POST)

        if form.is_valid():
            ...

            items = Item.objects.all()
            items_partial = render_to_string("items/_items.html", {'items': items})
            return HttpResponse(items_partial)
    else:
        form = ItemForm(item.__dict__)

    return render(request, "items/edit.html", {"form": form, "item": item})
    
...
```

為了讓它顯示在符合預期的地方，再設定 `hx-target="#items"` (在 partial 檔案中的 id 為 items )

```
# templates/items/edit.html

<form class="p-4" hx-post="/edit/{{ item.id }}" hx-target="#items">
  {% csrf_token %}
  ...
</form>

```

再加上 `hx-swap="outerHTML"` 讓他換掉 DOM 元素中的內容

```
# templates/items/edit.html

<form class="p-4" hx-post="/edit/{{ item.id }}" hx-target="#items" hx-swap="outerHTML">
  {% csrf_token %}
  ...
</form>
```

重新整理再試一次，應該就可以囉

![](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExMzN2YjRheWhsZjliNHIwZHY3eHUzeG50ZzNnbnQ1dmdmbTZ2YjlqaSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/RLYFZQ2SwlUzdOrmKs/giphy.gif)

### 當新增及編輯完成後，要讓框框消失

當完成任務後，表單就可以先隱藏起來了

這邊我們會需要用到 Alpine.js 

所以就先加上 Alpine.js 的 cdn 來源

```
# templates/shared/base.html

<head>
  ...
  <script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>
  ...
</head>
```

#### 設定框框的初始值

`x-data` 是 Alpine.js 的初始值，可以在這個屬性設定任變數與值，就可透過判斷該變數的值來完成互動效果

這邊我們會使用到 `x-data` 與 `x-show` 的屬性

`x-show` 是 Alpine.js 用來判斷該 DOM 元素是否顯示

我們先將框框最外層包上一個 div 標籤

並且透過 `x-data` 設定一個變數為 open ，值為 true ，表示他們一開始都是顯示的

```
# templates/items/new.html

<div x-data="{ open: true }">
  <h1 class="p-4 mb-5 text-3xl font-bold">新增待辦</h1>
</div>
```

接著要設定 `x-show` 為 `open` ，表示會判斷 open 的值來決定是否顯示

```
# templates/items/new.html

<div x-data="{ open: true }" x-show="open">
</div>
```

接著，當 form 表單被送出後， open 的值要變成 false，也就是不顯示

我們就要在 form 表單上做 trigger 事件，

事件為 submit，

因此我們可以這樣用 `@submit="open = ! open"`

```
# templates/items/new.html

<div x-data="{ open: true }" x-show="open">
  <h1 class="p-4 mb-5 text-3xl font-bold">新增待辦</h1>

  <form class="p-4" hx-post="/new/" hx-target="#items" hx-swap="outerHTML" @submit="open = ! open">
  </form>
</div>
```

這樣新增完成後，框框就會消失囉

![](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExNGJqZ2I2bmF4M3pmbGw3NTNmZ3JuaTdmNGRqYjhlanZ4NWRlaGNvayZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/urKGhAhxjGctr9KFgI/giphy.gif)

接著編輯框框也是一樣的

```
# templates/items/edit.html

<div x-data="{ open: true }" x-show="open">
  <h1 class="p-4 mb-5 text-3xl font-bold">編輯待辦</h1>

  <form class="p-4" hx-post="/edit/{{ item.id }}" hx-target="#items" hx-swap="outerHTML" @submit="open = ! open">
  </form>
</div>
```

由於有些待辦會更新成已完成，我們也可以透過 Alpine.js 的 `x-bind` 來依照該屬性渲染前面的勾勾

這邊我們的 `x-data` 會設定一個變數叫做 `completed` 去判斷該 item 的 status 是否完成

我們要把該變數 bind 到 class ，判斷是否完成，已完成就會是綠色的

```
# templates/items/_items.html

<tbody id="items">
  {% for item in items %}
    <tr x-data="{ completed: {{ item.status|is_completed }} }">
      <th :class="completed ? 'text-xl cursor-pointer text-green-600' : 
      'text-xl cursor-pointer'">☑</th>
      ...
    </tr>
  {% endfor %}
</tbody>
```

![](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExcmRxcWJ1c28xbm81d2N2YmFwN2Jrbnhpc21xYXBmbTR3Z2I2MHdociZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/aknBKAquwJduDvCzz6/giphy.gif)

以上，是 htmx 與 Alpine.js 的簡單應用介紹，

htmx 與 Alpine.js 並不是用來取代前端框架的，

他的目的是要讓後端工程師用更輕便且不需要打包的方式來完成前端效果，

增加網頁開發效率，也減少後端工程師的前端框架學習成本，

如果有興趣的話也可以至 [htmx](https://htmx.org/docs/#introduction) 與 [Alpine.js](https://alpinejs.dev/) 官網研究，

如果本文章對你有幫助，歡迎按讚或者留言討論 ヽ(●´∀`●)ﾉ
