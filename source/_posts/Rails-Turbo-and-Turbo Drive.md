---
title: Rails - Turbo 與 Turbo Drive
date: 2024-03-04 01:53:18
category: Rails
description: 使用 Rails 7 ，Turbo 讓你的網頁比別人更快
---

# Turbo

## 瀏覽器的載入模式

在 AJAX 還沒出現以前，使用者在瀏覽網頁，進行互動後，需要重新整理才會在網頁上看到互動後的結果

![](https://i.imgur.com/CFwObAB.jpeg)


## AJAX 模式

AJAX 的全名為 Asynchronous JavaScript and XML ，這些字詞的意思為 非同步 JavaScript 與 XML

非同步 JavaScript 大家應該都不陌生，指的是不需要等待上一個任務執行完再執行下一個，因此系統同時可以處理多個任務

而 AJAX 使用 XMLHttpRequest 來發送 Request，格式是採用 XML 格式

回傳的 Response 可以採用 XML 或者是 JSON ，現階段大多以 JSON 格式回傳

![](https://i.imgur.com/vxXZ5xN.jpeg)

如此一來，就能馬上渲染資料

![giphy](https://i.imgur.com/h0MzwaM.gif)

讓我們用 JS 的角度來看一下 AJAX 會是怎麼發送的，就會比較好理解 Turbo 的原理

```
const button = document.getElementById("button")

button.addEventListener('click', function(){
  let xhr = new XMLHttpRequest();

  xhr.open('GET', 'ajax.txt', true);

  xhr.send();

  xhr.onload = function(){
    if (this.status == 200) {
      let text = document.getElementById("text")
      text.innerHTML = this.responseText
    }
  }
})
```

其實原理跟一般的事件監聽很像，
差別在於 `XMLHttpRequest` 會透過 url 來獲取資料，
而 `xhr.open` 第三個參數指的是是否採取同步或非同步，
`true` 表示採取非同步

所以後續的程式碼都會一起執行，
並且及時地將拿回來的 Response 資料渲染



## 關於 Turbo

Rails 透過 Turbo 來加速網頁的載入速度，

Turbo 是 AJAX 技術的應用，但他做的事會比 AJAX 更多

而 Turbo 又分成幾個功能

1. Turbo Drive
加速載入頁面，讓使用者感受不到頁面之間的切換

2. Turbo Frame
不需要寫大量的 DOM 元素就能在同樣的頁面渲染表單或資訊

3. Turbo Stream
透過 Action Cable 即時渲染 Response 的內容

4. Turbo Native
用 Turbo 來打造 App (不過目前尚未設略，暫時不介紹)



今天我們會先介紹 Turbo Drive

## 關於 Turbo Drive

Turbo Drive 就是以前的 Turbolinks

用 Rails 寫過專案的人應該多少都遇過在點擊連結的時候完全沒作用，是因為 Turbolinks 把所有的連結預設事件都停掉了，在 Rails 5 ~ Rails 6 版本的專案，我們可以用 `data-turbo="false"` 來停掉連結的 Turbolinks

而在 Rails 7 中， Turbo Drive 不只將連結的預設事件停掉，連表單的預設事件也被停掉了，這樣做的目的是為了提高載入的速度！

只要掌握 Turbo Drive 的原理，它就會很好用！

### 從 Visit 開始的生命週期

在使用者點擊的那一刻， Turbo Drive 的生命週期就開始了，一直到 render 完頁面才結束

Turbo Drive 在 render 的時候會做以下這幾件事情

1. 將 body 換成 Response 的 body
2. 如果 head 的 title 以及 meta 改變，會將 head 合併，反之，head 將保持原樣
3. 有需要的話會將 html 的 lang 標籤更新
4. 除此之外，還會將 URL 更換掉

Turbo 會有幾個生命週期，

`turbo:click`
`turbo:before-visit`
`turbo:visit`
`turbo:before-cache`
`turbo:before-render`
`turbo:render`
`turbo:load`

我們先來看前置作業，當點擊連結，並且拜訪 URL 以前，忙碌的 Turbo 做了哪些事

### session.visit - Turbo 開始

當點擊時， Turbo 會先去判斷是否為 turbo frame，不是的話就會去拜訪該 URL

```
// src/core/session.js:97

visit(location, options = {}) {
  const frameElement = options.frame ? document.getElementById(options.frame) : null

  if (frameElement instanceof FrameElement) {
    ...
  } else {
    this.navigator.proposeVisit(expandURL(location), options)
  }
}
```

### Navigator - 停止預設事件

開始導去 URL 的時候，會先判斷該 URL 是否為同一頁，或者不同頁但是觸發 `turbo:before-visit` 事件，並且把預設事件停掉，就會開始接下來的訪問

```
// src/core/drive/navigator.js:12

proposeVisit(location, options = {}) {
  if (this.delegate.allowsVisitingLocationWithAction(location, options.action)) {
    this.delegate.visitProposedToLocation(location, options)
  }
}
```

### Session - 開始拜訪

在這個階段會先將 URL 做新舊版本的轉換，避免舊版本無法支援(這邊的版本是指 Turbo Native 的 adapter)
接著就會請瀏覽器去拜訪了

```
// src/core/session.js:239

visitProposedToLocation(location, options) {
  extendURLWithDeprecatedProperties(location)
  this.adapter.visitProposedToLocation(location, options)
}
```

### Browser Adapter - 拜訪前的敲門鈴確認

這時候 Turbo 會去確認 URL 的網域是不是一致的，一致才會繼續拜訪

```
// src/core/native/browser_adapter.js:13

visitProposedToLocation(location, options) {
  if (locationIsVisitable(location, this.navigator.rootLocation)) {
    this.navigator.startVisit(location, options?.restorationIdentifier || uuid(), options)
  } else {
    window.location.href = location.toString()
  }
}
```

### Navigator - 點下連結後的導向

在導向新的 URL 時，就會透過 expendURL 將 URL 設定成新的，不過不是在這邊換掉，是在 Visit 開始的時候才會換掉

```
// src/core/drive/navigator.js:18

startVisit(locatable, restorationIdentifier, options = {}) {
  this.stop()
  this.currentVisit = new Visit(this, expandURL(locatable), restorationIdentifier, {
    referrer: this.location,
    ...options
  })
  this.currentVisit.start()
}
```

```
// src/core/url.js:1

export function expandURL(locatable) {
  return new URL(locatable.toString(), document.baseURI)
}
```

### Visit - Turbo Drive 開始生效

當確認好狀態為 `initialized` 並且定義完基本設定後，就會開始了

```
// src/core/drive/visit.js:117

start() {
  if (this.state == VisitState.initialized) {
    this.recordTimingMetric(TimingMetric.visitStart)
    this.state = VisitState.started
    this.adapter.visitStarted(this)
    this.delegate.visitStarted(this)
  }
}
```

### Browser Adapter - 改頭換面的開始

當開始 Visit 時，第一件事會將 URL 換掉

```
// src/core/native/browser_adapter.js:21

visitStarted(visit) {
  this.location = visit.location
  visit.loadCachedSnapshot()
  visit.issueRequest()
  visit.goToSamePageAnchor()
}
```

接著 Turbo Drive 需要判斷連結有沒有在 Cached 中，有的話就會直接渲染 Cached 中的頁面

```
// src/core/drive/visit.js:247

loadCachedSnapshot() {
  const snapshot = this.getCachedSnapshot()
  if (snapshot) {
    const isPreview = this.shouldIssueRequest()
    this.render(async () => {
      this.cacheSnapshot()
      if (this.isSamePage || this.isPageRefresh) {
        this.adapter.visitRendered(this)
      } else {
        if (this.view.renderPromise) await this.view.renderPromise

        await this.renderPageSnapshot(snapshot, isPreview)

        this.adapter.visitRendered(this)
        if (!isPreview) {
          this.complete()
        }
      }
    })
  }
}
```

接著會去看 Cached 是否已經有 Response ，
有的話就會模擬之前的 Request ，
沒有的話就會發送一個新的 Request

為什麼可以先載入 Response ，可參考此 [MDN 的說明](https://developer.mozilla.org/en-US/docs/Web/API/FetchEvent/preloadResponse)

```
// src/core/drive/visit.js:166

issueRequest() {
  if (this.hasPreloadedResponse()) {
    this.simulateRequest()
  } else if (this.shouldIssueRequest() && !this.request) {
    this.request = new FetchRequest(this, FetchMethod.get, this.location)
    this.request.perform()
  }
}
```

再來就是判斷新 URL 跟目前所在的頁面是否為同一頁，
同一頁的話就會挪到指定的 Anchor 

```
// src/core/drive/visit.js:281

  goToSamePageAnchor() {
    if (this.isSamePage) {
      this.render(async () => {
        this.cacheSnapshot()
        this.performScroll()
        this.changeHistory()
        this.adapter.visitRendered(this)
      })
    }
  }
```

### Session - 拜訪

接著

假設目前的 Visit 並沒有接受 Stream 格式的 Response ，就將它標記為正在處理中，這個處理可能是在處理 Request 以及換頁，

總之，不是以 Turbo Drive 的方式去處理

接著會判斷是在看是不是為同一頁，同一頁的話就會觸發 `turbo:visit` 並訪問該 URL

```
// src/core/session.js:246

  visitStarted(visit) {
    if (!visit.acceptsStreamResponse) {
      markAsBusy(document.documentElement)
      this.view.markVisitDirection(visit.direction)
    }
    extendURLWithDeprecatedProperties(visit.location)
    if (!visit.silent) {
      this.notifyApplicationAfterVisitingLocation(visit.location, visit.action)
    }
  }
```

以上只是渲染之前所做的事情 ( 實際上要處理的應該遠比我提的更多，我這邊只先提個大致的流程 )


### 不同的 Visit

Turbo Drive 又分成兩種 Visit

一種是 `Application Visits` ，另一種是 `Restoration Visits`

在這之前我們可以來了解一下 Turbo Drive 處理 Cached 的機制

Turbo Drive 會將最近造訪過的 URL 暫存，有兩個目的

1. 在歷史紀錄中切換 URL ( 即所謂的 Restoration Visits  ) 的時候，不需要發送 Request 就能渲染
2. 在一般的 URL 切換中，能快速渲染頁面(但還是會發送 Request)，讓使用者感受不到有在切換

Application Visits 將會發送 Request ，並且如果 Cached 有該 URL 頁面資訊的話，就會渲染頁面資訊

Restoration Visits 
使用者點擊了瀏覽器的上一頁，或是下一頁，因為已經瀏覽過， Cached 中可能會有資料，所以就不會發送 Request ，而是直接 render Cached 裡面的頁面資訊

上述兩種 Visit 最大的差別應該是在於是否為歷史紀錄中的切換以及是否有發送 Request
