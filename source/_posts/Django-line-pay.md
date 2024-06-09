---
title: Django - LINE Pay 串接實戰
date: 2024-06-10 03:11:18
category: Django
---

LINE Pay 已經是現在大眾很習慣的付款方式

不過目前用 Django 串接 LINE Pay 的資源很少

所以今天想跟大家分享如何在 Django 中串接 LINE Pay

分成以下四個步驟

1. 申請 SandBox 帳號密碼
2. 申請 Channel ID & Key
3. 串接 Request
4. 串接 Confirm

因為目前還沒有正式的商店，所以就先用 Sandbox 

* 使用 LinePay v3

### GitHub 參考

https://github.com/cacachang/django_line_pay

## Step 1. LINE Pay 相關申請

至 [LINE Pay Developers](https://pay.line.me/tw/developers/techsupport/sandbox/creation?locale=zh_TW) 申請建立 Sandbox
填寫完畢後，至 Mail 收信拿 Sandbox 帳號密碼

![LINE Pay Sandbox](https://imgur.com/3G965Ag.jpg)

登入 [LINE Pay 後台](https://pay.line.me/portal/tw/auth/login)

![LINE Pay](https://imgur.com/FaMU0kZ.jpg)


登入成功後，進入後台申請 Channel ID & Channel Secret Key

收到 Mail，填入驗證碼後就可以拿到 Channel ID & Channel Secret Key

![LINE Pay](https://imgur.com/HpA6RUG.jpg)

將 Channel ID & Channel Secret Key 放到環境變數中

```
# .env

LINE_CHANNEL_ID=xxxxxxxx
LINE_CHANNEL_SECRET_KEY=xxxxxxxx
```

## Step 2. 向 LINE Pay 請求付款資訊

我們要使用 LINE Pay 來做付款，

請求付款是要打 Request API 

所以我們必須要發送 request 到該 API


#### 設定 urls 與 views

金流的資料相對比較機密，我會採用後端的方式去打 API 

所以我們就需要設定一個 urls 與 views 

當使用者按下結帳按鈕時，就會至該網址，並且於 views 發送 request 給 LINE Pay

```
# payment/urls.py

from django.contrib import admin
from django.urls import path
from . import views

app_name = "payment"

urlpatterns = [
    path('request', views.request, name='request'),
]

```

記得也要加入 templates

#### 組裝 request 需要的 headers & body

官方手冊 Request API 的規格

方法 `POST`
URL `/v3/payments/request`

headers 會需要 body 的資料，所以我們就先來建立 body

#### 建立 body

這邊我們先建立必填的 items

![LINE Pay body](https://imgur.com/CNHLU4F.jpg)

items 之多，圖片不及備載，如果有額外需求，可以至[官方手冊](https://pay.line.me/tw/developers/apis/onlineApis?locale=zh_TW)翻相關的 items 並加入

接下來我們就先來做 body 吧

基本的總額 / 幣別，是串金流非常基本的參數，這邊就不多做介紹

`orderId` 需放訂單的 `id`，到時候 API 打回來的 response 會有 orderId 讓我們去找到該 order 
* 文章用所以先用 uuid 來代替，在專案上請使用訂單的 id 來取代

`packages` 把他當成商品組合，要塞該商品的 id 以及總額，如果不是組合的話，`package_id` 可以用 uuid 帶過即可，`products` 則是組合中的商品們

`redirectUrls` 在我們發送 request 出去後，取消付款以及付款確認後要轉回來的網址
* HOSTNAME 環境變數可以使用 NGROK 或網域

```
# payment/views.py

from django.shortcuts import render
from django.utils import timezone
from pathlib import Path
from django.shortcuts import redirect
import os
import uuid
import environ

BASE_DIR = Path(__file__).resolve().parent.parent
env = environ.Env()
environ.Env.read_env(os.path.join(BASE_DIR, '.env'))

def request(request):
    if request.method == "POST":
        order_id = f"order_{str(uuid.uuid4())}"
        package_id = f"package_{str(uuid.uuid4())}"

        payload = {
            'amount': 100,
            'currency': 'TWD',
            'orderId': order_id,
            'packages': [{
                'id': package_id,
                'amount': 100,
                'products': [{
                    'id': '1',
                    'name': '測試商品',
                    'quantity': 1,
                    'price': 100,
                }]
            }],
            'redirectUrls': {
                'confirmUrl': f"https://{env('HOSTNAME')}/payment/confirm",
                'cancelUrl': f"https://{env('HOSTNAME')}/payment/cancel"
            }
        }

    else:
        return render(request, "payment/checkout.html")
```


#### 建立 headers

當我們把 body 做好後，就該來做 headers 了

先來看一下官方手冊吧

`Content-Type` 指的是使用的資料格式
`X-LINE-ChannelId` 填入剛剛申請的 Channel ID，這樣才知道是哪間商店
`X-LINE-MerchantDeviceProfileId` 目前沒有實體店，略過
`X-LINE-Authorization-Nonce` 是為了避免搞混多筆一樣的交易，所以多這個參數
`X-LINE-Authorization` Hash 過後的交易及其他資料(包含 body)

![LINE Pay Request Header](https://imgur.com/1DsAdiO.jpg)

我們先列出 header 必要的 key 

```
# payment/views.py

def create_headers(body, uri):
    headers = {
        'Content-Type': 'application/json',
        'X-LINE-ChannelId': '',
        'X-LINE-Authorization-Nonce': '',
        'X-LINE-Authorization': ''
    }

    return headers
```

#### X-LINE-ChannelId

接著我們將 Channel Id 從環境變數取出作為 `X-LINE-ChannelId` 的 value

```
# payment/views.py

def create_headers(body, uri):
    channel_id = env('LINE_CHANNEL_ID')

    headers = {
        'Content-Type': 'application/json',
        'X-LINE-ChannelId': channel_id,
        'X-LINE-Authorization-Nonce': '',
        'X-LINE-Authorization': ''
    }

    return headers
```

#### X-LINE-Authorization-Nonce

再來製作一個 nonce ，目的在於區別各個交易

我們就用 uuid 來做吧

```
# payment/views.py

def create_headers(body, uri):
    ...
    nonce = str(uuid.uuid4())

    headers = {
        'Content-Type': 'application/json',
        'X-LINE-ChannelId': channel_id,
        'X-LINE-Authorization-Nonce': nonce,
        'X-LINE-Authorization': ''
    }

    return headers
```


#### X-LINE-Authorization

headers 中以 `X-LINE-Authorization` 處理起來比較耗費心力一點

我們先來看一下官方手冊給的公式

> Signature = Base64(HMAC-SHA256(Your ChannelSecret, (Your ChannelSecret + URI + RequestBody + nonce)))

`Your ChannelSecret` 是我們剛剛申請的 Channel Secret Key 

`URI` 由於我們是打 Request API，所以這邊指的是 `/v3/payments/request` 
**要注意，不需要加上 Sandbox 的 URL**

`RequestBody` 我們剛剛組合的 body (也就是 payload)

`nonce` 可使用 uuid 或 timestamp 產生

我們先將這四個資料找出來，並且串起來

#### secret_key

```
# payment/views.py

def create_headers(body, uri):
    ...
    secret_key = env('LINE_CHANNEL_SECRET_KEY')
    

    headers = {
        'Content-Type': 'application/json',
        'X-LINE-ChannelId': channel_id,
        'X-LINE-Authorization-Nonce': nonce,
        'X-LINE-Authorization': ''
    }

    return headers
```

#### uri / body

由於 Confirm API 也會需要用該方法建立 header 

所以 body 與 uri 就用引數

我們要在呼叫 `create_headers` 方法時帶入

uri 已經存在環境變數 `LINE_SIGNATURE_REQUEST_URI`中了，所以將他叫出來即可，要注意值是 `/v3/payments/request`

body 則是我們一開始組裝的 payload

```
# payment/views.py

def request(request):
    if request.method == "POST":
        ...

        payload = {
            ...
        }

        signature_uri = env('LINE_SIGNATURE_REQUEST_URI')        
        headers = create_headers(payload, signature_uri)

    else:
        return render(request, "payment/checkout.html")
```

因為 headers 的 Content-Type 格式是 application/json
所以當 body 要記得轉成 json 格式

```
# payment/views.py

import json
...

def create_headers(body, uri):
    ...
    body_to_json = json.dumps(body)

    headers = {
        'Content-Type': 'application/json',
        'X-LINE-ChannelId': channel_id,
        'X-LINE-Authorization-Nonce': nonce,
        'X-LINE-Authorization': ''
    }

    return headers
```

#### nonce

我們可以用剛剛做出來的 nonce ，這邊就不需要再另外做了

```
# payment/views.py

def create_headers(body, uri):
    ...

    headers = {
        'Content-Type': 'application/json',
        'X-LINE-ChannelId': channel_id,
        'X-LINE-Authorization-Nonce': nonce,
        'X-LINE-Authorization': ''
    }

    return headers
```

#### 組裝成 message

Signature 公式
> Signature = Base64(HMAC-SHA256(Your ChannelSecret, (Your ChannelSecret + URI + RequestBody + nonce)))

四個資料都準備好後，就可以把他們組裝起來了

```
# payment/views.py

def create_headers(body, uri):
    ...
    nonce = str(uuid.uuid4())
    secret_key = env('LINE_CHANNEL_SECRET_KEY')
    body_to_json = json.dumps(body)
    message = secret_key + uri + body_to_json + nonce

    headers = {
        'Content-Type': 'application/json',
        'X-LINE-ChannelId': channel_id,
        'X-LINE-Authorization-Nonce': nonce,
        'X-LINE-Authorization': ''
    }

    return headers
```

#### 對 message 及 secret_key 做 hash

我們再來看一下官方手冊給的公式

> Signature = Base64(HMAC-SHA256(Your ChannelSecret, (Your ChannelSecret + URI + RequestBody + nonce)))

組起來後我們就要先來對這包資料做 HMAC

HMAC 是一種雜湊方式，可以保有完整資料，而且也可以拿來做身份的驗證

我們在傳送資料的過程中，隨時有可能會被有心人士更改，

將資料加上雜湊，可以讓資料難以被更改以外

也可以透過解析來做身份驗證
(判斷接收方與發送方計算出來的是否一致)

由於 hmac 方法只接受 bytes 或 bytearray 資料

#### 改資料為 bytes 

所以我們得先將 secret_key 與 message 轉為 bytes 格式

`encode` 會將字串轉換格式，預設是轉換為 `utf-8` 

UTF-8 編碼是將字串轉為位元組的方式

```
# payment/views.py

def create_headers(body, uri):
    ...
    secret_key = env('LINE_CHANNEL_SECRET_KEY')
    body_to_json = json.dumps(body)
    nonce = str(uuid.uuid4())
    message = secret_key + uri + body_to_json + nonce
    
    binary_message = message.encode()
    binary_secret_key = secret_key.encode()

    headers = {
        'Content-Type': 'application/json',
        'X-LINE-ChannelId': channel_id,
        'X-LINE-Authorization-Nonce': nonce,
        'X-LINE-Authorization': ''
    }

    return headers
```

#### HMAC 處理

Signature 公式
> Signature = Base64(HMAC-SHA256(Your ChannelSecret, (Your ChannelSecret + URI + RequestBody + nonce)))

hmac 第一個引數為 key ，這邊我們要放 binary_secret_key

第二個引數為 message，要放 binary_message

第三個引數為雜湊模式，要用 SHA256 的方式雜湊，第三個參數要加上 `hashlib.sha256`

> hmac.new(key, msg=None, digestmod)

```
# payment/views.py

import hmac
import hashlib

...

def create_headers(body, uri):
    ...
    secret_key = env('LINE_CHANNEL_SECRET_KEY')
    body_to_json = json.dumps(body)
    nonce = str(uuid.uuid4())
    message = secret_key + uri + body_to_json + nonce
    
    binary_message = message.encode()
    binary_secret_key = secret_key.encode()
    
    hash = hmac.new(binary_secret_key, binary_message, hashlib.sha256)

    headers = {
        'Content-Type': 'application/json',
        'X-LINE-ChannelId': channel_id,
        'X-LINE-Authorization-Nonce': nonce,
        'X-LINE-Authorization': ''
    }

    return headers
```

#### Base 64 

Signature 公式
> Signature = Base64(HMAC-SHA256(Your ChannelSecret, (Your ChannelSecret + URI + RequestBody + nonce)))

Base64常用於處理文字資料，可用來傳輸

如想了解更細部的資訊，可參閱 [Base64](xhttps://zh.wikipedia.org/wiki/Base64)

做好就可以作為 `X-LINE-Authorization` 的 value 了

```
# payment/views.py

import base64
...

def create_headers(body, uri):
    ...
    secret_key = env('LINE_CHANNEL_SECRET_KEY')
    body_to_json = json.dumps(body)
    nonce = str(uuid.uuid4())
    message = secret_key + uri + body_to_json + nonce
    
    binary_message = message.encode()
    binary_secret_key = secret_key.encode()
    
    hash = hmac.new(binary_secret_key, binary_message, hashlib.sha256)
    
    signature = base64.b64encode(hash.digest()).decode()

    headers = {
        'Content-Type': 'application/json',
        'X-LINE-ChannelId': channel_id,
        'X-LINE-Authorization-Nonce': nonce,
        'X-LINE-Authorization': signature
    }

    return headers
```


#### 打 API 囉！

我們將 headers 處理好後，

接著將 body 轉為 json 

並且設定好要打的 url (請用環境變數存放) 為 `https://sandbox-api-pay.line.me/v3/payments/request`


就可以打 API 出去了

```
# payment/views.py

import requests
...

def request(request):
    if request.method == "POST":
        url = f"{env('LINE_SANDBOX_URL')}{env('LINE_REQUEST_URL')}"
        ...
        body = json.dumps(payload)

        response = requests.post(url, headers=headers, data=body)

    else:
        return render(request, "payment/checkout.html")
```

### 接收 response

打過去 Request API 後，會回傳一包 response 

這時候 LINE Pay 的付款頁面就會在這包裡面

我們要將 response 解析

成功的回傳代碼是 `0000`

如成功的話，將頁面網址找出來再 redirect 過去

失敗的話就印出回傳訊息

而付款頁面是放在 info 的 paymentUrl 的 web

```
# payment/views.py

def request(request):
    if request.method == "POST":
        url = f"{env('LINE_SANDBOX_URL')}{env('LINE_REQUEST_URL')}"
        ...

        response = requests.post(url, headers=headers, data=body)
        
        if response.status_code == 200:
            data = response.json()
            if data['returnCode'] == '0000':
                return redirect(data['info']['paymentUrl']['web'])
            else:
                print(data['returnMessage'])
                return render(request, "payment/checkout.html")
        else:
            print(f'Error: {response.status_code}')
            return render(request, "payment/checkout.html")

    else:
        return render(request, "payment/checkout.html")
```

### 設定 CSRF 以及 ALLOWED_HOST

由於我這邊使用 NGROK ，是外部的服務

除了要告訴 Djanog 哪些域名是可以被訪問的以外

還要避免偽造請求

所以需要再額外設定 CSRF 與 ALLOWED_HOST

```
# core/settings.py

ALLOWED_HOSTS = [
    env('HOSTNAME')
]

CSRF_TRUSTED_ORIGINS = [
    f"https://{env('HOSTNAME')}",
]
```

到這邊應該就可以前進 LINE Pay 付款了

![LINE Pay](https://imgur.com/y5sCFQa.jpg)

手機掃描付款畫面

![LINE Pay](https://imgur.com/OoA0eWb.jpg)


## Step 3. 向 LINE Pay 申請完成交易

付款申請差不多到這邊，接著消費者完成交易後，店家這邊需要再做確認

所以再來要打 Confirm API

由於處理方式跟 Request API 差不多，類似的地方我就不多做介紹

#### 設定 urls 與 views

一樣設定 urls 與 views 

當使用者付款成功後，會馬上轉到 confirm 的 url 去完成交易

```
# payment/urls.py

from django.contrib import admin
from django.urls import path
from . import views

app_name = "payment"

urlpatterns = [
    path('request', views.request, name='request'),
    path('confirm', views.confirm, name="confirm"),
]

```

記得也要加入 templates

#### 組裝 request 需要的 headers & body

官方手冊 Request API 的規格

方法 `POST`
URL `/v3/payments/{transactionId}/confirm`

在組裝 headers 與 body 以前，我們要先拿到該筆交易的 id 讓 LINE Pay 與我們辨認

### 拿到 transaction_id 與 order_id

Confirm API 的 url 需要包含 transaction_id 

LINE Pay 才會知道是要確認哪筆款項

所以我們要從 requests 中找到 `transactionId`

orderId 則是方便我們從資料庫中找出資料，並針對該訂單進行更新

```
# payment/views.py

def confirm(request):
    transaction_id = request.GET.get('transactionId')
    order_id = request.GET.get('orderId')
```

### 組裝 body

打 Confirm API 一樣需要組裝 body 

只要包含 amount 與 currency 即可

這邊需要去資料庫中撈出該筆 order_id 的資料，並且將 amount 填進去

不過展示所以就先寫假資料

![LINE Pay Confirm](https://imgur.com/sUfSctm.jpg)

```
# payment/views.py

def confirm(request):
    transaction_id = request.GET.get('transactionId')
    order_id = request.GET.get('orderId')
    
    payload = {
        'amount': 100,
        'currency': 'TWD',
    }
```

### 建立 headers

跟剛剛的 Request API 一樣

只要將 body 與 uri 做為參數傳進去即可

要記得 uri 要塞 transaction_id 進去 `/v3/payments/{transaction_id}/confirm`

```
# payment/views.py

def confirm(request):
    transaction_id = request.GET.get('transactionId')
    order_id = request.GET.get('orderId')
    
    payload = {
        'amount': 100,
        'currency': 'TWD',
    }
    
    signature_uri = f"/v3/payments/{transaction_id}/confirm"
    headers = create_headers(payload, signature_uri)
```

### 發送 reqeust 到 Confirm API 

將 body 轉成 json 格式後

設定 url 後，就可以打 API 過去了

url 的網址為 `https://sandbox-api-pay.line.me/v3/payments/{transaction_id}/confirm`

```
# payment/views.py

def confirm(request):
    transaction_id = request.GET.get('transactionId')
    order_id = request.GET.get('orderId')
    url = f"{env('LINE_SANDBOX_URL')}/v3/payments/{transaction_id}/confirm"
    
    body = json.dumps(payload)

    response = requests.post(uri, headers=headers, data=body)
```

#### 解析 response

這邊的步驟也跟 Request API 一樣

`returnCode` 成功為 `0000`

如果是 0000 的話就轉到付款成功頁面

失敗就轉至失敗頁面，並可設定錯誤訊息

```
# payment/views.py

def confirm(request):
    ...

    response = requests.post(uri, headers=headers, data=body)
    
    data = response.json()
    if data['returnCode'] == '0000':
        return render(request, "payment/success.html")
    else:
        print(data['returnMessage'])
        return render(request, "payment/fail.html")
```

到這邊就差不多囉，接著就把 success 與 fail 的 urls / views / templates 補上就可以囉

如果我們一開始在打 Request API 的時候沒有設定 options.payment.capture 為 false 的話，到這步就完成了

如果設定為 false 的話，狀態會保持 `待請款（授權）` ，必須要再打 `Capture API`


如果這篇文章對你有幫助的話，歡迎按讚或留言

有任何想討論的，也歡迎底下留言給我唷
