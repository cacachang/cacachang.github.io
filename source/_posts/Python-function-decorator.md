---
title: Python - 函式裝飾器 function decorator
date: 2024-04-21 23:29:34
category: Python
description: Python 中的 @ 是什麼？原來 function 也需要大衣來包裝自己！
---

今天想來跟大家介紹 function decorator

### Higher-order function

在這之前，先介紹一下什麼叫做 `Higher-order function`，就會比較好理解 function decortor

根據維基百科的介紹，`Higher-order function` 必須至少滿足以下其中一個條件

1. 至少有一個或多個 functions 作為引數
2. 結果要返回一個 function

> In mathematics and computer science, a higher-order function (HOF) is a function that does at least one of the following:
> 1. takes one or more functions as arguments (i.e. a procedural parameter, which is a parameter of a procedure that is itself a procedure),
> 2. returns a function as its result.

讓我們來看看例子吧

當我們呼叫 `outer_fun` 時，傳了一個 function 做為引數，符合了 `Higher-order function` 的第一條規範，這時我們可以說 `outer_fun` 是個 `Higher-order function`

```
def outer_fun(inner):
  print(inner_fun)

def inner_fun():
  print("I'm inner function")

outer_fun(inner_fun) // <function inner_fun at 0x1029eccc0>
```

* 不過如果我們在呼叫的時候，傳進去的引數不是 `function` ，然後也沒有 `return function` (第二個定義)，
`outer_fun` 就不能被稱作是個 `Higher-order function`

接著我們再稍微加工一下，在 `outer_fun` 中定義一個 function 並且 return 此 function，也就是 `inner_fun` ，這時候符合了第二點的規範，`outer_fun` 就是個 `Higher-order function` 了

```
def outer_fun():
  def inner_fun():
    print("I'm inner function")
  
  return inner_fun

outer_fun() // inner_fun 沒有執行，所以是空的
```

我們再來點進階的範例，我們來一步步看

1. 定義了一個 function `outer_fun`
2. foo 被指向 `outer_fun` ，帶入 3 為引數並且執行
3. 定義了一個 function `inner_fun`
4. 回傳 `inner_fun` 這個物件，此時 foo 是指向 `inner_fun` 這個物件

這時候的 `outer_fun` 也是回傳一個 function ，符合 `Higher-Order function` 第二個定義，因此可稱為 `Higher-Order function`

`outer_fun` 回傳的是 `inner_fun` 這個 function 物件，

並非 `inner_fun` 的執行結果，因此在執行程式碼並不會印出任何東西

```
def outer_fun(x):
  def inner_fun(x):
    calculator = x + x
    print(calculator)

  return inner_fun

foo = outer_fun(3)
```

現在的 foo 指向了 `inner_fun` 這個 function 物件，

如果我們想要 `inner_fun` 執行結果的話，只要執行 foo 並且帶入參數即可

這時候的 `outer_fun` 回傳的仍然是 `inner_fun` 這個 function 物件，因此仍為 `Higher-Order function`

1. 定義了一個 function `outer_fun`
2. foo 被指向 `outer_fun` ，帶入 3 為引數並且執行
3. 定義了一個 function `inner_fun`
4. 回傳 `inner_fun` 這個物件，此時 foo 是指向 `inner_fun` 這個物件
5. foo 執行，也就是 `inner_fun` 執行，帶入 3 作為引數並執行
6. 進入 `inner_fun`
7. 計算 引數相加 `x + x` ，算出結果為 3 ，並將 calculator 指向該結果
8. 印出 calculator 

```
def outer_fun(x):
  def inner_fun(x):
    calculator = x + x
    print(calculator)

  return inner_fun

foo = outer_fun(3)

foo(3) // 6
```

到這邊大家應該對於 `Higher-Order function` 多少有個概念了

再複習一下，`Higher-Order function` 需要符合以下其中一個定義

1. 至少有一個或多個 functions 作為引數
2. 結果要返回一個 function

那我們就帶著這兩個觀念，來看 `function decorator`

### Decorator pattern

`function decorator` 是一種設計模式，並不是只有存在於 Python ，其他程式語言，像是 JavaScript 或 Ruby 也會有，概念都是差不多的，只是差在於使用的頻率高不高

這個設計模式被稱作為 `Decorator pattern` 

我們來看看維基百科的介紹

在物件導向程式中， `Decorator pattern` 在不影響同個類別下的其他實體，可以讓獨立的物件添加其他行為

> In object-oriented programming, the decorator pattern is a design pattern that allows behavior to be added to an individual object, dynamically, without affecting the behavior of other instances of the same class.

我們來就拿第一個範例來看，為了方便了解，我會將 `outer_fun` 的程式碼改為 return inner


```
def outer_fun(inner):
  return inner

def inner_fun():
  print("I'm inner function")

outer_fun(inner_fun) // return inner_fun 該物件
```

為了要拿到傳進去 outer_fun 的 function 引數的執行結果，我們將程式碼這樣改

