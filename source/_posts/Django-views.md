---
title: Django - views 的職責與方式
date: 2024-05-12 17:58:34
category: Django
---

提到 Views 之前，我們先來認識 MTV 架構

Django 所採用的模式是 MTV 模式

使用者瀏覽網頁時，伺服器的運作流程如下：

1. urls.py 會將 request 傳送至 View

2. View 收到 request，處理過程中需要資料庫的資料時，就會向 Model 要資料

3. Model 收到 View 來的指令，將處理好的資料給 View (* Model 不是代表資料庫，而是 `負責與資料庫溝通` 的一個 Layer )

4. View 收到資料後，將資料做最後的處理

5. View 拿 Template 的東西來進行 render，最後由 View 回應給使用者

對於 Django 的 MTV 架構有一點概念後，我們就來看看 `views` 吧！

### 基本用法

我們需要做一個路徑來給 views ，好讓 Django 收到需求之後，知道該去哪裡

來看看下方例子：

我們在 url 定義了一個路徑 `users/` 

這表示，當使用者進入到該路徑時，會去找 views 的 index 方法

url 不熟的話歡迎參閱 [Django - URLS 的那些基礎用法](https://ninglab.com/Django-urls-1/)

```
# user/url.py

from django.urls import path
from . import views

urlpatterns = [
    path('users/', views.index, name="index")
]
```

在這之前，我們要先去 views 中建立名為 `index` 的方法，讓 request 過來的時候可以提供需要的資料

```
# user/views.py

def index(request):
    users = User.objects.all()    
    return render(request, "index.html", {"users": users})
```

對於 `templates` 到 `views` 有概念後，我們就可以再深入來看看 `views`

### views 的職責 - Request and Response 

views 是一種 Function ，負責接受 request ，並且回傳 response

回傳的格式可以是 html / xml 或者是 image 

頁面收到 request 時， Django 會將建立一個 `HttpRequest 物件`，會包含 request 中的 `metadata 資料`

HttpRequest 會作為`第一個參數`傳遞至對應的 views 中

剛剛提到的 `index` 例子

該引數的 request 就是 HttpRequest 物件

```
# user/views.py

def index(request):
    users = User.objects.all()    
    return render(request, "index.html", {"users": users})
```

當 views 處理好資料後，`必須` 要傳遞一個 HttpResponse 物件回去

以 `index` 例子來說，我們是回傳一個 render 方法回去

讓我們來看一下 render 這個方法

他會將參數變成一包 `HttpResponse 物件`，並且回傳

> Combines a given template with a given context dictionary and returns an HttpResponse object with that rendered text.

```
# user/views.py

def index(request):
    users = User.objects.all()    
    return render(request, "index.html", {"users": users})
```

了解 views 的職責後，我們就來看一下方式

## views

在 Django 中， views 可以用兩種方式來做

第一種是 `Function-based views` 

第二種是 `Class-based views` 

在專案中，你可以同時使用 Function-based views 或者是 Class-based views 

取決於程式碼以及需求

## Function-based views

Django 預設的 views 方式，也就是 `一般的 Function`，相較於 `Class-based views` 更簡單好寫且好閱讀

也因為簡單，通常會應用在靜態頁面中

以上方程式碼的例子來說，就是一種 `Function-based views`

```
# user/views.py

def index(request):
    users = User.objects.all()    
    return render(request, "index.html", {"users": users})
```

## Class-based views

隨著專案日益龐大，如果這時候還是使用 `Function-based views` 會有不好擴充，以及重複程式碼的缺點

`Class-based views` 就是為了解決這個問題而產生的

就以 edit 頁面來看好了，當我們使用 `Function-based views` 的話會像這樣：

```
def edit(request, id=None):
    user = get_object_or_404(User, id=id)

    if request.method == 'POST':
        form = UserForm(request.POST)

        if form.is_valid():
            user.name = form.cleaned_data["name"]
            user.age = form.cleaned_data["age"]
            user.email = form.cleaned_data["email"]
            user.address = form.cleaned_data["address"]
            user.phone = form.cleaned_data["phone"]

            user.save()

            return redirect("show", id=id)
    else:
        form = UserForm(user.__dict__)


    return render(request, "edit.html", {"form": form, "user": user})
```

但如果用 `CBV ( Class-based views )` 呢？

views 就會變成這樣：

```
class UserUpdateView(UpdateView):
    model = User
    fields = ['name', 'age', 'email', 'address', 'phone']
    template_name = 'edit.html'

    def get_object(self, queryset=None):
        id = self.kwargs.get('id')
        return User.objects.get(id=id)
```

這時候我們需要去改變 `urls.py` 中的設定

會用到 UserUpdateView 的 `as_view 類別方法` 來呼叫 UserUpdateView

```
from django.urls import path
from . import views
from .views import (UserUpdateView)

urlpatterns = [
    path('users/<int:id>/edit/', UserUpdateView.as_view(), name="edit"),
]
```

程式碼變得非常乾淨簡潔，如果需要哪些設定就再加上去就好了

### Django 中的 CBV

Django 有提供很多 CBV

以下就用 CRUD 來舉例

#### CreateView

會產生一個新建資料的 Form 表單

我們用一般的 `Function-based views` 會這樣寫

```
def new(request):
    form = UserForm()
    return render(request, "new.html", {"form": form})
```

但改成 `CBV` 就會變這樣

```
class UserCreateView(CreateView):
    model = User
    fields = ['name', 'age', 'email', 'address', 'phone']
    template_name = 'new.html'
    success_url = 'index'
```

個人覺得原本的 `Function-based views` 已經蠻簡潔了，所以也不一定要改成 `Class based views` 的方式

* `model` 是指要 create 哪個 model 的資料
* `fields` 是只有哪些欄位需要填寫
* `template_name` 是要渲染在哪個 template 中
* `success_url` 是讓你可以設定成功會導去的路徑，但如果在 model 有設定 `get_absolute_url() ` 就不需要再設定這個， Django 會直接去抓 `get_absolute_url() ` 的設定

#### UpdateView

會渲染已有資料的表單，前提是要提供參數去找出資料

使用 `Function-based views` 

```
def edit(request, id=None):
    user = get_object_or_404(User, id=id)

    if request.method == 'POST':
        form = UserForm(request.POST)

        if form.is_valid():
            user.name = form.cleaned_data["name"]
            user.age = form.cleaned_data["age"]
            user.email = form.cleaned_data["email"]
            user.address = form.cleaned_data["address"]
            user.phone = form.cleaned_data["phone"]

            user.save()

            return redirect("show", id=id)
    else:
        form = UserForm(user.__dict__)


    return render(request, "edit.html", {"form": form, "user": user})
```

使用 `CBV` 

```
class UserUpdateView(UpdateView):
    model = User
    fields = ['name', 'age', 'email', 'address', 'phone']
    template_name = 'edit.html'
    success_url = reverse_lazy('index')

    def get_object(self):
        id = self.kwargs.get('id')
        return User.objects.get(id=id)
```

使用 UpdateView 是需要傳遞 `pk` / `slug` 去讓他找出資料

不過 User 剛好沒有這兩個欄位，所以就用了 `get_object` 來找資料

#### DeleteView

只要 request 是 POST ，且資料存在的話，就會刪除

用 `Function-based views`

```
def delete(request, id=None):
    if request.method == "POST":
        user = get_object_or_404(User, id=id)

        user.delete()
        return redirect("index")
    else:
        return HttpResponseNotAllowed(["POST"])
```

使用 `CBV`

```
class UserDeleteView(DeleteView):
    model = User
    success_url = reverse_lazy("index")

    def get_object(self):
        id = self.kwargs.get('id')
        return User.objects.get(id=id)
```

Django 的 `Function-based views` 以及 `Class-based views` 各有擁護者，

且 `Class-based views` 並不是用來取代 `Function-based views` 的，他們各有所長

至於要使用哪個方式，我個人認為還是要照需求以及團隊的 coding style 來決定
