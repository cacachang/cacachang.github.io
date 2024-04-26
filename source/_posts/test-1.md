---
title: 工程師的日常 - 從測試開始 PART 1
date: 2023-01-15 23:27:08
category: 測試
description: 搞定測試，開啟你的快樂開發之路
---

身為一個 Rails Developer
寫測試是必備的基本功

今天就來跟大家聊聊寫測試這件事

* 什麼是測試？
* 為什麼要寫測試？
* 測試分成哪些類型
* 有哪些工具可以幫我們完成測試？
* 測試該從哪開始？
* 3A 原則


## 什麼是測試？

我們來看看維基百科怎麼定義。

> 測試
  描述一種用來促進鑑定軟體的正確性、完整性、安全性和品質的過程。

寫出簡單明瞭、完整符合需求的程式碼，應該是所有工程師的理想目標

寫測試可以幫助我們達成這個目標
當然寫測試的目的並不只有這個

## 為什麼要寫測試？

寫測試的好處多多；壞處，我想不太到

1. 有明確的規格可遵循開發
    客戶開出需求給我們，通常是範圍極大，或者沒有聚焦於重點的要求
    這時候要做的是將需求轉換為規格，讓我們跟客戶達成共識，有了規格，在日後的開發也會有一套標準能遵循，避免日後的爭議
    
2. 了解程式的整個脈絡
    會將功能切分成好幾個小區塊，設想各種不同的情境，最後將整個情境串連起來，讓我們更能掌握整個程式碼運作的脈絡
    
3. 保證程式碼能運作
    自動化測試能確保我們寫出來的程式在其他情境下是跑得動的
    
4. 減少 bug 發生的機率
    經過不斷地修改、驗收測試，可預期使用者在流程中會遇到哪些 bug ，這些我們可以提前在測試中消除掉，且少了 bug 好處多多，有機會讓開發時程加速，以及重構應該也會更輕鬆哦！

## 測試分成哪些模式？

* 測試模式

1. 單元測試
    將功能切分成一個個小單位，以不同的狀況及情境去測試，這時候的測試不會碰到資料庫的資料，可使用變數來進行測試。
    舉例來說，今天要測試 我買了一件商品後，剩下多少錢
    就可以用變數來完成這個測試

    ```ruby
    RSpec.describe 'purchase', type: :feature do
      it 'Calculate immediately after shopping' do
        
        #用變數定義及運算
        my_money = 100
        milk = 40
          
        available_balance = my_money - milk
        
        expect(available_balance).to be 60
      end
    end
    ```
    ```
    $ rspec spec/features/test_spec.rb
    
    .Finished in 0.0106 seconds (files took 1.65 seconds to load)
    1 example, 0 failures
    ```
    
2. 整合測試
    是業界比較常寫到的測試，會需要針對資料庫或是外部系統去做資料的新增/讀取/修改/刪除，假設我們今天要做記帳系統，要去測試 使用者每花一次錢，帳戶剩下多少
    ```ruby
    RSpec.describe 'purchase', type: :feature do
      it "Calculate user's account immediately after shopping" do
        account_balance = Account.new(balance: 100)&.balance
        milk_price = Product.new(price: 40)&.price
        
        account_balance = account_balance - milk_price
        
        expect(account_balance).to be 60
      end
    end
    ```
    ```
    $ rspec spec/features/test_spec.rb
    
    Finished in 0.0198 seconds (files took 0.94423 seconds to load)
    1 example, 0 failures
    ```
    
3. 系統測試
    測試整個應用程式的所有功能，會包含外部的系統(如金流系統)一起測試
    
4. 使用者接受度測試
    應用程式是為了使用者的需求而寫出來的，好用與否也是使用者說的算，所以在上線前必須要先讓使用者測試


## 測試該從哪裡開始？

BDD 行動驅動開發

程式開發人員與測試人員一起將需求轉換成規格，
需要考量到不同的情境，規劃出不同的腳本，
並使用腳本進行測試及開發，
降低使用者及開發人員的資訊落差，
讓開發出來的功能符合使用者的需求。

TDD 測試驅動開發

當腳本完成後，就會開始進入測試流程，
不斷地測試、未通過、撰寫/修改程式碼，直到最後通過。


## 測試分成三大部分 3A 原則

分別為 Arrange / Act / Assert，我們拿剛剛的買牛奶例子來解釋

```ruby
RSpec.describe 'purchase', type: :feature do
  it 'Calculate immediately after shopping' do
    
    # Arrange 安排，將要測試的資料都備齊
    my_money = 100
    milk = 40
      
    # Act 執行，撰寫方法並測試
    available_balance = my_money - milk
    
    # Assert 驗收，執行結果是否符合預期規格
    expect(available_balance).to be 60
  end
end
```

測試通過也不能代表程式碼都是沒有問題的，
但是能盡量減少 bug 出現的機率，所以正確的開發流程必須包含測試。

如果開發完再補上測試，可能會有怎麼修都過不了的狀況(遇到會很痛ＱＱ)，
這代表程式碼並不符合規格，開發時間也會因此而延長。
