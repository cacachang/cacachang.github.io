---
title: Python - 實體方法與類別方法的小故事
date: 2024-05-26 20:48:51
category: Python
description: Python 中的實體方法跟類別方法差異在哪裡？靜態方法又是什麼？
---

開發時難免會需要許多邏輯，為了讓程式碼好閱讀及好整理，通常我們會整理成方法

今天會來介紹 Python 中的幾種方法

* 實體方法 Instance Method
* 類別方法 Class Method
* 靜態方法 Static Method

理解這些方法以前，

我們先來看看什麼是 「 Class 類別 」、「 Instance 實體 」

Python 為物件導向語言 (Object Oriented Programming)

也就是說，在 Python 的世界中，幾乎所有的東西都是物件，

`物件` 會有 `狀態 (state)` 與 `行為 (behavior)`

包括我們待會會討論到的 `類別` 與 `實體` ，在 Python 中也都是物件

### Class 類別

在 Python 中，Class 類別就像是模型

我們可以依照模型來製造許多的成品(就是所謂的 Instance 實體)

### Instance 實體

從類別中產生出來的物件，會保有類別的 `狀態` 與 `行為`

### function 方法

在 Python 中，方法也是物件

現在我們就來看看方法的種類吧
### Instance Method 實體方法

看名字就像是實體專用的方法，不過真的是這樣嗎？

在 Python 中，當實體在使用方法的時候，會將自己做為方法的引數來呼叫，我們來看看例子

當 `ning` 作為 `getting_old` 的 receiver 的時候，

會把自己作為引數來做呼叫

```
class Person:
    def __init__(self, age):
        self.age = age

    def getting_old(self):
        self.age += 1
      
ning = Person(age=17)

ning.getting_old() // 18
```


這時候的 `getting_old` 會是一個 `bound method`

```
class Person:

    def __init__(self, age):
        self.age = age

    def getting_old(self):
        self.age += 1


ning = Person(age = 17)

print(ning.getting_old) // <bound method Person.getting_old of <__main__.Person object at 0x102f626c0>>
```

而 `bound method` 為 `method object` 的實體

據 Python 官方手冊對於 `method object` 的解釋：

當實體在呼叫該方法時，它會將實體作為第一個引數，並進行呼叫

> the special thing about methods is that the instance object is passed as the first argument of the function.

```
print(ning.getting_old.__class__) // <class 'method'>
```

那如果改成類別呢？

此時 `getting_old` 會是一個 function 物件

```
class Person:
    def __init__(self, age):
        self.age = age

    def getting_old(self):
        self.age += 1

print(Person.getting_old) // <function Person.getting_old at 0x100c40e00>
```

不過這時候就會出現錯誤訊息，因為 `getting_old` 必須要有一個參數(self)

```
class Person:
    def __init__(self, age):
        self.age = age

    def getting_old(self):
        self.age += 1

print(Person.getting_old()) // TypeError: Person.getting_old() missing 1 required positional argument: 'self'
```

為什麼同樣的方法，不同的 receiver 會變成不同的物件呢？

當物件找到方法後，會透過一個叫做 `__get__` 的方法來判斷物件是 `類別` 還是 `實體` ，並且回傳 `method-wrapper` 的實體

```
print(ning.getting_old.__get__) // <method-wrapper '__get__' of function object at 0x10498cd60>

print(Person.getting_old.__get__) // <method-wrapper '__get__' of function object at 0x10498cd60>
```


接著我們來看一下 `__init__` 初始化樣子

```
print(ning.getting_old.__init__) // <method-wrapper '__init__' of method object at 0x100af5ec0>

print(Person.getting_old.__init__) // <method-wrapper '__init__' of function object at 0x100af0e00>
```

在這邊我們就可以看到兩者的差異

用 ning(實體) 來存取方法，方法會被轉為 `method object`，也就是我們剛剛看到的 `bound method`
用 Person(類別) 來存取方法，一樣會是 `function object`

再複習一下 `bound method` 為方法將屬於該物件，可想像成是將方法綁在該物件上，只有該物件能做使用，並且在呼叫時，會將物件本身作為引數帶入

