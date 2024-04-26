---
title: Rails 的預設環境變數存取方式 - Master Key
date: 2024-02-22 23:03:03
category: Rails
---

在 Rails 5.2 版本後，
Master key 就成了 Rails 內建的環境變數套件，用於管理環境變數

在 5.2 版本前， Rails 是使用 `config/secrets.yml` 或 `config/secrets.yml.erb` 來存取環境變數，且可以直接在任何環境中直接設定，無需針對特定環境做一個檔案，加上進版控後就會造成機密資料外洩的問題，因此後來拿掉了。

相較於我們常用的 dotenv ， master key 相對嚴謹的多


### 運作原理

我們可以將 `master.key` 想像成只有你擁有的鑰匙，鑰匙只能開啟你的保險箱 `credentials.yml.enc`

`master.key` 可以用來解密 `credentials.yml.enc`，但應該避免分享 master.key 或放進版控中，就像你不會把你家鑰匙給陌生人一樣


![](https://hackmd.io/_uploads/HySD3SOF3.jpg)


### 如何使用？

建立一個新的 Rails 應用程式時，會自動生成一個 master.key

在執行命令之前，需要設置編輯器 EDITOR，跟你的環境說該開哪個編輯器：

`export EDITOR="code"`

我們來下指令編輯環境變數：


```
EDITOR="code --wait" bin/rails credentials:edit
```

假設你對於 vim 比較熟悉，就用 vim 來開啟吧：

```
EDITOR="vim" bin/rails credentials:edit
```

會打開一個 yaml 檔案：

![](https://hackmd.io/_uploads/rJoTcSdKh.png)



* secret_key_base: 用於加密和解密像是 cookie 這樣的機密數據

在這個文件中，將你的環境變數加進去：

```
# credential.yml

aws:
  access_key_id: 123
  secret_access_key: 345
```

設定完就可以儲存並關閉

要驗證環境變數是否已正確存入，可以使用以下指令：

```
rails credentials:show
```

如果設置正確，Rails 會生成一個 credential.yml.enc 文件，並且你會得到以下結果：

```
aws:
  access_key_id: 123
  secret_access_key: 345

# 用作 Rails 中所有 MessageVerifiers 的基本密鑰，包括保護 cookie 的密鑰。
secret_key_base: dfcfa5ca64218b13d1dbf61745e420daea8ffeeef2a41eb8498c2378deb3bdbca91d5752ea97944f1a37b4458f7e7d535b91eef337010be67e664463e6e9457f
```

如果需要在 Rails 中存取環境變數，可以用這個方式：

```
Rails.application.credentials.aws.access_key_id
```

## English Version

Master Key is a tool used for managing environment variables in Rails applications.
It has been the default method for handling sensitive data since Rails 5.2.

Before Master Key, Rails used the `config/secrets.yml` or `config/secrets.yml.erb` to store sensitive information, but this method had security problems.


## How is Master Key working？

We can image the `master.key` file as a key that only you have, and `credentials.yml.enc` as a security box that stores valuable assets.

The `master.key` can open the encrypted `credentials.yml.enc` file, `master.key` file should not be shared or included in GitHub version control, just like you wouldn't give your house key to a stranger for security reasons.

![](https://hackmd.io/_uploads/HySD3SOF3.jpg)

## How to store confidential data using Master Key？

A `master.key` file is generated when creating a new rails application.

You can edit it using the command below:

* you need to set EDITOR as a key and pair the value with the code before, `export EDITOR="code"`.

```
EDITOR="code --wait" bin/rails credentials:edit
```

If you prefer using Vim instead of Visual Studio Code, you can use this command:

```
EDITOR="vim" bin/rails credentials:edit
```

This command opens the credential file, inside the file, you'll see a YAML structure:

![](https://hackmd.io/_uploads/HJuRcSuY2.png)


Let's take a look at the `secret_key_base`, which is used to encrypt and decrypt confidential data like cookies.

Put your key-value pairs for an environment variable in this file, for example:

```
# credential.yml

aws:
  access_key_id: 123
  secret_access_key: 345
```

Once you added environment variables, save and close the file.

To verify the environment variables were saved correctly, you can use the following command:

```
rails credentials:show
```

if everything was set up correctly, Rails will generate a `credential.yml.enc` file, and you'll get the result:

```
aws:
  access_key_id: 123
  secret_access_key: 345

# Used as the base secret for all MessageVerifiers in Rails, including the one protecting cookies.
secret_key_base: dfcfa5ca64218b13d1dbf61745e420daea8ffeeef2a41eb8498c2378deb3bdbca91d5752ea97944f1a37b4458f7e7d535b91eef337010be67e664463e6e9457f
```

Now, if you need to access an environment variable within your Rails application, you can use the following code:

```
Rails.application.credentials.aws.access_key_id
```

Congratulations on successfully setting up the environment variable through Master Key!
