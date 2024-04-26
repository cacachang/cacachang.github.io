---
title: Basic - 環境變數
date: 2024-02-22 23:00:10
category: 程式通識
---

## 什麼是環境變數？ 

在我們開始之前，讓我們先了解環境是什麼 

### 環境 

環境是指 operating system 或 microservice，例如我們常用的終端機。 

### 變數 

在任何程式語言中，我們有兩個基本元素：常數和變數 他被會由一個 key 跟 value 配對而成 

常數：在執行期間不會被改變及重新定義（不過在某些語言中可能會被改變） 
變數：在執行期間更改和重新定義 常數和變數在記憶體空間會被配專屬的位置 

### 何時使用環境變數？ 

每個環境中都會有預設的環境變數 通常用於存放服務的用戶名及密碼、API路徑和其他機密資料 
application 在初始化期間載入這些環境變數 
在運行期間，key 會被替換為對應的 value，我們就可以輕鬆且安全地使用服務 

### 如何使用環境變數？ 

我們先來查看系統中已經存在的環境變數 

在終端機中下指令： 

```
> echo $PATH
```

顯示 PATH 環境變數的 value： 

```
xxx/.rvm/gems/ruby-3.2.2/bin:xxx/.rvm/gems/ruby-3.2.2@global/bin:/xxx/.rvm/rubies/ruby-3.2.2/bin 
```

接下來，我們使用 export 來設置環境變數： 

```
> export PASSWORD=password 
```

要顯示 PASSWORD 變數的值，我們一樣使用： 

```
> echo $PASSWORD 
```

這樣我們就成功設定了自定義的環境變數 

接下來想額外分享一個指令 

我們可以針對特定的環境設定環境變數，用以下指令： 

```
> PASSWORD=password irb 
```

在 irb 中查看我們剛剛設定的環境變數： 

```
> ENV["PASSWORD"] # password 
```

### 設定預設環境變數 

透過 export 或者在變數定義後加上特定的環境來設定環境變數，設定只能在當下的工作階段，一旦離開，就得重新設定環境變數。 

要確保環境變數存在於每個工作階段，我們需要在 .zshrc 或 .bashrc 檔案中進行配置。 

打開 zshrc 或 .bashrc 檔案（已有事先設定環境變數 EDITOR 為 code，還沒設定的話可以按照上個步驟去設定） 

以 zshrc 為例， bashrc 也可以照做： 

```
> code ~/.zshrc 
```

在 zshrc 檔案的最下面加環境變數並且儲存檔案： 

```
> export PASSWORD=password 
```

接下來開一個新的終端機，並查看環境變數： 

```
> echo $PASSWORD # password 
```

以上就完成設定了 

### 在應用程序中設置環境變數 

在不影響系統的環境變數下，該怎麼針對個別應用程式設定環境變數，有些套件可以幫我們達成這件事 以 dotenv 為例，我們只需在 .env 文件中放置一組變數。 

```
# .env.development.local 

# service 
HOSTNAME=127.0.0.1:3000 
```

在 application，我們可以使用這段程式碼來存取環境變數： 

```
ENV["HOSTNAME"] 
```

不過如果是要部署正式環境中，就必須要在各別環境中做設定，如何設定須取決於各個服務 
之後會再介紹環境變數更詳細的內容 


## English Version

## What is environment variable?

Before we start, let's understand the concept of environment.

![](https://hackmd.io/_uploads/SyyEno5wn.jpg)

## Environment

An Environment can refer to operating system or microservice, such as terminal that commonly used by developers, where commands can be executed.

## Variable

In any computer programming language, there are two fundamental components, constants and variables.

they are defined by a pair of key and value.

Constants remain unchanged when programming executing.
(however, it may changed in some language if you go out of your way to modify setting)

Variables can be modified and reassigned during program execution.

Both represent unique locations stored data and value in memory.

## When do we use environment variable?

Environment variable are predefined in each environment.

They are commonly used to store user name and password of service, API paths, even sensitive data.

Application loaded these environment variables during initialization.

During runtime, the keys are replaced with current values, enable us to access service easily while ensuring security.

## How do we use them?

First, let's check environmant variable which already present in system.

use the following command in terminal:

```
> echo $PATH
```
The value of PATH(environment variable) as shown below.

```
xxx/.rvm/gems/ruby-3.2.2/bin:xxx/.rvm/gems/ruby-3.2.2@global/bin:/xxx/.rvm/rubies/ruby-3.2.2/bin
```

Next, we use export command to set environment variable

```
> export PASSWORD=password
```
It's will display PASSWORD's value when we use command:

```
> echo $PASSWORD
```
By doing so, we have successfully set up a custom environment variable.

But I want to share another command with you.

We can add environment after variable assignment, setting key-value pair within within a specific environment.

```
> PASSWORD=password irb
```

once we enter the following command in irb, value will display.

```
> ENV["PASSWORD"]

# password
```

In the given situation, the steps are primarily for development purposes.

## Setting default environment variable

We can set environment variable with command export or add specific environment after variable assignment, which only present in current session.

then, how to set default environment variable that persist in each session?

At beginning, we need to add environment variable in zshrc file or .bashrc

open .bashrc or .zshrc file (I have previously set `EDITOR` as key and paired with value code in environment, you can try to follow the instruction provided earlier to set environment variable)

```
> code ~/.zshrc
```

put setting in the end of the file and save the file

```
> export PASSWORD=password
```

Now, you can see the environment variable in every session

```
> echo $PASSWORD
```

## Set environment variable in application

Package like dotenv can help us achieve the feature without affecting environment variable in system.

Using dotenv as an example, we can simply use it just put a bunch of variables in an `.env` file.

```
# .env.development.local

# server
HOSTNAME=127.0.0.1:3000
```

In application, we can access environment variables using the code.

```
ENV["HOSTNAME"]
```
If we want to deploy the application to a specific platform, we'll need to make further settings.

We will introduce deeper concept of environment variable in the remaining article, see u soon.
