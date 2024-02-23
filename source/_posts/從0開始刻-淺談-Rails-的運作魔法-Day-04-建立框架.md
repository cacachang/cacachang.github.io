---
title: 從0開始刻 淺談 Rails 的運作魔法 - Day 04 建立框架
date: 2022-08-17 23:41:48
category: Rebuild Rails
---
我們終於要開始來復刻 Rails 囉！

為你的框架取個好聽的名字吧！

我們這邊會用Rainbow作為名稱

*如果之後想把框架推到rubygem.org中，記得別用到重複的名字，而無法推上去


#### step 1 建立套件基本檔案

下指令後，終端機會跳出幾個問題


```shell=
# 是否要針對你的套件設定持續整合(CI)
Do you want to set up continuous integration for your gem? 
```

📃 web小辭典

▶ CI 持續整合( Continuous Integration )

開發應用程式時
我們會頻繁的修改功能、push檔案進行版控
並非每次push檔案都會順利的整合
以人工來編譯既耗時又沒效率

CI不僅可以幫我們自動化找出錯誤
還會再上線以前做測試
來確保程式碼是沒有問題的

▶ CD 持續部署( Continuous Deployment )

當CI測試沒問題後，就可以透過CD來自動化部署應用程式上線

比較常見的CI/CD工具：Github、Git


```shell=
# 輸入CI service
Enter a CI service. github/travis/gitlab/circle/(none): 
```

```shell=
# 要包含變更紀錄嗎？ (含錯誤修復、新功能等)
Do you want to include a changelog?
```

```shell=
# 要加入程式碼偵錯、格式檢查功能嗎？
Do you want to add a code linter and formatter to your gem?
```

```shell=
# 選擇 coding style 偵測工具
Enter a linter. rubocop/standard/(none): 
```

上述用好之後，就會生成套件基本的檔案
[點一下幫助回想GEM](https://ninglab.com/2022/08/16/%E5%BE%9E0%E9%96%8B%E5%A7%8B%E5%88%BB-%E6%B7%BA%E8%AB%87-Rails-%E7%9A%84%E9%81%8B%E4%BD%9C%E9%AD%94%E6%B3%95-Day03-GEM/)

```shell=
> bundle gem rainbow

create  rainbow/Gemfile
create  rainbow/lib/rainbow.rb
create  rainbow/lib/rainbow/version.rb
create  rainbow/sig/rainbow.rbs
create  rainbow/rainbow.gemspec
create  rainbow/Rakefile
create  rainbow/README.md
create  rainbow/bin/console
create  rainbow/bin/setup
create  rainbow/.gitignore
```

</br>

#### step 2 修改 README 檔案並安裝

到README.md檔案中
執行 Installation 裡面的指令

```md=
## Installation

# 安裝GEM 讓他加入到 Gemfile中
Install the gem and add to the application's Gemfile by executing:

    $ bundle add rainbow

# bundler沒有作用的話，就用gem安裝
If bundler is not being used to manage dependencies, install the gem by executing:

    $ gem install rainbow
    
```

```shell=
# 安裝成功

Fetching rainbow-3.1.1.gem
Successfully installed rainbow-3.1.1
Parsing documentation for rainbow-3.1.1
Installing ri documentation for rainbow-3.1.1
Done installing documentation for rainbow after 0 seconds
1 gem installed
```

</br>

#### step 3 到 gemspec 裡面設定套件資訊

務必要把TODO跟FIXME改掉
否則之後gem build的時候會出現錯誤訊息

```md=
Gem::Specification.new do |spec|
  spec.name = "rainbow"
  spec.version = Rainbow::VERSION
  spec.authors = ["Ning"]
  spec.email = ["ning@ninglab.com"]

  spec.summary = "Make coding colorful"
  spec.description = "A ruby frameword"
  spec.homepage = "https://github.com/cacachang/Rainbow"
  spec.required_ruby_version = ">= 2.6.0"

  spec.metadata["allowed_push_host"] = "https://ninglab.com/"

  spec.metadata["homepage_uri"] = "https://ninglab.com/"
  spec.metadata["source_code_uri"] = "https://github.com/cacachang/Rainbow"
  spec.metadata["changelog_uri"] = "https://github.com/cacachang/Rainbow"
```

</br>

#### step 4 加入 runtime depandency

前幾天有介紹到 Rack 在 Rails 中扮演著與HTTP溝通的橋樑
Rainbow 也會需要 Rack 來幫忙
我們先用 dependency 加入 Rack


```shell=
# rainbow.gemspec 最後一行加入
spec.add_runtime_dependency "rack"
```

📃 **web小辭典：dependency**

Rubygem 提供兩種 dependency
分別是 Runtime dependency 以及 Development dependency

Runtime dependency :套件需要哪些東西才能運作
Development dependency :通常會在修改套件功能時使用

</br>

#### step 5 設定套件版本

```ruby=
# lib/rainbow/version.rb
# 修改成 0.0.1版本
module Rainbow
  VERSION = "0.0.1"
end
```

</br>

#### step 6 建立 gemspec 及版本

```shell=
gem build rainbow.gemspec
```
```shell=
gem install rainbow-0.0.1.gem
```

▶ 假如失敗了
請回去檢查gemspec的TODO及URL是否都有填妥、填正確


基本設定都差不多了，不過要怎麼讓應用程式在這個框架中運作呢？
我們明天來做個簡單的小應用程式玩玩


參考文獻：
https://rebuilding-rails.com/
https://5xruby.tw/posts/rubocop-intro
https://guides.rubygems.org/patterns/
https://blog.kennycoder.io/2020/04/07/CI-CD-%E6%8C%81%E7%BA%8C%E6%80%A7%E6%95%B4%E5%90%88-%E9%83%A8%E7%BD%B2-%E5%9B%A0%E7%82%BA%E6%87%B6%EF%BC%8C%E6%89%80%E4%BB%A5%E6%9B%B4%E8%A6%81CI-CD%EF%BC%81%E6%A6%82%E5%BF%B5%E8%AC%9B%E8%A7%A3%EF%BC%81/
