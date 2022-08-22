---
title: 從0開始刻 淺談 Rails 的運作魔法 - Day03 GEM
date: 2022-08-16 23:27:08
tags:
categories: Rebuild Rails

---
用過 Rails 應該對於 gem 不會太陌生
好用一直用，但你知道 gem 是什麼嗎？


## Rubygem

Rubygem 就很像一個五金行
囊括所有應用在Ruby上的套件

![](https://i.imgur.com/Fdf9rk1.jpg)


## GEM是什麼？

想像一下，當我們要組家具
大部分的人應該都直接去家具店挑
或是到 IKEA 買回來 DIY
少數專業的師傅可能會自己取材製作

其中 IKEA 販售的傢俱組合
被完善處理過的零組件就像是GEM
可以讓我們更快速地組好傢俱
且功能也很完整✨



寫程式時，不可能所有功能都手刻
所以我們需要適時引入套件
讓我們寫的網頁、應用程式功能更完善

![](https://i.imgur.com/UqV5nHz.jpg)


不同的GEM套件有不同的功能
這邊舉例一些比較常用的GEM

1. 加入特別的功能到專案中
2. 更順利的安裝API
3. 建立一個網頁應用程式

有這些套件，我們就能專心在開發上👩‍💻👨‍💻🧑‍💻
順帶一提，我們所使用的 Rails 也是 GEM 的一種哦！

</br>

## GEM套件組成

讓我們來拆解一下 GEM 裡面的檔案
之後手刻會遇到，各位可以稍微了解一下😉

![](https://i.imgur.com/9kQkc9i.jpg)



## gem指令

既然知道 GEM 這麼好用
那我們也要知道如何使用！

### 安裝

在終端機打 gem install + GEM 名稱即可安裝

```ruby=
gem install nokogiri
```

### 搜尋

搜尋 Rubygem 裡面的套件，可以搭配正則表示法

```ruby=
> gem search rack

*** REMOTE GEMS ***

airake (0.4.5)
airbrake (13.0.2)
airbrake-api (4.6.1)
airbrake-extended (0.0.8)
airbrake-faraday_sender (0.1.1)
```

### 列出已經安裝的套件

開啟套件清點簿！

```ruby=
> gem list

*** LOCAL GEMS ***

actioncable (7.0.3, 6.1.6.1, 6.1.6, 6.1.4.6)
actionmailbox (7.0.3, 6.1.6.1, 6.1.6, 6.1.4.6)
actionmailer (7.0.3, 6.1.6.1, 6.1.6, 6.1.4.6)
actionpack (7.0.3, 6.1.6.1, 6.1.6, 6.1.4.6, 5.2.2)
actiontext (7.0.3, 6.1.6.1, 6.1.6, 6.1.4.6)
```

### 卸載

一言不合就解除！解除安裝套件 
```ruby=
gem uninstall rake
```

</br>

## 我知道GEM是什麼了，但是Gemfile又是什麼？

像是食譜的食材清單
要成就一道美食需要哪些食材

Gemfile 寫著你的專案需要哪些gem套件及版本
而且不需要使用 require 就能安裝好

下圖是 Gemfile 檔案，裡面清楚註明套件及版本

![](https://i.imgur.com/5nZIR5D.png)



咦，不用 require 的話，又沒有下達 gem 指令，這些套件要怎麼安裝？

這時候我們就要提到 Bundler

## Bundler 

Bundler 像是一種相依性管理的工具
雖然說 gem 可以幫我們安裝套件
但套件有各種版本
版本與版本之間也不一定相依
Bundler 可以幫我們解決版本相依性的問題

📌 生活小例子
Bundler 很像調解會
負責調解人跟人之間的衝突
期待化解爭執

雖然 Bundler 會幫我們解決版本相依性問題
不過 Bundler 之前，還是要記得確認 gemfile 裡面的套件及版本哦 😉



## 除了Gemfile，還有gemspec說明書

.gemspec 包含了 gem 的名字、簡短介紹、作者名稱、相依套件的清單、gem所包含的檔案清單，可能還會有 hompage 等等

如果你想看 gem 的資訊，就可以來這個檔案參觀🤓

```shell=
Gem::Specification.new do |spec|
 spec.name          = "rulers"
 spec.version       = Rulers::VERSION
 spec.authors       = ["Ning"]
 spec.email         = ["a24701770@gmail.com"]

 spec.summary       = %q{"rulers"}
 spec.description   = %q{}
 spec.homepage      = "https://ninglab.com/"
 spec.required_ruby_version = Gem::Requirement.new(">= 2.3.0")
```



## Bundler後出現的 Gemfile.lock

gemfile.lock 像是版本管理資料
說明你安裝的套件現在是用哪個版本的


## 版本介紹

在 gemfile 應該很常看到一堆數字
有時候還會出現 ~>
我們來了解一下這些符號跟數字是什麼意思

```ruby=
# 指定安裝版本 6.1.4.6
gem 'rails', '6.1.4.6'
```

```ruby=
# 安裝大於或等於 1.4.4 的版本
gem 'bootsnap', '>= 1.4.4'
```

```ruby=
# 安裝大於 5.0 且相對穩定的版本
gem 'webpacker', '~> 5.0'
```

現在我們來看一下數字的部分
以 1.0.2 這個版本來看
每個數字及位置各有代表的版本意思

主要版號：將功能大修改，通常硬裝都會失敗，要解除原版本再重裝
次要版號：增加新功能，可以覆蓋安裝在原有版本，但有時也會有失敗風險
修正版本：微幅調整現有功能，通常可以覆蓋安裝

![](https://i.imgur.com/0chz1AI.jpg)


參考資料：
* https://rubygems.org/
* https://railsbook.tw/chapters/09-using-gems
* https://medium.com/never-hop-on-the-bandwagon/gemfile-and-gemfile-lock-in-ruby-65adc918b856