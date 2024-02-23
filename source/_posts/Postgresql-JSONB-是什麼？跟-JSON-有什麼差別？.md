---
title: Postgresql - JSONB 是什麼？跟 JSON 有什麼差別？
date: 2024-02-22 22:52:07
category: 資料庫
---
## jsonb 是什麼 PostgreSQL 

支援的特有格式，而 b 指的是 binary ，因為 jsonb 是使用二進位來儲存格式 

## json 跟 jsonb 的差別 

![](https://hackmd.io/_uploads/SJTJ_rLI3.jpg) 

#### 什麼時候會用到 

![](https://hackmd.io/_uploads/ByuyRSLU2.jpg) 

## 在 rails 該怎麼使用 

1. new 一個專案 

``` 
> rails new jsonb_0602 
``` 

2. gemfile 安裝 pg，並 bundle 

``` 
# Gemfile 

gem 'pg', '~> 1.1' 
``` 

``` 
> bundle 
``` 

3. 把 database.yml 改成 pg 

``` # database.yml 

default: &default 
adapter: postgresql 
pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %> 
timeout: 5000 

development: <<: *default 
database: json_database 

test: <<: *default 
database: json_database 

production: <<: *default 
database: json_database 
``` 

4. 把資料庫建立起來 

``` 
> rails db:create 
``` 

5. 產生一個 model，例如發票 Receipt 設定其中一個欄位為 jsonb，例如發票資訊 `receipt_information` 設定完記得 migrate 

``` 
> rails g scaffold Receipt receipt_type:string receipt_information:jsonb 

> rails db:migrate 
``` 

5. 啟動 server 

``` 
> rails s 
``` 

6. 到新增頁面 `http://127.0.0.1:3000/receipts/new` 

7. 修改新增頁面，在 receipt_information 使用 fields_for 

``` 
# app/views/receipts/new.html.erb 
# app/views/receipts/_form.html.erb 

<%= form.label :receipt_information, style: "display: block" %> 
<%= form.fields_for :receipt_information do |note_form| %> 
  <%= note_form.text_field :gui_number, placeholder: '請輸入統編' %> 
  <%= note_form.text_field :company_name, placeholder: '請輸入公司名稱' %> 
  <%= note_form.text_field :address, placeholder: '請輸入公司地址' %> 
<% end %> 
``` 
8. params 清洗要這樣清洗 

``` 
def receipt_params 
  params.require(:receipt).permit(:receipt_type, receipt_information: [:gui_number, :company_name, :address]) 
end 
``` 

10. 到 console 確認資料 

```
> rails c

3.2.2 :002 > Receipt.last
[#<Receipt:0x000000010991dde0
  id: 1,
  receipt_type: "triplicate_uniform_invoice",
  receipt_information: {"gui_number"=>"11111111", "company_name"=>"xxx", "address"=>"台北市中正區"},
  created_at: Thu, 01 Jun 2023 15:59:08.733933000 UTC +00:00,
  updated_at: Thu, 01 Jun 2023 15:59:08.733933000 UTC +00:00>]
```

## 資料成功存到資料庫了，但驗證呢？ 

* Rails 不支援 jsonb 的驗證 
[Ruby JSON Schema Validator](https://github.com/voxpupuli/json-schema#ruby-json-schema-validator) 

使用教學： 

1. 安裝套件

```
# Gemfile

gem 'json-schema'
```

2. 建立 schema，設定完重開 server

```
# config/initializers/json_validator.rb

RECEIPT_INFORMATION_SCHEMA = {
  type: "object",
  required: [
      "gui_number",
      "company_name",
      "address"
  ],
  properties: {
      date: {
        type: "string",
        pattern: "^[0-9]{8}$"
      },
      company_name: {
        type: "string"
      }
      address: {
        type: "string"
      }
  }
}
```

3. 在 model require 套件，並設定 validate 方法

```
# app/models/receipt.rb

class Receipt < ApplicationRecord
  require "json-schema"

  before_validation :validate_information

  private

  def validate_information
    errors.add(:receipt, "Invalid information format. Please enter a valid information format.") unless JSON::Validator.validate(RECEIPT_INFORMATION, receipt_information)
  end
end
```

* 台灣的統編有一定規則，並非純 8 個數字而已，上述僅供範例使用，若要嚴謹請另外找套件或者參考統編規則寫驗證