> A method is a function that “belongs to” an object.

而單純的 function object，即是指沒有被綁定在物件上、不屬於任何物件的 function


### Class Method 類別方法

除了實體方法，Python 中也有類別方法，聽起來也像是專門給類別使用的

不過說穿了，類別方法也只是一般的 function object 而已

用類別來呼叫方法時，會印出預期中的結果

```
class Person:
    def show_method():
        print("This is show method")

    ...

Person.show_method() // This is show method
```

這時候的 `show_method` 就只是一般的方法

```
class Person:
    def show_method():
        print("This is show method")

    ...

print(Person.show_method) // <function Person.show_method at 0x104e60cc0>
```

有實驗精神的我們，用 `__init__` 來看一下

的確是一般的 function object

```
print(Person.show_method.__init__) // <method-wrapper '__init__' of function object at 0x102364cc0>
```

如果換成是實體來作為 receiver 呢？

首先在 `__get__` 之後就被轉為 `method object`（也就是 `bound method`）

將方法綁在 ning 身上了

```
ning = Person(age=17)
print(ning.show_method.__init__) // <method-wrapper '__init__' of method object at 0x10040a040>
```

```
class Person:
    def show_method():
        print("This is show method")

    ...

ning = Person(age=17)
print(ning.show_method) // <bound method Person.show_method of <__main__.Person object at 0x10433a690>>
```

但由於 `bound method` 是需要將物件本身當作引數傳進 function 並呼叫，

類別方法並沒有任何參數，因此引發錯誤

```
class Person:
    def show_method():
        print("This is show method")

    ...

ning = Person(age=17)
ning.show_method() // TypeError: Person.show_method() takes 0 positional arguments but 1 was given
```

### 實體方法與類別方法

其實實體方法與類別方法，本質上都是單純的 function，甚至跟有沒有參數無關

差別在於存取及呼叫的時候：

如果 receiver 是實體，就會轉成 `method object` ，將方法綁定在物件上，並將物件作為引數帶入呼叫

如果 receiver 是類別，依舊為一般的 `function object`

所以並沒有實體不能使用類別方法，類別不能使用實體方法的規則，只是因為在呼叫時期的動作不同，就會引發錯誤


### @classmethod

由於版本相容性的問題 Python3 推出了一個裝飾器 `@classmethod` (忘記裝飾器可參考 [PYTHON - 函式裝飾器 FUNCTION DECORATOR](https://ninglab.com/Python-function-decorator/)

必須要使用 `cls` 作為參數帶入，這時候不管是實體還是類別，

存取的時候都變成了 `bound method`

```
class Person:

    @classmethod
    def show_method(cls):
        print("This is show method")
        
print(ning.show_method) // <bound method Person.show_method of <class '__main__.Person'>>
print(Person.show_method) // <bound method Person.show_method of <class '__main__.Person'>>
```

而實體跟類別都可以做使用

> A class method can be called either on the class (such as C.f()) or on an instance (such as C().f()).

當實體呼叫的時候，實體會被忽略，實際上是該實體的類別在呼叫，

所以實體呼叫時也不會有錯誤訊息，因為會去找該實體的類別



### Static Method 靜態方法

靜態方法不需要傳遞 類別 或者 實體 做為參數進去 function

通常沒有涉及到更改 類別 或 實體 的 attribute ，就可以用靜態方法來做

例如今天要計算傳進來的參數 * 100 等於多少，就很適合用靜態方法

(當然要傳 類別 或 實體 的 attribute 進去也可以，不過比起靜態方法，實體方法或類別方法可能更適合)


Python3 有提供靜態方法的裝飾器 `@staticmethod`

裝上去即可使用

```
class Person:

    @staticmethod
    def show_method(yoyo):
        print(yoyo*100)
        
ning = Person(age=17)
        
Person.show_method(10) // 1000
ning.show_method(ning.age) // 1700
```

參考資料：

https://docs.python.org/3/tutorial/classes.html

https://realpython.com/instance-class-and-static-methods-demystified/