```
def outer_fun(inner):
  return inner()

def inner_fun():
  print("I'm inner function")

outer_fun(inner_fun) // I'm inner function
```

這樣看起來還算蠻單純的，不過這不算是 `Decorator pattern`，這只是一般的 `Higher-Order function`

來看看 GeeksforGeeks 的解釋：
`Decorator` 可以讓你動態的在物件上增加行為，並且當被包裝起來的那個 function (這邊稱 wrapper) 在調用的時候不會改變原本的行為

> Decorator Method is a Structural Design Pattern which allows you to dynamically attach new behaviors to objects without changing their implementation by placing these objects inside the wrapper objects that contains the behaviors.

如果今天專案中的許多地方都有用到 outer_fun ，改了 `outer_fun` 就可能會出錯，假設有個地方傳進去的是字串，他應該就會引發字串無法呼叫這個訊息

```
def outer_fun(inner):
  return inner()

def inner_fun():
  print("I'm inner function")

outer_fun("aa") // TypeError: 'str' object is not callable

```

這時候我們使用 function decorator 來改剛剛的例子，只要掛上這層外衣，就可以執行

* Python 提供了我們 `@` 的語法糖衣，讓 Decorator pattern 寫起來更易讀

```
def outer_fun(inner):
    return inner

@outer_fun
def inner_fun():
    print("I'm inner function")

inner_fun() // I'm inner function
```

即便今天有其他地方不是傳 function 進去，也不會噴錯

```
def outer_fun(inner):
    return inner

@outer_fun
def inner_fun():
    print("I'm inner function")

inner_fun() // I'm inner function
outer_fun(3) // 不會印出任何東西，會 return inner function
```

如果是比較複雜的狀況呢，讓我們來看看 BMI 計算機

以我自己的做法，會先處理身高，處理完再將它丟到 BMI 計算的 function 中去做計算

不過這樣在使用的時候就會傳一大堆參數，而且 `calculator(bmi_calculator, 50, 160) ` 這樣寫起來並不直觀

```
def calculator(func, weight, height):
    height_in_meters = height / 100
    return func(weight, height_in_meters)

def bmi_calculator(weight, height):
    bmi = weight / (height ** 2)
    return bmi

result = calculator(bmi_calculator, 50, 160)
print(result) // 19.531249999999996
```

如果是使用 Decorator Pattern 的方式

一開始我會將身高丟到 calculator 中，
並且在 calculate_height 中處理身高，
最後呼叫 bmi_calculator 來得到結果 

我們可以把 calculator 想像成一間代工廠，將所有的生產線用一個工廠包裝起來

而在這個工廠中，我會各別處理元件，處理完的元件會個別組裝起來，最後送出成品

```
def calculator(func):
  def calculate_height(weight, height):
    height_in_meters = height / 100
    
    return func(weight, height_in_meters)
  return calculate_height
 

def bmi_calculator(weight, height):
    return weight / (height * height)

decorated_bmi_calculator = calculator(bmi_calculator)
result = decorated_bmi_calculator(50, 160)
print(result) // 19.531249999999996
```

上述的方式，我個人認為已經簡潔許多，不過在呼叫的時候還是很麻煩，而且易讀性不高

我們來 `@` 這個語法糖衣來簡化一下

```
def calculator(func):
    def calculate_height(weight, height):
        def square_height(height):
            height_in_meters = height / 100

            return height_in_meters
        return func(weight, square_height(height))

    return calculate_height


@calculator
def bmi_calculator(weight, height):
    return weight / (height * height)

result = bmi_calculator(50, 160)
print(result) // 19.531249999999996
```

這樣就看起來簡潔又易讀囉！

### 總結 - 何時會需要用到 function decorator ?

再來回顧一下 Decorator pattern 的概念

`Decorator` 可以讓你動態的在物件上增加行為，並且當被包裝起來的那個 function (這邊稱 wrapper) 在調用的時候不會改變原本的行為

> Decorator Method is a Structural Design Pattern which allows you to dynamically attach new behaviors to objects without changing their implementation by placing these objects inside the wrapper objects that contains the behaviors.

我自己會歸類成幾個時候需要使用：

1. 不想改變原本的程式碼

就像剛剛的 `outer_fun` 跟 `inner_fun` 的範例

2. 有多種額外的行為需要做時

以剛剛的 BMI 計算機來看，我們需要先處理身高，再進行計算，這時候就很適合用 `function decorator`

不然用一般的方式需要傳入很多參數，且也不好閱讀，我們就可以用 `function decorator` 來優化

3. 使用套件的方法時

也許我們在做專案的時候用不太到 `function decorator` ，不過多少應該都會用到套件，

像是在 Flask 的路徑會使用到：

```
@app.route('/')
def index():
    return 'Hello, World!'
```

Django 的方法多少也會用到：

```
@login_required
def my_view(request):
    ...
```

參考資料：
https://www.geeksforgeeks.org/decorator-method-python-design-patterns/

https://en.wikipedia.org/wiki/Decorator_pattern

https://www.sitepoint.com/javascript-decorators-what-they-are/